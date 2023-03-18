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

# crushmap变更时, df容量不准确

* 看资源池为什么df输出的不对, 是否是osd_stat此时没收集完? 如何判断? 怀疑是pg peer完毕之前引起的. 以前创建的慢感知不到,现在可以感知了
* osd_stat
* 
TODO: 为什么网卡异常会引起应用层乱序?

* 资源池Df的信息, 目前来看, 其实是从crush rule里读的, 这里似乎维持了一个缓存?
* crush_rule显示了一个3副本前的raw 容量.
* 
如果是新建的rule, 则重新计算. 但是如果只是crush map变更呢? 如何感知重新计算呢?

不对啊, 看着是每次df都重新计算的, 那个`avail_by_rule`是个函数内变量.

没有调用啊?

我建了个新的crush rule ,为啥日志里没变更呢?

一点没有?

难道走的这个`avail_space_by_rule`

理解错了, 的确是通过这个变量缓存的.

`dump_pool_stats_full`在这里才会真实的去触发?

大致知道了, 如果这个crush rule的容量还没被加载, 返回就是0

``` c++
  int64_t get_rule_avail(int ruleno) const {
    auto i = avail_space_by_rule.find(ruleno);
    if (i != avail_space_by_rule.end())
      return avail_space_by_rule[ruleno];
    else
      return 0;
  }
```

所以, 其实就是这个容量触发更新的地方怀疑不够及时?

目前疑似`PGMapStatService`的`dump_pool_stats`触发`PGMap`的`dump_pool_stats_full` 只有这一个地方触发更新.

`pgservice->dump_pool_stats(osdmon()->osdmap, &ds, f.get(), verbose);`需要的是这个.

mgr的接口和mon的`ceph df`函数理论上触发更新?

根据这个`const MonPGStatService *pgservice;`, 其实就一个触发来源才对呀? 走`mgr`的`service`, 似乎的确不走那个更新的地方, 那我理解错了.

不对, `PGMapStatService`这里的调用是会走到`PGMap`里的`dump_pool_stats`.

这里其实继承了?`class PGMonStatService : public MonPGStatService, public PGMapStatService`


`MonPGStatService`这个和上面这几个什么关联性呢? 

和`PGMapStatService`同时被继承到一个class里?

所以关键应该是在这个共同的派生类上?

``` c++
class PGMonitor : public PaxosService {
  PGMap pg_map;
  std::unique_ptr<PGMonStatService> pgservice;
```
`PGMonStatService`的构造函数里会传入这个, 


`PGMonitor`是这个`class PGMonitor : public PaxosService `, 有点遗忘了, 负责的是`PaxosService`

是不是可以理解为, 这个就是`PGMap`的更新的一致性代码管理方.

TODO: 比如这个`PGMonStatService`使用了`PGMonitor`的哪些函数接口?

TODO: 题外话, `PaxosService`的is_readable不知道怎么理解.

获取pgmap的元数据的时候, 好像是通过这个来查的

``` c++
  void dump_info(Formatter *f) const override {
    f->dump_object("pgmap", pgmap);
    f->dump_unsigned("pgmap_first_committed", pgmon->get_first_committed());
    f->dump_unsigned("pgmap_last_committed", pgmon->get_last_committed());
  }
```

所以刚才继承的那俩类, 还有在单独提供使用吗?

`MonPGStatService`


`Monitor`里搞了个这个`const MonPGStatService *pgservice;` 现在看来这个并不是pg的`paxosservice`

TODO: `MgrStatMonitor`和`MonStatMonitor`有什么关联性?

`MgrPGStatService : public MonPGStatService `继承关系

似乎这个是在`MgrStatMonitor`这里初始化的, 所以又到了mgr这里?

根据下面这个, 是`12`版本及以后用`MgrStatMonitor`, `PGMonitor`是以前的呢?

