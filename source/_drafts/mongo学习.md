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
# 插入ISO时间
new Data() 
db.TodoList.insert({'task':'todo', 'date': new Date()})


```

```

# Reference
1. [Mac OSX下使用Homebrew安装MongoDB](https://iengchen.github.io/2016/06/13/mac-install-mongodb-use-homebrew/)
2. Mongo 权威指南
3. [MongoDB数据表基本操作](https://www.cnblogs.com/libingql/archive/2011/06/09/2076440.html)
