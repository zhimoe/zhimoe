+++
title = "Java 6/7/8 中文件读写"
date = 2016-02-01
categories = [ "编程",]
tags = [ "java", "code",]
toc = "true"
+++

  
如何在 Java 中读写文件，这里保留 Java6/7 版本，但是你永远不应该使用它们，优先使用 Path,Files,Paths 三个类。
资料：[Reading and writing text files](http://www.javapractices.com/topic/TopicAction.do?Id=42)

## Java8 最佳实践

不要用 File 对象，改用 Path 对象，该对象既表示文件路径，也表示文件文本（应该认为文件也是路径的一部分）,对于以前的 File，可以 File.toPath() 得到一个 Path 对象。
Files 是一个静态类，操作文件内容.Paths 是静态工具类，操作文件路径，例如拼接文件路径，以前要使用平台无关的分隔符表示：File.pathSeparator, File.separator.
例如，构建一个文件对象：`Path path = Paths.get("~/test/", "foo", "bar", "a.txt");`
<!--more-->

## read file to string in java 6/7/8
```java
package angus.java.interview;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;

public class FileToStringJava678 {
	public static void main(String[] args) throws IOException {
		// How to read file into String before Java 7
		InputStream is = new FileInputStream("filetoStringjava678.txt");
		BufferedReader buf = new BufferedReader(new InputStreamReader(is));

		String line = buf.readLine();
		StringBuilder sb = new StringBuilder();

		while (line != null) {
			sb.append(line).append("\n");
			line = buf.readLine();
		}

		String fileAsString = sb.toString();
		System.out.println("Contents (before Java 7) : " + fileAsString);

		// Reading file into Stirng in one line in JDK 7 with using proper character encoding
		String fileString = new String(Files.readAllBytes(Paths.get("filetoStringjava678.txt")),
				StandardCharsets.UTF_8);
		System.out.println("Contents (Java 7 with character encoding ) : " + fileString);

        //java 7 按行读取
        BufferedReader br = new BufferedReader(new FileReader(file));
        String line;
        while((line = br.readLine()) != null) {
            // do something with line.
        }

        //java 8 按行读取
        String fileName = "c:/lines.txt";
        try (Stream<String> stream = Files.lines(Paths.get(fileName))) {
            stream.forEach(System.out::println);//or other thing you do with stream
        } catch (IOException e) {
            e.printStackTrace();
        }
        
		// It's even easier in Java 8
		Files.lines(Paths.get("filetoStringjava678.txt"), StandardCharsets.UTF_8).forEach(System.out::println);

        
	}
}

```

## java 8 file io demo
```java
public class Java8IO {
    public static void main(String[] args) throws IOException {
        //读取所有字节：
        Path path = Paths.get("alice.txt");
        String content = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
        System.out.println("Characters: " + content.length());
        // 读取所有行：
        List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
        System.out.println("Lines: " + lines.size());
        // JAVA 8 延迟处理：
        try (Stream<String> lineStream = Files.lines(path, StandardCharsets.UTF_8)) {
            System.out.println("Average line length: " +  lineStream.mapToInt(String::length).average().orElse(0));
        }

        // 按单词读取：
        try (Scanner in = new Scanner(path, "UTF-8")) {
            in.useDelimiter("\\PL+");//？
            int words = 0;
            while (in.hasNext()) {
                in.next();
                words++;
            }
            System.out.println("Words: " + words);
        }
        
        // 读取一个网页：   
        URL url = new URL("https://horstmann.com/index.html");
        try (BufferedReader reader
                = new BufferedReader(new InputStreamReader(url.openStream()))) {
            Stream<String> lineStream = reader.lines();////!!!! BufferedReader TO Stream
            System.out.println("Average line length: " + 
				lineStream.mapToInt(String::length).average().orElse(0));
        }
        
        // PrintWriter 向文本写文件：
        path = Paths.get("hello.txt");
        try (PrintWriter out = new PrintWriter(Files.newBufferedWriter(path, StandardCharsets.UTF_8))) {
            out.println("Hello");
        }
        
        // Files.write 向文本写文件：
        content = "World\n";
        Files.write(path, content.getBytes(StandardCharsets.UTF_8), StandardOpenOption.APPEND);
        
        // 多行写入
        String fileName = "file.txt";
        Path path = Paths.get("file1.txt");
        List<String> list = new ArrayList<>();
        try (Stream<String> lines = Files.lines(Paths.get(fileName))) {
            lines.forEach(list::add);
            Files.write(path, list, StandardCharsets.UTF_8);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 打印错误栈：  
        StringWriter writer = new StringWriter();
        Throwable throwable = new IllegalStateException();
        throwable.printStackTrace(new PrintWriter(writer));
        String stackTrace = writer.toString();
        System.out.println("Stack trace: " + stackTrace);

        // 输入流保存到文件：
        Files.copy(inputStream,filepath,StandardCopyOption.REPLACE_EXISTING);
        
        // 直接将 url 中的 pdf 保存下来：
        // 适用于任何二进制文件：
        URL url = new URL("http://www.cninfo.com.cn/1202417936.PDF");
        try (InputStream in = new BufferedInputStream(url.openStream())) {
            Files.copy(in, Paths.get(url.getFile().substring(1)),StandardCopyOption.REPLACE_EXISTING);
        }
        // url.getFile().substring(1) 去掉起始地斜杠符
        // copy() 有三种形式

        // 还有一种方式用于 jdk7 之前：
        URL website = new URL("XXX.pdf");
        ReadableByteChannel rbc = Channels.newChannel(website.openStream());
        FileOutputStream fos = new FileOutputStream(url.getFile().substring(1));
        fos.getChannel().transferFrom(rbc, 0, Long.MAX_VALUE);
        // FileChannel 的抽象方法 abstract long	transferFrom(ReadableByteChannel src, long position, long count) 

    }
}

```
