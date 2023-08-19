+++
title = "面试题-类加载过程和子类重写父类方法的调用"
date = 2016-01-01
categories = [ "编程",]
tags = [ "java", "code",]
toc = "true"
+++


最近非常火的一道携程面试题Java
```java
public class Base {
    private String baseName = "base";
    public Base() {
        callName();
    }

    public void callName() {
        System.out.println(baseName);
    }

    static class Sub extends Base {
        private String baseName = "sub";

        public void callName() {
            System.out.println(baseName);
        }
    }

    public static void main(String[] args) {
        Base b = new Sub(); // 输出？
    }
}
```

<!--more-->

我的理解：
先理解两个方法:  
class 的(clinit)方法和(init)方法不同：这两个方法一个是虚拟机在装载一个类初始化的时候调用的（`<clinit>`）.另一个是在类实例化时调用的（`<init>`）.

在加载类时需要类的初始化,JVM对应的字节码方法是`<clinit>`,这个方法会初始化static变量和执行static{}代码块,按源码定义的顺序执行.**注意**：如果static{}代码块中引用了static 变量,那么一定要使用之前定义static变量.ide会提示的.

这时,class的其他成员变量和方法都没有被执行.变量的内存都已经分配,值为null或者0（基本类型）,false(布尔类型).
当创建一个类的实例时,此时会调用`<init>`方法,这个方法会初始化非static变量和执行{}代码块.注意,这两个也是按源码顺序执行的.所以代码块如果要使用非static变量,一定要先定义.同样ide一般会提示的.但是要明白这个顺序.

以上说的执行顺序通过eclipse调试可以确定是正确的.

所以组合起来 创建一个类的实例对象需要下面的顺序：

``` s
父类P static代码块和static变量初始化 
-> 子类S static代码块和static变量初始化  
-> 父类P 非static代码块和非static变量初始化 
-> 父类P构造函数 
-> 子类S非static代码块和非static变量初始化 
-> 子类S构造函数
```

回到面试题：我们看看创建一个实例对象的调用栈：
![创建实例时调用栈-图片](https://cdn.staticaly.com/gh/zhimoe/zhimoe.pic@main/pic/base-sub.7jmh61bdbbo0.webp)

可以看到依次进入16, 8, 21行代码: 

16行：`    static class Sub extends Base`

8行：`     callName();//Base()构造函数中`

21行：`    System.out.println (baseName) ;//Sub的callName()`

根据前面的分析,这个类没有static代码块和static变量,也没有代码块.所以第一个执行的是父类非静态成员的base="base";接着执行构造函数Base();这里到了魔法的一步,调用的callName()是子类（21行）的方法.这个行为就是**动态单分派**.详细资料看最后.由于子类的非static变量初始化没有完成,所有子类中的base变量是null.输出也是null.

！！！所以,不要再构造函数中调用可能会被子类覆盖的方法.

有的面试题会出现陷阱:在调用callName()方法改为this.callName(). 其实都是一样的.在调用Base构造函数时没有Base的实例对象,调用者其实还是Base$Sub这个类.


还有一个进阶版：

```java
public class Basic {
	public void add(int i) {
		System.out.println("Basic add");
	}

	public Basic() {
		add('a');
	}

	public static void main(String[] args) {
		Basic a = new A();
		B b = new B();
	}

}

class A extends Basic {
	public void add(int i) {
		System.out.println("A add");
	}
}

class B extends Basic {
	public void add(char i) {
		System.out.println("B add");
	}

}
```
不仅考察单分派,还有重载的静态多分派. 进阶版问题的解释需以下知识点-java的静态分派和动态单分派.

[CSDN-类加载机制-深入java虚拟机 读书笔记](http://blog.csdn.net/ns_code/article/details/17881581)
[方法分派](http://rednaxelafx.iteye.com/blog/260206)

重载是静态多分派,编译时期确定.  
覆盖是动态单分派,运行时通过实际类型绑定.
静态多分派: 所有依赖静态类型来定位方法执行版本的分派过程就叫做静态分派,静态分派最典型的应用就是方法重载.
动态单分派: 根据运行期实际类型确定方法执行版本的分派过程叫做动态分派,动态分派最典型的应用就是方法重写.
同时理解:动态单分派就是多态,java的面向接口编程的根基就是多态.
