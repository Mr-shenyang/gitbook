
# Condition是什么

我们可以通过两种方式实现线程的阻塞和唤醒操作：

1. 通过java内置的Object.wait(),Object.notify(),Object.notifyAll()目的是实现线程的阻塞和唤醒
2. 通过LockSupport提供的park和unpark实现线程的阻塞和唤醒

而Condition是第三种实现线程阻塞和唤醒的方式，其业务实现更加灵活；

||Object|Condition|
|:-----:|:-----:|:-:|
挂起|wait()|await()
唤醒|notify()|signal()
唤醒全部|notifyAll()|signalAll()

话不多说，上代码，上类图。

```java
public interface Condition {

    void await() throws InterruptedException;
    /** 部分方法省略*/

    void signal();

    void signalAll();
}
```

![AbstractOwnableSynchronizer类图](/images/juc/锁/JUC_Condition_类图.png)

## AbstractQueuedSynchronizer#ConditionObject实现

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
