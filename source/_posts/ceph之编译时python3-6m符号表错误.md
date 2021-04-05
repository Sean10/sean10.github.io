---
title: ceph之编译时python3.6m符号表错误
subtitle: tech
date: 2020-12-02 14:22:56
updated:
tags: [ceph, 编译, python, CentOS]
categories: [专业]
---

今天在编译`ceph`时, 发生了`libpython3.6m.a: could not read symbols: Bad value`的问题, 在这附近有一个疑似相关的具体符号表的提示.

```
/usr/binld: /usr/lib/python3.6/config-3.6m-x86_64-linux-gnu/libpython3.6m.a(abstract.o): relocation R_X86_64_32S against `_Py_NotImplementedStruct` can not be used when makeing a shared objectl recompile with -fPIC
```

第一感觉看上去的意思是ceph编译时尝试去链接的静态库的符号表没有通过`-fPIC`的方式生成,从而完成共享库代码段复用的功能. 

但是回忆一下这个问题发生的背景, 如果我用的是同一套yum源安装的依赖, 理论上python出现小版本不兼容的可能性较低.

那么, 就存在一个可能性, 这个报错链接找到的库并不是yum源里安装的, 可能是同事通过源码编译安装的python的库.

通过`rpm -qf /usr/lib/python3.6/config-3.6m-x86_64-linux-gnu/libpython3.6m.a`查到这个库并不是安装的, 验证了我的猜测. 而`rpm -qf /usr/lib64/python3.6/config-3.6m-x86_64-linux-gnu`则发现这里的文件才是`python36-libs`里提供的.

既然如此, 就将`/usr/lib`下的这个版本错误的库挪走, 重新编译, 这次报的是`-lpython3.6m`未找到.

查看`/usr/lib64/libpython3.6m.so`发现这个文件并不存在, 只有`/usr/lib64/libpython3.6m.so.1.0`存在,而软链并不存在.

通过`rpm -ql python36-libs`发现的确这个包中并不提供指向的链接.

根据目前对包管理的理解, 怀疑这个缺失的指向真实文件的软链很有可能在`python36-devel包中`, 安装后果然如此, 链接找到了. 执行编译, 毫无问题了~

# 引申的疑问

因为目前出现的python3.6m.so这个库的疑问, python3-libs这个库里没有提供指向.so的链接, 而是在python3-devel包中存在

而python34则都是在python34-libs里.

但是python36应该是能够正常使用的吧?

那是不是其实可以作答哦这样一点, 如果编译时我指定了链接, 然后在符号表中, 他就能找到针对这个链接指向的真实路径 ,然后存起来.

到生产环境里, 如果这个路径中存在, 就不需要ldconfig的默认路径了呢? 会不会有这个选择呢?

还是说这个libpython3.6m.so这个库基本只会在编译C与python之间的库时使用,而且只编倾向于静态的? 然后这样编完到生产环境里就不需要了呢?

