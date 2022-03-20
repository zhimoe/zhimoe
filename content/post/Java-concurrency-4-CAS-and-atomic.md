---
title: 'Java concurrency 4 CAS and atomic'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 
  
Java并发编程笔记4

<!--more-->

#### AtomicLong code:
```java
public final long incrementAndGet() {
    for (;;) {
        long current = get();
        long next = current + 1;
        if (compareAndSet(current, next))
          return next;
    }
}
//in java 8:
public final long incrementAndGet() {
        return unsafe.getAndAddLong(this, valueOffset, 1L) + 1L;
}

```

#### 基础

第一个版本是基于cas的,cas基于一个基础：有三个值,新值N,预期内存中的值E,内存中需要更新的值V,如果V == E,那么将V设置为N,返回V,结束；如果V != E,说明有别的线程动了这个v,那么不做修改直接返回V.cas在X86下对应的是 CMPXCHG 汇编指令

java8中则使用了x86的优化指令atomic fetch-and-add ,上面的代码直接等价于cpu的一条指令atomic fetch-and-add .[性能更好](atomic fetch-and-add vs compare-and-swap)

而compareAndSet利用JNI来完成CPU指令的操作.

```java
public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```


#### 注意

java.util.concurrent.atomic中的原子类使用了很多cas,但是这个方法一个是自己实现和使用需要很仔细,另一个在真的高并发中可能陷入死循环,因为方法中本身就是一个死循环：`for (;;)`.java8为此提供了LongAdder.

关于垃圾自动回收的语言不会出现cas中aba问题的原理：[stackoverflow](http://stackoverflow.com/questions/19660737/aba-in-lock-free-algorithms)

