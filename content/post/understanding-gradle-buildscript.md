---
title: '理解Gradle build脚本结构与语法'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - code
  - groovy
  - gradle
--- 

在看这个之前，希望你有用ant或者maven的使用经验，还有，对groovy的语法有一个简单的了解，不懂也没关系，下面会介绍。
理解gradle文件的前提是理解一个重要的groovy概念:closure

<!--more-->

#### closure
一个closure是一个定义在groovy文件中的{}代码块，这个代码块类似js中的匿名函数，它可以被赋值给变量，可以被调用，可以接收参数，还可以作为参数传递给别的函数。

closure中最重要的两个概念是委托对象和作为参数传递的语法格式（理解gradle文件很重要）。
#### groovy方法调用括号的省略

groovy提供非常优雅的方法调用格式，总结起来是:

```groovy
//可以省略参数括号，并且链式调用
// equivalent to: turn(left).then(right)
turn left then right

//groovy数字可以直接转换成字符串
// equivalent to: take(2.pills).of(chloroquinine).after(6.hours)
take 2.pills of chloroquinine after 6.hours

//两个参数用逗号隔开
// equivalent to: paint(wall).with(red, green).and(yellow)
paint wall with red, green and yellow

//命名参数用冒号
// with named parameters too
// equivalent to: check(that: margarita).tastes(good)
check that: margarita tastes good

//闭包作为参数也可以省略括号
// with closures as parameters
// equivalent to: given({}).when({}).then({})
given { } when { } then { }

//没有参数的方法必须有括号
// equivalent to: select(all).unique().from(names)
select all unique() from names

//如果调用链元素为奇数，那么最后一个元素是前面方法链返回对象的属性
//cookies 是take(3)返回值的一个属性
// equivalent to: take(3).cookies
// and also this: take(3).getCookies()
take 3 cookies

```

上面调用的格式是dsl的基础。也是看懂gradle文件格式的基础。

让我们再深入一点，上面讲的是调用格式，那么怎么创建这种可以链式调用的方法呢？
- groovy和scala的方法返回值不需要return，最后一行就是返回值。
- closure是一个匿名函数，格式`{ [closureParameters -> ] statements }`，默认自带一个名为it的参数，所以只接受一个参数时可以省略->。
- closure可以访问scope（作用域）内任何变量。并且这个scope是可以通过委托来改变的。
- groovy中Map对象的value如果是closure，那么可以接着调用:`mapp.keyy({closure}) `
有了上面的基础，我们看一个简单的例子:


```groovy
//将closure赋值给一个变量，这个closure接收一个参数，参数名是默认的，it
show = { println it }
square_root = { Math.sqrt(it) }


//为了容易理解，我将参数的type都添加上了，
//please方法需要一个closure，接着返回一个map，map的key是the，value是一个closure，
//这个closure接收一个closure，并返回一个map，这个map的of的value又是一个closure(不要晕了)
//最后一个closure接收一个参数n。
def please(Closure action) {
     [the: { Closure what ->
        [of: { n -> action(what(n)) }]
    }]
}
//调用:
// 等价: please(show).the(square_root).of(100)
please show the square_root of 100
// ==> 10.0

```

总结一下就是，将你需要的操作封装成一个closure，给一个直观的命名，保证整个DSL调用语句有语义，定义返回一个map的函数作为入口，map的key是方法名，value是closure，这样可以在key后面传递一个closure接着调用这个value。


### 委托对象

gradle脚本是一个配置脚本，类似maven中pom.xml文件，不过gradle脚本更为强大，因为.gradle文件就是groovy文件，所以还可以在脚本里面直接定义groovy对象让脚本使用。
委托对象就是一个groovy对象，用来执行gradle构建脚本中的closure。


```groovy
 as a build script executes, it configures an object of type Project. 
 This object is called the delegate object of the script. 
 The following table shows the delegate for each type of Gradle script.

三种不同的gradle脚本对应的委托对象
Build script（build.gradle） ->Project
Init script	->Gradle
Settings script(setting.gradle)	->Settings

```
构建中的每一个project，Gradle都会创建一个Project对象，并将这个对象与构建脚本相关联。  

Project对象与build.gradle是一对一的关系。

