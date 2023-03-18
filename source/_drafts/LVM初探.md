---
title: LVM初探
subtitle: tech
date: 2022-05-22 10:22:15
updated:
tags: [lvm, linux]
categories: [专业]
---


`/etc/lvm/archive` 和`/etc/lvm/backup`可以查看每个lvm和vg创建出来的时间

[CentOS / RHEL : How to find the creation time of LVM volume – The Geek Diary](https://www.thegeekdiary.com/centos-rhel-how-to-find-the-creation-time-of-lvm-volume/)


### lvm扩容

如果是所属的pool依旧有空间的, 直接敲扩容命令即可.

``` bash
lvextend -L +45G EXTENDED/mon
xfs_growfs /dev/mapper/mon
```