---
title: python单元测试mock和stub小记
subtitle: mock
date: 2020-06-23 21:06:11
updated:
tags: [mock, python, unittest]
categories: [专业]
---

## 背景

单测主要遇到需要`mock`数据的场景目前遇到比较多的是两类
* HTTP接口
* 对底层业务逻辑API

HTTP接口相关的mock工具非常多了，暂时就不具体说了。这篇主要记录的是针对依赖的API的mock。

在python单元测试数据mock有两座山，一个是`unittest.mock`，一个是`pytest`的`mockeypatch`。当然，还有一些`easymock`之类的库，但是没去研究，暂时搁置。

很多文章里提到[Test Double at XUnitPatterns\.com](http://xunitpatterns.com/Test%20Double.html)，好像TDD开发中的概念，这里做了非常详细的划分。

不过，主体上其实对数据的模拟主要是2点。
* mock
* stub

## mock与stub
* mock主要用于测试对象的行为验证
* stub主要用于提供模拟数据


## mock使用

虽然上面的理论上是这样说，但是对于python的`unittest`库里，并没有按照这两个来分开实现，统一都是叫做`mock.patch`。

不过使用的方法上的确有差异
```
# mock
mock_instance.assert_called_with

# stub
mock_instance.return_value = 'xxx'
mock_instance.side_effect = {'a': 'xxx'}
```
`mock`应该主要指的是检查函数的调用、异常的触发情况，而`stub`主要就负责提供我们要的模拟数据

### 具体使用方式
主要使用的是`pytest`及`pytest-mock`，`pytest-mock`提供了一个`mocker`的`fixture`。

PS. `fixture`是pytest的一个封装，将一个函数封装，作为全局参数使用，在被写入某些函数的参数时，这些被封装的函数会执行一遍（对于像`patch`这类修改运行时的操作，能够保留）

#### patch基本使用
```

```

#### patch调用封装的接口
a使用b模块中提供的接口，而单元测试脚本patch掉b模块，手动封装b模块会提供的数据

``` python
# a.py
from b import B

class A:
    def output(self):
        b = B()
        print(b.get_data())

# b.py
class B:
    def __init__(self):
        pass

    def get_data(self):
        return "B"

# test_a.py
from pytest_mock import mocker

def test_A_output(mocker):
    # Important!!这里要注意，你要patch掉的内容是在这个模块被导入的地方，而不是这个模块定义的地方
    
    inst = mocker.patch("a.B")
    inst.get_data().return_value = "A"
    a = A()
    a.output()
    # A
```

上面要注意的地方，原因主要在于模块使用时的运行时上下文，在这个模块被导入前，这个模块是不存在于代码的运行时中，只有当这个模块被导入了，这个模块才存在于运行时，patch的操作才能生效。除非是在你的当前文件中，这样定义和导入默认都已经完成了，就不会遇到这个问题了。

### 样例

#### return_value
下面所有的`mock`都可以用上面提到的`mocker`替换

``` python
# mymodule.py

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import os.path

class RemovalService(object):
    """A service for removing objects from the filesystem."""

    def rm(filename):
        if os.path.isfile(filename):
            os.remove(filename)
            return True
        return False

# test_mymodule.py
from mymodule import RemovalService

import mock
import unittest

class RemovalServiceTestCase(unittest.TestCase):
    
    @mock.patch('mymodule.os.path')
    @mock.patch('mymodule.os')
    def test_rm(self, mock_os, mock_path):
        # instantiate our service
        reference = RemovalService()
        
        # set up the mock
        mock_path.isfile.return_value = False
        
        ret = reference.rm("any path")
        assert ret == False
        
        # test that the remove call was NOT called.
        self.assertFalse(mock_os.remove.called, "Failed to not remove the file if not present.")
        
        # make the file 'exist'
        mock_path.isfile.return_value = True
        
        ret = reference.rm("any path")
        
        mock_os.remove.assert_called_with("any path")
        assert ret == True

```
#### side_effect
##### 根据入参返回不同的结果
``` python
def my_side_effect(*args, **kwargs):
    if args[0] == 42:
        return "Called with 42"
    elif args[0] == 43:
        return "Called with 43"
    elif kwargs['foo'] == 7:
        return "Foo is seven"

temp = mock.patch("myModule.mod")
temp.get_value.side_effect = my_side_effect
print(temp.get_value(42))

# 或者直接用匿名函数
my_side_dict = {
    "42": "Called with 42",
    "43": "Called with 43"
}
temp = mock.patch("myModule.mod")
temp.get_value.side_effect = lambda x: my_side_dict[x]
print(temp.get_value(42))

```
##### 测试是否返回异常
``` python
myMethod = Mock(side_effect=KeyError('whatever'))
with self.assertRaises(KeyError):
    myMethod()
```

## Reference
1. [浅谈mock和stub \| 忆桐之家的博客](http://hongyitong.github.io/2016/12/23/%E6%B5%85%E8%B0%88mock%E5%92%8Cstub/)
2. [unittest\.mock — mock object library — Python 3\.8\.3 documentation](https://docs.python.org/3/library/unittest.mock.html) 
3. [Monkeypatching/mocking modules and environments — pytest documentation](https://docs.pytest.org/en/latest/monkeypatch.html)
4. [Python Mocking: A Guide to Better Unit Tests \| Toptal](https://www.toptal.com/python/an-introduction-to-mocking-in-python)
5. [简述软件开发中的单元测试 \| 无人之岛](https://blog.woodcoding.com/%E6%B5%8B%E8%AF%95/2019/03/31/first-to-touch-unittest/)