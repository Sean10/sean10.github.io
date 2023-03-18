---
title: 正则表达式BRE/ERE/PRE
subtitle: tech
date: 2020-05-16 23:53:18
updated:
tags: [re, 正则表达式, linux]
categories: [专业]
---

## 概述
在开发过程中，经验遇到使用不同语言不同工具时，正则表达式无法直接复用的问题。主要原因在于不同语言使用的正则表达式的标准也是不一样的。

<!--more-->

* Basic Regular Expression
* Extended Regular Expression
* Perl Regular Expression

## 主要区别[^1]

流派 | 说明                          
-----|------------------------------
BRE | () {} + ? \|都必须转义使用
ERE | 元字符不必转义, + ? ( ) { } \|可以直接使用
PRE | 除了ERE支持的之外，还支持\d,\s等元字符

BRE、ERE可以使用`POSIX`字符集来操作

## 使用支持

### grep
支持BRE\ERE\PRE，通过参数控制，默认BRE, `-P`开启PRE, `-E`开启ERE

### sed
支持BRE\ERE，默认BRE，`-r`开启ERE

### awk
支持ERE，默认ERE。


### C

c语言的regex根据`man 3 regex`看到的内容来看，支持BRE和ERE.

另外由于C语言字符串处理的性质，一般使用元字符时，需要额外的转义符，如\\s

### Python

python使用的是PRE，所以处理起来要相对C要简单不少

## 疑问
* 为什么search和findall同时存在呢?
* findall输出的内容里, 匹配输出过的字符串位置不会再作为输入被读取了呢?
  * 当我需要严格的全部匹配结果的时候...应该怎么做呢?

# 常用

### 不包含指定字符串所在行, 即过滤,取反能力

不包含`kuebelet`的行
```
^((?!kubelet).)*$
```

### 零宽断言


在`aaa`后面的字符串, 不捕获aaa.
```
(?<=aaa).*
```


## Reference
1. [正则表达式基础知识\|Jerkwin](https://jerkwin.github.io/2014/04/03/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)