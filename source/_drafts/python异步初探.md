---
title: python异步初探
date: 2018-04-17 17:55:57
updated:
tags: [python, 异步, 协程]
categories: [专业]
---

在多线程爬虫学习时，性能提升没有那么明显，就想试试传说中的协程。

<!--more-->

在用aiohttp时，试着修改代理时，发现他不支持https代理，贡献者说到，会出现使用https代理的原因只是单纯我们不理解代理的工作模式。的确如此。

唉，暂时由于ddl将近，并且aiohttp运行失败，还是用回最初的多进程版本了。


# Reference
1. [How to use HTTPS proxy with aiohttp?](https://github.com/aio-libs/aiohttp/issues/845)
