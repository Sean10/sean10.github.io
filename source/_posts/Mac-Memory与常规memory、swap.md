---
title: mac OS memory与常规memory、swap
date: 2018-05-08 23:45:18
updated:
tags: [Memory, OS]
categories: [专业]
---

在跑Tensorflow的时候，在Mac上没有出现的内存耗尽导致的dead问题，出现在了ubuntu上，本以为会和mac一样用硬盘来进行交换内存，而不只是使用默认设置的swap空间，结果并没有如想象的那样。

<!--more-->

mac OS的activity monitor中的Memory提供了好多名词

## 简要
* Physical Memory: 实际RAM使用的
* Memory Used:
	* App Memory: APP使用的内存量，包含其虚拟内存和物理内存
	* Wired Memory: 系统核心占用的，此内存中的信息无法移动到硬盘，因此必须保留在 RAM 中。联动内存的大小取决于当前使用的应用程序
	* Compressed: RAM中被压缩的部分,从而让出一些RAM的空间给其他进程，
* Cached Files: 最近使用的内存被临时存储，在使用其他应用前再次打开会直接从这里访问，加快访问速度
* Swap Used: 硬盘中用来交换RAM中无用内容的大小

## 内存详细
* Real Memory Size: 被进程使用的实际内存
* Virtual Memory Size: 进程空间大小，64bit内存地址空间，所以会很大，不重要
* Shared Memory Size: 表示已经被加载到物理内存中的进程私有数据所对应的虚拟内存地址空间大小。
* Private Memory Size: 表示已经被加载到物理内存中的进程私有数据所对应的虚拟内存地址空间大小。

## 比较

在操作系统书中讲述的内存管理，其实mac OS并没有与他有太大差别，现在理解来看大致思想还是相近的。

看下mac OS是在哪个部分能够出现30G占用这种情况，是swap空间没有设置上限吗？

![](https://ws1.sinaimg.cn/large/006tKfTcly1fr5g16dvejj30m30gcafa.jpg)

看下面的swap大小与上面的virtual memory size的确对上了，的确是swap空间够大的原因……而检测过程中，占用大量内存的问题所在是tensorboard,暂时只好手动导出数据用matplot画了。

# Reference
1. [Apple Activity Monitor Instrument
](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Instrument-ActivityMonitor.html)
2. [Mac 的 memory 和 real size memory 有什么区别](https://www.zhihu.com/question/36188357)
3. [Mac OS 内存管理知识](http://elf8848.iteye.com/blog/1373854)
4. [Memory terminology in Mavericks Activity Monitory](https://apple.stackexchange.com/questions/107578/memory-terminology-in-mavericks-activity-monitory)
5. [How to use Activity Monitor on your Mac
](https://support.apple.com/en-us/HT201464)
