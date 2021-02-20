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

## Node

```java
static final class Node {
    /**
    * SIGNAL -1 表示当前节点的后继节点包含的线程需要运行
    * CANCELLED 1 表示当前线程被取消
    * CONDITION -2 表示当前节点在等待condition，也就是在condition队列中；
    * PROPAGATE -3 表示当前场景下后续的acquireShared能够得以执行
    * 0 表示当前节点在sync队列中，等待着获取锁
    */
    volatile int waitStatus;

    volatile Node prev;

    volatile Node next;

    volatile Thread thread;

    Node nextWaiter;
}
```

这是一个很清爽的链表数据结构，包括了线程thread、节点状态waitStatus以及prev、next和nextWaiter；

## ConditionObject

AQS中还有个比较重要的内部类ConditionObject，在介绍ConditionObject之前，先看看其父接口Condition。

### Condition是什么

```java
public interface Condition {

    void await() throws InterruptedException;
    /** 部分方法省略*/

    void signal();

    void signalAll();
}
```

我们知道java内置的Object.wait(),Object.notify(),Object.notifyAll()目的是实现线程的阻塞和唤醒，Condition对标了这些方法，但是内置方法不同的是Condition可以创建多个，业务实现更加灵活；
||Object|Condition|
|:-----:|:-----:|:-:|
挂起|wait()|await()
唤醒|notify()|signal()
唤醒全部|notifyAll()|signalAll()

### ConditionObject实现

```java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
    //method 略
}
```

**await**
阻塞线程

```java
public final void await() throws InterruptedException {
    //线程中断，抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    //将当前线程添加到队列中
    Node node = addConditionWaiter();

    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // 利用UNSAFE实现线程阻塞
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```
