+++
title = "Java 并发 4-线程池与执行器"
date = "2018-06-13T09:57:52+08:00"
categories = [ "编程",]
tags = ["java", "并发", "thread-pool",]
toc = "true"
+++


### thread pool classes hierarchy

```text
java thread pool class hierarchy
Executor (java.util.concurrent)
|---ExecutorService (java.util.concurrent)
    |---AbstractExecutorService (java.util.concurrent)
    |   |---ForkJoinPool (java.util.concurrent)
    |   |---ThreadPoolExecutor (java.util.concurrent)
    |   |   |---ScheduledThreadPoolExecutor (java.util.concurrent)
    |---DelegatedExecutorService in Executors (java.util.concurrent)
    |---ScheduledExecutorService (java.util.concurrent)
```

<!--more-->

### three thread pool interfaces

`Executor`, a simple interface that supports launching new tasks.

`ExecutorService`, a sub-interface of Executor, which adds features that help manage the life cycle, both of the
individual tasks and of the executor itself.

`ScheduledExecutorService`, a sub-interface of ExecutorService, supports future and/or periodic execution of tasks.

### common thread pool implements

`ThreadPoolExecutor`是 thread pool 最常用的实现。一般通过`Executors`静态工厂方法来创建。

```java
//Executors.newFixedThreadPool
//Executors.newCachedThreadPool
//Executors.newSingleThreadExecutor

//同样的，Executors 还提供了 ScheduledExecutorService 的工具方法
//Executors.newSingleThreadScheduledExecutor
```

```java
/**
 * corePoolSize - 保留存活的线程个数
 * maximumPoolSize - 最大线程个数
 * keepAliveTime - 线程数超过 corePoolSize 时，空闲线程存活时间
 * unit - keepAliveTime 的单位，毫秒秒分等
 * workQueue – 任务队列，只保存通过 execute() 方法提交的 Runnable 任务
 * threadFactory – 给自己创建一个线程的工厂方法
 * handler – 当线程池达到数量限制或者任务队列满了，对新任务提交的处理策略
 */
class ThreadPoolExecutor {
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {}
}
```
JDK 默认的拒绝策略 RejectedExecutionHandler 有：
```java
/**
 * ThreadPoolExecutor.AbortPolicy - 默认的 handler，抛出一个 RejectedExecutionException
 * ThreadPoolExecutor.CallerRunsPolicy - 提交任务的线程自己执行这个任务
 * ThreadPoolExecutor.DiscardPolicy - 抛弃这个任务
 * ThreadPoolExecutor.DiscardOldestPolicy - 抛弃任务队列中最早提交上来的任务，然后尝试重新提交当前这个任务
 */

```

### 任务提交执行流程
![任务提交执行流程](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/threadpool.5d6mli4zovs0.svg)

### fork/join 框架
fork/join和上面ThreadPoolExecutor的区别在于使用了任务窃取算法，工作线程完成自己的任务后可以从其他线程偷取任务，提高整体的任务效率.
核心是一个`ForkJoinPool` class 和一个扩展的`AbstractExecutorService`.   执行`ForkJoinTask` 任务。
在 JDK8 中有个`java.util.Arrays.parallelSort()`使用的就是 fork/join.

当然，[不是所有人都满意 JDK7 引入的 Fork/Join 框架](http://coopsoft.com/ar/CalamityArticle.html).

