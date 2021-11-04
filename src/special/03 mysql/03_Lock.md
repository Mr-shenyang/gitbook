# lock和latch

在Mysql中关于锁存在两个概念：latch和lock
latch：一般称之为闩锁（轻量级锁），其目的用来保证并发线程操作临界资源的正确性；
lock：其目的事务操作的数据库中对象的正确性；

||lock|latch|
|---|---|---|
|对象|事务|线程|
|保护|数据库内容|内存数据结构|
|持续时间|整个事务过程|临界资源|
|模式|表锁、行锁、意向锁|读写锁、互斥量|
|死锁|通过waits-for-graph、time out等机制进行死锁检测和处理|无死锁检测和处理机制。|
|存在于|Lock manager的哈希表中|每个数据结构的对象中|

latch更底层，一般的数据库锁指的是Lock；我们可以通过命令

```mysql
show engine innodb status;
```

以及information_schema架构中的表INNODB_TRX、INNODB_LOCKS、INNODB_LOCK_WAITS来观察锁信息；

MySQL里面的锁大致可以分成全局锁、表级锁和行锁三类。

# 全局锁

全局锁就是对整个数据库实例加锁。
典型使用场景：做全库逻辑备份。
命令：

```SQL
#加锁
Flush tables with read lock
#解锁
unlock tables
```

# 表级锁
MySQL里面表级别的锁有两种：一种是表锁，一种是元数据锁（meta data lock，MDL)。

## 表锁

命令：

```SQL
#加锁
lock tables t1 read, t2 write;
#解锁
unlock tables
```

## 元数据锁（meta data lock，MDL)

MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，保证读写的正确性。

# 行锁

MySQL的行锁是在引擎层由各个引擎自己实现的。InnoDB存储引擎实现了两种标准的行级锁：

1. 共享锁（S Lock）：允许事务读一行数据
2. 排他锁（X Lock）： 允许事务删除或者更新一行数据；

在InnoDB中读取数据时并不是全部需要读锁的，InnoDB通过多版本控制（MVCC）的方式实现了**一致性非锁定读**。如果需要**一致性锁定读**可通过如下语法实现：

```SQL
#排他锁
select *** for update;
#共享锁
select *** lock in share mode;
```

## InnoDB行锁算法

InnoDB存储引擎有三种行锁算法，其分别是：

* Record Lock：单个行记录上的锁
* Grap Lock：间隙锁，锁定一个范围，但是不包含记录本身
* Next-Key Lock：Record Lock + Grap Lock，锁定一个范围，并且锁定记录本身
