# 简介

锁是并发编程中绕不开话题，大神Doug Lea在包java.util.concurrent.locks中给我们提供了并发编程中常用的锁。
下图是concurrent.locks提供的锁；
![类图](/images/sourcecode/concurrent/lock/LockCD.png)

类图并不复杂，包括同步器（AbstractOwnableSynchronizer,AbstractQueuedSynchronizer,AbstractQueuedLongSynchronizer）、锁（Lock,ReentrantLock）、读写锁（ReadWriteLock,ReentrantReadWriteLock）、工具类（LockSupport）、Condition以及jdk 1.8 提供的新的读写锁（StampedLock）。

本章将逐步对其源码解析剖析解读。
