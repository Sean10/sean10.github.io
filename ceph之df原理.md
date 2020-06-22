---
title: ceph之df原理
subtitle: df
date: 2020-05-24 20:04:33
updated:
tags: [ceph, 存储]
categories: [专业]
---


ceph df主要得到的是global以及每个pool的used以及max_available.

<!--more-->
``` c++
#Montor.cc文件入口ceph df
pgservice->dump_fs_stats(&ds, f.get(), verbose);
pgservice->dump_pool_stats(osdmon()->osdmap, &ds, f.get(), verbose);
```

## global
### FileStore
ceph上个版本使用filestore时，计算是基于osd的所在磁盘的statfs来计算的。因此，rbd层面的大的稀疏文件，在落到osd层上之后实际就并不占用那么多了。

``` c++
# 主体函数
PGMapDigest::dump_fs_stats
    osd_sum
PGMap::calc_stats
    osd_sum = osd_stat_t();
OSDService::update_osd_stat

KStore::statfs
```
### BlueStore
``` c++
PGMapDigest::dump_fs_stats
    osd_sum
PGMap::calc_stats
    osd_sum = osd_stat_t();
#疑似现在只有这个函数会刷新数据了
OSD::tick_without_osd_lock()
OSDService::set_statfs
BlueStore::statfs（找不到怎么那边怎么调用到bluestore的，直接来找同名函数了，理论上应该是继承的接口吧？）
BlueFS::get_free

bluefs这块还不太理解……
```


## pool
是否可以理解为，如果不使用ceph df提供的接口，手动去计算可用pg的容量，就可以直接代表可用容量？

### FileStore

``` c++
PGMapDigest::dump_pool_stats_full
PGMapDigest::dump_object_stat_sum
疑似是在osd写入时做的更新？
PrimaryLogPG::write_update_size_and_usage
```

### BlueStore
待阅读

``` c++
PGMapDigest::dump_pool_stats_full
    PGMap.cc: pool_raw_used_rate
PGMapDigest::dump_object_stat_sum   这里似乎计算的是去除降级后的？


仅在MgrStatMonitor::preprocess_statfs中出现了
PGMapDigest::get_statfs（这个像是已经deprecated?)
    PGMapDiest::get_pool_free_space

```


#### pool可用量计算


在代码中，在计算max_avail容量时，在`PGMap::get_rules_avail`函数中，mon会去迭代所有的资源池，根据`pool_id`拿到pool的如使用的`ruleno`的rule id、类型和size信息。拿到上述信息后，使用`ruleno`找到`crush map`信息，在这里建立一张基于crush map对各级bucket及osd进行广度优先搜索查找到的`map<int, float>`的`id`为key,`crush weight`为value的表，并进行归一化，让key更新为`weight/sum`的值。

然后在计算资源池整体可用时，使用上面拿到的osd的`kb_avail`除以此处拿到的表中osd id对应的归一化结果，取得资源池原始的最大可用空间。

现在要根据当前配置的副本数来获得对于资源池来说可用的空间，在`PGMapDigest::dump_pool_stats_full`函数中副本数变量为`raw_used_rate`。

* 针对`Replicated`类型，直接赋为资源池`size`
* 针对`Erasure`类型，从`erasure_code_profile`中获取到`k`和`m`，赋为$(k+m)/k$

最后直接得到资源池可用空间$\frac{max_avail}{raw_used_rate}$

#### pool使用量计算
在`PGMapDigest`的成员变量`mempool::pgmap::unordered_map<int32_t,pool_stat_t> pg_pool_sum`中保存着pool的pg的对应资源池中的使用量`num_bytes`的和。该数据在`ceph df detail`中进行汇总

```
struct pool_stat_t {
  object_stat_collection_t stats;
  ...
};

struct object_stat_collection_t {
  /**************************************************************************
   * WARNING: be sure to update the operator== when adding/removing fields! *
   **************************************************************************/
  object_stat_sum_t sum;
  ...
};

struct object_stat_sum_t {
  /**************************************************************************
   * WARNING: be sure to update operator==, floor, and split when
   * adding/removing fields!
   **************************************************************************/
  int64_t num_bytes;    // in bytes
  int64_t num_objects;
  int64_t num_object_clones;
  int64_t num_object_copies;  // num_objects * num_replicas
  int64_t num_objects_missing_on_primary;
  int64_t num_objects_degraded;
  int64_t num_objects_unfound;
  int64_t num_rd;
  int64_t num_rd_kb;
  int64_t num_wr;
  int64_t num_wr_kb;
  int64_t num_scrub_errors;	// total deep and shallow scrub errors
  int64_t num_objects_recovered;
  int64_t num_bytes_recovered;
  int64_t num_keys_recovered;
  int64_t num_shallow_scrub_errors;
  int64_t num_deep_scrub_errors;
  int64_t num_objects_dirty;
  int64_t num_whiteouts;
  int64_t num_objects_omap;
  int64_t num_objects_hit_set_archive;
  int64_t num_objects_misplaced;
  int64_t num_bytes_hit_set_archive;
  int64_t num_flush;
  int64_t num_flush_kb;
  int64_t num_evict;
  int64_t num_evict_kb;
  int64_t num_promote;
  int32_t num_flush_mode_high;  // 1 when in high flush mode, otherwise 0
  int32_t num_flush_mode_low;   // 1 when in low flush mode, otherwise 0
  int32_t num_evict_mode_some;  // 1 when in evict some mode, otherwise 0
  int32_t num_evict_mode_full;  // 1 when in evict full mode, otherwise 0
  int64_t num_objects_pinned;
  int64_t num_objects_missing;
  int64_t num_legacy_snapsets; ///< upper bound on pre-luminous-style SnapSets
  int64_t num_large_omap_objects = 0;
}
```

#### osd特殊场景
在osd out之后，该osd在2.4.1中提到的`kb_avail`会变成0，因此可用容量不需要计算

### max_avaible

* 单pool计算公式
资源池容量计算，主要在osd容量的基础上，遵循下述公式进行

$$max.avail = min(\frac{osd\_avail_0}{\frac{weight_0}{\sum_{i=0}^nweight_i}}, \frac{osd\_avail_1}{\frac{weight_1}{\sum_{i=0}^nweight_i}}, \dots, \frac{osd\_avail_j}{\frac{weight_j}{\sum_{i=0}^nweight_i}})/pool.size$$

* max_avail: 该资源池最大可用空间
* min: 取括号范围内的最小值
* osd_avail: 表示某个编号osd对应的可用空间
* weight: 表示对应某个编号osd对应的crush weight
* pool_size: 表示pool对应副本数(或纠删码经计算后的数值)

## rbd
### rbd du
需要开启`object-map fast-diff`功能之后统计拿到的数据才是正确的，否则就会出现每层快照占用的容量都会比原始数据要大

### rbd diff
``` bash
rbd diff rbd/zp | awk '{ SUM += $2 } END { print SUM/1024/1024 " MB" }'
```


## 异常情况

### osd down
对容量无影响
### osd out
~~在未达到min size阶段可继续使用的pool，只有osd正式out了，容量才会变更。~~

仅当被从crush中挪走或不存在in的osd，才会真的容量变化。

## Reference
1. [Ceph df分析\_运维\_weixin\_44389885的博客\-CSDN博客](https://blog.csdn.net/weixin_44389885/article/details/101478537?ops_request_misc=&request_id=&biz_id=102&utm_term=ceph%20df&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-101478537)
2. [ceph全局GLOBAL容量和POOLS级别容量计算\_awk\_pansaky的博客\-CSDN博客](https://blog.csdn.net/pansaky/article/details/86690230?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)