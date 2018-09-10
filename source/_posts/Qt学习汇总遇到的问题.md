---
title: Qt学习汇总遇到的问题
date: 2017-06-15 16:42:46
tags: [Qt, C++]
categories: 专业
---

使用qt信号槽时，始终no such slots，原因是我没有在槽函数的类中使用Q_OBJECT MACRO
这个的原因是什么呢？
在类声明的开始位置必须加上Q_OBJECT语句，这条语句是不可缺少的，它将告诉编译器在编译之前必须先应用 moc工具进行扩展
这个时候先执行qmake，再build

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

qmake project 变量使用


一个pro生成多个app似乎现在不行了，只能通过subdirs来实现了？

通过$$PWD来导入公用包


用CLion的Cmake和 命令直接clang++都没办法把sqlite3成功编译通过，但是用Qt就能过，真的是奇特。


sqlite3 使用怪怪的

select * from sqlite_master;

始终得到结果为空

然后报错

libc++abi.dylib: terminating with uncaught exception of type std::__1::system_error: SQL logic error

CREATE TABLE main.table_name(
   column1 datatype  PRIMARY KEY(one or more columns),
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
);

现在尝试用回俊宁大佬的库，但是连创建table都使用不了……