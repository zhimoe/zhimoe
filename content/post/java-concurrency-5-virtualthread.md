+++
title = "Java并发5-虚拟线程（VirtualThread）"
date = "2023-09-13T09:57:52+08:00"
categories = [ "编程",]
tags = ["java", "并发", "virtual thread",]
toc = "true"
+++

回调和反应式编程都可以实现系统吞吐量有效提升，但是这两种编程模式存在阅读、编写、调试困难的问题，所以实际项目中还是以线程池为主。但是java的线程是平台线程，可以理解为并行线程数最多等于CPU核数(macOS查看核数`sysctl hw.physicalcpu hw.logicalcpu`)，并且存在线程内存占用大，上下文切换耗时高问题，所以在高并发请求中表现不如前面两种模式（spring reactive和vertx模式并没有流行起来）。
[JEP 444: Virtual Threads](https://openjdk.org/jeps/444) 主要目标在优化IO密集型任务时创建平台线程会消耗过多内存以及线程上下文切换耗时问题。

虚拟线程的优势： 1. 和线程API兼容（大部分兼容） 2. 降低应用内存使用，提升系统可用性，减少内存不足异常OutOfMemoryError: unable to create new native thread 3. 提升代码可读性（相比reactive编程）。

本文是VirtualThread快速笔记，包含API使用、限制和在Spring Boot的实际使用以及与Kotlin协程的对比。

<!--more-->

### VirtualThread API
创建虚拟线程有以下方法
```java
// 1
Runnable task = () -> { System.out.println("Hello Virtual Thread!"); };
Thread.startVirtualThread(task);

// 2 
Thread vThread = Thread.ofVirtual().start(task);

// 3
Thread vThread = Thread.ofVirtual().unstarted(task);
vThread.start();

// 4 
Executor vExecutor = Executors.newVirtualThreadPerTaskExecutor();
vExecutor.execute(task); // unlimited virtual threads

// 5 newThreadPerTaskExecutor but with VirtualThreadFactory
ThreadFactory vThreadFactory = Thread.ofVirtual().name("vt-", 1).factory();
Executor vExecutor = Executors.newThreadPerTaskExecutor(vThreadFactory);
vExecutor.execute(task);
```
虚拟线程相比平台线程，在创建耗时和内存占用具有很大优势。作为对比，同一台机器上创建1W个平台线程和虚拟线程。

| 类型             | 创建时间  | 内存占用  |
| --------------- | -------- | -------- |
| virtual thread  | 91 ms    | 4.4mb    |
| platform thread | 998 ms   | 14.3gb   |


### limitations of VirtualThread
下面说的carrier thread就是执行虚拟线程的系统线程（platform thread）。

1. Avoid synchronized blocks/methods, use `ReentrantLock`. 
   ```java
   Object monitor = new Object();
        //...
        public void aMethodThatPinTheCarrierThread() throws Exception {
        // The virtual thread cannot be unmounted because it holds a lock,
        // so the carrier thread is blocked.
        // also called pinned thread or pinning
        synchronized(monitor) {
            Thread.sleep(1000); 
        }
    }
   ```
2. Avoid monopolization. 即避免CPU密集型的任务使用虚拟线程。如果一个task耗时非常长，那么该虚拟线程对应的platform thread（即the carrier thread）无法让出去执行其他任务，JVM会创建新的线程。这种场景应该使用线程池技术。

3. Cation the carrier thread pool elasticity. 当前发生1或者2的情况时，JVM会创建新的系统线程，容易导致系统内存被耗尽。 

4. Avoid Object pooling, or reduce `ThreadLocal` Usage: 因为线程池数量有限制且线程会复用，所以创建比较耗时的对象会被池化以复用。但是虚拟线程不满足线程的这两个假设，池化对象并不能被复用。更糟糕的是，由于虚拟线程个数一般没有限制，每个虚拟线程都有ThreadLocal对象的话，可能耗尽JVM堆内存。[JEP 429: Scoped Values](https://openjdk.org/jeps/429) will fix this. 

5. 关注线程安全，虚拟线程本质还是多线程编程，和多线程一样需要关注共享状态问题。

### use VirtualThread in Spring Boot

```java
@SpringBootApplication
@Slf4j
public class VirtualthreadApplication {

    public static void main(String[] args) {
        SpringApplication.run(VirtualthreadApplication.class, args);
    }

    @Bean
    public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtualThreadExecutorCustomizer() {
        return protocolHandler -> {
            log.info("Configuring " + protocolHandler + " to use VirtualThreadPerTaskExecutor");
            protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        };
    }

}
```
### use VirtualThread in Quarkus

Using virtual threads in Quarkus is straightforward. You only need to use the `@RunOnVirtualThread` annotation. It indicates to Quarkus to invoke the annotated method on a virtual thread instead of a regular platform thread.

```java
@Path("/greetings")
public class VirtualThreadApp {

  @RestClient RemoteService service;

  @GET
  @RunOnVirtualThread
  public String process() {
    // Runs on a virtual thread because the
    // method uses the @RunOnVirtualThread annotation.

    // `service` is a rest client, it executes an I/O operation
    var response = service.greetings(); // Blocking, but this time, it
                                        // does neither block the carrier thread
                                        // nor the OS thread.
                                        // Only the virtual thread is blocked.
	return response.toUpperCase();
  }

}
```


[When Quarkus meets Virtual Threads](https://quarkus.io/blog/virtual-thread-1/#five-things-you-need-to-know-before-using-virtual-threads-for-everything))

###  internal and compare to kotlin coroutine
> A coroutine is an instance of suspendable computation.  - Kotlin doc
和kotlin的协程类似，java的虚拟线程同样不能自己执行，而是需要挂载到平台线程上面才能执行。下面是虚拟线程的生命周期:
```java
/*
    * Virtual thread state and transitions:
    *
    *      NEW -> STARTED         // Thread.start
    *  STARTED -> TERMINATED      // failed to start
    *  STARTED -> RUNNING         // first run
    *
    *  RUNNING -> PARKING         // Thread attempts to park
    *  PARKING -> PARKED          // cont.yield successful, thread is parked
    *  PARKING -> PINNED          // cont.yield failed, thread is pinned
    *
    *   PARKED -> RUNNABLE        // unpark or interrupted
    *   PINNED -> RUNNABLE        // unpark or interrupted
    *
    * RUNNABLE -> RUNNING         // continue execution
    *
    *  RUNNING -> YIELDING        // Thread.yield
    * YIELDING -> RUNNABLE        // yield successful
    * YIELDING -> RUNNING         // yield failed
    *
    *  RUNNING -> TERMINATED      // done
    */
    private static final int NEW      = 0;
    private static final int STARTED  = 1;
    private static final int RUNNABLE = 2;     // runnable-unmounted
    private static final int RUNNING  = 3;     // runnable-mounted
    private static final int PARKING  = 4;
    private static final int PARKED   = 5;     // unmounted
    private static final int PINNED   = 6;     // mounted
    private static final int YIELDING = 7;     // Thread.yield
    private static final int TERMINATED = 99;  // final state
```
![state of virtual thread](https://blog.rockthejvm.com/images/virtual-threads/virtual-thread-states.png)
绿色表示虚拟线程挂载（mounted）在 平台线程（carrier thread）。蓝色表示unmounted并让出线程（去执行其他虚拟线程或者任务）。 紫色表示pinned。
核心代码解读参考[Some Virtual Threads InternalsPermalink](https://blog.rockthejvm.com/ultimate-guide-to-java-virtual-threads/#8-some-virtual-threads-internals)


### 参考

[java 21 doc](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html#GUID-DC4306FC-D6C1-4BCC-AECE-48C32C1A8DAA)

[JEP 444: Virtual Threads](https://openjdk.org/jeps/444) 

[Why are Thread.stop, Thread.suspend and Thread.resume Deprecated?](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/doc-files/threadPrimitiveDeprecation.html)

[The Ultimate Guide to Java Virtual Threads](https://blog.rockthejvm.com/ultimate-guide-to-java-virtual-threads/)

[When Quarkus meets Virtual Threads](https://quarkus.io/blog/virtual-thread-1/#five-things-you-need-to-know-before-using-virtual-threads-for-everything))