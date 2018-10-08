---
title: [译]sqlite为什么autoincrement不推荐使用
subtitle: sqite
date: 2018-09-15 22:51:18
updated:
tags: [sqlite3, C++, Qt]
categories: [专业]
---

在使用sqlite3写一个建议订单系统的时候，没设置主键，默认将第一个条属性设置成了主键，而这条属性并不具有唯一性，导致经常出现插入失败的问题。

在这个背景下，我考虑设置一个数据库里的自增主键，之后这部分就可以自动生成了。

但是在我使用的[ORM_LITE]()中没有提供这样的属性，那么是不是这个属性不推荐使用呢？

找到了下面这篇文章[SQLite AUTOINCREMENT : Why You Should Avoid Using It](http://www.sqlitetutorial.net/sqlite-autoincrement/)

<!--more-->

下文是这篇的翻译。

## SQLite ROWID表简介
无论何时，创建表时不指定`WITHOUT ROWID`选项，都会得到一个名为rowid的隐式自动增量列。

rowid列存储64位有符号整型，用于唯一标识表中的行。

我们来看下面的例子。

首先，创建一个包含两列`first_name, last_name`的新表`people`:

``` sql
CREATE TABLE people (
 first_name text NOT NULL,
 last_name text NOT NULL
);
```
[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#1)

其次，用以下`INSERT`语句插入`people`一行:
``` sql
INSERT INTO people (first_name, last_name)
VALUES
 ('John', 'Doe');
```
[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#2)

第三，用以下`SELECT`语句从`people`中查询数据
``` sql
SELECT
 rowid,
 first_name,
 last_name
FROM
 people;
```
[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#3)

![](http://www.sqlitetutorial.net/wp-content/uploads/2015/12/SQLite-AUTOINCREMENT.jpg)

所以，SQLite会自动创建一个名为rowid的隐式列，并 在您插入新行时自动分配一个整数值。

可以通过`rowid`的两个别名`_rowid_`和`oid`来使用它

如果创建具有`INTEGER PRIMARY KEY`列的表，则该列指向`rowid`列。

以下语句删除`people`表并重新创建它，不过这一次，我们添加另一列带有`INTEGER PRIMARY KEY`属性的名为`person_id`的列。

``` sql
DROP TABLE people;
 
CREATE TABLE people (
 person_id INTEGER PRIMARY KEY,
 first_name text NOT NULL,
 last_name text NOT NULL
);
```

[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#4)

现在`person_id`列实际上就是`rowid`列。

那么，Sqlite如何分配一个整型值给rowid列呢？

如果插入一个新行时，你没有指定一个rowid值或者使用`NULL`值， Sqlite会分配一个比表中最大的rowid大1的整型。 当还没有插入任何行时， rowid是1。

首先，插入带有最大值的一行插入`people`表。

``` sql
INSERT INTO people (
 person_id,
 first_name,
 last_name
)
VALUES
 (
 9223372036854775807,
 'Johnathan',
 'Smith'
 );
 ```

 [Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#5)

 ![](http://www.sqlitetutorial.net/wp-content/uploads/2015/12/SQLite-maximum-rowid-value.jpg)

 第二，插入不指定`person_id`的另一行

 ``` sql
 INSERT INTO people (
 first_name,
 last_name
)
VALUES
 (
 'William',
 'Gate'
 );
 ```

[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#6)

![](http://www.sqlitetutorial.net/wp-content/uploads/2015/12/SQLite-INSERT-row-without-rowid.jpg)

## SQLite AUTOINCREMENT 属性

SQLite 推荐你不应该使用`AUTOCREMENT`属性，因为:

>The AUTOINCREMENT keyword imposes extra CPU, memory, disk space, and disk I/O overhead and should be avoided if not strictly needed. It is usually not needed.
>
>AUTOINCREMENT关键字会产生额外的CPU,内存,磁盘空间和磁盘I/O开销，如果不是严格需求，应该避免使用。通常不是必须的。

此外，SQLite为`AUTOCREMENT`列分配值的方式与`rowid`列的使用方式略有不同。

请参阅以下示例。

首先，再次删除并重新创建人员表。这次，我们使用`AUTOINCREMENT`属性。

``` sql
DROP TABLE people;
 
CREATE TABLE people (
 person_id INTEGER PRIMARY KEY AUTOINCREMENT,
 first_name text NOT NULL,
 last_name text NOT NULL
);
```

[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#7)

其次，将具有最大行id值的行添加到people表中。

``` sql
INSERT INTO people (
 person_id,
 first_name,
 last_name
)
VALUES
 (
 9223372036854775807,
 'Johnathan',
 'Smith'
 );
```

[Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#8)

第三，在people表中插入另一行

``` sql
INSERT INTO people (
 first_name,
 last_name
)
VALUES
 (
 'John',
 'Smith'
 );
 ```

 [Try it](http://www.sqlitetutorial.net/tryit/query/sqlite-autoincrement/#9)

 这次，SQLite发出了一条错误消息：

 ```
[Err] 13 - database or disk is full
 ```

 因为它不会重新使用未被使用数字。

 `AUTOCREMENT`这个属性的主要目的是*防止SQLite复用还未使用的值或者使用先前删除的行*

 如果还没有任何像这样的需求，那么你就不应该在主键中使用`SQLite AUTOINCREMENT`属性。

 在这篇教程中，你已经学习到`AUTOINCREMENT`属性如何运作的，以及它如何影响`SQLite`分配值给主键的方式。

 # Reference
 1. [SQLite AUTOINCREMENT : Why You Should Avoid Using It](http://www.sqlitetutorial.net/sqlite-autoincrement/)


