---
title: ceph之df的MAX AVAIL
subtitle: tech
date: 2020-05-28 18:50:05
updated:
tags: [ceph]
categories: [专业]
---

ceph df输出的统计数据的计算法昂是比我一眼以为的要复杂一些。

尤其是MAX AVAIL这个值。目前遇到的问题是发现在L版本,当osd out了之后，`ceph df`拿到的`MAX AVAIL`始终没有变化。

<!--more-->
这样对于使用者来说是存在问题的，当出现降级的情况时，若业务层使用这个接口看到资源池仍有可用空间，但实际上依旧up的osd并不能提供这些空间，是会导致业务人员出现误判……直接导致业务数据写不下去的情况的。

代码里是这样写了，但是是否上面的想法只是我们理解错误产生的误用呢？

向上追溯一下这段关键计算的提交记录吧，看看有没有相关场景。

先搜索主要代码

``` c++
// PGMap.cc
int64_t proj = (int64_t)(avail / (double)p->second);
```

利用blame查找每行的记录，还是不错的。

[Bug \#10257: Ceph df doesn't report MAX AVAIL correctly when using rulesets and OSD in ruleset is down and out \- Ceph \- Ceph](https://tracker.ceph.com/issues/10257#note-1)

这篇里只加了跳过kb=0的选项

[mon: incorrect MAX AVAIL in "ceph df" by liuchang0812 · Pull Request \#17513 · ceph/ceph](https://github.com/ceph/ceph/pull/17513)

这篇里好像是之前漏了/raw_used_rate这个副本size

[mon/PGMap: factor mon\_osd\_full\_ratio into MAX AVAIL calc by liewegas · Pull Request \#12923 · ceph/ceph](https://github.com/ceph/ceph/pull/12923/commits/f223ac92917f4bc18e5b9b3ad61afa155e4d088a)

这个操作引入的full_ratio参数到MAX AVAIL计算中

[mon/PGMap: Fix %USED calculation bug\. · ceph/ceph@d10c6c2](https://github.com/ceph/ceph/commit/d10c6c26f9ef7372e2c95da79d23b07ce5f4e0e5#diff-ecab4c883be988760d61a8a883ddc23f)

修复%USED（raw的应该) 没有乘以副本数的问题。

[osd,os,mon: extend 'ceph df' report to provide both USED and RAW\_USED · ceph/ceph@7ca25df](https://github.com/ceph/ceph/commit/7ca25dfd5a4f05d1f942b73f08e82b8694a73c99#diff-ecab4c883be988760d61a8a883ddc23f)

这个应该是在L之后的版本，在这次commit中引入Statfs这个中间结构用来存储total、avail\used数据，免于直接接触kb

[mon/PGMap: GLOBAL \-> RAW STORAGE in 'df' output · ceph/ceph@738789b](https://github.com/ceph/ceph/commit/738789b0577f879bb756d65b71066300902ab116#diff-ecab4c883be988760d61a8a883ddc23f)

这个也是L之后的版本，把ceph df输出的GLOBAL改成了RAW，防止错误理解

get_rule_weight_osd_map这个函数的新增并没有看出有立刻被用在计算资源池容量这里。

应该可以从这个接口被调用的地方来分析，新增的人是否没考虑过osd异常的场景？

[mon/PGMap: move summary information into parent PGMapDigest object · ceph/ceph@8f25216](https://github.com/ceph/ceph/commit/8f2521617ee31453909ddd0a6a5fd177c8fbf034)

这里被重构，挪了一下位置……

[mon/PGMap: factor mon\_osd\_full\_ratio into MAX AVAIL calc · ceph/ceph@f223ac9](https://github.com/ceph/ceph/commit/f223ac92917f4bc18e5b9b3ad61afa155e4d088a)

这里又被做了格式化，哎

[mon: move "df" dump code from PGMonitor to PGMap · ceph/ceph@519a01d](https://github.com/ceph/ceph/commit/519a01d973f10b97e4ba497e065fdf53b6064029)

这里又重构了……

在这次提交以前这部分代码在src/mon/PGMonitor.cc 

[mon/pgmonitor: use appropriate forced conversions in get\_rule\_avail · ceph/ceph@024caa7](https://github.com/ceph/ceph/commit/024caa721d6632eda0cdcdd704188de209f30bd6)

哎，这次对这行改了下显式类型转换，从float改成double

[Bug \#10257: Ceph df doesn't report MAX AVAIL correctly when using rulesets and OSD in ruleset is down and out \- Ceph \- Ceph](https://tracker.ceph.com/issues/10257#note-1)

这个看起来和OSD out相关，但很可惜，只处理了错误计算为0的问题，并没有考虑正常场景？


[mon: include 'max avail' in df output · ceph/ceph@7a9652b](https://github.com/ceph/ceph/commit/7a9652b58ea70f9a484a135bde20d872616c5947)

终于找到了引入的地方,这里增加的max avail的值。

[PGMonitor: fix bug in caculating pool avail space · ceph/ceph@04d0526](https://github.com/ceph/ceph/commit/04d0526718ccfc220b4fe0c9046ac58899d9dafc)

这里处理的那个osd_map

## 提交

在最新的分支里，也没有相关信息。看起来得提个ISSUE看看有没有答复了。

``` md
Environment: Luminous 12.2.12

I have a question about the pool's `MAX AVAIL` of `ceph df`.

When i out a osd, the `MAX AVAIL` doesn't change. Only when i remove the osd from the crush map of the pool, the `MAX AVAIL` decrese.

For example,  I have a pool with 10 osd. When 9 osds out, `ceph df` still output the MAX AVAIL with 10 osds, not the remained 1 osd. Only when i remove the out osds from the crush map, the MAX AVAIL changed to the remained size. 

But in my understanding, when the osd out, the recovery begin. When the recovery begins, the out osd has no use for the pool. If `MAX AVAIL` doesn't change, It may provide error avail size for the users.

The calculate logic is in `PGMap::get_rules_avail`. Maybe after called `get_rule_weight_osd_map`, we could recalculate the weight map to remove the osd which `osd_info.kb` is zero. 

Or could modify the `get_rule_weight_osd_map`, but i don't know whether there is some other impact.
```
新建了这个issue, 希望能得到回复吧。

[Bug \#45809: When out a osd, the \`MAX AVAIL\` doesn't change\. \- Ceph \- Ceph](https://tracker.ceph.com/issues/45809)

