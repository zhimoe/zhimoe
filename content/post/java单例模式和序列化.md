---
title: '单例模式和序列化'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

[参考资料](http://www.hollischuang.com/archives/205)

### 饱汉式 

```java
public class Singleton {  
    private static Singleton instance = null  
    private Singleton (){}   
	    public static Singleton getInstance() {  
	        if(instance == null)             instance = new Singleton();         return instance;  
    }  
}  //饱汉式，使用时创建

```
### 饿汉式

```	java
	//加载时创建对象 static
public class Singleton {  
    private Singleton instance = null;  
    static {  
        instance = new Singleton();  
    }  
    private Singleton (){}  
    public static Singleton getInstance() {  
        return this.instance;  
    }  
} 

```	

### 静态内部类

```
public class Singleton { 
	private Singleton (){} 
	private static class SingletonHolder {  
            private static final Singleton INSTANCE = new Singleton();  
     }  
     
    public static final Singleton getInstance() {  
      return SingletonHolder.INSTANCE;  
    }  
}
  //这个比较好，线程安全，也达到了延迟加载效果。

```

### 枚举类

```
//这个是最好的 这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，可谓是很坚强的壁垒啊
	public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
访问这个单例 Singleton.INSTANCE 

```
### 双重校验锁
其实是不安全的，多线程开销很大，甚至死锁。原因在于指令重排序。

```
	public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}  
	

```
### 序列化
使用静态内部类举例，只要提供一个readResolve方法

```	
public class Singleton { 
		private Singleton (){} 
	    
	     private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
     }  
     
    public static final Singleton getInstance() {  
      return SingletonHolder.INSTANCE;  
    }  
	    
    private Object readResolve() throws ObjectStreamException{         
           return SingletonHolder.INSTANCE;
    }
	
}

```
无论是实现Serializable接口，或是Externalizable接口，当从I/O流中读取对象时，readResolve()方法都会被调用到。实际上就是用readResolve()中返回的对象直接替换在反序列化过程中创建的对象，而被创建的对象则会被垃圾回收掉。

	
	
	
	
	
