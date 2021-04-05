---
title: KVM虚拟机镜像压缩
date: 2018-07-09 21:39:49
updated:
tags: [libvirt, KVM]
categories: [专业]
---

最近刚入职，做的还比较简单，熟悉libvirt，把KVM镜像给压缩一下。

<!--more-->

KVM主要硬盘镜像有qcow2和raw两种类型。

qcow2格式是copy on write的，使用时才占用硬盘空间，而raw则是创建时即分配了。

使用`ll -h`和`du -h`可以看到实际分配的和磁盘占用的大小。

```
qemu-img convert -f qcow2 -c -O qcow2 xx.qcow2 xx_compressed.qcow2
```

通过上述命令可以实现qcow2格式的压缩，但是无法更改硬盘初始创建时上限的大小。只有raw格式可以减小上限，一开始以为只有上限压缩了以后，才能把这样的镜像放到较小的硬盘分区里。

首先尝试了
```
qemu-img convert -f qcow2 -O raw xxx.qcow2 xxx.raw
qemu-img resize -f raw xxx.raw -10G
qemu-img convert -f raw -O qcow2 xxx.raw xxx.qcow2
```
但，的确就和其他博客里说的一样，这样会由于raw内部分区并不是顺序，而不是稀疏的，会导致破坏文件系统，如windows启动时就会导致蓝屏。

如果要这样做，就需要首先让系统分区进行压缩，如使用win7的磁盘管理进行压缩卷操作，亦或是使用gparted、libguestfs进行处理。

这里有个问题，qcow2格式用`qemu-img info`看到的大小是3.7G，用raw格式也仅有7.8G左右，而在win7内部磁盘管理看到的空间占用却是11.2G。这里的不对称是因为文件系统格式的原因吗？

一开始一直在尝试安装libguestfs，因为似乎`virt-resize`可以做到压缩上限，但是很可惜，内网环境yum安装依赖实在是太艰难了，在将CentOS安装包的iso镜像挂载上作本地源之后，还是存在一些依赖问题。折腾了快有2天时间，在supermin5上还是没能搞定。

最后，在看qcow2压缩的博客时，发现没有人提到过要修改这个上限的问题，qcow2本身就是写时分配空间，，那么其实，是不是无关紧要呢？我分配了一个4G的硬盘，将3.7G压缩后的镜像移动到这块硬盘再启动，windows启动一点没有问题，在系统内部的文件分区显示依旧是11.2G左右。

哎，尝试了好多，最后还是回到了原点。
