## Condition是什么？

你以为AQS就是前一章那么点内容吗？Too young too simple；
AQS中还有一个内部类ConditionObject，它继承自Condition接口。

我们知道我们可以通过两种方式实现线程的阻塞和唤醒操作：

1. 通过java内置的Object.wait(),Object.notify(),Object.notifyAll()目的是实现线程的阻塞和唤醒
2. 通过LockSupport提供的park和unpark实现线程的阻塞和唤醒

而Condition是第三种实现线程阻塞和唤醒的方式，其业务实现更加灵活；

||Object|Condition|
|:-----:|:-----:|:-:|
挂起|wait()|await()
唤醒|notify()|signal()
唤醒全部|notifyAll()|signalAll()

```java
public interface Condition {
    //挂起
    void await() throws InterruptedException;
    //唤醒一个等待线程
    void signal();
    //唤醒所有等待线程
    void signalAll();
    /** 部分方法省略*/
}
```

## ConditionObject是什么？

Condition中对线程阻塞和唤醒只是给出了定义，具体实现还得依赖ConditionObject；其模型结构如下：

```java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
}
```
