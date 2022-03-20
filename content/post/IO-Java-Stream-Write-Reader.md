---
title: 'IO-Java-Stream-Write-Reader'
date: 2016-03-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

上次总结了java中不同读写文件的方法,这次总结一下基本的IO流.网上的总结大部分是以Stream和Reader、Writer来介绍的.这次从封装层次来介绍.

<!--more-->

### java reader writer stream
![Java IO继承图](http://pic002.cnblogs.com/images/2012/384764/2012031413373126.jpg)

首先是byte流,每次read()读取8 bits,并用一个int的低八位保存：

```java
FileInputStream in = null;
FileOutputStream out = null;
		try {
			in = new FileInputStream("xanadu.txt");
			out = new FileOutputStream("outagain.txt");
			int c;
			while ((c = in.read()) != -1) {
				out.write(c);
			}
			
		} finally { 
			if (in != null) {
				in.close();
			}
			if (out != null) {
				out.close();
			}
		}

```
byte流是很基础的流,接下来是字符流,使用int的低16位保存读取内容,一个汉字,使用上面那个字节流,需要读取2次,使用下面的字符流,只用一次.其实背后还是一个桥接
具体的对象体现：
FileReader extemds InputStreamReader,  
FileWriter extends OutputStreamWriter  


InputStreamReader:字节到字符的桥梁  
OutputStreamWriter:字符到字节的桥梁：  

```
        FileReader inputStream = null;
        FileWriter outputStream = null;

        try {
            inputStream = new FileReader("xanadu.txt");
            outputStream = new FileWriter("characteroutput.txt");

            int c;
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }

```

ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流,它们分别从Byte 数组、StringBuffer、和本地文件中读取数据.StringBufferInputStream 已经被Deprecated,设计错误,只是为了兼容.

File I/O现在已经不推荐使用了,推荐nio2的Path及其工具类Files,Paths;
[Path 官方教程](http://docs.oracle.com/javase/tutorial/essential/io/path.html)

ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）

注意：OutputStream子类中没有StringBuffer为目的地的.ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流.


几个特殊的类：  
PushbackInputStream 的功能是查看最后一个字节,不满意就放入缓冲区.主要用在编译器的语法、词法分析部分.输出部分的BufferedOutputStream 几乎实现相近的功能.

PrintStream 也可以认为是一个辅助工具.主要可以向其他输出流,或者FileInputStream 写入数据,本身内部实现还是带缓冲的.本质上是对其它流的综合运用的一个工具而已.一样可以踢出IO 包！System.out 和System.err 就是PrintStream 的实例！ System.in是inputStream的实例！
**你永远不应该new PrintStream,请用PrintWriter**


看看字符流的对比：
![Reader/Writer](http://pic002.cnblogs.com/images/2012/384764/2012031413390861.png)

CharReader、StringReader 是两种基本的介质流,它们分别将Char 数组、String中读取数据.PipedReader 是从与其它线程共用的管道中读取数据.

BufferedReader 很明显就是一个装饰器,它和其子类负责装饰其它Reader 对象.

FilterReader 是所有自定义具体装饰流的父类,其子类PushbackReader 对Reader 对象进行装饰,会增加一个行号.

InputStreamReader 是一个连接字节流和字符流的桥梁,它将字节流转变为字符流.FileReader 可以说是一个达到此功能、常用的工具类,在其源代码中明显使用了将FileInputStream 转变为Reader 的方法.我们可以从这个类中得到一定的技巧.Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致.后面会有Reader 与InputStream 的对应关系.

OutputStreamWriter 是OutputStream 到Writer 转换的桥梁,它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）.功能和使用和OutputStream 极其类似,后面会有它们的对应图.

PrintWriter 和PrintStream 极其类似,功能和使用也非常相似.但是还是有不同的,`PrintStream` prints to an `OutputStream`, and `PrintWriter` prints to a `Writer`. 

**你永远不应该new PrintStream,请用PrintWriter**

```java
PrintStream stream = new PrintStream(outputStream);
//With the PrintWriter you can however pass an OutputStreamWriter with a specific encoding.  
PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream, "UTF-8"));

```

### Piped流  

这是线程之间通信使用的.后面介绍.

### RandomAccessFile类

该对象并不是流体系中的一员,其封装了字节流,同时还封装了一个缓冲区（字符数组）,通过内部的指针来操作字符数组中的数据. 该对象特点：

该对象只能操作文件,所以构造函数接收两种类型的参数：a.字符串文件路径；b.File对象.  
该对象既可以对文件进行读操作,也能进行写操作,在进行对象实例化时可指定操作模式(r,rw)  
注意：该对象在实例化时,如果要操作的文件不存在,会自动创建；如果文件存在,写数据未指定位置,会从头开始写,即覆盖原有的内容. 可以用于多线程下载或多个线程同时写数据到文件.  


### Scanning and formatting

The `scanner` API breaks input into individual tokens associated with bits of data,The formatting API assembles data into nicely formatted, human-readable form.  
[formatting](http://docs.oracle.com/javase/tutorial/essential/io/formatting.html)


```java
int i = 2;
double r = Math.sqrt(i);
System.out.format("The square root of %d is %f.%n", i, r);

```

```java
 Scanner s = new Scanner(new BufferedReader(new FileReader("xanadu.txt")));

```

By default, a scanner uses white space to separate tokens. also,u can set :
`s.useDelimiter(",\\s*");`

### I/O from commandline 

You might expect the Standard Streams to be character streams, but, for **historical** reasons, they are byte streams. `System.out` and `System.err` are defined as `PrintStream` objects. Although it is technically a byte stream, `PrintStream` utilizes an internal character stream object to emulate many of the features of character streams.

By contrast, System.in is a byte stream with no character stream features. To use Standard Input as a character stream, wrap System.in in InputStreamReader.

`InputStreamReader cin = new InputStreamReader(System.in);`

！！！！妈的,老子开始就困惑很久了,一直不明白System.out怎么可以直接打印出中文.

jdk1.5开始读写控制台以前常用的是Scanner：

```java
Scanner scanner = new Scanner(System.in);  
scanner.nextLine();  

```
从 JDK1.6开始,基本类库中增加了java.io.Console 类,用于获得与当前 Java 虚拟机关联的基于字符的控制台设备.在纯字符的控制台界面下,可以更加方便地读取数据.

```java
        Console console = System.console();  
        if (console == null) {  
            throw new IllegalStateException("不能使用控制台");  
        }  
        return console.readLine(prompt);  

```

### Data Streams

Data streams support binary I/O of primitive data type values (boolean, char, byte, short, int, long, float, and double) as well as String values. All data streams implement either the DataInput interface or the DataOutput interface. This section focuses on the most widely-used implementations of these interfaces, DataInputStream and DataOutputStream.  


致谢：[Oubo的博客](http://www.cnblogs.com/oubo/archive/2012/01/06/2394638.html)
