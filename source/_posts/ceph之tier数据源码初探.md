---
title: ceph之tier数据源码初探
subtitle: tier_source_code
date: 2020-07-04 21:07:29
updated:
tags: [ceph, tier, 源码]
categories: [专业]
---


# 理解


### rados实现思路

TODO: rados对象相比一个文件, 其实差别就在于rados对象自带了offset位置的概念. 并不存在太多其他的元数据信息.

所以基本可以忽略rados对象的元数据信息? onode等可能不行?

rados层的聚合, 即在tier上的时候, 表现就是存在这个rados对象, 但是这个rados对象只被识别出几K实际写入的对象大小. 而如果是聚合拼接时, 则意味着, 上层其实访问的也是这个大对象.  但是只访问这个对象中的一部分的时候, 怎么办呢?

tier层访问的rados对象还是小的. 但是每次落下去会合并成大的. 那问题就来了. 读上来的时候读上来一批小对象, 第二次合并的时候的小对象必然不会是第一次那种组合了. 会是一个新的组合. 这个时候要落hdd, 如果直接合并, 那就是一批新的. 除非有一个journal的实现, 每次请求都是一种journal. 

tier的删和读, 写, 都以4M为粒度最好?

这些索引数据, 预计是需要放在ssd上的. 

除非是对象存储那种, 只写一次, 所以他的小对象聚合, 不用考虑修改的问题. 只有写和读.

都是写少读多. 那理论上写完, 读的时候涉及部分小对象的逻辑? 难道通过预读来减少大合并文件中只有几个小文件才是用户要读的文件的问题?

所以针对这种通用场景, 只有一种办法, 那就是日志层级. 将随机写转换为顺序写, 然后通过compaction等方式把日志转化成实际的对象?

适合写多读少场景.

通过拆分整块写, 可以避免读那个被写了部分的offset的问题.

改数据的时候没问题, 通过gc来处理, 每次都写新的大对象. 

读的时候, 就通过预读, 调整预读的大小等来完成覆盖. 如果预读的效果不好, 那其实还不如直接就读这个4K对象, 如果能够分析出行为, 则可以试试预读512K, 1M等方式.


## tier理解边界

* 如果是以短时间的访问粒度, 拆解全随机和存在热数据比例?
* 即存取周期

* 无热数据, db,wal分离的hdd性能好.
  * 关键延时点: promote
* 有热数据, 开命中集tier好, 则tier能作为热数据访问的介质, 其他数据依旧访问hdd
  * 关键延时点proxy
* 待定
  * 有部分热数据, tier远比每日访问的数据大, 关闭命中集此时能让ssd提供比hdd更多的IOPS

主要拆解为
* 目标小io IOPS高, 延时低. 尽量让任意组合的ssd和hdd, 均能用出ssd的能力?

* 延时
  * 读延时
  * 写延时
* 命中率
  * 淘汰算法
  * 缓存大小, 热数据大小

## 命中率

ssd的容量大小, 其实取决于热数据到底有多少? 刚好卡在热数据范围最好.

如果完全没有热数据情况下, 即便都落ssd, 也价值不大?

缓存击穿时, 我感觉还是一种关键, 但是主要是需要硬盘不成为瓶颈的情况下, 这样按理论上就不会受到flush和evict影响, 剩下的就是writeback和writethrough的差异了

TODO: 单纯命令集的效果的话, 应该就是ssd承接部分命中的请求, 未命中的请求全落hdd.

这样ssd的使用率如果命中的请求打不满, 那就浪费很大. 所以可能需要配合小对象都落ssd的方案, 这样就可以提高ssd的利用率? 这个方案应该是主流方案吧?

## 问题
* 所有rbd, 超出tier大小性能差, 随着越大, 性能越差, 直到下限
  * 多个rbd, 并不会存储冷热数据(指的是淘汰策略), 命中次数多的依旧会被踢掉. 表现是flush /evict很多
* tier不满的情况下, 对象不存在的第一次写时, 缓存层应该能全部缓住. 但是启动hit_set时, 存在与刚才那句预期不符.  promote应该只产生一次请求就可以了. 

