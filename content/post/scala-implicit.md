---
title: "Scala 2 Implicit"
date: "2019-03-30T12:58:17+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
---
 
scala 2 implicit notes

<!--more-->

## 隐式参数
```scala
//隐式参数是在调用时可以自动填充的参数, 需要在调用范围内（scope)有一个隐式变量可供填充.
def addInt(i:Int)(implicit n: Int) = i + n


//需要提供一个隐式变量n
implicit val sn = 1
addInt(2) // 3

//如果有两个满足类型的隐式变量,则在编译addInt(2)时报错

//scala的方法中ExecutionContext一般作为implicit参数.
```

## 隐式转换方法
如果想要给String实现一个mkStr方法,简单的给String添加一个Ops!前缀再返回.

```scala
//1. 首先实现一个包含目标方法的类型,实现该方法
class StrOps(s:String) {
  def mkStr(): String = {
    return "Ops! " + s
  }
}

//2. 告诉scala编译器String可以通过类型转换获得mkStr这个方法：
implicit final def string2StrOps(s: String) = new StrOps(s)

//3. 现在用户可以直接认为String有mkStr方法
val s = "who changed my string"
s.mkStr() //res2: String = Ops! who changed my string

//在scala.Predef中定义了大量的隐式转换,例如RichInt,RichDouble,StringOps这些
```

## implicit class
可以看到第2步非常的冗余,于是[SIP-13](https://link.zhihu.com/?target=https%3A//docs.scala-lang.org/sips/implicit-classes.html)提出一个implicit class,将上面的1,2步合并:
```scala
implicit class StrOps(s:String) {
  def mkStr(): String = {
    return "Ops! " + s
  }
}
```
注意,这个只是一个语法糖.去糖后就是上面的那个形式. implicit class有3个约束和一个注解问题：

1. 必须要有主一个构造函数且只能一个构造参数（implicit参数除外）.构造参数就是源类型. 这个构造函数即等价上面第2步的隐式转换方法：
    ```scala
    implicit class RichDate(date: java.util.Date) // OK!
    implicit class Indexer[T](collecton: Seq[T], index: Int) // BAD!
    implicit class Indexer[T](collecton: Seq[T])(implicit index: Index) // OK!
    ```
2. 只能定义在其他trait/class/object中：
    ```scala
    object Helpers {
        implicit class RichInt(x: Int) // OK!
    }
    implicit class RichDouble(x: Double) // BAD!
    ```

3. 在当前scope内,不允许有和implicit class同名的方法,对象,变量.因为case class会自动生成同名object对象,所以implicit class不能是case class.
    ```scala
    object Bar
    implicit class Bar(x: Int) // BAD!

    val x = 5
    implicit class x(y: Int) // BAD!

    //cuz case class has companion object by default 
    implicit case class Baz(x: Int) // BAD! conflict with the companion object
    ```

4. 还有就是implicit class的注解在去语法糖后会自动添加到类和方法,除非在注解中指明范围：
    ```scala
    @bar
    implicit class Foo(n: Int)

    //desugar
    @bar implicit def Foo(n: Int): Foo = new Foo(n)
    @bar class Foo(n:Int)

    //除非在注解中指明：genClass / method
    @(bar @genClass) implicit class Foo(n: Int)

    //desugar得到
    @bar class Foo(n: Int)
    implicit def Foo(n: Int): Foo = new Foo(n)
    ```


## implicitly方法

scala的PreDef中有有一个implicitly方法,表示在当前scope征召一个隐式变量并返回该变量.
```scala
//PreDef
@inline def implicitly[T](implicit e: T) = e
```
> implitly[T] means return implicit value of type T in the context
```scala
implicit class Foo(val i: Int) {
   def addValue(v: Int): Int = i + v
} 

implicit val foo:Foo = Foo(1)
val fooImplicitly = implicitly[Foo] // Foo(1)

```

## value class

scala 还有一个概念：[value class](https://link.zhihu.com/?target=https%3A//docs.scala-lang.org/overviews/core/value-classes.html)
```scala
class Wrapper(val underlying: Int) extends AnyVal
//1. 一个public val参数表示runtime类型,这里是Int. 编译时是Wrapper类型,所以value class目的是降低分配开销.
//2. value class 需要 extends AnyVal
//3. value class 只能有 defs, 不能有vals, vars, or nested traits, classes or objects,
//   因为def是通过静态方法实现的,而val,var这些则必须创建相应类型了.
//4. value class 只能扩展 通用trait（universal traits）,
//   universal traits是A universal trait is a trait that extends Any, only has defs as members, and does no initialization.
```

## extension method

当implicit class类型参数是`AnyVal`子类时,value class和上面的implicit class形式相近,所以可以通过value class降低implicit class的分配开销.例如RichtInt
```scala
implicit class RichInt(val self: Int) extends AnyVal {
  def toHexString: String = java.lang.Integer.toHexString(self)
}
```
因为RichInt是value class,在运行时（runtime）不会有RichInt这个类,而是Int,而3.toHexString实际是通过静态方法实现的： `RichInt$.MODULE$.extension$toHexString(3)`,这么做好处是减少对象分配开销（avoid the overhead of allocation）.如果implicit class的类型参数不是AnyVal子类,那么在runtime时会有相应类型对象被创建,用户察觉不到区别.

value class还有其他作用和局限性,可以参考上面链接.