---
title: C++远程开发&&跨平台Sourcetrail
subtitle: C++_remote_develop_and_sourcetrail
date: 2020-04-24 10:28:13
updated:
tags: [C++, Clion, Sourcetrail, Index]
categories: [专业]
---

## Clion配置

Clion支持基于cmake的远程开发，最近用来在windows上开发linux程序时的高亮、提示、补全。相对以前开发时很多unix特定的头文件无法提示出来，现在要好得多了。

基本配置方式就是在`perference->Build\Execution\Deployment->toolchains`里添加remote debug环境，也可以配置gdb server做远程调试。

如果新引入了动态库，需要重新让clion去下载一下依赖的库建立索引的时候，点击`Tools->Resync with Remote Hosts`重新同步就可以了。

<!--more-->

## Sourcetrail阅读代码

另外，配合去年开源的Sourcetrail来看代码感觉也不错，调用图索引的也还可以。而且sourcetrail在建立C/C++的索引的时候，也可以使用cmake导出的一个cdb成果物。

``` bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS ..
```

编译时增加上述选项就可以导出compile_commands.json这个文件，包含代码src路径和include路径、编译选项等等，用这种方式建立索引的时候就不会报错了。但是好像大部分时候看代码时也不是那么需要某些头文件的提示，缺失也影响不大。

## 跨平台sourcetrail建立索引[^2]

但是在开发的时候大部分代码其实主要还是linux环境偏多，并不兼容MinGW的时候，就容易出现错误文件数量过多，影响阅读的时候。针对这种情况，issue里也给出了一个方法，suorcetrail提供了command line功能，可以在编译环境中， 完成索引，然后再把生成的数据库文件拖回本机打开工程，就能直接看到代码了。

1. 可以用图形界面直接创建一个sourcetrail工程(得到.srctrlprj文件)，然后将在编译环境中的源码路径或者是上面提到过的compile_commands.json的路径给填上。
2. 把这个srctrlprj文件放到编译环境中，在编译环境中执行`Sourcetrail index <path/to/your/project.srctrlprj>`
3. 把生成的srctrldb文件拖回本地srctrlprj文件所在目录，用Sourcetrail打开即可。

## 还是太卡了
* OpenGrok + Universal CTags
* understand

## Reference
1. [Support CMake files for project setup · Issue \#35 · CoatiSoftware/Sourcetrail](https://github.com/CoatiSoftware/Sourcetrail/issues/35)
2. [Remote indexing · Issue \#134 · CoatiSoftware/Sourcetrail](https://github.com/CoatiSoftware/Sourcetrail/issues/134)
