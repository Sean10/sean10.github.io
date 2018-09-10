---
title: rsyslog & journald日志系统结构
date: 2018-08-02 01:45:01
updated:
tags: [linux, rsyslog, systemd]
categories: [专业]
---

之前在公司服务器上验证了好多次流程图，结果是被改过代码了可能，运行结果与原生CentOS镜像结果不同。被迫混淆了好久。

<!--more-->




# 基本背景

## 实验环境
* Centos 7.4
* rsyslog 8.24
* systemd环境

# rsyslog与systemd-journald日志流向
<!--![](http://10.192.44.64:5000/_uploads/photos/rsyslog_journald.png)-->

目前来看，lsof只能查看该进程监听的socket,不显示它发送的socket。通过strace追踪，RecMsg会显示发送方的进程pid，从那里可以看到是哪个进程发送到自己监听的socket的信息。

``` bash

# rsyslog

[root@localhost ~]# strace -T -ttt -f -p 35341
strace: Process 35341 attached with 3 threads
[pid 35343] 1532962871.699907 futex(0x559329e6e2ac, FUTEX_WAIT_PRIVATE, 23, NULL <unfinished ...>
[pid 35342] 1532962871.699926 select(4, [3], NULL, NULL, NULL <unfinished ...>
[pid 35341] 1532962871.699940 select(1, NULL, NULL, NULL, {367, 871364} <unfinished ...>
[pid 35342] 1532962894.732966 <... select resumed> ) = 1 (in [3]) <23.033022>
[pid 35342] 1532962894.733015 recvmsg(3, {msg_name(0)=NULL, msg_iov(1)=[{"<13>Jul 30 11:01:34 root: newer", 8096}], msg_controllen=64, [{cmsg_len=32, cmsg_level=SOL_SOCKET, cmsg_type=0x1d /* SCM_??? */}, {cmsg_len=28, cmsg_level=SOL_SOCKET, cmsg_type=SCM_CREDENTIALS, {pid=10247, uid=0, gid=0}}], msg_flags=0}, MSG_DONTWAIT) = 31 <0.000008>
[pid 35342] 1532962894.733070 futex(0x559329e6e2ac, FUTEX_WAKE_OP_PRIVATE, 1, 1, 0x559329e6e2a8, {FUTEX_OP_SET, 0, FUTEX_OP_CMP_GT, 1} <unfinished ...>
[pid 35343] 1532962894.733094 <... futex resumed> ) = 0 <23.033169>
[pid 35342] 1532962894.733098 <... futex resumed> ) = 1 <0.000018>
[pid 35343] 1532962894.733110 futex(0x559329e6e0b0, FUTEX_WAIT_PRIVATE, 2, NULL <unfinished ...>
[pid 35342] 1532962894.733116 futex(0x559329e6e0b0, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 35343] 1532962894.733128 <... futex resumed> ) = -1 EAGAIN (Resource temporarily unavailable) <0.000012>
[pid 35342] 1532962894.733140 <... futex resumed> ) = 0 <0.000021>
[pid 35343] 1532962894.733154 futex(0x559329e6e0b0, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 35342] 1532962894.733160 select(4, [3], NULL, NULL, NULL <unfinished ...>
[pid 35343] 1532962894.733175 <... futex resumed> ) = 0 <0.000016>
[pid 35343] 1532962894.733194 write(5, "Jul 30 11:01:34 localhost root: "..., 38) = 38 <0.000020>
[pid 35343] 1532962894.733234 futex(0x559329e6e2ac, FUTEX_WAIT_PRIVATE, 25, NULL

# systemd-journald

[root@localhost ~]# strace -ttt -f -p 10247
strace: Process 10247 attached
1532962887.731909 epoll_wait(7, [{EPOLLIN, {u32=3876856720, u64=94080840508304}}], 10, -1) = 1
1532962894.732687 clock_gettime(CLOCK_BOOTTIME, {246287, 470494988}) = 0
1532962894.732723 ioctl(5, FIONREAD, [31]) = 0
1532962894.732755 recvmsg(5, {msg_name(0)=0x7ffc9785fab0, msg_iov(1)=[{"<13>Jul 30 11:01:34 root: newer", 24575}], msg_controllen=136, [{cmsg_len=32, cmsg_level=SOL_SOCKET, cmsg_type=0x1d /* SCM_??? */}, {cmsg_len=28, cmsg_level=SOL_SOCKET, cmsg_type=SCM_CREDENTIALS, {pid=35455, uid=0, gid=0}}, {cmsg_len=70, cmsg_level=SOL_SOCKET, cmsg_type=SCM_SECURITY, "unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023\0"}], msg_flags=MSG_CMSG_CLOEXEC}, MSG_DONTWAIT|MSG_CMSG_CLOEXEC) = 31
1532962894.732825 sendmsg(5, {msg_name(29)={sa_family=AF_LOCAL, sun_path="/run/systemd/journal/syslog"}, msg_iov(1)=[{"<13>Jul 30 11:01:34 root: newer", 31}], msg_controllen=28, [{cmsg_len=28, cmsg_level=SOL_SOCKET, cmsg_type=SCM_CREDENTIALS, {pid=35455, uid=0, gid=0}}], msg_flags=0}, MSG_NOSIGNAL) = -1 ESRCH (No such process)
1532962894.732870 sendmsg(5, {msg_name(29)={sa_family=AF_LOCAL, sun_path="/run/systemd/journal/syslog"}, msg_iov(1)=[{"<13>Jul 30 11:01:34 root: newer", 31}], msg_controllen=28, [{cmsg_len=28, cmsg_level=SOL_SOCKET, cmsg_type=SCM_CREDENTIALS, {pid=10247, uid=0, gid=0}}], msg_flags=0}, MSG_NOSIGNAL) = 31
1532962894.733261 open("/proc/35455/cgroup", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.733307 open("/proc/35455/comm", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.733331 readlinkat(AT_FDCWD, "/proc/35455/exe", 0x5590e71435b0, 99) = -1 ENOENT (No such file or directory)
1532962894.733872 open("/proc/35455/cmdline", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.733923 open("/proc/35455/status", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.733946 open("/proc/35455/sessionid", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.733988 open("/proc/35455/loginuid", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.734014 open("/proc/35455/cgroup", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
1532962894.734050 fstat(14, {st_mode=S_IFREG|0640, st_size=8388608, ...}) = 0
1532962894.734146 ftruncate(14, 8388608) = 0

这里可以看到，35455是logger发送时建立的进程pid
[root@localhost ~]# journalctl | grep 35455
Jul 30 10:31:18 localhost.localdomain root[35455]: 123
Jul 30 10:32:35 localhost.localdomain root[35455]: 123
Jul 30 10:34:03 localhost.localdomain root[35455]: 123
Jul 30 10:35:30 localhost.localdomain root[35455]: 123
Jul 30 11:01:34 localhost.localdomain root[35455]: newer
```

# 常见问题

## 日志重复
见上图，可知默认rsyslog.conf中启动了imuxsock与imjournal两个模块，分别会通过不同的渠道获得syslog日志，因此会导致重复。配置方法详见下章配置信息对应项。

## 日志丢失

### 由于日志频次丢失

rsyslog的imjournal模块读取数据库有一个频率上限设置，而systemd-journald也有一个数据库读取频率上限设置。满足rsyslog频率上限，messages中就会drop日志；满足systemd-journald上限，journald就会miss日志。配置方法详见下章配置信息对应项。

```
rsyslogd: imjournal: 84667 messages lost due to rate-limiting

systemd-journal[1770]: Missed 1427 kernel messages

```

### journal正常，rsyslog停止记录日志

当前测试似乎是在/var/log/messages被移动或者被删除或者被轮转了导致的这个问题

在logrotate那里执行了达到100M就轮转的功能，会删除.1文件然后备份

但是这边的现象还是有点怪，在我关闭那个特别高频的日志写入之后，就会更新最新的了，但这是为什么呢？

这里的messages是在/b_iscsi/log/messages里的，这块的设置倒是和我没太大关系，不过原因还是得测试一下，在自己的53.31的/var/log/messages里测试是没什么问题的。

通过lsof，并且从logrotate那里删除自动备份的操作，结果发现，rsyslog还是会出现被删除的情况，但是没找到哪里触发的，

logrotate会在100M时删除
sys_space这个进程会在messages达到110M时删除

Aug 29 17:02:28 localhost syslog: [do_record_log_t:1013] get log info error

目前更换回/var/log/messages，没有出现被删的情况，但是每满130M会出现一次rsyslog不再读取journal的问题,需要移除那个快速写入的程序才能接着更新

看了下卡死的那个时候，rsyslog的lsof显示并不是像我之前想的那样，读取的全都是Deleted文件，反而有些是正常的文件。

刚才试了一下，top里可以看到rsyslog卡死之后读取了30M缓存。

欸，但是命名rsyslog是直接读取systemd-journald数据库啊，哪里有缓存的位置

## 测试journald数据库性能上限


 数据库文件大小上限设置为2T,在尚未抵达文件大小上限时，出现了间歇的丢失日志

```
Aug  7 15:53:23 localhost journal: Missed 68 kernel messages
Aug  7 15:53:23 localhost journal: Missed 770 kernel messages
Aug  7 15:53:23 localhost journal: Missed 16 kernel messages
Aug  7 15:53:23 localhost journal: Missed 95 kernel messages
Aug  7 15:53:23 localhost journal: Missed 77 kernel messages
Aug  7 15:53:23 localhost journal: Missed 65 kernel messages
Aug  7 15:53:23 localhost journal: Missed 71 kernel messages
Aug  7 15:53:23 localhost journal: Missed 92 kernel messages
Aug  7 15:53:23 localhost journal: Missed 110 kernel messages
Aug  7 15:53:23 localhost journal: Missed 89 kernel messages
Aug  7 15:53:23 localhost journal: Missed 206 kernel messages
Aug  7 15:53:23 localhost journal: Missed 5 kernel messages
Aug  7 15:53:23 localhost journal: Missed 485 kernel messages
Aug  7 15:53:23 localhost journal: Missed 243 kernel messages
Aug  7 15:53:23 localhost journal: Missed 105 kernel messages
Aug  7 15:53:23 localhost journal: Missed 893 kernel messages
Aug  7 15:53:23 localhost journal: Missed 947 kernel messages
Aug  7 15:53:23 localhost journal: Missed 837 kernel messages
Aug  7 15:53:23 localhost journal: Missed 890 kernel messages
Aug  7 15:53:23 localhost journal: Missed 892 kernel messages
Aug  7 15:53:23 localhost journal: Missed 914 kernel messages
Aug  7 15:53:23 localhost journal: Missed 894 kernel messages
Aug  7 15:53:23 localhost journal: Missed 896 kernel messages
Aug  7 15:53:23 localhost journal: Missed 888 kernel messages
Aug  7 15:53:23 localhost journal: Missed 901 kernel messages
Aug  7 15:53:23 localhost journal: Missed 928 kernel messages
Aug  7 15:53:23 localhost journal: Missed 887 kernel messages
Aug  7 15:53:23 localhost journal: Missed 884 kernel messages
Aug  7 15:53:23 localhost journal: Missed 905 kernel messages
:

```

## 测试再persistent与volatile状态下，数据库停止记录条件

目前在设置了日志上限的情况下，并没有出现数据库卡死，都是在覆写之前的日志。

在公司镜像上测试，发现，在volatile状态下的日志，和在persistent状态下，同样会自动覆写之前的日志。

仅当journald日志大小达到该分区上限时，目前测试为当RunMaxUse大于分区可用空间时，会导致日志卡死。

## imjournald journal reloaded问题

这个问题在[7]中被提到，主要时由于systemd-journald正在轮转数据库文件，因此导致数据库文件变动，所以会出现这个reload日志。

根据[8]中添加的这个日志信息，可以看到是为了在journald切换文件位置时，为了不用重启rsyslog而添加的自动切换\加载功能。

经过测试，journal日志每被切割一次，都会产生一个reloaded信息（日志level是info，不是Error，所以可以忽视）

## /dev/log 丢失

参照[11]，这个socket似乎在systemd设备上，是由systemd-journald.socket提供，
如果是单独的rsyslog的日志管理下，则是由imuxsock插件创建，

>Normally, with rsyslogd, the imuxsock module will create the /dev/log socket on its own, unlinking the previous entry before creating it. When rsyslogd is stopped (possibly because restart which fails because of faulty configuration), rsyslogd removes /dev/log.
>
>However, the rsyslog supplied with RHEL7 is expected to be used in conjunction with systemd, and the imuxsock module will actually open and remove /run/systemd/journal/syslog socket. Meanwhile, the /dev/log device is created by the system service-file systemd-journald.socket which triggers journald.


在systemd-journald.socket这个服务的Unit文件里是这么写的

```
[root@localhost ~]# less /lib/systemd/system/systemd-journald.socket 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Journal Socket
Documentation=man:systemd-journald.service(8) man:journald.conf(5)
DefaultDependencies=no
Before=sockets.target

# Mount and swap units need this. If this socket unit is removed by an
# isolate request the mount and swap units would be removed too,
# hence let's exclude this from isolate requests.
IgnoreOnIsolate=yes

[Socket]
ListenStream=/run/systemd/journal/stdout
ListenDatagram=/run/systemd/journal/socket
ListenDatagram=/dev/log
SocketMode=0666
PassCredentials=yes
PassSecurity=yes
ReceiveBuffer=8M
```

这里可以看到监听的/dev/log端口创建是由这个服务管理的。

## /var/log/messages 时间存在跳变，乱序

暂时无法复现

## journal input/output Error

理论上来说，应该是journald连接写入到数据库被阻断了，数据库无法访问导致的问题。

不过应该仅针对journalctl读取时的问题

## 将/var/log/journal目录做了软链接后，指向路径是被挂载的盘，在系统启动尚未挂载上时，会导致日志不合并，会丢失

欸，我试下了，挂载上后，新的日志就看不到了，而卸载掉后，这次启动的日志文件还是在的。

在测试中，新挂载的盘中与systemd-journal并没有建立连接，在lsof中看到的该路径下的文件是现在已经被隐藏了的目录，通过`stat`查看了文件inode，的确如此。

```
systemd-j 432 root   17u      REG                8,1  8388608  531238 /root/log/journal/abf6c3e0a96f452ab2efd6c2d1a9c1e0/system.journal


[root@thor ~]# stat /root/log/journal/abf6c3e0a96f452ab2efd6c2d1a9c1e0/system.journal 
  File: 鈥root/log/journal/abf6c3e0a96f452ab2efd6c2d1a9c1e0/system.journal鈥
  Size: 8388608    Blocks: 16384      IO Block: 4096   regular file
Device: 811h/2065d      Inode: 42729475    Links: 1
Access: (0640/-rw-r-----)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2018-08-22 14:31:18.865335001 +0800
Modify: 2018-08-22 14:27:43.244195482 +0800
Change: 2018-08-22 14:27:43.244195482 +0800
 Birth: -
[root@thor ~]# umount /dev/sdc1
[root@thor ~]# stat /root/log/journal/abf6c3e0a96f452ab2efd6c2d1a9c1e0/system.journal 
  File: 鈥root/log/journal/abf6c3e0a96f452ab2efd6c2d1a9c1e0/system.journal鈥
  Size: 8388608    Blocks: 16408      IO Block: 4096   regular file
Device: 801h/2049d      Inode: 531238      Links: 1
Access: (0640/-rw-r-----)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2018-08-22 14:34:01.246346582 +0800
Modify: 2018-08-22 14:39:01.133367970 +0800
Change: 2018-08-22 14:39:01.133367970 +0800
 Birth: -
 ```

# 配置信息
## /etc/rsyslog.conf

### $ModLoad imuxsock

该模块导入监听本机syslog socket的功能，从syslog中接收日志,该配置项依赖systemd-journald.conf中的ForwardToSyslog=Yes

#### $OmitLocalLogging 与 $SystemLogSocketName

该配置信息测试结果与官网有所区别，测试中指定socketName为/dev/log(可修改)，系统默认指向socket为/run/systemd/journal/syslog。使用详见下表。

``` bash
# 显式关闭 OmitLocalLog,未指定socketName || 非显式关闭，未指定socketName
[root@localhost ~]# lsof -c rsyslog
COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF   NODE NAME
rsyslogd 1713 root  cwd    DIR              253,0      224     64 /
rsyslogd 1713 root  rtd    DIR              253,0      224     64 /
rsyslogd 1713 root  txt    REG              253,0   663960    961 /usr/sbin/rsyslogd
rsyslogd 1713 root  mem    REG              253,0    38128   9671 /usr/lib64/rsyslog/imuxsock.so
rsyslogd 1713 root  mem    REG              253,0    24520   9672 /usr/lib64/rsyslog/lmnet.so
rsyslogd 1713 root  mem    REG              253,0  2127336  61000 /usr/lib64/libc-2.17.so
rsyslogd 1713 root  mem    REG              253,0    88720     84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
rsyslogd 1713 root  mem    REG              253,0    20040  87327 /usr/lib64/libuuid.so.1.3.0
rsyslogd 1713 root  mem    REG              253,0    40824 322855 /usr/lib64/libfastjson.so.4.0.0
rsyslogd 1713 root  mem    REG              253,0    15424 322847 /usr/lib64/libestr.so.0.0.0
rsyslogd 1713 root  mem    REG              253,0    44448  72710 /usr/lib64/librt-2.17.so
rsyslogd 1713 root  mem    REG              253,0    19776  61006 /usr/lib64/libdl-2.17.so
rsyslogd 1713 root  mem    REG              253,0   144792  72706 /usr/lib64/libpthread-2.17.so
rsyslogd 1713 root  mem    REG              253,0    90664  87310 /usr/lib64/libz.so.1.2.7
rsyslogd 1713 root  mem    REG              253,0   164264  60993 /usr/lib64/ld-2.17.so
rsyslogd 1713 root    0r   CHR                1,3      0t0   5423 /dev/null
rsyslogd 1713 root    1w   CHR                1,3      0t0   5423 /dev/null
rsyslogd 1713 root    2w   CHR                1,3      0t0   5423 /dev/null
rsyslogd 1713 root    3u  unix 0xffff880037064800      0t0  29648 /run/systemd/journal/syslog

# 显式打开OmitLocalLog 并 指定socketName || 并不指定socketName
[root@localhost ~]# lsof -c rsyslog
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
rsyslogd 1765 root  cwd    DIR  253,0      224     64 /
rsyslogd 1765 root  rtd    DIR  253,0      224     64 /
rsyslogd 1765 root  txt    REG  253,0   663960    961 /usr/sbin/rsyslogd
rsyslogd 1765 root  mem    REG  253,0    38128   9671 /usr/lib64/rsyslog/imuxsock.so
rsyslogd 1765 root  mem    REG  253,0    24520   9672 /usr/lib64/rsyslog/lmnet.so
rsyslogd 1765 root  mem    REG  253,0  2127336  61000 /usr/lib64/libc-2.17.so
rsyslogd 1765 root  mem    REG  253,0    88720     84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
rsyslogd 1765 root  mem    REG  253,0    20040  87327 /usr/lib64/libuuid.so.1.3.0
rsyslogd 1765 root  mem    REG  253,0    40824 322855 /usr/lib64/libfastjson.so.4.0.0
rsyslogd 1765 root  mem    REG  253,0    15424 322847 /usr/lib64/libestr.so.0.0.0
rsyslogd 1765 root  mem    REG  253,0    44448  72710 /usr/lib64/librt-2.17.so
rsyslogd 1765 root  mem    REG  253,0    19776  61006 /usr/lib64/libdl-2.17.so
rsyslogd 1765 root  mem    REG  253,0   144792  72706 /usr/lib64/libpthread-2.17.so
rsyslogd 1765 root  mem    REG  253,0    90664  87310 /usr/lib64/libz.so.1.2.7
rsyslogd 1765 root  mem    REG  253,0   164264  60993 /usr/lib64/ld-2.17.so
rsyslogd 1765 root    0r   CHR    1,3      0t0   5423 /dev/null
rsyslogd 1765 root    1w   CHR    1,3      0t0   5423 /dev/null
rsyslogd 1765 root    2w   CHR    1,3      0t0   5423 /dev/null

# 非显式开启OmitLocalLog ，指定socketName || 显式关闭OmitLocalLog, 并指定SocketName
[root@localhost ~]# lsof -c rsyslog
COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF     NODE NAME
rsyslogd 1779 root  cwd    DIR              253,0      224       64 /
rsyslogd 1779 root  rtd    DIR              253,0      224       64 /
rsyslogd 1779 root  txt    REG              253,0   663960      961 /usr/sbin/rsyslogd
rsyslogd 1779 root  mem    REG              253,0    38128     9671 /usr/lib64/rsyslog/imuxsock.so
rsyslogd 1779 root  mem    REG              253,0    24520     9672 /usr/lib64/rsyslog/lmnet.so
rsyslogd 1779 root  mem    REG              253,0  2127336    61000 /usr/lib64/libc-2.17.so
rsyslogd 1779 root  mem    REG              253,0    88720       84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
rsyslogd 1779 root  mem    REG              253,0    20040    87327 /usr/lib64/libuuid.so.1.3.0
rsyslogd 1779 root  mem    REG              253,0    40824   322855 /usr/lib64/libfastjson.so.4.0.0
rsyslogd 1779 root  mem    REG              253,0    15424   322847 /usr/lib64/libestr.so.0.0.0
rsyslogd 1779 root  mem    REG              253,0    44448    72710 /usr/lib64/librt-2.17.so
rsyslogd 1779 root  mem    REG              253,0    19776    61006 /usr/lib64/libdl-2.17.so
rsyslogd 1779 root  mem    REG              253,0   144792    72706 /usr/lib64/libpthread-2.17.so
rsyslogd 1779 root  mem    REG              253,0    90664    87310 /usr/lib64/libz.so.1.2.7
rsyslogd 1779 root  mem    REG              253,0   164264    60993 /usr/lib64/ld-2.17.so
rsyslogd 1779 root    0r   CHR                1,3      0t0     5423 /dev/null
rsyslogd 1779 root    1w   CHR                1,3      0t0     5423 /dev/null
rsyslogd 1779 root    2w   CHR                1,3      0t0     5423 /dev/null
rsyslogd 1779 root    3u  unix 0xffff88003b564800      0t0    30441 /dev/log
rsyslogd 1779 root    4u  unix 0xffff88003b560800      0t0    30443 socket
rsyslogd 1779 root    5w   REG              253,0   754001 17131680 /var/log/messages
rsyslogd 1779 root    6w   REG              253,0    24654 17131681 /var/log/secure
```

由上述可知，在不同情况下，rsyslog实际监听的socket如下

| x            | 显式开启 | 显式关闭                    | 非显式开启                  |
| ------------ | -------- | --------------------------- | --------------------------- |
| 指定socket   | null     | /dev/log                    | /dev/log                    |
| 不指定socket | null     | /run/systemd/journal/syslog | /run/systemd/journal/syslog |

### $ModLoad imjournal

该模块导入直接读取systemd-journald数据库的功能，可直接从journald数据库中读取syslog日志、内核日志以及服务stdout\stderr等信息。

imjournal在lsof中可以看到，直接读取的数据库
``` bash
[root@localhost ~]# lsof  /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
systemd-j 10247 root  mem    REG   0,19 25165824 7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
systemd-j 10247 root   12u   REG   0,19 25165824 7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
rsyslogd  10255 root  mem    REG   0,19 25165824 7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
rsyslogd  10255 root    5r   REG   0,19 25165824 7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal

[root@localhost ~]# lsof -c systemd-journal
COMMAND     PID USER   FD      TYPE             DEVICE SIZE/OFF     NODE NAME
systemd-j 10247 root  cwd       DIR              253,0      224       64 /
systemd-j 10247 root  rtd       DIR              253,0      224       64 /
systemd-j 10247 root  txt       REG              253,0   274752 16874824 /usr/lib/systemd/systemd-journald
systemd-j 10247 root  mem       REG               0,19 25165824  7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
systemd-j 10247 root  mem       REG              253,0    19888    87389 /usr/lib64/libattr.so.1.1.0
systemd-j 10247 root  mem       REG              253,0   402384    87259 /usr/lib64/libpcre.so.1.2.0
systemd-j 10247 root  mem       REG              253,0    19776    61006 /usr/lib64/libdl-2.17.so
systemd-j 10247 root  mem       REG              253,0    19384    87371 /usr/lib64/libgpg-error.so.0.10.0
systemd-j 10247 root  mem       REG              253,0  2127336    61000 /usr/lib64/libc-2.17.so
systemd-j 10247 root  mem       REG              253,0   144792    72706 /usr/lib64/libpthread-2.17.so
systemd-j 10247 root  mem       REG              253,0    88720       84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
systemd-j 10247 root  mem       REG              253,0    44448    72710 /usr/lib64/librt-2.17.so
systemd-j 10247 root  mem       REG              253,0    37056    87391 /usr/lib64/libacl.so.1.1.0
systemd-j 10247 root  mem       REG              253,0   155744    87307 /usr/lib64/libselinux.so.1
systemd-j 10247 root  mem       REG              253,0   535064    87381 /usr/lib64/libgcrypt.so.11.8.2
systemd-j 10247 root  mem       REG              253,0   157424    87316 /usr/lib64/liblzma.so.5.2.2
systemd-j 10247 root  mem       REG              253,0   164264    60993 /usr/lib64/ld-2.17.so
systemd-j 10247 root  mem       REG               0,19        8     7972 /run/systemd/journal/kernel-seqnum
systemd-j 10247 root    0r      CHR                1,3      0t0     5423 /dev/null
systemd-j 10247 root    1w      CHR                1,3      0t0     5423 /dev/null
systemd-j 10247 root    2w      CHR                1,3      0t0     5423 /dev/null
systemd-j 10247 root    3u     unix 0xffff880037e83000      0t0    42542 /run/systemd/journal/stdout
systemd-j 10247 root    4u     unix 0xffff880037e81400      0t0    42544 /run/systemd/journal/socket
systemd-j 10247 root    5u     unix 0xffff880037e80800      0t0    42546 /dev/log
systemd-j 10247 root    6w      CHR               1,11      0t0     5429 /dev/kmsg
systemd-j 10247 root    7u  a_inode                0,9        0     5419 [eventpoll]
systemd-j 10247 root    8u  a_inode                0,9        0     5419 [timerfd]
systemd-j 10247 root    9u      CHR               1,11      0t0     5429 /dev/kmsg
systemd-j 10247 root   10r      REG                0,3        0     7973 /proc/sys/kernel/hostname
systemd-j 10247 root   11u  a_inode                0,9        0     5419 [signalfd]
systemd-j 10247 root   12u      REG               0,19 25165824  7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
systemd-j 10247 root   13u  a_inode                0,9        0     5419 [timerfd]

[root@localhost ~]# lsof -c rsyslog
COMMAND    PID USER   FD      TYPE             DEVICE SIZE/OFF     NODE NAME
rsyslogd 10255 root  cwd       DIR              253,0      224       64 /
rsyslogd 10255 root  rtd       DIR              253,0      224       64 /
rsyslogd 10255 root  txt       REG              253,0   663960      961 /usr/sbin/rsyslogd
rsyslogd 10255 root  mem       REG               0,19 25165824   307917 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-0000000000000f6d-0005722d8a52e79b.journal
rsyslogd 10255 root  mem       REG               0,19 25165824  2282729 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-00000000000074f5-0005722d9bdab060.journal
rsyslogd 10255 root  mem       REG               0,19 25165824  4612418 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-000000000000dcd4-0005722db4fed7dd.journal
rsyslogd 10255 root  mem       REG               0,19 25165824  7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
rsyslogd 10255 root  mem       REG               0,19  6455296     7976 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-0000000000000001-000571ff63cd3044.journal
rsyslogd 10255 root  mem       REG              253,0    68192    87353 /usr/lib64/libbz2.so.1.0.6
rsyslogd 10255 root  mem       REG              253,0    99944    87368 /usr/lib64/libelf-0.168.so
rsyslogd 10255 root  mem       REG              253,0   402384    87259 /usr/lib64/libpcre.so.1.2.0
rsyslogd 10255 root  mem       REG              253,0    19888    87389 /usr/lib64/libattr.so.1.1.0
rsyslogd 10255 root  mem       REG              253,0   297328   115274 /usr/lib64/libdw-0.168.so
rsyslogd 10255 root  mem       REG              253,0   111080    72708 /usr/lib64/libresolv-2.17.so
rsyslogd 10255 root  mem       REG              253,0    19384    87371 /usr/lib64/libgpg-error.so.0.10.0
rsyslogd 10255 root  mem       REG              253,0   535064    87381 /usr/lib64/libgcrypt.so.11.8.2
rsyslogd 10255 root  mem       REG              253,0   157424    87316 /usr/lib64/liblzma.so.5.2.2
rsyslogd 10255 root  mem       REG              253,0   155744    87307 /usr/lib64/libselinux.so.1
rsyslogd 10255 root  mem       REG              253,0  1139680    61008 /usr/lib64/libm-2.17.so
rsyslogd 10255 root  mem       REG              253,0    20032    87393 /usr/lib64/libcap.so.2.22
rsyslogd 10255 root  mem       REG              253,0    25072     9665 /usr/lib64/rsyslog/imjournal.so
rsyslogd 10255 root  mem       REG              253,0    24520     9672 /usr/lib64/rsyslog/lmnet.so
rsyslogd 10255 root  mem       REG              253,0  2127336    61000 /usr/lib64/libc-2.17.so
rsyslogd 10255 root  mem       REG              253,0    88720       84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
rsyslogd 10255 root  mem       REG              253,0    20040    87327 /usr/lib64/libuuid.so.1.3.0
rsyslogd 10255 root  mem       REG              253,0    40824   322855 /usr/lib64/libfastjson.so.4.0.0
rsyslogd 10255 root  mem       REG              253,0    15424   322847 /usr/lib64/libestr.so.0.0.0
rsyslogd 10255 root  mem       REG              253,0    44448    72710 /usr/lib64/librt-2.17.so
rsyslogd 10255 root  mem       REG              253,0    19776    61006 /usr/lib64/libdl-2.17.so
rsyslogd 10255 root  mem       REG              253,0   144792    72706 /usr/lib64/libpthread-2.17.so
rsyslogd 10255 root  mem       REG              253,0    90664    87310 /usr/lib64/libz.so.1.2.7
rsyslogd 10255 root  mem       REG              253,0   164264    60993 /usr/lib64/ld-2.17.so
rsyslogd 10255 root  mem       REG              253,0   162560   237027 /usr/lib64/libsystemd.so.0.6.0
rsyslogd 10255 root    0r      CHR                1,3      0t0     5423 /dev/null
rsyslogd 10255 root    1w      CHR                1,3      0t0     5423 /dev/null
rsyslogd 10255 root    2w      CHR                1,3      0t0     5423 /dev/null
rsyslogd 10255 root    3r  a_inode                0,9        0     5419 inotify
rsyslogd 10255 root    4u     unix 0xffff880037338800      0t0  9404081 socket
rsyslogd 10255 root    5r      REG               0,19 25165824  7373149 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system.journal
rsyslogd 10255 root    6r      REG               0,19 25165824  4612418 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-000000000000dcd4-0005722db4fed7dd.journal
rsyslogd 10255 root    7r      REG               0,19 25165824  2282729 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-00000000000074f5-0005722d9bdab060.journal
rsyslogd 10255 root    8r      REG               0,19 25165824   307917 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-0000000000000f6d-0005722d8a52e79b.journal
rsyslogd 10255 root    9r      REG               0,19  6455296     7976 /run/log/journal/a288ec3729494d3dad642453d0b272b7/system@cea802f344b94cb6b1e2b867690eca83-0000000000000001-000571ff63cd3044.journal
rsyslogd 10255 root   10w      REG              253,0 49250519 17152683 /var/log/messages
rsyslogd 10255 root   11w      REG              253,0     4634 17152684 /var/log/secure
```



#### $imjournalRatelimitInterval 0

该属性设置为5s,代表以5s为一个间隙统计日志频次，达到下一条配置设置的频次，即放弃读取journald数据库信息(待验证是读取前放弃，还是读取后丢弃)。

#### $imjournalRatelimitBurst 0

该属性设置为上一条设置的间隙期间读取日志条数上限，如5s内读取1000条，达到该频次即停止。与上一条同时设置为0，即关闭该上限

>Note that it is not recommended to turn of ratelimiting, except that you know for sure journal database entries will never be corrupted. Without ratelimiting, a corrupted systemd journal database may cause a kind of denial of service (we are stressing this point as multiple users have reported us such problems with the journal database - information current as of June 2013).

但是官方并不建议关闭该上限，可能会导致数据库阻塞等问题导致其他服务出现异常。




### $ModLoad imklog

该模块导入直接从平台内核中读取内核日志的功能，可以避过journald数据库读写性能瓶颈。

### $ModLoad imkmsg

该模块通过/dev/kmsg设备，获取结构化日志

### $ModLoad imudp

导入该模块，并打开防火墙放行，可远程访问该主机获取日志信息

### $ModLoad imtcp

导入该模块，并打开防火墙放行，可远程访问该主机获取日志信息

### 转发规则
rhel7系统中，一般log默认保存在下述目录，/var/log 目录保管由rsyslog维护的各种特定于系统和服务的日志文件。

/var/log/messages大多数系统日志消息记录在此。例外是与身份验证，电子邮件处理相关的定期运行作业的消息以及纯粹与调试相关的信息。
/var/log/secure安全和身份验证相关的消息和错误的日志文件。
/var/log/maillog与邮件服务器相关的日志文件。
/var/log/cron crond计划任务的日志
/var/log/boot.log与系统启动相关的消息记录在此。


建议不直接修改rsyslog.conf的规则，在这个目录`$IncludeConfig /etc/rsyslog.d/*.conf`下存放自定义的转发规则

``` bash
:msg, contains, "of user root"            ~ 
& ~

:msg, contains, "Removed session"            ~ 
& ~

:msg, contains, "please try to use systemctl"		~
& ~	

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log

```

转发过滤规则，见[文档](https://www.rsyslog.com/doc/v8-stable/configuration/filters.html?highlight=info%20mail%20none%20authpriv%20none%20cron%20none%20news%20none)

### begin forwarding rule
暂未使用的远程访问日志配置信息，语法见[Legacy Action-Specific Configuration Statements](https://www.rsyslog.com/doc/v8-stable/configuration/action/index.html?highlight=actionfiledefaulttemplate)

## /etc/systemd/journald.conf

systemd-journald主要获得以下信息
* Kernel log messages, via kmsg
* Simple system log messages, via the libc syslog(3) call
* Structured system log messages via the native Journal API, see sd_journal_print(4)
* Standard output and standard error of service units. For further details see below.
* Audit records, originating from the kernel audit subsystem

### RateLimitInterval=30s
频率间隙为30s

### RateLimitBurst=1000
每个间隙频次上限为1000，直到下个间隙不会接收日志

### Storage=auto

该配置选项控制journald日志的存储位置，以下4个选项"volatile", "persistent", "auto" and "none"，对应`/run/log/journal`,`/var/log/journal`,根据`/var/log/journal`目录建立与否判断，收到即drop,但转发forward还是生效的(如imjournal等读取数据库文件等方式无效)。

### RuntimeMaxUse=

设置内存中journald文件上限，一旦达到该上限，journald数据库就会阻塞住。

### RuntimeKeepFree=

RuntimeKeepFree表示需要保留的内存空间，剩余空间不足其设置，journald数据库同样会阻塞住。

SystemMaxUse= 与 RuntimeMaxUse= 的默认值是10%空间与4G空间两者中的较小者； SystemKeepFree= 与 RuntimeKeepFree= 的默认值是15%空间与4G空间两者中的较大者； 如果在 systemd-journald 启动时， 文件系统即将被填满并且已经超越了 SystemKeepFree= 或 RuntimeKeepFree= 的限制，那么日志记录将被暂停。 也就是说，如果在创建日志文件时，文件系统有充足的空闲空间， 但是后来文件系统被其他非日志文件过多占用， 那么 systemd-journald 只会立即暂停日志记录， 但不会删除已经存在的日志文件。

### RuntimeMaxFileSize=
SystemMaxFileSize= 与 RuntimeMaxFileSize= 限制单个日志文件的最大体积， 到达此限制后日志文件将会自动滚动。 默认值是对应的 SystemMaxUse= 与 RuntimeMaxUse= 值的1/8 ， 这也意味着日志滚动默认保留7个历史文件。

日志大小的值可以使用以1024为基数的 K, M, G, T, P, E 后缀， 分别对应于 1024, 1024², … 字节。

### ForwardToSyslog=yes

转发syslog日志到syslog socket，从而使rsyslog调用imuxsock从该socket接收日志

该选项可被内核引导选项覆盖`systemd.journald.forward_to_syslog=, systemd.journald.forward_to_kmsg=, systemd.journald.forward_to_console=, systemd.journald.forward_to_wall=`，允许/禁止将收集到的日志： 转发到传统的 syslog 守护进程, 转发到内核日志缓冲区, 转发到系统控制台, 作为wall警告信息转发给所有已登录的用户

### MaxLevelStore=debug
* MaxLevelSyslog=debug
* MaxLevelKMsg=notice
* MaxLevelConsole=info
* MaxLevelWall=emerg

以上配置控制在数据库上存储以及转发的最大日志等级。

日志等级一共分为"emerg", "alert", "crit", "err", "warning", "notice","info", "debug"

## /lib/systemd/system/systemd-journald.service

### StandardOutput=null

设置进程的标准输出(STDOUT)。 可设为 inherit, null, tty, journal, syslog, kmsg, journal+console, syslog+console, kmsg+console, socket, fd 之一。

* inherit 表示使用 StandardInput= 设置的值。
* null 表示 /dev/null ， 也就是所有写入都会被丢弃。
* tty 表示 TTY(由 TTYPath= 设置)， 如果仅用于输出， 那么进程将无需取得终端的控制权， 亦无需等待其他进程释放终端控制权。
* journal 表示 systemd 日志服务(通过 journalctl(1) 访问)。 注意，所有发到 syslog 或 kmsg 的日志都会 隐含的复制一份到 journal 中。
* syslog 表示 syslog(3) 日志服务。 注意，此时所有日志都会隐含的复制一份到 journal 中。
* kmsg 表示内核日志缓冲区(通过 dmesg(1) 访问)。 注意，此时所有日志都会隐含的复制一份到 journal 中。
* journal+console, syslog+console, kmsg+console 与上面三个值类似， 不同之处在于所有日志都会再复制一份到系统的控制台上。
* socket 的解释与 StandardInput= 中的解释完全相同。
* fd 表示将标准输出(STDOUT)连接到一个由 socket 单元提供的文件描述符。 可以通过 "fd:foobar" 格式 明确指定文件描述符的名称。 描述符名称的默认值为 "stdout" ，也就是 "fd" 等价于 "fd:stdout" 。 必须明确使用 Sockets= 选项 提供至少一个定义了文件描述符名称的 socket 单元。 注意，文件描述符的名称不一定和定义它的 socket 单元的名称一致。 如果出现了多个匹配，那么以第一个为准， 详见 systemd.socket(5) 手册对 FileDescriptorName= 选项的讲解。

如果单元的标准输出(StandardOutput=)或标准错误(StandardError=)中含有 journal, syslog, kmsg 之一， 那么该单元将会自动隐含的获得 After=systemd-journald.socket 依赖(见上文)。

## /etc/logrotate.conf

仅对持久化后的/var/log/journal有效

## journald持久化

持久化保存journal的日志，默认保存一个月的日志

直接修改journald.conf中的storage为persistent就切换到var路径下了，切换到volatile就自动回/run/log了。
``` bash
systemctl restart systemd-journald.service
systemctl restart systemd-journald.socket
```

如果出现了切换到persistent状态下，日志已经存到了/var/log/journal，但是/run/log/journal路径依旧存在的状况的话，可能是自动切换有些不同。就手动将volatile修改成auto,手动`mkdir /var/log/journal`，这样再重启比较适合防丢日志。

在切换前，为了防止journal数据库文件大小刚好超过设置的上限，然后由于重启了服务，没能及时自动清理掉超过的部分，从而导致数据库假死，建议使用`journalctl --vacuumm-size=250M`，可以清除日志直到满足这个大小限制。不过这个要求sytemd版本318及以上才支持这个选项。

如果版本不支持的话，那就还是`rm -rf`掉这些日志或者手动删掉一些数据库文件吧，升级systemd的版本似乎带来的风险相比这些日志的价值要大得多。[12]

# 调试方法
## 检验rsyslog配置信息
``` bash
# 可以让rsyslogd 进入 Debug模式
[root@localhost ~]# rsyslogd -N6
rsyslogd: version 8.24.0, config validation run (level 6), master config /etc/rsyslog.conf
rsyslogd: invalid or yet-unknown config file command 'IMJournalStateFile' - have you forgotten to load a module? [v8.24.0 try http://www.rsyslog.com/e/3003 ]
```

## strace -p pid追踪进程

# 参考资料
1. [imjournal: Systemd Journal Input Module](https://www.rsyslog.com/doc/v8-stable/configuration/modules/imjournal.html?highlight=imjournalstatefile)
2. [rsyslog-logger-message-duplicated](https://unix.stackexchange.com/questions/196877/rsyslog-logger-message-duplicated)
3. [systemd service 单元语法](https://zh.opensuse.org/openSUSE:How_to_write_a_systemd_service)
4. [Filter Conditions](https://www.rsyslog.com/doc/v8-stable/configuration/filters.html?highlight=info%20mail%20none%20authpriv%20none%20cron%20none%20news%20none)
5. [imuxsock: Unix Socket Input Module](https://www.rsyslog.com/doc/v8-stable/configuration/modules/imuxsock.html?highlight=omitlocallogging)
6. [man journald.conf](http://www.jinbuguo.com/systemd/journald.conf.html)
7. [rsyslog daemon have unkown log entries "rsyslogd:imjournal: journal reloaded" from time to time](https://bugzilla.redhat.com/show_bug.cgi?id=1497985)
8. [switching to persistent journal possible without rsyslog restart](https://github.com/rsyslog/rsyslog/pull/1747/files#diff-b1ea6478b8060f07cd30ecde78bfdc49R518)
9. [Journal is reloaded and duplicate messages are output into log file](https://bugzilla.redhat.com/show_bug.cgi?id=1495631)
10. [关于Rsyslogd 的一些配置 (高性能、高可用 rsyslogd)](http://www.tsingfun.com/html/2015/dev_1123/high_performance_rsyslogd.html)
11. [How do I restore `/dev/log` in systemd+rsyslog host?](https://unix.stackexchange.com/questions/317064/how-do-i-restore-dev-log-in-systemdrsyslog-host)
12. [How would I upgrade systemd?](https://askubuntu.com/questions/627174/how-would-i-upgrade-systemd)
13. [Is systemd-journald a syslog implementation?
](https://unix.stackexchange.com/questions/332274/is-systemd-journald-a-syslog-implementation)