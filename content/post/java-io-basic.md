+++
title = "Java Stream Write Reader notes"
date = 2016-03-01
categories = [ "编程",]
tags = [ "java", "code",]
toc = "true"
+++


上次总结了 java 中不同读写文件的方法，这次总结一下基本的 IO 流。网上的总结大部分是以 Stream 和 Reader、Writer 来介绍的。这次从封装层次来介绍。

<!--more-->

### 概览

![Java IO 继承图](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/java_io_stream_reader.4lgp0r6e14w0.webp)

首先理解计算机文件格式都是二进制数据，例如文本，图片，视频，音频等，但是文本非常特殊，所以单独有一类封装设计。
对于非文本类的文件，一般是读取字节 (stream)，而对于文本类文件，则可以读取字符 (reader)
当然，文本文件也可以使用 inputstream 读取后再转换`Reader reader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);`

### stream
首先是 byte 流，每次 read() 读取 8 bits，并用一个 int 的低八位保存：

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
byte 流是很基础的流，接下来是字符流，使用 int 的低 16 位保存读取内容，一个汉字，使用上面那个字节流，需要读取 2 次，使用下面的字符流，只用一次。其实背后还是一个桥接
具体的对象体现：
FileReader extemds InputStreamReader,  
FileWriter extends OutputStreamWriter  

InputStreamReader:字节到字符的桥梁  
OutputStreamWriter:字符到字节的桥梁：  

```java
FileReader inputReader = null;
FileWriter outputStream = null;

try {
    inputReader = new FileReader("xanadu.txt");
    outputStream = new FileWriter("characteroutput.txt");

    int c;
    while ((c = inputReader.read()) != -1) {
        outputStream.write(c);
    }
} finally {
    if (inputReader != null) {
        inputReader.close();
    }
    if (outputStream != null) {
        outputStream.close();
    }
}

```

ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从 Byte 数组、StringBuffer、和本地文件中读取数据.StringBufferInputStream 已经被 Deprecated，设计错误，只是为了兼容。

File I/O 现在已经不推荐使用了，推荐 nio2 的 Path 及其工具类 Files、Paths;
[Path 官方教程](http://docs.oracle.com/javase/tutorial/essential/io/path.html)

ObjectInputStream 和所有 FilterInputStream 的子类都是装饰流（装饰器模式的主角）
注意：OutputStream 子类中没有 StringBuffer 为目的地的。ObjectOutputStream 和所有 FilterOutputStream 的子类都是装饰流。


几个特殊的类：  
PushbackInputStream 的功能是查看最后一个字节，不满意就放入缓冲区。主要用在编译器的语法、词法分析部分。输出部分的 BufferedOutputStream 几乎实现相近的功能。

PrintStream 也可以认为是一个辅助工具。主要可以向其他输出流，或者 FileInputStream 写入数据，本身内部实现还是带缓冲的。本质上是对其它流的综合运用的一个工具而已。一样可以踢出 IO 包！System.out 和 System.err 就是 PrintStream 的实例！System.in 是 InputStream 的实例！
**你永远不应该 new PrintStream，请用 PrintWriter**


### reader writer
CharReader、StringReader 是两种基本的介质流，它们分别将 Char 数组、String 中读取数据.PipedReader 是从与其它线程共用的管道中读取数据。

BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它 Reader 对象。

FilterReader 是所有自定义具体装饰流的父类，其子类 PushbackReader 对 Reader 对象进行装饰，会增加一个行号。

InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流.FileReader 可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将 FileInputStream 转变为 Reader 的方法。我们可以从这个类中得到一定的技巧.Reader 中各个类的用途和使用方法基本和 InputStream 中的类使用一致。后面会有 Reader 与 InputStream 的对应关系。

OutputStreamWriter 是 OutputStream 到 Writer 转换的桥梁，它的子类 FileWriter 其实就是一个实现此功能的具体类（具体可以研究一 SourceCode）.功能和使用和 OutputStream 极其类似，后面会有它们的对应图。

PrintWriter 和 PrintStream 极其类似，功能和使用也非常相似。但是还是有不同的，`PrintStream` prints to an `OutputStream`, and `PrintWriter` prints to a `Writer`. 

**你永远不应该 new PrintStream，请用 PrintWriter**

```java
PrintStream stream = new PrintStream(outputStream);
//With the PrintWriter you can however pass an OutputStreamWriter with a specific encoding.  
PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream, "UTF-8"));

```

### RandomAccessFile 类

该对象并不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。该对象特点：

该对象只能操作文件，所以构造函数接收两种类型的参数：a.字符串文件路径；b.File 对象。 
该对象既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式 (r、rw)  
注意：该对象在实例化时，如果要操作的文件不存在，会自动创建；如果文件存在，写数据未指定位置，会从头开始写，即覆盖原有的内容。可以用于多线程下载或多个线程同时写数据到文件。 


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

You might expect the Standard Streams to be character streams, but, for **historical** reasons, they are byte streams. `System.out` and `System.err` are defined as `PrintStream` objects. Although it is technically a byte stream, `PrintStream` utilizes an internal character stream object to emulate many of the features of character streams.(！！！妈的，老子开始就困惑很久了，一直不明白 System.out 怎么可以直接打印出中文.)

By contrast, `System.in` is a byte stream with no character stream features. To use Standard Input as a character stream, wrap System.in in InputStreamReader.

`InputStreamReader cin = new InputStreamReader(System.in);`

jdk1.5 开始读写控制台以前常用的是 Scanner：

```java
Scanner scanner = new Scanner(System.in);  
scanner.nextLine();  

```
从 JDK1.6 开始，基本类库中增加了 java.io.Console 类，用于获得与当前 Java 虚拟机关联的基于字符的控制台设备。在纯字符的控制台界面下，可以更加方便地读取数据。

```java
Console console = System.console();  
if (console == null) {  
    throw new IllegalStateException("不能使用控制台");  
}  
return console.readLine(prompt);
```

### Data Streams

Data streams support binary I/O of primitive data type values (boolean, char, byte, short, int, long, float, and double) as well as String values. All data streams implement either the DataInput interface or the DataOutput interface. This section focuses on the most widely-used implementations of these interfaces, DataInputStream and DataOutputStream.  

