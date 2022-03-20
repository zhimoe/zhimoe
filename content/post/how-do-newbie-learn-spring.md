---
title: "Java新手如何学习SpringMVC框架"
date: "2015-08-21T22:02:50+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - spring
 - java
---

知乎回答备份,[原答案写于15年](https://www.zhihu.com/question/21142149/answer/52383396).
6,404 人赞同了该回答

<!--more-->

4年之后感觉自己当年写的真好.O(∩_∩)O哈哈~

评论里面有人写到现在都用spring boot,个人觉得boot 只要搞清楚一个autoconfig就懂了小半了.

学习框架的同时还是需要针对性地深入学习一些Java基础,例如反射,CDI, JDBC,Class类和MySQL 以及 http（nginx的使用）.求精不求多,新手也不要搞什么mongodb,etcd,zk这些,有了前面的基础,后面上手使用新东西会很快的. 举个例子,很多人学习mybatis的使用,但是JDBC只会一个Class.forName+Statement,显然也不知道mybatis的好处和底层的.

还是要多写,不要复制,单个项目去掉复制代码还有5000行的话,其实就能够理解到课本上的“高内聚,低耦合”是什么意思了.

--------------原回答--------------
#### 1 
想说说自己Spring的学习路程.课余自学Spring将近一年了,还是不得其道.起初是去年（14年）暑假学习了一下JSP,并没有深入理解,所以导致学习Spring时对着书本写一些demo,感觉自己理解了,其实并不知道内部时什么原理,出了问题不停的百度,一个小问题好几天解决不了.

学习一种框架最先需要知道的是为什么需要使用这个框架,任何一个框架的发明都是为了解决编程中的一些痛点,打开任何一本hibernate或者其他框架的入门书,第一章都是介绍框架的理念和优势.如果需要理解这些理念和优势,那么你需要知道不使用这个框架之前是怎么处理的,才能知道框架做了一些什么事情.

针对Spring的学习,第一步就是了解没有spring和struts框架之前的Java web是如何开发的.你会知道那时候使用JSP和Servlet.然后你就知道,Servlet是一个规范,那在Spring里面,Servlet去哪了？ 这时候你知道了DispatchServlet.然后你会了解到IoC和AOP. 

> 很多新的技术只不过是引入了新的编程元素对原来技术进行了封装.


##### 2
其实Java Web开发,spring不是第一步,首先需要理解的是 HTTP协议. chrome的DevTools和curl,postman要有基本使用.

还要知道服务器发送给浏览器的响应是没有没有JS,CSS和图片等外部资源的,浏览器在解析响应时才会再次请求这些资源,这里会出现一些静态资源请求不到的问题,SpringMVC是怎么配置的？还有chrome并发请求数量限制,如何合并雪碧图提高网页加载速度等知识点,属于http知识了.

接下来,学习Servlet和JSP.这个步骤不是可以跳过的,现在流行的框架Spring MVC和Struts2其实都是基于Servlet的,只有深入理解了Servlet才能理解后面的新技术.

下面几个知识点可以检测你是否理解了Servlet：

1、什么是ServletContext,和tomcat等web容器的关系时什么？Servlet 工作原理解析

简单的说,我们在浏览器点击链接和按钮产生的消息不是发送给Servlet的,而是发送给web容器的(在JSP出现之前,web容器也叫Servlet容器),web容器接收消息后不知道怎么处理,转交给我们编写的Servlet处理,那么web容器怎么和Servlet交流呢？于是就出现了Servlet接口,接口是定义一种规范的良好表达形式. 只要我们编写的Java类符合Servlet规范,那么就能被Web容器识别并被容器管理.

2、什么是Session？Session在实际工程中的应用场景.以及@SessionAttribute注解的局限性.

3、JSP是面向服务器的,它并不知道浏览器是什么鬼,是我们在写JSP时预设客户端是浏览器,JSP就是一个Servlet.JSP的常用对象和指令.

4、JSP的中文编码乱码有几种情况？各自的解决方法？提示： JSP文件的编码,浏览器的解析编码,GET请求的编码,POST的编码.

5、Servlet是一种接口规范,其中请求和响应是Servlet容器通过向方法的参数赋值HttpServletRequest或者HttpServletResponse传递的.在Struts1里面,将doGet()方法里的响应移到返回值里.在Struts2里则:

> 在Controller中彻底杜绝引入HttpServletRequest或者HttpServletResponse这样的原生Servlet对象.
> 同时将请求参数和响应数据都从响应方法中剥离到了Controller中的属性变量.


这是一个很大的技术改造,也造成了Struts2的盛行.Spring MVC走的是中间路线,Spring的2.0.8之前的版本甚至直接使用Servlet的doGet的.Spring MVC现在开始流行主要还是因为Schema xml的精简和基于注解的配置.所以这里出现了新的知识点：Schema Based XML的相关知识和Java5引入的注解原理.


书籍：推荐许令波的书《深入分析Java Web技术内幕(修订版)》和计文柯的《深入理解spring技术内幕》,特别是第二本,对spring的分析很是彻底.
