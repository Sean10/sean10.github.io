---
title: ceph之L版本purged_snaps过多问题
tags:
  - ceph
  - snap
categories:
  - 专业
subtitle: tech
date: 2023-03-22 22:06:44
updated:
---


# 背景及初步定位

背景是通过perf捕获ceph-osd的cpu消耗, 发现在下述环节极高.

* `PG::_prepare_write_info`
	* interval_set<snapid_t>::operator==
		* `_Rb_tree_increment`

由于找不到现场版本的`debuginfo`了, 因此姑且先推导一下.

interset的`==`, 对应是map的`==`, `_Rb_tree_increment`初步定位应该是 map的迭代器调用的? 

``` c++
  bool operator==(const interval_set& other) const {
    return _size == other._size && m == other.m;
  }
```


``` c
_Self&
operator++() _GLIBCXX_NOEXCEPT
{
    _M_node = _Rb_tree_increment(_M_node);
    return *this;
}
```

根据上述Map的内部实现来看, 基本就是他了, 比较map, 根据下述stl实现来看, 确实是`operator==`产生的大量迭代


``` c++
//迭代器基类
struct __rb_tree_base_iterator
{
  typedef __rb_tree_node_base::base_ptr base_ptr;
  typedef bidirectional_iterator_tag iterator_category;
  typedef ptrdiff_t difference_type;

  base_ptr node;    //节点基类类型的指针，将迭代器连接到RB-tree的节点

  void increment()
  {
    if (node->right != 0) {//如果node右子树不为空，则找到右子树的最左子节点
      node = node->right;
      while (node->left != 0)
        node = node->left;
    }
    else {//如果node右子树为空，则找到第一个“该节点位于其左子树”的节点
      base_ptr y = node->parent;
      while (node == y->right) {
        node = y;
        y = y->parent;
      }
      if (node->right != y)
        node = y;
    }
  }





class Map
...
      typedef _Rb_tree<key_type, value_type, _Select1st<value_type>,
		       key_compare, _Pair_alloc_type> _Rep_type;

      /// The actual tree structure.
      _Rep_type _M_t;


  /**
   *  @brief  Map equality comparison.
   *  @param  __x  A %map.
   *  @param  __y  A %map of the same type as @a x.
   *  @return  True iff the size and elements of the maps are equal.
   *
   *  This is an equivalence relation.  It is linear in the size of the
   *  maps.  Maps are considered equivalent if their sizes are equal,
   *  and if corresponding elements compare equal.
  */
  template<typename _Key, typename _Tp, typename _Compare, typename _Alloc>
    inline bool
    operator==(const map<_Key, _Tp, _Compare, _Alloc>& __x,
	       const map<_Key, _Tp, _Compare, _Alloc>& __y)
    { return __x._M_t == __y._M_t; }


...

# stl_tree.h
class _Rb_tree
...
	friend bool
      operator==(const _Rb_tree& __x, const _Rb_tree& __y)
      {
		  return __x.size() == __y.size()
		  && std::equal(__x.begin(), __x.end(), __y.begin());
      }

# 根据[std::equal \- cppreference\.com](https://en.cppreference.com/w/cpp/algorithm/equal)可知, equal内的实现必然用到了`operator++` 
...

      _Self&
      operator++() _GLIBCXX_NOEXCEPT
      {
		_M_node = _Rb_tree_increment(_M_node);
		return *this;
      }

      _Self
      operator++(int) _GLIBCXX_NOEXCEPT
      {
		_Self __tmp = *this;
		_M_node = _Rb_tree_increment(_M_node);
		return __tmp;
      }


```



# 定位到purged_snaps后

ceph的快照删除官方文档如下, 因此`purged_snaps`高在业务场景很正常.

> SNAP REMOVAL
> To remove a snapshot, a request is made to the Monitor cluster to add the snapshot id to the list of purged snaps (or to remove it from the set of pool snaps in the case of pool snaps). In either case, the PG adds the snap to its snap_trimq for trimming.
> 
> A clone can be removed when all of its snaps have been removed. In order to determine which clones might need to be removed upon snap removal, we maintain a mapping from snap to hobject_t using the SnapMapper.
> 
> See PrimaryLogPG::SnapTrimmer, SnapMapper
> 
> This trimming is performed asynchronously by the snap_trim_wq while the pg is clean and not scrubbing.
> 
> The next snap in PG::snap_trimq is selected for trimming
> 
> We determine the next object for trimming out of PG::snap_mapper. For each object, we create a log entry and repop updating the object info and the snap set (including adjusting the overlaps). If the object is a clone which no longer belongs to any live snapshots, it is removed here. (See PrimaryLogPG::trim_object() when new_snaps is empty.)
> 
> We also locally update our SnapMapper instance with the object’s new snaps.
> 
> The log entry containing the modification of the object also contains the new set of snaps, which the replica uses to update its own SnapMapper instance.
> 
> The primary shares the info with the replica, which persists the new set of purged_snaps along with the rest of the info.
> 
> RECOVERY
> Because the trim operations are implemented using repops and log entries, normal pg peering and recovery maintain the snap trimmer operations with the caveat that push and removal operations need to update the local SnapMapper instance. If the purged_snaps update is lost, we merely retrim a now empty snap.


目前可以通过ceph pg 2.892 query 查到对应pg的purged_snaps, 足足有2412条, 确实数量很大, 对应的迭代高也看来合理.

~~那剩下就是为啥有的osd 有这个的情况下cpu消耗不高了? ~~该问题可忽略, 后定位节点间cpu型号性能有差异, osd cpu消耗低的节点cpu性能确实更好.


### `_prepare_write_info`的 call trace的触发逻辑?


主要是`pglog`的正常IO处理流程

