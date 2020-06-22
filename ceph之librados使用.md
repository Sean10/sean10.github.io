---
title: ceph之librados使用
subtitle: tech
date: 2020-05-21 03:43:44
updated:
tags: [ceph, 存储, rados]
categories: [专业]
---

librados是ceph各组件对外暴露的模块，藉由librados接口，可以高效的使用ceph内的组件进行CRUD等操作。

<!--more-->

``` python
import rados
cluster = rados.Rados(conffile = 'ceph.conf', conf = dict (keyring = '/path/to/keyring'))
cluster.connect()
cmd = json.dumps({"prefix": "osd safe-to-destroy", "ids": ["2"], "format": "json"})
c.mon_command(cmd, b'')
```
这里的conf=keyring主要用于当打开了cephx等认证措施时，ceph.conf中又没有记录认证所用到的keyring文件路径时，进行额外设置使用。

最近我在参照python的librados使用，调用C的接口时，就一直遇到cephx认证打开之后，无法成功`rados_connect`的情况，具体原理还得细看，但应该和cephx及这个keyring存在强相关性是可以确认的了。

连接成功之后，主要使用下属这几个接口来查询`pool`,`osd`,`pg`等一些信息。

``` python
Rados.mon_command(self, cmd, inbuf, timeout=0, target=None)
Rados.osd_command(self, osdid, cmd, inbuf, timeout=0)
Rados.mgr_command(self, cmd, inbuf, timeout=0, target=None)
Rados.pg_command(self, pgid, cmd, inbuf, timeout=0)
```

就目前而言，我在抓取IO等指标时，用的比较多的接口是`mon_command`和`mgr_command`，使用这几个接口能进行的操作，都有提供cli和rest接口。我主要是在使用cli熟悉接口之后，会去代码中的`MonCommands.h`和`MgrCommands.h`文件中去找相近的prefix，然后参照着使用。

但是偶尔也是会遇到像`ceph osd pool stats`和`ceph pg ls-by-pools`这类没怎么能找到相关prefix的情况。这个时候可以充分利用起ceph这个cli入口其实是个python文本的功能了。

`python -m pgd /bin/ceph osd pool stats`执行，在`new_style_command`函数内的`json_command`处打断点，进入后，打印`cmddict`就可以看到cli发送出去的prefix拼接出来是什么样子的了，然后再拿着这个去代码里搜就好了。



## Reference
1. [Librados \(Python\) — Ceph Documentation](https://docs.ceph.com/docs/master/rados/api/python/)