# ExecutorService相关

**Q1.** ExecutorService中shutdown()、shutdownNow()、awaitTermination()都是什么含义，它们有什么区别？  
**A：**  
含义：  
shutdown()：停止接收新任务，原来的任务继续执行；  
shutdownNow()：停止接收新任务，原来的任务停止执行并返回未执行的任务列表；  
awaitTermination()：阻塞当前线程，直到超时、所有线程执行完毕或者当前线程被中断
区别：  
shutdown()：只是关闭了提交通道；  
shutdownNow()：关闭提交后会将未执行的线程关闭；  
awaitTermination（）：只是阻塞线程，并没有关闭提交通道；  

**Q2.** ExecutorService中isShutdown()、isTerminated()有什么区别？  
**A：** isShutdown()指定是线程池已经关闭提交通道；而isTerminated()指的是线程池关闭后所有的线程都运行完毕，isTerminated()必须在showdown()之后才会返回true；  
