---
title: Python设计模式初探
subtitle: POF
date: 2018-11-17 17:02:32
updated:
tags: [Python]
categories: [专业]
---

## 单例模式

### 不支持并发
``` python
class Singleton(object):
    _instance = None
     def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls._instance
    
if __name__ == '__main__':
    s1 = Singleton()
    s2 = Singleton()
    assert id(s1)==id(s2)

```

上面这个，为什么要调用super创建object对象呢？这样创建的对象还是Singleton对象吗。

测试了一下，为什么这里使用super创建的对象依旧是Singleton对象呢？

噢，是不是因为当前这个类的__new__对象已经被重写了，要创建对象只能调用object父类里的__new__对象。

而且，原本默认的__new__内的实现就是调用object里的__new__吧？通过传入cls参数进行制定类型

### 并发

``` python
class Singleton(object):
    objs = {}
    objs_locker = threading.lock()
    def __new__(cls, *args, **kwargs):
        if cls in cls.objs:
            return cls.objs[cls]
        cls.objs_locker.acquire()
        try:
            if cls in cls.objs:
                return cls.objs[cls]
            cls.objs[cls] = object.__new__(cls)
        finally:
            cls.objs_locker.release()
```

### Borg模式

``` python
class Borg:
    __shared_state = {}
    def __init__(self):
        self.__dict__ = self.__shared_state

```

## 工厂模式

### 反射


## Mixin模式

## 代理模式

### 装饰器



## 状态模式

## 工厂模式

