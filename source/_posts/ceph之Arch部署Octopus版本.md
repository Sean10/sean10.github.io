---
title: ceph之Arch部署Octopus版本
subtitle: arch
date: 2021-03-21 21:55:55
updated:
tags: [ceph, arch]
categories: [专业]
---

# arch安装实验

``` bash
pacman -Syy ceph
```


安装完发生了一个奇怪的问题, `ceph -v`执行报缺`rados`库, python3-rados装到了`python3.9`的`site-package`里, 但是我设备里`pacman`目前只装了`python3.8`, 这个的包里没有看到这个库

执行了`pacman -Syy python`升级到`3.9`, 可以用了, 升级过程中`python3.9`的库里少了好多我之前安装图形界面用到的`python`库.  我最后是把`3.8`的`site-package`里的包不重复的复制进去安装的.

`arch`有一个好处, 大佬们多, 源里最新版的东西很多. 虽然`pacman`里没有, 但是`AUR`源里有.

`yay -Syy cephadm`就安装上了.


## 部署
cephadm bootstrap --mon-ip 192.168.115.130

发现官方文档里没提需要安装`chrony`, `docker`,配置`hostname`, 配置`/etc/hosts`.

都配置完之后. 很可惜还是出现了失败.

通过`ceph log last cephadm`查到, 失败原因是`ssh`连不上, 然后我发现我的`arch`默认没装`openssh`导致的. 安装之后就通过了.
i
然后创建几块虚拟盘进行操作. 不过我这里用默认的自动对全盘进行创建的逻辑无效, `ceph orch apply osd --all-available-devices --dry-run`扫了下,我的环境好像不符合条件. 我猜是盘的格式化问题(我没有用sgdisk初始化过, 所以应该还是一个不符合ceph默认用的分区表的盘)

不过通过单独指定`ceph orch daemon add osd ceph01:/dev/sdb`, 还是可以成功的.

成功建出来了, 发现默认就有一个`device_health_metrics`的名字的资源池.

## 重设

sudo cephadm rm-cluster --fsid   d9850914-866a-11eb-8eec-000c2934ee39 --force

### 一些小问题
我居然在没清理干净集群的情况下, 成功部署了一次?


## 快速启动ceph调试环境

源码版本是有`vstart`的, 不知道生产版本有没有, emm, 没有.


# Reference
1. [使用cephadm快速搭建ceph集群\-胡源的博客\-51CTO博客](https://blog.51cto.com/hongchen99/2507660)