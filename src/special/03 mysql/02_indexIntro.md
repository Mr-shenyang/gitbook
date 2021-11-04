# 01 索引基础

 索引是一组按一个或多个键的值组织的逻辑结构；用于提升性能和确保唯一性；

## 1.1 索引结构简介

哈希表、N叉树、B+树等，很多数据结构都能作为索引结构，每个数据结构都有其不同的适应场景。比如：哈希表这种结构适用于只有等值查询的场景，N叉树和B+树虽然都是树型结构，由于B+树的非叶子节点只保存索引，不保存行记录。所以其具有：1）IO次数更少；2）查询性能更稳定；3）范围查询简便；所以在InnoDb采用的就是B+树作为其数据结构；

## 1.2 B+树结构

//TODO

# 2 InnoDB索引

## 2.1 InnoDB索引模型

InnoDB中索引分成主键索引（又称聚簇索引）和非主键索引（又称二级索引）；它们的差别点顾名思义在于主键索引是由主键构成的索引，每个表都有主键索引；非主键索引是由我们自定义的索引，其叶子节点只会包含主键信息，不会包含具体的行信息；（如下图）：

## 2.2 InnoDB查询过程简析

第一章我们知道Mysql查询过程如下图：
![mysql架构](/images/mysql/select-process.png)
一条SQL查询语句首先经过分析器的词法分析和语法分析，然后由优化器基于成本预测出最佳执行计划，最后交由存储引擎进行查询。
这里就了解下InnoDB存储引擎在查询时做了什么？

这里我们先假定有个表tuser，且在标注插入9条数据，其表结构如下

```sql
CREATE TABLE `tuser` (
  `id` int(11) NOT NULL,
  `id_card` varchar(32) DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `ismale` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `id_card` (`id_card`),
  KEY `name_age` (`name`,`age`)
) ENGINE=InnoDB
```

数据信息如截图：
![tuser_info](/images/mysql/index/tuser_info.jpg)

可以看到其存在聚簇索引`id`和非主键索引`id_card`、`name_age`，下面就以几条sql的形式介绍存储引擎的查询过程；

+ 场景一
  
  ```sql
  select * from tuser where id = 1
  ```
  
  此时会直接命中聚簇索引，并将聚簇索引叶子节点的数据加载到内存中返回给执行器；

+ 场景二

  ```sql
  select * from tuser where id_card = '222'
  ```

  此时会先通过非主键索引`id_card`找到行所在的id信息，然后在通过聚簇索引获取出所在行的全部信息；
  上述这个现象称之为**回表**

>**回表**：定位主键值后，再定位行记录的过程

+ 场景三

  ```sql
  select id,name,age from tuser where name = '张一' and age = 10
  ```
  
  此时通过索引name_age查询就能获取SQL所需的所有列数据，无需回表。
  上述现象称之为**索引覆盖**

>**索引覆盖**：只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表的现象；

+ 场景三
  这个场景我们准备了两条sql分别如下

  ```sql
  select * from tuser where name = '张一'
  select * from tuser where age = 10
  ```
  
  我们可以用explain语法分析其执行计划，发现第一条sql命中了索引，而第二条sql采用的是全表扫描，这种现象称之为：**最左匹配**

>**最左匹配**：指在联合索引中，如果你的 sql 语句中用到了联合索引中的最左边的索引，那么这条 sql 语句就可以利用这个联合索引去进行匹配；

+ 场景四

  ```sql
  select id,name,age from tuser where name like '张%' and age < 10
  ```
  
  该SQL是查询出”张“姓并且年龄小于10的user；表中存在4个user姓”张“，在Mysql5.6版本以前，非主键索引会返回4个ID，回表后判定出2条数据符合；而Mysql5.6版本及以后非主键索引只会返回2条满足条件的sql，原因是该版本引入了**索引下推**功能

>**索引下推**：Mysql5.6版本引入的功能，其旨在减少回表次数；

## 3 explain语法详解

[Mysql官网](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

在Sql调优过程中explain语法是常用的基本操作。
执行explain语法后会看到表格：

> |id|select_type|table|partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra|
> |--|--|--|--|--|--|--|--|--|--|--|--|
> |1|SIMPLE|t_indexSelect|NULL|range|a,b|a|5|NULL|1000|50.00|Using index condition; Using where|

+ id
  SELECT识别符。这是SELECT的查询序列号
  1. SQL从大到小的执行
  2. id相同时，执行顺序由上至下
  3. id为NULL最后执行

+ select_type
  + SIMPLE, 表示此查询不包含 UNION 查询或子查询
  + PRIMARY, 表示此查询是最外层的查询
  + UNION, 表示此查询是 UNION 的第二或随后的查询
  + DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询
  + UNION RESULT, UNION 的结果
  + SUBQUERY, 子查询中的第一个 SELECT
  + DEPENDENT SUBQUERY: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.
  
+ table
    表示查询涉及的表或衍生表
+ partitions
  匹配的分区
+ type
  + 表示关联类型或访问类型，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。依次从最优到最差分别为：**system > const > eq_ref > ref > range > index > ALL**一般来说，得保证查询达到range级别，最好达到ref；  
  + system: 表中只有一条数据. 这个类型是特殊的 const 类型
  + const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.
  + eq_ref: 此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 =, 查询效率较高.
  + ref: 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 最左前缀 规则索引的查询.
  + range: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.
  + index: 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.
  + ALL: 表示全表扫描, 这个类型的查询是性能最差的查询之一.
+ possible_keys
  MySQL 在查询时, 能够使用到的索引. 注意, 即使有些索引在 possible_keys 中出现, 但是并不表示此索引会真正地被 MySQL 使用到. MySQL 在查询时具体使用了哪些索引, 由 key 字段决定.
+ key
  MySQL 在当前查询时所真正使用到的索引.
+ key_len
  查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.
+ ref
  引用到的上一个表的列
+ rows
  MySQL 查询优化器根据统计信息, 估算 SQL 要查找到结果集需要扫描读取的数据行数.**该值越小越好**
+ filtered
  按表条件过滤行的百分比
+ Extra
  + Using filesort： 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果. 一般有 Using filesort, 都建议优化去掉, 因为这样的查询 CPU 资源消耗大.
  + Using index："覆盖索引扫描", 表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, 往往说明性能不错
  + Using temporary:查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化.
