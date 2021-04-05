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





#### FileStore

``` c++
PGMapDigest::dump_pool_stats_full
PGMapDigest::dump_object_stat_sum
疑似是在osd写入时做的更新？
PrimaryLogPG::write_update_size_and_usage
```

#### BlueStore
待阅读

``` c++
PGMapDigest::dump_pool_stats_full
    PGMap.cc: pool_raw_used_rate
PGMapDigest::dump_object_stat_sum   这里似乎计算的是去除降级后的？


仅在MgrStatMonitor::preprocess_statfs中出现了
PGMapDigest::get_statfs（这个像是已经deprecated?)
    PGMapDiest::get_pool_free_space

```
### Nautilus变化
根据[^5]可以知道

> * ceph df [detail] output (POOLS section) has been modified in json
>   format:
> * 'bytes used' column renamed to 'stored'. Represents amount of data
>   stored by the user.
> * 'raw bytes used' column renamed to "stored_raw". Totals of user data
>    over all OSD excluding degraded.
> * new 'bytes_used' column now represent amount of space allocated by 
>   all OSD nodes.
> * 'kb_used' column - the same as 'bytes_used' but in KB.
> * new column 'compress_bytes_used' - amount of space allocated for compressed
>   data. I.e. comrpessed data plus all the allocation, replication and erasure
>   coding overhead.
> * new column 'compress_under_bytes' amount of data passed through compression
>  (summed over all replicas) and beneficial enough to be stored in a
>   compressed form.

所以在这个版本中将之前版本中存在的多种容量统计给分类了.

pool级别分为
* stored(old bytes used) 逻辑空间占用量, 主要针对的是写入多少, 不考虑由于`COW`引起的实际占用量较小
  * store_nromalized = pool_stat.get_user_bytes(raw_used_rate, per_pool)
    * store_stats.data_stored / raw_used_rate
      * Bluestore.cc
        * res_statfs->data_stored += l.length
    * 传统模式 stats.sum.num_bytes + stats.sum.num_bytes_hit_set_archive; 
      * 12版本好像没加命中集归档的容量
  * 12版本这里直接拿的`sum.num_bytes`
  * 通过ceph report可以看到资源池的这个统计
* stored_raw(old raw bytes used) 用户使用的总容量, 包含所有osd除去降级的(这个应该是带副本的吧)
  * pool_stat.get_user_bytes(1.0, per_pool)
  * 这里应该是统计store_stats.data_stored的时候一些down的osd就不统计了, 所以为1.0的`raw_used_rate`.
* new bytes used 实际这个资源池里的OSD分配的空间大小(这里应该是不包含其他资源池因为复用osd占用的空间吧?)
  * pool_stat.get_allocated_bytes
    * store_stats.allocated;  
* compress_bytes_used compress的数据 , 分配的空间大小(包含副本/EC的开销), 也就是说除去副本数就是用户数据实际在设备里存储吧
  * statfs.data_compressed_allocated
* comprress_under_bytes compress的用户数据大小?
  * statfs.compressed_original




``` dot
diagraph calltrace {
  Monitor::handle_command -> mgrstatmon()->dump_cluster_stats
  Monitor::handle_command -> mgrstatmon()->dump_pool_stats -> PGMapDigest::dump_pool_stats_full -> pool_stat_t &stat = pg_pool_sum.at(pool_id)
  pool_stat_t &stat = pg_pool_sum.at(pool_id) -> pool_stat.get_allocated_bytes
  pool_stat_t &stat = pg_pool_sum.at(pool_id) -> store_nromalized = pool_stat.get_user_bytes(raw_used_rate, per_pool)
  pool_stat_t &stat = pg_pool_sum.at(pool_id) -> pool_stat.get_used_bytes(1.0, per_pool)
  pool_stat_t &stat = pg_pool_sum.at(pool_id) -> statfs.data_compressed_allocated
  pool_stat_t &stat = pg_pool_sum.at(pool_id) -> statfs.compressed_original
}
```

