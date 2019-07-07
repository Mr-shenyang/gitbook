# 1 GC简述

## 1.1 GC是什么

GC是Garbage Collection的简称，中文称为“垃圾回收”

## 1.2 GC解决什么问题

* 哪些内存需要回收

* 什么时候回收？

* 如何回收？

# 2 GC算法介绍

虽然GC概念

# 2.1 

垃圾回收

# 3 HotSpot算法介绍

jvm规范对垃圾收集器的实现，没有做规定因此不同的厂商、不同版本都会有差异。下图展示了7种作用于不同分代的收集器。

![image](/images/gc_overview.png)

特别提醒**两个收集器之间存在连线，则它们可以搭配使用**

## 3.1 Serial

![Serial运行示意图](/images/gc_serial.png)

## 3.2 Serial Old

![Serial Old运行示意图](/images/gc_serialOld.png)

## 3.3 ParNew

![ParNew运行示意图](/images/gc_parNew.png)

## 3.4 Parallel Scavenge

![Parallel Scavenge运行示意图](/images/gc_parNew.png)

## 3.5 Parallel Old

![Parallel Old](/images/gc_parallelOld.png)

## 3.6 CMS

![CMS](/images/gc_cms.png)

## 3.7 G1

![G1](/images/gc_g1.png)

