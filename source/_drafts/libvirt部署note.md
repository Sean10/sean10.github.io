---
title: libvirt部署note
date: 2018-07-04 22:59:29
updated:
tags: [虚拟化, libvirt]
categories: [专业]
---

# 准备新kvm 磁盘刻录 等等 技巧


<!--more-->

# debug日志打开
``` conf
#/etc/libvirt/libvirtd.conf
log_filters="1:libvirt 1:util 1:qemu"
log_outputs="1:file:/var/log/libvirt/libvirtd.log"
```
## lxc 的debug日志


# 准备新kvm 磁盘刻录 等等 技巧

## 磁盘刻录命令:



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
```
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

http://www.cnblogs.com/oftenlin/p/4325023.html

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

使用modify-nic修改

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

ghost找不到硬盘分区，存在可能是SATA RAID/AHCI Mode 的原因

IDE 比较老

AHCI比较新

NVMe最新



Serial ATA Advanced Host Controller Interface

可能winxp时代的win pe版本，不支持AHCI模式的硬盘，因此需要新版本win pe

20180714

回忆:

DWORD
Windows下经常用来保存地址(或者存放指针) 
其他unsigned long可以用的地方，它也是可以用的

win api 

CreateFile

VC支持ascii和unicode两种字符类型，用_T可以保证从ascii编码类型转换到unicode编码类型的时候，程序不需要修改。


# 寻找iSCSI设备描述符

wmic diskdrive list

# 减缓windows lazy write 频率
这里说，是由cache manager管理，手动调用的函数cache flush 





下面这段是下面的链接中相关的部分。

Cache flush frequency
If the Cache Manager does not try to write modified file data back to a file, and free memory becomes scarce, the memory manager's modified writer thread writes the unwritten data back to a file. The system does not rely on the memory manager to flush file data back to the disk. Instead, the Cache Manager tries to write the data back to nonvolatile storage in a timely manner by using the "lazy writing" process. As programs modify file data, the Cache Manager keeps track of how much data is modified, or "dirty." The Cache Manager writes back one-eighth of the cache's modified data to disk every second.

主要讲的就是1/8每秒的flush过程。


https://support.microsoft.com/en-us/help/837331/about-cache-manager-in-windows-server-2003  

这个链接就是完整的那篇file-caching，依旧只提到了1/8的flush 和 手动flush的那个 FlushFileBuffers函数。

https://docs.microsoft.com/en-us/windows/desktop/fileio/file-caching


下面这篇里提到cache manager的lazy write算法逐渐优化，性能理论上是越来越好。

https://serverfault.com/questions/809825/windows-server-2012-write-caching




https://docs.microsoft.com/en-us/windows/desktop/api/FileAPI/nf-fileapi-flushfilebuffers

https://serverfault.com/questions/424581/should-i-disable-write-caching-on-my-windows-2008-vm

20180716

# 文件夹同步，下载了Always Sync这个工具来用，还可以吧，可以同步我的文件夹和共享硬盘上的了。

# django学习

manage.py与django-admin.py都使用了django.core.management包的方法.execute_manager()方法专门提供给manage.py使用,execute_from_command_line()专门

提供给django-admin.py使用.(地球人都看的出来,⊙﹏⊙b).

这俩个命令的主要差别在于execute_manager()多执行了一个setup_environ(settings_mod)方法.弄清这个方法的作用,就知道manage.py与django-admin.py的主要区别了.

http://www.cnblogs.com/pythoner/archive/2011/07/30/2121599.html

启动的命令主要是根据下面这里传入的参数来启动

INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 'django.contrib.admin',
    'webvirtmgr',
    'servers',
    'instance',
    'create',
    'gunicorn',
)

basestring是str和unicode的超类。

查看内建函数：

class basestring(object)

class str(basestring)

class unicode(basestring)

所以str和unicode是不同的，判断时需要注意字符串类型。


---

类变量 定义时没有使用self,在类内的方法里使用时可以使用Self.xxx调用它，对可以，因为在类内，self等同于类名嘛。

django在访问的panel似乎使用了这几个中间件
        self._view_middleware = []
        self._template_response_middleware = []
        self._response_middleware = []
        self._exception_middleware = []
        
        request_middleware = []
        
        
setting设置了这几个中间件


MIDDLEWARE_CLASSES = (
    'django.middleware.common.CommonMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    # Uncomment the next line for simple clickjacking protection:
    # 'django.middleware.clickjacking.XFrameOptionsMiddleware',
)


 request_middleware
 这个在判断hasattr之后，才定义成类内私有变量
 
 _prefixes = local()
 
设置每个线程使用不同的_prefixes 和 _urlconfs

 
 -----
 
 
 # bing搜索引擎设置
 
 https://cn.bing.com/search?ensearch=1&q=django+get_script_name+%E4%BD%BF%E7%94%A8&go=%E6%8F%90%E4%BA%A4&qs=n&sp=-1&ghc=1&pq=pycharm+%E5%AF%BC%E5%85%A5+%E5%B7%A5%E7%A8%8B&sc=0-13&sk=&cvid=2C9D10250B6A4D4098B0D8093A11E434&first=11&FORM=PORE&filt=custom
 
 https://cn.bing.com/search?ensearch=1&q=%s&go=%E6%8F%90%E4%BA%A4&qs=n&form=QBLHCN&sp=-1&ghc=1&pq=pycharm+%E5%AF%BC%E5%85%A5+%E5%B7%A5%E7%A8%8B&sc=0-13&sk=&cvid=2C9D10250B6A4D4098B0D8093A11E434
 
 
 修改为:https://cn.bing.com/search?ensearch=1&q=%s&go=%E6%8F%90%E4%BA%A4&qs=n&sp=-1&ghc=1&pq=pycharm+%E5%AF%BC%E5%85%A5+%E5%B7%A5%E7%A8%8B&sc=0-13&sk=&cvid=2C9D10250B6A4D4098B0D8093A11E434&first=11&FORM=PORE&filt=custom
 
 https://cn.bing.com/search?ensearch=1&q=%s&go=%E6%8F%90%E4%BA%A4&filt=custom
 
 # 打基线升级包

 1，下载最新的svn代码
2，找到C文件进行编译（10.192.53.20），并放置到U:\bin\rdbios\smi\bn_cli下
3，将svn中smi下的代码导出到10.192.44.5（U:\bin\rdbios\smi）
4，更改Readme.cof，基线包最后添加smi的svn版本号用于排查问题（Nxxxx）
5，更改版本信息（U:\bin\rdbios\smi\config\system_version.conf）
	StorOS Version:9.2.0
	Update Version:8
	StorOS Flag:Leite
	Extend Vesion:
	Date:180716
6,在44.5的/home/lizhixuan/bin目录下执行lizhixuan@localhost:~/bin> product_bios_920 


20180717

# 试着安装昨天下载成功的windows 7 AIK工具。

提示无法安装，发现windows installer 服务没有打开，启动后遭遇蓝屏，之后多次重启，同样，怀疑之前安装了什么导致的蓝屏。

尝试卸载ac97驱动，依旧蓝屏，存在问题

# 透传NVIDIA显卡

## lspci | grep NVI 得到信息

## virsh nodedump

<device>
  <name>pci_0000_04_00_0</name>
  <path>/sys/devices/pci0000:00/0000:00:03.0/0000:04:00.0</path>
  <parent>pci_0000_00_03_0</parent>
  <capability type='pci'>
    <domain>0</domain>
    <bus>4</bus>
    <slot>0</slot>
    <function>0</function>
    <product id='0x1bb3' />
    <vendor id='0x10de'>NVIDIA Corporation</vendor>
    <numa node='0'/>
    <pci-express>
      <link validity='cap' port='0' speed='8' width='16'/>
      <link validity='sta' speed='8' width='16'/>
    </pci-express>
  </capability>
</device>

    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
         <address domain='0x000' bus='0x04' slot='0x00' function='0x0' />
      </source>
   </hostdev>


传入进去了总之，总之现在虚拟机里，lspci是可以看到了，就是宿主机里lspci依旧也可以看到

# SVN图标不显示

windows提供的15个自选Icon ，按照安装顺序先来后到，被OneDrive等占据了就不行了

http://www.cnblogs.com/likebeta/archive/2012/07/01/2571731.html


# 查找如何 透传 串口设备

目前直接修改libvirt xml 得到的记过是模拟了几个串口设备，然后在dev/pts/中可以检测到

https://blog.csdn.net/isclouder/article/details/80774592


查看串口是否可用，可以对串口发送数据比如对com1口，echo lyjie126 > /dev/ttyS0
查看串口名称使用 ls -l /dev/ttyS* 一般情况下串口的名称全部在dev下面，如果你没有外插串口卡的话默认是dev下的ttyS* ,一般ttyS0对应com1，ttyS1对应com2，当然也不一定是必然的；
查看串口驱动：cat /proc/tty/driver/serial
查看串口设备：dmesg | grep ttyS*

# 修改 静态ip 网络
修改/etc/sysconfig/network-s/ifcfg-eth0



-----------

# 吃饭发票保留，准备开

各位同事，请注意吃饭时候索取小票或发票，用于晚餐补的报销。
新同事尤其注意，请注意累计饭店小票（金域周边的）和发票（较远地方，平时请假或聚会吃饭索取），发票内容稍后邮件中发送。
名称：杭州海康威视数字技术股份有限公司
 
纳税人识别号：91330000733796106P 
 
开户银行：中国工商银行杭州广电支行
 
银行账号：1202051309900002966
 
地址：杭州市滨江区阡陌路555号
 
电话:0571-88075998 

20180718

# 修复渣土一体机脚本检测块设备bug，并调整脚本异常处理，去除调试信息

# 异常 ，是否应仅在对应实例内处理

转绝对路径 存在 软连接问题

os.path.realpath

# 传递串口

直接传递单个ttyS*，在虚拟机里串口调试里收不到


1.串行端口终端（/dev/ttySn）
串行端口终端（Serial Port Terminal）是使用计算机串行端口连接的终端设备。计算机把每个串行端口都看作是一个字符设备。有段时间这些串行端口设备通常被称为终端设备，因为那时它的最大用途就是用来连接终端。这些串行端口所对应的设备名称是/dev/tts/0（或/dev/ttyS0）、/dev/tts/1（或/dev/ttyS1）等，设备号分别是（4,0?）、（4,1?）等，分别对应于DOS系统下的COM1、COM2等。若要向一个端口发送数据，可以在命令行上把标准输出重定向到这些特殊文件名上即可。例如，在命令行提示符下键入：echo test > /dev/ttyS1会把单词”test”发送到连接在ttyS1（COM2）端口的设备上。
2.伪终端（/dev/pty/）

伪终端（Pseudo Terminal）是成对的逻辑终端设备，例如/dev/ptyp3和/dev/ttyp3（或着在设备文件系统中分别是/dev/pty/m3和/dev/pty/s3）。它们与实际物理设备并不直接相关。如果一个程序把ttyp3看作是一个串行端口设备，则它对该端口的读/写操作会反映在该逻辑终端设备对的另一个上面（ttyp3）。而ttyp3则是另一个程序用于读写操作的逻辑设备。这样，两个程序就可以通过这种逻辑设备进行互相交流，而其中一个使用ttyp3的程序则认为自己正在与一个串行端口进行通信。这很象是逻辑设备对之间的管道操作。
对于ttyp3（s3），任何设计成使用一个串行端口设备的程序都可以使用该逻辑设备。但对于使用ptyp3的程序，则需要专门设计来使用ptyp3（m3）逻辑设备。
例如，如果某人在网上使用telnet程序连接到你的计算机上，则telnet程序就可能会开始连接到设备ptyp2（m2）上（一个伪终端端口上）。此时一个getty程序就应该运行在对应的ttyp2（s2）端口上。当telnet从远端获取了一个字符时，该字符就会通过m2、s2传递给getty程序，而getty程序就会通过s2、m2和telnet程序往网络上返回”login:”字符串信息。这样，登录程序与telnet程序就通过“伪终端”进行通信。通过使用适当的软件，就可以把两个甚至多个伪终端设备连接到同一个物理串行端口上。
在使用设备文件系统（device filesystem）之前，为了得到大量的伪终端设备特殊文件，HP-UX AIX等使用了比较复杂的文件名命名方式。
3.控制终端（/dev/tty）？
如果当前进程有控制终端（Controlling Terminal）的话，那么/dev/tty就是当前进程的控制终端的设备特殊文件。可以使用命令”ps –ax”来查看进程与哪个控制终端相连。对于你登录的shell，/dev/tty就是你使用的终端，设备号是（5,0）。使用命令”tty”可以查看它具体对应哪个实际终端设备。/dev/tty有些类似于到实际所使用终端设备的一个联接。
4.虚拟控制台（/dev/ttyn）, /dev/console
在UNIX系统中，计算机显示器通常被称为控制台终端（Console）。它仿真了类型为Linux的一种终端（TERM=Linux），并且有一些设备特殊文件与之相关联：tty0、tty1、tty2等。当你在控制台上登录时，使用的是tty1。使用Alt [F1—F6]组合键时，我们就可以切换到tty2、tty3等上面去。tty1 –tty6等称为虚拟终端，而tty0则是当前所使用虚拟终端的一个别名，系统所产生的信息会发送到该终端上。因此不管当前正在使用哪个虚拟终端，系统信息都会发送到控制台终端上。
你可以登录到不同的虚拟终端上去，因而可以让系统同时有几个不同的会话期存在。只有系统或超级用户root可以向/dev/tty0进行写操作。tty# 是系统视频监视器上全屏显示的终端，用于在不能使用framebuffer设备的机器上存取视频卡，而/dev/console由系统内核管理，接收系统消息。
    https://blog.csdn.net/bianhonglei/article/details/4932583
    
# passthrough script python

正则表达式，[\s\S]才是通用的

试了下，明明在在线测试里 `.*NV[\s\S]*`是能够匹配到我要的结果的

pattern.search可以得到结果，但是pattern.match就得不到结果。

match只匹配从该行初始就匹配到的结果。而search可以从任意位置匹配到

“.” 在一般情况下匹配除 “\n” 以外的任何字符，但在“[]”内只匹配自身，所以“[.\n]”这样的写法无法匹配任意字符；如果将使用RegexOptions.Singleline选项，“.”代表任意字符，包括“\n”，所以有上面第2种写法。

分割方法

``` python
import re
DATA = "Hey, you - what are you doing here! welcome to jb51?"
print re.findall(r"[\w']+", DATA
```
# code review 概念

LOC(line of code )

python 判断行数

`find . -name "*.py" -type f -exec grep . {} \; | wc -l`

意识到为什么显示LOC少了，因为上一个版本是志轩哥提交的

# 20180719

# 跟zx哥集成测试渣土一体机

# 双网域配置，双网卡

虚拟机kvm里的网络是绑定在了宿主机的br0网桥上的

```
# 查看网卡接口
virsh iface-list


# 查看route
route

结果正常

# KVM虚拟网络管理命令(virtual network)：
virsh命令参数	功能	用法举例
net-autostart	配置一个虚拟网络开机自启(--disable可以关闭)	virsh net-autostart br0
net-create	通过一个xml文件创建一个虚拟网络	virsh net-create ./virbr1.xml
net-define	通过xml文件定义一个虚拟网络，仅定义不实例化	virsh net-define ./virbr1.xml
net-destory	停止由其名称(uuid)指定的虚拟网络，立即生效	virsh net-destroy br0
net-dumpxml	以xml文件的形式输出一个虚拟网络的配置信息	virsh net-dumpxml br0
net-edit	编辑一个虚拟网络的配置文件(修改虚拟网络配置)	virsh net-edit br0
net-info	返回要查看的虚拟网络的基本信息	virsh net-info default
net-list	查看当前的虚拟网络信息(可以带参数)	virsh net-list --all
net-name	 	 
net-start	开始一个不活跃的虚拟网络	virsh net-start br0
net-undefine	将一个不活跃的虚拟网络取消定义	virsh net-undefine br0
net-uuid	 	 
net-update	 	 
```

结果，中圣哥轻松解决，说eth1的网关不能写，同时只能有一个网关，不然新写入的网关会被作为dns网关处理，导致前一个写的网关无法生效。

# 继续写透传GPU脚本

这两个linux16执行的有区别吗
``` shell

linux16 /boot/vmlinuz-3.10.0-123.el7.x86_64 root=UUID=3db86cd1-5593-4211-896b-31d25a4b88e6  rootfstype=ext4  ro vconsole.keymap=us crashkernel=256M  vconsole.font=latarcyrheb-sun16 LANG=en_US.utf8 biosdevname=0 net.ifnames=0 console=ttyS0,115200 console=tty0 acpi=on libata.fua=1 aerdriver.forceload=y nmi_watchdog=1 8250.nr_uarts=8 intel_iommu=on iommu=pt pci-stub.ids=10de:1bb3 

linux16 /boot/vmlinuz-3.10.0-123.el7.x86_64-002 root=UUID=3db86cd1-5593-4211-896b-31d25a4b88e6  rootfstype=ext4  ro vconsole.keymap=us crashkernel=256M  vconsole.font=latarcyrheb-sun16 LANG=en_US.utf8 biosdevname=0 net.ifnames=0 console=ttyS0,115200 console=tty0 acpi=on libata.fua=1 aerdriver.forceload=y nmi_watchdog=1 8250.nr_uarts=8
```

镜像上有区别，第二个不是我们的系统，不用管


20180720

# 透传 存在 新旧版本
vfio-pci是pci-stub的新版本

https://ycnrg.org/vga-passthrough-with-ovmf-vfio/

使用libvirt透传PCI/PCI-E设备时需要知道要透传设备的总线地址，以在域定义中指定要透传的设备。一般落实到QEMU中有这些为透传准备的设备模型，包括pci-assgn、vfio-pci、vfio-vga等

USB\外设重定向、透传

QEMU中串口与并口设备一般都可进行透传与重定向操作，其中透传比较简单，只需要将本地串口/并口的设备节点当做设备后端即可，形如“-parallel /dev/lp0”，而重定向的思路与USB over TCP/IP较为类似。

重定向时我们需要使用工具将客户端串口/并口设备的输入/输出暴露到客户端的网络端口，hypervisor再将客户端的IP地址与TCP端口作为虚拟机的串口/并口设备后端参数进行连接，如图9-2所示。

Linux客户端中可以使用ser2net作为串口/并口的服务端，Windows客户端中也有对应的实现，笔者以Linux中的ser2net为例，介绍QEMU中的串口与并口重定向。


http://www.sohu.com/a/141299947_610730

其实只要很简单的一步，就能够实现非root权限就能访问/dev/ttyS*设备了。

　　首先我们来看看为什么普通账户会没有权限访问ttyS设备了：

ls -l /dev/ttyS0
crw-rw---- 1 root dialout 4, 64  8月 30 21:53 /dev/ttyS0
　　从上面的输出，我们很容易看出来，ttyS设备的用户主是root，而所属的组是dialout，并且owner和group都是有相同的rw权限的，但others是没有任何权限的。

我们用root登陆，所以倒是没有任何影响

用刚刻的53.25的设备，需要挂载hda6到/mnt/kvm上，这样才能有img和iso文件，然后，需要在2004页面配置网桥br0，之后才能启动kvm01

Ctrl+]退出virsh console kvm01 --devname serial0

tty命令可以得到当前对应的/dev/pts/1

`echo "sd" > /dev/pts/1`
和 `echo "sd" > /dev/tty`一样效果

/dev/pts是远程登陆(telnet,ssh等)后创建的控制台设备文件所在的目录。由于可能有好几千个用户登陆，所以/dev/pts其实是动态生成的，不象其他设备文件是构建系统时就已经产生的硬盘节点(如果未使用devfs)

https://blog.csdn.net/suiyuan19840208/article/details/7234722

getty可以实现从串口登陆控制台，需要修改guest grub。

除了console连接， 还可以virio连接，不过这样就像ssh直接连接了，而不是serial0那样连接上也只是互相发送消息，有没有可以host echo，然后guest cat的呢

https://blog.csdn.net/isclouder/article/details/80774592

## parallel port 是什么呢

下面这篇里说，parallel这个功能在新版本里是不支持的，只对于一些老版本支持，不知道我们这个属不属于

https://bugzilla.redhat.com/show_bug.cgi?id=1077124

## qemu-guest-agent 这个似乎可以，host serial guest走socket

暂时没有尝试

## VMWare有3种添加串口的模式，有一种支持直接添加物理串口给虚拟机，kvm上这种技术在哪里呢？

 `/usr/libexec/qemu-kvm --help | grep -e 'chardev\|device'`
 qemu-kvm里似乎是有支持的，那么libvirt在哪里呢

# libvirt部署好文章

https://xstarcd.github.io/wiki/Cloud/ubuntu_kvm_virtualization_solution.html

20180723

# 
# python how to init a list of dict
最后先试着用了a dict of list

# 是否有同时对两个列表进行排序，匹配是否在对面中存在


20180724

# 修复透传块设备bug

# 自己制作rpm包
# 学习spec配置文件信息设置
# 让这个rpm能够像yum那样具有解决依赖的能力， 如打ssl的升级包

python字符转ASCII，需要用ord()和 chr()

# rpm 打包 原理

# rsyslog学习

# win10 下载vs2017 offline包最后遭遇各种问题

# 打升级包
```
[/b_iscsi/bin/ 00777]这里实在smi目录下对应的文件在的目录

[jobs_before_update]这里写打之前要做的操作，比如杀进程什么的

[jobs_after_update] 这里写打完以后要做的操作。注意命令中间不能有换行
chmod 0644 /etc/systemd/journald.conf
chmod 0644 /etc/systemd/logind.conf

[jobs_before_reverse]

[jobs_after_reverse]


```

```
那个readme.conf中，global_version改成915或者920就好了
```


# RHEL 似乎是个做过这个方面的大佬

https://www.cnblogs.com/itxdm/category/827203.html

chmod 的0644/2644， 似乎第一位是setuid这类的东西

20180725 

 linux strace命令 监控 执行过程

 strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间

 /run is the "early bird" equivalent to /var/run, in that it's meant for system daemons that start very early on (e.g. systemd and udev) to store temporary runtime files like PID files and communication socket endpoints, while /var/run would be used by late-starting daemons (e.g. sshd and Apache).