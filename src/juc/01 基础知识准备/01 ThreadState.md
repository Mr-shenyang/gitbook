# 1 状态介绍

今天聊聊线程状态。在Thread类中提供了Thread.State的内部枚举，这个枚举中定义了流程状态包括
|State|状态|说明
|:-:|:-:|:-|
|**NEW**|新建态|新建未开始的线程状态|
|**RUNNABLE**|运行态|在jvm处于执行状态，但是其可能等待其他操作系统资源，比如CPU,IO|
|**BLOCKED**|阻塞态|
|**WAITING**|等待状态|
|**TIMED_WAITING**|定时等待状态|
|**TERMINATED**|终态|

然后从哪里开始聊呢，先上一个状态轮转图吧

![thread状态图](/images/juc/01基础/01_ThreadStatus.png)

# 2 主要操作

* start()
  一个线程只能start一次。主要是通过调用native start0()来实现
* sleep()
  sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务**sleep方法不会释放锁**,底层通过调用native sleep实现
* join()
  实际是利用了wait()，只不过它不用等待notify()/notifyAll()，且不受其影响。它结束的条件是：1）等待时间到；2）目标线程已经run完（通过isAlive()来判断）
* interrupt()
  该方法仅仅设置中断标志位，底层通过调用native interrupt0实现，也就是说**线程何时中断，由线程自己决定**

# 3 QA

**Q:阻塞与等待的区别：**
阻塞：当一个线程试图获取对象锁（synchronized），而该锁被其他线程持有，则该线程进入阻塞状态。它的特点是*使用简单，由JVM调度器来决定唤醒自己，而不需要由另一个线程来显式唤醒自己，不响应中断。*
等待：当一个线程等待另一个线程通知调度器一个条件时，该线程进入等待状态。它的特点是*需要等待另一个线程显式地唤醒自己，实现灵活，语义更丰富，可响应中断*。例如调用：Object.wait()、Thread.join()以及等待Lock或Condition。
需要强调的是虽然synchronized和JUC里的Lock都实现锁的功能，但线程进入的状态是不一样的。**synchronized会让线程进入阻塞态，而JUC里的Lock是用LockSupport.park()/unpark()来实现阻塞/唤醒的，会让线程进入等待态**。但话又说回来，虽然等锁时进入的状态不一样，但被唤醒后又都进入runnable态，从行为效果来看又是一样的。

**Q:阻塞线程有几种方式？他们有什么区别？**
block线程大致有一下四种方式Thread.sleep、Object.wait、LockSupport.park和Condition.await四种方式；
下面就比较下这四种方式的异同
![thread状态图](/images/juc/01基础/01_block线程.png)