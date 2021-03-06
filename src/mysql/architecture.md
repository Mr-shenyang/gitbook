# 1 MySQL概述

![mysql架构](/images/mysql/architecture_view.png)

* 1 连接器  
  负责跟客户端建立连接、获取权限、维持和管理连接。
  >需要注意的是：由于验证通过之后连接器会获取该用户的权限，之后这个连接里的所有权限判断逻辑都依赖于此权限，因此，一个用户成功建立连接后，即使管理员对这个用户的权限做了修改，也不会影响已经存在连接的权限，修改完后，只有再新建的连接才会使用新的权限设置。

* 2 查询缓存  
  以(key,value)的形式存储执行过的语句和结果。在解析一个查询语句之前，如果查询缓存是打开的，那么MySQL会优先检查这个查询是否命中查询缓存中的数据。
  
  查询缓存是从4.1 版本开始支持，默认是关闭的，从8.0版本该模块被移除
  
  > 一般情况下，不建议使用查询缓存，这是因为对于表一个的更新操作，这个表上所有的查询缓存都会被清空。
  > 因此除了很少更新的配置表外可以使用查询缓存来提供查询速度，其他的一般不建议使用查询缓存。
  
  关于查询缓存的更详细信息可以参考文档[MySql  查询缓存笔记](https://segmentfault.com/a/1190000003039232)

* 3 分析器  
  包括两段**词法分析**和**语法分析**
  这个阶段会检查：  
  1）是否使用了错误的关键字；  
  2）使用的关键字顺序是否正确；  
  3）检查数据表和数据列是否存在等  

  我们经常看到的“You have an error in your SQL syntax”提示，就是在这个阶段判断出来的错误。(ps对于这类错误，重点关注near 后面的即可)

* 4 优化器  
  一般情况下，一条查询可以有多种执行方法，最后都是返回相同结果。优化器的作用就是找到这其中最好的执行计划。  
  MySQL使用基于成本的查询优化器。它会根据统计信息和代价模型预测一个查询使用某种执行计划时的成本，并选择其中成本最少的一个。
  
  >查询优化器中有两个依赖：统计信息和代价模型。统计信息的准确与否、代价模型的合理与否都会影响优化器选择最优计划。

* 5 执行器  
  通过调用存储引擎定义好的API，操作存储引擎，并将结果返回给客户端
* 6 存储引擎  
  在存储引擎层，MySQL采用插件的形式，支持多种存储引擎，不同的存储引擎，功能也不同，自从MySQL5.5后InnoDB成为默认存储引擎。

# 2 查询流程简述

![mysql架构](/images/mysql/select-process.png)

# 3 更新SQL执行过程

MySql的更新和查询在总体流程上相差不多，不同的是更新操作伴随着数据的变更，需要进行日志记录。

MySql的变更日志有两种，一种是innodb引擎记录的日志，另外一种是server层日志。

## 3.1 redo log

试下想，如果每次更新操作都有，先找到相关行，然后更新在写入磁盘。整体的IO消耗将会非常的大。所以InnoDB引入了redo log并配合Mysql框架的binlog来解决这里的性能消耗问题。

redo log包括两部分：一是内存中的日志缓冲(redo log buffer)，其是易失性的；二是磁盘上的重做日志文件(redo log file)，其是持久的。
> InnoDB是事务的存储引擎，其通过 Force Log at Commit 机制来实现事务的持久性，即当事务提交时（Commit）时，必须先将该事务的所有日志写入到重做日志文件进行持久化，待事务的Commit操作完成才算完成。

![mysql架构](/images/mysql/innodb_logbuffer.png)

![mysql架构](/images/mysql/innodb_redologbuffer.png)

### 3.2 undo log

### 3.3 binlog

### 3.4 日志小结

1. redo log是InnoDB引擎特有的；binlog是Mysql server层实现的，所有引擎都可以使用；
2. redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog是逻辑日志，记录的是这个语句上的原始逻辑。
3. redo log是循环写的，空间固定可能用完；binlog是可以追加写入，不允许被覆盖、删除。

## 3.4 更新流程简述

这里以 update T set c = c+1 where ID = 2 这条sql语句简单梳理下MySQL的更新过程

1. 执行器先找引擎取ID=2的这一行；
2. 执行器拿到引擎给的行数据，把这个值加1，得到新的数据，在调用引擎接口写入新数据； 
3. InnoDB引擎将数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare状态。然后告知执行器执行完成了，事务随时可以提交； 
4. 执行器生成这个操作的binlog，并把binlog写入磁盘； 
5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成提交（Commit）状态，更新完成。

![mysql架构](/images/mysql/update-process.png)

redo log被分成prepare和commit“双阶段提交”。

# 4 Q/A

* 1.Q:使用长连接时，MySQL可能内存涨的特别快，这是为什么？如何解决?  
    A:
    原因：Mysql在执行过程中临时使用的内存是管理在连接对象中的，这些资源会在连接断开时才释放
    解决办法：  
      1)定期断开长链接；  
      2)5.7以后版本提供了mysql_rest_connection命令来初始化连接资源。这个过程不需要重连和权限验证
  
* 2. 