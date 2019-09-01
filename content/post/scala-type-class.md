---
title: "Scala Type Class"
date: "2019-03-31T12:58:17+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
---

### intro

```scala
def insertionSort(xs: List[Int]): List[Int] = {

  def insert(y: Int, ys: List[Int]): List[Int] =
    ys match {
      case List() => y :: List()
      case z :: zs =>
        if (y < z) y :: z :: zs
        else z :: insert(y, zs)
    }

  xs match {
    case List() => List()
    case y :: ys => insert(y, insertionSort(ys))
  }
}
```
假设有个排序算法如上，代码的第一个问题就是没有泛型，只能对Int排序。那如果改成`insertionSort[T](xs: List[T]): List[T]`显然不行，因为只有Number类型才有`y < z`方法。为了让这个函数实现多态（polymorphic ），需要将`y < z`这部分参数化。修改函数形式：

```scala
def insertionSort[T](xs: List[T])(lessThan: (T, T) => Boolean) = {
  def insert(y: T, ys: List[T]): List[T] =
    ys match {
      //...omit some lines
      case z :: zs =>
        if (lessThan(y, z)) y :: z :: zs
        else …
    }

  xs match {
    //...omit some lines
    case y :: ys => insert(y, insertionSort(ys)(lessThan))
  }
}
```
这样可以对任意类型排序，只要你提供排序方法lessThan.例如：

```scala
val fruits = List("apple", "pear", "orange", "pineapple")
insertionSort(fruits)((x: String, y: String) => x.compareTo(y) < 0)
```
既然lessThan方法如此重要，我们定义一个trait:
```scala
trait Ordering[A]{
  def lessThan(x:A,y:A):Int
}
```
然后排序函数接受一个Ordering实例进行排序：
```scala
  def insertionSort[T](xs: List[T])(ord: Ordering[T]): List[T] = {
    def insert(y: T, ys: List[T]): List[T] =
      ys match {
        case List() => y :: List()
        case z :: zs =>
          if (ord.lessThan(y, z)) y :: z :: zs
          else z :: insert(y, zs)
      }

    xs match {
      case List() => List()
      case y :: ys => insert(y, insertionSort(ys)(ord))
    }
  }
```
现在这个排序函数比较完美了，如果借助隐式参数，那么调用可以更简洁：
```scala
  def insertionSort[T](xs: List[T])(implicit ord: Ordering[T]): List[T] = {
    def insert(y: T, ys: List[T]): List[T] =
      ys match {
        case List() => y :: List()
        case z :: zs =>
          if (ord.lessThan(y, z)) y :: z :: zs
          else z :: insert(y, zs)
      }

    xs match {
      case List() => List()
      case y :: ys => insert(y, insertionSort(ys))
    }
  }

//implicit val for insertionSort function
implicit val stringOrdering: Ordering[String] = new Ordering[String] {
  def lessThan(x: String, y: String): Boolean = x.compareTo(y) < 0
}

val fruits = List("apple", "pear", "orange", "pineapple")
insertionSort(fruits)
```
`insertionSort(fruits)`的调用形式看上去不够优雅（不够OO),我们希望调用的形式是：`fruits.insertionSort` 得到排序后的list。通过之前文章的隐式转换很容易想到，我们需要将insertionSort方法作为一个implicit class的方法，这样即可通过隐式转换得到`fruits.insertionSort`:
```scala
//1. 注意原函数的参数改为class参数传入
//2. 由于原函数用了递归，所以这里加了一层函数
implicit class FruitOps[T](list: List[T])(implicit ord: Ordering[T]) {
  def insertionSort(): List[T] = {
    insertSortInternal(list)(ord)
  }

  def insertSortInternal[T](xs: List[T])(implicit ord: Ordering[T]): List[T] = {

    def insert(y: T, ys: List[T]): List[T] =
      ys match {
        case List() => y :: List()
        case z :: zs =>
          if (ord.lessThan(y, z)) y :: z :: zs
          else z :: insert(y, zs)
      }

    xs match {
      case List() => List()
      case y :: ys => insert(y, insertSortInternal(ys))
    }
  }

}
//
val fruits = List("apple", "pear", "orange", "pineapple")
fruits.insertionSort //List(pear, apple, orange, pineapple)
```
### type class
类型参数化和隐式参数结合即scala的type class，实现在不修改原有代码的前提下对type(这里是List[String])增加行为和属性。例如，已有类型Person，想要增加一个print方法实现打印个人简历。如果通过继承，那么Person代码会被改变，且print方法不能随意替换。如果通过Printer工具类同样Person只能有一种print实现（或者多个Printer工具类）。通过type class和，可以实现`person.print`的调用格式，且print的具体实现是由当前scope的implicit type class instance决定，在不同的scope可以通过提供不同的type class实例达到不同的print效果。

type class由3部分组件：
+ type class：a trait with type parameter,即Ordering[T]
+ type class instance: 即上面的val stringOrdering: Ordering[String]，一般为implicit
+ interface for user: 即上面的FruitOps，是一个implicit class.

### one more example
```scala
class Person(val name: String, val address: String)
//type class
trait HtmlWriter[A] {
  def write(data: A): String
}

// a type class instance
implicit object PersonWriter extends HtmlWriter[Person] {
  override def write(data: Person): String = {
    s"<span>${data.name} and ${data.address} </span> "
  }
}

// type class interface for user
implicit class HtmlUtil[A](data:A){
  def toHtml(implicit writer: HtmlWriter[A]):String={
    writer.write(data)
  }
}

val p = new Person("xiongdahu","beijing")
p.toHtml

```