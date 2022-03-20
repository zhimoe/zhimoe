---
title: "Scala Python 文件读取跳过转义字符"
date: "2020-06-04T21:30:31+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
 - python
---
在文件读取的时候,会遇到非法转义字符,导致文件按行读取失败.此时可以通过忽略转义字符来解决.本文记录了scala和python的方法.

<!--more-->

## 背景
有50G的服务器日志,拆分为几千个txt文件,编码是utf8,使用scala和python按行处理：

scala
```scala
def main(args: Array[String]): Unit = {
  for (line <- Source.fromFile("./txt1.log","UTF8").getLines()) {
    if (line.contains("ABC")) {
      //do something      
    }
  }
}
```
python
```python
with open('./txt1.log','r',encoding='utf-8') as f:
    for line in f:
        pass #do something
    
```

但是文本中有一些行包含非法的转义字符,例如：
```text
http://bbc.com/search.html \xa3\xa9 404 \r\n 李晓明
```
导致程序异常:
```text
#scala
java.nio.charset.MalformedInputException: Input length = 1

#python
'utf-8' codec can't decode byte 0xa3 in position 168: invalid start byte
```
## 方案
一般遇到这种非法转义字符,可以跳过这个错误,看成raw string来处理.

scala
```scala
import java.nio.charset.CodingErrorAction
import scala.io.{Codec, Source}

implicit val codec = Codec("UTF-8")
codec.onMalformedInput(CodingErrorAction.REPLACE)
codec.onUnmappableCharacter(CodingErrorAction.REPLACE)

// 注意,fromFile方法没有提供"UTF8"参数
def main(args: Array[String]): Unit = {
  for (line <- Source.fromFile("./test.file").getLines()) {
    if (line.contains("ABC")) {
      //do something
    }
  }
}
```
python
```python
with open('./txt1.log','r',encoding='utf-8',errors='ignore') as f:
    for line in f:
        pass #do something
    
```
如果确认文本中没有中文的话,也可以使用下面的方式直接将其转义掉
```python
with open('./txt1.log','r',encoding='unicode_escape') as f:
```