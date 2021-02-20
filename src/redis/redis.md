# redis

## redis简介

Remote Dictionary Server(Redis) 是一个基于内存的key-value存储系统,其中value类型支持String,List,Set,Hash,Zset五种类型。

## redis支持的数据类型

**基础数据类型：** 字符串String、列表List、哈希Hash、集合Set、有序集合Zset

**扩展数据类型：** Bitmaps、HyperLogLogs、GEO

**redis底层数据结构：** 简单动态数组SDS、链表、字典、跳跃链表、整数集合、压缩列表、对象

Redis为了平衡空间和时间效率，针对value的具体类型在底层会采用不同的数据结构来实现。

![data_structure](/images/redis/data_structure.png)

# QA

**1.redis实现分布式锁的?**  
答：
（1）自2.8版本起，setnx命令是原子操作。这保证单节点的Redis服务是支持锁的特性的。
（2）引入了RedLock算法，实现在多节点场景下对分布式锁的支持；

>
>1.[RedLock](https://redis.io/topics/distlock)是什么？
> RedLock是分布式算法算法。
>（1）获取当前Unix时间，以毫秒为单位；
>
>（2）依次尝试从所有实例中，使用相同的key和具有唯一性的value获取锁；
>（当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间，这样可以避免客户端死等）
>
>（3）当且仅当从半数以上的Redis节点取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
>
>（4）如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间。
>
>（5）如果因为某些原因，获取锁失败（没有在半数以上实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁，无论Redis实例是否加锁成功。
>（因为可能服务端响应消息丢失了但是实际成功了，毕竟多释放一次也不会有问题）
>
>2.RedLock有什么缺陷吗？
>（1）依赖客户端时钟，如果发生时钟跳跃会有问题；
>（2）长时间的GC pause，会造成线程A先获取锁后刚好Gc停顿，而线程B在锁过期后又获取了锁并执行；
>（3）长时间的网络延迟；
>
>参考：https://mp.weixin.qq.com/s/Tvdu1-1nRnCyFvOXGMmz_A 

