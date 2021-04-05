---
title: VMWare远程控制tensorflow踩坑
date: 2018-05-07 18:14:08
updated:
tags: [VMWare, jupyter]
categories: [专业]
---

最近做毕设跑tensorflow占用资源挺大的，就想用旧电脑来跑，本来计划的是用旧电脑双系统里装好的ubuntu来跑，不过最近发现跑完保存模型和tensorboard，甚至在跑的时候就用了很多虚拟内存，至少mac上就达到了30G。这样的话，只给分配了20个G的ubuntu就相当不够用了。只好在windows上开虚拟机再配置一下远程了。

<!--more-->

安装完ubuntu，为了能让远程直接ssh到虚拟机里，需要对NAT进行设置，在虚拟网络设置中，比如我就把将主机的10086端口映射到虚拟机IP的22端口，这样就可以直接远程ssh了。

首先修改一下apt源到清华，不过为什么感觉反而这次更慢了呢？

不过在使用VMWare共享文件夹给ubuntu的时候倒是遭遇了一些问题。

# 1. legacy install

一开始为了减少桌面环境的资源占用，修改了grub的选项，默认启动命令行界面了。理论上这个部分对我之后安装vmware-tools是没有什么影响的，照常挂载cdrom,`mount -t auto /dev/cdrom /mnt/cdrom`，然后解压到根目录执行就好。

vmware据说是安装了vmware tools以后就可以访问共享文件夹，但是在ubuntu里安装失败多次，成功一次以后依旧没能发现那个文件夹。

`sudo ./vmware-install.pl -d`之后是完全不够的

需要按照2对tools打一个补丁，

然后使用更新的vmfuse替换mount，``之后就能访问共享文件夹了。

# 2. open-vm-tools install


``` bash
sudo apt-get install git
git clone https://github.com/rasa/vmware-tools-patches.git
cd vmware-tools-patches
sudo ./patched-open-vm-tools.sh
```

# Python环境重建

好久没在ubuntu下重新配置环境了


```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.6 get-pip.py
```

然后把pip源改到清华，OK。


终于可以跑起来了，之后再装好jupyter打开对外访问,win10防火墙开放一下修改的映射端口，就OK啦~

Jupyter远程跑的时候，频频会弹出窗口提示

`Notebook has changed since we opened it. Overwrite the changed file?`

一开始搜了下，最高票都提示说是bug，是自动modify_checkpoint间隙太短，在5.5版本里添加了修改这个时间的配置文件。但我之前单机使用是没有问题的，新装的版本和单机用的notebook版本是相同的，所以变量是控制了的，之后发现有人多次提到主要出现原因是因为同时多个窗口在访问这个页面，那么，细想一下，在启动notebook时默认会自动打开一个网页，text启动的ubuntu是不是也存在了这个情况(目前猜想是由于毕竟是desktop版本的ubuntu，所以还是存在自动打开网页进程的能力)，那么就去修改一下配置文件，默认不打开网页，解决了一个小问题。

但是依旧会定时弹出这个网页……暂时先集中到内容上，之后在处理吧

# jupyter kernel重建

在另一台电脑上重新安装jupyter，发现由于系统内存在多个python3版本，执行的python3内核并不是我所要的。

首先修改ipython

解决方式:
```
which ipython
which python3
```

将ipython文件中启动方式里的python路径改为你要用的那个版本的python即可，像我就是修改为python3。

再使用`jupyter kernelspec list`得到所需要修改kernel.json路径，将其中的python路径改为自己所要用的即可。

不过这里显然存在一个问题，这里我list出来的是我在virtualenvwrapper里配置的环境里装的jupyter的kernel的路径，虽然这里改了以后能够运作了，但这应该是存在问题的。

看[6]中提到的，`ipython kernel install`可以按自己想要的安装内核，但是我安装后还是没能出现在上面的list中，稍后再看吧。


# Reference
1. [vmhgfs-fuse替换mount](https://ask.csdn.net/questions/163546)
2. [vmware-tools-patches](https://askubuntu.com/questions/762755/no-vmhgfs-file-system-installed-to-use-use-shared-folder)
3. [pip install doc](https://pip.pypa.io/en/stable/installing/)
4. [tsinghua pypi](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)
5. [Anaconda & ipython路径问题 & jupyter notebook 启动核心问题（使用方法&不常见的问题）](https://zhuanlan.zhihu.com/p/31074090)
6. [jupyter与python的内核](https://letianfeng.github.io/python/2018/04/24/jupyter_and_python_kernel.html)