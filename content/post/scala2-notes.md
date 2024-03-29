+++
title = "Scala 学习笔记"
date = "2019-03-31T00:11:50+08:00"
categories = [ "编程",]
tags = [ "code",]
toc = "true"
+++

some notes on scala, includes:
* [x] setup with maven
* [x] import
* [x] == and eq
* [x] case class
* [x] for...yield
* [x] companion object and class
* [x] method and function(def val)
* [x] _ in scala
* [x] => in scala
* [x] () {} in scala
* [x] implicit 
* [x] string

<!--more-->
## setup with maven

目前用 sbt 的项目比较少，maven 的更多。而且 sbt 烧 cpu. maven 项目使用 scala 参考我的 gist:[scala_maven_pom.xml](https://gist.github.com/zhimoe/db6caeed070bfabe70102e2bace0e5b0)

学习 scala 可以使用 scala 插件的 worksheet，这是一个基于脚本互动的 REPL. 本文后面的代码全部在 worksheet 中测试。

## import

scala 的 import 语句很灵活，可以在任何地方导入 class 内部外部，方法内部，代码块内部，这样做有一个好处，限制导入方法和对象的 scope，防止污染变量。在后面学了 implicit 隐式转换后，就知道 import scope 有多重要了。
```scala
import scala.math._ // import everything in math package 
import java.util.{ ArrayList => _, _} 
//第一个下划线表示隐藏 ArrayList，第二个表示通配符，导入所有

//默认，scala 导入：
java.lang._
scala._
scala.Predef._ 
//推荐看一下 Predef 的源代码包括：
//Predef 中定义的方法和属性
//常用方法和类
//打印方法 println 等
//一些调试和错误方法
//一个特殊的方法表示方法未实现  
def ??? : Nothing = throw new NotImplementedError
//Predef 还有大量的隐式转换和隐式参数

```
## == and eq

scala 里面`==`等价于 java 的`equals`方法即内容比较，并且可以正确处理`null`(还记得 java 规范里面烦人的 `"A".equals(m)`规范么？). 而地址 (引用) 比较使用`eq` 方法，这个方法其实很少用到，应用代码一般无需比较 2 个变量的地址。

## case class
case class 类似 data class，即 java 的 pojo bean，但是提供了更多的方法。

```scala
// 5 个特性
// 1.添加 companion object,apply 方法，unapply 方法
// 2.toString, hashCode and equals and copy methods
case class Student(name: String, marks: Int)

val s1 = Student("Rams", 550)
val s2 = s1.copy()
val s3 = s1.copy(marks = 590)
s2 == s1 //true
s3 == s1 //false

// 3. 构造函数参数自动成为成员变量，即自动给构造参数添加 val 前缀
// 4. 可以用于模式匹配
// 5. 默认的，case class 和 case object 是可序列化的 (实现 Serializable),也即是可以网络传输的

```

## for...yield
>Scala’s “for comprehensions” are syntactic sugar for composition of multiple operations with `foreach, map, flatMap, filter or withFilter`
>scala 的 for 推导其实就是组合多个`foreach, map, flatMap, filter or withFilter`的语法糖。
以下代码结果 r1,r2 完全一致：

```scala
val c1 = List(1, 2, 3)
val c2 = List("a", "b", "c")
val c3 = List("!", "@", "#")

val r1 = for (x <- c1; y <- c2; z <- c3) yield {
  x + y + z
}

//<==>
val r2 = c1.flatMap(x => c2.flatMap(y => c3.map(z => {
  x + y + z
})))

assert(r1 == r2)//true

```


## companion object 
Scala 中，除了方法，一切都是对象！函数也是对象，根据参数的个数，函数的类型为 FunctionN.N 为函数参数个数。
伴生对象用于定义一些静态方法 (工厂方法),其中 apply 和 unapply 方法常用。apply 方法用于代替 new 的工厂方法。
同时，companion objects can access private fields and methods of their companion trait/class. 

```scala
class Person(name: String, age: Int) {
  private var skill: String = "no skill"
  def introduce() = println(s"my name is $name, I am $age years old")
}

// companion object name should be identical to the class name. 
object Person {
  def apply(name: String, age: Int): Person = {
    new Person(name, age)
  }
  //apply method override
  def apply(name: String, age: Int, skill: String): Person = {
    val p = new Person(name, age)
    p.skill = skill
    p
  }
}

val dahu = Person("dahu", 30)
dahu.introduce
```

伴生对象在模式匹配和抽取器的应用
```scala
//关于抽取器和 unapply 方法的进一步示例：
trait User

class FreeUser(
                val name: String,
                val score: Int,
                val upgradeProbability: Double)
  extends User

class PremiumUser(
                   val name: String,
                   val score: Int)
  extends User

object FreeUser {
  def unapply(user: FreeUser): Option[(String, Int, Double)] =
    Some((user.name, user.score, user.upgradeProbability))
}

object PremiumUser {
  def unapply(user: PremiumUser): Option[(String, Int)] =
    Some((user.name, user.score))
}

val freeUsr = new FreeUser("john", 70, 0.5)
freeUsr match {
  case FreeUser(name, _, p) => if (p > 0.75) println(s"what can I do for you,$name")
  else println(s"hello,$name")
  case _ => println("who are you")
}

//bool 抽取器
object premiumCandidate {
  def unapply(user: FreeUser): Boolean = user.upgradeProbability > 0.4
}

// bool 抽取器的用法
freeUsr match {
  case freeUser@premiumCandidate() => println(s"恭喜成为黄金会员候选人")
  case _ => println("欢迎回来")
}
//来源：[Scala 初学者指南](http://danielwestheide.com/scala/neophytes.html)

```

## method and function(def val)

先看函数定义
> A function can be invoked with a list of arguments to produce a result.
> A function has a parameter list, a body, and a result type. Functions that are
> members of a class, trait, or singleton object are called methods.
> Functions defined inside other functions are called local functions. Functions
> with the result type of Unit are called procedures. Anonymous functions in
> source code are called function literals. At run time, function literals are
> instantiated into objects called function values.
> 
> quote from：<Programming in Scala Second Edition>
> Martin Odersky - Lex Spoon - Bill Venners

函数由一个参数列表，一个函数体，一个结果类型构成。函数如果作为 class,trait 或者 object（注意，这里的 object 是 scala 特有的单例对象，不是 Java 中的 instance）的成员，那么这个函数叫方法。函数和方法的区别就是函数时 FunctionN 的一个实例，编译后是一个单独的 class 文件，而方法是依附对象的，调用方法的格式是 obj.method(param),而调用函数的格式本质是将调用函数对象的 apply 方法。
函数定义在别的函数内部叫局部函数。函数返回值是 Unit 称为过程（procedures）.
匿名函数是通过函数字面量（ ()=>{函数体} ）定义的函数。在运行时，函数字面量被实例化对象，叫函数值。
函数和方法的区别，大部分情况下不用在意区别：
函数是有类型的： (T1, ..., Tn) => U，是 trait FunctionN 的一个实例对象，函数有一个 apply 方法，用来实际执行 function 的函数体。函数还有 toString, andThen ,conpose 等方法。

```scala
val fn: Int => String = i => i+"123" //声明一个函数
fn(3) //实际背后是 fn.apply(3);
```
scala 中除了 method，一切都是 instance
method 只能用 def 声明，function 可以是 val 和 def 声明
method 可以有类型参数[] ,function 不能有，函数在声明时就需要知道具体类型。
```scala
def fn(p: List[String]): Map[T] = {...} //is function
def m[T](t: List[T]): Map[T] = {...} //is method，可以有泛型参数。

将method转换成function有两种方法：
val f1 = m1 _   //下划线表示参数列表 eta-expansion
val f2: (Int) => Int = m1  //m1 的入参和返回值要和 f2 的一样

//scala 可以自动将 method 转换为 function，如果一个方法需要一个函数作为参数，
//那么可以直接将 m1 传递给他，不需要 下划线。
//每一次将方法转换成 function 都是得到一个新的 function object.
//function 既然是一个 instance，那么编译成 class 文件会有一个 class 文件。

```

## _ in scala

```scala
/**
  * class Reference[T] {
  * private var contents: T = _  
  * //使用类型默认值初始化变量，如果 T 是 Int，则 contents 是 0,T 是 boolean，则是 false；Unit 则是 ()
  * }
  * 
  *
  * List(1, 2, 3) foreach (print _ ) //output 123，表示实参
  *
  * //在匿名函数中作为参数占位符：
  * List(1, 2, 3) map ( _ + 2 )
  * // _ + 2 是一个匿名函数
  *
  * //模式匹配中的最后一行作为通配符
  * case _ => "this is match anything other than before cases "
  *
  * expr match {
  *
  * case List(1,_,_) => " a list with three element and the first element is 1"
  * case List(_*)  => " a list with zero or more elements "
  * case Map[_,_] => " matches a map with any key type and any value type "
  * case _ =>
  * }
  *
  *
  * //import 中作为通配符和隐藏符
  * import java.util.{ ArrayList => _, _} 
  * //第一个下划线表示隐藏 ArrayList，第二个表示通配符，导入所有
  *
  * //将方法变为 value
  * method _     // Eta expansion of method into method value
  *
  * //tuple 的访问
  * tpl._2 //返回 tpl 第二个元素，注意，tuple 是从 1 开始的
  *
  *
  * //还有很多高级的概念，目前还不理解，so 上给出的答案
  * def f[M[_]]       // Higher kinded type parameter
  * def f(m: M[_])    // Existential type
  * _ + _             // Anonymous function placeholder parameter
  * m _               // Eta expansion of method into method value
  * m(_)              // Partial function application
  * _ => 5            // Discarded parameter
  * case _ =>         // Wild card pattern -- matches anything
  * val (a, _) = (1, 2) // same thing
  * for (_ <- 1 to 10)  // same thing
  * f(xs: _*)         // Sequence xs is passed as multiple parameters to f(ys: T*)
  * case Seq(xs @ _*) // Identifier xs is bound to the whole matched sequence
  * var i: Int = _    // Initialization to the default value
  * def abc_<>!       // An underscore must separate alphanumerics from symbols on identifiers
  * t._2
  */

```

## => in scala
 
### 函数字面量分隔参数和函数体
在函数字面量中 `=>`分隔参数和函数体。也可以表示一个函数类型。
```scala
(x: Int) => x * 2表示一个匿名函数,接收一个整数,返回参数乘以2的结果.

scala> val f: Function1[Int,String] = argInt => "my int: "+argInt.toString
f: Int => String = <function1>

// Int => String 等价 Function1[Int,String]
scala> val f2: Int => String = myInt => "my int v2: "+myInt.toString
f2: Int => String = <function1>
//注意，匿名函数没有参数也要括号 ()=>{}；
//() => Unit 表示没有返回值的函数
```
	
### call-by-name parameter
在函数的参数声明中使用`=>`(e.g. `def f(arg: => T`)) 表示这个参数是"by-name parameter",表示这个参数只有在函数体中包含这个参数的语句被执行才会被 evaluate.
这个特点叫 call-by-name,arg 可以是一个代码块，甚至函数，在传递给 f 时不会 evaluate，只有 f 函数体内部调用 arg 时，arg 才会被执行。
```scala
scala> def now()={println("nano time:");System.nanoTime}

scala> def callByName(p: => Long):Long = {println("call-by-name:"+p);p;}
	callByName: (p: => Long)Long
	
scala> def justCall(p : Long) :Long = {println("just-call:"+p);p;}
	justCall: (p: Long)Long
	
scala> callByName(now())
	nano time:
	call-by-name:5664511571389
	nano time:
	res2: Long = 5664511727048
//now() 在 callByName 的函数体的每个出现的地方都执行了

scala> justCall(now())
	nano time:
	just-call:5667489483159
	res3: Long = 5667489483159
//now() 只在传递参数的时候被执行了。
```