Gradle的脚本是配置脚本，当脚本执行时，它是在配置某一个特殊类型的对象。比如一个构建脚本的执行，它就是在配置一个Project类型的对象。这个对象叫做脚本的代理对象。

委托有个重要的概念就是scope，指closure的变量引用范围:有时变量不在当前scope中，但是可以通过委托，改变closure的委托对象，这样就拥有了委托者的scope，从而可以在closure中使用委托者的变量。


关于groovy closure 的委托有三个重要属性


```
• this: refers to the instance of the class that the closure was defined in.
• owner: is the same as this, unless the closure was defined inside another closure in which case the owner refers to the outer closure.
• delegate: is the same as owner. But, it is the only one that can be programmatically changed, and it is the one that makes Groovy closures really powerful.

the closure itself will be checked first, followed by the closure's this scope, than the closure's owner, then its delegate. However, Groovy is so flexible this strategy can be changed. Every closure has a property called resolvedStrategy. This can be set to:
	• Closure.OWNER_FIRST
	• Closure.DELEGATE_FIRST
	• Closure.OWNER_ONLY
	• Closure.DELEGATE_ONLY

来自 <https://dzone.com/articles/groovy-closures-owner-delegate> 

```

gradle是dsl解析工具，是对groovy语法的扩展，build.gradle可以理解为就是一个.groovy文件，gradle会解析这个文件，发现里面的closure，并将这些closure委托给一个对象去执行。
gradle将groovy的委托机制发挥到极致，要理解gradle内部，就要理解closure的委托！！

### closure作为参数传递

将closure作为参数传递的方法有多种:

```groovy
//method accepts 1 parameter - closure 
myMethod(myClosure)

//if method accepts only 1 parameter - parentheses can be omitted 
myMethod myClosure

//I can create in-line closure 
myMethod {println 'Hello World'}

//method accepts 2 parameters 
myMethod(arg1, myClosure)

//or the same as '4', but closure is in-line 
myMethod(arg1, { println 'Hello World' })

//if last parameter is closure - it can be moved out of parentheses 
myMethod(arg1) { println 'Hello World' }

```
注意第三种和最后一种调用方式，是不是和gradle文件中很眼熟？只不过在gradle脚本中出现的closure更加复杂，因为有closure嵌套！！！但是万变不离其宗。下面我们会介绍嵌套不过是**委托链**的表现。

看一个脚本代码:


```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

```
buildscript是一个方法，接收一个closure。至于这个方法在哪，可以定义在任何地方，但是可以肯定的是，这个方法一定能够被Project对象调用。
因为build.gradle脚本就是委托给Project对象执行的。事实上，Project对象也不是亲自执行这个方法，而是委托给ScriptHandler执行。
这里，我们ScriptHandler对象会搜索到两个配置closure:repositories和dependencies。我们可以在ScriptHandler api中搜索到这两个方法。从api中我们又发现:

传递给dependencies的closure又被委托给了`DependencyHandler`对象....... 这就是委托链。

