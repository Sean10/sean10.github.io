---
title: python单元测试框架比较
subtitle: test
date: 2020-06-02 15:00:38
updated:
tags: [python, pytest]
categories: [专业]
---

旧文简单整理
<!--more-->
## 背景
主要测试框架: unittest，和在unittest基础上开发而来的pytest和nose\nose2

## 定位
### pytest

>The pytest framework makes it easy to write small tests, yet scales to support complex functional testing for applications and libraries.


### nose2
>It’s unittest with plugins.
>nose2’s purpose is to extend unittest to make testing nicer and easier to understand.
nose2的主要目的是扩展Python的标准单元测试库unittest，因此它的定位是“带插件的unittest”。

>## nose2 vs pytest
>nose2 may or may not be a good fit for your project.
> If you are new to python testing, we encourage you to also consider pytest, a popular testing framework.
官方对于新使用单元测试的更推荐使用pytest。


#### 上一个大版本功能
nose（已进入维护阶段，不再开发新功能）


## 功能
### 共同点
都是基于unittest原生库开发而来。

### 差异点
详见[\[转\]Python测试框架对比\-\-\-\-unittest, pytest, nose, robot framework对比 \- bonelee \- 博客园](https://www.cnblogs.com/bonelee/p/11122758.html)[^5]

基于测试功能上来说，均具有。

总体来说，pytest入门、扩展性最强，nose2比较要求写单元测试的功底的样子？


#### 社区生态
* pytest
  * 6K+的star数量，518个贡献者。
  * 名字中含有pytest的python库 5308个
* nose2
  * 625个star, 58个贡献者。
* nose[已进入维护阶段，不再开发新功能]
  * 1.3K的star, 70个贡献者
  * 名字中含有nose的python库，707个。

## 结论

毫无疑问，用py.test就完事了。

## Reference
1. [pytest\-dev/pytest: The pytest framework makes it easy to write small tests, yet scales to support complex functional testing](https://github.com/pytest-dev/pytest)
2. [nose\-devs/nose2: The successor to nose, based on unittest2](https://github.com/nose-devs/nose2)
3. [三种最流行的Python测试框架，我该用哪一个？ \- 知乎](https://zhuanlan.zhihu.com/p/68088736)
4. [【pytest】（二） pytest与unittest的比较 \- 知乎](https://zhuanlan.zhihu.com/p/87206256)
5. [\[转\]Python测试框架对比\-\-\-\-unittest, pytest, nose, robot framework对比 \- bonelee \- 博客园](https://www.cnblogs.com/bonelee/p/11122758.html)
6. [pytest vs nose detailed comparison as of 2020 \- Slant](https://www.slant.co/versus/9149/9150/~pytest_vs_nose)