---
title: C++_Note
date: 2017-02-10 22:49:18
updated:
tags: [C++]
categories: [专业]
---

使用qt信号槽时，始终no such slots，原因是我没有在槽函数的类中使用Q_OBJECT MACRO
这个的原因是什么呢？
在类声明的开始位置必须加上Q_OBJECT语句，这条语句是不可缺少的，它将告诉编译器在编译之前必须先应用 moc工具进行扩展
这个时候先执行qmake，再build

<!--more-->

开了一个refresh信号的线程，但是一使用emit signalRefresh就爆出系统错误
>:-1: error: symbol(s) not found for architecture x86_64
>:-1: error: linker command failed with exit code 1 (use -v to see invocation)

这个错误经常会出现

---

c++跨文件或跨类传输变量参数是通过在对应文件声明类，然后调用该类获得参数的函数

但是qt里我一旦这样做，就会出现上面那个系统错误。

---

普通形参加不加const限定符对实参没有影响，引用形参和指针形参前面没有const限定符时，实参必须是非const的，而前面有const限定符时对实参也没有什么影响。

为什么会出现这种情况？

原因在于实参的传递方式不同，函数中的形参是普通形参的时，函数只是操纵的实参的副本，而无法去修改实参，实参会想，你形参反正改变不了我的值，那么你有没有const还有什么意义吗？引用形参和指针形参就下不同了，函数是对实参直接操纵，没有const的形参时实参的值是可以改变的，这种情况下怎能用函数来操纵const实参呢。

----

ui文件似乎必须要建一个类通过setupUI才能使用
主要需要新建一个dialog来show就可以了

------
Qt的模态对话，setModal和dialog.exec() == box::accepted有什么区别？

----

QT的close没有反应，似乎要使用事件循环

---

setAttribute (Qt::WA_DeleteOnClose);
这个明明是解决内存泄露问题，为什么可以关闭窗口呢，而close却做不到

---

qVector输入之后，造成implicitly  deleted because base class "QObject" has a deleted copy constructor

---

Qt 资源文件，可以通过添加qrc来实现路径的操作
但是    //路径这里存在问题，为什么添加了qrc之后依旧不需要前缀就可以使用了呢？
明明理论上说要":/db/user.db"
但只是"user.db"就可以使用了


----

始终不能调用数据库，原因是我没有使用query.finish，以及建表语法出现了错误。

----

事件似乎和信号有点相似，又有点不同。

----------

事件是一个新的概念，与信号不一样。

----
又遇到了当初的link错误，没有详细信息的那种
SF的答案是这个，感觉比较靠谱，我也确实有在Compile Output里看到问题，是client.o的问题，不过这个文件我都没有引用，为什么会错呢

Posting comment by N1ghtLight as an answer.

Duplicate symbols found error is a linker error, which says that linker has found more than one symbols with the same name. Following are some common causes for this.

You have written a function definition in a header file (outside a class), which is included in two or more cpp files.
You have defined a static variable twice.
You have written a function definition twice in a cpp file.
You can find out what are the duplicate symbols by checking Compile Output tab in Qt Creator

---------
把databaseControl设置继承QObject就出现编译不过的问题

就是上面的link错误，不过更复杂

---

遇到这个错误error: member function not viable this argument has type const
但是我一时忘了怎么解决的了

-----

call to implicitly-deleted copy constructor

/Users/sean10/Qt/5.8/clang_64/lib/QtCore.framework/Headers/qlist.h:461: error: call to implicitly-deleted copy constructor of 'centralAirConditioner'
                current->v = new T(*reinterpret_cast<T*>(src->v));
                                 ^ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
遇到这个错误，发现又是因为继承了基类QObject的原因

-----

暂时设计了从multithread发送信号到databaseControl,暂时没能测试出数据是否发送成功

----
好像知道为什么发送了信号，但槽函数没有执行的原因了，似乎是因为信号和槽不在一个线程里
但是默认是会执行异步的啊

上面的猜想是错误的，实际的问题是在于我把UserRegister写在mainwindow的构造函数里的，导致还没有执行connect就把信号已经发送出去了，所以这个信号浪费了

----
稍后添加对于开关机的容错措施
数据锁尚未添加