``` c++
struct pool_stat_t {
  object_stat_collection_t stats;
  store_statfs_t store_stats;
  ...
}
```
#### 变量解释
* raw_used_rate
* num_object_copies
* num_object_degraded
* per_pool
  * osd_sum.num_osds == osd_sum.num_per_pool_osds
  * [mon: use per-pool stats only when all OSDs are reporting · ceph/ceph@5dcb6d8](https://github.com/ceph/ceph/commit/5dcb6d81bbc2a4c4d0da4a33d9f6bbba5065a1ad)
  * [osd: per-pool osd stats collection by ifed01 · Pull Request #19454 · ceph/ceph](https://github.com/ceph/ceph/pull/19454)
    * 这里应该是引入点, 为了解决压缩率统计的问题?
  * [Backport #37565: luminous: OSD compression: incorrect display of the used disk space - bluestore - Ceph](https://tracker.ceph.com/issues/37565)
    * Luminous无法被backport. 哎
* data_stored
* num_bytes 
  * 这个还是在`PrimaryLogPG.cc`里进行统计
  * PG::_update_calc_stats
  * OSD::send_pg_stats






#### todo

问题:
  * 资源池显示的使用量实际上并不是osd上实际分配和占用的, 那为什么资源池显示已用满了基本上就不让用了呢? 还是说这两个基本上是一致的?
    * 按照bluestore这层应该也是和`FileStore`时一样, 实际上是有洞的,这种情况下, 为什么不默认提供更多的空间呢? 是怕出问题吧? 只有当压缩的时候, 才提供?
  * compressed的容量是否
  * used和 objects数量是否有关呢
    * 像是rbd上层4K随机写, 在rados层看到的就是写了2W5的对象以后就100G了, rados还是按照4M对象创建了...
    * 按照之前我们看pg的op, 理论上应该和objects数量无关才对.
    * pg处理的是Rados层的请求, 所以其实问题在于rbd层的4k 是怎么转换到4M的.
    * 问题其实是写了这些对象以后, df看到的容量就满了, 而实际上osd还没满.
  * mon如何打印出他当前收集的容量信息呢?
  * 
### Luminous


#### pool可用量计算


在代码中，在计算max_avail容量时，在`PGMap::get_rules_avail`函数中，mon会去迭代所有的资源池，根据`pool_id`拿到pool的如使用的`ruleno`的rule id、类型和size信息。拿到上述信息后，使用`ruleno`找到`crush map`信息，在这里建立一张基于crush map对各级bucket及osd进行广度优先搜索查找到的`map<int, float>`的`id`为key,`crush weight`为value的表，并进行归一化，让key更新为`weight/sum`的值。

然后在计算资源池整体可用时，使用上面拿到的osd的`kb_avail`除以此处拿到的表中osd id对应的归一化结果，取得资源池原始的最大可用空间。

现在要根据当前配置的副本数来获得对于资源池来说可用的空间，在`PGMapDigest::dump_pool_stats_full`函数中副本数变量为`raw_used_rate`。

* 针对`Replicated`类型，直接赋为资源池`size`
* 针对`Erasure`类型，从`erasure_code_profile`中获取到`k`和`m`，赋为$(k+m)/k$

最后直接得到资源池可用空间$\frac{max_avail}{raw_used_rate}$

#### pool使用量计算
在`PGMapDigest`的成员变量`mempool::pgmap::unordered_map<int32_t,pool_stat_t> pg_pool_sum`中保存着pool的pg的对应资源池中的使用量`num_bytes`的和。该数据在`ceph df detail`中进行汇总

可以用下面这个命令看到这些统计
``` bash
ceph report
```

``` c++
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

#### osd使用的空间
12.2.13这部分函数似乎被重构了, `update_osd_stat`没有入参了.

``` c++
void OSD::heartbeat()
->
void OSDService::update_osd_stat(vector<int>& hb_peers)
-> 
int BlueStore::statfs(struct store_statfs_t *buf)
这里通过指针设置了`Compress`和`compress_origin`的变量.


```
这里有个`available`, 为什么压缩后, 这里显示的减去之后获取到的容量还是实际使用了`compress_origin`的空间?

``` c++
struct store_statfs_t 
```

这里的`available`是什么时候计算的?

``` bash
 ceph daemon osd.1 pref dump | grep bluestore
```
* bluestore_allocated
* bluestore_stored
* bluestore_compressed
* bluestore_compressed_allocated
* bluestore_compressed_original

这里的容量分别是什么意思?

11.76-9.89=1.86g

这个`compressed`容量只有在占用正式结束了才会统计? 在流式使用空间的过程中, osd_used在逐渐增加, 但是这个compress一直没变.

似乎这里把`struct store_statfs_t stbuf`拿到的`stbuf`给通过`OSDService::set_osd_stat`给传过去了, 然后这里的值就是`osd_stat`的计数器里的值了. 稍后就直接传给`mon`了.
好像初始化的时候, osd默认就占用`1.01G`空间?

`num_bytes`似乎和`osd_stat`里的`used`和`available`没什么关系?目前看到的现象是`ceph osd df`里看到的osd的`used`只有18.8G, 但是`ceph df`看到的资源池的使用量是`61.6G`, 3个osd, 2副本. 而我真实写入的文件大小是用`/dev/zero`生成的21G文件+ 一堆之前的, `ceph df`看到的是用户的操作

我关闭Compress之后, osd的used增加大概是3个G, 差不多欸, 我写的是10G对象, 拆到每个osd上. 理论上应该有7个G左右.

而那几个开了compress的大概是增加1个G.

`sum.num_bytes`在哪里计算的呢?

`PGMapDigest`下的`public`成员变量`mempoool::pgmap::unordered_map<int32_t,pool_stat_t> pg_pool_sum`. 应该是这里更新的.
`auto it2 = pg_pool_sum.find(it.first)`->`const pool_stat_t *pstat`->`const object_stat_sum_t& sum`


`void PGMap::update_pool_deltas(`这个有点像, 不过这里计算的应该是差值吧? `pg_pool_sum`里的差值有哪些?应该不是

尝试搜索`num_bytes = `, 搜了下, 没有, 那应该意味着是通过聚合性质的做的计算, 按照`C++`的限制,应该是会通过字符串作为key的列表, 转换成对应的变量. emm, 只找到`dump`时的字符串与原始计数器 和 一个mon这里的`pcb.add_u64(`的任务, 这里的这个`mon`的东西是全局统计, 通过`PGMonitor::update_logger`这里更新.

啊, 忘了, 应该搜`num_bytes +=`, 在`PrimaryLogPG.cc`里出现了osd的统计`ctx->delta_stats->num_bytes`, 在`osd_types.cc`里也出现了操作符重载的`void object_stat_sum_t::add`的操作.

然后`publish_stats_to_osd`

pg_info_t -> pg_stat_t -> pg_stats_publish -> osd_stat_updated = true -> send_pg_stats -> m->pg_stat[pg->info.pgid.pgid] = pg->pg_stats_publish;



> 这里会将变化的pg的stat_queue_item入队到pg_stat_queue中。然后设置osd_stat_updated为True。入队之后，由tick_timer在C_Tick_WithoutOSDLock这个ctx中通过send_pg_stats()将PG的状态发送给Monitor。这样Monitor就可以知道pg的的变化了。

mon哪里处理这个接收到的最新的pg数据呢?
搜索这个msg结构体中的`.pg_stat`, 看到个下面这个, 是不是`ClusterState::ingest_pgstats`呢, 的确是这个, 在`DaemonServer::ms_dispatch`中调用这个更新的数据

在`ClusterState`中定义了`PGMap::Incremental pending_inc;`这里填充的`pg_stat_updates`

在比如`PGMap::apply_incremental`的好多地方都直接处理了这个更新的数据. 这样我就不好找到底把df的内容存哪去了.

最后应该是存到了`PGMap`里的`mempool::pgmap::unordered_map<pg_t,pg_stat_t> pg_stat里.

这个pgmap应该是哪里实例化的呢?

PGMonitor ? 这个的初始化的`Monitor`在`ceph_mon.cc`里, 

似乎初始化`ceph-mon`的时候只读取了`version_t v = store->get("monmap", "last_committed");`? 

那数据难道还是在osd上? 而且mon数据丢失似乎是可以恢复过来的.

通过ceph pg dump --format json-pretty是可以导出`num_bytes`的, 而`daemon osd.x perf dump`反而拿不到. 

通过`ceph-objectstore-tool`可以拿到`OSD里存的pg`的信息

#### ceph-objectstore-tool查看OSD数据库中bluestore pg信息
如何解密然后修改呢

export pg信息似乎走的是`RadosDump`

欸,如果我要修改某个值, 要停掉这个pg所在的所有osd?

* ObjectStoreTool::do_export
  * get_log
    * PGLog::IndexedLog
      * 这个为啥不能用pg_log_t解码呢?

根据[^9这个来看,其实还是遵守了kv数据结构的,只是好像在哪层被过滤掉了?

试了下, get-bytes拿到的是对象的内容

那么是否说明其实也是计算, 而不是累加的呢?

info这个数据似乎无从修改, 只有pg的元数据可以修改的样子?

这个疑似PGLog::IndexedLog, 不知道怎么改

按照现在看到的objects的问题, 是不是这个pg提供的objects指标全都是负数,所以上层统计的也都是

但是现在在数据库里list, list不出这个pg, 是代表什么呢?

我应该只能从代码到底是怎么list, 用的什么接口然后查不到这个pg来看吧?

用`rados ls -p data02`, 因为这个资源池理论上其实不应该有任何object, 所以应该也是查不到东西, 但是实际存在了几个对象信息为负数的object.

以osd级别来显示object, 看看带不带pg的信息.

pg上面统计的信息, 感觉像是以object为单位处理的啊. 假如说真是以pg自己来存储自己的信息, 那么object和pg之间就存在歧义性了.

ceph pg dump显示的pg id不完整, 被坑了. 


* scrub是一个方案, 看看能不能修复他, 根据EC
* 现在pg的指标是负数, 还有什么信息能够用来处理呢?object无法查到
  * rados ls 卡住,无论哪个, 可能受影响了
  * ceph-objectstore-tool 查询这个pg, list也是空的


* ceph pg repair pg.$id 居然就修复了. 所以repair过程发生了什么
* todo
  * pgid居然是As1

* librados: wait_for_osdmap done waiting
  * 是否代表某个客户端占用了这个吧所?
  * 打开--debug_objecter 40 --debug_rados 40看到的内容就是上述的.
  * objecter给出的信息更多一点, 看着
  * 这中间慢在哪里呢?ms_dispatch


#### ceph osd df
* SIZE 没问题
* USE 是上面的store呢还是allocated?
* AVAIL 又是哪种呢?

在正常情况下ceph osd df 看到的osd的used是跟ceph df看到的stored一致的, 但是这个数据是在osd这里实际存储的, 而ceph df却并不是

这里看到的osd的容量是

* `pgs->get_osd_sum().kb`
  * PGStatService *pgs
  * PGMapStatService
    * get_osd_sum()
      * return pgmap.osd_sum;
      * pgmap怎么更新? 指向的是ClusterState中的pg_map
        * mgr
          * ClusterState::ingest_pgstats
          * ClusterState::notify_osdmap
        * mon
          * tick()
            * send_report()

根据[^8]里可以看到这个`MonPGStatService`是`ceph-mon`把内容转嫁到`ceph-mgr`的中间层.

#### pool使用的空间 used
所以根据上面来看, 这边的容量并不是根据`osd_stat`汇总的了, 而是`pg`这层自己维护的?

根据[^7]


* bytes used 逻辑空间占用量, 主要针对的是写入多少, 不考虑由于`COW`引起的实际占用量较小
  * 12版本这里直接拿的`sum.num_bytes`
  * 通过ceph report可以看到资源池的这个统计
* raw bytes used 用户使用的总容量, 包含所有osd除去降级的(这个应该是带副本的吧)
  * pool_stat.get_user_bytes(1.0, per_pool)
* compress_bytes_used compress的数据 , 分配的空间大小(包含副本/EC的开销), 也就是说除去副本数就是用户数据实际在设备里存储吧
  * statfs.data_compressed_allocated
  * 12版本还在osd里
* comprress_under_bytes compress的用户数据大小?
  * statfs.compressed_original
  * 

sum.num_bytes属于`pool_stat_t`里的`object_stat_collection_t stats`, 这个东西怎么从`pg_map`里得到的?

在`PGMap::calc_stats`里似乎`pg_sum = pool_stat_t()`, 但是这个`pool_stat_t`的构造函数里只是构造并初始化为0, emm, 只在`decode`的时候被调用, 倒是的确没什么影响.



#### cephfs使用的空间
cephfs上删除太多文件, 出现了ceph df看到的used为`EiB`的情况, 通过`json`打印出来, 看到实际上统计的`bytes`为负数, 引起的.

根据[^6], 好像说是和`cephfs`文件系统有关系? 


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

根据[^4], 好像`rbd`的`discard`选项, 影响这个容量的问题.

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
2. [ceph全局GLOBAL容量和POOLS级别容量计算\_awk\_pansaky的博客\-CSDN博客](https://blog.csdn.net/pansaky/article/details/86690230?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)$$
3. [Ceph中的容量计算与管理 \- gold叠 \- 博客园](https://www.cnblogs.com/goldd/p/6610618.html)
4. [kubernetes - How to explain Ceph space usage - Stack Overflow](https://stackoverflow.com/questions/55962248/how-to-explain-ceph-space-usage)
5. [osd: per-pool osd stats collection by ifed01 · Pull Request #19454 · ceph/ceph](https://github.com/ceph/ceph/pull/19454/files)
6. [Bug #16671: "ceph df"object num is negative - Ceph - Ceph](https://tracker.ceph.com/issues/16671)
7. [Ceph中的容量计算与管理 - gold叠 - 博客园](https://www.cnblogs.com/goldd/p/6610618.html)
8. [https://runsisi.com/2019-02-27/ceph-pg-stat](https://runsisi.com/2019-02-27/ceph-pg-stat)
9. [pglog出错导致osd启动失败的解决办法 \- 简书](https://www.jianshu.com/p/174c9d376b83)