```
  // make sure we're using the right pg service.. remove me post-luminous!
  if (osdmap.require_osd_release >= CEPH_RELEASE_LUMINOUS) {
    dout(10) << __func__ << " pgservice is mgrstat" << dendl;
    mon->pgservice = mon->mgrstatmon()->get_pg_stat_service();
  } else {
    dout(10) << __func__ << " pgservice is pg" << dendl;
    mon->pgservice = mon->pgmon()->get_pg_stat_service();
  }
```

`require_osd_release luminous`, 所以实际上走进的代码应该是MgrStatMonitor?

走的其实是`digest`的`dump_pool_stats`

// FIXME: we don't guarantee avail_space_by_rule is up-to-date before this function is invoked

官方在这里其实也写了, 等待修复.

目前来看, 只有触发一下PGMap的`get_rules_avail`才能够更新数据. 所以在Digest里有办法更新嘛?

目前初步看, 只有`ActivePyModules`里会触发PGmap的

`class ClusterState`这里是干啥的呢?

`PGMapStatService`里也会.

但是这个目前只在`ClusterState`里使用了, mon里实际上并不使用`PGMonitor`的查询接口了.

似乎只有`PGMonitor`使用`PGMap`

那digest这个到底?


> mon/PGMap: move summary information into parent PGMapDigest object

Everything summary-ish that we need to send to the mon is moved into a
parent class. The child PGMap retains the detail.

The parent gets its own encode(), and PGMap::encode_digest() will call it
to encode just the summary info.

