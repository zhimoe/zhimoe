---
title: "Scala Collection Tips"
date: "2019-05-19T16:49:14+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
---
scala collection 提供了一整套独立于Java的高性能集合,使用上非常灵活,所以需要清楚一些常用的方法:


* [x] reduce fold scan
* [x] 集合的符号方法
* [x] 数组,tuple
* [x] 2.13的集合架构

## reduce fold scan
```scala
//reduce是一个二元函数,遍历整个集合
List(1, 3, 5).reduceLeft(_ + _)
// == ((1+3)+5)
//reduceRight start from end of the collection
//also you can given initial argument
List(1, 3, 5).foldLeft("")(_ + _)
// == 135
//foldLeft 等价于 \: 操作符
(0 /: List(1, 3, 5)) (_ - _)

//folding 常用于替代for-loop
val wf1 = scala.collection.mutable.Map[Char, Int]()
for (c <- "Mississippi") wf1(c) = wf1.getOrElse(c, 0) + 1
// Now freq is Map('i' -> 4, 'M' -> 1, 's' -> 4, 'p' -> 2)

//注意使用了不可变map
val wf = (Map[Char, Int]() /: "Mississippi") {
  (m, c) => m + (c -> (m.getOrElse(c, 0) + 1))
}

//scan 方法可以获得每一步中间结果集
(1 to 10).scanLeft(0)(_ + _)
//Vector(0, 1, 3, 6, 10, 15, 21, 28, 36, 45, 55)

```

## 集合的符号方法

```scala
//+ 表示添加一个元素到无序集合
// :+ +:表示添加到有序集合的首/尾
//elem append or prepend to coll (Seq)
coll :+ elem
elem +: coll

//add elem to set/map
coll + elem
coll + (e1,e2,...)


coll ++ coll2
coll2 ++: coll

// prepend to lst
elem :: lst 
lst2 ::: lst
// 等价list ++: list2
list ::: list2

// 含有=的表示修改,必须是mutable的集合

// TIP: As you can see, Scala provides many operators for adding and removing
// elements. Here is a summary:
// 1. Append (:+) or prepend (+:) to a sequence.
// 2. Add (+) to an unordered collection.
// 3. Remove with the - operator.
// 4. Use ++ and -- for bulk add and remove.
// 5. Mutations are += ++= -= --=.
// 6. For lists, many Scala programmers prefer the :: and ::: operators.
// 7. Stay away from ++: +=: ++=:.
```
NOTE: For lists, you can use +: instead of :: for consistency, with one
exception: Pattern matching (case h::t) does not work with the +: operator.

## 其他

```scala
//数组的笔记
val ints = new Array[Int](30) // empty array
val ints2 = Array[Int](1, 2, 3, 4) // array with init values
val matrix4x9 = Array.ofDim[Double](4, 9)
//update
ints2(3) = 1000
// or
ints2.update(3, 1000)
//求和
val ints2Sum = ints2.sum

val days = Array("Monday", "Tuesday", "Wednesday", "Thrusday", "Friday", "Saturday", "Sunday")
//遍历
for (i <- 0 until days.length) println(days(i))
for (day <- days) println(day)
days foreach println

//遍历中使用index
days.zipWithIndex.map { case (e, i) => (i, e) }
//faster
for (i <- days.indices) yield (i, days(i))
//Possibly fastest
Array.tabulate(days.length) { i => (i, days(i)) }
//肯定最快
val b = new Array[(Int, String)](days.length)
var i = 0
while (i < days.length) {
  b(i) = (i, days(i))
  i += 1
}

//filter
days.filter(day => day.length > 4)
//map
Array(1, 2, 3, 4, 5).map(x => x * x)
//sort
Array(3, 6, 2, 0, 8, 5).sortWith((e1, e2) => e1 < e2) //小的在前
//reduce,下面的会提示使用sum,
Array(1, 2, 3, 4, 5).reduce((e1, e2) => e1 + e2)

//不定长数组
import scala.collection.mutable.ArrayBuffer

val arr = ArrayBuffer[Int]()


//tuple
val oneAndTwo = (1, 2)
val oneAndTwo1 = Tuple2(1, 2)
//Pair is alias of Tuple2
val oneAndTwo2 = Pair(1, "two")
val oneAndTwo3 = 1 -> 2
//访问元素下标是从1开始,这是因为tuple里面每个元素类型不一样,为了能够和list等区分开
//使用了类似Haskell/ML的习惯
val two = oneAndTwo._2

//option
val emptyOpt: Option[Int] = None
val fullOpt: Option[Int] = Some(42)
emptyOpt match {
  case Some(value) => println(value)
  case None => println("Empty")
}
fullOpt.get //42
emptyOpt.isEmpty //true

//either
def divide(a: Double, b: Double): Either[String, Double] = {
  if (b == 0.0) Left("Division by zero") else Right(a / b)
}
divide(4, 0)


def either(flag: Boolean): Either[String, List[Int]] = {
  if (flag) Right(List(1, 2, 3))
  else Left("Wrong")
}

val content = either(true).right.map(_.filter(_ > 0))

//cast
Seq(1).toArray
Seq(1).toBuffer
Seq(1).toList
Seq((1, 2)).toMap
Seq(1).toStream
Seq(1).toString
Seq(1).toVector

Seq(1).toTraversable
Seq(1).toIndexedSeq
Seq(1).toIterable
Set(1).toSeq
Seq(1).toSet

//zip, zipAll, zipWithIndex, unzip
"abcde" zip 1.to(5)
//zipAll:第二个参数是调用者元素缺失使用的默认值,第三个参数是第一个实参不够长的默认值
"abcde".zipAll(1.to(2), "caller", "arg")
//尝试自己实现一个zipAll?
//
"abcde" zipWithIndex

Seq((1, 2), (3, 4), (5, 6)) unzip
//
val s = Seq("a", "b")
```

## scala 2.13 collection
基本重写了.参考这两个文档:
[collections migration 2.13](https://docs.scala-lang.org/overviews/core/collections-migration-213.html)
[the architecture of scala 2.13’s collections](https://docs.scala-lang.org/overviews/core/architecture-of-scala-213-collections.html)
