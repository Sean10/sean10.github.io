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

找到了俊宁大佬自己的作业，从那里的实例来进行操作


Qt dialog传递参数
https://blog.csdn.net/xzh_blue/article/details/51490747

现在很奇怪，有时候能够充值上去，有时候就报这个错误，导致失败

```
[INFO] send message: {"define":2}
[INFO] receive message: {"define":3,"username":"ls1"}
[INFO] Get user balance request comes
[INFO] send message: {"balance":0,"define":1}
SQL error: 'UNIQUE constraint failed: OrderInfo.type' at 'insert into OrderInfo(type,amount,out_account,in_account,record_time) values (4,25,'cash','ls1',1536938826);'
[INFO] Someone offline, now 0 conne
```

C/S架构下， 登录过程中 密码 是否需要加密？

出现了一个权限值为25，似乎是哪里传输的时候顺序出错了？把转账的值写入到了权限的位置？

[QT 点击自定义QDialog类"确定"按钮 , 模态框立刻关闭 , 之后又做空值检查问题解决 \- CSDN博客](https://blog.csdn.net/wenyun_kang/article/details/64439517)

mac 自带openssl 0.98，而官方最新版 1.1

[\(6 条消息\)UUID是如何保证唯一性的？ \- 知乎](https://www.zhihu.com/question/34876910)

现在有一个问题，我的tableView没有释放过standardItem，因此一定会出现内存泄露问题

[\(6 条消息\)互联网中TCP Socket服务器的实现过程需要考虑哪些安全问题？ \- 知乎](https://www.zhihu.com/question/30507539)

引用openssl库的方式

clang client.c -o client -I/usr/local/opt/openssl/include -L/usr/local/opt/openssl/lib -lcrypto -lssl

像上面这样加上最后那4条就可以正常运行了

openssl生成过程

```
mkdir misc
cd misc
sudo cp /usr/local/etc/openssl/openssl.cnf ./
cp /usr/local/etc/openssl/misc/* ./

//这里生成的是rsa的ca.key和 .rnd 输入密钥是fighton
openssl genrsa -out ./private/ca.key -rand ./private/.rnd -des 2048

// 这里确认了密钥，然后要求输入下面那些内容
openssl req -new -x509 -days 3650 -key ./private/ca.key -out ./private/ca.crt -config openssl.cnf

<!-- Enter pass phrase for ./private/ca.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ZH
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Hikvision
Organizational Unit Name (eg, section) []:bjyf
Common Name (e.g. server FQDN or YOUR name) []:sean10
Email Address []:sean10reborn@gmail.com -->


// 这个应该是显示一下证书
openssl x509 -in ./private/ca.crt -noout -text


## 
// 产生server证书

openssl genrsa -out ./private/server.key 1024

openssl req -new -key ./private/server.key -out ./newcerts/server.csr -config openssl.cnf
<!-- You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ZH
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:bjyf
Organizational Unit Name (eg, section) []:hik
Common Name (e.g. server FQDN or YOUR name) []:sean10
Email Address []:sean10reborn@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:helloworld
An optional company name []:hik -->


touch index.txt
touch serial

echo 00 > serial


openssl ca -in ./newcerts/server.csr -cert ./private/ca.crt -keyfile ./private/ca.key -config openssl.cnf -policy policy_anything -out ./certs/server.crt


openssl genrsa -out ./private/proxy.key 1024
//这步要是产生错误，请看后面的解决方法
openssl req -new -key ./private/proxy.key -out ./newcerts/proxy.csr -config openssl.cnf
openssl ca -in ./newcerts/proxy.csr -cert ./private/ca.crt -keyfile./private/ca.key -config openssl.cnf -policy policy_anything -out ./certs/proxy.crt
openssl x509 -in ./certs/proxy.crt -noout -text

```

[基于OpenSSL自建CA和颁发SSL证书 \| Sean's Notes](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)


Qt 如何 引入 openssl库

[QT总结第3篇：如何在QT中添加\.lib，\.dll还有\.h文件 \- CSDN博客](https://blog.csdn.net/alspwx/article/details/12649225)

现在在编译时，ui_widget.h丢失？

奇怪了。将lssl等加到LIBS这里之后，server可以编译了，但是client又出现了上面的问题，明明这次client一行代码都没改……

终于发现了，我在socket.h里多导入了一个curve.h这个包，这个包里的一个函数qblog和 Qt 的一个包 重名了


case中无法定义同一个变量

用if else 代替 switch 语句;

2：在case中用{}将代码括起来,这样在{}中就能定义变量了;

3：如果变量在各个case中都要用的话,就把变量定义在switch外面吧;

目前在switch case中定义的变量无法被后续的代码发现，从而使用

可能是作用域的原因？

想要手动定义，但是无法通过typeid .name 打印出来

还需要找到办法手动定义

openSSL [OpenSSL主配置文件openssl\.cnf \- 骏马金龙 \- 博客园](https://www.cnblogs.com/f-ck-need-u/p/6091027.html)

[基于OpenSSL自建CA和颁发SSL证书 \| Sean's Notes](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)

[使用c语言实现在linux下的openssl客户端和服务器端编程 \- 欢跳的心 \- 博客园](https://www.cnblogs.com/etangyushan/p/3679457.html)

[openssl编程轻松入门（含完整示例）\-飞月\-51CTO博客](http://blog.51cto.com/mooon/909932)

[\(6 条消息\)用 C\+\+ 写 HTTPS 客户端和服务器大体步骤有哪些？ \- 知乎](https://www.zhihu.com/question/41717174)

[OPENSSL编程入门学习 \- 骑着蜗牛逛世界 \- 博客园](http://www.cnblogs.com/LittleHann/p/3741907.html)

用Cross Functional Vertical就可以画图了

https://www.processon.com/view/5938c757e4b036140a0ef5ef?fromnew=1

https://www.processon.com/diagrams/new#temp-system