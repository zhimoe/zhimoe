+++
title = "Java 并发 2-同步与锁"
date = 2016-01-01
categories = [ "编程",]
tags = [ "java", "并发",]
toc = "true"
+++



### Object.wait/notify/notifyAll

这三个方法是在 class Object 上面的，也就是所有对象都有这个方法。这里对象就是上一篇中类比的资源，可以当成一个信号量。
`Object.wait()` to suspend a thread（等价于`sem.wait()`）。将当前线程暂停并释放当前对象锁，直到其他线程调用了当前对象的 notify/notifyAll 方法。
`Object.notify()` to wake a thread up（等价`sem.signal()`）。唤醒一个在等待当前对象锁的线程。
以前用 wait/notify 的地方，现在可以用 CyclicBarrier 和 CountDownLatch 同步工具代替

wait 方法使用模板：
```java

synchronized (obj) {
    while (condition does not hold){ // 这里即使线程被虚假唤醒，条件还是不满足，则继续 wait
        obj.wait();
    }
    //... Perform action appropriate to condition
}
```

为什么 wait 方法必须在 synchronized 保护的同步代码中使用？因为需要保证 while 判断和 wait 两个操作是一个原子操作。

两个线程交替打印 A/B
```java
    public static void main(String[] args) {
            PrintTask pt = new PrintTask("A");
            new Thread(pt::print).start();
            new Thread(pt::print).start();
    }


    static class PrintTask {
        private String currentChar;

        PrintTask(String initialChar) {
            this.currentChar = initialChar;
        }

        public void print() {
            synchronized (this) {
                while (true) {
                    System.out.println(Thread.currentThread().getId() +" "+currentChar);
                    if ("A".equals(currentChar)) {
                        currentChar = "B";
                    } else {
                        currentChar = "A";
                    }
                    try {
                        this.notifyAll();
                        this.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }

                }
            }
        }
    }
```


### synchronized 和 volatile

volatile 比较简单，可以理解为将一个变量强制同步给所有线程可见，但是不能解决并发写的问题。

java 的每个对象都有内在锁（或者叫监视器锁，intrinsic lock or monitor lock），涉及两个 JVM 指令：monitorenter、monitorexit。

synchronized method 和 synchronized block 的区别：
如果是 synchronized(this),那么和 synchronized 方法没有任何区别，锁定对象都是方法所在的对象。
但是 synchronized block 可以锁定其他对象，而且 synchronized block 的范围是可以控制更灵活，synchronized 方法的边界只能是整个方法


使用 ReentrantLock 场景：
需要以下高级特性时：可定时的，可轮询的，可中断的锁，公平队列，非块结构。
[Synchronization vs Lock-stackoverflow](https://stackoverflow.com/questions/4201713/synchronization-vs-lock)

### Semaphore
可以理解为资源的许可证数量。
sem.acquire(2): 获取两个许可证。
sem.release() 释放资源的一个许可证。
信号量非常有用，也是基础，基于信号量，可以构建出很多其他的同步工具：turnstile 和 rendezvous，barrier，mutex，Multiplex

### CountDownLatch
CountDownLatch 是管理一组线程和一个主线程的先后。主线程 wait 后就阻塞，直到所有的 CountDownLatch 调用 countDown 后主线程接着开始。

```java
package zhimoe.intrview.concurrent;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchTest {
	// 这个方法将启动多个任务，并让它们同时执行，计算完成的时间
	public long timer(int taskNums) throws InterruptedException {
		CountDownLatch startLatch = new CountDownLatch(1);
		CountDownLatch finishLatch = new CountDownLatch(taskNums);
		for (int i = 0; i < taskNums; i++) {
			Task task = new Task(startLatch, finishLatch, i);
			new Thread(task).start();
		}

		long start = System.nanoTime();
		startLatch.countDown();// 准备好线程后开始同时启动所有任务
		finishLatch.await();// 等待任务完成
		long end = System.nanoTime();
		return end - start;

	}

	public static void main(String[] args) throws InterruptedException {
		CountDownLatchTest ct = new CountDownLatchTest();
		long time = ct.timer(100);
		System.out.println(
			TimeUnit.NANOSECONDS.toSeconds(time) + " SENCODS");
	}

}

class Task implements Runnable {
	CountDownLatch startLatch;
	CountDownLatch finishLatch;
	int time;

	Task(CountDownLatch startLatch, CountDownLatch finishLatch, int time) {
		this.startLatch = startLatch;
		this.finishLatch = finishLatch;
		this.time = time;
	}

	@Override
	public void run() {
		try {
			startLatch.await();// 等待主线程通知任务开始
			System.out.println("doing the task!");
			Thread.sleep(time * 100); // 模拟任务过程
		} catch (InterruptedException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		} finally {
			System.out.println("task done");
			finishLatch.countDown();// 告诉主线程任务完成
		}

	}

}
```
### CyclicBarrier
### Exchanger
### Lock