* 缓存击穿定义是?
  * 目前现场的压力时确实有点大.
  * 导致IOPS上不去的一个因素, 是否就是这种击穿状态下, IOPS被flush和evict占用的?
* G5那种, 每天预计访问1-2T数据, 而缓存池20T以上, 总数据100T+, 是否能保障使用?
  * 1个T数据, 今天, 因为tier足够大, 在ceph的热数据概念下, 其实这批数据不算热数据.
  * 这种其实也是一种智能promote的能力, 如果缓存池比较大的情况下.(持疑问)

## 拆解

* 性能 跟延时有关, 以及队列深度.
  * IOPS, 1ms=1000IOPS
  * 多队列情况下, 就有一定的放大.
  * 固态, 支持多队列, 一般队列深度是8, 总IOPS会相比单队列强很多. 在平均延时可能上涨一部分的情况下.
    * 多队列. 
    * nvme的队列深度一般是32/64.
  * 软件层面, 内核提供的aio的syscall, aio_submit, aio_getevent这套能把一个队列中的多个op异步下发. 
  * 软件层面的这种多队列怎么实现, 应该还是依托上面固态的硬件的.


* promote
  * 对象进入blocked状态, 等待取上来
* flush+evict
  * flush本身, 如果盘的压力没满, 会阻塞write. (这个是个影响点, 可能影响的是evict)
    * flush和write同一个对象, 会阻塞. (待定)
    * 这种情况下如果发现阻塞了, 重新入队, 如果稳态测试这个场景, 理论上性能应该会相比普通场景性能会下降一笔. 
  * evict是目前影响最大的, 淘汰算法.
    * 跟hit_set有关
    * 应该是会阻塞write.

### 状态定义

* 写满
* 常态
  * 指的是访问的部分对象需要从底下promote 
  * 淘汰算法已经在常态生效 
    * 即我们场景里的flush+evict状态
* 击穿, 最差性能
  * 因为底下太多, 实际上基本命中不了了, 每次都是promote. 即底下hdd的性能.
  * 让他直接落hdd反而更好

* tier的大小, 是应该跟着业务, 让他起到部分命中的效果.


* 根据上面的evict, 应该就需要针对业务, 提高命中率的方案, 在对象粒度不变的情况下, 可以提高性能.




开了命中集之后的方案来进行. 

* 新写对象的性能
  * 其实可以当做问题来解决, 这些对象不应该落到hdd上. 
  * 全写新对象的测试场景, 4K, 只要没被flush过一次, 就不会走到需要hdd读一遍的情况
  * fio 新对象
* 这批对象全部写满了, 小于tier大小, 然后全部不在tier上.
  * 手动写的librbd, 每个4M只写其中1个4K, 延时肯定都是hdd+ssd的延时, 那IOPS比裸压hdd的IOPS更低.
  * rados flush-and-evict.
  * 走自己的librbd的下入, 每个4M, 1024次4K. 第一次肯定是不命中. 第一遍所有请求都是不重复的4M的. 第二次才开始剩下的. 
  * 剩下的平均延时应该就是1次hdd+1023次ssd . 理论上限. 

* rbd_index分离, rbd_header一整个rbd也才读一次, 应该不至于有什么影响.

* 常态逻辑下需要支持的场景
    * 分散对象, 即大部分场景不存在重复命中, 可能是将来深度学习模型能够解决的问题
    * 重复命中的问题, 然后控制重复命中的频次和比例, 比如说100%对象都重复1次, 或者60%的对象重复一次, LRU的命中效率是多少.
      * 1G的内存 和 2G的对象,,LRU: 可能60%

    * 几个用来的计算的参数
      * * tier大小与业务总数据大小关系
      * 重复对象的比例, 以及重复次数
      * 现在的管理对象粒度(现在是固定的4M)

* 加个指标, 命中的范围.  只是举例

* 0-1ms,  10%
* 1-10ms, 20%
* 10ms-20ms, 10%

* 小对象, 提高对象粒度


# 分析

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



# 命中集



``` c

    if (hit_set)
      agent_estimate_temp(soid, &temp);
    agent_state->temp_hist.add(temp);
    agent_state->temp_hist.get_position_micro(temp, &temp_lower, &temp_upper);
```


