---
title: systemd之systemd-tty-ask-password-agent卡住.md
subtitle: tech
date: 2021-04-10 23:47:41
updated:
tags: [systemd, linux, service]
categories: [专业]
---

# 背景
这个问题其实之前遇到的还挺多的, 简单来说, 就是某个A服务启动B服务,  但是B服务又必须等待另一个服务启动完毕(比如A服务), 产生了一定的死锁. 而在`ps`的进程表中看到的就是`systemd-tty-ask-password-agent`这样一个`agent`进程卡住. 一般查一下当前卡住的`service`和他依赖的`after`,`before`的几个服务, 就知道了, 一般就是循环死锁了.

具体理论这块还没去查源码, 暂时只能给个结论.有大佬看过的话, 希望也能分享下~


# 样例
```  bash
# new2.service
[Unit]                                                                               
Description=System Logging Service                                                   
Documentation=man:rsyslogd(8)                                                        
Documentation=https://www.rsyslog.com/doc/                                           
                                                                                     
[Service]                                                                            
Type=forking                                                                         
EnvironmentFile=-/etc/sysconfig/rsyslog                                              
ExecStart=/root/new2.sh                                                              
                                                                                     
# Increase the default a bit in order to allow many simultaneous                     
# files to be monitored, we might need a lot of fds.                                 
                                                                                     
[Install]                                                                            
WantedBy=multi-user.target   


# new2.sh
#!/bin/bash
echo "heelo"
sleep 1000 &
systemctl restart new1


# new1.service
[Unit]                                                                     
Description=System Logging Service                                         
Documentation=man:rsyslogd(8)                                              
Documentation=https://www.rsyslog.com/doc/                                 
After=new2.service                                                         
                                                                           
[Service]                                                                  
Type=forking                                                               
EnvironmentFile=-/etc/sysconfig/rsyslog                                    
ExecStart=/root/new1.sh                                                    
                                                                           
# Increase the default a bit in order to allow many simultaneous           
# files to be monitored, we might need a lot of fds.                       
                                                                           
[Install]                                                                  
WantedBy=multi-user.target                                                 

# new.sh
#!/bin/bash
echo "heelo"
sleep 1000 &
``` bash