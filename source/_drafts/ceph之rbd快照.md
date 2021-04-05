---
title: ceph之rbd快照snapshot
subtitle: tech
date: 2020-05-25 10:09:44
updated:
tags: [ceph, rbd, rados]
categories: [专业]
---

rbd的快照实现是基于rados快照实现的。与资源池的快照不同

<!--more-->

rbd主要采用的是基于COW的快照。而如IBM XIV等应用了ROW技术的快照。

COW和ROW主要区别就是一个是存在写性能损耗，一个存在读性能损耗。但是ROW在做完快照后，如果对象单元是1M，新的修改只写入8K，则还得从原来的数据那里复制过来，这样写性能和COW的差距反而小了。[^2]不过ROW在分布式场景下，因为分布式本就会把对象crush到不同的节点上存储，这样在磁盘性能会成为瓶颈的场景下，ROW由于会充分利用起多个节点，反而性能不会逊色COW多少。

rbd在做快照期间，会protect源数据，保障元数据的不可写，防止快照创建出后不一致。（这个protect期间，应该是会将需要写入的内容存到栈或者队列中临时阻塞的吧？这个队列长度能支撑多少？）

## 快照Clone

在Snapshot的实现上，最重要的其实就是Clone操作，那么在FileStore层面，Object数据存储是实际上就是一个文件，Object间克隆依赖OSD数据目录的文件系统，如Ext4或者XFS会直接完全拷贝数据，使用Btrfs会利用ioctl的BTRFS_IOC_CLONE_RANGE命令，kv数据克隆通过一个巧妙的KeyMapping实现COW策略(略微复杂，后面文章解读)，而xattr则完全copy实现(xattr在Ceph中较少用到)。

## 
>Note Rolling back an image to a snapshot means overwriting the current version of the image with data from a snapshot. The time it takes to execute a rollback increases with the size of the image. It is faster to clone from a snapshot than to rollback an image to a snapshot, and it is the preferred method of returning to a pre-existing state.

官方更推荐做clone?但是这样的话，怎么让上层使用的rbd切换到这个新作的clone呢？

>Note You may clone a snapshot from one pool to an image in another pool. For example, you may maintain read-only images and snapshots as templates in one pool, and writeable clones in another pool.

clone的话就会出现多层链路，导致性能下降的问题了把。

## Reference
1. 《Ceph设计原理与实现》
2. [存储专栏：深度解读高端存储的快照技术\_存储在线](http://www.dostor.com/article/2013-09-04/7686494.shtml)
3. [解析CEPH: 存储引擎实现之一 filestore\-西伯利亚·狼\-51CTO博客](https://blog.51cto.com/kernal/1540535)