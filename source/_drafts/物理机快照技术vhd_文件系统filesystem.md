
# 文件系统

DAS 一般都是识别成设备，如U盘、硬盘等

NAS 一般都是识别为一个文件系统，实现的是文件级别的共享

SAN， 实现的是块级别的共享，但是需要专门的锁管理软件才能实现多主机并发访问

ftp是一个文件传输的协议， ftp协议中使用了类似http协议的响应吗，vsftpd的服务端软件支持用户认证

NFS文件系统，使用了RPC的机制，远程过程调用使得客户端可以调用服务端的函数。

samba其实是有人实现的window文件共享使用的CIFS协议。

### 分布式场景

这种是不是就是主要指ceph的存在了？



## XFS

原本以为只有看到的btrfs和libguestfs这两种可以做到快照，没想到简单搜了下，这个比较眼熟的xfs文件系统似乎也支持这种东西？

噢，原来是因为比较老旧，支持的特性不够多，然后有大佬想给他增加功能。阿里巴巴的大佬？

CoW文件系统中对于索引树的更新操作，使得共享、快照、子卷成为可能。快照其实只是继续保留一颗已被取代的索引树。

XFS属于纯数据CoW

## Btrfs


## overlayfs

关于物理机的系统还原操作，之前只知道可以做还原点，然后备份还原什么的。

差分磁盘技术

似乎从stackoverflow看上到说，其实一种loop device的东西其实就是支持这个的，不过仅支持用于Raw格式的磁盘上而已。

libguestfs就可以实现这种效果

这个回答说是等价的方式就是基于loop device的device-mappe snapshot.但说是这种方式并不能普遍实现

而对于实体机来说，


overlay filesystem的思路

像btrfs这种自带快照功能的。

或者像是devicde mapper这种块设备的快照机制（LVM，逻辑卷支持这种空能）

btrfs底层据说是个B树，

一些嵌入式设备的恢复出厂设置就是这样实现的。

## device mapper framework

linux的device mapper可以把一个块设备覆盖在另一个块设备智商，底层的快射别是一个压缩过的支持随机访问的制度文件系统（squashfs)注意不是所有的压缩格式都支持随机访问，有些压缩文件及时只想解压其中的一个文件，也得把很多其他文件都解压出来。上层的块设备是可读写的。

mapped device 
/dev/mapper/rootfs
|
v
Mapping Table
|       |
v       v
taget device(squashRootfs)      target device(cowfile.out)

一般来说，下层的只读文件系统和上层的可读写文件系统分别是一个文件（类似虚拟磁盘），在系统启动过程中挂载上来，成为一个文件系统，再chroot进去。

livecd是一个比较早的应用。

根目录就是squashfs，上层是Ramdisk，就是把内存当成磁盘来用。

有关ramdisk的技术细节，可以看看赵磊写的《写一个块设备驱动》，或者直接看内核里的Ramdisk源码


上述的查分磁盘方案是快设备层面的。每个查分出来的块设备都有自己的缓存

docker的device mapper 存储驱动程序就是利用这个框架的thin provisioning 和快照功能来管理Docker镜像和容器。

当前比较流行的是LVM2、EVMS、dmraid。

在CentOS中，docker默认使用的存储驱动是overlay。使用块设备比直接使用文件系统性能更好。

device mapper也分为用户控件部分和内核空间部分。

主要包含三个重要概念，映射设备（mapped device),映射表(map table)，目标设备（target device)

* 映射设备即对外提供的逻辑设备，映射设备向下寻找必须找到支撑的目标设备
* 映射表存储映射设备和目标设备的映射关系
* 目标设备可以是映射设备或者物理设备，如果目标设备是一块映射设备，则属于嵌套，理论上可以无限迭代下去

映射驱动在内核空间时插件，device mapper在内核中通过一个个模块化的Target driver插件实现对IO请求的过滤或者重新定向等工作，当前已经实现的插件包括软RAiD、加密、多路径、镜像、快照等，这体现了在Linux内核设计中策略和机制分离的原则。

### docker中的device mapper核心技术

* COW
* thin provisioning
* snapshot

docker如何使用thin provisioning上做到像UnionFS那样的分层敬仰呢？

这原来是来自于thin provisioning 的snapshot技术



### thin provisioning区别

### squashFS 

boot loader -> kernel -> initrd -> rootfs
机器商店时首先BIOS会启动，然后装载USB设备汇总的Boot Loader、kernel、initrd到内存中，由于这些文件大小综合小于10M，所以直接拷贝到内存中再执行没有任何问题。

在非嵌入式系统中，这部分文件通常储存在可直接读写的硬盘上，因此直接挂在到根目录后（例如mount /dev/sda1 /mnt)就可以进行读写操作

在嵌入式系统中，是一个压缩的文件系统，大小通常是好几百兆，解压后的大小都超过1G，如果直接mount到系统目录，那么系统目录是只读的，不可以进行写入。如果加载到内存中可以进行读写，但是这么大的文件直接解压到内存中对于嵌入式设备来说是不可接受的，因此需要一种可以不拷贝rootfs到内存中，又可以对最终的根文件系统进行读写的方法。

「为什么这里说嵌入式文件系统是压缩的？然后为什么是只读的？」

在嵌入式的环境下，内存和外存资源都需要节约使用，如果使用RAMDISK方式来使用文件系统，首先要把外村上的映像文件解压到内存中，构造器RAMDISK环境，才可以开始运行程序。

「为什么说RAMDISK方式，需要把映像文件解压到内存中？」

「实际上系统起来以后，究竟使用了多少系统资源来维护内核？」

[3]这篇文章讲的挺不错的。

#### CramFS




# vsan 快照 vsansparse

写 通过 ROW机制优化
读, 通过ROW写入的delta文件的缓存进行优化.

[VMware 替代专题｜VMware 与 SmartX 快照原理浅析与 I/O 性能对比](https://www.zhihu.com/tardis/zm/art/555072314?source_id=1003)

[Data Protection for VMware vSAN \| VMware](https://core.vmware.com/resource/data-protection-vmware-vsan#sec9839-sub1)

## Reference
1. [差分磁盘：从“恢复出厂设置”说起 \| Bojie Li](https://ring0.me/2014/02/how-factory-reset-works/)
2. [Overlay filesystem](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt)
3. [基于 SquashFS 构建 Linux 可读写文件系统](https://www.ibm.com/developerworks/cn/linux/1306_qinzl_squashfs/index.html)