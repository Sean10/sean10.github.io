---
title: KVM宿主机与虚拟机交互
subtitle: KVM交互
date: 2018-08-16 20:08:01
updated:
tags: [KVM, 虚拟化]
categories: [专业]
---

# 环境背景

使用libvirt管理kvm， 需要在不知道虚拟机ip的情况下，与虚拟机进行交互。

翻了翻，qemu有提供一个qemu-guest-agent的工具，在虚拟机内安装后就可以让宿主机获取虚拟机内的信息了。

<!--more-->


# 操作引导

根据redhat的指导[1], 在rhel7的系统的虚拟机xml里插入以下内容

```
<channel type='unix'>
   <target type='virtio' name='org.qemu.guest_agent.0'/>
</channel>
```
原理上，上面这部分为虚拟机添加了一个叫做`org.qemu.guest_agent.0`的串口，而在宿主机上则是在路径`/var/lib/libvirt/qemu/channel/target/<domain-6-kvm01>/org.qemu.guest_agent.0`创建了一个unix socket。

不过为什么要在虚拟机内建立的是串口呢？而不是和宿主机一样的unix socket呢？


虚拟机内安装这个包之后，启动这个服务就可以了。
```
yum install qemu-guest-agent
systemctl start qemu-guest-agent
systemctl enable qemu-guest-agent
```

这里需要注意，默认安装的qga的service文件中的串口名都是默认的guest_agent.0，如果在指定时修改了这个，手动指定了bind socket路径和target，那么虚拟机内的服务文件就需要替换这些默认名。并且，宿主机就监听不到我们所使用的端口了，需要手动使用下面的socat命令监听了。


然后宿主机，有3种
1. 执行`socat /var/lib/libvirt/qemu/channel/target/<domain-6-kvm01>/org.qemu.guest_agent.0  readline`，在这里交互式输入json格式的命令可以得到结果
2. virsh qemu-agent-command kvm_instance '{"execute":"guest-network-get-interfaces"}'
3. 也可以通过qemu-guest-agent提供的api执行命令，不过似乎要添加他的so包，没尝试

## 常用命令

```
{"execute": "guest-info"}

通过下述3个命令在虚拟机内写信息到文件内
{"execute":"guest-file-open", "arguments":{"path":"/home/testqga","mode":"w+"}}

{"execute":"guest-file-write","arguments":{"handle":0,"buf-b64":"aGVsbG8gd29ybGQhCg=="}}
# 注意这个handle是返回的句柄
{"execute":"guest-file-close", "arguments":{"handle":0}}


# 获取虚拟机网络信息ip
{"execute":"guest-network-get-interfaces"}
```

# 接口扩展

如果想要在虚拟机内执行没有默认提供的命令，就需要编辑qga源码，自己添加了。当时还以为是提供的一个类似配置文件的方式进行扩展，然后实际上是自己修改源码，扩展。

去github上`svn checkout https://github.com/qemu/qemu/trunk/qga`拉下这个文件夹，参考[4]操作编译就好了。



# 在虚拟机内执行命令

如果只是需要执行shell脚本的话，参照[3],似乎可以通过上面的方法写入到fsfreeze-hook.d中，然后fsfreeze-hook来执行。

```
virsh qemu-agent-command instance '{"execute":"guest-fsfreeze-freeze"}'
virsh qemu-agent-command instance '{"execute":"guest-fsfreeze-thaw"}'
```

不过也可以通过上面的network interface得到ip之后通过socket或者rpc等等自己定制接口来执行就是了。



# Reference
1. [How to enable QEMU guest agent in KVM](https://access.redhat.com/solutions/732773)
2. [qemu-guest-agent api](https://qemu.weilnetz.de/doc/qemu-ga-ref.html#API-Reference)
3. [openstack通过qemu-guest-agent在物理机上操作虚拟机](https://blog.csdn.net/cugb1004101218/article/details/49785859)
