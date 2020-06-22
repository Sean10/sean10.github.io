---
title: 《Effective_C++》note
date: 2017-11-28 22:47:56
updated:
tags: [C++]
categories: [专业]
---

# in-class 初值设定

只支持整型常量

<!--more-->

class static专属变量可以不定义，直接声明并使用

如果要取地址，就在定义文件里再定义const ，不要初始化，因为值在声明时已经默认了。

这不知道是哪个C++的版本加入的功能，旧编译器不支持初值

不支持in class时，可以用`enum{NumTurns = 5}`替代

这叫做enum hack补偿做法

这是template metaprogramming的基本技术

bitwise constness(physical constness) and logical constness

二进制常量性 和 逻辑常量性
利用Mutable解决掉non-static的const摆动场，释放bitwise constness


# 新式转型

static_const转型操作

const_cast去除const操作

core dump错误

基类使用派生类函数
1. 使用virtual从继承向上移动
2. 使用安全容器

绝对拒绝cascading dynamip_cast，理由不理解

尽可能使用C++ style新式转型

无参数构造函数，为什么也需要用到初始化操作？
（噢，可能是为了不忘记哪个参数不需要初值）

不同编译单元内定义之non-local static对象，编译器对初始化顺序并没有设定

**多个编译单元内的non-local static对象经由“模板隐式具现化 implicit template instantiations"”形成

将non-local 移到专属函数中，调用一个函数返回一个reference指向所含的对象，

这是Singleton模式的一个常见实现手法

但是在多线程环境下存在不确定性

做法：在程序的单线程启动阶段手工调用所有reference-returning函数，可消除与初始化有关的”竞速形式(race conditions)

编译器拒绝生成拷贝构造函数和操作符的几种情况

将成员函数声明为private并不去实现，就可以阻止编译器生成public拷贝构造函数等等

声明一个uncopyable基类，继承他就可以直接阻止编译器

c++的异常绝对不要放在析构函数里

连锁赋值

返回*this，这个是返回引用

自我赋值，释放，陷阱

传统做法，添加identity test

异常安全性是什么？

工厂函数factory function

# RAII，资源取得时机便是初始化时机(Resource Acquisition is Initialization)



智能指针

或者 referenced counting smart pointer

# resource handler

shared_ptr存在删除器，可以指定调用函数作为释放这种行为

`std::tr1::shard_ptr<Investor> ptr`
`ptr(m1, Unlock)`

有时APIs直接接触原始资源，无法通过资源管理类来接触

对原始资源的访问最好是通过显式转换，但是对客户来说，隐式转换 可能更加方便

隐式转换也是可以在自定义类中重载的吗？

**以独立语句将newed对象存储于智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露**

# 优良接口

许多客户端错误可以因为导入新类型而获得预防

以函数替换对象，表现某个特定月份

和non-local static对象的初始化次序有什么关系？

嗯，想起来了，构造函数的赋值和初始化次序可能不定，还是直接调用函数效果更确定

tr1::shared_ptr支持定制型删除器，可防范DLL问题，可被用来自动解除互斥锁。

这条不太理解，不知道DLL可能发生什么错误

设计class犹如设计type

函数内的static local对象何时释放？
global slinton

在一个命名空间内添加多个头文件也是一种做法

**特化版本**是什么，好像没见过

条款25，特化swap，感觉不理解，特别复杂

避免返回handles（包括references\指针、迭代器）指向对象内部

# 异常安全性

* 不泄露任何资源
* 不允许数据败坏

异常安全码

copy and swap一般化策略

pimpl idiom

function template和头文件是啥关系，似乎没啥关系

# 条款31 将文件件的编译依存关系降至最低

我们可以这样做，将person分割为两个classes，一个只提供接口，另一个负责实现该接口。

这样可以实现接口与实现相分离

std头文件依旧引用，除此之外。接口使用前置声明。

关键:用“声明的依存性”替换“定义的依存性”

Pass By Value就需要用到定义，而如果传递指针或引用，就不需要定义，只要声明

**为声明式和定义式上提供不同的头文件**

就像<iosfwd>

这个被称为Handle classes.

2种:
* 接口与实现相分离
* 抽象基类，叫interface class

类似Java的Interface

继承这个抽象基类的叫做具象类(concrete classes)，调用基类的创建函数，在创建函数中调用具象类对象动态分配，并返回static，然后通过基类中提供的接口对返回的对象进行操作。哇，这也是一种接口与实现相分离

handle classes 和Interface classes解除了接口和实现之间的耦合关系，从而降低了文件间的编译依存性(compilation dependencies)

virtual意味着接口必须被继承，non-virtual意味着接口和实现必须被继承

转角函数 forwarding function

# 函数接口继承和函数实现继承

virtual和inline究竟有什么关系？

接口和缺省实现应该分开

可以在纯虚函数的定义中进行缺省实现

简朴的(非纯)impure vritual函数具体制定接口继承及缺省实现继承。

缺省实现继承不还是pure virtual函数实现的吗？

难道这个简朴的，指的不是virtual函数？

non-virtual interface(NVI手法)，外覆器(wrapper)



Template Method设计模式

虚函数private，还能被继承吗？

不能的话，那有什么意义？

strategy设计模式

传递函数指针

tr1::function可以实现stategy设计模式

任何兼容的可调用物(callable entity)

与给定之目标签名式(target signature)兼容

上面2条什么意思？

非虚函数，Base类指针指向派生类，执行的函数式Base类中的，而不是派生类中的。

如果是虚函数，那么就都执行派生类中的。

静态绑定，动态绑定

dynamically bound,stataically bound

不能调用继承Private的virtual函数，那么允许重新定义这个虚函数的价值是什么呢？

如果继承的private是非虚函数，也是可以重新定义的吧？

难道这个不行

Java和C#自带 组织derived class 重新定义virtual 函数

空白基类最优化

# 多重继承

virtual inheritance

# 泛型编程

衍生出模板元编程

显式接口，运行期多态

template中指涉一个嵌套丛书类型名称，就必须在紧邻它的前一个位置放上关键字typename

模板 特化

全特化模板的调用很有可能编译器不会去查询，也就无法调用，存在一些问题


共性与变性分析

模板化基类 被覆盖

working set

因非类型模板参数造成的代码膨胀，可以用class来替换template，但是完全理解不了

Traits是一门技术，获取类型信息的，无论是内置还是自动类型

TMP元编程技术越来越牛了

可以定制new和delete，set_new_handler
