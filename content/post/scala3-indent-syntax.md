+++
title = "Scala3 缩进语法总结表"
date = "2022-02-02T10:19:05+08:00"
categories = [ "编程",]
tags = [ "scala3", "cheatsheet",]
toc = "true"
+++


Scala 3 在语法上面新增了一种 Python 的缩进格式，两种格式都可以使用。但是目前部分情况还是需要使用括号。
个人对新语法是支持的。缩进可以极大地提供代码的可读性和整洁，最大的体会就是 SparkStreaming 的 rdd 处理代码，新手容易写出十几个}括号嵌套代码。
当然缺点是缩进不利于代码复制和格式化。

下面是书本上关于 Scala3 的语法对比。注意，两个语法格式都是支持的。for 和 if 去掉小括号真的是太棒了。

<!--more-->

![scala3-indent-syntax](https://cdn.jsdelivr.net/gh/zhimoe/picx-images-hosting@master/pic/scala3-indent.67cs88jvxlw0.webp)