[ceph中PGLog处理流程 \| Ivanzz](https://ivanzz1001.github.io/records/post/ceph/2019/02/05/ceph-src-code-part14_1)


# 高版本修复了该设计问题


```
  removed_snaps_queue [8f1~1,8f4~1]
  
```

查看16版本环境, 发现好像是改成了removed_snaps_queue, 且purged_snaps在`pg query`里也看不到了.

初步看, 虽然pg_info还比较`purged_snaps`, 但是这项大部分时间为空了. 即不存在该版本的问题了


根据这条PR[mon,osd,osdc: refactor snap trimming \(phase 1\) by liewegas · Pull Request \#18276 · ceph/ceph](https://github.com/ceph/ceph/pull/18276/commits),  提到曾经设计在这里提到过 [Ceph Etherpad 时间轴](https://pad.ceph.com/p/removing_removed_snaps/timeslider#4697) (注意, 只有4697版本可以看, 之后2020年被人用了机翻.)




在19年, 15版本分支中做了73条commit修改, [osd,mon: remove pg_pool_t::removed_snaps by liewegas · Pull Request #28330 · ceph/ceph (github.com)](https://github.com/ceph/ceph/pull/28330)

看`Phase 2和3: remove SnapSet::snaps`, 好像在17版本已经准备去除了?

## 目标

> - snaps are per-pool, so we should annouce deletion via OSDMap
> 
> - successful purge is the intersection of all pool PGs purged_snaps
> 
> - once a pool has purged, we can remove it from removed_snaps AND purged_snaps.
> 	- this should also be announced via the OSDMap
> 
> - mon and osd need the full removed set, but it can be global (and mostly read-only) once each pool has purged?

1.  基于资源池的snap id信息, 通过osdmap公告
2. 成功的清理, 是所有池的pool pg上的purged_snaps的交集
3. 一旦池清理完毕, 可以从removed_snaps和purged_snaps去除该snap id.
	1. 也应该通过osdmap公告
4.  mon和osd需要完整的removed快照集合, 一旦池清理完毕,就可以是全局完整和只读的


主要差异总结:

-- | planD | planC | planB | planA
-- |  -- | -- | -- | --
osdmap相关 | 仅维护当前正在删除和删除完正在purged的snap | pg_pool_t增加recent_removed_snaps_lb_epoch, 维护该epoch后删除的snap列表 | pg_pool_t增加代表最早的deletion操作的removed_snaps_lb_epoch维护, new_purged_snaps维护在该epoch之后的 | 只维护deleted_snaps 
请求 | - | 请求参数中增加removed_snaps和purged_snaps的snapc | 请求参数中增加removed_snaps和purged_snaps的snapc | 维护一个序号, 仅当存在比这个序号低的snap信息, 才去访问old_purged_snaps, 否则忽略
pginfo | pginfo中维护removed_snaps和purged_snaps | -  | - |
mon | 维护删除操作的epoch | 维护删除操作的epoch, 并负责根据定义的窗口大小更新osdmap的recent_removed_snaps | 维护删除操作的epoch, 定义一个时间周期参数, 满足该周期, 聚拢最早的删除的快照, 从而更新removed_snaps_lb_epoch | 每个周期(如100个osdmap)更新purged_snaps的快照interval_set 





# 解读phase-1


``` c++
本次pr之前已增加了13版本的逻辑, 保留了12版本功能
6e1b7c4 osd/PG: use new mimic osdmap structures for removed, pruned snaps

gh pr view 18276 --json commits| jq '.commits[]|.oid , .messageHeadline'
使用vscode正则替换`^([0-9a-f]+)\s*\n`为`$1 `




c536d4c294ec1f1c8cac6ca44675d25cb361e4f2 osd/osd_types: note about removed_snaps hack
# add in the new seq, just to try to keep the interval_set contiguous
c8bfe3fa53323c05f061ae01dba93574f4cf0747 osd/PG: share_pg_info shares past_itnervals, not PastIntervals()
# 去除过度防御设计的PastInervals, 仅当peered之后才需要新建
81d63f2994db1d6cf9b48dc8e4f6b473e83ca520 osd/OSDMap: improve osdmap flag dumping in json
# [重要]在osdmap里增加了flag_set的打印, 没看出`OSDMap::get_flag_set`这个函数起了什么作用, 还是对`flags`变量的操作.
df7523b882c798f17ecda6d2c605feaccc8b2040 qa/suites/rados/singleton/all/thrash-eio: more whitelist
# qa日志白名单增加OBJECT_
ea308ad54eb46782ec6197143bd5a5058026f4f0 include/interval_set: add get_end() to iterator
# [重要]增加通过begin+length获取结束的snapid的函数get_end
3119cf5ceea6253d1623a09e8716c29901b04d70 include/mempool: add flat_set alias   
# [重要]未理解, 不过后面就被重构掉了, 不重要了
1b1eec29ae17677a3388404afa274515d9aa812d include/types: flat_set operator<<            
# [重要]TODO: 同上, 怀疑是某个下面要用的数据结构要用的?
b9c5a243958997ab793636a80480ef04eb58d0ab osd/osd_types: SnapSet: remove get_first_snap_after()        
# [重要]删除指定快照后的一个快照的函数
e89649dca590266de4e31ac50627052fabe9e658 mds/SnapServer: fix reset()
# [重要]将mds的osdmap SnapServer::reset_state这里pi->removed_snaps.range_end()更正为snap_seq
1f133a2350afedff9e10e725ff23684aaefb43b9 mon/OSDMonitor: reset OSDMap state before decode     
# [重要]OSDMonitor::update_from_paxos里在decode前增加重置, 应该是某个缺陷问题? 
37c4affa25bc5462d83cdf21097ef1576150c429 mon/OSDMonitor: clear pending_metadata* in create_pending
# [重要]和commit一样 , PaxosService::_active触发create_pending()
553048fbf97af999783deb7e992c8ecfa5e55500 osd/OSDMap: track newly removed and purged snaps in each epoch  
# [关键]  增加了new_removed_snaps和new_purged_snaps, OSDMap::apply_incremental主要是new_removed_snaps会加入到removed_snaps_queue, new_purged_snaps里的会从removed_snaps_queue里去除
还存在一个缺失点, 那就是这俩new_purged_snaps和new_removed_snaps的更新.
9d606c587e1f9d8dd2b9e5554bf56d97710c4be7 mon/OSDMonitor: record removed_snaps by epoch outside of the osdmap
# [关键] 
typedef interval_set<
    snapid_t,
    mempool::osdmap::flat_map<snapid_t,snapid_t>> snap_interval_set_t;
这个结构是什么?

OSDMonitor::encode_pending增加写入到osd的db里的make_snap_epoch_key
prepare_remove_snaps里添加了待删除的snap到new_removed_snaps
      if (!pi.removed_snaps.contains(*q) &&
	  (!pending_inc.new_pools.count(p->first) ||
	   !pending_inc.new_pools[p->first].removed_snaps.contains(*q))) {
TODO:但是为什么是通过上面这个判定才添加是个问题, 没理解, 和new_pools有什么关系? new_pools在这里有什么指代含义?


49833c3bb264949b8126796997a95a95b50af411 mon/OSDMonitor: share snaps removed during a map gap
# [关键]  get_removed_snaps_range 根据传入的epoch版本, 从mon的db中读取写入的make_snap_epoch_key等
# MOSDMap中增加gap_removed_snaps好像是用在当客户端需要获取比mon中存的removed_snaps更老的版本的时候用的? 但是好像还没看到哪里使用
# 增加了MOSDMap兼容性的解析的compat_version标记
38e96ec2794d193f4a6cf6d5372d3b1c849ac4d2 mon/MgrStatMonitor: dump PGMapDigest at debug level 20
# [重要]增加了日志打印pgmap digest
32d7538a506be03b077da77095702b47f8150f34 osdc/Objecter: prune new_removed_snaps from active op snapc's
# [重要]objecter定义了_prune_snapc, 在Objecter::_scan_requests和Objecter::handle_osd_map中触发
# _prune_snapc主要是去除已经存在于new_removed_snaps中的snap id. 没看明白有啥特殊价值?
b1b8fc67388e1b28d8adabcf241bfbca51db2dfd osdc/Objecter: rename _scan_requests force_resend -> skipped_map
# [重要]TODO: 变量重命名, 暂不理解
192a8dc7862fcbd532c6a11a5afcf1b9cb0a8c51 osdc/Objecter: apply removed_snaps from gap to in-flight requests
# [重要]_scan_requests入参中增加gap_removed_snaps
# Objecter::handle_osd_map和_scan_requests如果skipped_map=true, 修剪gap_removed_snaps
a53ba7314c53e75d1e0b8a0edd29181db3c93863 osd,mon: add 'nosnaptrim' osd flag
# [重要]添加nosnaptrim实现
# SnapTrimmer状态机的can_trim增加flag判定
# PrimaryLogPG::kick_snap_trim()先检查flag
# boost::statechart::result PG::RecoveryState::Active::react(const ActMap&)重构的状态机或者mark_clean会触发kick_snap_trim()
345d3b655a62f221e06187b67d53aeb5f7239062 osd/osd_types: add purged_snaps to pg_stat_t       
# [重要]见题知义
6df912b18b4c493c9a8f2e65ecd158f119c8c210 osd/PG: share purged_snaps with mgr at mimic    
# [重要]设置osd_max_snap_prune_intervals_per_epoch默认100, 并在PG::publish_stats_to_osd()的pg_stat_t中按该参数为上限填充purged_snaps项
86f0b811882de334f0d7e577b6c47bef6aba2422 mon/PGMap: add purged_snaps map to PGMapDigest                  
# [关键]  PGMapDigest增加purged_snaps, 实现PGMap::calc_purged_snaps(), 使用所有pg的pg_stat_t里的purged_snaps合并出池的purged_snaps
# PGMap::encode_digest中调用calc_purged_snaps
# 增加mutex_lock, 实现with_mutable_pgmap
# mgr DaemonServer::send_report()使用with_mutable_pgmap
e5f62fb8ac8a1ef79aa97361e83c796b0d94fc28 osd/PG: move debug_verify_cached_snaps check into PGPool::update     
# [重要]PGPool::update(CephContext *cct, OSDMapRef map) 增加context 
# osd_debug_verify_cached_snaps调整到update函数中 , 待理解
33c9907662ad5b504581b373c4f4e3b4b5a1cc63 osd/PG: some whitespace   
# 字符格式化
f04729cedee9c2c53dd56b952b6b2167ae392da1 osd/PG: break out of Active AdvMap handler if interval change
# [重要]PG::RecoveryState::Active::react(const AdvMap& advmap) 通过PastIntervals增加快速检查重新peer 
231ec67b7a6d0ca228266cccd0ef53a77b5428e6 osd/PG: simplify replica purged_snaps update       
# [重要]改为仅由主处理purged_snaps更新, 从只负责同步
6e1b7c4c14be575a554ffe1d6e71c0d6189486af osd/PG: use new mimic osdmap structures for removed, pruned snaps
# [关键]  PGPool中, cached_removed_snaps和newly_removed_snaps只在12版本使用
# PG::activate中, 12版本snap_trimq = pool.cached_removed_snaps;
# 13版本则使用get_removed_snaps_queue中的, 并去除pginfo的purged_snaps已包含的. 并将purged_snaps中且在removed_snaps中的替换掉原来的pginfo的purged_snaps. 这里的purged_snaps只放有可能还在removed_snaps队列中的.
# PG::RecoveryState::Active::react(const AdvMap& advmap)状态机增加13版本处理逻辑
# 遍历new_removed_snaps,  pg->snap_trimq增加不相交的部分的snap_interval_set_t, 如果存在重复的部分, 设置bad为true. 如果出现重复就assert崩溃, 说明不可能重复. 设置pg->dirty_info = true; 和pg->dirty_big_info = true;
# 遍历new_purged_snaps, 如果pg_info的purged_snaps中包含了该部分, 则去除
# 如果存在交集, 和上文同样一起崩溃
# if (pg->dirty_big_info) {
      // share updated purged_snaps to mgr/mon so that we (a) stop reporting
      // purged snaps and (b) perhaps share more snaps that we have purged
      // but didn't fit in pg_stat_t.
      pg->publish_stats_to_osd();
      pg->share_pg_info();
    }
16c5bcc0218069edcd5e16b8d17912d47b8e70b3 osd/osd_types: pg_pool_t: add FLAG_{SELFMANAGED,POOL}_SNAPS flags
# [重要]add selfmanaged_snaps ,而不是通过removed_snaps的空与否区分
fd6a59ebf45a397e9530b9350ee46db99e70e5f8 mon/OSDMonitor: convert removed_snaps on first mimic map
# [关键] PG增加last_require_osd_release, 针对12升13的第一次PG进入active状态机的响应, 跳过上面的检查崩溃逻辑. 进入下一次处理
# OSDMonitor::encode_pending( 增加 根据removed_snaps空设置pi.flags
# 并设置new_removed_snaps以及mon中的key
9607a2db466835406d6fb73ad6776eccdf3a7d6c mon/OSDMonitor: prune purged snaps
# [重要]osd实现`try_prune_purged_snaps`, 并在bluestore的key中存储`purged_snap_{pool_id}_{snap_id的十六进制}`的信息.
f2d602acb8d8142119a89f17ccc28e7ee6a34be9 mon/OSDMonitor: propagate new_removed_snaps to other tiers
# [重要] OSDMap::Incremental::propagate_snaps_to_tiers中将base的new_removed_snaps信息同步给tier池, 怎么看着是553048fbf97af999783deb7e992c8ecfa5e55500 osd/OSDMap: track newly removed and purged snaps in each epoch  这条后边的提交?
8c44dabe4b26bb2df77145f1b2227c87087f315e osd/PG: ignore purged_snaps inconsistencies for now
# [重要]注释了bad = true; 说明去除了上面的崩溃逻辑


```


TODO: 这里的15版本之前的兼容性是为了什么服务的呢?

[osd/PG: use new mimic osdmap structures for removed, pruned snaps · ceph/ceph@6e1b7c4](https://github.com/ceph/ceph/commit/6e1b7c4c14be575a554ffe1d6e71c0d6189486af)

在开始pr前, 通过该修改调整了osdmap等的格式?


``` mermaid
---
title: luminous快照删除实现
---
sequenceDiagram

client ->> mon: send remove snap
mon ->> osd: send `OSDMap::newly_removed_snaps`
osd ->> tp_peering: `snaptrimmer_state_machine snap_trimq`
tp_peering ->> tp_peering: `pg_info_t::purged_snaps`
tp_peering ->> osd: `OSDMap::cached_removed_snaps`
osd ->> mon: `OSDMap::removed_snaps`
```



``` mermaid
---
title: mimic设计
---



sequenceDiagram
client ->> mon: remove snap
mon ->> osd: `OSDMap::new_removed_snaps`
osd ->> pg: `OSDMap::new_removed_snaps->snap_trimq`
%% 这里是每有一个成功删除就产生吗? 还是以什么为边界发送新的? 比如pg完成一次pg的?
pg ->> osd: `pg_info_t::purged_snaps`
osd ->> mon: `Mondb::gap_removed_snaps`

```

## 梳理FAQ

### 大方向梳理


#### 升级更新

* 
	* OSDMontior.cc: 处理OSDMap::Increment
		* new_removed_snaps
			* 处理mon数据库中的snap信息 OSD_SNAP_PREFIX
* osdmap更新
	* removed_snaps 归档
	
	* removed_snap_queue中
	* 新增的newly_removed_snaps和newly_purged_snaps的维护
* `PGPool::update(CephContext *cct, OSDMapRef map)`
	* new_removed_snaps
* PG::activate(
	* 13版本 snap_trimq 用removed_snaps_queue, 12版本继续用cached_removed_snaps
	* 13版本的话, 并将pg_info_t中的purged_snaps更新成当前仅需要放的部分, 不再维持那么多, 12版本维持不变.
* PG::publish_stats_to_osd()
	* 13版本后从pg_info_t中将purged_snaps更新到pg_stat_t中, 12版本的话, 该osd依旧不提供该信息
* PG::RecoveryState::Active::react(const AdvMap& advmap)
	* 13版本则用新逻辑, 且如果该pg的上个版本last_require_osd_release是12, 则snap_trimq减去pg_info_t中的purged_snaps. TODO:这块持疑问, 是因为snap_trimq来源的地方复用?
		* 12版本继续用newly_removed_snaps
* PGPool 初始化
	* 12版本, 需要构造cached_removed_snaps

* pg_info_t中的removed_snaps清理
* pg_stat_t中的removed_snaps维护

 * 涉及pg激活流程? 以及快照删除状态机的流程

##### 通信流程串起来

* pg_pool_t::removed_snaps  用在用户定义场景
	* unmanaged_snap
		* OSDMonitor::prepare_remove_snaps
		* OSDMonitor::preprocess_remove_snaps
		* pg_pool_t::add_unmanaged_snap
		* pg_pool_t::remove_unmanaged_snap
		* pg_pool_t::encode
		* pg_pool_t::decode
			* const mempool::osdmap::map<int64_t,pg_pool_t>& pools 
			* 响应osdmap, 创建/删除快照, 确实会带来osdmap变更
		* OSDMap::Incremental::propagate_snaps_to_tiers
	* pool属性的全局snaps
		* pg_pool_t::build_removed_snaps
			* snap existence/non-existence defined by snaps[] and snap_seq
			* 当未生成removed_snaps时, 查询pg_pool_t->snaps, 遍历当前存在哪些快照, 生成该项
		* pg_pool_t::maybe_updated_removed_snaps
			* PGPool::update
			* pg_pool_t::add_snap
			* pg_pool_t::remove_snap

* PGPool::cached_removed_snaps // current removed_snaps set  应该是正在干活的
	* PGPool::update
		* PG::handle_advance_map(
		* 检测到当前osd内存里的cached_removed_snaps没有osdmap里的removed_snaps新了,  则更新cached_removed_snaps为新的removed_snaps, newly_removed_snaps为subtract后的差.
			* 如果不是子集, 则直接更换cached_removed_snaps.
	* PG::activate
		* 作为主pg时, 设置snap_trimq为cached_removed_snaps, 并去除pg_info_t中的purged_snaps
		* 需要更新pglog时, 发送MOSDPGLog, 包括本pg的purged_snaps
	* PG::handle_advance_map
		* 开启调试选项osd_debug_verify_cached_snaps时, 从build_removed_snaps生成一次removed_snaps进行比较, 校验通过才能继续

* PGPool::newly_removed_snaps  // newly removed in the last epoch  最新删除的
	* PG::RecoveryState::Active::react(const AdvMap& advmap)
		* pg->snap_trimq.union_of(pg->pool.newly_removed_snaps);
		* 收到重构事件时, 检测到newly_removed_snap非空, 则合并到snap_trimq中. 从而用于干活
* pg_info_t::purged_snaps
	* pg_info_t::encode
	* pg_info_t::decode
	* `bool operator==(const pg_info_t& l, const pg_info_t& r)`
	* PG::activate
	* `PG::split_into(pg_t child_pgid, PG *child, unsigned split_bits)`
	* `PG::_prepare_write_info`
		* 根据环境里看到dirty_big_info 应该为false
		* 在提供pg_info时, 临时将purged_snaps置空, encode完再还原 map<string,bufferlist> km;
	* PG::read_info
	* PG::filter_snapc
	* PG::proc_primary_info
		* PG::RecoveryState::ReplicaActive::react(const MInfoRec& infoevt) 重构恢复时, 更新pg_info_t的purged_snaps
	* PGLog::merge_log(
	* PrimaryLogPG::AwaitAsyncWork::react(const DoSnapWork&)
		* pg_info_t中的purged_snaps 添加get_next_objects_to_trim的 snap_to_trim
		* pg的snap_trimq清理掉snap_to_trim
* PGTransaction::ObjectOperation::updated_snaps 
* PG::snap_mapper
	* PG::clear_object_snap_mapping
	* PG::update_object_snap_mapping
	* PG::update_object_snap_mapping
	* PG::update_snap_map
	* `PG::_scan_snaps(ScrubMap &smap)`
		* scrub的时候触发, 似乎只是校验快照元数据, 本身不起啥作用.
		* TODO:scrub校验侧逻辑
	* PrimaryLogPG::on_local_recover(
	* PrimaryLogPG::AwaitAsyncWork::react(const DoSnapWork&)
* snapset
	* legacy看着是12版本以下, 更早的时候的


* 首先所有进程重启, 此时, 进程空间知道自身feature为13了
	* 检测osdmap的继续走12版本逻辑
	* 检测HAVE_FEATURE的, 则都会进入新逻辑? 这里有问题? 
* 此时将require_osd_release更新为13
	* 此时osdmap的也都触发新逻辑, 但是osd通信总有先后, 意味着存在13版本和12版本通信的情况. 此时怎么保障部崩溃?
	* osd和mgr通信也是一点. 
		* pg_stat_t
	* pg状态机
	* mon与osd通信的, 新的gap_removed_snaps的兼容性. 


##### 测试场景
着重观察
升级后
* mgr与osd通信的pg_stat_t新增对象的解析
* mon与osd的历史gap_removed_snaps的处理
	* 查询与新增写入
* pg_info_t转到pg_stat_t
* osdmap/pg_stat_t/pg_info_t/MOSDMap的编解码


稳定性
1. mgr升级后
	1. 与低版本通信. 
	2. 全服务重启
	3. 正常业务测试
2. mgr/mon升级后,
	1. 与低版本通信
	2. 全服务重启
	3. 正常业务性能/功能接口测试
3. mgr/mon/osd局部升级后
	1. 与低版本通信
	2. 全服务重启
	3. 正常业务性能/功能接口测试
4. mgr/mon/osd全部升级后
	1. 与低版本通信
	2. 全服务重启
	3. 正常业务性能/功能接口测试
5. qa/unittest中, 是否存在相关测试集?
	1. test_snap_mapper.cc
		1. snaps
	2. testPGLog.cc
		1. pg_info_t中的purged_snaps

#### mon更新数据库持久化的效果? 以及性能影响


#### 12版本的设计实现中, 是如何持久化的? 且都在哪些地方使用到?

* pg_pool_t::removed_snaps  用在用户定义场景
	* unmanaged_snap
		* OSDMonitor::prepare_remove_snaps
		* OSDMonitor::preprocess_remove_snaps
		* pg_pool_t::add_unmanaged_snap
		* pg_pool_t::remove_unmanaged_snap
		* pg_pool_t::encode
		* pg_pool_t::decode
		* OSDMap::Incremental::propagate_snaps_to_tiers
	* pool属性的全局snaps
		* pg_pool_t::build_removed_snaps
			* snap existence/non-existence defined by snaps[] and snap_seq
			* 当未生成removed_snaps时, 查询pg_pool_t->snaps, 遍历当前存在哪些快照, 生成该项
		* pg_pool_t::maybe_updated_removed_snaps
			* PGPool::update
			* pg_pool_t::add_snap
			* pg_pool_t::remove_snap


* PGPool::cached_removed_snaps // current removed_snaps set  应该是正在干活的
	* PGPool::update
		* PG::handle_advance_map(
		* 检测到当前osd内存里的cached_removed_snaps没有osdmap里的removed_snaps新了,  则更新cached_removed_snaps为新的removed_snaps, newly_removed_snaps为subtract后的差.
			* 如果不是子集, 则直接更换cached_removed_snaps.
	* PG::activate
		* 作为主pg时, 设置snap_trimq为cached_removed_snaps, 并去除pg_info_t中的purged_snaps
		* 需要更新pglog时, 发送MOSDPGLog, 包括本pg的purged_snaps
	* PG::handle_advance_map
		* 开启调试选项osd_debug_verify_cached_snaps时, 从build_removed_snaps生成一次removed_snaps进行比较, 校验通过才能继续

* PGPool::newly_removed_snaps  // newly removed in the last epoch  最新删除的
	* PG::RecoveryState::Active::react(const AdvMap& advmap)
		* pg->snap_trimq.union_of(pg->pool.newly_removed_snaps);
		* 收到重构事件时, 检测到newly_removed_snap非空, 则合并到snap_trimq中. 从而用于干活


* pg_info_t::purged_snaps
	* pg_info_t::encode
	* pg_info_t::decode
	* `bool operator==(const pg_info_t& l, const pg_info_t& r)`
	* PG::activate
	* `PG::split_into(pg_t child_pgid, PG *child, unsigned split_bits)`
	* `PG::_prepare_write_info`
		* 根据环境里看到dirty_big_info 应该为false
		* 在提供pg_info时, 临时将purged_snaps置空, encode完再还原 map<string,bufferlist> km;
	* PG::read_info
	* PG::filter_snapc
	* PG::proc_primary_info
		* PG::RecoveryState::ReplicaActive::react(const MInfoRec& infoevt) 重构恢复时, 更新pg_info_t的purged_snaps
	* PGLog::merge_log(
	* PrimaryLogPG::AwaitAsyncWork::react(const DoSnapWork&)
		* pg_info_t中的purged_snaps 添加get_next_objects_to_trim的 snap_to_trim
		* pg的snap_trimq清理掉snap_to_trim
* PGTransaction::ObjectOperation::updated_snaps 
* PG::snap_mapper
	* PG::clear_object_snap_mapping
	* PG::update_object_snap_mapping
	* PG::update_object_snap_mapping
	* PG::update_snap_map
	* `PG::_scan_snaps(ScrubMap &smap)`
		* scrub的时候触发, 似乎只是校验快照元数据, 本身不起啥作用.
		* TODO:scrub校验侧逻辑
	* PrimaryLogPG::on_local_recover(
	* PrimaryLogPG::AwaitAsyncWork::react(const DoSnapWork&)



* snapset
	* legacy看着是12版本以下, 更早的时候的

##### SnapTrimmer状态机 


``` mermaid
---
title: SnapTrimmer
---
stateDiagram-v2

state if_state <<choice>>
state if_state2 <<choice>>
state if_state3 <<choice>>
state if_state4 <<choice>>



	NotTrimming --> if_state2: KickTrim
	if_state2 --> Trimming: pg active+clean+primary+no scrub
	if_state2 --> WaitScrub: pg scrub
	Trimming --> NotTrimming: Reset
	Trimming --> Trimming: KickTrim
	WaitScrub --> NotTrimming: Reset
	WaitScrub --> WaitScrub: KickTrim
	WaitScrub --> NotTrimming: ScrubComplete



state Trimming {
[*] --> WaitReservation
	WaitTrimTimer --> if_state : SnapTrimTimerReady
	if_state --> AwaitAsyncWork: can_trim
	if_state --> NotTrimming: not can_trim
	WaitRWLock --> if_state3: TrimWriteUnblocked
	if_state3 --> AwaitAsyncWork: can_trim
	if_state3 --> NotTrimming: not can_trim
	WaitRepops --> if_state4: RepopsComplete
	if_state4 --> WaitTrimTimer: can_trim
	if_state4 --> NotTrimming: not can_trim
	AwaitAsyncWork --> NotTrimming: DoSnapWork and 干活
	AwaitAsyncWork --> WaitRepops
	AwaitAsyncWork --> WaitRWLock
	WaitReservation --> AwaitAsyncWork: SnapTrimReserved
	WaitReservation --> NotTrimming: not can_trim
	}

```


##### snap_mapper

``` c
/**
 * SnapMapper
 *
 * Manages two mappings:
 *  1) hobject_t -> {snapid}
 *  2) snapid -> {hobject_t}
 *
 * We accomplish this using two sets of keys:
 *  1) OBJECT_PREFIX + obj.str() -> encoding of object_snaps
 *  2) MAPPING_PREFIX + snapid_t + obj.str() -> encoding of pair<snapid_t, obj>
 *
 * The on disk strings and encodings are implemented in to_raw, to_raw_key,
 * from_raw, to_object_key.
 *
 * The object -> {snapid} mapping is primarily included so that the
 * SnapMapper state can be verified against the external PG state during
 * scrub etc.
 *
 * The 2) mapping is arranged such that all objects in a particular
 * snap will sort together, and so that all objects in a pg for a
 * particular snap will group under up to 8 prefixes.
 */
```


##### clones clone_snaps

##### snapSet
> If the head is deleted while there are still clones, a snapdir object is created instead to house the SnapSet.


##### rbd 创建和删除快照 时间长 原因?

好像是mon响应慢了?

``` log

2023-04-24 11:34:36.966012 7faa90ff9700 10 librbd::ImageWatcher: 0x7faa7801b160 current lock owner: [9975878,140370134551008]
2023-04-24 11:34:36.966015 7faa90ff9700 10 librbd::Watcher::C_NotifyAck 0x7faa78022960 finish: r=0
2023-04-24 11:34:36.966237 7faacffff700 10 librados: finish completed notify (linger op 0x7faa800038a0), r = 0
2023-04-24 11:34:36.966273 7faa90ff9700 20 librbd::watcher::Notifier: 0x7faa7801b1e0 handle_notify: r=0
2023-04-24 11:34:36.966281 7faa90ff9700 20 librbd::watcher::Notifier: 0x7faa7801b1e0 handle_notify: pending=0
2023-04-24 11:34:38.125463 7faa93fff700 10 monclient: _renew_subs
2023-04-24 11:34:38.125478 7faa93fff700 10 monclient: _send_mon_message to mon.worker3 at 192.168.70.4:6789/0
2023-04-24 11:34:38.171149 7faa90ff9700  5 librbd::SnapshotCreateRequest: 0x7faa80002ad0 handle_allocate_snap_id: r=0, snap_id=41321



2023-04-24 11:48:28.274546 7f5f7325d700 10 mon.master1@0(leader).osd e35237 preprocess_query pool_op(create unmanaged snap pool 3 auid 0 tid 21 name  v0) v4 from client.10348311 192.168.70.1:0/3015903241                                                                                                                                                                                                 2023-04-24 11:48:28.274557 7f5f7325d700 20 is_capable service=osd command=osd pool op unmanaged-snap write on cap allow *                                                                             2023-04-24 11:48:28.274561 7f5f7325d700 20  allow so far , doing grant allow *                                                                                                                        2023-04-24 11:48:28.274562 7f5f7325d700 20  allow all                                                                                                                                                 2023-04-24 11:48:28.274565 7f5f7325d700  7 mon.master1@0(leader).osd e35237 prepare_update pool_op(create unmanaged snap pool 3 auid 0 tid 21 name  v0) v4 from client.10348311 192.168.70.1:0/3015903241                                                                                                                                                                                                   2023-04-24 11:48:28.274570 7f5f7325d700 10 mon.master1@0(leader).osd e35237 prepare_pool_op pool_op(create unmanaged snap pool 3 auid 0 tid 21 name  v0) v4     

```


##### snapdir?

CEPH_SNAPDIR 
说是通过clone保留了原始快照造出来的. 


##### pg状态机和PGPool::update的关联



##### snaptrim逻辑

``` bash
rbd::Shell::execute() --> rbd::action::snap::execute_remove() --> rbd::action::snap::do_remove_snap() --> librbd::Image::snap_remove2() --> librbd::snap_remove() --> librbd::Operations<librbd::ImageCtx>::snap_remove() --> Operations<I>::snap_remove() --> librbd::Operations<librbd::ImageCtx>::execute_snap_remove() --> librbd::operation::SnapshotRemoveRequest::send() --> cls_client::snapshot_remove() --> ... --> 发送op给rbd_header对象所在的Primary OSD

// OSD删除快照信息
cls_rbd::snapshot_remove() --> cls_cxx_map_remove_key() --> ReplicatedPG::do_osd_ops(CEPH_OSD_OP_OMAPRMKEYS)

// RBD客户端向Monitor发送删除快照的消息
librbd::operation::SnapshotRemoveRequest::send() --> SnapshotRemoveRequest<I>::send_release_snap_id() --> Objecter::delete_selfmanaged_snap() --> 
Objecter::pool_op_submit() --> Objecter::_pool_op_submit() --> MonClient::send_mon_message()

// Monitor删除快照信息
OSDMonitor::prepare_pool_op() --> pg_pool_t::remove_unmanaged_snap() --> pg_pool_t::removed_snaps




rbd::Shell::execute() --> rbd::action::snap::execute_remove() --> rbd::action::snap::do_remove_snap() --> librbd::Image::snap_remove2() --> librbd::snap_remove() --> librbd::Operations<librbd::ImageCtx>::snap_remove() --> Operations<I>::snap_remove() --> librbd::Operations<librbd::ImageCtx>::execute_snap_remove() --> librbd::operation::SnapshotRemoveRequest::send() --> cls_client::snapshot_remove() --> ... --> 发送op给rbd_header对象所在的Primary OSD

// OSD删除快照信息
cls_rbd::snapshot_remove() --> cls_cxx_map_remove_key() --> ReplicatedPG::do_osd_ops(CEPH_OSD_OP_OMAPRMKEYS)

// RBD客户端向Monitor发送删除快照的消息
librbd::operation::SnapshotRemoveRequest::send() --> SnapshotRemoveRequest<I>::send_release_snap_id() --> Objecter::delete_selfmanaged_snap() --> 
Objecter::pool_op_submit() --> Objecter::_pool_op_submit() --> MonClient::send_mon_message()

// Monitor删除快照信息
OSDMonitor::prepare_pool_op() --> pg_pool_t::remove_unmanaged_snap() --> pg_pool_t::removed_snaps
```

[snaptrim中的qos | 李厅](https://lovethegirl.github.io/2020/02/16/snaptrim_qos/)


##### 关键的trim_object和get_next_object_to_trim

这里应该是通过事务生成pglog里用的update_snaps设置snaps,然后正常pg逻辑里从pglog merge的? merge是直接merge的purged_snaps, 所以这里update的snaps是怎么更新成purged_snaps呢?
 应该不是

``` c++
  } else if (r == -ENOENT) {
    // Done!
    ldout(pg->cct, 10) << "got ENOENT" << dendl;

    ldout(pg->cct, 10) << "adding snap " << snap_to_trim
		       << " to purged_snaps"
		       << dendl;
    pg->info.purged_snaps.insert(snap_to_trim);
    pg->snap_trimq.erase(snap_to_trim); 
```

##### SnapTrimmer进入初始状态的条件?

``` c++
snap_trimmer_machine

PGQueueable
PGSnapTrim

在AwaitAsyncWork 实例化的时候, 会queue_for_snap_trim

boost::statechart::result PG::RecoveryState::Active::react(const ActMap&) 

在RecoveryState

应该是重构状态机理触发的

重构状态机进入条件就是收到新的osdmap之类的

是在tp_peering线程, peering_wq

但是那个好像是常规状态机, 一旦进入干活, 还是tp_osd_tp干活

PGQueueable , 是否跟这个有关?
```


#### 13版本的设计中, 初步理解是在mon的数据库中持久化, 那osd需要用到时, 是会产生一条新的和mon通信的协议吗?

* pg_pool_t::removed_snaps
	* OSDMonitor::encode_pending(MonitorDBStore::TransactionRef t) 
		* 第一次升级的时候, 将removed_snaps转换为new_removed_snaps , 并写入到mon数据库中
	* OSDMonitor::preprocess_remove_snaps(MonOpRequestRef op)
	* OSDMonitor::prepare_remove_snaps(MonOpRequestRef op)
	* "tier add  --force-nonempty"
	* pg_pool_t::is_removed_snap(snapid_t s) const
	* pg_pool_t::build_removed_snaps(interval_set<snapid_t>& rs) const
	* pg_pool_t::maybe_updated_removed_snaps(const interval_set<snapid_t>& cached) const
	* pg_pool_t::add_unmanaged_snap(uint64_t& snapid)
	* pg_pool_t::remove_unmanaged_snap(snapid_t s)
	* pg_pool_t::encode(bufferlist& bl, uint64_t features) const
	* pg_pool_t::decode(bufferlist::iterator& bl)
	* OSDMap::Incremental::propagate_snaps_to_tiers(CephContext *cct,
	* `PGPool::update(CephContext *cct, OSDMapRef map)`
		* 检测osdmap版本, 12版本旧逻辑, 13版本则清理掉12版本的newly_removed_snaps和cached_removed_snaps
	* PG::activate
		* osdmap 12版本, snap_trimq继续使用cached_removed_snaps
		* 13版本, snap_trimq使用removed_snaps_queue
* pg_pool_t::new_removed_snaps
	* OSDMonitor::encode_pending(MonitorDBStore::TransactionRef t) 
	* OSDMonitor::prepare_remove_snaps(MonOpRequestRef op)
	* OSDMonitor::prepare_pool_op(MonOpRequestRef op)
	* OSDMap::Incremental::propagate_snaps_to_tiers(CephContext *cct
	* OSDMap::Incremental::encode(bufferlist& bl, uint64_t features) const
	* OSDMap::Incremental::decode(bufferlist::iterator& bl)
	* OSDMap::apply_incremental(const Incremental &inc)
	* OSDMap::encode(bufferlist& bl, uint64_t features) const
	* OSDMap::decode(bufferlist::iterator& bl)
	* const mempool::osdmap::map<int64_t,snap_interval_set_t>&
	* PG::RecoveryState::Active::react(const AdvMap& advmap)
	* `Objecter::_prune_snapc(`
	* get_new_removed_snaps()
* pg_pool_t::new_purged_snaps
	* OSDMonitor::encode_pending(MonitorDBStore::TransactionRef t)
	* OSDMonitor::try_prune_purged_snaps()
	* OSDMap::Incremental::encode(bufferlist& bl, uint64_t features) const
	* OSDMap::Incremental::decode(bufferlist::iterator& bl)
	* OSDMap::apply_incremental(const Incremental &inc)
	* OSDMap::encode(bufferlist& bl, uint64_t features) const
	* OSDMap::decode(bufferlist::iterator& bl)
	* get_new_purged_snaps()
	* PG::RecoveryState::Active::react(const AdvMap& advmap)
* removed_snaps_queue
	* OSDMonitor::encode_pending(MonitorDBStore::TransactionRef t)
	* OSDMap::apply_incremental(const Incremental &inc)
	* OSDMap::encode(bufferlist& bl, uint64_t features) const
	* OSDMap::decode(bufferlist::iterator& bl)
	* get_removed_snaps_queue()
	* PG::activate
		* osdmap版本大于13, 兼容
* PGMapDigest::purged_snaps
	* OSDMonitor::try_prune_purged_snaps()
	* PGMapDigest::encode(bufferlist& bl, uint64_t features) const
	* PGMapDigest::decode(bufferlist::iterator& p)
	* PGMap::calc_purged_snaps()
* pg_stat_t::purged_snaps
	* pg_stat_t::encode(bufferlist &bl) const
	* pg_stat_t::decode(bufferlist::iterator &bl)
	* `operator==(const pg_stat_t& l, const pg_stat_t& r)`
* pg_info_t::purged_snaps
	* pg_info_t::encode(bufferlist &bl) const
	* pg_info_t::decode(bufferlist::iterator &bl)
	* `operator==(const pg_info_t& l, const pg_info_t& r)`
	* PG::activate
	* PG::split_into
	* PG::publish_stats_to_osd()
	* `PG::_prepare_write_info`
	* PG::filter_snapc(vector<snapid_t> &snaps)
	* PG::proc_primary_info(ObjectStore::Transaction &t, const pg_info_t &oinfo)
	* PG::RecoveryState::Active::react(const AdvMap& advmap)
	* PGLog::merge_log
	* PrimaryLogPG::AwaitAsyncWork::react(const DoSnapWork&)
* MOSDMap::mempool::osdmap::map<int64_t,OSDMap::snap_interval_set_t> gap_removed_snaps; 
	* encode_payload 
	* decode_payload
	* OSDMonitor::send_incremental(
		* 	* 在osd启动阶段, 会需要从mon拿到指定版本之间的MOSDMap? 
	* OSDMonitor::get_removed_snaps_range
		* 取决于first_committed和当前osdmap版本, 是不是比如trim或者compact的时候, 这个first_commited才会更改? 否则mon内存里其实一直存着所有的removed_snaps? 确实
	* `Objecter::_scan_requests(`
	* Objecter::handle_osd_map
	* 只有client和mgr的objecter客户端层, 这边一开始查下这个. osd干活的时候就不再查了. 
		* 客户端查到之后用作什么? 好像主要是剪枝, 确保客户端的请求的snapset不再包含该快照版本? 减少误判? 
			* 比如客户端拿到的id是旧的场景吗?
	* 什么时候通知mon更新这个? 
		* 由osdmap的版本更新, 收到osdmap中的new_removed_snaps和removed_snaps的时候


#### mds中也有SnapServer, 也涉及该项, 需要了解该部分中的使用逻辑

###  removed_snaps的入口添加在哪里? 是怎么加入到increment的事务里的?

OSDMonitor::prepare_remove_snaps

MRemoveSnaps

void pg_pool_t::remove_snap(snapid_t s)
{
  assert(snaps.count(s));
  snaps.erase(s);
  snap_seq = snap_seq + 1;
}


  case POOL_OP_DELETE_SNAP:
    {
      snapid_t s = pp.snap_exists(m->name.c_str());
      if (s) {
	pp.remove_snap(s);
	pending_inc.new_removed_snaps[m->pool].insert(s);
	changed = true;
      }
    }
    break;

Objecter::delete_pool_snap

### removed_snaps 残留代表啥?


### new_purged_snaps 实际执行?

PrimaryLogPG::kick_snap_trim()

snap_trimq应该是干活的, 这块状态机基本上没咋改应该.

### moncap里提到了快照的注释? 






###  mds如何使用的快照? 


### 合入过程

忽略, 看错了.
* osdmap的decode, 需要feature识别...但是有差别, 在快照前应该还有个v功能. 
初步理解, 还是需要引入mimic这个版本的feature标记. 才能区分是老osd? 能不能不引入mimic, 直接引入feature识别? 

SERVER_M 这个flag已经在了

12这个patch需要转换下. 把SERVER_MIMIC换成 SERVER_M


20这个patch需要查下pg_stat_t, 12版本是22, patch是23->24   涉及PR, 2条commit  https://hub.nuaa.cf/ceph/ceph/pull/18058/commits

a25221e

结果这个修改, 立马就被重构掉了 [osd_types.cc: don't store 32 least siognificant bits of state twice · ceph/ceph@0230fe6 · GitHub](https://hub.nuaa.cf/ceph/ceph/commit/0230fe673396cc84b595ff2215546880bd77eb5b)

也是2条commit的PR [osd_types.cc: reorder fields in serialized pg_stat_t by branch-predictor · Pull Request #19965 · ceph/ceph · GitHub](https://hub.nuaa.cf/ceph/ceph/pull/19965)

ae7472f
0230fe6

目前初步看到的障碍是 , 12版本 snaptrimq_len还在invalid后面, 13-14也做了一此reorder, 其实也兼容

13版本是033d246
12版本是ca4413d

所以为了满足12版本的兼容性? 

这里那12升13怎么保证兼容性?  

根据[ceph/osd_types.cc at nautilus · ceph/ceph · GitHub](https://github.com/ceph/ceph/blob/nautilus/src/osd/osd_types.cc) 可以知道14和12是保证了兼容性的.

pg_stat_t::decode(bufferlist::iterator &bl)


需要先合入osdmap增加的

27d6f43 


中间0020左右, 手动把新的recovery_unfound给合入了, 暂时不知道为啥会合入这段, 手动合入了.osd_types.h的#define那段

然后0028, 在pg_pool_t::encode增加了v改到27
0030, PGMapDigest& get_digest 这个的修改依赖高版本将mon里的部分函数重构到mgr中的修改, 所以尝试性修改是维持不变. 根据12版本现状做处理


编译通过, 实际运行
2023-04-12 14:43:11.981804 7f44f6d7d700  0 log_channel(cluster) log [DBG] : pgmap v10: 192 pgs: 165 active+clean, 18 peering, 9 unknown; 180GiB data, 415GiB used, 1.20TiB / 1.61TiB avail
2023-04-12 14:43:12.871023 7f44fdd8b700 -1 failed to decode message of type 87 v1: buffer::malformed_input: void object_stat_collection_t::decode(ceph::buffer::list::iterator&) no longer understand old encoding version 2 < 122
2023-04-12 14:43:13.866212 7f4500d91700 -1 failed to decode message of type 87 v1: buffer::malformed_input: void object_stat_collection_t::decode(ceph::buffer::list::iterator&) no longer understand old encoding version 2 < 122
2023-04-12 14:43:13.982740 7f44f6d7d700  4 OSD 0 is up, but has no stats
2023-04-12 14:43:13.982742 7f44f6d7d700  2 wc osd 3avail: 71955388378
2023-04-12 14:43:13.982744 7f44f6d7d700  2 wc osd 4avail: 86134315413
2023-04-12 14:43:13.982744 7f44f6d7d700  2 wc osd 5avail: 61272017967
2023-04-12 14:43:13.982748 7f44f6d7d700  4 OSD 0 is up, but has no stats
2023-04-12 14:43:13.982749 7f44f6d7d700  2 wc osd 1avail: 827202140267


mgr无法处理osd发来的消息, 同理, osd上也有

搜到个scrub的测试脚本? 


推荐的升级流程

The upgrade order starts with managers, monitors, then other daemons.

Each daemon is restarted only after Ceph indicates that the cluster will remain available.

compatv确实是2, 但是小于25, 那个25哪来的? 还有header.version的6 哪来的?

终于想起来了, mgr应该是没重新编译, mon也是

编译mgr没问题 ,mon需要 把PGMonitor.cc这里的int32_t改成  uint64_t, 因为pg数据结构改了  mempool::pgmap::unordered_map<uint64_t,int32_t> num_pg_by_state;

替换之后 , 崩在了

``` log
 ceph version 12.2.12 (1436006594665279fe734b4c15d7e08c13ebd777) luminous (stable)
 1: (()+0xa73541) [0x55c718be4541]
 2: (()+0xf5d0) [0x7f88944635d0]
 3: (gsignal()+0x37) [0x7f8893484207]
 4: (abort()+0x148) [0x7f88934858f8]
 5: (__gnu_cxx::__verbose_terminate_handler()+0x165) [0x7f8893d937d5]
 6: (()+0x5e746) [0x7f8893d91746]
 7: (()+0x5e773) [0x7f8893d91773]
 8: (()+0x5e993) [0x7f8893d91993]
 9: (object_stat_collection_t::decode(ceph::buffer::list::iterator&)+0x530) [0x55c7188ca990]
 10: (pg_stat_t::decode(ceph::buffer::list::iterator&)+0x1e5) [0x55c7188d0da5]
 11: (pg_info_t::decode(ceph::buffer::list::iterator&)+0x13a) [0x55c7188d149a]
 12: (PG::read_info(ObjectStore*, spg_t, coll_t const&, ceph::buffer::list&, pg_info_t&, PastIntervals&, unsigned char&)+0x231) [0x55c71871d731]
 13: (PG::read_state(ObjectStore*, ceph::buffer::list&)+0x7b) [0x55c718725efb]
 14: (OSD::load_pgs()+0x9b4) [0x55c718677334]
 15: (OSD::init()+0x2169) [0x55c718695ca9]
 16: (main()+0x2d07) [0x55c7185974a7]
 17: (__libc_start_main()+0xf5) [0x7f88934703d5]
 18: (()+0x4c65b3) [0x55c7186375b3]
```
找到问题了,pg_stat_t里 之前合并purged_snaps合并错了, 应该是环境里的osd被我之前误启动已经污染了, 暂时跳过升级的验证项

先单纯验证功能

osdmap里 removed_snaps_queue
和pg query看不到pg_info里有purged_snaps.

好像是我新增的代码都没生效? 是要让他满足HAVE_FEATURES选项, 在OSDMap.cc里获取过程uint64_t OSDMap::get_encoding_features() const

  if (require_osd_release < CEPH_RELEASE_MIMIC) {
    f &= ~CEPH_FEATURE_SERVER_MIMIC;
  }


monCommands.h里 require-osd-release需要增加
| mimic |
| ----- |
| 
增加这段, 让她可以通过mimic


    if (rel == osdmap.require_osd_release) {
      // idempotent
      err = 0;
      goto reply;
    }
    assert(osdmap.require_osd_release >= CEPH_RELEASE_LUMINOUS);
    if (rel == CEPH_RELEASE_MIMIC) {
      if (!osdmap.get_num_up_osds() && sure != "--yes-i-really-mean-it") {
        ss << "Not advisable to continue since no OSDs are up. Pass "
           << "--yes-i-really-mean-it if you really wish to continue.";
        err = -EPERM;
        goto reply;
      }
      if ((!HAVE_FEATURE(osdmap.get_up_osd_features(), SERVER_MIMIC))
           && sure != "--yes-i-really-mean-it") {
	ss << "not all up OSDs have CEPH_FEATURE_SERVER_MIMIC feature";
	err = -EPERM;
	goto reply;
      }
    } else {
      ss << "not supported for this release yet";
      err = -EPERM;
      goto reply;
    }


CEPH_FEATURES_ALL 增加MIMIC, 否则osd被判定为低版本的


合入了d9cd2d7 mon feature mimic的这个commit

ceph mon feature set mimic --yes-i-really-mean-it

目前没看到起到什么作用.

 ceph features可以忽略, 即便是16/17, 打出来都是叫luminous

SIGNIFICANT_FEATURES
好像默认osdmap用的这个feature, 这里没加上MIMIC

启动后, pg_info里确实没了, pg_stat_t里确实也没看到, 

]
osd_snap / removed_epoch_1_00000f08
osd_snap / removed_epoch_39_00000f08
osd_snap / removed_epoch_39_00000f0b
osd_snap / removed_epoch_39_00000f0e
osd_snap / removed_epoch_39_00000f11
osd_snap / removed_epoch_39_00000f14
osd_snap / removed_epoch_39_00000f17
osd_snap / removed_epoch_39_00000f1a
osd_snap / removed_epoch_39_00000f1d
osd_snap / removed_epoch_39_00000f20
osd_snap / removed_epoch_39_00000f23
osd_snap / removed_epoch_39_00000f26
osd_snap / removed_epoch_39_00000f29
osd_snap / removed_epoch_39_00000f2c
osd_snap / removed_epoch_39_00000f2f
osd_snap / removed_epoch_39_00000f32
osd_snap / removed_epoch_39_00000f35
osd_snap / removed_epoch_39_00000f38
osd_snap / removed_epoch_39_00000f3b
osd_snap / removed_epoch_39_00000f3e
osd_snap / removed_epoch_39_00000f41
osd_snap / removed_epoch_39_00000f44
osd_snap / removed_epoch_39_00000f47
osd_snap / removed_epoch_39_00000f4a
osd_snap / removed_epoch_39_00000f4d
osd_snap / removed_epoch_39_00000f50
osd_snap / removed_epoch_39_00000f53
osd_snap / removed_epoch_39_00000f56
osd_snap / removed_epoch_39_00000f59
osd_snap / removed_epoch_39_00000f5c
osd_snap / removed_epoch_39_00000f5f
osd_snap / removed_epoch_39_00000f62
osd_snap / removed_epoch_39_00000f65
osd_snap / removed_epoch_39_00000f68
osd_snap / removed_epoch_39_00000f6b
osd_snap / removed_epoch_39_00000f6e
osd_snap / removed_epoch_39_00000f71
osd_snap / removed_epoch_39_00000f74
osd_snap / removed_epoch_39_00000f77
osd_snap / removed_epoch_39_00000f7a
osd_snap / removed_epoch_39_00000f7d
osd_snap / removed_epoch_39_00000f80
osd_snap / removed_epoch_39_00000f83
osd_snap / removed_epoch_39_00000f86
osd_snap / removed_epoch_39_00000f89
osd_snap / removed_epoch_39_00000f8c
osd_snap / removed_epoch_39_00000f8f
osd_snap / removed_epoch_39_00000f92
osd_snap / removed_epoch_39_00000f95
osd_snap / removed_epoch_39_00000f98
osd_snap / removed_epoch_39_00000f9b

1. 看上去removed_snaps是生效了, 但是pg_stat_t里的purged_snaps好像没工作
2. osdmap的removed_snaps_queue确实工作了, 但是旧的removed_snaps里的好像没清理掉.




# TODO:解读phase 1.5

``` bash

41c24a13261222cc76f45a51ba213d83f566fc81 osd: remove luminous compat code for removed_snaps
713d48eff06a84aee8c98c9fed39696546e411b3 osd/osd_types: remove build_removed_snaps(), maybe_update_removed_sna…
248fe11dd25a98f82eda6af5ef91956f9131209c mon/OSDMonitor: remove support for pre-mimic conversion
04b946d9131973ada2913bef057c447c41e675d1 mon/OSDMonitor: remove pre-mimic snap behavior support
99970bb795d61fc1691e7846bd583e602f23b4d6 mon: drop mon_debug_no_require_mimic
58c4c8fc8ef9f85c3efdd31740f2a91054258b08 osd: move snap_interval_set_t to osd_types
e963ee6039a29def026d90100e2ca5454b658649 osd/PeeringState: removed pre-mimic removed snap tracking
b59a25d9c985e414e3015c76f3fd84a1525afe3c osd/PG: drop pre-mimic snap_trimq code
9b6acc2caddc235436aa1f0d6ae6d955a12dc769 osd/PeeringState: drop some mimic conditionals
3f9c28823aafc6b970eefba1455a26df98563536 mon/OSDMonitor: avoid is_removed_snap()
cabc48c40332875e8827f4f0e6dbde9ab7e9fd7a osd/PrimaryLogPG: use osdmap removed_snaps_queue for snap trimming
a6f85089c0d0213481c8038f57820d3d31cf9ebd osd/PrimaryLogPG: find_object_context: trust SnapSet's clone_snaps
25b690fe06c261648564e3e0428f2003a85b07ca messages/MRemoveSnaps: int -> int32_t on encoded type
8c0c63c7e1b4e8eeb23622ac5cf58dcec3e575c6 mds/SnapServer: int -> int32_t for encoded type
b1a5bff4a888d69bc783d8ffae855c639fb98969 osd/PrimaryLogPG: make best effort to sanitize clones on copy-from
c88d860dcef695124ae0d7af72b8e3046ee4bc41 osd/PrimaryLogPG: trim_objects: only filter SnapSet::snaps for pre-oc…
096449813272af262be27baf4205eaf3c542b78d osd/PrimaryLogPG: only filter SnapSet::snaps for flush for pre-octopu…
a362fedc81ab5ae76149397b2b1544206568ea9f osd/PrimaryLogPG: change fabrication of promoted clone snaps
f43d2147896c22a25ebe3c05296a1debbbeaea4f osd/osd_types: SnapSet::get_ssc_as_of: use clone_snaps
8f714e2ba6944f15764739f9ea9f3863d31defd1 osd/osd_types: mark SnapSet::snaps as legacy
de5d3f6a0ad2d63744fa3accb022da7896f87cc4 mon/OSDMonitor: only maintain pg_pool_t::removed_snaps for pre-octopus
d7c343eebff1b40bb7f0ecb7ca91ca2d10ed3c2f osd/PrimaryLogPG: only maintain SnapSet::snaps for pre-octopus compat
f58f5cfc5197e283bb1b1b0f234a6133b9a7feaa osd/PrimaryLogPG: use get_ssc_as_of for snapc for flushing clones
419251a401f58c78fabe006cd978a968bb6c8087 ceph_test_rados_api_tier_pp: fix osd version checks
7fe43cc4e4652a5cda6e2b789220c16677cf2582 ceph_test_rados: stop doing long object names
b945de141f3a97efd339ee82f1ccd2caeae3a40f mon/OSDMonitor: only update removed_snaps when pre-octopus
fa7655351c61d36b189131d7e3e8107605fdf54b mon/OSDMonitor: make snap removal handle dups safely
d056372e7bd50c9755e5fc866fcc6e02e9dcc63c vstart.sh: wait for mgr volume module to start up
dcc8b6b28a0b18bb4d7043bf7bdec751dfa3f05f vstart.sh: remove useless auth add for osds
75edd21c4572c833450baf7e9d68a8feec7d33e4 vnewosd.sh: add script to add a new osd to an existing vstart
bc4c31a439a2cd201d89538fd519feefeb2aeb34 mon/PaxosService: add C_ReplyOp
7eeee5b49bf67ccd197820a692472a6707e97074 CMakeLists: include 'cephfs' (which includes libcephfs) in 'vstart' t…
ab405ab56036ad44f79e36f65031585a86b1e967 mds/SnapServer: handle MRemoveSnaps acks from mon
e457b73075b63433a63f6ddbf945555f2573a5ee mon/OSDMonitor: send MRemoveSnaps back to octopus MDS
ec4cd082617b88ccb61ca74205cf38fa27b43b5e mon/OSDMonitor: use structured binding for prepare_remove_snaps
e8a359814cf533bbb4064b1bd27d40c6d7bff629 osd/SnapMapper: document stored keys and values
14fdb52c50c018c6580fc0a1115495875ebb1a9e mon/OSDMonitor: document osd snap metadata format
84bea65d3e3b4c5bf04591700f8da8e99e7a1bc2 osd/SnapMapper: include poolid in snap index
94ebe0eab968068c29fdffa1bfe68c72122db633 osd: adjust snapmapper keys on first start as octopus
a12d80a8137a47743996d89379e63cf80d005ae3 mon/OSDMonitor: lookup_pruned_snap -> lookup_purged_snap
441f42b8add06304df331b655b75da9b3a491cd8 mon/OSDMonitor: fix lookup_purged_snap implementation
19a590e11494d33477442a4f749852c38b111a74 mon/OSDMonitor: make_snap_key -> make_removed_snap_key, make_purged_s…
c0f362434560ea773350b5677365d7eed83c2c42 mon/OSDMonitor: refactor snap key and value helpers
c0233801a8bd78ce44e6bbf8baab1eae73e9fa5c mon/OSDMonitor: generalize/refactor lookup_*_snap
1e7718c37c5ebbc98d4c3caec91d71a99f5c0982 mon/OSDMonitor: move (removed, purged) snap update into a helper
acd7e903d3eb7745e92dbcbf2b851a77d0dc1929 mon/OSDMonitor: make {removed,purged}_snap storage more efficient
61d8407514da8ce68d0bed18ff5c4cbe512cdf01 osd/osd_types: clean up initial values for OSDSuperblock
3217ca77cbd683a0f5d918c4cadb0cf5a149eacc osd/osd_types: add purged_snaps_last to OSDSuperblock
847dc5fb78a6abe1ae0e010645e905fa833c16bb mon/OSDMonitor: make_snap_epoch_key -> make_removed_snap_epoch_key
47fb89c072b63202ba61e9c8bcd411816af2701b mon/OSDMonitor: record purged_snaps for each epoch
4e5093cee357c79672991d1899d30dcbe22810ac mon/OSDMonitor: record pre-octopus purged snaps with first octopus map
7315d3fdba0ab917fc2b01d1d7ae99220fcbcf43 mon/OSDMonitor: add messages to get past purged_snaps
87b539c2b65c80a105e1c374ff88cb69b571686f osd: record purged_snaps when we store new maps
81f7edc6bda47fce626abfc861b268c61c6a26bd osd: sync old purged_snaps on startup after upgrade or osd creation
8d9a27f5ea9d860f2c407346b8b7ef7b99afcbce ceph_test_rados_api_snapshots_pp: (partial) test to reproduce stray c…
7628474723489d9bb5ef97e35a2e102a8ae04d63 osd/PrimaryLogPG: always remove the snap we are trimming
6192fb60313d42761a2ff59d2dee5581a68d6139 osd: implement scrub_purged_snaps command
a1f649c2e763eb21c47f013e1bf8b6ae24700359 mds/SnapServer: make not about pre-octopus compat code
d3d06bcd7c1230e8f4d49a0dafa7b32d3860f39a osdc/Objecter: don't worry about gap_removed_snaps from map gaps
bc0553147b229855def6da92e4dd2b58a32ba9ad mon/OSDMonitor: do not bother reporting gaps in removed_snaps
d831abeae1688a18eb446dd1a63eb6ed94f45d81 mon/OSDMonitor: record snap removal seq as purged
6b26fcd1bdd8910cb2dbd84fe342cd65be2a70ec mon/OSDMonitor: fix bug in try_prune_purged_snaps
e4aed74cd4799d922b8cba2d65996b7cbe90cab5 osd/OSDMap: add last_purged_snaps_stamp to osd_xinfo_t
a03e8ab662559b395380864980bfec7958585a53 osd: record last_purged_snaps_scrub in superblock
ab475cda08ff7cf872d342caa5680a97973dc535 osd: log purged_snaps scrub to cluster log
4f4dedb8d62bcafb3c698f35ab374b7a4d72ed88 osd: report last_purged_snaps_scrub as part of beacon
85ddc1a0345c24487f85103dd6a4b9a4dad87e2b mon/OSDMonitor: record last_purged_snaps_scrub from beacon to osdmap
0d10e63d8d158ec503aab42a8b8f16997e8a8a87 ceph_test_rados_api_snapshots_pp: drop unnecessary assert
fc2a96638ddec2a9e1f5e16045adb71e816bc8c5 osd/OSDMap: SERVER_OCTOPUS feature bit is now significant
647cfb460371b2f3255160d73d7432d41fcc0b9f osd: move scrub_purged_snaps to helper
5b0ed6ff9e10fbbe70c789796099223a38996401 osd: automatically scrub purged_snaps every deep scrub interval
adacc20046c5f3a96b59df3fe01f9764e8437156 ceph_test_rados_api_tier_pp: tolerate ENOENT or success from deleted …
b17850a6653f2f5870907ee54c72955735ffc84a

```

 