---

优先设置spinBox的lowtem会导致workTemperature变成这个lowTem,明明是满足他的约束的
噢，可能是因为spinBox一开始是0，没有设置，当有了约束之后，自动变成了18，满足了值变动，所以导致初始化被覆盖了。

---

tcp粘包问题：
怎么分也需要双方组织一个比较好的包结构，所以一般可能会在头加一个数据长度之类的包，以确保接收。

----
从控机修改工作模式，但没有发送给主控机的过程，这个问题有待修正
这里应该根据文档来写：从控机没有调整工作模式的机会，只能由主控机设定

---

connectToHost似乎是mac的问题，大大的windows执行就没有问题。

---

找到每次修改完工作温度，室温就顺便同步的原因了，是因为信号问题，每发送一个SignalSendRequest就执行了9次SendRequest.
但是没有找到有多次connect的函数啊。

即便是用了uniqueConnection还是执行了多次。

找到问题了，有另外一个函数不停发送了signal，定时发送

----

第二次从控机开机，主控机上没有再显示出那台从控机状态？
why?

还有解决粘包问题
---

把室温用电子时钟模式展示出来


----

可以使用doxygen来进行根据注释生成文档

/**
 * 要输出成文档的注释
 */

或者
//!

---

C++在类头文件中试图定义QVector的成员变量，提示错误，不能理解。


std::vector<int> numList[9];

头文件这样定义成员变量，但是在构造函数中不能对他初始化。

不管是QVector还是vector都不行。

问题的回答是这么说的：

>这是定义，不是声明。良好的C++规范要求，除了类定义、inline函数定义、const常量定以外，头文件内不应该放置其他定义。
>
>其次：猜测一下，你这是在试图定义(或声明)类的成员变量么？
>
>建议：补充一下C++基础。

找到原因了，成员函数内不能定义，毕竟只是一个声明.
在主函数内就可以了

----


用QVector需要先进行resize,就像分配内存一样？因为QVector我不能直接定义一个81个单元的Vector。


------

为什么重定向输入，结果读取不了呢？明明从控制台输入就可以。


-------

使用macro宏定义时，始终没能正确替换表达式

现在虽然会了用clang编译选项，但是还是非常不对。

结果居然是因为我宏用了小写……导致了错误，我居然一直没发现。

------

tmp[] = {9,9};
//vector_2 = tmp;
vector<int> vector_2(tmp, tmp+1);

这样就报出了
```
code.cpp:158:9: error: expected expression
    tmp[] = {9,9};
        ^
code.cpp:158:13: error: expected expression
    tmp[] = {9,9};
            ^
2 errors generated.
```

奇怪，用`vector<int> vector_2(begin(tmp), end(tmp));`
同样也有这个问题，但是上面那个赋{1}的单项的就没有问题。

发现问题了，只能初始化的时候直接赋列表值，之后就只能一个个赋，或者再定义一个了。

------

发现segmentfault的原因了，传入指针时，得先定义或者new一下，不然传入赋完值，定义的指针位置什么都没变，就得不到你要的结果了。

---------


C++11添加的raw字符串

C++允许省略C中的结构体的struct语句

C++11支持数组、字符串、结构体列表初始化

c++仅当使用可变参数时函数参数使用……，而C在函数原型中使用可以使用空格()，C++就必须至少是(……),但是更推荐句首说的规则。

有一层const关系时，可以通过非const数据的指针来修改const类型的指针指向的非const类型的数据。

函数指针中(*pf)居然与pf在使用时等价，天哪~

这种函数引用操作的意义是什么？怎么感觉和在函数中使用没什么区别呀？
噢，可以省去写重载几个方法的过程，直接传入参数和使用的方法就可以返回结果。

c++11的auto(auto似乎只是自动类型推断，出了错还是程序员的问题)类型转换和显式、隐式类型转换区别在哪里？

如何传递参数给引用调用时，这个参数是个表达式时，会报错，因为这样会把引用的参数被赋值成那个为了计算表达式而自动建立的临时变量

但是用const引用即可，可以保证这个引用不会被临时变量修改，又可以创建临时变量来计算表达式。

