---
title: redis/SQL存储网站数据
subtitle: redis
date: 2018-12-04 22:55:40
updated:
tags: [redis]
categories: [专业]
---


## 主要功能
* 当前在线用户数
* 平均访问时长
* 今日在线用户最高数以及相应的时间
* 最近15min浏览次数
* 近30日平均浏览次数、平均访问时长
* 总PV\UV,月PV\UV

## 实现方式
### 当前在线用户数
通过redis的expire机制采用心跳包进行统计，在线用户数可以通过set来记录

### 访问时长
恐怕需要一个关系数据库来进行存储了，记录访问的时间，和redis中sessionid失效的时间作为退出时间。

## 今日在线用户最高数以及相应的时间
每分钟计算一次，存储到数据库中

## UV、PV
可以通过存储在SQL中的访问时长来进行统计UV, 和 PV


## Reference
