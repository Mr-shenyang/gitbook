# redis简介

Remote Dictionary Server(Redis) 是一个基于内存的key-value存储系统,其中value类型支持String,List,Set,Hash,Zset五种类型。

# redis支持的数据类型

**基础数据类型：** 字符串String、列表List、哈希Hash、集合Set、有序集合Zset

**扩展数据类型：** Bitmaps、HyperLogLogs、GEO

**redis底层数据结构：** 简单动态数组SDS、链表、字典、跳跃链表、整数集合、压缩列表、对象

Redis为了平衡空间和时间效率，针对value的具体类型在底层会采用不同的数据结构来实现。

![data_structure](/images/redis/data_structure.png)
