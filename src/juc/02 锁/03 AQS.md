# Synchronizer是什么

在正式开始学习锁之前，先让我们了解下同步器，因为Doug Lea大神设计锁的核心都是基于同步器实现的。

# AbstractOwnableSynchronizer

三个同步器的中AbstractOwnableSynchronizer相对比较单薄，其类图如下

![AbstractOwnableSynchronizer类图](/images/sourcecode/concurrent/lock/AbstractOwnableSynchronizer.png)

可以看到其声明了一个独占线程exclusiveOwnerThread，并提提了get、set方法；

# AbstractQueuedSynchronizer

这个类也就是程序员常说的AQS。其本质是一个FIFO的等待队列，其类图如下：

![AbstractQueuedSynchronizer类图](/images/sourcecode/concurrent/lock/AbstractQueuedSynchronizer.png)

可以看到AQS本身继承自AbstractOwnableSynchronizer，它内部包含了Node类和继承自接口Condition的ConditionObject类。

