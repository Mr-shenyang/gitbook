# CountDownLatch

>CountDownLatch是一种一次性同步辅助工具，它实现了一个或一组线程等待被其他线程唤醒的机制；

## 核心方法

>await: 阻塞线程，直到CountDownLatch中计数器到0；才会被唤醒
>countDown: 递减CountDownLatch中的计数器

## 样例

赛车比赛中：发令人需要等所有赛车进场后才能开始比赛，计时器需要等待所有赛车进场后才能开始计时；

```java
public class CountDownLatchDemo {
  	public static void main(String[] args) {
  		int n =10;
		CountDownLatch countDownLatch = new CountDownLatch(n);

		Thread thread_1 = new Thread(() -> {
			log.info("发令人就位：");
			try {
				countDownLatch.await();
			} catch (InterruptedException e) {
				log.error("error", e);
			}
			log.info("发令人发令");
		});

		Thread thread_2 = new Thread(() -> {
			log.info("计时人员就位：");
			try {
                // await 会直接阻塞线程
				countDownLatch.await();
			} catch (InterruptedException e) {
				log.error("error", e);
			}
			log.info("计时人员计时");
		});

		thread_1.start();
		thread_2.start();
		for (int i = 1; i<= n; i++) {
			final  String carName = "No " + i;
			new Thread(()->{
				try {
					TimeUnit.SECONDS.sleep(new Random().nextInt(10));
				} catch (InterruptedException e) {
					log.error("sleep error",e);
				}
				log.info(carName + "进场");
                // 触发countDownLatch逐步递减计时器
				countDownLatch.countDown();
			}).start();
		}
  	}
}
```

## 原理解析

CountDownLatch类存在一个内部类Sync，其继承自AbstractQueuedSynchronizer（AQS），通过AQS来实现CountDownLatch的两个核心功能；

**New**
每一个CountDownLatch实例都对于一个Sync实例；

```java 源码
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    //实例化一个指定count的Sync
    this.sync = new Sync(count);
}
```

**await**
![时序图](/images/juc/tools/CountDownLatch_await.png)

```java 源码
/** 
* 1：CountDownLatch.await
*/
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
/** 
* 2：AQS.acquireSharedInterruptibly
*/
public final void acquireSharedInterruptibly(int arg)throws InterruptedException {
    //如果线程中断则直接抛异常
    if (Thread.interrupted())
        throw new InterruptedException();
    //尝试获取资源，获取成功直接返回，获取失败则block线程
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

/** 
* 3：CountDownLatch#Sync.tryAcquireShared
*/
protected int tryAcquireShared(int acquires) {
    //实例化后的Sync的状态 == 0 则返回表示成功的1否则返回表示失败的-1
    return (getState() == 0) ? 1 : -1;
}

/** 
* 4：AQS.doAcquireSharedInterruptibly【核心】
*/
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    //创建节点并添加至等待队列
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        //无限循环
        for (;;) {
            //获取节点的前驱节点
            final Node p = node.predecessor();
            //前驱节点为头结点
            if (p == head) {
                //试图在共享模式下获取对象状态
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //获取成功则：设置头结点并进行繁殖
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            //在获取失败后是否需要禁止线程并且进行中断检查
            if (shouldParkAfterFailedAcquire(p, node) &&parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
/** 
* 4.1：AQS.addWaiter
*/
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
    //只有当队尾不存在，或者CAS设置队尾失败是才会走到enq方法
    //enq方法很简单其核心就是将节点插入到队列，如果有必要也会做队列的初始化
    enq(node);
    return node;
}

/** 
* 4.2：AQS.setHeadAndPropagate
* 这段代码比较烧脑，简单概括就是：在节点获取到对象状态时，需要发信号给队列中的下一个节点
*/
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
    /*
     * Try to signal next queued node if:
     *   Propagation was indicated by caller,
     *     or was recorded (as h.waitStatus either before
     *     or after setHead) by a previous operation
     *     (note: this uses sign-check of waitStatus because
     *      PROPAGATE status may transition to SIGNAL.)
     * and
     *   The next node is waiting in shared mode,
     *     or we don't know, because it appears null
     *
     * The conservatism in both of these checks may cause
     * unnecessary wake-ups, but only when there are multiple
     * racing acquires/releases, so most need signals now or soon
     * anyway.
     */
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            //这里暂时不聊这个函数，待说到countDown是会有介绍
            //主要功能唤醒后续结点
            doReleaseShared();
    }
}
```

**countDown**
![时序图](/images/juc/tools/CountDownLatch_countDown.png)

```java 源码
/** 
* 1：CountDownLatch.countDown
*/
public void countDown() {
    sync.releaseShared(1);
}

/** 
* 2：AQS.releaseShared
*/
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

/** 
* 3：CountDownLatch#Sync.releaseShared
*/
protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        // 当前AQS的状态已经为0时表明释放失败
        if (c == 0)
            return false;
        int nextc = c-1;
        //CAS机制将AQS的state递减
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
/** 
* 4：AQS.doReleaseShared
*/
/**
 * Release action for shared mode -- signals successor and ensures
 * propagation. (Note: For exclusive mode, release just amounts
 * to calling unparkSuccessor of head if it needs signal.)
 */
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
     */
    //自旋
    for (;;) {
        Node h = head;
        //头结点不为空且不为尾结点(阻塞队列里面只有一个结点)
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                //唤醒后继节点
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        //如果头结点变化了则跳出循环
        if (h == head)                   // loop if head changed
            break;
    }
}

```

## QA

**Q** CountDownLatch的底层实现原理是什么?
**A** CountDownLatch是基于AQS机制实现一种同步辅助工具；其实现了一个或一组线程等待被其他线程唤醒的机制；

**Q** CountDownLatch一次可以唤醒几个任务?
**A** 多个