### 模式匹配中分隔 case 模式和返回值
在 case 语句中，=> 分隔模式和返回表达式。
```scala
var a = 1  
a match{  
    case 1 => println("One")  
    case 2 => println("Two")  
    case _ => println("No")  
}
```

## () {} in method call
```scala
// 规则 1:{}表示 code block，你可以在里面放几乎任何语句，block 的返回值是由最后一句决定
// 规则 2:block 内容如果只有一句可以省略{},但是 case clause 除外:{case ...}
// 规则 3: 单参数方法如果实参是 code block，那么可以省略 ()
{
  import util.Try
  println{"hello"}
  5
}
val tupleList = List[(String, String)]()

//规则 2
tupleList takeWhile( { case(t1,t2) => t1==t2 } )

// 规则 2
List(1, 2, 3).reduceLeft(_+_)

// 一种特殊情况，提示：隐式转换
val r = List(1, 2, 3).foldLeft(0) {_+_}
//val l = r{"hello"}

//不要调用这个方法
def loopf(x: Int): Int = loopf {x}

//使用{}的特殊情况:for 推导可以和 () 互换，一般建议是除了 yield 的其他情况都用 ()
for{tpl <-tupleList} yield tpl._2

//不建议
for{tpl <-tupleList} {
  println(tpl)
}
//推荐
for(tpl <-tupleList) {
  println(tpl)
}

//补充，方法定义时如果没有返回值可以省略=,称为 procedure,scala 2.13 已经废弃，不要这么写
//don't
def p(in:String ){
  println(s"hello $in")
}
```

