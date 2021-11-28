---
title: 'Java concurrency 1 basic'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
  - concurrency
--- 

知识点太多了。先列举一些知识点，然后在分别做一点笔记。

#### 模式
1。 共享可变性  
2。 隔离可变性  
3。 纯粹不可变性:STM,  

#### IO密集型任务和计算密集型任务

读写文件和网络请求这种算IO密集型任务，阻塞时间长，任务阻塞系数接近1；线程池大一点好，
判断质数的这种任务属于计算密集型任务，阻塞系数约为0。  
poolSize = cores/(1-blockingCofficient); cores 是处理器核心数。

场景：  
根据网络服务api计算给定股票代码和股票数的资产总值。-IO密集   
判断n以内的所有素数。  -- 计算密集   

#### Java5以前的一些同步方法api

尽量不要使用，但是要理解。wait/notify、join等函数，synchronized volatile关键字的理解。笔记   

+ 用ExecutorService代替Thread及其方法。笔记  
+ 用Lock和子类的方法代替synchronized。但是不绝对。 笔记  
+ 以前用wait/notify的地方，现在可以用CyclicBaerrier和CountDownLatch同步工具代替。笔记

#### 同步容器和并发容器

同步容器包括Vector和Hashtable(java.util.Properties 也是一个HashTable)，使用synchronized同步。不建议使用，但是要知道HashTable和HashMap区别：   
Java 中 HashMap 和 HashTable 有几个不同点： 
+ Hashtable 是同步的，然而 HashMap 不是。 这使得HashMap更适合非多线程应用，因为非同步对象通常执行效率优于同步对象。  
+ Hashtable 不允许 null 值和键。HashMap允许有一个 null 键和一个 NULL 值。  
+ HashMap的一个子类是LinkedHashMap。所以，如果想预知迭代顺序（默认的插入顺序），只需将HashMap转换成一个LinkedHashMap。用Hashtable就不会这么简单。  
+  如果同步对你来说不是个问题，我推荐使用HashMap。如果同步成为问题，你可能还要看看ConcurrentHashMap。

迭代hashmap最佳方式：  
```
	 Iterator it = mp.entrySet().iterator();
	    while (it.hasNext()) {
	        Map.Entry pair = (Map.Entry)it.next();
	        System.out.println(pair.getKey() + " = " + pair.getValue());
	        it.remove(); // avoids a ConcurrentModificationException
	    }

```


并发容器是和java.util.concurrent包一块发布的。包括很多新的并发容器：  

[并发容器架构图](http://blog.csdn.net/fenglibing/article/details/37729335)  
图中最底部的都是jdk1.5增加的并发容器：主要有：ConcurrentHashMap,CopyOnWriteList, BlockingQueue等。


#### 参考书籍：

Doug Lea 《Concurrent Programming in Java》 2004  

Brian Goetz 《java concurrency in practice》 2007

Venkat 《Programming concurrency on the JVM》


