---
title: python日志模块初探
subtitle: tech
date: 2021-08-13 19:28:12
updated:
tags: [log, python, library]
categories: [专业]
---

# 标准
支持的日志等级

`SyslogHandler`的`syslog`协议部分主要遵循`RFC5424`.

在安装了`systemd`和`rsyslog`的设备上, 如果启用了`syslog`协议, 日志一般会先经由`systemd-journald`管控的`/dev/log`套接字, 由`rsyslog`从`systemd-journald`的`socket`中读取日志, 而后根据`rsyslog`的过滤器写入到`/var/log/messages`中.






# 支持能力
* 基于`filter`实现的`handler`和`logger`, 支持等级控制
* 基于`name`的`logger`字典

* 仅支持多线程, 不支持多进程

## 限制
> * 日志紊乱：比如两个进程分别输出xxxx和yyyy两条日志，那么在文件中可能会得到类似xxyxyxyy这样的结果
> * 日志丢失：虽然读写日志是使用O_APPEND模式，保证了写文件的一致性，但是由于buffer的存在（数据先写入buffer，再触发flush机制刷入磁盘），fwrite的操作并不是多进程安全的
> * 日志丢失的另一种情况：使用RotatingFileHandler或者是TimerRotatingFileHandler的时候，在切换文件的时候会导致进程拿到的文件句柄不同，导致新文件被重复创建、数据写入旧文件

# 使用说明
## formatter可用属性

[logging \-\-\- Python 的日志记录工具 — Python 3\.9\.6 文档](https://docs.python.org/zh-cn/3/library/logging.html#logrecord-attributes)

## handler
## SyslogHandler

## StreamHandler


# 场景
## 作为组件, 整个组件使用一个日志等级, 控制基本的输出等级.

```


```

## 对日志性能存在要求的日志组件

## 作为公有库时的日志记录, 跟随对应的入口日志等级进行控制



# 自定义能力
## LogRecord

## extra参数

## Handler
根据目标的需求, 自定义开发handler.



# Reference
1. [Python面试官：聊聊多进程场景下Logging模块？ \- 知乎](https://zhuanlan.zhihu.com/p/361058730)
