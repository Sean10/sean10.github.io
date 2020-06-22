---
title: virtualwrapper管理环境
date: 2018-03-23 22:12:05
updated:
tags: [python, virtualenv]
categories: [专业]
---

在看工程源码时，要配置python2.7环境，这个时候发现一直用的virtualenvwrapper建立环境后就无法切换到2.7来安装环境了。

一开始找了好久，以为需要比如pyenv+virtualenv之类的多个工具才能管理python版本和环境。

结果找啊找发现，virtualenvwrapper就支持这个要求。

```
mkvirtualenv --python python2.7 envname
```

不过在官方doc里倒是没能搜索到这个功能，只有`mkvirtualenv --help`才能看到这个功能……

为什么把这里的help里的文字放到官方搜索里都搜不到…………

# Reference
1. [使用virtualenv搭建独立的python环境](http://blog.51cto.com/qicheng0211/1561685)
