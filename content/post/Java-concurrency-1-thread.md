+++
title = "Java 并发 1-线程与任务"
date = 2016-01-01
categories = [ "编程",]
tags = [ "java", "并发",]
toc = "true"
+++


### 基本概念

线程：任务执行的环境，可以理解为传送带。注意任务必须在线程上面被执行。

任务：Runnable/Callable 的实现，可以理解为传送带上面的工序。

资源：线程在处理任务具体工序时需要使用的对象，例如信号量，锁，并发集合。需要注意，任务本身描述也是一个对象（即 Runnable/Callable 子类实例），所以在 Runnable 代码里面你会看到 synchronized(this) 的用法，就是把任务描述本身当作一个资源，甚至 Thread 内部也可以将当前 thread 对象当做资源。

### 任务

多线程编程的核心元素就是任务，任务是独立的活动，不依赖其他任务的状态、结果、以及边界效应。定义任务的内容使用 Runnable 和 Callable。

Runnable 接口表示没有返回的一个过程（procedure），没有受检异常；
Callable 接口的 call 方法会返回一个结果，并有可能抛出受检异常。如果要表示没有返回值，可以使用`Callable<Void>`，但是不鼓励使用这个代替 Runnable，当一个任务内容没有返回值，只是利用副作用时，应该优先使用 Runable，使得含义清晰，并且 JDK 中`ScheduledExecutorService`也有只能接收 Runable 的方法。

Future 接口描述了任务的生命周期，并提供方法获得任务执行的结果。该接口有一个实现类：`FutureTask`.该类的实例一定和一个具体任务相关。`ExecutorService`所有的 submit 方法都会返回一个 Future 实例。你也可以直接通过 FutureTask 构造函数将 Runnable/Callable 对象构建成一个 FutureTask 实例，该实例将管理该任务的生命周期。

注意，FutureTask 实现了 Runnable 和 Future（通过实现 RunnableFuture 接口），所以既可以使用 ExecutorService，也可以使用 Thread 执行 FutureTask 任务内容。

```java
// FutureTask 接口关系
interface RunnableFuture<V> extends Runnable, Future<V> 

public class FutureTask<V> implements RunnableFuture<V> 
```

### 线程 Thread

创建一个线程的方法是 new Thread()，但是向线程提交任务的方式有两种：一是直接继承 Thread 将任务编码在自定义 Thread 的 run 方法里面；二是将 Runnable 实例传递给 Thread 构造函数，区别只不过是否将任务绑定在线程实例上而已，第二种方式更灵活实现了线程与任务的解耦，权责分明。
启动线程使用 t.start()，注意，如果是调用 t.run() 是在当前线程中执行任务，不是新线程。

线程的生命周期

![线程生命周期](https://cdn.jsdelivr.net/gh/zhimoe/picx-images-hosting@master/pic/java-thread-lifecycle.4w2mxew710c0.webp)

详细说明参考[Java 并发编程](https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/Java%20%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%2078%20%E8%AE%B2-%E5%AE%8C/03%20%E7%BA%BF%E7%A8%8B%E6%98%AF%E5%A6%82%E4%BD%95%E5%9C%A8%206%20%E7%A7%8D%E7%8A%B6%E6%80%81%E4%B9%8B%E9%97%B4%E8%BD%AC%E6%8D%A2%E7%9A%84%EF%BC%9F.md)

### Thread.join 方法

当调用 t.join 时，调用线程（calling thread）会暂停，一直到线程 t 终止或者抛出一个 InterruptedException。

    The join() method may also return if the referenced thread was interrupted.
    In this case, the join method throws an InterruptedException.

### ExecutorService

执行器框架，root 接口是 Executor，只有一个 execute 方法执行 runnable 实例。更常用是子接口 ExecutorService，除了可以执行 runnable,callable，还可以 invoke 一个 callable 集合：

```javadoc
<!-- ExecutorService methods -->
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
<T> T invokeAny(Collection<? extends Callable<T>> tasks)

<T> Future<T> submit(Callable<T> task)
Future<?> submit(Runnable task)
```

#### ScheduledExecutorService

The ScheduledExecutorService interface supplements the methods of its parent ExecutorService with schedule, which
executes a Runnable or Callable task after a specified delay. In addition, the interface defines scheduleAtFixedRate and
scheduleWithFixedDelay, which executes specified tasks repeatedly, at defined intervals.

scheduleAtFixedRate: 第一次是 initialDelay 后执行，第二次是 initialDelay + 1 * period 后执行，类推。

scheduleWithFixedDelay: 是前面任务执行结束后开始计算间隔计时。

两个方法都不会并发执行任务，特别是第一个方法，如果任务时间比参数中等待时间 period 长，那么只会延期执行。对于第二个方法，本来就是要等前面结束才执行，所以没有这个问题。两个方法遇到异常，那么后面任务也不会执行，因为任务是重复的，后面也会遇到异常。周期任务可以取消，或者遇到执行器终结才结束。

### 线程池如何实现复用的

其实非常简单 就是 Thread Worker 的 runWorker() 方法中有一个 while 循环不停获取 task 并调用 task.run 方法。后面单开一篇详细介绍。

### CompletionService

如果有多个任务，那么 ExecutorService 只能不停的轮询 Future 看是否有任务结束，并取得结果.CompletionService 则是另外是自动的告诉你那些任务结果已经准备好。注意构造方法需要一个 ExecutorService

```text
# 辅助理解
ExecutorService = incoming queue + worker threads
CompletionService = incoming queue + worker threads + output queue
```

[CompletionService 参考](http://stackoverflow.com/questions/4912228/when-should-i-use-a-completionservice-over-an-executorservice)

```java
ExecutorService executor = Executors.newFixedThreadPool(numberOfThreadsInThePool);
CompletionService<String> completionService = new ExecutorCompletionService<String>(executor);
for (final String num: nums) {
  completionService.submit(new Task(num)); //Task is Callable
}
try {
  for (int t = 0, n = nums.size(); t < n; t++) {
    Future<String> f = completionService.take();
    System.out.print(f.get());
  }
} catch (InterruptedException e) {
  Thread.currentThread().interrupt();
} catch (ExecutionException e) {
  Thread.currentThread().interrupt();
} finally {
  if (executor != null) {
    executor.shutdownNow();
  }
}
```

#### 参考书籍：

Doug Lea “Concurrent Programming in Java” 2004

Brian Goetz “java concurrency in practice” 2007

Venkat “Programming concurrency on the JVM”
