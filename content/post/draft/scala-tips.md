---
title: "Scala常见最佳实践"
date: "2020-02-08T20:06:03+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
draft: true

---

发现一个非常细心的Scala最佳实践总结,翻译做一个笔记.

<!--more-->
[scala-best-practices](https://nrinaudo.github.io/scala-best-practices/)

## 使用余数是否为0判断偶数
判断奇数使用 `x % 2 != 0` 比使用 `x % 2 == 1`更加准确,因为`-3 % 2 = -1`. 偶数的定义是除以2余数为0,奇数的定义是不是偶数的数.
>这条适用所有语言.

## 使用 isNaN,而不是 ==NaN
根据scala规范,NaN不等于任何对象,甚至自己.`Double.NaN == Double.NaN` 是`false`.所以你应该使用`isNaN`
```scala
def slowFilterNaN(ds: Seq[Double]): Seq[Double] = ds.filter(d => !d.isNaN)
// the following is faster than previous one
def fastFilterNaN(ds: Seq[Double]): Seq[Double] = ds.filter(d => d == d)
```

## 