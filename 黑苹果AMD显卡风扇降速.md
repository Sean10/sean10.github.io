---
title: 黑苹果AMD显卡风扇降速
subtitle: hacintosh
date: 2020-05-28 23:14:10
updated:
tags: [hacintosh, AMD]
categories: [随想]
---

旧文补发

在第一次尝试黑苹果的时候，按照大佬们的经验之谈，配置一个A卡，这样配置核显做硬解就能充分利用起显卡的性能来了。

但是很可惜的是，在做教程的时候，这些大佬们没提到关于温控的问题。



<!--more-->
## 方案1

对于黑苹果来说，A卡独显的风扇默认是无法通过软件控制到的，即便注入了驱动，也只能控制机箱风扇。

对于A卡的风扇来说，需要单独定制操作。翻遍黑锅论坛，发现有大佬开发了针对AMD Vega64显卡风扇的温控操作。而rx系列就没有了……

在搜索过程中，发现除了直接软件接管控制之外，似乎还有一种直接写入到驱动的方法，不过这个难度可能稍大，也可能挺费时间……

我姑且选择了拔掉独显，单纯使用集显驱动了。但是方案姑且记录在这里吧。

## 方案2
>如果你在 Windows 下使用过 AMD 显卡驱动程序，则你一定对于 AMD Watt Man 有印象。这是随驱动程序安装的用于用户自定义性能模式的工具。你可以调整风扇转速方案，设定核心和内存频率以及电压。其实质就是生成一个自定义的 Power Table 供 GPU 加载使用以控制其行为模式。如果我们可以将这个 Power Table 设定好并抽取出来，然后注入到 macOS 对于该显卡的驱动当中，就可以完成对于显卡行为的自定义控制。[^2]

embed the Soft Power Table in the kext.

上面两句是关键，根据上面两句来看，我之前构思的，能够切换到win10把显卡的bios里设置的参数改了，用北极星编辑器和flash工具刷进去，然后切回mac来使用的思路，是不是不太可行呢。因为从上面来看，这些参数可能实际是在驱动加载的时候生效的，而不是在硬件上。但是从刷bios这一个动作来看，应该也是存在于bios中的。可能这是2种方式吧。根据[^3]来看，的确是有2种方式了。



## Reference
1. [MacOS – AMD’s Radeon Adrenalin Software \| GPU, Monitor & Peripherals](https://egpu.io/forums/gpu-monitor-peripherals/macos-amds-radeon-adrenalin-software/)
2. [Knock me down, and I’ll keep getting back up \- 知乎](https://zhuanlan.zhihu.com/p/62844147)
3. [【心得】MSI ARMOR RX580 8G 降壓降頻心得 @電腦應用綜合討論 哈啦板 \- 巴哈姆特](https://forum.gamer.com.tw/C.php?bsn=60030&snA=511548)