c++11有了 右值引用，可以接受常量了，据说是用来实现移动语义，(move semantics)

返回引用的作用，传统返回，会先把返回的结构复制到一个临时位置，再复制给要赋值的变量，而引用的返回就省去临时位置了。(但是如果这个返回的是该函数中的一个临时变量，这样就会越界，程序出错了，因为该临时变量已经被释放了，却还要从这个位置调用变量)

PS:返回引用的函数实际上是被引用的变量的别名。

在使用引用返回时时，如果不使用const，会导致该函数成为左值，可以被赋值，导致错误

重载运算符时省略const是个好方法，但是一般还是使用const更好。

继承有另一个特征，基类引用可以指向派生类对象，而不用进行强制类型转换。

一个基类作为形参的函数，可以输入这个基类的派生类作为实参！！！

棒！

默认参数

函数多态和函数重载，似乎不太一样
一个函数的多个形式，是长什么样呢？据说是同一回事，但是通常使用函数重载

在检查特征标时，编译器将引用和类型本身视为同一个特征标。

将非const变量作为实参赋给const变量是合法的。

返回类型可以不同，但是特征标也必须不同。

编译器会跟踪重载函数，进行名称修饰或者名称矫正

模板里的typename是C++98添加的，在这以前都是用class的，所以能用typename还是用typename吧
template需要声明也需要定义

模板重载

模板局限性，有些类型的操作不支持
### 显示具体化

具体化优先于常规模板，而非模板函数优先于具体化和常规模板。
`template<> void Swap<job>(job &, job &);`


实例化和具体化

隐式实例化，就是使用模板生成函数定义

现在支持了显式实例化，
`template void Swap<int>(int, int);`

显式实例化可以将原本不匹配的模板函数调用，进行了强制类型转换以后进行调用。

。

以上统称为具体化。
相同之处在于，它们表示的都是使用具体类型的函数定义，而不是通用描述。

选择最佳重载函数时
1. 完全匹配
2. 提升转换(char/short到in, float到double)
3. 标准转换(int to char,long to double)
4. 用户定义的转换，如类声明中定义的转换

术语“most specialized”并不一定意味着显示具体化，指编译器推断使用哪种类型时执行的转换最少。

这个属于C++98增加的函数模板的部分排序规则

不知道模板函数中，应该在声明中使用哪种类型，现在可以用关键字decltype

还有一种函数声明语法（C++11后置返回类型)
```
double h(int x, float y);
变成
auto h(int x, float y) -> double;

使用
template <class T1, class T2>
auto gt(T1 x, T2 y) -> deltype(x+y)
{
    ...
    return x + y;
}
```

c++内存模型

## 存储持续性
### 自动存储持续性
### 静态存储持续性
### 线程存储持续性(C++11)
### 动态存储持续性

内部链接性和外部链接性
const默认内部链接性
可以通过extern扩展

内联函数 似乎不需要满足单定义规则

multable可以让优化

实现可以使用其他语言链接性

new失败返回，std::bad_alloc

new是在堆中建立
p1 = new(buffer) int;在buffer中建立，buffer指定的是静态内存中，不能用delete

## 名称空间
using编译指令和using声明

C++保证函数不会修改变量，
`void Stock::show() const`
声明为const成员变量

用枚举来创建符号常量
可以用static在类声明中定义常量
static const int Mouth = 12;
^
|
像这样

c++11提供了新枚举
``` c++
enum class egg {Small, Medium, Large, Junbo};
enum class t_shirt {Small, Medium, Large, junbo};

```
不过这种不支持隐式类型转换，不能自动变成int，需要显式转换

当构造函数只接受一个参数是，下面这3个是等价的
``` c++
Swtonewt incognito = 275;
Swtonewt incognito(275);
Swtonewt incognito = Swtonewt(275);
```
不过，如果把构造函数声明成explicit，就不能用上面的第一行了，就关闭了隐式自动转换

转换函数：
把类赋给基本类型变量：
``` c++
operator int();
operator double();
```
* 转换函数必须是类方法；
* 转换函数不能指定返回类型
* 转换函数不能有参数

过程中自动应用了类型转换

出现多个转换函数，用隐式存在二义性时，编译器会error

