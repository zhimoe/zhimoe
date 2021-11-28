---
title: 'Java动态代理'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

### 好文
[Java 动态代理机制分析及扩展](http://www.ibm.com/developerworks/cn/java/j-lo-proxy1/)

更深入的一篇:
[java设计模式-动态代理模式](http://nemotan.github.io/2015/11/java%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/)


### 优势

相比 静态代理，动态代理具有更强的 灵活性，因为它不用在我们设计实现的时候就指定 某一个代理类来代理哪一个被代理对象，我们可以把这种指定延迟到程序运行时由 JVM来实现。

### 实例

动态代理类接口，接口规范方法。

```java
package angus.interview.proxy;

public interface Subject {
	public void request();

}

```
需要被代理的真实的类:


```java
package angus.interview.proxy;

public class SubjectImpl implements Subject {
	@Override
	public void request() {
		System.out.println(" subject request");
	}

}

```

先创建一个代理类。然后利用反射创建一个用真实类加载器创建的一个对象。该对象调用request方法实际上调用的是代理类的invoke方法。

```java
package angus.interview.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxy implements InvocationHandler {
	private Object target;

	public Object bind(Object target) {
		this.target = target;
		return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                                              target.getClass().getInterfaces(), 
                                              this); 
		// 要绑定接口this(这是一个缺陷，cglib弥补了这一缺陷)
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("------------------before------------------");
		Object result = method.invoke(target, args);
		System.out.println("-------------------after------------------");
		return result;
	}

}


static void main(){
    DynamicProxy proxy = new DynamicProxy();
    Subject subject= proxy.bind(SubjectImpl);
    subject.request();
}

```

和静态代理模式比较的好处

在静态代理模式时,一个真实角色必须对应一个代理角色,如果大量使用会导致类的急剧膨胀;而动态代理则不会有这个问题，我们将接口中的方法委托给invoke方法，并在invoke中实现拦截。

### 源码分析

参考:http://rejoy.iteye.com/blog/1627405 主要原来:生成了一个代理类的class文件。 Proxy.newProInstance()方法

```java
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
    throws IllegalArgumentException {

    if (h == null) {
        throw new NullPointerException();
    }  

    final Class<?>[] intfs = interfaces.clone();

    final SecurityManager sm = System.getSecurityManager();

    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    // 这里是生成class的地方  
    Class<?> cl = getProxyClass0(loader, intfs);

    // 使用我们实现的InvocationHandler作为参数调用构造方法来获得代理类的实例  
    try {
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        if (sm != null && ProxyAccessHelper.needsNewInstanceCheck(cl)) {
            return AccessController.doPrivileged(new PrivilegedAction<Object>() {
                public Object run() {
                    return newInstance(cons, ih);
                }
            });
        } else {
            return newInstance(cons, ih);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString());
    }

}  

```

其中newInstance只是调用Constructor.newInstance来构造相应的代理类实例，这里重点是看getProxyClass0这个方法的实现:

```java
private static Class<?> getProxyClass0(ClassLoader loader,
                                          Class<?>... interfaces) {
        // 代理的接口数量不能超过65535，这是class文件格式决定的
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }
        // JDK对代理进行了缓存，如果已经存在相应的代理类，则直接返回，否则才会通过ProxyClassFactory来创建代理
        return proxyClassCache.get(loader, interfaces);
    }

```
其中代理缓存是使用WeakCache实现的，如下

```java

    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());


```
具体的缓存逻辑这里暂不关心，只需要关心ProxyClassFactory是如何生成代理类的，ProxyClassFactory是Proxy的一个静态内部类，实现了WeakCache的内部接口BiFunction的apply方法:


```java
    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>> {
        // 所有代理类名字的前缀
        private static final String proxyClassNamePrefix = "$Proxy";
        // 用于生成代理类名字的计数器
        private static final AtomicLong nextUniqueNumber = new AtomicLong();
        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
            // 省略验证代理接口的代码……
            String proxyPkg = null;     // 生成的代理类的包名
            // 对于非公共接口，代理类的包名与接口的相同
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }
            // 对于公共接口的包名，默认为com.sun.proxy[源码](http://hg.openjdk.java.net/jdk6/jdk6/jdk/rev/695dd7ceb9e3)
            if (proxyPkg == null) {
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }
            // 获取计数
            long num = nextUniqueNumber.getAndIncrement();
            // 默认情况下，代理类的完全限定名为:com.sun.proxy.$Proxy0，com.sun.proxy.$Proxy1……依次递增
            String proxyName = proxyPkg + proxyClassNamePrefix + num;
            // 这里才是真正的生成代理类的字节码的地方
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces);
            try {
                // 根据二进制字节码返回相应的Class实例
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                throw new IllegalArgumentException(e.toString());
            }
        }
    }


```
ProxyGenerator是sun.misc包中的类，它没有开源，但是可以反编译来一探究竟:

```java
    public static byte[] generateProxyClass(final String var0, Class[] var1) {
        ProxyGenerator var2 = new ProxyGenerator(var0, var1);
        final byte[] var3 = var2.generateClassFile();
        // 这里根据参数配置，决定是否把生成的字节码（.class文件）保存到本地磁盘，
        //我们可以通过把相应的class文件保存到本地，再反编译来看看具体的实现，这样更直观
        if(saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction() {
                public Void run() {
                    try {
                        FileOutputStream var1 = new FileOutputStream(ProxyGenerator.dotToSlash(var0) + ".class");
                        var1.write(var3);
                        var1.close();
                        return null;
                    } catch (IOException var2) {
                        throw new InternalError("I/O exception saving generated file: " + var2);
                    }
                }
            });
        }
        return var3;
    }

```
saveGeneratedFiles这个属性的值从哪里来呢:

```java
    private static final boolean saveGeneratedFiles = ((Boolean)AccessController.doPrivileged(
    new GetBooleanAction("sun.misc.ProxyGenerator.saveGeneratedFiles"))).booleanValue();


```
GetBooleanAction实际上是调用Boolean.getBoolean(propName)来获得的，而Boolean.getBoolean(propName)调用了System.getProperty(name)，所以我们可以设置sun.misc.ProxyGenerator.saveGeneratedFiles这个系统属性为true来把生成的class保存到本地文件来查看。

反编译class文件

自己创建文件写入生成的动态代理类:


```java
package angus.interview.proxy;

import java.io.FileOutputStream;
import java.io.IOException;

import sun.misc.ProxyGenerator;

@SuppressWarnings("restriction")
public class ProxyGeneratorUtils {
	public static void writeProxyClassToHardDisk(String path) {
		// 获取代理类的字节码
		byte[] classFile = ProxyGenerator.generateProxyClass("$Proxy11", SubjectImpl.class.getInterfaces());

		FileOutputStream out = null;

		try {
			out = new FileOutputStream(path);
			out.write(classFile);
			out.flush();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				out.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

}

```

测试我们的工具类:


```java
package angus.interview.proxy;

public class TestProxy {

	public static void main(String[] args) {
		System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
		DynamicProxy proxy = new DynamicProxy();
		Subject sproxy = (Subject) proxy.bind(new SubjectImpl());
		sproxy.request();
		ProxyGeneratorUtils.writeProxyClassToHardDisk("$Proxy11.class");

	}

}

```
刷新目录，得到一个$Proxy11.class,反编译使用Java Decompiler，GUI傻瓜式，支持最新语法，编译慢，效果好:  
可以看到    
$Proxy11继承Proxy，并实现了Subject，同时我们写的那个InvocationHandler的子类DynamicProxy也被传递进去了。
重点看request方法的代码，只有一行 `  this.h.invoke(this, m3, null);`其中h的引用就是`DynamicProxy`.  
m3就是`  m3 = Class.forName("angus.interview.proxy.Subject").getMethod("request", new Class[0]);`


```java
import angus.interview.proxy.Subject;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy11 extends Proxy  implements Subject
{
  private static Method m1;
  private static Method m2;
  private static Method m3;
  private static Method m0;
  
  public $Proxy11(InvocationHandler paramInvocationHandler)
  {
    super(paramInvocationHandler);
  }
  
  public final boolean equals(Object paramObject)
  {
    try
    {
      return ((Boolean)this.h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final String toString()
  {
    try
    {
      return (String)this.h.invoke(this, m2, null);
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final void request()
  {
    try
    {
      this.h.invoke(this, m3, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  public final int hashCode()
  {
    try
    {
      return ((Integer)this.h.invoke(this, m0, null)).intValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
  
  static
  {
    try
    {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m3 = Class.forName("angus.interview.proxy.Subject").getMethod("request", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
}


```
