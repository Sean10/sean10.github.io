---
title: libvirt部署note
date: 2018-07-04 22:59:29
updated:
tags: [虚拟化, libvirt]
categories: [专业]
---

# 准备新kvm 磁盘刻录 等等 技巧


<!--more-->


# 准备新kvm 磁盘刻录 等等 技巧

## 磁盘刻录命令:


```
# 查看插入的磁盘，查看是哪个盘，比如查看插入的ssd是sdn
fdisk -l

# 写入命令
dd if=/nas/nas/dd_img/iraid/rongheyij'jtiji/C-IVMS256-20160523.img of=/dev/sdn bs=10M &
# 查看dd进度
watch -n 5 pkill -USR1 ^dd$

# 判断进程是否结束
ps -elf | grep -w dd


iostat -X
# 查看CPU和IO状况


实际操作
dd if=/nas/nas/dd_img/iraid/c/iVMS-3000C-H24_CN_STD_9.2.0_V2.3.4-2_LOUYU24_180626.img of=/dev/sdm bs=10M &

ll -h G /nas/nas/dd_img/iraid/c/iVMS-3000C-H24_CN_STD_9.2.0_V2.3.4-2_LOUYU24_180626.img
查看大小

将硬盘刻成镜像
dd if=/dev/sdl of=/nas/nas/dd_img/iraid/c/jingweiyitiji/iVMS-3000C-H24_CN_STD_9.2.0_V2.3.4-2_JINGWEI24_180710.img bs=1M count=63488 &

```




ABI定义了compiler该产生什么样的obj.
常见的一个问题是：
如果一个动态库是以ABI1.2的标准生成，但是系统使用的标准库是ABI1.3的标准生成，那么这个动态库将不能使用系统的标准库。
通常使用libgcc_s.so来标识gcc的ABI版本信息，使用 libstdc++.so来标识标准库的ABI版本信息。
	
什么是能够保证ABI的稳定的ABI模式，


libvirt的远程uri访问中那些命令不太理解，外部参数的使用也完全摸不着头脑

熟悉libvirt的接口，熟悉virsh命令

[root@localhost ~]# virsh help


# 实践过程
```
virsh list --all 查看虚拟机
cat /var/log/libvirt/qemu/platform.log 查看虚拟机日志

虚拟机的名字就是这里说到的域



qemu-img构建一个img,然后用virsh 把这个磁盘挂载到xml文件启动的虚拟机中。

ll /mnt/temp 
platform.img
 virsh define 
 
 # 通过qemu-img创建磁盘文件
 qemu-img create -f qcow2 wc_test 30G
 
 # 停止域
  virsh destroy wc_test
 # 删除域
  virsh dumpxml node1 > node1.xml //导出node1虚拟机配置文件
 
 virsh undefine wc_test
 
 
 
  查看进程端口
 netstat -tunlp

 根据`ps aux | grep wc_test`查看进程id
 
 安装镜像过程中，出现寻找不到硬盘问题，需要再xml文件中载入驱动的cdrom，viostor/2k8r2/amd64,然后其中要注意，去除cdrom driver 总线那行，也可以根据实际填写
 
 xml文件 总线
 <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/> //域、总线、槽、功能号，slot值同一虚拟机上唯一
 
```

其中，出现bootmgr信息缺失问题时，时由于之前已经安装了部分内容进入了那个硬盘，导致引导程序可以在那个硬盘中读取到部分，但是由于未安装完成，必然是有信息缺失的，所以在虚拟机中按F12进入bios，依旧从系统盘启动重新安装就可以了。

设置的win2008密码是hik12345+

--------------

关于win10 Ctrl+Shift+F与IDEA冲突的问题，可以通过下述方法临时解决

先Ctrl+Shift+D ,再Ctrl+Shift+F
或者
先Ctrl+F，按住Ctrl，再按Shift+F。

# 7.5

新纪哥说，KVM在我们这块主要是用于IaaS,属于基础架构，docker组做的则是Paas上的应用，有点好奇。

IaaS做的是为用户提供所需要的虚拟机或存储等资源来装载应用。

## KVM和Docker 概念 区别

Docker，基于LXC资源隔离技术，

Hypervisor才是标准的虚拟化技术