## implicit

implicit 分为隐式参数和隐式转换方法。
### 隐式参数
```scala
//1.隐式参数
class Prefixer(val prefix: String)

def addPrefix(s: String)(implicit p: Prefixer) = p.prefix + s
// addPrefix 需要提供一个隐式实际参数，否则报错。当然可以在调用时显式传递一个参数
implicit val myImplicitPrefixer = new Prefixer("***")
addPrefix("abc") 
// returns "***abc"
```
### 隐式转换
```scala
//1. 定义一个含有目标方法的 class
class BlingString(s:String) {
  def bling = "*"+s+"*"
}

//2. 定义隐式转换方法
implicit def str2BlingString(s:String) = new BlingString(s)

//3. 使用目标方法
val s = "hello"
s.bling // *hello*

//在 scala.Predef 中定义了大量的隐式转换，例如 RichInt,StringOps 这些，提供类似 mkString 这些方法
//太阳底下无新事，scala 常用对象的灵活丰富的语法都是通过隐式转换添加的。
```
### implicit class
可以看到上面的第 1,2 步非常的繁琐，于是[SIP-13](http://link.zhihu.com/?target=https%3A//docs.scala-lang.org/sips/implicit-classes.html)提出一个`implicit class`,将上面的 2 步合并：
```scala
implicit class BlingString(s:String) {
  def bling = "*"+s+"*"
}
//implicit def str2BlingString(s:String) = new BlingString(s)

val hi = "hello"
hi.bling // *hello*
```
注意，这个只是一个语法糖。去糖后就是上面的那个形式。implicit class 有 3 个约束和一个注解问题：

1.  必须要有主一个构造函数且只能一个构造参数（implicit 参数除外）.构造参数就是源类型。这个构造函数即等价上面第 2 步的隐式转换方法：

    ```scala
    implicit class RichDate(date: java.util.Date) // OK!
    implicit class Indexer[T](collecton: Seq[T], index: Int) // BAD!
    implicit class Indexer[T](collecton: Seq[T])(implicit index: Index) // OK!

    ```

2. 只能定义在其他 trait/class/object 中：

    ```scala
    object Helpers {
    implicit class RichInt(x: Int) // OK!
    }
    implicit class RichDouble(x: Double) // BAD!

    ```

3. 在当前 scope 内，不允许有和 implicit class 同名的方法，对象，变量。因为 case class 会自动生成同名 object 对象，所以 implicit class 不能是 case class.

    ```scala
    object Bar
    implicit class Bar(x: Int) // BAD!

    val x = 5
    implicit class x(y: Int) // BAD!

    //cuz case class has companion object by default 
    implicit case class Baz(x: Int) // BAD! conflict with the companion object

    ```

4. 还有就是 implicit class 的注解在去语法糖后会自动添加到类和方法，除非在注解中指明范围：

    ```scala
    @bar
    implicit class Foo(n: Int)

    //desugar
    @bar implicit def Foo(n: Int): Foo = new Foo(n)
    @bar class Foo(n:Int)

    //除非在注解中指明：genClass / method
    @(bar @genClass) implicit class Foo(n: Int)

    //desugar 得到
    @bar class Foo(n: Int)
    implicit def Foo(n: Int): Foo = new Foo(n)
    ```

### value class
scala 还有一个概念：[value class](http://link.zhihu.com/?target=https%3A//docs.scala-lang.org/overviews/core/value-classes.html)

```scala
class Wrapper(val underlying: Int) extends AnyVal
//1. 一个 public val 参数表示 runtime 类型，这里是 Int. 编译时是 Wrapper 类型，所以 value class 目的是降低分配开销。
//2. value class 需要 extends AnyVal
//3. value class 只能有 defs, 不能有 vals, vars, or nested traits, classes or objects,
//   因为 def 是通过静态方法实现的，而 val,var 这些则必须创建相应类型了。
//4. value class 只能扩展通用 trait（universal traits）,
//   universal traits 是 A universal trait is a trait that extends Any, only has defs as members, and does no initialization.
//
```

### extension method
当 implicit class 类型参数是 AnyVal 子类时，value class 和上面的 implicit class 形式相近，所以可以通过 value class 降低 implicit class 的分配开销。例如 RichtInt

```scala
implicit class RichInt(val self: Int) extends AnyVal {
  def toHexString: String = java.lang.Integer.toHexString(self)
}
```

因为 RichInt 是 value class，在运行时（runtime）不会有 RichInt 这个类，而是 Int，而`3.toHexString`实际是通过静态方法实现的： `RichInt$.MODULE$.extension$toHexString(3)`,这么做好处是减少对象分配开销 (avoid the overhead of allocation).如果 implicit class 的类型参数不是 AnyVal 子类，那么在 runtime 时会有相应类型对象被创建，用户察觉不到区别。
value class 还有其他作用和局限性，可以参考上面链接。如果发现错误，请指出，先谢过。

 [Implicit Design Patterns in Scala​www.lihaoyi.com](http://www.lihaoyi.com/post/ImplicitDesignPatternsinScala.html)
 [The Neophyte's Guide to Scala​](https://danielwestheide.com/scala/neophytes.html)

### 集合类的 implicit 转换

```scala
//scala 集合和 java 集合的转换是 scala 编程最常用的，毕竟 java 有大量第三方库。
//scala 提供了两种方法，第一种方法就是隐式转换 collection.JavaConversions(scala 2.8)
//很快意识到隐式转换对于使用者的代码阅读比较复杂，在 2.8.1 提供了显示转换 collection.JavaConverters,
//先看 JavaConversions 隐式转换：
object JavaConversions extends WrapAsScala with WrapAsJava
//在 WrapAsJava
  implicit def mapAsJavaMap[A, B](m: Map[A, B]): ju.Map[A, B] = m match {
    case null                 => null
    case JMapWrapper(wrapped) => wrapped.asInstanceOf[ju.Map[A, B]]
    case _                    => new MapWrapper(m)
  }

//然后看下 collection.JavaConverters._，稍微复杂一些，但是换汤不换药，底层还是隐式转换，
object JavaConverters extends DecorateAsJava with DecorateAsScala
//在 DecorateAsJava 中有很多隐式转换方法，这些方法将 scala 集合转换为 AsJava 对象
//(注意下面的 ju，是 java.util 缩写，详情见 [征服 scala_1](https://zhuanlan.zhihu.com/p/22670426))
implicit def seqAsJavaListConverter[A](b : Seq[A]): AsJava[ju.List[A]] = new AsJava(seqAsJavaList(b))
// 而 AsJava 中定义了 asJava 方法，这样我们就可以在 scala 集合上面调用 asJava
class AsJava[A](op: => A) {
    /** Converts a Scala collection to the corresponding Java collection */
    def asJava: A = op
}
//并且 asJava 方法的实现是作为构造参数传入 AsJava 的
//上面的 seqAsJavaList 就是将 scala.Seq 转换为 ju.List 的具体实现
def seqAsJavaList[A](s: Seq[A]): ju.List[A] = s match {
  case null                   => null
  case JListWrapper(wrapped)  => wrapped.asInstanceOf[ju.List[A]]
  case _                      => new SeqWrapper(s)
}

//综上，JavaConverters 用的还是隐式转换，只不过增加了一个中间类 AsJava/AsScala.
```

### 隐式转换的 scope

```scala
//无论是隐式参数还是隐式转换，编译器都要知道去哪里查找这些 implicit 参数或者方法，
//例如 import collection.JavaConverters._
//由于 scala import 可以出现在任何地方，这为控制 implicit 的 scope 提供了灵活性
//这一块我不是完全清楚，只提供一个自己的理解
// 1.首先是当前 scope 的 Implicits 定义，例如，当前方法内，class 内
// 2.显式导入 import collection.JavaConversions.asScalaIterator
// 3.通配符导入 import collection.JavaConverters._
// 4.类型的伴生对象内 (这个常用)
// 5.参数类型的隐式 scope (2.9.1 添加):class 构造参数的隐式转换搜索返回会被应用到
class A(val n: Int) {
  def +(other: A) = new A(n + other.n)
}
object A {
  implicit def fromInt(n: Int) = new A(n)
}

new A(1) + 2 // new A(1) + A.fromInt(2)
//6.类型参数的隐式转换，下面的 sorted 方法期望有一个 Ordering[A],
//在伴生对象中提供了一个 A -> Ordering[A] ,
class A(val n: Int)
object A {
    implicit val ord = new Ordering[A] {
        def compare(x: A, y: A) = implicitly[Ordering[Int]].compare(x.n, y.n)
    }
}
List(new A(5), new A(2)).sorted
// 注意 implicitly[Ordering[Int]] 表示在当前 scope 内搜索一个隐式参数值
def implicitly[T](implicit e: T): T = e
```

## string
```scala

// The s String Interpolator:
val name = "James"
println(s"Hello, $name")  // Hello, James

// The f Interpolator
val height = 1.9d
val name = "James"
println(f"$name%s is $height%2.2f meters tall")  // James is 1.90 meters tall

// The raw Interpolator
// The raw interpolator is similar to the s interpolator except that 
// it performs no escaping of literals within the string. 
// Here’s an example processed string
// 即不翻译转义字符
scala>raw"a\nb"
res1: String = a\nb

// """ triple quotes string
// triple quotes """ to escape characters
val donutJson4: String =
    """
      |{
      |"donut_name":"Glazed Donut",
      |"taste_level":"Very Tasty",
      |"price":2.50
      |}
      """
   .stripMargin
// |会被忽略
// """还有个很好的用处，正则表达式：
// 在 java 中表示一个或多个空格，"\\s+"
// 在 scala 中只要 """\s+""",对于复杂正则表达式非常有用。
```

## links

[https://www.btbytes.com/scala.html](https://www.btbytes.com/scala.html)

[https://booksites.artima.com/programming_in_scala_2ed/examples/index.html](https://booksites.artima.com/programming_in_scala_2ed/examples/index.html)

[http://blog.higher-order.com/assets/fpiscompanion.pdf](http://blog.higher-order.com/assets/fpiscompanion.pdf)

[https://courses.cs.washington.edu/courses/cse341/09au/notes/scala.html](https://courses.cs.washington.edu/courses/cse341/09au/notes/scala.html)

[https://github.com/dnvriend/my-scala-notes](https://github.com/dnvriend/my-scala-notes)

[https://gist.github.com/jamesyang124/d65b067327452792287a](https://gist.github.com/jamesyang124/d65b067327452792287a)

