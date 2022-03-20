---
title: 'Java AOP example'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - aop
--- 

Java AOP笔记

<!--more-->

找到一个最简单的介绍,不怎么想翻译,直接看原文吧:
[A Simple Introduction to AOP](https://www.javacodegeeks.com/2012/06/simple-introduction-to-aop.html)

提醒个点,使用注解的方式写切面时,增加了一个空方法,即：  
```java

    @Pointcut("execution(* org.bk.inventory.service.*.*(..))")
    public void serviceMethods(){

    }

```
在使用xml配置的话,就不需要这个方法了,serviceMethods方法名是后面配置切点的引用.

如果不想引入spring的话,可以直接使用aspectj或者jboss aop.