`pow2_hist_t`: histogram of ages we've encountered.

应该是位运算. cbits, 64字节, 值为1, bin得到的就是1, 值为2, 则得到的是2, 因为要去除前导的0, 值为4, 得到3, 8得到4.

hist_t里存的啥? 好像low和up得到的就是bin前面的, 和加上bin之后的差值.

`agent_estimate_temp`这里添加的热度? 

迭代历史的热度, 增加热度统计 int last_n = pool.info.hit_set_search_last_n; `grade_table` 看以下代码, 历史的所有命中集的热度是统一对待的. 

``` c++
void calc_grade_table() {
    unsigned v = 1000000;
    grade_table.resize(hit_set_count);
    for (unsigned i = 0; i < hit_set_count; i++) {
      v = v * (1 - (hit_set_grade_decay_rate / 100.0));
      grade_table[i] = v;
    }
  }
```

如果历史几次命中集都有命中, 则热度偏高一点. 

穿进去的热度v应该是1000000以上或者0. 这里应该是牺牲准确度, 但是减少空间消耗. 1000000应该会被归在20位, 即bin[20] = 1

然后算upper和lower的时候, 如果没有其他的命中过, 则是lower 0 , upper 1
total = 1
即lowwer 0, upper 1000000

如果前面有个热度不高的, 不考虑具体热度多少, 只考虑当前这个热度, 在那个阶段的比例?比如, 命中了比较多的, 那1000000有20次, 2000000有4次, 此次这个对象的热度是1500000 则lower 20 , upper 4
lower `20/44*1000000, upper 24/44*1000000`, 最后得到一个450000 和 550000这种比例.


TODO: 这里的hist是以什么为单位的? 热度统一控制的目的?

好像又回到`evict_effort`了, 假设当前evict_effort是500000, 则这种因为upper较高, , 按照说的是统一正交化了的. 

只有当前对象所处热度的那个位时较高时, 才不会被踢掉. 

agent_state这个对象的粒度? 

`PrimaryLogPG`为单位的, 那就是pg为粒度. 这个pg每处理一个对象就Add一下.

每agent_work一次就加一次`hist_age`,  默认参数1000, 也就是1000次读写, 就降级一次hist?... 所以这里关心的是pg内全局的对象热度, 是命中的比较多, 还是不命中的比较多.命中的比较多时, 为啥就会被evict?

`osd_agent_min_evict_effort`这个默认是0.1, 即如果是0.81的数据量, 0.8为水线, 则取较大的, 即0.1, 而不是0.01/0.2=0.05的比例作为effort. 这里应该是为了取要踢多少对象.

超过full_ratio的情况, 需要均一化. 但是下面这段并没有完全做均一化, 默认只处理掉0.1的话, 完全存在超过1.1的比例的情况. 

这里其实是把比例按100000为粒度合并, 避免太多差异值. 所以full_ratio那里有计算, 如果直接进入FULL状态, 就不判断热度了.

另外这里的这个按位运算方式, 肯定有个什么术语...热度梯度统计?

``` c++
    // quantize effort to avoid too much reordering in the agent_queue.
    uint64_t inc = cct->_conf->osd_agent_quantize_effort * 1000000;
    ceph_assert(inc > 0);
    uint64_t was = evict_effort;
    evict_effort -= evict_effort % inc;
    if (evict_effort < inc)
      evict_effort = inc;
    ceph_assert(evict_effort >= inc && evict_effort <= 1000000);
```

比如2000000对应21位, 如果这个命中热度的对象, 出现频次较高时, 

`TierAgentState`, 


hitset的淘汰逻辑?


## Todo
### write_full\truncate等OP类型实际指代的是什么?

### pg counter有类似`pg map`的同步周期?
### 计算什么时候会达到平衡的时候, 预计应该是会与tier的flush速度+evict速度 是否等同于上层下发的写io速度相平衡? 也就是我们需要得到evict速度的增长曲线, 得到evict与容量占比的计算公式, 然后计算上flush的计算速度与osd, 盘性能的相关性, 再计算一下业务的压力, 就可以得到到底设置多大的上水线可以让这个缓存池满足使用了.

### TierAgentState



### ignore_cache和ignore_overlay逻辑


