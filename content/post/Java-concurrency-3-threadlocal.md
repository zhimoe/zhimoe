---
title: 'Java并发编程-ThreadLocal'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
  - 并发
--- 


在通常的业务开发中，ThreadLocal 有两种典型的使用场景。

场景1，ThreadLocal 用作保存每个线程独享的对象，为每个线程都创建一个副本，这样每个线程都可以修改自己所拥有的副本, 而不会影响其他线程的副本，确保了线程安全。

场景2，ThreadLocal 用作每个线程内需要独立保存信息，以便供其他方法更方便地获取该信息的场景。每个线程获取到的信息可能都是不一样的，前面执行的方法保存了信息后，后续方法可以通过 ThreadLocal 直接获取到，避免了传参，类似于全局变量的概念。

<!--more-->

### 场景1： 保存线程不安全的工具类
注意：实际开发中使用DateTimeFormatter 代替 SimpleDateFormat.

```java
import java.text.SimpleDateFormat;
import java.util.Random;

public class ThreadLocalExample implements Runnable{

    // SimpleDateFormat is not thread-safe, so give one to each thread
    private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>(){
        @Override
        protected SimpleDateFormat initialValue()
        {
            return new SimpleDateFormat("yyyyMMdd HHmm");
        }
    };
    
    public static void main(String[] args) throws InterruptedException {
        ThreadLocalExample obj = new ThreadLocalExample();
        for(int i=0 ; i<10; i++){
            Thread t = new Thread(obj, ""+i);
            Thread.sleep(new Random().nextInt(1000));
            t.start();
        }
    }

    @Override
    public void run() {
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" default Formatter = "+formatter.get().toPattern());
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //formatter pattern is changed here by thread, but it won't reflect to other threads
        formatter.set(new SimpleDateFormat());
        
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" formatter = "+formatter.get().toPattern());
    }

}

```
### 场景2：不同线程保存独立信息
例如需要记录mysql的SQL执行耗时以及其他相关信息

```java
public class MysqlQueryInterceptor implements com.mysql.cj.interceptors.QueryInterceptor {
    private final ThreadLocal<LocalDateTime> startTimeHolder = new ThreadLocal<>();

    @Override
    public <T extends Resultset> T preProcess(Supplier<String> supplier, Query query) {
        startTimeHolder.set(LocalDateTime.now());
        return null;
    }
}
```
上面的ThreadLocal也可以使用slf4j的MDC(Mapped Diagnostic Context)。