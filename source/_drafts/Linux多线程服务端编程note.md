---
title: Linux多线程服务端编程note
date: 2018-04-04 22:25:46
updated:
tags: [Linux, 多线程, C++]
categories: [专业]
---

# 线程安全的对象生命期管理

## 同步原语（synchronization primitives）
事件是一种同步原语，
具有下述操作：
* wait
* set
* clear

基本的进程线程同步互斥控制机制
1. 临界区 通过对多线程的串行化来访问公共资源或一段代码，速度快，适合控制数据访问。
2. 互斥量 为协调一起对一个共享资源的单独访问而设计的。
3. 信号量 为控制一个具备有限数量用户资源而设计。
4. 事件 用来通知线程有一些事件已发生，从而启动后继任务的开始

<!--more-->

### 临界区（Critical Section）

>确保在某一时刻只有一个线程能访问数据的简便办法。在任意时刻只允许一个线程对共享资源进行访问。假如有多个线程试图同时访问临界区，那么在有一个线程进入后其他任何试图访问此临界区的线程将被挂起，并一直持续到进入临界区的线程离开。临界区在被释放后，其他线程能够继续抢占，并以此达到用原子方式操作共享资源的目的。

临界区包含两个操作原语： EnterCriticalSection（） 进入临界区 LeaveCriticalSection（） 离开临界区

### 互斥量（Mutex）

>互斥量跟临界区很相似，只有拥有互斥对象的线程才具备访问资源的权限，由于互斥对象只有一个，因此就决定了任何情况下此共享资源都不会同时被多个线程所访问。当前占据资源的线程在任务处理完后应将拥有的互斥对象交出，以便其他线程在获得后得以访问资源。互斥量比临界区复杂。因为使用互斥不但仅能够在同一应用程式不同线程中实现资源的安全共享，而且能够在不同应用程式的线程之间实现对资源的安全共享。

互斥量包含的几个操作原语：
*　CreateMutex（） 创建一个互斥量
*　OpenMutex（） 打开一个互斥量
*　ReleaseMutex（） 释放互斥量
*　WaitForMultipleObjects（） 等待互斥量对象

### 信号量（Semaphores）

>信号量对象对线程的同步方式和前面几种方法不同，信号允许多个线程同时使用共享资源，这和操作系统中的PV操作相同。他指出了同时访问共享资源的线程最大数目。他允许多个线程在同一时刻访问同一资源，但是需要限制在同一时刻访问此资源的最大线程数目。在用CreateSemaphore（）创建信号量时即要同时指出允许的最大资源计数和当前可用资源计数。一般是将当前可用资源计数配置为最大资源计数，每增加一个线程对共享资源的访问，当前可用资源计数就会减1，只要当前可用资源计数是大于0的，就能够发出信号量信号。但是当前可用计数减小到0时则说明当前占用资源的线程数已达到了所允许的最大数目，不能在允许其他线程的进入，此时的信号量信号将无法发出。线程在处理完共享资源后，应在离开的同时通过ReleaseSemaphore（）函数将当前可用资源计数加1。在任何时候当前可用资源计数决不可能大于最大资源计数。

## 对象构造线程安全

>唯一要求是在构造期间不要泄露this指针




## 进程间通信

首先，我一般是通过一个中间层，比如redis、数据库等来实现的。但是理论上原生在一般场景下是足够使用了的才对。

首先通过python实现IPC通信方式, socket\pipe\IPC（这个C里有，这里也应该有把？）


### PIPE

PIPE还是有比较大的局限性的，这里可以看到只能是单向的。单工场景。

另外，这里可以看到进程间的PIPE主要依赖fork机制建立，对于完全没有关联的两个进程能不能使用PIPE通信，这个问题我还有点疑问。应该是可以的，默认管道是匿名管道，但是实际上是有命名管道的。

``` python
import multiprocessing

def proc1(pipe):
    pipe.send("hello")
    print ("proc 1 : ",pipe.recv())

def proc2(pipe):
    print ("proc 2 : ",pipe.recv())
    pipe.send("hello ,too")
    print("{}".format(pipe.recv()))

pipe=multiprocessing.Pipe()

p1=multiprocessing.Process(target=proc1,args=(pipe[0],))
p2=multiprocessing.Process(target=proc2,args=(pipe[1],))

def main():
    p1.start()
    p2.start()

    p1.join()
    p2.join()

if __name__ == '__main__':
    main()

```

### Queue

这个也是基于PIPE的实现，只是支持了全双工的形式，似乎。

### Listener和 Client

在python的multiprocessing.connection包中有Listener和Client类可以实现多进程之间的通信，这种通信方式根据平台的不同会自动选择socket或者named pipe的方式来实现通信。

这里发现，进程间通信的同步，果然是必要条件。

``` python
#server 

#!/usr/bin/env python
# encoding: utf-8
__author__ = 'outofmemory.cn'
from multiprocessing.connection import Listener

address = ('localhost', 6000)     # family is deduced to be 'AF_INET'
listener = Listener(address, authkey=b'secret password')

while True:
    conn = listener.accept()
    print('connection accepted from', listener.last_accepted)
    while True:
        data = conn.recv().decode()
        print("get {}".format(data))
        if data == 'close':
            conn.close()
        try:
            result = "hello"
            # result = ','.join(list(jieba.cut(data,False)))
            print("send {}".format(result))

            conn.send_bytes('get {}'.format(data).encode())
        except Exception as e:
            print(e)

listener.close()
```


``` python
#client
#encoding=utf-8
__author__ = 'outofmemory.cn'

from multiprocessing.connection import Client

address = ('localhost', 6000)

conn = Client(address, authkey=b'secret password')

for x in range(0, 50):
    conn.send('这是一个美丽的世界'.encode())
    print(conn.recv_bytes().decode())

conn.send('close'.encode())

conn.close()

```

# 事件循环

要实现一个协程的库，首先需要一个事件循环、一个上下文的表示，一个上下文的切换。

事件循环是为了用来监听如select和epoll这类东西的。

## 从内核的角度

好吧，似乎协程也叫做用户态线程

如go的开发者开发的libtask，早期C中使用的

实模式下的变化才能，就是只有一个进程，这种京城下，就是手动触发中断，然后系统保存



## 用户态线程

这个主要其实是glibc的clone完成的，在这个clone的传入flag参数中，允许用户指定要共享的空间，

[snippet/linux\-system\-programming\.md at master · xgfone/snippet](https://github.com/xgfone/snippet/blob/master/snippet/docs/linux/program/linux-system-programming.md)

# Reference
1. [《Linux多线程服务端编程》]
2. [C++中四种进程或线程同步互斥的控制方法](https://blog.csdn.net/zhu2695/article/details/51148272)
3. [C++ 并发编程（二）：Mutex（互斥锁）](https://segmentfault.com/a/1190000006614695)
4. [C++11 并发指南三(std::mutex 详解)](http://www.cnblogs.com/haippy/p/3237213.html)
