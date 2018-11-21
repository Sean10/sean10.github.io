---
title: Python类方法基础
subtitle: python
date: 2018-11-11 23:38:46
updated:
tags: [Python]
categories: [专业]
---


<!--more-->

## __slots__ 限制实例属性

## @property装饰器

## singledispatch函数重载

## __iter__ 类迭代

## __getattr__ 动态 REST SDK

``` python
class Chain(object):

    def __init__(self, path=''):
        self._path = path

    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))

    def __str__(self):
        return self._path

    __repr__ = __str__
```

## enum @unique 迭代类

## __all__

一个由string组成的list, 类似于js的`module.exports = {a, b}`这种吧

## __hash__


## pickle

### __getinitargs__

## __bases__

可以用来修改继承的基类

## 