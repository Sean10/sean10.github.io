---
title: LaTex学习记录
date: 2018-05-04 20:30:59
updated:
tags: [LaTex]
categories: [专业]
---

<!--more-->

L T E X 忽略命令后面的空格。如果你希望在命令后面得到一空格，可以

在命令后面加上 {} 和一个空格，或者加上一个特殊的空白距离命令。{} 将

A 阻止 L T E X 吞噬掉命令后面的空格。

I read that Knuth divides the people working with \TeX{} into \TeX{}nicians and \TeX perts.\\ Today is \today.

I read that Knuth divides the people working with T E X into T E Xnicians and T E Xperts. Today is 8th March 2003.

许多命令需要一个参数（parameter）并用一对大括号（curly braces）{ }

将其括起来置于命令名称的后面。 也有一些命令支持用方括号（square A brace）括起来的可选参数。下面的例子中使用了一些 L T E X 命令，并将在 以后对他们进行解释。

----------------------------


# 编译过程

``` latex
xelatex main.tex
bibtex main.aux
xelatex main.tex
open -a preview main.pdf
```

# Reference

1. [](https://blog.csdn.net/u014466412/article/details/53282615)
