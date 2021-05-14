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

![thread状态图](/images/juc/ThreadStatus.png)

# 2 演示说明

通过图我们可以清晰看到，流程的状态流转。下面通过具体的代码进行演示，

## 1 场景1 NEW-->RUNNABLE-->TERMINATED

这个是异常线程正常执行的场景，线程创建后状态变成NEW,执行start方法后线程变成RUNNABLE,最后当线程完成执行后状态变成TERMINATED。下面的代码也验证了这一状态流转场景。

``` java
public class ThreadStateDemo {
	private static final Logger log = LoggerFactory.getLogger(ThreadStateDemo.class);
 
	public static void main(String[] args) {
		ThreadStateDemo threadStateDemo = new ThreadStateDemo();
		threadStateDemo.sceneOne();
	}
 
	private void sceneOne(){
    	System.out.println("场景1：");
		try {
			SceneOneThread sceneOneThread = new SceneOneThread();
			sceneOneThread.start();
			/**
			 * 主线程sleep10毫秒，保证sceneOne线程执行完成
			 */
			Thread.sleep(10);
			
			/**
			 * 输出执行完成后状态
			 */
			System.out.print(sceneOneThread.getState());
			
		} catch (Exception e){
			log.error("error:",e);
		}
	}
	
	class SceneOneThread extends Thread{
		
  		public SceneOneThread() {
			System.out.print(getState());
		}
		
		@Override
		public synchronized void start() {
			System.out.print("--start()-->");
			super.start();
		}
		
		@Override
		public void run() {
			System.out.print(getState());
			System.out.print("--completed-->");
		}
	}
}
```

输出结果是：
> 场景1：
> NEW--start()-->RUNNABLE--completed-->TERMINATED

## 2 NEW-RUNNABLE-BLOCKED-RUNNABLE-TERMINATED

## 3 NEW-RUNNABLE-WAITING-RUNNABLE-TERMINATED

## 4 NEW-RUNNABLE-TIMED_WAITING-TERMINATED
