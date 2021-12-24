# 1 是什么？

AbstractQueuedSynchronizer[AQS]是抽象的队列式的同步器，它是除java自带的synchronized关键字之外的锁机制的基石；

# 2 核心原理

AQS中维护了一个state（资源）和一个FIFO的列表：

![AQS原理图](/images/juc/01基础/AQS_原理.png)

**state:** AQS定义了一个volatile的state用于表示共享资源，同时配套了*getState()*、*setState()*、*compareAndSetState()*；

**head:** 指向CLH队列头部，该队列用于维护了一个自旋的锁队列；

**tail:** 指向CLH队列尾部；

> AQS是自旋锁：在等待唤醒的时候，经常会使用自旋（while(!cas())）的方式，不停地尝试获取锁，直到被其他线程获取成功

AQS中定义了两种资源共享方式,并配套了对应的资源获取、释放方式

* Exclusive：独占，只有一个线程能执行，如ReentrantLock；
   acquire(int)：独占方式尝试获取资源，成功则返回true，失败则返回false。
   release(int)：独占方式尝试释放资源，成功则返回true，失败则返回false。

* Share：共享，多个线程可以同时执行，如Semaphore、CountDownLatch、ReadWriteLock，CyclicBarrier；
   acquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
   releaseShared(int)：共享方式。释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

# 3 Node

在开始进行AQS学习之前我们需要先聊聊其内部类Node，

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
    /***
     * 记录了下一个等待Condition的节点，或者共享模式下的特殊值
     */
    Node nextWaiter;
}
```

这是一个很清爽的链表数据结构，包括了线程thread、节点状态waitStatus以及prev、next和nextWaiter；

# 4 独占模式

## 4.1 获取资源[acquire]

此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，**且整个过程忽略中断的影响**。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

代码很短，但含义很丰富，函数流程如下：

1. tryAcquire()尝试直接去获取资源，如果成功则直接返回；
2. addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
3. acquireQueued()使线程阻塞在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上

### 4.1.1 tryAcquire

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

AQS只是一个同步器框架，具体的实现需要子类按需实现，这里包括：是否公平获取资源，能不能重入；

### 4.1.2 addWaiter

此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点

```java
private Node addWaiter(Node mode) {
    //基于当前Thread创建指定模式的Node；
    Node node = new Node(Thread.currentThread(), mode);
    // 设置队尾的节点为pred；
    Node pred = tail;
    // pred不为空，即队尾设置
    if (pred != null) {
        //新创建的Node添加到队尾；
        node.prev = pred;
        //采用CAS机制设置新创建节点为新的队尾
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //只有当队尾不存在，或者CAS设置队尾失败时才会走到enq方法
    //enq方法很简单其核心就是将节点插入到队列，如果有必要也会做队列的初始化
    enq(node);
    return node;
}

private Node enq(final Node node) {
    //CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
        if (t == null) { 
            if (compareAndSetHead(new Node()))
                tail = head;
        } 
        //正常流程，放入队尾
        else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

### 4.1.3 acquireQueued

通过第一步我们获取资源失败，通过第二步我们创建了新的节点并放入队尾，第三步我们就要进入等待状态休息，直到其他线程彻底释放资源后唤醒自己；

```java
final boolean acquireQueued(final Node node, int arg) {
    //标记是否成功拿到资源
    boolean failed = true;
    try {
        //标记等待过程中是否被中断过
        boolean interrupted = false;
        //自旋
        for (;;) {
            //拿到前驱节点
            final Node p = node.predecessor();
            //如果前驱节点就是head节点，可以尝试获取下资源
            if (p == head && tryAcquire(arg)) {
                //拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null
                setHead(node);
                p.next = null; // help GC
                failed = false; //表明成功获取资源
                return interrupted; //返回等待过程中是否被中断过
            }
            //如果可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;//interrupted标记为true
        }
    } finally {
        // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
        if (failed)
            cancelAcquire(node);
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //拿到前驱的状态
    int ws = pred.waitStatus;
    //如果前驱状态是SIGNAL，表明其拿到资源后会通知当前节点，返回true，当前节点可以休息了
    if (ws == Node.SIGNAL)
        return true;
    //如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        //如果前驱正常，那就把前驱的状态设置成SIGNAL，表明前驱获取资源后会通知后续节点
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

//shouldParkAfterFailedAcquire只是为了设置前驱节点状态，parkAndCheckInterrupt才是真正的将当前流程阻塞住；
private final boolean parkAndCheckInterrupt() {
    //调用park()使线程进入waiting状态
    LockSupport.park(this);

    return Thread.interrupted();
}
```

至此整个acquire就结束了，最后通过一个流程图来整体回顾下，比较这是重点中的重点

### 4.1.4 总结

![AQS原理图](/images/juc/01基础/AQS_acquire.png)

## 4.2 释放资源【release】

此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒等待队列里的下一个线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

## 4.2.1 tryRelease

```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

跟tryAcquire()一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。

## 4.2.2 unparkSuccessor

```java
private void unparkSuccessor(Node node) {
    
    int ws = node.waitStatus;
    if (ws < 0)
        //置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next;
    //如果为空或已取消
    if (s == null || s.waitStatus > 0) {
        s = null;

        // 从后向前找<=0的结点（有效的结点）
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

这个函数并不复杂。一句话概括：用unpark()唤醒等待队列中最前边的那个未放弃线程；这里我们也用s来表示吧。此时，再和acquireQueued()联系起来，s被唤醒后，进入if (p == head && tryAcquire(arg))的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire()寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire()的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire()也返回了！！And then, DO what you WANT!

# 5 共享模式

## 5.1 获取资源[acquireShared]

此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，**整个过程忽略中断**

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

### 5.1.1 tryAcquireShared

```java
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
```

这里tryAcquireShared()依然需要自定义同步器去实现

### 5.1.2 doAcquireShared

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);//加入队列尾部
    boolean failed = true;//是否成功标志
    try {
        boolean interrupted = false;//等待过程中是否被中断过的标志
        //有见自旋
        for (;;) {
            final Node p = node.predecessor();//前驱
            //如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
            if (p == head) {
                //尝试获取资源
                int r = tryAcquireShared(arg);
                //成功
                if (r >= 0) {
                    //将head指向自己，还有剩余资源可以再唤醒之后的线程
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    //如果等待过程中被打断过，此时将中断补上。
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);//head指向自己
    //如果还有剩余量，继续唤醒下一个邻居线程
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

## 5.2 释放资源[releaseShared]

此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean releaseShared(int arg) {
    //尝试释放资源
    if (tryReleaseShared(arg)) {
        //唤醒后继结点
        doReleaseShared();
        return true;
    }
    return false;
}
```

### 5.2.1 tryReleaseShared

```java
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```

这里tryReleaseShared()依然需要自定义同步器去实现;

### 5.2.2 doReleaseShared

```java
private void doReleaseShared() {
    //自旋
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        //head没有变化时就会退出循环
        if (h == head)                   // loop if head changed
            break;
    }
}
```
