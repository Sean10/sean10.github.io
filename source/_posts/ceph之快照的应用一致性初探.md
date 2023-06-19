---
title: ceph之快照的应用一致性初探
tags:
  - ceph
  - consistency
  - snapshot
categories:
  - 专业
subtitle: tech
date: 2023-06-11 23:52:15
updated:
---


以下讨论都是基于存储系统上的, 主要描述的为对块存储进行快照.

根据目前理解, 应用一致性可以粗分为

* 基于时间点的快照
	* 没有明确的一致性等级支持
* 支持崩溃一致性的快照(Crash-consistent backup)
* 应用一致性快照. (Application-consistent backup) (包含内存中pending IO)
	* 包含内存(pending IO)的快照
		* (存在大量停机时间)
	* mirror的同步技术的, 包含pending IO
	* 较通用的实现方案
		* (存在少量停机时间, 主要取决于内存缓了多少?)

## 基于时间点的快照(不一定一致/不一致)

很简单粗暴, 不管是否有 pending IO, 直接保存当前的硬盘状态, 做COW/ROW链接, 完成快照.

## 支持崩溃一致性的快照


> 采用崩溃一致性的备份来恢复云服务器，在数据恢复后，由于没有保证数据库事务的一致性，通常需要依赖数据自身的保护机制做自动的日志回滚，数据才能正常启动，数据是恢复到离备份时间点最近的一个一致性状态，相比应用一致性备份的恢复，RPO（Recovery Point Objective，指的是最多可能丢失的数据的时长）会更大，RTO（Recovery Time Objective 指的是从灾难发生到整个系统恢复正常所需要的最大时长）也更长。

基本上表达的一种实现方式可能就是会建多个快照, 然后会识别多个快照内业务的可用性, 无法通过数据库/文件系统修复工具回滚继续提供服务, 则去触发下一个快照. 直到找到一个能用的快照?
## 应用一致性快照. (Application-consistent backup)

### 包含内存(pending IO)的快照

以qemu和qcow2为例, qcow2支持内存快照, 但是是依赖于虚拟机先由qemu控制进入paused状态的. 这部分一般由于内存一般划的比较大, 内存完整写入, 一般比较慢. 停机时间特别长, 一般都是不太能接受的. 虽然应用一致性确实保障了.

### mirror的同步技术的快照

TODO: 这个暂时还支持猜想, 还没去细找是否有基于这类实现的.

诸如drbd/rbd mirror等, 本身除了这个业务本身, 还提供了其他设备进行备份, 其实应该算得上是备份技术了. 它确实能保障应用一致性, 但是其实是双活了. 针对快照时间点, 是在对端做flush, 然后创建快照了. 

这个快照点, 基本上可以是在mirror对端按事件进行冻结, 然后flush, 然后创建快照了. 对本端无影响.



### 应用一致性快照

按照目前的认知, 这个可能是目前用的比较多的方案. 

主要就是在创建快照前和后分别提供了一个hook接口. 针对文件系统类具有一定的通用约束的, 可以直接默认使用`fsfreeze` 类的偏通用的接口完成静默, 然后再进行对应的pending IO的持久化. 

非文件系统类业务, 则需要上层业务自己去触发了, 就是根据对应的数据库开发对应的持久化方式, 完成快照的一致性保障. 如果是明确指定类型的数据库, 可能还能直接配合对应的方案做自动化检测和识别去完成对应数据库内的持久化快照, 无需用户自己实现. 

不过当客户机上是windows 时, 则直接可以通过VSS来实现应用一致性快照. 无需开发介入了.


看到阿里云的文档里也写的很清楚, 如下表格.