![Container, VMs对比，LXC和cgroup区别](https://img-blog.csdn.net/20140722105009015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2hpY2hhb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


--------------

# 将IVMS-A200在系统登陆前启动

即将其添加为服务

使用windows serve 2003 tool kit可以实现

遭遇问题:
在系统登陆前后都无法连接上这个服务，需要手动打开应用登陆。

将服务设置为本地系统 ，勾选上允许与桌面进行交互后，登陆即可连接上，

这样来说，看到存在桌面UI的程序，需要勾选这个交互才会启动。

# 打升级包

10.192.44.5 telnet访问

wangchao44
密码: Wc4432

在这个系统上，vim backspace无法使用，因此修改vim配置文件

参考
https://www.cnblogs.com/shaojun/archive/2011/01/28/1946632.html



# 云架构上

docker 有了windows core，可以用来运行windows程序了，但是呢，出于kvm和docker是同级，所以这边要做的还是kvm的打包镜像模板。

参考资料

https://stefanscherer.github.io/is-there-a-windows-docker-image-for/

Paas理论上覆盖了中间件的范畴，会包含中间件的服务，并且具有更多的功能和更长的生命周期。(https://www.aliyun.com/zixun/content/1_1_15511.html)


# 企业级虚拟化产品
VMWare\HyperV\Xen\KVM\Docker 

## KVM
OpenStack是主流


CPU硬件虚拟化 支持？

`cat /proc/version`看了下zx哥让我试的服务器版本，发现除了是Red Hat之外，还有一个SMP名词，以前自己玩ubuntu的时候好像没看到过，是一个对称多处理技术(Symmetrical Multi-Processing)技术，可以运用多CPU共享内存、总线等。



# Ceph是什么呢
分布式文件系统（也是个对象存储生态环境），在OpenStack社区比较流行

同类的有
* GlusterFS，分布式NAS最多
* Lustre,HPC高性能计算
* HDFS, hadoop云计算的存储引擎
* moosefs
* IBM, GPFS老牌，强大，贵
* Facebook Haystack， 专有的图片存储系统的原型，闭源

ref.(https://www.zhihu.com/question/26993542)

Ceph的代码是通过C ,C++实现的，而为什么这里说到相比现代点的语言，可读性、结构良好、条理清晰的代码，C++难写出来呢？

这里说，其他同等级的高质量的代码，大多是用C写出来的。




这里有一张，超融合中各种存储的图

![](https://pic1.zhimg.com/88517e35f17409d52607caf21d535df0_r.jpg)

超融合的概念，就是计算、存储、网络全部虚拟化统一提供给客户

Ref.(https://www.zhihu.com/question/40977780?sort=created)


win10 全角与半角切换，shift + space

发现了hik的mirror站点，pip镜像可以从这个源下

http://mirrors.hikvision.com.cn/python-pypi/simple/

# 7.6 

Ceph用的Paxos算法，这个算法在分布式系统的书里看到过，基本各种分布式用的都是这个。

* 通过算来代替寻找节点
* 去中心化，不存在Master-Slave，不通过查表，而是通过计算得到各节点位置，（这个计算的方式有点意思）

在数据去重、压缩、同步、异步上存在一些缺陷

Ceph主要3个守护进程Ceph OSD\ Monitor \MDS

底层核心RADOS

CRUSH算法(Controlled Replication Under Scalable Hashing)

基于可扩展哈希的控制复制算法

在计算过程中，由Cluser Map（集群映射）\ Data Distribution Policy（数据分布规则）和给出的一个额随机整数x共同决定个数据对象最后的位置



原生云架构？

linux安装可以直接制定内核路径，直接加载

多CPU协作技术，SMP\MPP\NUMA架构，

`numactl --hardware`为什么这里也有2个CPU,那么在Centos那里可以看到的SMP又是什么？

KSM技术(Kernel SamePage Merging)，合并内存,为了内存超用，只适合测试环境，对性能要求不高的

内存气球技术

CPU绑定技术

NUMA自动平衡策略，可能并不均衡，需要手动协调

CPU热添加技术

KVM Nested技术

虚拟机，Virtio驱动下，比e1000驱动的网络发包性能好2倍

linux，一般virtio较好。windows上就根据系统版本，不同驱动性能都有可能出现

vhost-net（必须virtio）

MacVTap

网卡PCI-Passthrough

SR-IOV虚拟化技术(the Single ROot I/O Virtualization)，将PCI-E设备共享给虚拟机使用的标准

Open vSwitch 虚拟化软件交换机

树上说虚拟化网络管理，Open vSwitch非常稳定，但是hik没看到被安装， 那应该有其他的管理工具

qemu磁盘虚拟化，virtio类型，

裸盘可以用lvm来管理

在虚拟化中，为了充分发挥SSD性能，虚拟机磁盘应该用裸设备映射，而不是qcow2格式的文件系统(能用qcow2的能用lvm管理的裸机映射来替换？)

唔，内网中，https的网页就可以访问，http就不行

pxe 预启动环境

P2V， 物理机转虚拟机

KVM桌面虚拟化(Virtual Desktop Infrastructure, VDI)

远程访问协议RDP\PCoIP\Spice，公司内主要设置的VNC

## 几种常见开源文件系统在KVM中的应用


### DRBD

### GlusterFS 

### MooseFS

### Sheepdog


OpenStack 管理KVM

RabbitMQ Cluster

OpenNebula

VPN（Virtual Private Network)

自动化运维， Puppet



---------

缩容img

```
qemu-img convert -f qcow2 -O raw model_test model_test.raw
qemu-img resize model_test.raw -71G

-c 可以压缩
qemu-img convert -f qcow2 -c -O qcow2 model_test.qcow2 model_test_compress.qcow2

压缩完，磁盘空间只有3.6G了，非常不错

根据http://openwares.net/2012/04/26/shrink_kvm_qcow2_disk/ 缩容windows，需要用系统的分区工具进行缩容

之后才能执行resize 

尝试用lmt来调整分区看看，目前raw是7.8G， 但进入lmt分区管理，看到的是最少只能压缩到11.15G

现在尝试在无网络Centos中安装rpm包

rpm -ivh libguestfs-tools-c-1.36.10-6.el7.centos.x86_64.rpm --nodeps

找了好久配置本地源，yum list始终提示需要repodata/repomd.xml

运行最初上面那行代码加上强制--force，结果却提示没有--force这个路径，这版本是差了多少，连force都没有

还是失败了，好些so文件依赖错误。

用自己电脑下载了官网的tar.gz包。用百度云发过来，自己编译，结果还是包依赖缺失。

现在看到Centos自己的iso里可能就包含了这些包，用这个做源试试看

yumlist终于得到结果了

先卸载了之前装的失败的

```
 rpm -qa | grep libguestfs | rpm -e libguestfs-bash-completion-1.36.10-6.el7.centos.noarch \
libguestfs-1.36.10-6.el7.centos.x86_64 \
libguestfs-man-pages-uk-1.36.10-6.el7.centos.noarch \
libguestfs-xfs-1.36.10-6.el7.centos.x86_64 \
libguestfs-java-1.36.10-6.el7.centos.x86_64\
libguestfs-winsupport-7.2-2.el7.x86_64\
libguestfs-gobject-1.36.10-6.el7.centos.x86_64\
libguestfs-rescue-1.36.10-6.el7.centos.x86_64\
libguestfs-benchmarking-1.36.10-6.el7.centos.x86_64\
libguestfs-javadoc-1.36.10-6.el7.centos.noarch\
libguestfs-tools-1.36.10-6.el7.centos.noarch\
libguestfs-gobject-doc-1.36.10-6.el7.centos.noarch\
libguestfs-man-pages-ja-1.36.10-6.el7.centos.noarch\
libguestfs-tools-c-1.36.10-6.el7.centos.x86_64\
libguestfs-gfs2-1.36.10-6.el7.centos.x86_64\
libguestfs-rsync-1.36.10-6.el7.centos.x86_64\
libguestfs-java-devel-1.36.10-6.el7.centos.x86_64\
libguestfs-gobject-devel-1.36.10-6.el7.centos.x86_64\
libguestfs-devel-1.36.10-6.el7.centos.x86_64\
libguestfs-inspect-icons-1.36.10-6.el7.centos.noarch

```
.2 缩小分配的磁盘镜像空间上限(未完成)

首先，经过查询，找到guestfish和gparted两个linux下操作虚拟机文件分区的工具，但由于无网络下的安装依赖问题，暂时搁置，选择尝试已有工具。

使用老毛桃对win7进行分区压缩，提示对Vista及以上版本压缩后可能出现问题，实际上压缩完成后的确出现蓝屏，文件缺失问题。

使用win7自带磁盘管理，仅能从16G压缩至12G。

而该镜像使用qemu-img info查看，qcow2文件仅占用了3.6G，而raw也仅占用了7.8G。内外不符，尚未找到确定的原因，暂时猜测文件稀疏等问题。

根据《深度实践KVM》及大量博客推荐，开始尝试无网络下，安装guestfish，即libguestfs包。

1.2.1 尝试安装rpm包，从centos源站下载了该rpm包，实际安装时仅提示安装100%，实际操作时发现需要的工具命令并未安装完全。

1.2.2 想到昨天发现的Hik pypi的镜像源，试图寻找yum源，最终发现了mirrors.hikvision.com.cn中CentOS存在部分源。但在多次尝试修改源后发现，源信息不全，无法直接使用。该源各组似乎各自维护自己所需要的包，经检索，在OS组下找到了guestfish包，根据(https://centos.pkgs.org/7/centos-updates-x86_64/libguestfs-tools-c-1.36.10-6.el7_5.2.x86_64.rpm.html)里所要求的依赖包，手动从海康源中下载，计划手动安装。在手动安装了约10个左右后，发现依赖不全。并且出现了嵌套依赖的缺失。



海康源站mirrors.hikvision.com.cn

# 块设备

doc在这里，到时候查着改就行了
https://libvirt.org/formatdomain.html#elementsDevices



```

20180707

raw之所以也可能会类似qcow2那样，用du看到的空间也非常小的原因，是在于在其之下的ext4格式文件系统，因此qcow2在这种背景下并没有优势，只是其支持的功能复杂，因此具有价值


PaaS上的中间件开发，将各种接口进行适配，平台化？



>下面是被用于CI / CD创建脚本的主要框架和工具：
>
>自动构建管理: Apache Ant, Apache Maven, Gradle, …
>持续集成: Jenkins, Bamboo, …
>持续交付: Chef, Puppet, SaltStack, Ansible, …

其实docker组用的是docker swarm这个框架写的那个配置？

docker和KVM

docker还是更轻量级，不愧为微服务使用，而KVM还是太笨重了

什么叫所有的容器都必须使用同样的操作系统和内核，不也是可选的吗？

现在连windows的镜像也已经发布了呀？

的确，KVM需要hypervisor支持

不过在隔离性伤，相比KVM，容器公用一部分的运行库,怎么说

http://www.cnblogs.com/boyzgw/p/6807986.html



20180709

```
只读挂载
mount -o ro xxx.ios /mnt/cdrom
yum list | grep libvirt
手动remove旧的包
yum remove libvirt
yum install libvirt

似乎下述代码能修好libvirt和systemd的问题
# cp /usr/lib/systemd/system/libvirtd.service /etc/systemd/system/libvirtd.service 
# sed -i s/notify/simple/ /etc/systemd/system/libvirtd.service 
# systemctl daemon-reload
# systemctl reenable libvirtd     # If you enabled it before

来源https://www.reddit.com/r/archlinux/comments/490h9d/libvirtqemu_weird_timeout_issue/
```

新设备缺少了br0网桥配置，需要本机network里添加这个设置之后，在virsh net-define 这个配置
在virsh net-list里看到才比较正常

新机器最后使用命令时，依赖依旧有问题，libguestfs和supermin5上面异常退出，

偶然想到，看到的博客都没有提到修改qco2硬盘上限这个问题，由于qcow2格式是写时复制，想到会不会并不检测硬盘大小是否满足上限，因此找zx哥分了一个仅比实际占用硬盘大小大，远比设置的上限小的硬盘分区，实际测试，成功启动了win7

把一个带平台的系统刻到ssd上，拿回来压缩。

试着用在线平台，但是阻塞在scp，连了显示器，看到系统halted了

# win7 系统 刻成 iso

简单查询，似乎已经安装好的系统，没办法封装成iso，只能采用ghost备份还原的方法

看到了sysprep工具，似乎可以做到

20180710

`
tar zcvf xx.img.tar.gz xxx.img`

32G的要压缩多久呢

重新看了一下以前看的那些

```
dd if=/dev/zero of=/mytempfile
# that could take a some time
rm -f /mytempfile
# 这种清零的操作，可以对磁盘分区的连续性起到作用吗？

```

Shrink the Disk File
Shut down the VM.
Log into the Proxmox node and go to the VM's disk storage directory.
IMPORTANT: Create a backup of your existing VM disk file:
```
mv image.qcow2 image.qcow2_backup
```
Option #1: Shrink your disk without compression (better performance, larger disk size):
```
qemu-img convert -O qcow2 image.qcow2_backup image.qcow2
```
Option #2: Shrink your disk with compression (smaller disk size, takes longer to shrink, performance impact on slower systems):

```
qemu-img convert -O qcow2 -c image.qcow2_backup image.qcow2
```
Example: A 50GB disk file I shrank without compression to 46GB, but with compression to 25GB. Time to compress was almost twice as long as an uncompressed shrink.
Boot your VM and verify all is working.
When you verify all is well, it should be safe to either delete the backup of the original disk, or move it to an offline backup storage.

Ref.(https://pve.proxmox.com/wiki/Shrink_Qcow2_Disk_Files)

这里是否可以这么理解，通过清零后的磁盘，再使用-c指令，可以让镜像变得更小。


# 安装了平台以后的win7

磁盘管理的压缩卷没有反应，日志里说到是服务被禁用，在Services.msc中找到disk 磁盘清理，设置禁用为手动，可以压缩了

但是可压缩空间小于可分配空间，根据下面那个关闭系统保护、删除还原点、关闭分页，得到了

Ref.(https://jingyan.baidu.com/article/1709ad808a599f4634c4f082.html)

但是还是存在8个G的不可移动，进行一下磁盘碎片清理

唔，重启之后就启动不了，一直paused

# 封装win7到iso? 目前用sysprep可以弄成wim文件


# 一体机镜像

virsh start时，提示没有br0，

进入https://10.192.53.96:2004界面，按照顺序进行网络配置，绑定前两个，创建网桥

修改网络配置，/etc/sysconfig/network-scripts/ 

# 写个py代码，实现脚本功能

需求:
python xxx.py --kvm $kvm_name --source $source_block_dir

就会根据kvm_name获取到xml文件，然后在其中devices选项中写入block的选项，source 设置为$source_block_dir。

追加完后，重新define设备，根据是否已经存在，将libvirt提供的错误给返回。

似乎需要写成一个接口，会返回成功状态0

所以 之后 要将 这个过程型，再进行多种情况考虑，写成对象型

考虑到，目前使用这两个参数，必定是同时使用的，没有找到绑定这两个参数的方式，即一旦使用了kvm，自动提示必须要-source选项

# arg_parse使用

这里的optional arguments 和 positional arguments区别在哪里？

20180711

透传逻辑卷的脚本基础上按照流程途中的逻辑进行相应的修改，

1	块设备透传 在一键配置中加入 ，景象中虚拟机默认关机，开机之后做完一间配置之后，透传快设备，然后开启虚拟机

# 记录 过程

考虑到是作为接口开发的，因此，类内实例应该是保存输入的信息，如kvm名以及路径名等，然后通过外部调用那些操作的接口来执行

1. 在命令错误提示那里，连续输入的2个参数的名字需要修改，如-k kvm_name source，目前还是-k KVM KVM

可以通过设置2个选项，并且都是必须的实现

测试:
python kvmCmd.py 
输出:
usage: kvmCmd.py [-h] -k KVM -s SOURCE
kvmCmd.py: error: argument -k/--kvm is required

2. 当遇到异常时，raise状态 后 强制退出， 如sys.exit(1)，让echo $?能得到结果是1就行

raise并不能强制退出，只会把捕获的异常再次抛出

python里的raise就是常见的throw

综上，sys.exit()的退出比较优雅，调用后会引发SystemExit异常，可以捕获此异常做清理工作。os._exit()直接将python解释器退出，余下的语句不会执行。

一般情况下使用sys.exit()即可，一般在fork出来的子进程中使用os._exit()

 Ref.(https://blog.csdn.net/g863402758/article/details/53304480)

$# 传递到脚本的参数个数
$* 以一个单字符串显示所有向脚本传递的参数
$$ 脚本运行的ID号
$! 后台运行的最后一个进程的ID号
$@ 与$#相同，但是使用时加引号，并在引号中返回每个参数。
$- 显示shell使用的当前选项。
$? 显示最后命令的推出状况。0表示没有错误。 

try catch 在js上浏览器会自动进行优化，性能有所提升，不过目前没有涉及，暂时不管


# 查看块设备命令
```
lvs 
report-lun
report-lun info -i 0(id号)
路径在/dev/mapper，默认在名字
```

# arp 绑定 
8C-EC-4B-54-DA-66
ip 10.192.44.63
用管理员权限绑定了，`arp -s 10.192.44.63 8C-EC-4B-54-DA-66`



20180712 

# 刻制915、920卡

# 给45.55透传音频设备

似乎可以透传音频的pci，然后更改为spice连接即可

lspci 查看pci设备，但是这里看不到是ac97 model还是什么？


00:03.0 Audio device: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor HD Audio Controller (rev 06)
00:1b.0 Audio device: Intel Corporation 8 Series/C220 Series Chipset High Definition Audio Controller (rev 05)

Intel 和 AMD 都在它们的新一代处理器架构中提供对设备透传的支持（以及辅助管理程序的新指令）。Intel 将这种支持称为 Virtualization Technology for Directed I/O (VT-d)，而 AMD 称之为 I/O Memory Management Unit (IOMMU)。

唉，在内网里，装个驱动还得靠驱动人生

nvc透传的并不是原始设备，而是仿真的一个声卡。

本来似乎需要在宿主机里取消挂载声卡，用下面的脚本，但实际上更换了下model, vnc那里就检测到了。

```
[root@host016 home]# vi bind.sh
#!/bin/bash

modprobe vfio-pci

for dev in "$@"; do
        vendor=$(cat /sys/bus/pci/devices/$dev/vendor)
        device=$(cat /sys/bus/pci/devices/$dev/device)
        if [ -e /sys/bus/pci/devices/$dev/driver ]; then
                echo $dev > /sys/bus/pci/devices/$dev/driver/unbind
        fi
        echo $vendor $device > /sys/bus/pci/drivers/vfio-pci/new_id
done

```

```
透传物理机声卡到guest os

virsh nodedev-list --tree 查看相关信息
lspci -nn 查看设备

virsh nodedev-dumpxml pci_0000_00_1b_0（pci_0000_00_03_0）

然后再执行system-complete-reboot

宿主机 解挂pci设备

 cat /proc/cmdline |grep iommu

vim /boot/grub2/grub.cfg 

vim里查找 intel_iommu=pt， 在这里后面加上pci-stub.ids=8086:8c20

# 确认权限

TRUNK/HIKOS/hikos 
TRUNK/HIKOS/system
TRUNK/9.2.X-iRAID/HIKDISK-MANAGER有权限



TRUNK/9.2.X无权限
BRANCHES/9.2.X看得到文件夹，看不到内容
TRUNK/9.2.X-iRAID/MSGBUS

    




20180713

# 卸载声卡，需要采用其他方式，娟姐提供的仅限NVIDIA显卡的驱动的

卸载N卡驱动使用的是intel_iommu=on vfio-pci ids=8086:0c0c的内核参数


如果要透传pci设备，是用到了IOMMU的虚拟地址转换的，从而可以实现卸载和透传，使用cat /proc/cpuinfo可以看到AT-D的支持。

在CPU里看到AT-x的设置，但是AT-D呢

E3 1250 1230 支持 vt-d

rdp由于使用的是本机的物理设备如声卡来接收音频信息，因此对guest OS的声卡没有要求，同样可以做到传递音频信息，centos上也有支持windows rdp的应用，如xfreerdp

# ghost 替换gho, 得到iso

# 使用microsoft自带的， dism 和 imagesx和oscdimg进行打包https://answers.microsoft.com/en-us/windows/forum/windows_7-windows_install/where-is-oscdimgexe/b6ccd22e-b478-4222-b370-d5aaf021f575


目前打包遭遇问题，由于缺少virt io驱动，在进入如ghost和win pE时，无法识别出硬盘，导致无法对硬盘进行封包、备份操作。

而win pe时无法安装硬盘驱动。

20180714