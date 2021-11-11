---
title: ceph之tier数据源码初探
subtitle: tier_source_code
date: 2020-07-04 21:07:29
updated:
tags: [ceph, tier, 源码]
categories: [专业]
---


## 分析关键点
### 埋点统计接口
``` bash
# tier_flush\tier_promote等数据

ceph daemon osd.{id} perf dump | grep tier

# num_write\num_read\num_promote等数据
ceph pg dump_json
```

### tier主要触发任务
#### 数据迁移
下面的`flush`和`evict`动作主要根据热度命中集和当前空间占用的`ratio`有关.
* flush
* evict
  * 主要zero, 因此主要是在ssd上的写.
  * 置零, 从op的角度应该也有什么东西能够观察到吧.
* promote(感觉只和这个有关系了)

#### IO操作
根据`flush_mode`和`evict_mode`状态判断触发下面的`may_read`,`can_proxy_write`等操作的成功与否.

下面的每一个操作好像都是`may`或者`can`的, 看上去像是有开关之类的.

* write_back
  * 读:
    * 缓存不存在
      * read proxy(osd层的proxy_read怎么转换到pg层上?)
      * read forward(会redirct, 统计数据应该是直接记录到base pool上的pg层的read_num上)
      * 这两个是怎么选择的呢?
    * 缓存命中 (cache pool的read)
  * 写
    * 缓存不命中
      * write_proxy(proxy_write)
      * promote_object
    * 缓存命中 (cache pool的write)

## 源码阅读过程

### tier_proxy_read数据来源
tier_proxy_read这个数据是哪里统计增加的?是在`finish_proxy_read`里增加的,那么同理

