+++
title = "Useful Scala Code Snippets"
date = "2019-04-26T07:54:00+08:00"
categories = [ "编程",]
tags = [ "code", "scala",]
toc = "true"
+++


## merge two map and sum its values

多个 map 合并，key 相同时则 value 相加

```scala
val map1 = Map(1 -> 1, 2 -> 2)
val map2 = Map(1 -> 11, 3 -> 3)
val map3 = Map(1 -> 111, 3 -> 3)

val mapList = List(map1, map2, map3)

val merged = mapList.reduce((m1, m2) =>
  m1 ++ m2.map { case (k, v) => k -> (v + m1.getOrElse(k, 0)) }
)
```

<!--more-->

## 文件读

```scala
// """"""可以避免\\符号
val file = """d:\data\file.txt"""
for (line <- Source.fromFile(file, encoding).getLines()) {
    print(line)
}
```

## 文件写

```scala
  //资源管理
  def using[A <: {def close() : Unit}, R](resource: A)(fun: A => R): R = {
    import scala.language.reflectiveCalls
    try {
      fun(resource)
    } finally {
      resource.close()
    }
  }
  
  using(new OutputStreamWriter(new FileOutputStream(outputFile), StandardCharsets.UTF_8)) {
      writer => writer.write(s"""${line}\n""")
  }
```

## 统计词频
```scala
 val nanoUnit = 1000000

  //分词并统计词频
  def main(args: Array[String]): Unit = {
    val path = """D:\code\ideaProjects\scala-notes\data\src\out"""
    val files: List[File] = new File(path).listFiles.filter(_.isFile).toList
    val start = System.nanoTime()
    val wfList = ListBuffer[mutable.Map[String, Long]]()
    val futures = for (file <- files) yield Future {
      countWrodsInFile(file, "UTF-8")
    }
    for (f <- futures) {
      val words: mutable.Map[String, Long] = Await.result(f, Duration.Inf)
      wfList += words
    }
    //merge the word frequency map
    val finalWf = wfList.reduce((m1, m2) =>
      m1 ++ m2.map { case (k, v) => k -> (v + m1.getOrElse(k, 0L)) }
    )
    val end = System.nanoTime()
    println(s"container size=${finalWf.size}")
    // sort map
    val wordsFreq = finalWf.toList.sortWith(_._2 > _._2)

    write2file(wordsFreq, Paths.get(path, "final.txt").toFile)
    println(s"total used time = ${(end - start) / nanoUnit} ms")
    println(s"cups = ${Runtime.getRuntime.availableProcessors()}")
  }

  def countWrodsInFile(file: File, encoding: String): mutable.Map[String, Long] = {
    val wf = mutable.Map[String, Long]().withDefaultValue(0)
    for (line <- Source.fromFile(file, encoding).getLines()) {
      val l = line.trim
      wf.update(l, wf(l) + 1)
    }
    println(s"${file.getName} has words:${wf.size}")
    wf
  }

  def write2file(wf: Seq[(String, Long)], out: File): Unit = {
    using(new OutputStreamWriter(new FileOutputStream(out), StandardCharsets.UTF_8)) {
      writer =>
        for (it <- wf) {
          writer.write(s"""${it._1} ${it._2}\n""")
        }
    }
  }

```