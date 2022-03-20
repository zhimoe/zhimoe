---
title: 'Java concurrency 3 synchronized or Lock'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

### synchronized method和synchronized block的区别
如果是synchronized(this),那么和synchronized 方法没有任何区别,锁定对象都是方法所在的对象.

```java
synchronized void mymethod() { ... }

void mymethod() {
  synchronized (this) { ... }
}
```
<!--more-->

但是synchronized block可以锁定其他对象,而且synchronized block的范围是可以控制更灵活,synchronized 方法的边界只能是整个方法

```java
 private void method() {
    //  code here
    //  code here
    //  code here
    synchronized( lock ) { 
        // very few lines of code here
    }     
    //  code here
    //  code here
    //  code here
}
```

### 不要忘记synchronized 
 
这个指令是JVM内置的,也是未来可以优化的.如果只是简单的同步一个资源对象,就使用synchronized,而且,使用Lock有就必须出现一堆的try/finally.

使用ReentrantLock场景：  
需要以下高级特性时 ： 可定时的,可轮询的,可中断的锁,公平队列,非块结构.

[stackoverflow回答](http://stackoverflow.com/questions/4201713/synchronization-vs-lock)
