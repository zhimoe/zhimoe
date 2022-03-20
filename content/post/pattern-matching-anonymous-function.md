---
title: "Pattern Matching Anonymous Function"
date: "2019-03-31T13:10:41+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
---
Scala中模式匹配匿名函数

<!--more--> 

Scala中很多使用if的地方都可以用match case来替换.常见的就是下面的这种写法:
```scala
val res = msg match {
	case it if it.contains("H") => "Hello"
	case _ => "Other"
}
//更常见的用法是去匹配参数的模式:
case class Player(name: String, score: Int)
def message(player: Player) = player match {
  case Player(_, score) if score > 100000 =>
    "Get a job, dude!"
  case Player(name, _) =>
    "Hey, $name, nice to see you again!"
}
def printMessage(player: Player) = println(message(player))
```
其实case还有一种在匿名函数中的用法,看如下的代码,在词频统计或者过滤中很常见:
```scala
val wordFrequencies = ("habitual", 6) :: ("and", 56) :: ("consuetudinary", 2)  :: Nil
def wordsWithoutOutliers(wordFrequencies: Seq[(String, Int)]): Seq[String] =
  wordFrequencies.filter(wf => wf._2 > 3 && wf._2 < 25).map(_._1)
```
上面的代码有比较大的问题是访问tuple元素的方式比较难看,Scala提供了一种pattern matching anonymous function解决这个问题:
```scala
def wordsWithoutOutliers(wordFrequencies: Seq[(String, Int)]): Seq[String] =
 wordFrequencies.filter { case (_, f) => f > 3 && f < 25 } map { case (w, _) => w }
```
注意到省略了最早版本的 `wf =>`,IDEA其实会提示你省略这个冗余部分. 
另一个问题就是上面的操作中我们先过滤想要的序列,然后对序列进行了map映射操作.Scala 集合的 API 有一个叫做 `collect` 的方法,对于 `Seq[A]` ,它有如下方法签名：

```scala
def collect[B](pf: PartialFunction[A, B]): Seq[B]
```
这个方法将给定的_偏函数(partial function)_ 应用到序列的每一个元素上, 最后返回一个满足条件并处理后新的序列 ,这里偏函数做了 `filter` 和 `map` 要做的事情.
现在,我们来重构 `wordsWithoutOutliers` ,首先定义需要的偏函数：

```scala
val pf: PartialFunction[(String, Int), String] = {
 case (word, freq) if freq > 3 && freq < 25 => word
}
wordFrequencies.collect(pf)
```
我们为这个案例加入了 _守卫语句_,不在区间里的元素就没有定义.
以上来自[Scala初学者指南](http://danielwestheide.com/scala/neophytes.html)
当然有中文版:[Scala初学者指南-gitbook](https://www.gitbook.com/book/windor/beginners-guide-to-scala/details)

