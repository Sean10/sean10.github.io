---
title: libvirt部署note
date: 2018-07-04 22:59:29
updated:
tags: [虚拟化, libvirt]
categories: [专业]
---

# 准备新kvm 磁盘刻录 等等 技巧


<!--more-->


## 磁盘刻录命令:


```
# 查看插入的磁盘，查看是哪个盘，比如查看插入的ssd是sdn
fdisk -l

# 写入命令
dd if=/nas/nas/dd_img/iraid/rongheyitiji/C-IVMS256-20160523.img of=/dev/sdn bs=10M &
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

```




ABI定义了compiler该产生什么样的obj.
常见的一个问题是：
如果一个动态库是以ABI1.2的标准生成，但是系统使用的标准库是ABI1.3的标准生成，那么这个动态库将不能使用系统的标准库。
通常使用libgcc_s.so来标识gcc的ABI版本信息，使用 libstdc++.so来标识标准库的ABI版本信息。
	
什么是能够保证ABI的稳定的ABI模式，


libvirt的远程uri访问中那些命令不太理解，外部参数的使用也完全摸不着头脑

熟悉libvirt的接口，熟悉virsh命令

[root@localhost ~]# virsh help
分组的命令：

 Domain Management (help keyword 'domain'):
    attach-device                  从一个XML文件附加装置
    attach-disk                    附加磁盘设备
    attach-interface               获得网络界面
    autostart                      自动开始一个域
    blkdeviotune                   设定或者查询块设备 I/O 调节参数。
    blkiotune                      获取或者数值 blkio 参数
    blockcommit                    启动块提交操作。
    blockcopy                      启动块复制操作。
    blockjob                       管理活跃块操作
    blockpull                      使用其后端映像填充磁盘。
    blockresize                    创新定义域块设备大小
    change-media                   更改 CD 介质或者软盘驱动器
    console                        连接到客户会话
    cpu-baseline                   计算基线 CPU
    cpu-compare                    使用 XML 文件中描述的 CPU 与主机 CPU 进行对比
    cpu-stats                      显示域 cpu 统计数据
    create                         从一个 XML 文件创建一个域
    define                         从一个 XML 文件定义（但不开始）一个域
    desc                           显示或者设定域描述或者标题
    destroy                        销毁（停止）域
    detach-device                  从一个 XML 文件分离设备
    detach-disk                    分离磁盘设备
    detach-interface               分离网络界面
    domdisplay                     域显示连接 URI
    domfstrim                      在域挂载的文件系统中调用 fstrim。
    domhostname                    输出域主机名
    domid                          把一个域名或 UUID 转换为域 id
    domif-setlink                  设定虚拟接口的链接状态
    domiftune                      获取/设定虚拟接口参数
    domjobabort                    忽略活跃域任务
    domjobinfo                     域任务信息
    domname                        将域 id 或 UUID 转换为域名
    dompmsuspend                   使用电源管理功能挂起域
    dompmwakeup                    从 pmsuspended 状态唤醒域
    domuuid                        把一个域名或 id 转换为域 UUID
    domxml-from-native             将原始配置转换为域 XML
    domxml-to-native               将域 XML 转换为原始配置
    dump                           把一个域的内核 dump 到一个文件中以方便分析
    dumpxml                        XML 中的域信息
    edit                           编辑某个域的 XML 配置
    inject-nmi                     在虚拟机中输入 NMI
    send-key                       向虚拟机发送序列号
    send-process-signal            向进程发送信号
    lxc-enter-namespace            LXC 虚拟机进入名称空间
    managedsave                    管理域状态的保存
    managedsave-remove             删除域的管理保存
    maxvcpus                       连接 vcpu 最大值
    memtune                        获取或者数值内存参数
    migrate                        将域迁移到另一个主机中
    migrate-setmaxdowntime         设定最大可耐受故障时间
    migrate-compcache              获取/设定压缩缓存大小
    migrate-setspeed               设定迁移带宽的最大值
    migrate-getspeed               获取最长迁移带宽
    numatune                       获取或者数值 numa 参数
    qemu-attach                    QEMU 附加
    qemu-monitor-command           QEMU 监控程序命令
    qemu-agent-command             QEMU 虚拟机代理命令
    reboot                         重新启动一个域
    reset                          重新设定域
    restore                        从一个存在一个文件中的状态恢复一个域
    resume                         重新恢复一个域
    save                           把一个域的状态保存到一个文件
    save-image-define              为域的保存状态文件重新定义 XML
    save-image-dumpxml             在 XML 中保存状态域信息
    save-image-edit                为域保存状态文件编辑 XML
    schedinfo                      显示/设置日程安排变量
    screenshot                     提取当前域控制台快照并保存到文件中
    setmaxmem                      改变最大内存限制值
    setmem                         改变内存的分配
    setvcpus                       改变虚拟 CPU 的号
    shutdown                       关闭一个域
    start                          开始一个（以前定义的）非活跃的域
    suspend                        挂起一个域
    ttyconsole                     tty 控制台
    undefine                       取消定义一个域
    update-device                  从 XML 文件中关系设备
    vcpucount                      域 vcpu 计数
    vcpuinfo                       详细的域 vcpu 信息
    vcpupin                        控制或者查询域 vcpu 亲和性
    emulatorpin                    控制火车查询域模拟器亲和性
    vncdisplay                     vnc 显示

 Domain Monitoring (help keyword 'monitor'):
    domblkerror                    在块设备中显示错误
    domblkinfo                     域块设备大小信息
    domblklist                     列出所有域块
    domblkstat                     获得域设备块状态
    domcontrol                     域控制接口状态
    domif-getlink                  获取虚拟接口链接状态
    domiflist                      列出所有域虚拟接口
    domifstat                      获得域网络接口状态
    dominfo                        域信息
    dommemstat                     获取域的内存统计
    domstate                       域状态
    list                           列出域

 Host and Hypervisor (help keyword 'host'):
    capabilities                   性能
    freecell                       NUMA可用内存
    hostname                       打印管理程序主机名
    node-memory-tune               获取或者设定节点内存参数
    nodecpumap                     节点 cpu 映射
    nodecpustats                   输出节点的 cpu 状统计数据。
    nodeinfo                       节点信息
    nodememstats                   输出节点的内存状统计数据。
    nodesuspend                    在给定时间段挂起主机节点
    sysinfo                        输出 hypervisor sysinfo
    uri                            打印管理程序典型的URI
    version                        显示版本

 Interface (help keyword 'interface'):
    iface-begin                    生成当前接口设置快照，可在今后用于提交 (iface-commit) 或者恢复 (iface-rollback)
    iface-bridge                   生成桥接设备并为其附加一个现有网络设备
    iface-commit                   提交 iface-begin 后的更改并释放恢复点
    iface-define                   定义（但不启动）XML 文件中的物理主机接口
    iface-destroy                  删除物理主机接口（启用它请执行 "if-down"）
    iface-dumpxml                  XML 中的接口信息
    iface-edit                     为物理主机界面编辑 XML 配置
    iface-list                     物理主机接口列表
    iface-mac                      将接口名称转换为接口 MAC 地址
    iface-name                     将接口 MAC 地址转换为接口名称
    iface-rollback                 恢复到之前保存的使用 iface-begin 生成的更改
    iface-start                    启动物理主机接口（启用它请执行 "if-up"）
    iface-unbridge                 分离其辅助设备后取消定义桥接设备
    iface-undefine                 取消定义物理主机接口（从配置中删除）

 Network Filter (help keyword 'filter'):
    nwfilter-define                使用 XML 文件定义或者更新网络过滤器
    nwfilter-dumpxml               XML 中的网络过滤器信息
    nwfilter-edit                  为网络过滤器编辑 XML 配置
    nwfilter-list                  列出网络过滤器
    nwfilter-undefine              取消定义网络过滤器

 Networking (help keyword 'network'):
    net-autostart                  自动开始网络
    net-create                     从一个 XML 文件创建一个网络
    net-define                     从一个 XML 文件定义(但不开始)一个网络
    net-destroy                    销毁（停止）网络
    net-dumpxml                    XML 中的网络信息
    net-edit                       为网络编辑 XML 配置
    net-info                       网络信息
    net-list                       列出网络
    net-name                       把一个网络UUID 转换为网络名
    net-start                      开始一个(以前定义的)不活跃的网络
    net-undefine                   取消定义一个非活跃的网络
    net-update                     更新现有网络配置的部分
    net-uuid                       把一个网络名转换为网络UUID

 Node Device (help keyword 'nodedev'):
    nodedev-create                 根据节点中的 XML 文件定义生成设备
    nodedev-destroy                销毁（停止）节点中的设备
    nodedev-detach                 将节点设备与其设备驱动程序分离
    nodedev-dumpxml                XML 中的节点设备详情
    nodedev-list                   这台主机中中的枚举设备
    nodedev-reattach               重新将节点设备附加到他的设备驱动程序中
    nodedev-reset                  重置节点设备

 Secret (help keyword 'secret'):
    secret-define                  定义或者修改 XML 中的 secret
    secret-dumpxml                 XML 中的 secret 属性
    secret-get-value               secret 值输出
    secret-list                    列出 secret
    secret-set-value               设定 secret 值
    secret-undefine                取消定义 secret

 Snapshot (help keyword 'snapshot'):
    snapshot-create                使用 XML 生成快照
    snapshot-create-as             使用一组参数生成快照
    snapshot-current               获取或者设定当前快照
    snapshot-delete                删除域快照
    snapshot-dumpxml               为域快照转储 XML
    snapshot-edit                  编辑快照 XML
    snapshot-info                  快照信息
    snapshot-list                  为域列出快照
    snapshot-parent                获取快照的上级快照名称
    snapshot-revert                将域转换为快照

 Storage Pool (help keyword 'pool'):
    find-storage-pool-sources-as   找到潜在存储池源
    find-storage-pool-sources      发现潜在存储池源
    pool-autostart                 自动启动某个池
    pool-build                     建立池
    pool-create-as                 从一组变量中创建一个池
    pool-create                    从一个 XML 文件中创建一个池
    pool-define-as                 在一组变量中定义池
    pool-define                    在一个 XML 文件中定义（但不启动）一个池
    pool-delete                    删除池
    pool-destroy                   销毁（删除）池
    pool-dumpxml                   XML 中的池信息
    pool-edit                      为存储池编辑 XML 配置
    pool-info                      存储池信息
    pool-list                      列出池
    pool-name                      将池 UUID 转换为池名称
    pool-refresh                   刷新池
    pool-start                     启动一个（以前定义的）非活跃的池
    pool-undefine                  取消定义一个不活跃的池
    pool-uuid                      把一个池名称转换为池 UUID

 Storage Volume (help keyword 'volume'):
    vol-clone                      克隆卷。
    vol-create-as                  从一组变量中创建卷
    vol-create                     从一个 XML 文件创建一个卷
    vol-create-from                生成卷，使用另一个卷作为输入。
    vol-delete                     删除卷
    vol-download                   将卷内容下载到文件中
    vol-dumpxml                    XML 中的卷信息
    vol-info                       存储卷信息
    vol-key                        为给定密钥或者路径返回卷密钥
    vol-list                       列出卷
    vol-name                       为给定密钥或者路径返回卷名
    vol-path                       为给定密钥或者路径返回卷路径
    vol-pool                       为给定密钥或者路径返回存储池
    vol-resize                     创新定义卷大小
    vol-upload                     将文件内容上传到卷中
    vol-wipe                       擦除卷

 Virsh itself (help keyword 'virsh'):
    cd                             更改当前目录
    connect                        连接（重新连接）到 hypervisor
    echo                           echo 参数
    exit                           退出这个非交互式终端
    help                           打印帮助
    pwd                            输出当前目录
    quit                           退出这个非交互式终端


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
分布式文件系统，在OpenStack社区比较流行

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