* flush
* try_flush
* redirect_reply
* do_proxy_read
* do_proxy_write
  * 

``` c++

标签: OPERATION_IGNORE_CACHE

    op.tier_flush();
    unsigned flags = librados::OPERATION_IGNORE_CACHE;
    int r = context->io_ctx.aio_operate(context->prefix+oid, completion,
					&op, flags, NULL);


static int do_cache_try_flush(IoCtx& io_ctx, string oid)
{
  ObjectReadOperation op;
  op.cache_try_flush();
  librados::AioCompletion *completion =
    librados::Rados::aio_create_completion();
  io_ctx.aio_operate(oid.c_str(), completion, &op,
		     librados::OPERATION_IGNORE_CACHE |
		     librados::OPERATION_IGNORE_OVERLAY |
		     librados::OPERATION_SKIPRWLOCKS,
		     NULL);
  completion->wait_for_complete();
  int r = completion->get_return_value();
  completion->release();
  return r;
}


do_cache_evict


  switch (obc->obs.oi.manifest.type) {
  case object_manifest_t::TYPE_REDIRECT:
    if (op->may_write() || write_ordered) {
      do_proxy_write(op, obc);


    if (op->may_write() || op->may_cache()) {
      do_proxy_write(op);

      // Promote too?
      if (!op->need_skip_promote() &&
          maybe_promote(obc, missing_oid, oloc, in_hit_set,
	              pool.info.min_write_recency_for_promote,
		      OpRequestRef(),
		      promote_obc)) {
	return cache_result_t::BLOCKED_PROMOTE;
      }
      return cache_result_t::HANDLED_PROXY;
    } 
```




## histgram算法

> 分位数近似计算  
> 分位数近似算法有很多种，比如 HdrHistogram 算法、q-digest 算法、GK 算法、CKMS 算法、T-Digest 算法等，其中 HdrHistogram 算法和 T-Digest 算法在软件系统中使用的比较多，T-Digest 算法用于 ElasticSearch、Kylin 等系统中，HdrHistogram 的简化版用于 Prometheus 中。下面我们简单介绍一下这两种常用算法：
> 静态分桶  
> 思想：将整个存储区域以规律性的区间划分为多个桶，整个规律性的区间可以是线性增长，也可以是指数增长。每个桶只记录落在该区间的采样数量，计算分位数时，会假设每个区间也是线性分布，从而计算出具体的百分位点的数值。这样通过牺牲小部分精度，达到减小空间占用，并且统计结果大致准确的结果。
> 典型的实现是：https://github.com/HdrHistogram/HdrHistogram。所以后续也称之为 Histogram 算法。
> 缺点：统计范围有限，需要预先确定，不能改变
> 


> 两种数据增长算法：Linear 和 Log2，Linear 是线性增长，适合对百分位数精度要求比较高，而且数据范围比较小的场景。Log2 是指数增长，适合对百分位数精度要求相对低，而且总的数据范围跨度较大的场景。当然精度大小还依赖于坐标增长单元




Ceph RBD 性能及 IO 模型统计追踪功能设计与实现
原文链接： https://www.infoq.cn/article/2ojeh5jgo4s1xkxiztcg

# tier实践讨论

[\[ceph\-users\] SSDs for journals vs SSDs for a cache tier, which is better?](https://ceph-users.ceph.narkive.com/roHS8cpX/ssds-for-journals-vs-ssds-for-a-cache-tier-which-is-better#post1)


# Reference
1. <ceph源码分析>
2. [Ceph 学习——OSD读写流程与源码分析（一）\_SEU\_PAN的博客\-CSDN博客\_primarylogpg](https://blog.csdn.net/CSND_PAN/article/details/78743426)
3. [openstack - What is the best size for cache tier in Ceph? - Stack Overflow](https://stackoverflow.com/questions/61311797/what-is-the-best-size-for-cache-tier-in-ceph)
4. [Ceph Tiring Cache 调优 \| Elvis Zhang](https://blog.shunzi.tech/post/ceph-tiring-cache-optimization/)
5. [Re: \[ceph\-users\] Regarding cache tier understanding](https://www.mail-archive.com/ceph-users@lists.ceph.com/msg11984.html)