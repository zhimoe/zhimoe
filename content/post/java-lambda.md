---
title: "Java 8 Lambda笔记"
date: "2020-08-06T22:49:34+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - java
 - lambda
---


### 问题
Java是OOP语言,使用对象封装。由于函数不是一等公民,无法在方法中传递函数/方法。 在Java 8之前,使用匿名类表示行为：
```java
// 监听器接口
public interface ActionListener {
    void actionPerformed(ActionEvent e);
}
// 使用匿名类传递一个行为
button.addActionListener(new ActionListener(){
    public void actionPerformed(Event e){
        System.out.println("button clicked");
    }
});

```
上面的代码主要的问题在于`addActionListener`方法期望的是一个行为,为了描述这个行为（代码即数据的概念）,在Java中不得不传入一个对象. 除了代码冗余,还存在下面问题
1. 业务逻辑淹没在匿名类语法中,就像Go语言的`if err != nil`一样
2. 匿名类中的 this 和变量名容易使人产生误解
2. 类型载入和实例创建语义不够灵活
3. 无法捕获非 final 的局部变量

### lambda表达式
为了解决上面的问题,Java8推出了lambda表达式——当接口只有一个抽象方法时,称为函数式接口（也叫单抽象方法类型,SAM类型）,可以使用lambda表达式表示这个接口的实现方法。
```java
button.addActionListener(e -> System.out.println("button clicked"));
```
其中的`e`是`actionPerformed(Event e)`方法的参数,`->` 后面的是方法体。 注意这里我们并没有提供e的类型,这是由`类型推导`技术实现的——javac根据`addActionListener`方法签名和`actionPerformed`方法签名推导出参数类型只能是`Event`。
不是所有情况都可以省略类型,但是请给IDE表现机会,只有在IDE提醒你有错误时再补充上类型信息。
下面都是合法的lambda表达式：
```java
Runnable tsk = () ->  println("");
Runnable tsk = name -> { println(name);}
BinaryOperator<Long> add = (Long x, Long y) -> x + y;
BinaryOperator<Long> add = (x, y) -> {return  x + y;} //类型推断, return和{}是冗余的
// <!-- 参数括号和大括号省略规则 -->
// 1. 参数()：无参数使用(),1个参数可以省略括号,其他使用()。 
// 2. 函数体{}：单语句的可以省略{},多条语句必须有{}
```
在Java中,已经有大量的函数式接口：
- `java.lang.Runnable`
- `java.util.concurrent.Callable`
- `java.security.PrivilegedAction`
- `java.util.Comparator`
- `java.io.FileFilter`
- `java.beans.PropertyChangeListener`

1. this指向调用者,也即是button
2. lambda的类型是根据上下文来决定的, 所以相同入参和返回值情况下，目标类型可能不同，在无法判断时，需要补充目标类型信息:
```java
Callable<String> c = () -> "done";
PrivilegedAction<String> a = () -> "done";

// error
var add = (Long x, Long y) -> x + y;
// 这里add会报错：
// java: cannot infer type for local variable add
//   (lambda expression needs an explicit target-type)
// 因为满足 (Long, Long) -> Long的函数式接口很多,编译器无法知道add目标类型应该是什么。
```
3. 当涉及到泛型时,类型推导总是有点力不从心,需要添加必要的类型信息：
         
### 函数式接口与@FunctionalInterface
有了lambda和函数式接口,框架方法在形参类型上面可以更加泛化了。例如你希望你的框架方法支持一个T->R的操作,你可能会定义一个
```java
    @FunctionalInterface
    public interface Transfer<T, R> {
        R apply(T t);
    }
```
这里T,R是泛型，这是一个非常泛化的函数式接口。所以Java8在util.function包中新增了43个函数式接口,目的就是方便框架开发者能够减少新建自己的FunctionalInterface。
基础的接口只有6个:
   

   | 接口             | 函数签名             | 举例                   |
   | :--------------- | :------------------- | :--------------------- |
   | `UnaryOperator ` | `R apply(T t);     ` | `String::toLocaerCase` |
   | `BinaryOperator` | `R apply(T t, U u);` | `BigInterger::add    ` |
   | `Predicate     ` | `boolean test(T t);` | `Collection::isEmpty ` |
   | `Function      ` | `R apply(T t);     ` | `Arrays::asList      ` |
   | `Supplier      ` | `T get();          ` | `Instant::now        ` |
   | `Consumer      ` | `void accept(T t); ` | `System.out::println ` |

上面的是基础接口,此外还有：
* Consumer, Function, Predicate各自有一个2个入参的版本,共3个:BiConsumer,BiFunction,BiPredicate.
* 6个基础接口对应入参为基本类型int,long,double的接口,共18个:IntSupplier,LongFunction...
* 6个基础接口对应返回值为基本类型int,long,double的Function和BiFunction,共6个: ToIntBiFunction,ToIntFunction...
* int,long,double基本类型互转的Function共6个：DoubleToIntFunction,DoubleToLongFunction,IntToDoubleFunction,IntToLongFuncion,LongToDoubleFunction,LongToIntFunction.
* Consumer有同时接受一个Object和一个基本类型的版本,共3个: ObjDoubleConsumer{void accept(T t, int value);}
* 最后还有一个BooleanSupplier{boolean getAsBoolean();}

第一次见到BooleanSupplier可能完全不知道使用场景,毕竟有Supplier<Boolean>不就可以了么？

上面的基础接口虽然非常通用,但是如果有更好的接口名称时,应该使用更合适的那个。例如Comparator{int compare(T o1, T o2);}和ToIntBiFunction<T, U> {int applyAsInt(T t, U u);}签名完全一致,但是还是在比较的时候使用Comparator。

在构建自己的函数式接口时,务必使用注解`@FunctionalInterface`标注你的接口，这样可以给IDE lint和使用者提供更加充分信息。

### 方法引用
如果lambda表达式的方法体过长,那么需要抽取方法,Java8提供了更近一步的语法——方法引用。 方法引用表示一个lambda表达式。只需要引用的方法签名和lambda目标类型的抽象方法签名一致即可。
方法引用一共有5种类型,其中,静态方法是最常用的类型。


| 方法引用类型                     | 方法引用                 | 对应lambda表达式                                |
| :------------------------------- | :----------------------- | :---------------------------------------------- |
| 静态方法                         | `Integer::parseInt`      | `str-> Integer.parseInt(str)`                   |
| 有限制(Bound receiver)实例引用   | `Instant.now()::isAfter` | `Instant then = Instant.now(); then.isAfter(t)` |
| 无限制(Unbound receiver)实例引用 | `String::toLowerCase`    | `str -> str.toLowerCase`                     |
| 类构造器                         | `TreeMap<K,V>::new`      | `()-> new TreeMap<K,V>()`                       |
| 数组构造器                       | `int[]::new`             | ` len->new int[len]`                            |

- Bound receiver其实很好理解,方法的receiver(上面的then = Instant.now())是固定的。
- Unbound receiver的含义是方法的接收者(上面的str)是不确定的, 通过入参的形式传入。 而在方法引用的形式上面反而像静态方法引用(`String::toLowerCase`， toLowerCase不是静态方法，所以不是静态方法引用)。 更粗暴的理解就是入参是方法的引用对象,所以方法引用对象取决于入参（不确定）。
- 数组构造器的比较难以理解,可以看成如下代码：
  ```java
  IntFunction<int[]> arrayMaker = int[]::new;
  int[] array = arrayMaker.apply(len) // 创建数组 int[len]
  ```
