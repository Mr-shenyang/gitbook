
MySQL里面的锁大致可以分成全局锁、表级锁和行锁三类。

# 全局锁

全局锁就是对整个数据库实例加锁。
典型使用场景：做全库逻辑备份。
命令：

```SQL
Flush tables with read lock
```

# 表级锁

MySQL里面表级别的锁有两种：一种是表锁，一种是元数据锁（meta data lock，MDL)。

## 表锁

命令：

```SQL
lock tables t1 read, t2 write;
```

## 元数据锁（meta data lock，MDL)

MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，保证读写的正确性。

# 行锁

MySQL的行锁是在引擎层由各个引擎自己实现的。
