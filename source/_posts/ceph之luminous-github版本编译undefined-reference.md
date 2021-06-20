---
title: ceph之luminous_github版本编译undefined_reference
subtitle: build
date: 2021-05-15 21:34:46
updated:
tags: [ceph]
categories: [专业]
---

主要问题是从svn下载的最新版的liminous分支, 编译时报这个错误

``` bash
undefined reference to OSD::HeartbeatInfo::HEARTBEAT_MAX_CONN
``


这条修复了这个问题[osd: make OSD::HEARTBEAT\_MAX\_CONN inline by tchaikov · Pull Request \#23424 · ceph/ceph](https://github.com/ceph/ceph/pull/23424)

但是只合入了`master`分支, luminos分支没有合入.

但是上面这个好像是高版本解决的问题

`static constexpr int`.

[C\+\+ Linker Error With Class static constexpr \- Stack Overflow](https://stackoverflow.com/questions/8452952/c-linker-error-with-class-static-constexpr)

好像是c++17才修复的问题,

最后我参照下面这个, 在常量外侧加一层int转换. 原理目前不懂

``` c++
return std::min(int(S::X), 0);
```