| 类型        | 说明                                                                                                                                                                                      | 实现方式                                                                                                                                      |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| 应用一致性快照   | 应用一致性快照在快照创建时刻备份内存数据及正在进行中的数据库事务，保证应用系统数据和数据库事务的一致性。通过应用一致性快照，没有数据的损坏及丢失，避免数据库启动时日志回滚，确保应用处于一致性的启动状态。<br/>应用一致性快照以标签APPConsistent:True标识。                                               | 根据操作系统类型，实现方式如下：<br/>Windows：通过卷影复制服务VSS（Volume Shadow Copy Service）实现。<br/>Linux：通过执行自定义Shell脚本（需要您根据应用自行编写脚本）实现。应用一致性的效果，由您自己编写的脚本负责保证。 |
| 文件系统一致性快照 | 如果开启应用一致性功能，但不满足相关条件，系统将会为您创建文件系统一致性快照。<br/>文件系统一致性确保在快照创建时刻同步文件系统内存和磁盘信息，冻结文件系统写操作，使得文件系统处于一致性的状态。通过文件系统一致性快照，可以避免操作系统在重启后进行chkdsk或fsck等磁盘检查修复操作。<br/>文件系统一致性快照以标签FsConsistent:True标识。 | 根据操作系统类型，实现方式如下：<br/>Windows：如果无Windows操作系统上特定应用的VSS Writer参与时，默认创建的为文件系统一致性。<br/>Linux：如果无对应的应用脚本，默认创建的为文件系统一致性。                         |


看到阿里云的描述, 基本就知道了, 也是通过由用户自身的shell脚本内的行为来完成在创建快照前内存缓存持久化到硬盘, 确保硬盘此刻是完整的. 文件系统侧则通过默认的fsfreeze等通用方案来进行持久化.


#### rbd的用例

> If the volume contains a file system, the file system should be in an internally consistent state before a snapshot is taken. Snapshots taken without write quiescing could need an fsck pass before they are mounted again. To quiesce I/O you can use fsfreeze command. See the fsfreeze(8) man page for more details.

[Snapshots — Ceph Documentation](https://docs.ceph.com/en/quincy/rbd/rbd-snapshot/)

rbd的文档里已经提供了fsfreeze的方案.



#### ceph rbd内的实现

ceph在16版本librbd提供了`quiesce`能力, 主要描述的就是在创建快照前和后分别提供了一个hook接口.

然后暴露给上层用户, 可能就比较完善了. 

- rbd device --device-type nbd map --quiesce --quiesce-hook ${QUIESCE_HOOK} \
	- 看了下16版本有提供rbd quiesce 逻辑, 针对snap create时可以主动block rbd块的io, 并提供静默io前后的hook. 没找着有写测试/使用样例, 不过rbd-nbd那边有写用例, 可以提供脚本做前置/后置检查.

* PR: [librbd: API for quiesce callbacks · idryomov/ceph@269f4d2](https://github.com/idryomov/ceph/commit/269f4d233a17cffc774897660cf60f1b0acf077e)
* commit: [librbd: API for quiesce callbacks · idryomov/ceph@269f4d2](https://github.com/idryomov/ceph/commit/269f4d233a17cffc774897660cf60f1b0acf077e)

## 方案比较

参考[What is Application Consistent Backup and How to Achieve It](https://www.ubackup.com/enterprise-backup/application-consistent-backups.html) 可知几个方案的比较结果

| Operation                                              | timebased (inconsistent) | Crash-consistent | Application-consistent |
| ------------------------------------------------------ | ------------------------ | ---------------- | ---------------------- |
| Consistent point in time backup of files               | No                       | Yes              | Yes                    |
| Application consistency                                | No                       | yes              | Yes                    |
| Aware of in memory and pending I/O                     | No                       | no               | Yes                    |
| Requires no special steps for application data restore | No                       | yes              | Yes                    |                                                       |                          |                  |                        |



# Reference
1.  [通过Go SDK创建应用一致性快照](https://www.alibabacloud.com/help/zh/elastic-compute-service/latest/create-application-consistent-snapshots-by-using-ecs-sdk-for-go)
2. [云备份技术解析 （三）应用一致性备份\-云社区\-华为云](https://bbs.huaweicloud.com/blogs/100341)
3. [What is Application Consistent Backup and How to Achieve It](https://www.ubackup.com/enterprise-backup/application-consistent-backups.html)