类中的静态存储类
static声明的变量
即便有多个类副本，也只有一个这个变量，是共享的。

C++11提供了2个特殊成员函数：
移动构造函数和移动赋值运算符

c++11空指针nullptr

静态类成员函数，只能用这种方法调用
``` c++
int count = String::HowMany();

```


# 继承
继承is-a关系：公有继承
is a kind of关系

而不是has a 关系

## 多态公有继承

* 虚方法
* 在派生类中重新定义基类的方法

虚函数能够根据指针/引用的对象实际是什么类型而确定选择基类的方法还是派生类的方法

可以使用一个对象的指针数组来保存他和他的派生类，实现一种多态性

虚析构函数也是有作用的，比如保证派生类的析构作用能够生效

静态联编(static binding)

使用了虚函数时，静态联编不足以做到，就需要使用到动态联编

由于效率和概念模型才使用2种联编方式

**如果要在派生类中重新定义方法，才定义虚函数，否则就定义非虚函数**

虚函数的工作原理

析构函数应当是虚函数，

因为基类的析构函数如果不是虚函数，就无法释放在派生类中新添加的对象的内存

友元不能是虚函数

重新定义虚函数，将隐藏基类的方法

不过如果返回类型是基类引用或指针，则可以修改为派生类的引用或指针，叫做covariance of return type

# 抽象基类
ABC概念
使用纯虚函数
包含纯虚函数的对象只能用作基类，无法被定义，声明结尾为0
``` c++
virtual double Area() const = 0;

```

## 继承和动态内存分配

# 友元类

# lambda表达式
``` c++
[](int x) {return x % 3 == 0;}//匿名函数

[](double x)->double{int y = x; return x-y;}//不止一句时要用后置语法，因为只有一句返回语句组成lambda时，自动类型推断才管用。

```

基类动态内存分配，派生类不用，则不用显示定义析构函数、复制构造函数和复制运算符
派生类有，则需要，并且派生类的复制构造函数需要调用基类的复制构造函数（因为没有权限访问基类成员）

c++11添加了可以继承的构造函数，不过默认不开

valarray类

使用公有继承，可以获得接口和实现（基类有纯虚函数只能获得接口）
使用组合，可以获得实现，但没有接口

explicit将禁止隐式类型转换

私有继承：可以直接在成员方法中使用基类的私有变量和方法了
也是has-a关系

也可以同时私有继承多个类

C++11的初始化列表是通过initializer_list<char> il来实现的，之后可以看下

在头文件<memory>中
智能指针，auto _ptr,unique _ptr和shared _ptr，auto在C++11已经被摒弃

throw()也被摒弃，但是这个是什么东西？

unique _ptr强调对象所有权，而shared _ptr则添加引用计数，在最后的计数为0时才释放

unique可用于数组的变体，new []，而其他2个没有

在用<vector>的迭代器的时候，可以用C++11的自动推断，auto pd = score.begin()挺有意思的

for_each(),random_shuffle(),sort()都挺特别的

c++11的循环类似python中的

for(auto x: books) ShowReview(w)

正向迭代器:双向通行算法

迭代器类型只是概念性描述

adapter，一个类或函数，可以将一些其他接口转换为STL使用的接口

看不太懂

就像这样
``` c++
#include <iterator>

ostream_iterator<int,char> out_iter(cout, " ");
```

vector反向打印

可以使用rbegin()的指向超尾的反向迭代器，rend()返回一个只想第一个元素的反向迭代器

插入迭代器

c++11添加的概念，可复制插入，可移动插入

移动构造函数

异常规范方面的修改，noexcept

作用域内枚举

右值引用（完全不记得了）

包装器，std::function<double(char, int)> ef

c++11，atomic原子操作，并行编程

多个专业库
regex支持正则表达式
---

不会写widget，所以先写console版本测试时，发现conio.h不是ANSI C标准的，在mac平台C++中 并不支持。


--
c++

hash函数

hash在C++11有了元编程

``` c++
hash<string> h;
size_t n = h(url);
cout << n;
```

-------------------------------------------------------

c++ stringstream getline作为分隔符相当好用

```c++
while(getline(ss, temp, ','))
{

}

```