[osd: tiering: add proxy read support · ceph/ceph@70d3d08](https://github.com/ceph/ceph/commit/70d3d08a0b2ac2f922ee8eaf5a7f261b919e6dd4)

这次提交好像是个核心开发者的提交,当时不是走的github的pr?

看了下代码,这里好像没有计数器,inc

虽然是在ceph daemon osd.x perf dump里拿到的数据,但实际上还是pg层的,只是这个数据pg那边的接口暂时没找到能直接打印出来的方式.不对,这里虽然是PG类里的,但是数据记在了这个pg所属的osd上.

[osd: add proxy write perf counter · ceph/ceph@b9ec7e6](https://github.com/ceph/ceph/commit/b9ec7e64b7f59a92343ddc095e0a6b6c0dc50bfb)

[Revision b9ec7e64 \- osd: add proxy write perf counter Signed\-off\-by: Zhiqiang Wang <zhiqiang\.wan\.\.\. \- Ceph \- Ceph](https://tracker.ceph.com/projects/ceph/repository/revisions/b9ec7e64b7f59a92343ddc095e0a6b6c0dc50bfb)

tracker单里咋也啥都没有呢?那他们怎么决定的呢?

在这个提交里增加的计数器.这个是直接提交在master分支里的了...哎


int PrimaryLogPG::do_osd_ops(OpContext *ctx, vector<OSDOp>& ops)

write这边是尝试写,或者跟tier有关的都会进行计数

而read这边则是读一些元数据信息会触发读计数

这样的话,write和read数据就没有那么准确了.


pg的counter好像不够实时, 有一个类似`pg map`的同步周期的样子,待核对.



rmw_flags flags这个是判断到底要不要promote,和may_proxy_write的结果判断的



bool can_proxy_write = get_osdmap()->get_up_osd_features() &
    CEPH_FEATURE_OSD_PROXY_WRITE_FEATURES;


    uint64_t OSDMap::get_up_osd_features() const
{
  return cached_up_osd_features;
}
void OSDMap::_calc_up_osd_features()
在这个函数里做的赋值,

[osd/OSDMap: cache get\_up\_osd\_features · Mirantis/ceph@e0e765f](https://github.com/Mirantis/ceph/commit/e0e765f9dd84be7af65887a101740f9fce22c3bd)

在这次提交里做的改名,改成`cached_up_osd_features`,之前叫`uint64_t OSDMap::get_up_osd_features() const`, 每次都是重新查.

[osd/OSDMap: get\_up\_osd\_features\(\) · Mirantis/ceph@1d8429d](https://github.com/Mirantis/ceph/commit/1d8429de57c07a739ee6b58ba3de35874e7d85c0)

age Weil authored and Yan, Zheng committed on Dec 29, 2013 

struct osd_xinfo_t {

变量在osdMap这个class里
  mempool::osdmap::vector<osd_xinfo_t> osd_xinfo;


### pg stats


### pool stat方向

dump时用的iops_wr和iops_rd

int64_t iops_wr = pos_delta

是从pg_sum_delta来的

``` c++
  mempool::pgmap::list< pair<pool_stat_t, utime_t> > pg_sum_deltas;


void PGMap::update_global_delta(CephContext *cct,
                                const utime_t ts, const pool_stat_t& pg_sum_old)
{
  update_delta(cct, ts, pg_sum_old, &stamp, pg_sum, &pg_sum_delta,
               &stamp_delta, &pg_sum_deltas);
}



  /* Aggregate current delta, and take out the last seen delta (if any) to
   * average it out.
   * Skip calculating delta while sum was not synchronized.
   */
  if(!old_pool_sum.stats.sum.is_zero()) {
    delta_avg_list->push_back(make_pair(d,delta_t));
    *result_ts_delta += delta_t;
    result_pool_delta->stats.add(d.stats);
  }
  size_t s = cct ? cct->_conf->get_val<uint64_t>("mon_stat_smooth_intervals") : 1;
  if (delta_avg_list->size() > s) {
    result_pool_delta->stats.sub(delta_avg_list->front().first.stats);
    *result_ts_delta -= delta_avg_list->front().second;
    delta_avg_list->pop_front();
  }

//这里add,实际对应的就是底层对每个num_write之类的变量做加减.

  mempool::pgmap::list<pair<pool_stat_t,utime_t> > *delta_avg_list)

//这里的减,减去的是上一次收集的数据?

  auto ts = per_pool_sum_deltas_stamps.find(p->first);
  assert(ts != per_pool_sum_deltas_stamps.end());
  client_io_rate_summary(f, out, p->second.first, ts->second);

//这里应该是pgmap维持那张pg_sum_deltas list里的stamp

  if (!stamp.is_zero() && !pg_sum_old.stats.sum.is_zero()) {
    utime_t delta_t;
    delta_t = inc.stamp;
    delta_t -= stamp;
    // calculate a delta, and average over the last 2 deltas.
    pool_stat_t d = pg_sum;
    d.stats.sub(pg_sum_old.stats);
    pg_sum_deltas.push_back(make_pair(d, delta_t));
    stamp_delta += delta_t;

//这里的inc.stamp是什么东西?



  list<Incremental*> inc;
  Incremental::generate_test_instances(inc);


  class Incremental {
  public:
    MEMPOOL_CLASS_HELPERS();
    version_t version;
    mempool::pgmap::map<pg_t,pg_stat_t> pg_stat_updates;
    epoch_t osdmap_epoch;
    epoch_t pg_scan;  // osdmap epoch
    mempool::pgmap::set<pg_t> pg_remove;
    float full_ratio;
    float nearfull_ratio;
    utime_t stamp;


        pool_stat_t d = pg_sum;


void PGMapDigest::decode(bufferlist::iterator& p)

void ClusterState::update_delta_stats()
{
  pending_inc.stamp = ceph_clock_now();
  pending_inc.version = pg_map.version + 1; // to make apply_incremental happy
  dout(10) << " v" << pending_inc.version << dendl;

  dout(30) << " pg_map before:\n";
  JSONFormatter jf(true);
  jf.dump_object("pg_map", pg_map);
  jf.flush(*_dout);
  *_dout << dendl;
  dout(30) << " incremental:\n";
  JSONFormatter jf(true);
  jf.dump_object("pending_inc", pending_inc);
  jf.flush(*_dout);
  *_dout << dendl;

  pg_map.apply_incremental(g_ceph_context, pending_inc);
  pending_inc = PGMap::Incremental();
}
```
所以pool_stat数据的更新,基本上就是和这个pg_sum数据的更新一致的.

这里的`stamp`周期到底是多长呢?


PGMap::apply_incremental 

像是在这merge的数据?

在PGMonitor::tick里会更新这个delta的数据.

## 理论
因为太多人不推荐这个方案了, 所以看看有没有推荐的.

### 推荐ssd用于WAL, 然后剩下的根据热数据大小控制缓存分层的大小[^3]
所以理论上控制好, 还是有用的?



# 下刷
* flush_target
* evict_target  PrimaryLogPG.cc:13468
* cache_target_full_ratio_micro
* uint64_t flush_target = pool.info.cache_target_dirty_ratio_micro;
        uint64_t flush_high_target = pool.info.cache_target_dirty_high_ratio_micro;
        uint64_t flush_slop = (float) flush_target * cct->_conf->osd_agent_slop;
*         uint64_t evict_target = pool.info.cache_target_full_ratio_micro;
        uint64_t evict_slop = (float) evict_target * cct->_conf->osd_agent_slop;
* agent_choose_mode

这次新增了flush_high  
> commit 8f6056aebbabcbe236d332f546d075e06a14c0ca
> Author: MingXin Liu <mingxin.liu@kylin-cloud.com>
> Date:   Thu May 28 14:33:10 2015 +0800
> 
>     Osd: revise agent_choose_mode() to track the flush mode
>     
>     Signed-off-by: Mingxin Liu <mingxinliu@ubuntukylin.com>
>     Reviewed-by: Li Wang <liwang@ubuntukylin.com>
>     Suggested-by: Nick Fisk <nick@fisk.me.uk>

commit c9daf8e5ea401f5bc2aafd4025991fb4903ffcd4
Author: Sage Weil <sage@inktank.com>
Date:   Mon Jan 27 17:57:53 2014 -0800

    osd/ReplicatedPG: add slop to agent mode selection
    
    We want to avoid a situation where the agent clicks on and off when the
    system hovers around a utilization threshold.  Particularly for trim,
    the system can expend a lot of energy doing a minimal amount of work when
    the effort level is low.  To avoid this, enable when we are some amount
    above the threshold, and do not turn off until we are the same amount below
    the target.
    
    Signed-off-by: Sage Weil <sage@inktank.com>

## 理解

> 该值需要根据缓存池的容量大小以及副本个数来设置，以三副本为例，target_max_bytes 不应该超过容量的 1/3，如果实际的负载使得存储池中的数据大小达到了容量的 1/3，后续的 IO 将被阻塞，所以需要设置别的参数来避免池中的数据到达该阈值。

为什么是1/3?

> 该类参数的设计目的：
> 作为刷回淘汰操作的触发条件，避免 OSD 被数据撑满。
> 为什么不直接使用存储池的容量作为该参数，是为了考虑另外一种场景，存在多个缓存池，使用相同的磁盘。

## flush 逻辑[^5]
> Agent will be always in idle state if target_max_bytes or
> target_max_objects not set on the pool irrespective of other tiering params
> set in the pool. dirty_micro and full_micro will not be calculated if those
> two params are zero which is by default I guess.

> Now, flush will be activated if dirty_micro is > flush_target. My
> understanding is, once it is activated it will iterate through all the dirty
> objects and flush all the dirty objects which is > cache_min_flush_age. Am I
> right ?

>> 7.       The cache_min_flush_age will only be applicable if the flush is
>> triggered after crossing the dirty_threshold, right ? If dirty_threshold is
>> not breached, the flush age param is never checked.


## evict逻辑
>> 6.       I saw the cache_min_evict_age is not been used anywhere, am I
>> missing anything ?
> 
> It's possible.  The _min_ params are a bit dangerous because they can 
> potentially confuse the cache agent (e.g., what if *all* objects are under 
> the min?  How/when do we decide to ignore the min, or, how/when do we 
> give up trying to find an older object?).


## blocked 时间

``` c++
            uint64_t over = full_micro - evict_target;
            uint64_t span = 1000000 - evict_target;
            evict_effort = MAX(over * 1000000 / span,
                               (unsigned) (1000000.0 * cct->_conf->osd_agent_min_evict_effort));
```
``` c++
            full_micro =
                    num_user_objects * avg_size * 1000000 /
                    MAX(pool.info.target_max_bytes / divisor, 1)
```


根据目前的理解, 当full_ratio达到1000000时, 根据上文, 也就是只有当达到`target_max_bytes`的时候才会彻底阻塞, 在这之前都是不会停止写io的.

TODO: 目前看到的是, flush和evict的速度, 基本取决于osd的业务压力, 会让存储空间瓶颈停留在80%.



### `evict_effort`与实际速度的计算

根据默认配置可知, `osd_agent_min_evict_effort`默认值为`0.1`,即默认的`evict_effort`为`0.1`.

`osd_agent_quantize_effort`默认值同样为`0.1`

``` c++
uint64_t inc = cct->_conf->osd_agent_quantize_effort * 1000000;
            assert(inc > 0);
            uint64_t was = evict_effort;
            evict_effort -= evict_effort % inc;
            if (evict_effort < inc)
                evict_effort = inc;
            assert(evict_effort >= inc && evict_effort <= 1000000);
```
看起来这段只是在凑`0.1-1`以`0.1`为单元.

>> if (full_micro > evict_target), the mode is set as
>> TierAgentState::EVICT_MODE_SOME. In this scenario evict_effort is calculated
>> and based on hit_set and temp calculation some clean objects are evicted. My
>> question is , can we quantify this value ? For ex, if the target_full_ratio
>> = 50%, once the eviction is triggered, what %objects will be evicted ?
> 
> The effort is calculated based on how far between target_full and 100% we 
> are.  This is mapped onto the estimation of atime.  We generate a 
> histogram of age for the objects we have examined, so after the agent has 
> run a bit we'll have a reasonable idea how the current object's age 
> compares to others and can decide whether this is older or newer than the 
> others; based on that we decide what to do.

## maybe_handle_cache
> see maybe_handle_cache().  That's 
> not strictly agent behavior per se.  Also, there is now a readforward mode 
> that doesn't promote on read ever, based on our discussion about the 
> performance of flash on read.

## 调用链

* agent_choose_mode
* OSDService::agent_entry()
  * PrimaryLogPG::agent_work
    * PG::agent_work(int max, int agent_flush_quota) = 0
    * PG::agent_work(int max) = 0
      * agent_maybe_evict
      * agent_maybe_flush


## Todo
### write_full\truncate等OP类型实际指代的是什么?

### pg counter有类似`pg map`的同步周期?
### 计算什么时候会达到平衡的时候, 预计应该是会与tier的flush速度+evict速度 是否等同于上层下发的写io速度相平衡? 也就是我们需要得到evict速度的增长曲线, 得到evict与容量占比的计算公式, 然后计算上flush的计算速度与osd, 盘性能的相关性, 再计算一下业务的压力, 就可以得到到底设置多大的上水线可以让这个缓存池满足使用了.

### TierAgentState



## Reference
1. <ceph源码分析>
2. [Ceph 学习——OSD读写流程与源码分析（一）\_SEU\_PAN的博客\-CSDN博客\_primarylogpg](https://blog.csdn.net/CSND_PAN/article/details/78743426)
3. [openstack - What is the best size for cache tier in Ceph? - Stack Overflow](https://stackoverflow.com/questions/61311797/what-is-the-best-size-for-cache-tier-in-ceph)
4. [Ceph Tiring Cache 调优 \| Elvis Zhang](https://blog.shunzi.tech/post/ceph-tiring-cache-optimization/)
5. [Re: \[ceph\-users\] Regarding cache tier understanding](https://www.mail-archive.com/ceph-users@lists.ceph.com/msg11984.html)