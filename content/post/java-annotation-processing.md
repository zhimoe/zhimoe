+++
title = "Java 注解和注解处理器"
date = 2016-01-01
categories = [ "编程",]
tags = [ "java", "code",]
toc = "true"
+++


### 注解处理

注解是 jdk1.5 出现的，但是自定义处理注解的功能是 1.6 才有的.Element 等关于注解源码抽象的支持类都是 1.6 出现的。
关于注解的定义就不说了，主要说说注解处理
本文根据以下资料并进行部分修改：
[JavaAnnotationProcessing](http://www.angelikalanger.com/Conferences/Slides/JavaAnnotationProcessing-JSpring-2008.pdf)

<!--more-->

### 基本知识

annotation processing integrated into javac compiler   

– since Java 6.0; known as `pluggable annotation processing`  
– compiler automatically searches for annotation processors  
– unless disabled with `-proc:none `option for javac  
– processors can be specified explicitly with `-processor ` option for javac or `-cp processor.jar`,processor.jar include /META-INF/service/javax.annotation.processing.Processor file and your processor decalared in file;
  

implement a processor class  
– must implement `Processor` interface  
– typically derived from `AbstractProcessor`  
– new package javax.annotation.processing  

同时自定义注解处理器需要指定注解选项：specify supported annotation + options  

– by means of annotations:
@SupportedAnnotationTypes
@SupportedOptions
@SupportedSourceVersion

编译器编译源码是会有很多轮 (round)：

1st round：编译器得到所有的注解 - 获取所有的注解处理器 - 进行 match 并 process，如果匹配的处理器中 process 方法的返回值是`true`,表示该注解被 claim，不再查询其他处理器。如果是`false`,接着查询匹配处理器处理，所以注解处理器在 META-INF/services/javax.annotation.processing.Processor 声明顺序是有关系的-- 所有的注解都被 claim 后，注解处理完成。

如果注解处理器产生新的 java 文件，那么新的一轮处理开始，前面被调用的那些处理器又被调用，直到没有 java 文件产生。

最后一轮又要调用一遍所有处理器，完成他们的各自工作。

最最后，编译器编译源码和注解处理器生成的源码。

还有一个很重要的类 AbstractProcessor：有一个引用 processingEnv    
提供了两个重要工具类：  
– `Filer` for creation of new source, class, or auxiliary files  
– `Messager ` to report errors, warnings, and other notices  

此外，一个产生 java 文件的重要方法：

```text
FileObject sourceFile = processingEnv.getFiler().createSourceFile(beanClassName);

process() method takes 2 arguments:

Set<? extends TypeElement> annotations  
– the annotation types requested to be processed  
– subset of the supported annotations  

RoundEnvironment roundenv  
– environment for information about the current and prior round  
– supplies elements annotated with a given annotation or all root elements in the source  
```

一个自定义的注解处理器格式如下：

```java
@SupportedAnnotationTypes({"Property"})
@SupportedSourceVersion(SourceVersion.RELEASE_6)
public class PropertyAnnotationProcessor extends AbstractProcessor {
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
        // process the source file elements using the mirror API
    }
}

```

jdk1.6 对注解的处理支持建立在对源码的抽象，Element 是`javax.lang.model.*`中定义的，各种 Element 是对源码抽象数据结构，如：  


```java
package com.example;	    // PackageElement
public class Foo {		    // TypeElement
    private int a;          // VariableElement
    private Foo other;      // VariableElement
    public Foo () {}        // ExecuteableElement

}

```
TypeElement 不能提供父类的信息，如果需要这些信息，需要从 Element 中得到 TypeMirror.TypeMirror::element.asType() 

### 实例：
动手写注解处理器：3 个类，一个定义注解 Comparator.java，一个使用注解的类 Name.java，一个处理注解 MyProcessor.java.  
我将定义一个注解@Comparator，使用在方法上，被注释的方法能够返回一个 Comparator.  
一个注解处理器，解析所有被注释的方法，为每一个方法产生一个 Comparator 类。

！！！注意，这里的内容和连接中资料的已经不一样了，资料里给的 process 方法并不能产生比较器类。

给出注解定义前看看注解怎么使用：

```java
// ./Name.java  
// ./  表示当前命令行文件夹，后面所有的 javc 命令都以这个文件夹为准
package java.interview.annotation;

public class Name {

	private final String first;
	private final String last;

	public Name(String f, String l) {
		first = f;
		last = l;
	}

	@Comparator("NameByFirstNameComparator")
	public int compareToByFirstName(Name other) {
		if (this == other)
			return 0;
		int result;
		if ((result = this.first.compareTo(other.first)) != 0)
			return result;
		return this.last.compareTo(other.last);
	}
}

```

其中被注解注释的方法将产生一个 NameByFirstNameComparator.java 文件：

```java
// ./angus/initerview/annotation/NameByFirstNameComparator.java

public class NameByFirstNameComparator implements java.util.Comparator<Name> {
	public int compare(Name o1, Name o2) {
		return o1.compareToByFirstName(o2);
	}

	public boolean equals(Object other) {
		return this.getClass() == other.getClass();
	}
}

```


我们定义注解：

```java
// ./Comparator.java 
package angus.interview.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Comparator {
	String value();

}

```

接下来定义我们的注解处理器，有详细注解，特别注意 generate 源码中的空格和分号不要弄丢了：

```java
package angus.interview.annotation;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Set;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.ExecutableElement;
import javax.lang.model.element.TypeElement;
import javax.lang.model.type.PrimitiveType;
import javax.lang.model.type.TypeKind;
import javax.lang.model.type.TypeMirror;
import javax.tools.Diagnostic;
import javax.tools.FileObject;

@SupportedAnnotationTypes({ "angus.interview.annotation.Comparator" })
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class MyProcessor extends AbstractProcessor {

	@Override
	public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
		
		
		for( final Element element: roundEnv.getElementsAnnotatedWith( Comparator.class ) ) {
			if(element instanceof ExecutableElement){
				ExecutableElement m = (ExecutableElement) element;
				TypeElement className = (TypeElement)m.getEnclosingElement();
				Comparator a = m.getAnnotation(Comparator.class);
				if (a != null) {

					TypeMirror returnType = m.getReturnType();

					if (!(returnType instanceof PrimitiveType)
							|| ((PrimitiveType) returnType).getKind() != TypeKind.INT) {
						processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR,
								"@Comparator can only be applied to methods that return int");
						continue;
					}
					// prepare for java file generation
					// t m a mean ?
					String comparatorClassName = a.value();
					String comparetoMethodName = m.getSimpleName().toString();
					String theProcessedClassesName = className.getQualifiedName().toString();
					try {
						writeComparatorFile(theProcessedClassesName, comparatorClassName, comparetoMethodName);
					} catch (IOException e) {
						e.printStackTrace();
						
					}

				}
			}
			
		}

		return true;// claimed now,no need next processor
	}

	/*
	 * 
	 * public class NameByFirstNameComparator implements java.util.Comparator<Name> { 
     *   public int compare(Name o1, Nameo2) { return o1.compareToByFirstName(o2); }
	 * 
	 *   public boolean equals(Object other) { return this.getClass() == other.getClass(); } }
	 */

	//!!!careful with spaces and ";"!!!
	private void writeComparatorFile(String fullClassName, String comparatorClassName, String compareToMethodName)
			throws IOException {
		int i = fullClassName.lastIndexOf(".");
		String packageName = fullClassName.substring(0, i);
		FileObject sourceFile = processingEnv.getFiler().createSourceFile(packageName + "." + comparatorClassName);
		if (sourceFile == null) {
			System.out.println("create source file failed");
		}
		PrintWriter out = new PrintWriter(sourceFile.openWriter());

		if (i > 0) {
			out.println("package " + packageName + ";");
		}
		String parametrizedType = fullClassName.substring(i + 1);//!!
		out.println(
				"public class " + comparatorClassName + " implements java.util.Comparator<" + parametrizedType + "> {");
		out.println();
		out.println("public int compare( " + parametrizedType + " o1 , " + parametrizedType + " o2 ){");
		out.println("return o1." + compareToMethodName + "(o2);");
		out.println("}");
		
		out.println();
		out.println();
		out.println("public boolean equals(Object other) {");
		out.println("return this.getClass() == other.getClass();");
		out.println("}");

		out.println("}");

		out.close();
	}

}


```

### 测试处理器
两种方法，

一种是使用 -cp：

在项目的根目录中（pom.xml 同级目录）新建 META-INF 文件夹，并在里面新建 services 文件夹，再在里面新建一个文件 javax.annotation.processing.Processor，并在该文件中注册我们的处理器，第一行写入：angus.interview.annotation.MyProcessor.
然后用 eclipse 将项目 export 得到一个 jar 包，jar 必须包含 target 文件夹（处理器 class 文件）和 META-INF 文件夹（注册处理器）.这里将 jar 包命名为 process.jar. 复制 jar 包到 Name.java 目录中，并在该目录打开终端，输入：
>javac -cp process.jar Name.java
   
将会得到 Name.class 文件和一个 angus 文件夹，最里面是 NameByFirstNameComparator.java 和 NameByFirstNameComparator.class.
打开 NameByFirstNameComparator.java，发现内容和上面给出的一模一样。

第二种方法是使用-processor 参数，但是还没搞懂 MyProcessor.class 应该放在哪里。暂时先到这。
