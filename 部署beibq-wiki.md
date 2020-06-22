---
title: 部署Moin_wiki记录
date: 2018-05-30 23:16:46
updated:
tags: [wiki, centos]
categories: [专业]
---

之前一直想写一个论坛或者wiki，不过最近感觉想先部署着，以后慢慢改了。


<!--more-->



-----

# 安装Moinmoin

```
curl http://static.moinmo.in/files/moin-1.9.9.tar.gz -o moin-1.9.9.tar.gz
python2.7 wikiserver.py

# 开放端口
iptables -A INPUT -p tcp --dport 18082 -j ACCEPT
/etc/rc.d/init.d/iptables save
/etc/init.d/iptables restart

```

# 安装Mysql(虽然最后这个框架没用到)

```
wget https://dev.mysql.com/get/mysql80-community-release-el6-1.noarch.rpm
yum localinstall mysql80-community-release-el6-1.noarch.rpm

yum repolist enabled | grep "mysql.*-community.*"

sudo yum install mysql-community-server

service mysqld start

sudo grep 'temporary password' /var/log/mysqld.log

# passwd: 5!iozIn&t;Vw
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'fighton';


# executing
pip3.6 install -r requirements.txt
# 缺少mysql_config

yum install mysql-devel

python3 manage.py runserver -h 0.0.0.0
```

# reference
1. [Installing MySQL on Linux Using the MySQL Yum Repository](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)
