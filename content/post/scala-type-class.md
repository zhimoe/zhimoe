+++
title = "Scala Type Class"
date = "2019-03-31T12:58:17+08:00"
categories = [ "编程",]
tags = [ "code", "scala",]
toc = "true"
+++

 
scala type class notes: 
[关于 scala type class 非常好的文章](https://scalac.io/typeclasses-in-scala)

<!--more-->

## 核心知识点
```scala
//scala 没有专门的 type class 语法，而是借助 trait + implicit + context bound 来实现的，
//所以很多时候识别 type class 比较困难。
//type class 由三部分构成
//1. type class: 即下面的 Show，定义一个行为 toHtml.
//2. type class instances：希望实现 toHtml 方法的类型实例
//3. user interface: type class 中伴生对象的同名方法或者隐式转换方法。

// type class
trait Show[A] {
  def toHtml(a: A): String
}

// 定义在伴生对象的好处就是 implicit 变量自动处于 scope 内
object Show {
  //利用伴生对象 apply 特点实现下面 Show[A].toHtml 调用方式，即隐藏 implicit sh
  def apply[A](implicit sh: Show[A]): Show[A] = sh

  //如果没有 apply，那么下面的 toHtml 需要一个隐式参数：
  //def toHtml[A](a: A)(implicit sh: Show[A]): String = sh.toHtml(a)
  //或者：
  //def toHtml[A: Show](a: A): String = implicitly[Show[A]].toHtml(a)


  //对外接口，提供 toHtml("type") 的调用形式
  def toHtml[A: Show](a: A) = Show[A].toHtml(a)
  
  //对外接口，通过隐式转换提供 10 toHtml 的调用形式
  implicit class ShowOps[A: Show](a: A) { // 惯例使用 TypeCls+Ops
    def toHtml = Show[A].toHtml(a)
  }

  //为了避免运行开销，可以将 Ops 类定义为 value class：
  //  implicit class ShowOps[A](val a: A) extends AnyVal {
  //    def toHtml(implicit sh: Show[A]) = sh.toHtml(a)
  //  }

  //上面两个对外接口都利用了伴生对象的 apply 方法和 context bound

  //type class instance int
  implicit val intCanShow: Show[Int] =
    int => s"<int>$int</int>"

  //type class instance string
  implicit val stringCanShow: Show[String] =
    str => s"<string>$str</string>"
}

//use type class
import Show._
print(10 toHtml)
print(toHtml("type"))
```

若使用`import Show._ `导入全部内容，则用户无法自己实现一些 type class instance 则会覆盖默认实例导致歧义。
可以将对外接口移动到单独的 ops 对象中：
```scala
trait Show[A] {
  def toHtml(a: A): String
}

object Show {
  def apply[A](implicit sh: Show[A]): Show[A] = sh

  object ops {
    def toHtml[A: Show](a: A) = Show[A].toHtml(a)

    implicit class ShowOps[A: Show](a: A) { // 惯例使用 TypeCls+Ops
      def toHtml = Show[A].toHtml(a)
    }

  }

  implicit val intCanShow: Show[Int] = int => s"<int>$int</int>"
  implicit val stringCanShow: Show[String] = str => s"<string>$str</string>"
}
```
使用：
```scala
import xxx.Show //如果需要实现自定义的 type class instance 则需要
import xxx.Show.ops._
```

## 自定义
```scala
    case class Person(name: String, age: Int)
    implicit val personOps: Show[Person] = p => s"<person>name=${p.name},age=${p.age}</person>"

    print(Person("lee", 19) toHtml)//<person>name=lee,age=19</person>

```

## Simulacrum

[Simulacrum](https://github.com/typelevel/simulacrum)通过宏为 type class 添加便捷语法，是否使用取决于个人判断。
若使用 Simulacrum，则可以一眼找出代码中所有的 type class，并且可省去很多样板代码。
另一方面，使用 `@typeclass`（Simulacrum 主要注解）则意味着需要依赖 macro paradise 编译器插件。

使用 Simulacrum 重写我们的 Show type class：
```scala
import simulacrum._

@typeclass trait Show[A] {
  def toHtml(a: A): String
}
```
有了 Simulacrum,type class 定义变得非常简洁，我们在其伴生对象中添加 Show[String] 实例：

```scala
//Simulacrum 会为 Show 自动生成 ops 对象，与前面自定义的基本一致。
object Show {
  implicit val stringShow: Show[String] = s ⇒ s"String: $s"
}
```