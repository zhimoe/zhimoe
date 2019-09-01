---
title: 'Java concurrency 2 Runnable Callable FutureExecutor'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

### 定义任务的内容  

多线程编程的核心元素就是任务，任务是独立的活动。不依赖其他任务的状态，结果，以及边界效应。  
定义任务的内容使用Runnable和Callable。

Runnable 接口表示没有返回的一个过程（procedure），没有受检异常。  
Callabe 接口的call方法会返回一个结果，并有可能抛出受检异常。如果要表示没有返回值，可以使用`Callable<Void> `,但是不鼓励使用这个代替Runable，但一个任务内容没有返回值，只是利用副作用时，应该优先使用Runable，使得含义清晰，并且JDK中ScheduledExecutorService也有只能接收Runable的方法。   

可以将Runnable定义的任务提交给Thread直接运行，但是这个线程是不可重用的。更好的方法是提交给执行器ExecutorService。

Future接口描述了任务的生命周期，并提供方法获得任务执行的结果。该接口有一个实现类：FutureTask。该类的实例一定和一个具体任务相关。ExecutorService所有的submit方法都会返回一个Future实例。你也可以直接通过FutureTask构造函数将Runnable/Callable构建一个FutureTask实例。该实例将管理该任务的生命周期

注意，FutureTask   实现了Runnable和Future（通过实现RunnableFuture<V> 接口，如下），所以既可以使用ExecutorService，也可以使用Thread执行任务内容。
```java
public class FutureTask<V> implements RunnableFuture<V>   
public interface RunnableFuture<V> extends Runnable, Future<V>
```

Future.get是一个阻塞方法，如果任务没有结束或者没有抛出异常，那么会一直等待下去，如果需要异步的使用ComletionService。

### ExecutorService

执行器框架，root 接口是Executor，只有一个execute方法执行runnable实例。更常用是子接口ExecutorService，除了可以执行runnable，callable，还可以invoke一callable集合：  
```java
<T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks)
<T> T	invokeAny(Collection<? extends Callable<T>> tasks)

<T> Future<T>	submit(Callable<T> task)
Future<?>	submit(Runnable task)
```

### ScheduledExecutorService  

The `ScheduledExecutorService` interface supplements the methods of its parent `ExecutorService` with schedule, which executes a `Runnable` or `Callable` task after a specified delay. In addition, the interface defines `scheduleAtFixedRate` and `scheduleWithFixedDelay`, which executes specified tasks repeatedly, at defined intervals.

`scheduleAtFixedRate`: 第一次是initialDelay 后执行，第二次是initialDelay + 1 * period 后执行，类推。

`scheduleWithFixedDelay`: 是前面任务执行结束后开始计算间隔计时。  

两个方法都不会并发执行任务，特别是第一个方法，如果任务时间比参数中等待时间period长，那么只会延期执行。对于第二个方法，本来就是要等前面结束才执行，所以没有这个问题。两个方法遇到异常，那么后面任务也不会执行，因为任务是重复的，后面也会遇到异常。周期任务可以取消，或者遇到执行器终结才结束。  

### CompletionService

如果有多个任务，那么ExecutorService只能不停的轮询Future看是否有任务结束，并取得结果。CompletionService则是另外是自动的告诉你那些任务结果已经准备好。注意构造方法需要一个ExecutorService

>   ExecutorService = incoming queue + worker threads    
    CompletionService = incoming queue + worker threads + output queue
    

[参考](http://stackoverflow.com/questions/4912228/when-should-i-use-a-completionservice-over-an-executorservice)

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

