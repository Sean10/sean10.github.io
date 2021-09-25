---
title: python常用调试
subtitle: python_debug
theme: solarized
revealOptions:
    transition: 'fade'
date: 2020-06-23 20:17:08
updated:
tags: [python, debug]
categories: [专业]
---


# `Python`常用调试方式



----

## pdb

``` bash
python3 -m pdb xxx.py
```

* break 或 b 设置断点	设置断点
* continue 或 c	继续执行程序
* list 或 l	查看当前行的代码段
* step 或 s	进入函数
* return 或 r	执行代码直到从当前函数返回
* exit 或 q	中止并退出
* next 或 n	执行下一行
* pp	打印变量的值
* help	帮助

----

### 批量执行输出

``` bash
(Pdb) commands 1
(com) p some_variable
(com) p another
(com) end
(Pdb) c
# 会打印出some_variable和another
```

----

### 侵入式断点

#### 断点
``` python
import pdb
pdb.set_trace()
```
#### 条件断点
``` python
while a > 0:
    if a == 15:
        print(a)
        pdb.set_trace()
```

---

### 条件断点

``` python
# b(reak) [([filename:]lineno | function) [, condition]]
b temp.py:13, a == 15
```


----

### 多线程调试

pdb原生不支持多线程调试，但是基于这个开发的不少第三方库支持, 可能rpdb\winpdb\Pyringe等有支持多线程？详见下述

* [PythonDebuggingTools \- Python Wiki](https://wiki.python.org/moin/PythonDebuggingTools)

稍后讲下`pycharm`多线程调试

----

## pycharm常用

### 条件断点
![](python常用调试/2020-06-23-15-09-34.png)

----


### 多线程调试
pycharm是开发了pydevd这个插件来完成的远程调试、多线程调试的功能

![](python常用调试/2020-06-23-17-50-47.png)

----


## 健壮的logging机制

对于代码量一大，手动gdb肯定不方便，又不像二进制程序会出现coredump，所以gdb的`backtrace`经常需要用来排查段错误。

日志才是最主要的排查手段，通过等级控制，打开debug日志复现一次，日志足够详细就能直接定位到问题行。如果不够详细，就需要补充了。

----

### Example

``` python
import logging
from logging.handlers import RotatingFileHandler

log_path = "./res/a.log"
logger = logging.getLogger("a")
logger.setLevel(logging.DEBUG)
fh = RotatingFileHandler(log_path, maxBytes=200000, backupCount=7, encoding="utf-8")
fh.setLevel(logging.INFO)
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)
fomatter = logging.Formatter('%(asctime)s\t%(module)s\t%(message)s', '%Y-%m-%d %H:%M:%S')
fh.setFormatter(fomatter)
console.setFormatter(fomatter)
logger.addHandler(fh)
logger.addHandler(console)
```



----

## 远程同步

* vscode: sftp
* pycharm: deployment

---

### vscode:sftp

![](python常用调试/2020-06-23-17-54-19.png)

---

#### Example

``` json
{
    "name": "Profile Name",
    "host": "name_of_remote_host",
    "protocol": "ftp",
    "port": 21,
    "secure": true,
    "username": "username",
    "remotePath": "/public_html/project",
    "password": "password",  
    "uploadOnSave": true
}
```

----


### 基于pycharm的deployment

![](python常用调试/2020-06-23-17-55-52.png)

----

#### 配置

![](python常用调试/2020-06-23-17-56-32.png)

---

## 参考文档
1. [pdb — The Python Debugger — Python 3\.8\.3 documentation](https://docs.python.org/3/library/pdb.html)

