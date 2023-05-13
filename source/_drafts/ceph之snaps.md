---
title: ceph之snaps
subtitle: tech
date: 2023-05-14 01:25:16
updated:
tags: [ceph, snap]
categories: [专业]
---




interval_set这里到底起的什么作用? 感觉就是内部的map<snapid_t>的封装? 

目前打印只提供了start和len, 这个和snapid_t, snapid_t好像就是个int类型的封装? 但是好像没有len概念?

好吧, snapid_t就是int, start和len 都是int型, 其实是借用了snapid_t这个int型, 把长度复用了这个数据结构




### 下一个关键问题, purged_snaps不同的pg, 怎么得到不同的结果? 比如和snaptrimq, removed_snaps关联? 

关键是info.purged_snaps里, 看起来query出来的就是主的.

对于那些cpu消耗不高的节点, 是否在进入该逻辑时, 实际上purged_snaps并没有那么多呢?


> osd: make fastinfo apply conditionally
> 
> This way we don't have to bother deleting the fastinfo
attr, which will reduce the amount of work the
> underlying store has to do.


``` c++
  // pg state
  pg_info_t info;               ///< current pg info
  pg_info_t last_written_info;  ///< last written info
```
 
初步理解是, 这里的目的是判断是否`last_written_info`和`info`没有变化? 如果没有变, 就只更新`pg_fast_info_t fast`的数据, 就可以去写pg_log了?

所以可以看下`l_osd_pg_fastinfo`, 是否多个osd之间存在差异? 差别几乎没有....

`l_osd_pg_biginfo`