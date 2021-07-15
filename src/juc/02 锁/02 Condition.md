
# Condition是什么



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