Squashed in here is a new num_pg_by_osd that could have been done
in a preceding patch but I did things in the wrong order. :(

Signed-off-by: Sage Weil <sage@redhat.com>

所以其实这个digest就是pgmap的一个封装版本.



在`void DaemonServer::send_report`这里会触发

```
      cluster_state.with_osdmap([&](const OSDMap& osdmap) {
	  // FIXME: no easy way to get mon features here.  this will do for
	  // now, though, as long as we don't make a backward-incompat change.
	  pg_map.encode_digest(osdmap, m->get_data(), CEPH_FEATURES_ALL);
```

这里会定时将pgmap给更新?

mgr的捕捉得通过asok的文件链接直接找. mgr_tick_period

2s更新一次, 那问题就是这个日志在哪?

忘记重新编译mgr了, 重新编了之后抓到了.

现在要找的是触发的地方.

mgr_tick_period

根据下面的记录来看,确实就是2s更新一次.

如果这个间隔被我改大了之后, ceph df的结果是查不到.

所以关键问题还是crush的更新阶段?


抓一下那次异常时的osd map看看


11-12 10:38:55
```
ceph osd getmap 110 -o osdmap.110
osdmaptool osdmap.110 --export-crush crush.110
crushtool -d crush.110 -o crush.110.txt
##测试均衡分布
osdmaptool osdmap.1034  --test-map-pgs-dump --pool 10
osdmaptool osdmap.1034 --tree
#打印ceph osd tree

ceph osd  getcrushmap -o {compiled-crushmap-filename}

 ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-3/ --pgid 4.0 obj4 dump
```

2021-11-12 10:38:54.756557 7f87acd59700  0 log_channel(cluster) log [DBG] : osdmap e76: 6 total, 6 up, 6 in

ceph osd dump 76
得到的没有容量

`osdmon()->osdmap`

```
  class OSDMonitor *osdmon() {
    return (class OSDMonitor *)paxos_service[PAXOS_OSDMAP];
  }
```

其实计算的时候, 还需要一个osd_stat

所以有没有办法得到完整的`PGMap`?

资源池存在,但是crush rule是新建的这种.

复现了. emm.

第一种: 

```
(gdb) bt
#0  PGMap::get_rules_avail (this=this@entry=0x5627ee1b9f90, osdmap=..., avail_map=avail_map@entry=0x5627ee1b9fc0) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mon/PGMap.cc:940
#1  0x00005627e27d4846 in PGMap::encode_digest (this=this@entry=0x5627ee1b9f90, osdmap=..., bl=..., features=features@entry=4611087853746454523)
    at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mon/PGMap.cc:1579
#2  0x00005627e28193a3 in operator() (osdmap=..., __closure=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/DaemonServer.cc:1454
#3  with_osdmap<DaemonServer::send_report()::__lambda31::__lambda32> (cb=<optimized out>, this=0x7ffe277a5fa0) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/osdc/Objecter.h:2045
#4  with_osdmap<DaemonServer::send_report()::__lambda31::__lambda32> (this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ClusterState.h:129
#5  operator() (pg_map=..., __closure=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/DaemonServer.cc:1470
#6  with_pgmap<DaemonServer::send_report()::__lambda31> (cb=<optimized out>, this=0x5627ee1b9c30) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ClusterState.h:105
#7  DaemonServer::send_report (this=this@entry=0x5627ee1ba930) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/DaemonServer.cc:1471
#8  0x00005627e286ba2e in Mgr::tick (this=0x5627ee1b9800) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/Mgr.cc:626
#9  0x00005627e28621fd in MgrStandby::tick (this=0x7ffe277a59e0) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/MgrStandby.cc:197
#10 0x00005627e282668a in operator() (a0=<optimized out>, this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/build/boost/include/boost/function/function_template.hpp:760
#11 FunctionContext::finish (this=<optimized out>, r=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/include/Context.h:493
#12 0x00005627e2821bc9 in Context::complete (this=0x5627ed8f6c30, r=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/include/Context.h:70
#13 0x00005627e2999074 in SafeTimer::timer_thread (this=0x7ffe277a76f0) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/common/Timer.cc:97
#14 0x00005627e299aa9d in SafeTimerThread::entry (this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/common/Timer.cc:30
#15 0x00007f365a1dedd5 in start_thread () from /lib64/libpthread.so.0
#16 0x00007f36592b9ead in clone () from /lib64/libc.so.6
```


第二种


```
(gdb) bt
#0  PGMap::get_rules_avail (this=this@entry=0x559f786fff90, osdmap=..., avail_map=avail_map@entry=0x559f786fffc0) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mon/PGMap.cc:940
#1  0x0000559f6d4bbe50 in PGMap::dump_pool_stats_full (this=0x559f786fff90, osd_map=..., ss=0x0, f=0x7ff400f95f60, verbose=<optimized out>)
    at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mon/PGMap.h:454
#2  0x0000559f6d4ecb7d in operator() (osd_map=..., __closure=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ActivePyModules.cc:296
#3  with_osdmap<ActivePyModules::get_python(const string&)::__lambda17::__lambda18> (cb=<optimized out>, this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/osdc/Objecter.h:2045
#4  with_osdmap<ActivePyModules::get_python(const string&)::__lambda17::__lambda18> (this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ClusterState.h:129
#5  operator() (pg_map=..., __closure=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ActivePyModules.cc:297
#6  with_pgmap<ActivePyModules::get_python(const string&)::__lambda17> (cb=<optimized out>, this=0x559f786ffc30) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ClusterState.h:105
#7  ActivePyModules::get_python (this=0x559f779abe00, what="df") at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ActivePyModules.cc:298
#8  0x0000559f6d50b1fd in ceph_state_get (self=0x7ff4015c10a0, args=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/BaseMgrModule.cc:334
#9  0x00007ff42b518cf0 in PyEval_EvalFrameEx () from /lib64/libpython2.7.so.1.0
#10 0x00007ff42b5186bd in PyEval_EvalFrameEx () from /lib64/libpython2.7.so.1.0
#11 0x00007ff42b5186bd in PyEval_EvalFrameEx () from /lib64/libpython2.7.so.1.0
#12 0x00007ff42b51b03d in PyEval_EvalCodeEx () from /lib64/libpython2.7.so.1.0
#13 0x00007ff42b4a4978 in function_call () from /lib64/libpython2.7.so.1.0
#14 0x00007ff42b47fa63 in PyObject_Call () from /lib64/libpython2.7.so.1.0
#15 0x00007ff42b48ea55 in instancemethod_call () from /lib64/libpython2.7.so.1.0
#16 0x00007ff42b47fa63 in PyObject_Call () from /lib64/libpython2.7.so.1.0
#17 0x00007ff42b47fb45 in call_function_tail () from /lib64/libpython2.7.so.1.0
#18 0x00007ff42b47fe7b in PyObject_CallMethod () from /lib64/libpython2.7.so.1.0
#19 0x0000559f6d50fcff in ActivePyModule::notify (this=0x559f7841f440, notify_type="pg_summary", notify_id="") at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/mgr/ActivePyModule.cc:90
#20 0x0000559f6d4da68a in operator() (a0=<optimized out>, this=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/build/boost/include/boost/function/function_template.hpp:760
#21 FunctionContext::finish (this=<optimized out>, r=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/include/Context.h:493
#22 0x0000559f6d4d5bc9 in Context::complete (this=0x559f78b0e6f0, r=<optimized out>) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/include/Context.h:70
#23 0x0000559f6d64f748 in Finisher::finisher_thread_entry (this=0x559f786ff940) at /home/wangchao/rpmbuild/BUILD/ceph-12.2.12/src/common/Finisher.cc:72
#24 0x00007ff4295a9dd5 in start_thread () from /lib64/libpthread.so.0
#25 0x00007ff428684ead in clone () from /lib64/libc.so.6
```



ceph mon remove {mon-id}

奇怪, 现阶段的问题是, 资源池都不显示. 而不是单纯不显示数字.

osdmap意思是也没更新引起的?

资源池名字和crush rule也都是从osd map里拿的. 那为啥会出现容量为0, 但是资源池存在的情况呢?

那就代表上一次的osd map里还存在crush rule为0, 但是pool存在的那次.


## TODO: 这中间也没干啥, 数字在变化? 默认的元数据?

2021-11-15 10:00:02.105074 7f0947dbe700  4 wc 1: 128092917534
2021-11-15 10:02:01.659658 7f0947dbe700  4 wc 1: 128092671774
2021-11-15 10:03:01.454611 7f0947dbe700  4 wc 1: 128092426014
2021-11-15 10:10:01.699644 7f2e29b8c700  4 wc 1: 128092770078
2021-11-15 10:10:01.854774 7f2e29b8c700  4 wc 1: 128092770078
2021-11-15 10:11:01.675594 7f2e29b8c700  4 wc 1: 128092917534






TODO: 待整理本片的内容.



* pgmap 是怎么知道他用的哪个pgmap_digest的变量的呢? 是全局只会有一个嘛? 还是怎么做到的呢?
* 

`Mgr`里似乎做的初始化`DaemonServer`, `DaemonServer`初始化的时候从上级收到`ClusterState`对象.


``` c++
Mgr::Mgr(MonClient *monc_, const MgrMap& mgrmap,
         PyModuleRegistry *py_module_registry_,
	 Messenger *clientm_, Objecter *objecter_,
	 Client* client_, LogChannelRef clog_, LogChannelRef audit_clog_) :
  monc(monc_),
  objecter(objecter_),
  client(client_),
  client_messenger(clientm_),
  lock("Mgr::lock"),
  timer(g_ceph_context, lock),
  finisher(g_ceph_context, "Mgr", "mgr-fin"),
  digest_received(false),
  py_module_registry(py_module_registry_),
  cluster_state(monc, nullptr, mgrmap),
  server(monc, finisher, daemon_state, cluster_state, *py_module_registry,
         clog_, audit_clog_),
  clog(clog_),
  audit_clog(audit_clog_),
  initialized(false),
  initializing(false)
```

TODO:`cluster_state`是从monc来的?

``` c++
// ceph_mgr.c
  MgrStandby mgr(argc, argv);
  int rc = mgr.init();
```

