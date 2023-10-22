+++
title = "Java AOP example"
date = 2016-01-01
categories = [ "编程",]
tags = [ "java", "aop",]
toc = "true"
+++


Java AOP: 找到一个最简单的介绍，不怎么想翻译，直接看原文吧：
[A Simple Introduction to AOP](https://www.javacodegeeks.com/2012/06/simple-introduction-to-aop.html)

注意，使用注解的方式声明切面时，增加了一个空方法去定义 Pointcut，即：  
```java
class Test{
    @Pointcut("execution(* org.bk.inventory.service.*.*(..))")
    public void serviceMethods(){
        //这是一个空方法用于声明 Pointcut，
        //后续的 @Before @Around 方法都关联这个方法
    }
}

```
<!--more-->

在使用 xml 配置的话，就不需要这个方法了，serviceMethods 方法名是后面配置切点的引用。

如果不想引入 spring 的话，可以直接使用 aspectj 或者 jboss aop。
