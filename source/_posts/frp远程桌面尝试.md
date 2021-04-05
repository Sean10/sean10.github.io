---
title: frp远程桌面尝试
subtitle: tech
date: 2018-08-09 23:11:43
updated:
tags: [frp, rdp]
categories: [专业]
---

迫于公司访问外网使用的机器性能以及对外网访问的各种路由、权限限制，尝试远程到家里的电脑来进行访问。

<!--more-->

# 历程

偶然间看到带我的导师用的anydesk连的家里的电脑，推动了我马上开始捣鼓远程桌面。

在学校内网时，主要直接使用RDP来访问，现在都被NAT过了，那么就只有一些通过服务器来中转的软件可以使用了，第一反应就是teamviewer,不过经过这个月的摸索，公司对http请求都会ban掉，对于一些存在过风险的软件恐怕更加会ban了，果不其然，teamviewer的请求都是被拒绝了的。

简单了解，anydesk基本功能等同teamviewer，简单安装尝试，效果很棒，第一天从第一次连接到晚上下班，连接一直都没有断开。

只是第二天就是噩梦的来临，anydesk连接的持续时间不足5分钟就会断开，当时曾以为是公司网络的限制，回到家后同样如此，那么猜测可能是个人License的区别了，官网看了下，个人使用版本年费79刀，才能保证session。唉，只好选择其他方式了。

那么，就回归看上去最费事的frp穿透了。

# frp穿透+rdp远程桌面

配置说简单也简单，但是在像公司网络限制的地方，就难以确定可能出现的问题原因究竟是配置问题还是网络问题了。
```
wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz
tar -zxvf frp_0.20.0_linux_amd64.tar.gz
cd frp_0.20.0_linux_amd64/
```

服务端基本可以不改配置，直接启动`./frps -c ./frps.ini`也行
```
# frps.ini
[common]
bind_port = 7000
```

客户端，按照需求从full.ini中选取需要的就可以了，比如rdp,启动方式同上
```
# frpc.ini
[common]
server_addr = X.X.X.X
server_port = 7000

[RDP]
type = tcp
local_ip = 0.0.0.0
local_port = 3389
remote_port = 6000
```

两个都连上以后就可以看到`start proxy success`的成功信息。

不过成功连上以后，性能却不甚理想……远比anydesk使用时卡顿的多，看来还需要探索其他方式。

如果不是使用rdp，只需要ssh\文件传输等功能的话，可能还是非常不错的。

需要后台的话，使用systemd管理的方式的，在`/lib/systemd/system/`里添加一个service文件

```
[Unit]
Description=frps daemon

[Service]
ExecStart=/root/frp_0.20.0_linux_amd64/frps -c /root/frp_0.20.0_linux_amd64/frps.ini
Restart=always

[Install]
WantedBy=multi-user.target
```

上面这样基本就可以了，注意执行命令必须都是绝对路径

systemd的基本控制命令如下
```
systemctl daemon-reload
systemctl restart frps 
systemctl enable frps # 将frps作为开机启动项
```