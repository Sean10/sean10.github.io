
下载src.rpm方式

deletes the user from the utmp(5) file of current users and records the logout in the wtmp file

之前在看Systemd的时候就提到utmp和wtmp是毒瘤，我之前以为这两个只是用来记录命令记录的，看来比这要可怕的多啊？和登入有关？

getty的控制登入

pam 是用来控制用户会话的，包括用户的资源限制，也就是这里的 ulimit 的，还有变量控制、密码认证等

systemd 并不会用到 pam 的 env、limits 等模块，所以平时用来控制资源的 /etc/security/limits.conf，控制环境变量的 /etc/environment 等文件，对于 systemd 的服务并不起作用

不过 systemd 会通过 pam_systemd.so 模块将 PAM 中的会话注册到 user.slice 中并进行相应的操作，
还有就是将会话中的进程放置到当前会话的 CGroup 控制组中，实现会话资源的统一管理

## 启动控制 

BIOS runs self-check
BIOS loads the boot sector and executes it
Bootloader like grub or lilo is executed
Bootmenu is shown (optional)
Kernel is loaded
Initial RAM disk is loaded
Kernel is executed
Kernel executes init
init executes, depending on your distro, version and configuration

SysV init scripts or
systemd or
upstart
The sense of all these programs is to start services like

dbus that allows communication between applications so that one application can call functions from another running application. This is something usually not visible to users, e.g. an application calling the window manager to put its own window into focus
login that allows users to log in on the CTRL_ALT_F* terminals. Login's process as seen by ps -A will in case of systemd be systemd-logind (may again vary by distribution)
udev that has a lot of names, e.g. for me I find it with ps -A as systemd-udevd. It assigns e.g. the device handles in /dev/ to devices that you connect, e.g. a USB disk
cron that will execute commands based on a time table in /etc/crontab, and also has a "@reboot" feature to start commands on boot.


## 高可用层次结构

* 架构层（传递心跳、集群事务信息）
* 成员关系层（承上启下，监控底层心跳，随时重建集群状态，维持数据一致性）
* 资源调度层（集群资源管理器，实现资源的调度分配）（Cluster Resource Manager)
* 应用层 

## 容器技术

现在应该主流的是docker贡献作为基础的cri containerd吧？

docker应该只是作为OCI标准中的容器运行时在工作了把，只是一套生态旧的比较完整。（名字完整叫做dockershim)

docker是作为容器引擎工作的。

CNCF网站上可以看到开发进度

all.devstats.cncf.io


katacontainer可以类似qemu一样有一个hypervisorceng

容器内也有不少gRPC接口

## Reference
1. https://blog.packagecloud.io/eng/2015/04/20/working-with-source-rpms/