---
title: linux网口排序
subtitle: tech
date: 2018-12-04 23:33:18
updated:
tags: [linux]
categories: [专业]
---

## 基础概念
### 网口命名，ens\eno\eth区别
首先我们有三块网卡，两种驱动

2）udev开启，扫描设备，加载驱动，内核给设备命名

3）假如内核命名的eth0 驱动是e1000e，进入了udev的规则，我们要给它改成eth3

4）但是在eth0进入udev没修改成eth3的时候，内核又将一个网络端口设备命名为eth3

5）则我们将eth0修改为eth3的时候，就会冲突，eth0 变成了eth3_rename

4）针对3）的问题，在init.d的网络服务启动之前（network），即在network脚本里靠前部分，加入一段代码，用于处理_rename问题。

1）通过ifconfig查找当前的端口名（显示全部 加参数-a）

2）通过1）查找的端口名和规则B进行比对，出现问题，则进行修改

用户态程序，可以通过ioctl系统调用，去修改网卡的名称。

## Reference
1. [linux 如何更改网卡的顺序](http://blog.sina.com.cn/s/blog_413d250e0102w2ut.html)
2. [linux网卡命名规则变为eno分析？ \- 叶赫那拉坤的博客 \- CSDN博客](https://blog.csdn.net/u010558281/article/details/68488791)