# 事务

事务是由一个或一组Sql组成的执行单元，该执行单元只能全成功或全部失败；
其具备四大特性：A（atomicity 原子性）C（consistency 持久性）I（isolation 隔离性）D（durability 一致性）。

锁保证了事务的隔离性，redo Log 保证了事务的原子性和持续性；undo Log保证了事务的一致性；

# redo Log vs undo Log

## redo Log
redo Log 又被称作重做日志，其通常是物理日志；

redo log包括两部分：一是内存中的日志缓冲(redo log buffer)，其是易失性的；二是磁盘上的重做日志文件(redo log file)，其是持久的。
> InnoDB是事务的存储引擎，其通过 Force Log at Commit 机制来实现事务的持久性，即当事务提交时（Commit）时，必须先将该事务的所有日志写入到重做日志文件进行持久化，待事务的Commit操作完成才算完成。

![mysql架构](/images/mysql/innodb_logbuffer.png)

![mysql架构](/images/mysql/innodb_redologbuffer.png)



## undo Log
undo Log用来帮助事务回滚及MVCC，其通常是逻辑日志
