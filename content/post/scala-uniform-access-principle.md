+++
title = "Scala uniform access principle"
date = "2020-01-31T18:40:10+08:00"
categories = [ "编程",]
tags = [ "code", "scala",]
toc = "true"
+++


虽然代码写的很水，但是我对各种编程语言一直比较感兴趣。除了工作中使用的 Java 之外，自己也了解 Python,Groovy,Scala,Kotlin,Clojure,Go,Rust.其中 Python 和 Scala 在工作中也偶尔使用。了解不同的编程语言语法对于编程思维的影响还是蛮有意思的。
例如，只会 Java 的开发者可能没有听过模式匹配 (pattern match).在我学习了 Scala 之后，我对模式匹配的理解就是更强更优雅的 switch+if. 而在我看过 rust 和 elixir 语言中关于模式匹配之后，我对模式匹配的理解就完全不一样了。

<!--more-->

这些语言中，论说对编程思维改变最大的当属 Clojure 莫属。Lisp 语言是一种非常优雅的语言。这种优雅的最大特点就是 Lisp(Clojure) 从语法上面做到了代码即数据。即 Clojure 的代码形式和其数据结构 list 的形式是一样的 (这也是 lisp 名字由来，LISt Processor).
这个特点的好处就是 Clojure 赋予了 list 这种数据结构强大的表达能力，可以在使用极其简练的语法在 list 数据结构实现复杂的逻辑。

>"It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures." —Alan Perlis

尽可能的减少语法的规则，这种语法特点在 Scala 上面也有体现。
<!--more-->

## uniform access principle

scala 中统一访问原则将 class 的方法和属性访问统一，都是通过`obj.mbr`访问。
这么做的好处是代码更加统一，而且重构更加方便。

>A function that takes no parameters, which is defined without any empty parentheses. 
>Invocations of parameter less functions may not supply parentheses. 
>This supports the uniform access principle, which enables the def to be changed into a val without 
>requiring a change to client code.

```scala
class Person {
    private var privateName = ""
    
    def name = privateName
    
    def name_=(value: String) = privateName = value
}

val john = new Person
john.name = "John Doe"
println(john.name)

```

统一也体现在集合访问形式上，在 Scala 中，Map,List,Array 的元素访问都是通过`coll(ki)`形式。`ki`表示 key 或者 index.个人非常喜欢这种统一。