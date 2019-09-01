---
title: 'Java concurrency 5 Synchronizer and AQS'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

#### 好难，看不懂呀！

#### 先自己写一个CountDownLatch的示例：
CountDownLatch是管理一组线程和一个主线程的先后。主线程wait后就阻塞，直到所有的CountDownLatch调用countDown后主线程接着开始。

```java
package angus.intrview.concurrent;

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
		System.out.println(TimeUnit.NANOSECONDS.toSeconds(time) + "   SENCODS");
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

#### CyclicBarrier

```java
// pass

```
