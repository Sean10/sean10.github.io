---
title: mongo学习
date: 2018-03-24 13:46:43
updated:
tags: [mongo, noSQL, 数据库]
categories: [专业]
---

面向文档

Key-Value对 > 文档 > 集合 > 数据库

<!--more-->

mongo插入时不执行代码，会先转化为BSON，所以免疫了传统注入攻击

但是mongo执行js代码时，存在被注入的可能性

mongo的操作是异步且不可靠的，无确认

## GridFS
```
mongofiles
```

# 操作记录

``` mongodb
//启动
mongod --port 5506 --fork --logpath mongodb.log

//创建数据库
use todos
这是只是进入了一个暂时为空的数据库，此时还没有创建，只是显示切换到了todos这个数据库而已。接着执行一个插入操作才真的创建了数据库，并创建了一个叫TodoList的Collection，可以在show dbs里看到。
db.TodoList.insert({'id':'1'})

使用上mongo就像是一个写时操作的系统。

// 连接已经在运行的数据库
mongo localhost:26789


//显示所有数据库
show dbs
//显示所有集合
show collections

use blog
// 这个就像select * from db.posts
db.posts.find()

mongo常用变量
```


## 插入ISO时间
new Data() 
db.TodoList.insert({'task':'todo', 'date': new Date()})

使用python插入时，只要插入一个`datetime.datetime`的实例就会自动在mongo shell中显示为ISODate，不需要自己转换为Isoformat[4]

## mongo数据库主键

在写todoList时，一开始想的是数据库里的id主键用自增的数字来表示，但是在查阅解决方式的过程中发现，现在的这种UUID的方式生成才是最适合mongo数据库，不应该将mysql等数据库的习惯带到新的数据库中。

# Reference
1. [Mac OSX下使用Homebrew安装MongoDB](https://iengchen.github.io/2016/06/13/mac-install-mongodb-use-homebrew/)
2. Mongo 权威指南
3. [MongoDB数据表基本操作](https://www.cnblogs.com/libingql/archive/2011/06/09/2076440.html)
4. [Create an ISODate with pyMongo](https://stackoverflow.com/questions/7651064/create-an-isodate-with-pymongo)
5. [MongoDB中如何不使用_id作为主键的简单策略](https://www.oschina.net/question/2269509_224279)
