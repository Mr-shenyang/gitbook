redis自诞生以来先后经历了三种集群模式：

## 主从模式

为了支持Redis宕机后能快速恢复，Redis提供了主从库模式来实现这一功能；

![主从集群](/images/redis/主从集群.png)

主从集群下添加从节点分成三阶段：
![主从添加从节点](/images/redis/主从_newNode.png)

## 哨兵（Sentinel）模式

主库模式解决了Redis宕机后能快速恢复能力。但是，如果Redis主库宕机了，还是存在一段时间的不可用；于是就有了哨兵模式；

![哨兵集群](/images/redis/哨兵集群.png)

哨兵模式即是在主从模式的基础上引入哨兵，它其实就是一个运行在特殊模式下的Redis进程，主从库实例运行的同时，它也在运行。哨兵主要负责的就是三个任务：**监控**、**选主（选择主库）**和**通知**。

* 监控：
  哨兵进程在运行时，周期性地给所有的主从库发送PING命令，检测它们是否仍然在线运行。如果从库没有在规定时间内响应哨兵的PING命令，哨兵就会把它标记为“下线状态”；同样，如果主库也没有在规定时间内响应哨兵的PING命令，哨兵就会判定"主观下线"，当多余一半的监控任务主库处于"主观下线"时，主库就会被判定为“客观下线”，此时哨兵集群将形式第二个使命“选主”；

* 选主
  首先，哨兵会按照在线状态、网络状态，筛选过滤掉一部分不符合要求的从库，然后，依次按照优先级、复制进度、ID号大小再对剩余的从库进行打分，只要有得分最高的从库出现，就把它选为新主库

* 通知
  哨兵会把新主库的连接信息发给其他从库，让它们执行replicaof命令，和新主库建立连接，并进行数据复制。同时，哨兵会把新主库的连接信息通知给客户端，让它们把请求操作发到新主库上

## 切片模式

有了哨兵模式，稳定的Redis集群已经初具有雏形了，可是问题又来了，无论是主从还是哨兵模式都只能有一个主库；当主库容量不足以支持业务需求时，越是就有了切片集群；

![切片集群](/images/redis/切片集群.png)

切片集群是一种服务器Sharding技术，3.0版本开始正式提供。

1. 采用slot(槽)的概念，每个Redis实例映射多个槽；
   >一共分成16384个槽，原理是将key进行crc16算法获取16bit长的值，然后对16384取余计算出key对应的槽；

2. Redis实例与实例直接通过Ping/Pong通信，使得任何一个实例都维护着一份槽与实例的映射；

3. 客户端操作Redis实例时，如果改key属于本实例所对应的槽，则直接操作，否则返回Move命令给客户端，告知客户端该key所在的Redis实例，由客户端重新发起调用；
