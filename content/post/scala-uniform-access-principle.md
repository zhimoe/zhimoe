+++
title = "scala uniform access principle"
date = "2020-01-31T18:40:10+08:00"
categories = [ "编程",]
tags = [ "code", "scala",]
toc = "true"
+++


虽然代码写的很水,但是我对各种编程语言一直比较感兴趣. 除了工作中使用的Java之外,自己也了解Python,Groovy,Scala,Kotlin,Clojure,Go,Rust.其中Python和Scala在工作中也偶尔使用. 了解不同的编程语言语法对于编程思维的影响还是蛮有意思的.
例如, 只会Java的开发者可能没有听过模式匹配(pattern match).在我学习了Scala之后,我对模式匹配的理解就是更强更优雅的switch+if. 而在我看过rust和elixir语言中关于模式匹配之后,我对模式匹配的理解就完全不一样了.

<!--more-->

这些语言中,论说对编程思维改变最大的当属Clojure莫属. Lisp语言是一种非常优雅的语言. 这种优雅的最大特点就是Lisp(Clojure)从语法上面做到了代码即数据.即Clojure的代码形式和其数据结构list的形式是一样的(这也是lisp名字由来,LISt Processor).
这个特点的好处就是Clojure赋予了list这种数据结构强大的表达能力,可以在使用极其简练的语法在list数据结构实现复杂的逻辑.

>"It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures." —Alan Perlis

尽可能的减少语法的规则,这种语法特点在Scala上面也有体现.
<!--more-->

## uniform access principle

scala 中统一访问原则将class的方法和属性访问统一,都是通过`obj.mbr`访问.
这么做的好处是代码更加统一,而且重构更加方便.

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

统一也体现在集合访问形式上,在Scala中,Map,List,Array的元素访问都是通过`coll(ki)`形式. `ki`表示key或者index.个人非常喜欢这种统一.