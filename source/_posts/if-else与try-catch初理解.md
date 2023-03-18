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

就代码风格上差异来说，主要的差异点好像在于以下

### 总结


针对不同类型的业务代码，其实是根据场景都适用的。

* 当业务逻辑中，异常场景不少见的时候，用LBYL处理更合适
* 当业务逻辑中，大部分场景是走的正常分支，只有少部分时候由于环境、网络波动等因素出现异常时，用EAFP更合适。
  * 按python哲学, 是更推荐`EAFP`的, 但是也不排斥利用静态语言的特性做一定程度的防御式编程
  * 也有人更推荐LBYL[^5]

> Python 的动态类型（duck typing）决定了 EAFP，而 Java等强类型（strong typing）决定了 LBYL。语言之间的设计哲学差异，Java 对类型要求非常严格，要求明确，类/方法等，它假定你应该知道，任何时候你正在使用的对象类型，以及它能做什么。相反，Python 的鸭子类型意味着你不必明确的知道对象的显示类型是什么，你只需要关心你在使用时候它能有相应的反馈。在这种宽松的限制下，唯一明确的态度就是认为代码会工作，准备面对结果。
> 这无关语言的好坏，每一门语言都有自己的哲学与态度，正确的对待，理解。

上面这句话说的很对, 对于python来说, 他提供了动态/静态语言的能力, 剩下的取决于开发者自身的运用了.

类型 | 预计成功比例 | 业务逻辑可阅读性 | 存在潜在不可控问题时的处理逻辑 | 预检查成本 | 多线程race condtion 风险
-- | -- | --  | -- | -- |  --
EAFP | 高 | 高(异常拆分恰当) | 省力. 通过父异常统一拦截 | 低 | 无 | 
LBYL | 低 | 高(在python3.6, mypy之后, 参数校验可从业务逻辑中去除) | 费力. 需要一个个异常判定, 工作量大, 风险高| 一般低, 偶尔高  | 有

### EAFP
  * 一旦出现异常，存在性能上较差的问题，但是如果在正常逻辑，倒是没有固定成本
    * 即预期大多数时间会是成功的
  * 业务逻辑代码阅读会非常顺畅，所有的异常判断全部在业务逻辑外
    * 这里应该有一点前提，业务逻辑封装的足够合适，否则每行做异常封装的话，看上去也并不美观
  * 潜在不可控的问题: 如各种 io 操作，包括但不限于网络请求、文件读者、数据库 crud...
  * 预检查成本高: 如对下层产生的压力足以等同于正常请求, 为了性能可能更推荐EAFP.

#### FAQ
* 如果是EAFP，因为未进行各种检查引起的问题，是否会需要回滚？
  * EAFP每个独立的exception针对对应的逻辑其实都应该做到幂等性, 进行独立rollback. 即EAFP的逻辑其实也是要拆分的恰当的, 不能通过一个大的try...catch...全包裹住, 异常
  * 如果拆分的足够合适，做了基本的入参检查之后，代码中在调用其他函数时，进行异常处理就可以. 
    * 建议捕捉下层收到的系统层级异常，然后抛出一个自定义的异常. 而不是将底层异常直接上传. 因为每个逻辑层次的错误代表不同的含义, 如最下层收到的参数的类型错误, 对应上层其实是某个逻辑错误.


### LBYL
  * 固定的检查耗时成本，但是其实消耗不大，因为`if`是最适合编译器和处理器做分支预测优化的部分了*。
    * 仅部分正常校验也会对下层业务产生较大压力时, 性能优化可解
  * 存在多线程`race condition`的潜在问题
  * 预检查会混杂在业务逻辑中
    * 不过这里其实也取决于代码内聚解耦能力, 如果封装的恰当，应该很多逻辑都是在函数入口封装掉，这样的话倒也不会影响到业务逻辑部分的阅读。
    * 参数类型检查, 在python3.6前的开始推行`type annotion`后, 通过`typing`特性的类型标注后, LBYL的参数检查可以不用再显式`isinstance`来判断了, 即解除了一个缺点: `参数检查会混杂在业务逻辑中`的劣势.
  * 预计操作失败的时间占比较高时


#### 静态分析对`Exception`的支持

好像在`mypy`里这个是在计划支持中, 具体进展暂未了解到. 但大致的趋势是2者皆可用. 并不绝对.

## Reference
1. [Python Tips \- 防御性编程风格 EAFP vs LBYL \- 知乎](https://zhuanlan.zhihu.com/p/36167239)
2. [如何理解 EAFP 和 LBYL 两种编程风格的区别？ \- 知乎](https://www.zhihu.com/question/354242039)
3. [方式 1 和方式 2 的却别到底在哪里？ \- V2EX](https://www.v2ex.com/t/620135)
4. [Write Cleaner Python: Use Exceptions](https://jeffknupp.com/blog/2013/02/06/write-cleaner-python-use-exceptions/)
5. [13 – Joel on Software](https://www.joelonsoftware.com/2003/10/13/13/)
6. [Python鸭子类型 duck typing\_Keyboard Interrupt的博客\-CSDN博客](https://blog.csdn.net/weixin_44772030/article/details/106661363)
7. [Python and EAFP principle: any way of differentiating betweeen 2 exceptions of the same type? \- Stack Overflow](https://stackoverflow.com/questions/58936504/python-and-eafp-principle-any-way-of-differentiating-betweeen-2-exceptions-of-t)