[ScriptHandler api](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/dsl/ScriptHandler.html#dependencies)  

[Project api](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)

**注意**:这里buildscript {...}整体称为一个 script block。 脚本块就是一个接受closure参数的方法调用。还有的方法是不接受closure的，那些称为statement（看下面解释）。
>A script block is a method call which takes a closure as a parameter

### 插件

先看看构建脚本的构成:
>A build script is made up of zero or more **statements** and **script blocks**. Statements can include method calls, property assignments, and local variable definitions. A script block is a method call which takes a closure as a parameter. The closure is treated as a configuration closure which configures some delegate object as it executes. 

就是说脚本有两种内容:script block和statement.
Project接口预先定义了几个block:

```groovy
allprojects { }	 Configures this project and each of its sub-projects.
artifacts { }	 Configures the published artifacts for this project.
buildscript { }	 Configures the build script classpath for this project.
configurations { }	 Configures the dependency configurations for this project.
dependencies { }	 Configures the dependencies for this project.
repositories { }	Configures the repositories for this project.
sourceSets { }	Configures the source sets of this project.
subprojects { }	Configures the sub-projects of this project.
publishing { }	Configures the PublishingExtension added by the publishing plugin.

```
这些closure参数基本都是委托给其他对象执行的。

可以看到，Project对象的方法是有限而且通用的。真正有用的是插件，gradle的很多功能也是通过官方写的插件提供的。
如果你看到一个顶级层的`something { ... }`block，但是在Project源码中没有找到something block的任何信息。那么这个方法就是通过插件提供的。gradle自带很多插件,像java，eclipse,groovy，android等。
看一个实际的例子:
在andoird开发中的构建脚本:


```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "com.trickyandroid.testapp"
        minSdkVersion 16
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

```
这里，出现了android{},Project对象并没有这个script block。所以，这其实是由插件提供的block。我们找到com.android.application[入口代码](https://android.googlesource.com/platform/tools/build/+/cab495f54cd31e4e93c36e6aa4b7af661aac2357/gradle/src/main/groovy/com/android/build/gradle/AppPlugin.groovy)

```groovy
extension = project.extensions.create('android', AppExtension,
                this, (ProjectInternal) project, instantiator,
                buildTypeContainer, productFlavorContainer, signingConfigContainer)
        setDefaultConfig(extension.defaultConfig, extension.sourceSetsContainer)


```
extensions是一个ExtensionContainer实例，其中create API:  
`<T> T	create(String name, Class<T> type, Object... constructionArguments)`
这里就创建了一个android属性，是一个AppExtension对象，我们在脚本中提供给android block的{}其实是配置了一个AppExtension对象。我们可以在AppExtension中找到compileSdkVersion等属性。

所以，插件扩展的Project对象，提供了很多方法，这样，可以在脚本中使用插件定义的方法（script block）了。

一个插件就是实现实现了org.gradle.api.Plugin接口的groovy类。

我们看怎么写一个插件:


```groovy
//build.gradle
apply plugin: GreetingPlugin

//这里提供closure 来配置插件提供的greeting script block
greeting {
    message = 'Hi'
    greeter = 'Gradle'
}

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {//注意我们是如果扩展Project对象的，通过extensions对象创建一个script block:greeting,而这个block关联的是一个对象
        project.extensions.create("greeting", GreetingPluginExtension)
        project.task('hello') << {  //注意我们是如何使用greeting的，没有通过extensioins
            println "${project.greeting.message} from ${project.greeting.greeter}"
        }
    }
}

class GreetingPluginExtension {
    String message
    String greeter
}
/*
    project.task('hello') << {  
        println "${project.greeting.message} from ${project.greeting.greeter}"
    }

    使用了重载操作符，等价:
    project.task('hello').leftShift({  println "${project.greeting.message} from ${project.greeting.greeter}" })

*/


```
[官方文档:如何自己写一个插件](https://docs.gradle.org/current/userguide/custom_plugins.html)

#####参考:
[gradle-tip-2](http://trickyandroid.com/gradle-tip-2-understanding-syntax/)

[Gradle深入与实战（六）Gradle的背后是什么？](http://benweizhu.github.io/blog/2015/03/31/deep-into-gradle-in-action-6/)

### DSL语法

gradle使用的基于groovy中的DSL语法，所谓的dsl，就是基于groovy发明的新的“编程语言”，gradle dsl是groovy的超集，就是你可以完全使用groovy的语法，但是你还是会看到很多不是groovy语法，这时不要困惑，这些语法不过是gradle利用groovy提供的元编程能力提供的新语法。
以新建task的语法为例，在Project API中有四个重载形式:

```groovy
Task task(String name, Closure configureClosure);
Task task(Map<String, ?> args, String name, Closure configureClosure);
Task task(Map<String, ?> args, String name) throws InvalidUserDataException;
Task task(String name) throws InvalidUserDataException;
```

但是你会看到这样的调用方式:

```groovy
task intro(dependsOn: hello) {
   doLast { println "I'm Gradle" }  
}
```

这是dsl，具体的解析方式在[TaskDefinitionScriptTransformer](https://github.com/gradle/gradle/blob/master/subprojects/core/src/main/java/org/gradle/groovy/scripts/internal/TaskDefinitionScriptTransformer.java)

具体见我在sf的提问[gradle task method syntax in build.gradle](http://stackoverflow.com/questions/38627258/gradle-task-method-syntax-in-build-gradle/38629336?noredirect=1#comment64685234_38629336)

#### more tips
[gradle-tips](https://github.com/shekhargulati/gradle-tips/blob/master/README.md)
