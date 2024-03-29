# 线程池是什么？

线程池（Thread Pool）是一种基于池化思想管理线程的工具。
我们知道*创建线程*、*销毁线程*、*调度线程*都是比较大的消耗。
线程池中维护多个线程，等待监督管理者分配可并发执行的任务。这样，一方面避免了处理任务时创建销毁线程的开销，另一方面避免了线程数量膨胀导致的过分调度问题，保证了对内核的充分利用。

# 线程池核心

线程池在内部实际是一个生产者消费者模型，将**任务**和**线程**两者解耦，从而很好的缓冲任务，复用线程。线程池的运行主要分成两部分：任务管理、线程管理。
任务管理部分充当“生产者的角色”，当任务提交后，线程池会判断该任务后续的流转：
（1）直接申请线程执行该任务；
（2）缓冲到队列中等待线程执行；
（3）拒绝任务。

线程管理部分是消费者，它们被统一维护在线程池内，根据任务请求进行线程的分配，当线程执行完任务后则会继续获取新的任务去执行，最终当线程获取不到任务的时候，线程就会被回收。

![framework](/images/juc/线程池/framework.png)

