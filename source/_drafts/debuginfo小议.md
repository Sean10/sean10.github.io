---
title: debuginfo小议
subtitle: tech
date: 2023-06-17 15:26:38
updated:
tags: [ceph, debug]
categories: [专业]
---


# 背景
主要是想要针对cpu消耗高时, 能够定位到什么逻辑占用的

# debuginfo区分

主要根据perf支持的模式来说

主要有`fp`, `dwarf`, `lbr`
* dwarf
	* 使用 DWARF Call Frame 信息来 unwind 堆栈
* fp
	* 使用帧指针方法。根据编译器优化，如使用 GCC 选项 `--fomit-frame-pointer` 构建的二进制文件，这可能无法取消验证堆栈。
* lbr
	* 使用 Intel 处理器上的最后分支记录硬件

`-fomit-frame-pointer`会导致无法获取到函数地址



# perf原理
perf主要是依托于event 事件进行埋点捕获


`/proc/kallsyms`代表内核函数地址

> ### 使用kallsyms_lookup_name()查找函数或变量的虚拟地址

> 因为有`kallsyms`的存在可以使得内核可以在运行过程中随时获得一个符号地址对应的符号名。而内核代码中可以通过 `printk("%pS\n", addr)` 打印符号名

`System.map` 是编译内核时生成的，它记录了内核中的符号列表，以及符号在内存中的虚拟地址。这个文件通过 `nm` 命令生成，具体可参考内核目录下的 `scripts/mksysmap` 脚本

[linux\-kallsyms \|](https://breezetemple.github.io/2017/12/08/linux-kallsyms/)




> perf使用更多是CPU的PMU计数器，PMU计数器是大部分CPU都有的功能，它们可以用来统计比如L1 Cache失效的次数，分支预测失败的次数等。PMU可以在这些计数器的计数超过一个特定的值的时候产生一个中断，这个中断，我们可以用和时钟一样的方法，来抽样判断系统中哪个函数发生了最多的Cache失效，分支预测失效等。

[在Linux下做性能分析3：perf \- 知乎](https://zhuanlan.zhihu.com/p/22194920)

> 由于perf和内核关联，所以理论上，你用哪个内核，就应该使用对应内核的perf，这能保证接口的一致

# FAQ

## gdb能读到, 而perf读不到, 只有函数地址的原因?


## 如何识别perf捕获不到符号表的原因? 比如verbose等模式?



## 指定用户态的`cycles:u`采集到的, 相比不指定, 会少哪些?

是否主要是我理解的内核态的事件?