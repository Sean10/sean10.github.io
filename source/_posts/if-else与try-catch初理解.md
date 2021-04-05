---
title: if...else(LBYL)与try...catch(EAFP)初理解
subtitle: code
date: 2020-06-24 01:00:36
updated:
tags: [编程范式, code, paradigm]
categories: [专业]
---

这两种其实在`python`里属于防御性编程下属的两种风格

* if...else属于LBYL(Look before you leap)
* try...catch属于EAFP(easier to ask for forgiveness than permission)

就目前的理解上来说，主要的差异点好像在于以下

* EAFP
  * 一旦出现异常，存在性能上较差的问题，但是如果在正常逻辑，倒是没有固定成本
  * 业务逻辑会非常顺畅，所有的异常判断全部在业务逻辑外
    * 这里应该有一点前提，业务逻辑封装的足够合适，否则每行做异常封装的话，看上去也并不美观
  * 各种 io 操作，包括但不限于网络请求、文件读者、数据库 crud...
* LBYL
  * 固定的检查耗时成本，但是其实消耗不大，因为`if`是最适合编译器和处理器做分支预测优化的部分了*。
  * 存在`race condition`的潜在问题
  * 参数检查会混杂在业务逻辑中
    * 不过这里也有一个疑惑点，如果封装的恰当，应该很多逻辑都是在入参检查处，这样的话倒也不会影响到业务逻辑部分的阅读。


所以，针对不同类型的业务代码，其实是根据场景都适用的。

* 当业务逻辑中，异常场景不少见的时候，用LBYL处理更合适
* 当业务逻辑中，大部分场景是走的正常分支，只有少部分时候由于环境、网络波动等因素出现异常时，用EAFP更合适。
  * python官方是更推荐`EAFP`的
  * 也有人更推荐LBYL[^5]

## Todo

需要考虑的场景
* 如果是EAFP，因为未进行各种检查引起的问题，是否会需要回滚？
    * 还是说，如果拆分的足够合适，做了基本的入参检查之后，代码中在调用其他函数时，进行异常处理就可以了？是否需要捕捉，然后抛出一个自定义的未知异常呢？



## 静态分析`Exception`

好像在`mypy`里支持了静态分析异常?

## Reference
1. [Python Tips \- 防御性编程风格 EAFP vs LBYL \- 知乎](https://zhuanlan.zhihu.com/p/36167239)
2. [\(11 封私信 / 1 条消息\) 如何理解 EAFP 和 LBYL 两种编程风格的区别？ \- 知乎](https://www.zhihu.com/question/354242039)
3. [方式 1 和方式 2 的却别到底在哪里？ \- V2EX](https://www.v2ex.com/t/620135)
4. [Write Cleaner Python: Use Exceptions](https://jeffknupp.com/blog/2013/02/06/write-cleaner-python-use-exceptions/)
5. [13 – Joel on Software](https://www.joelonsoftware.com/2003/10/13/13/)
