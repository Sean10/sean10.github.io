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

TODO: 单进程同步和异步io, 差异?


# async

> Running code in executor means to run it in OS threads.[^OS_threads]

[^OS_threads]:[python \- Why doesn't asyncio always use executors? \- Stack Overflow](https://stackoverflow.com/questions/53260688/why-doesnt-asyncio-always-use-executors)

# future

> Future相对于async, await的最大优势在于它提供了强大的链式调用，链式调用优势在于可以明确代码执行前后依赖关系以及实现异常的捕获

# Reference
1. [How to use HTTPS proxy with aiohttp?](https://github.com/aio-libs/aiohttp/issues/845)
