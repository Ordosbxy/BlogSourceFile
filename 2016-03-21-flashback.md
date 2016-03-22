---
title: flashback
date: 2016-03-21 17:32:01
tags:
  - flashback
  - 闪回
  - Oracle
categories:
  - Oracle数据库
---

![](http://7xpzxw.com1.z0.glb.clouddn.com/Oracle-flashback_pic.png)

Oracle的闪回操作

- 闪回查询
  - 闪回查询
  - 闪回版本查询
  - 闪回事务查询
- 闪回数据
  - 闪回表
  - 闪回删除
  - 闪回数据
  - 闪回归档

下面会分别介绍这些操作。在介绍这些操作之前先看下闪回特性是否开启。

## 检查闪回特性是否启用

参考资料：[打开或关闭oracle数据库的闪回功能步骤 ](http://blog.itpub.net/26194851/viewspace-763582/)

确认数据库闪回特性已经启用:v$database

```
SQL> select flashback_on from v$database;

FLASHBACK_ON
------------------
NO
```

如果闪回特性没有启用，则需要先启用闪回。

打开闪回的步骤：

```
SQL> shutdown immediate;
SQL> startup mount;
SQL> alter database flashback on;
SQL> alter database open;
```

打开之后可再次检查闪回特性是否打开。只要打开了闪回特性，就可以进行闪回操作。

## 闪回查询

查询某一个历史时间点的数据。

**查询某个时间点表中的数据——某个时间点表中快照的数据。**

```
SQL> create table t(x int , y int);

表已创建。

SQL> insert into t values(1,2);

已创建 1 行。

SQL> insert into t values(2,3);

已创建 1 行。

SQL>
SQL> commit;

提交完成。

SQL> select * from t;

         X          Y
---------- ----------
         1          2
         2          3

SQL> set time on
21:11:45 SQL> delete from t where x=1;

已删除 1 行。
21:12:11 SQL> commit;

提交完成。

21:12:18 SQL> select * from t;

         X          Y
---------- ----------
         2          3

21:12:21 SQL> select * from t as of timestamp to_timestamp('2016-03-21 21:11:45'
,'yyyy-mm-dd hh24-mi-ss');

         X          Y
---------- ----------
         1          2
         2          3
```

可见删除的数据已经提交，但是还是可以闪回查询到之前时间的数据。其中hh24表示可以用24小时制，否则只能小时不能超过12。至于为什么分钟用mi而不用mm，那是因为规定的格式就是mi，换成mm会显示和之前的月份mm冲突，换成其他的会显示日期格式无法识别。

**基于SCN的闪回查询**

如果有修改时候的SCN，那么就可以基于SCN进行闪回查询

```
21:18:34 SQL> select * from t;

         X          Y
---------- ----------
         2          3

21:20:36 SQL> select current_scn from v$database;

CURRENT_SCN
-----------
    2891887

21:20:53 SQL> insert into t values(3,4);

已创建 1 行。

21:21:13 SQL> commit;

提交完成。

21:21:16 SQL> select * from t;

         X          Y
---------- ----------
         2          3
         3          4

21:21:26 SQL> select * from t as of scn 2891887;

         X          Y
---------- ----------
         2          3
```

可见基于SCN闪回查询得到插入数据之前表中的数据

## 闪回版本查询

**闪回版本查询**也就是flashback versions query。

- See all versions of a row between two times
- See transactions that changed the row

一个时间段内的某一行的所有版本，以及改变该行的所有事务。



## 闪回事务查询

**闪回事务查询**也即是flashback trasaction query

- See all changes made by a transaction

闪回一个事务对数据的所有改变。




## 闪回表

将表闪回到历史的某个时刻。



## 闪回删除



## 闪回数据库



## 闪回归档