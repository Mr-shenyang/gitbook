# 01 事务

事务：是一系列操作组成的最小执行单元，具有A（Atomicity 原子性）C（Consistency 一致性）I（Isolation 隔离性）D（Durability 持久性）四大特性；

* 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
* 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
* 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
* 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中

# 02 spring事务简介

>特别说明：spring并不直接管理事务，而是提供了多种事务管理器。这些事务管理器代理了Jta等持久化机制所提供的事务。

spring事务核心类图如下：
![spring事务类图](/images/spring/springTransaction.png)
spring在PlatformTransactionManager对事务管理进行了统一的抽象，然后面对不同的平台有具体实现。

# 03 spring事务使用

## 3.1 注解使用模式

### 3.1.1 demo

### 3.1.2 解析

## 3.2 注解使用模式

### 3.2.2 demo

### 3.2.2 解析

# 04 DataSourceTransactionManager介绍

# 05 Q/A
* Q:Spring事务如何传递的？
  
* Q:Spring事务支撑timeOut吗？如何实现的
  A:
* d