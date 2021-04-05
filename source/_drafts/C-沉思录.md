---
title: C++沉思录
date: 2018-04-04 22:21:37
updated:
tags: [C++]
categories: [专业]
---

read ini配置文件,glib自带一个函数

[Key\-value file parser: GLib Reference Manual](https://developer.gnome.org/glib/unstable/glib-Key-value-file-parser.html)

[Strings: GLib Reference Manual](https://developer.gnome.org/glib/stable/glib-Strings.html)


[A libyaml Tutorial](https://www.wpsoftware.net/andrew/pages/libyaml.html)
[libyaml 解析配置文件 \| 属乌鸦的](http://www.hyuuhit.com/2019/04/20/libyaml/)

[libyaml examples](https://gist.github.com/meffie/89d106a86b81c579c2b2a1895ffa18b0)
上面这篇libyaml event的使用，我觉得比较好，不过发现libyaml好久没人维护了，算了。

gdb

-ggdb3

set logging on 
thread apply all bt



set logging of

set scheduler-locking off|on|step：在调式某一个线程时，其他线程是否执行。off，不锁定任何线程，默认值。on，锁定其他线程，只有当前线程执行。step，在step（单步）时，只有被调试线程运行。

set non-stop on/off:当调式一个线程时，其他线程是否运行。

set pagination on/off:在使用backtrace时，在分页时是否停止。

set target-async on/ff:同步和异步。同步，gdb在输出提示符之前等待程序报告一些线程已经终止的信息。而异步的则是直接返回。

[C语言头文件组织与包含原则 \- clover\_toeic \- 博客园](https://www.cnblogs.com/clover-toeic/p/3728026.html)

是不是多文件编译动态库，不允许直接使用其他.o里的内容啊

<!--more-->


gcc -lyaml --verbose