+++
title = "java generic"
date = 2016-05-01
categories = [ "编程",]
tags = [ "java", "code",]
toc = "true"
+++


### 泛型

```java
// 类
class Tuple<T, S> {
    private T first;
    private S second;
}
// 泛型方法也可在非泛型类里面
class ArrayAlg {
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }

}
```
<!--more-->

```java
String middle = ArrayAlg.<String>getMiddle("]ohnM, "Q.n, "Public");// right,<String>可以省略

String middle = GenericCls.getMiddle("hello",0,null);// error

//  Errr:(7, 45) java: 不兼容的类型: 推断类型不符合上限
//     推断: java.lang.Object&java.io.Serializable&java.lang.Comparable<? extends java.lang.Object&java.io.Serializable&java.lang.Comparable<?>>
//     上限: java.lang.String,java.lang.Object 
```

#### 类型限定

```java
public static <T extends Comparable> T min(T a)

// 如果多个类型,则：T extends Comparable & Serializable
// 只能有一个类,且类必须紧跟extends,但是可以有多个接口

```

### 类型擦除

```java

//Tuple<T,S>在虚拟机变为
class Tuple {
    private Object first;//当调用getFirst时,则发生强制转换
    private Object second;
}

//泛型方法同样有擦除
public static <T extends Comparable> T min(T a)
// =>
public static Comparable min(Comparable a)

```

### 约束

- 不能用基本类型实例化泛型,`Pair<double>`不允许
- 运行时参数类型检查只能检查原始类型

```java
if (a instanceof Pair<String>) // Error
if (a instanceof Pair<T>) // Error
Pair<String> p = (Pair<String>) a; //warning
```
- 不能创建参数化类型的数组

```java
Pair<String>[] table = new Pair<String>[10]; // Error
Pair<String>[] table; //声明是合法的,只是无法实例化
```
- 借助@SafeVarargs参数化类型的数组
```java
@SafeVarargs
public static <T> void addAll(Collection<T> coll, T... ts)
```
- Class类本身是泛型. 例如,String.daSS 是一个 Class<String> 的实例（事实上,它是唯一的实例.) 因此,makePair 方法能够推断出 pair 的类型
- 泛型类的静态上下文中类型变量无效
  
```java
public class Singleton<T> {
    private static T singlelnstance; // Error
    public static T getSinglelnstance{// Error
        if (singleinstance == null) {//construct new instance of T
            return singlelnstance;
        }
    }
}
```
- 不能抛出或捕获泛型类的实例
  
```java
public class Problem<T> extends Exception { /* . . . */ } // Error can't extend Throwable
```
- 泛型擦除的方法冲突

```java
public class Pair<T> {
    T first;
    T second;

    public boolean equals(T value) { //error 和Object.equals冲突
        return first.equals(value) && second, equals(value);
    }
}
```

### 泛型继承

```java
class Employee
class Manager extends Employee

//Pair<Employee> 和Pair<Manager> 没用任何继承关系

```
### 通配符 和 PECS

```java
Pair<? extends Employee〉
Pair<? super Manager
```
### 反射和泛型
