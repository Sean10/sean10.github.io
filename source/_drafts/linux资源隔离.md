## chroot

据说类似mount namespace？

据说是显式控制进程的根目录？

既是一个cli，也是一个系统调用

要使用chroot，是需要把这个隔离的目录中需要启动的进程的可执行程序和依赖库单独复制进来的。


[root@localhost ~]# ll /proc/4703/ns/
total 0
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 net -> 'net:[4026531992]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 uts -> 'uts:[4026531838]'
[root@localhost ~]# ll /proc/4703/root
lrwxrwxrwx. 1 root root 0 Jul  2 08:50 /proc/4703/root -> /home/chroot_sean10

### 权限管理

容器由于主要使用的是内核的功能，因此主要关注uid和gid


### lxc

作为一个chroot的进阶产品，

* 运行时的lxc
* 管理容器和镜像文件的守护进程lxd
* 管理文件系统的lxfuse

暂不支持kubernetes

目前的LXC使用下列内核功能来控制进程：

🏄‍♂️ 内核命名空间（进程间通信、uts、mount、pid、network和user）

🏄‍♂️ AppArmor和SELinux配置

🏄‍♂️ Seccomp策略

🏄‍♂️ chroot（使用pivot_root）

🏄‍♂️ Kernel Capibilities

🏄‍♂️ 控制组（cgroups）

它在0.9版之前都是使用LXC技术，但在0.9版之后，已不再是唯一且默认的运行环境。

## bocker学习

## Containerd

也是CNCF的孵化项目，似乎兼容OCI（open container 项目）

## mesos

## cgroup 

## namespace

跟namespace相关的API

* clone
* setns
* unshare

unshare是使当前进程加入新的namespace

clone是创建一个新的子进程，然后让子进程加入新的namespace，而当前进程保持不变

pid namespace

在用ns进行隔离的时候，
mount -t proc proc /proc实际执行了什么操作呢
指定type类型是proc

net namespace应该就是关键的网络相关了。

## filesystem ? device mapper




## Reference
1. [Linux命名空间学习教程（一） UTS \- DockOne\.io](http://dockone.io/article/76)