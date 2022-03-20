---
title: "Scala Future"
date: "2019-04-21T14:36:36+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - scala
---
some notes on scala future, includes:

<!--more-->

* [x] future
* [x] executor context
* [x] await future result
* [x] callback
* [ ] recover



## future
```scala

import java.time._
import scala.concurrent._
import ExecutionContext.Implicits.global
Future {
  Thread.sleep(10000)
  println(s"This is the future at ${LocalTime.now}")
}
println(s"This is the present at ${LocalTime.now}")
```

## executor context
future need a new thread to execute it task. `import ExecutionContext.Implicits.global` is a implicit threadpool.

## await for future result
```scala
// for 10.seconds conversion
import scala.concurrent.duration._
val f = Future { Thread.sleep(10000); 42 }
val result = Await.result(f, Duration.Inf)

// if f throw exception, it will rethrow to Await.result
// use ready() solve this
val f = Future { ... }
Await.ready(f, 10.seconds)
val Some(t) = f.value
// The f.value method returns an Option[Try[T]], 
// which is None when the future is not completed
// and Some(t) when it is is

// t is Try type instance
// A Try[T] instance is either a Success(v), where v is a value of type T or a Failure(ex)
val t = Some(t).get
t match {
case Success(v) => println(s"The answer is $v")
case Failure(ex) => println(ex.getMessage)
}
// or
if (t.isSuccess) println(s"The answer is ${t.get}")

```

## callback

```scala
val f = Future { Thread.sleep(10000)
  if (random() < 0.5) throw new Exception
  42
}
f.onComplete {
  case Success(v) => println(s"The answer is $v")
  case Failure(ex) => println(ex.getMessage)
}

```

## callback hell
```scala
val future1 = Future { getData1() }
val future2 = Future { getData2() }

future1 onComplete {
  case Success(n1) =>
    future2 onComplete {
      case Success(n2) => {
        val n = n1 + n2
        println(s"Result: $n")
      }
      case Failure(ex) => ...
    }
  case Failure(ex) => ...
}

// improve
val future1 = Future { getData1() }
val combined = future1.map(n1 => n1 + getData2())

// 
val future1 = Future { getData1() }
val future2 = Future { getData2() }
val combined = future1.map(n1 => future2.map(n2 => n1 + n2))

// use for-yield

for (
  n1 <- future1
  n2 <- future2
) yield n1+n2

```