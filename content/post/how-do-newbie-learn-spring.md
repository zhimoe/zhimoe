+++
title = "Java 新手如何学习 SpringMVC 框架"
date = "2015-08-21T22:02:50+08:00"
categories = [ "编程",]
tags = [ "code", "spring", "java",]
toc = "true"
+++


知乎回答备份，[原答案写于 15 年](https://www.zhihu.com/question/21142149/answer/52383396)
6,404 人赞同了该回答

<!--more-->

4 年之后感觉自己当年写的真好。O(∩_∩)O 哈哈~

评论里面有人写到现在都用 spring boot，个人觉得 boot 只要搞清楚一个 autoconfig 就懂了小半了。

学习框架的同时还是需要针对性地深入学习一些 Java 基础，例如反射，CDI，JDBC，Class 类和 MySQL 以及 http（nginx 的使用）。求精不求多，新手也不要搞什么 mongodb，etcd，zk 这些，有了前面的基础，后面上手使用新东西会很快的。举个例子，很多人学习 mybatis 的使用，但是 JDBC 只会一个 `Class.forName+Statement`，显然也不知道 mybatis 的好处和底层的。

还是要多写，不要复制，单个项目去掉复制代码还有 5000 行的话，其实就能够理解到课本上的“高内聚，低耦合”是什么意思了。

### 原回答
#### 1 
想说说自己 Spring 的学习路程。课余自学 Spring 将近一年了，还是不得其道。起初是去年（14 年）暑假学习了一下 JSP，并没有深入理解，所以导致学习 Spring 时对着书本写一些 demo，感觉自己理解了，其实并不知道内部时什么原理，出了问题不停的百度，一个小问题好几天解决不了。

学习一种框架最先需要知道的是为什么需要使用这个框架，任何一个框架的发明都是为了解决编程中的一些痛点，打开任何一本 hibernate 或者其他框架的入门书，第一章都是介绍框架的理念和优势。如果需要理解这些理念和优势，那么你需要知道不使用这个框架之前是怎么处理的，才能知道框架做了一些什么事情。

针对 Spring 的学习，第一步就是了解没有 spring 和 struts 框架之前的 Java web 是如何开发的。你会知道那时候使用 JSP 和 Servlet，然后你就知道，Servlet 是一个规范，那在 Spring 里面，Servlet 去哪了？这时候你知道了 DispatchServlet。然后你会了解到 IoC 和 AOP。

> 很多新的技术只不过是引入了新的编程元素对原来技术进行了封装。


##### 2
其实 Java Web 开发，spring 不是第一步，首先需要理解的是 HTTP 协议。chrome 的 DevTools 和 curl、postman 要有基本使用。

还要知道服务器发送给浏览器的响应是没有没有 JS、CSS 和图片等外部资源的，浏览器在解析响应时才会再次请求这些资源，这里会出现一些静态资源请求不到的问题，SpringMVC 是怎么配置的？还有 chrome 并发请求数量限制，如何合并雪碧图提高网页加载速度等知识点，属于 http 知识了。

接下来，学习 Servlet 和 JSP。这个步骤不是可以跳过的，现在流行的框架 Spring MVC 和 Struts2 其实都是基于 Servlet 的，只有深入理解了 Servlet 才能理解后面的新技术。

下面几个知识点可以检测你是否理解了 Servlet：

1、什么是 ServletContext，和 tomcat 等 web 容器的关系时什么？Servlet 工作原理解析

简单的说，我们在浏览器点击链接和按钮产生的消息不是发送给 Servlet 的，而是发送给 web 容器的 (在 JSP 出现之前，web 容器也叫 Servlet 容器)。web 容器接收消息后不知道怎么处理，转交给我们编写的 Servlet 处理，那么 web 容器怎么和 Servlet 交流呢？于是就出现了 Servlet 接口，接口是定义一种规范的良好表达形式。只要我们编写的 Java 类符合 Servlet 规范，那么就能被 Web 容器识别并被容器管理。

2、什么是 Session？Session 在实际工程中的应用场景。以及@SessionAttribute 注解的局限性。

3、JSP 是面向服务器的，它并不知道浏览器是什么鬼，是我们在写 JSP 时预设客户端是浏览器，JSP 就是一个 Servlet，JSP 的常用对象和指令。

4、JSP 的中文编码乱码有几种情况？各自的解决方法？提示：JSP 文件的编码，浏览器的解析编码，GET 请求的编码，POST 的编码。

5、Servlet 是一种接口规范，其中请求和响应是 Servlet 容器通过向方法的参数赋值 HttpServletRequest 或者 HttpServletResponse 传递的。在 Struts1 里面，将 doGet() 方法里的响应移到返回值里。在 Struts2 里则：

> 在 Controller 中彻底杜绝引入 HttpServletRequest 或者 HttpServletResponse 这样的原生 Servlet 对象。
> 同时将请求参数和响应数据都从响应方法中剥离到了 Controller 中的属性变量。


这是一个很大的技术改造，也造成了 Struts2 的盛行。Spring MVC 走的是中间路线，Spring 的 2.0.8 之前的版本甚至直接使用 Servlet 的 doGet 的。Spring MVC 现在开始流行主要还是因为 Schema xml 的精简和基于注解的配置。所以这里出现了新的知识点：Schema Based XML 的相关知识和 Java5 引入的注解原理。


书籍：推荐许令波的书《深入分析 Java Web 技术内幕 (修订版)》和计文柯的《深入理解 spring 技术内幕》。特别是第二本，对 spring 的分析很是彻底。
