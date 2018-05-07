---
title: VMWare远程控制踩坑
date: 2018-05-07 18:14:08
updated:
tags: [VMWare]
categories: [专业]
---

最近做毕设跑tensorflow占用资源挺大的，就想用旧电脑来跑，本来计划的是用旧电脑双系统里装好的ubuntu来跑，不过最近发现跑完保存模型和tensorboard，甚至在跑的时候就用了很多虚拟内存，至少mac上就达到了30G。这样的话，只给分配了20个G的ubuntu就相当不够用了。只好在windows上开虚拟机再配置一下远程了。

<!--more-->

安装完ubuntu，为了能让远程直接ssh到虚拟机里，需要对NAT进行设置，在虚拟网络设置中，比如我就把将主机的10086端口映射到虚拟机IP的22端口，这样就可以直接远程ssh了。

不过在使用VMWare共享文件夹给ubuntu的时候倒是遭遇了一些问题。

一开始为了减少桌面环境的资源占用，修改了grub的选项，默认启动命令行界面了。理论上这个部分对我之后安装vmware-tools是没有什么影响的。

vmware据说是安装了vmware tools以后就可以访问共享文件夹，但是在ubuntu里安装失败多次，成功一次以后依旧没能发现那个文件夹。

`sudo ./vmware-install.pl -d`之后后时完全不够的

需要按照2对tools打一个补丁，之后就能访问共享文件夹了。

``` bash
sudo apt-get install git
git clone https://github.com/rasa/vmware-tools-patches.git
cd vmware-tools-patches
sudo ./patched-open-vm-tools.sh
```

终于可以跑起来了，之后再装好jupyter打开对外访问，就OK啦~


# Reference
1. [vmhgfs-fuse替换mount](https://ask.csdn.net/questions/163546)
2. [vmware-tools-patches](https://askubuntu.com/questions/762755/no-vmhgfs-file-system-installed-to-use-use-shared-folder)
