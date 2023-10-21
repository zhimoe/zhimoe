+++
title = "Highlights in Scala for Impatient 2nd"
date = "2020-01-15T21:42:33+08:00"
categories = [ "编程",]
tags = [ "code", "scala", "scala-for-impatient",]
toc = "true"
+++


key points in scala-for-impatient 2nd book, best book for java developer to use scala in a rush.
scala-for-impatient 章节摘要，这本书对于 Java 开发者快速上手 Scala 帮助很大。

### Functions
```text
• if expression has a value.
• A block has a value — the value of its last expression.
• The Scala for loop is like an “enhanced” Java for loop.
• Semicolons are (mostly) optional.
• The void type is Unit.
• Avoid using `return` in a function.
• Beware of missing = in a function definition.
• Exceptions work just like in Java or C++, but you use a “pattern matching” syntax for catch.
• Scala has no checked exceptions.

```

<!--more-->

### Arrays
```text
• Use an Array if the length is fixed, and an ArrayBuffer if the length can vary.
• Don’t use new when supplying initial values.
• Use () to access elements.
• Use for (elem <- arr) to traverse the elements.
• Use for (elem <- arr if . . . ) . . . yield . . . to transform into a new array.
• Scala and Java arrays are interoperable; with ArrayBuffer, use scala.collection.JavaConverters._ 
don't use scala.collection.JavaConversions.
```

```scala
import scala.collection.mutable.ArrayBuffer
val b = ArrayBuffer[Int]()
// Or new ArrayBuffer[Int]
// An empty array buffer, ready to hold integers

b += 1
// ArrayBuffer(1)
// Add an element at the end with +=

b += (1, 2, 3, 5)
// ArrayBuffer(1, 1, 2, 3, 5)
// Add multiple elements at the end by enclosing them in parentheses

b ++= Array(8, 13, 21)
// ArrayBuffer(1, 1, 2, 3, 5, 8, 13, 21)
// You can append any collection with the ++= operator

b.trimEnd(5)
// ArrayBuffer(1, 1, 2)
// Removes the last five elements

b.insert(2, 6)
// ArrayBuffer(1, 1, 6, 2)
// Insert before index 2


// iterate array with index 
// use view or use index(it's faster)
for (i<-b.indices){val v = b(i)}

```

### Maps & Tuples
```scala
var scores = Map("Alice" -> 10, "Bob" -> 3, "Cindy" -> 8)
val scores2 = Map(("Alice", 10), ("Bob", 3), ("Cindy", 8))
val bobsScore = scores("Bob") // Like scores.get("Bob") in Java

val scores1 = scala.collection.mutable.Map("Alice" -> 10, "Bob" -> 3, "Cindy" -> 8)
val scores3 = scala.collection.mutable.Map[String, Int]()
scores1("Bob") = 10 
val v = scores1("Bob") // NPE if key is not exists
scores1.get("Bob") // None if key is not exists
scores1.getOrElse("Bob",10)// 10 if key is not exists
scores1 += ("Bob" -> 10, "Fred" -> 7)
scores1 -= "Alice"

// for immutable
val newScores = scores + ("Bob" -> 10, "Fred" -> 7) // New map with update
val scores4 = scores - "Alice"
scores -= "Alice"


for ((k, v) <- scores){}

for ((k, v) <- scores) yield (v, k)

// sorted map
val sortedScores = scala.collection.mutable.SortedMap("Alice" -> 10,"Fred" -> 7, "Bob" -> 3, "Cindy" -> 8)
// insert order
val months = scala.collection.mutable.LinkedHashMap("January" -> 1,"February" -> 2, 
"March" -> 3, "April" -> 4, "May" -> 5)

import scala.collection.JavaConverters._

// tuple 
val t = (1, 3.14, "Fred")
val second = t._2 // Sets second to 3.14


val keys = Array()
val values = Array()
val kv = keys.zip(values).toMap()

```

### Object
```text
• Use objects for singletons and utility methods.
• A class can have a companion object with the same name.
• Objects can extend classes or traits.
• The apply method of an object is usually used for constructing new instances of the companion class.
• To avoid the main method, use an object that extends the App trait.
• You can implement enumerations by extending the Enumeration object.
```

### Package
```text

The key points of this chapter are:
• Packages nest just like inner classes.
• Package paths are not absolute.
• A chain x.y.z in a package clause leaves the intermediate packages x and x.y invisible.
• Package statements without braces at the top of the file extend to the entire file.
• A package object can hold functions and variables.
• Import statements can import packages, classes, and objects.
• Import statements can be anywhere.
• Import statements can rename and hide members.
• java.lang, scala, and Predef are always imported
```

### Inheritance
```text
• The `extends` and `final` keywords are as in Java.
• You must use `override` when you override a method.
• Only the primary constructor can call the primary superclass constructor.
• You can `override` fields.
```
### Files
```text
• Source.fromFile(...).getLines.toArray yields all lines of a file.
• Source.fromFile(...).mkString yields the file contents as a string.
• To convert a string into a number, use the toInt or toDouble method.
• Use the Java PrintWriter to write text files.
• "regex".r is a Regex object.
• Use """...""" if your regular expression contains backslashes or quotes.
• If a regex pattern has groups, you can extract their contents using the syntax for 
  (regex(var1, ...,varn) <- string).
```

### Traits
```text
Key points of this chapter:
• A class can implement any number of traits.
• Traits can require implementing classes to have certain fields, methods, or superclasses.
• Unlike Java interfaces, a Scala trait can provide implementations of methods and fields.
• When you layer multiple traits, the order matters—the trait whose methods execute first goes to the back.
```

### Operators
```scala
val a = 10
//a: Int = 10

-a
// res0: Int = -10
//means the same as a.unary_-.

a.unary_-
//res1: Int = -10


```

### High Order Functions
```text
Array(3.14, 1.42, 2.0).map{ (x: Double) => 3 * x }
//==
Array(3.14, 1.42, 2.0) map { (x: Double) => 3 * x }

// diff method and function

```

### Collections
```text
The key points of this chapter are:
• All collections extend the Iterable trait.
• The three major categories of collections are sequences, sets, and maps.
• Scala has mutable and immutable versions of most collections.
• A Scala list is either empty, or it has a head and a tail which is again a list.
• Sets are unordered collections.
• Use a LinkedHashSet to retain the insertion order or a SortedSet to iterate in sorted order.
• + adds an element to an unordered collection; +: and :+ prepend or append to a sequence; 
  ++ concatenates two collections; - and -- remove elements.
• The Iterable and Seq traits have dozens of useful methods for common operations. 
  Check them out before writing tedious loops.
• Mapping, folding, and zipping are useful techniques for applying a function or operation to 
  the elements of a collection
```

```text
 Iterable trait methods:
head, last, headOption, lastOption
tail, init
length, isEmpty
map(f), flatMap(f), foreach(f), transform(f), collect(pf)
reduceLeft(op), reduceRight(op),foldLeft(init)(op), foldRight(init)(op)
reduce(op), fold(init)(op),aggregate(init)(op, combineOp)
sum, product, max, min
count(pred), forall(pred), exists(pred)
filter(pred), filterNot(pred), partition(pred)
takeWhile(pred), dropWhile(pred), span(pred)
take(n), drop(n), splitAt(n)
takeRight(n), dropRight(n)
slice(from, to), view(from, to)
zip(coll2), zipAll(coll2, fill, fill2), zipWithIndex(cation! the 2nd value in tuple is index)
grouped(n), sliding(n)
groupBy(k) //
mkString(before, between, after), addString(sb, before, between, after)

toIterable, toSeq, toIndexedSeq,
toArray, toBuffer, toList, toStream,
toSet, toVector, toMap, to[C]
```

```text
Important Methods of the Seq Trait:

contains(elem), containsSlice(seq),
startsWith(seq), endsWith(seq)

indexOf(elem), lastIndexOf(elem),
indexOfSlice(seq),
lastIndexOfSlice(seq),
indexWhere(pred)

prefixLength(pred),
segmentLength(pred, n)

padTo(n, fill)

intersect(seq), diff(seq)

reverse

sorted, sortWith(less), sortBy(f)

permutations, combinations(n)
```

```scala
//The map and flatMap methods are important because they are used
//for translating for expressions. For example, the expression:

for (i <- 1 to 10) yield i * i
//is translated to
(1 to 10).map(i => i * i)
//and
for (i <- 1 to 10; j <- 1 to i) yield i * j
//becomes
(1 to 10).flatMap(i => (1 to i).map(j => i * j))

```

```scala

val coll = List()
coll.par.sum
coll.par.count(_ % 2 == 0)
for (i <- (0 until 100000).par) print(s" $i")

(for (i <- (0 until 100000).par) yield i) == (0 until 100000)

```

### Pattern Matching
```text
The key points of this chapter are:
• The match expression is a better switch, without fall-through.
• If no pattern matches, a MatchError is thrown. Use the case _ pattern to avoid that.
• A pattern can include an arbitrary condition, called a guard.
• You can match on the type of an expression; prefer this over isInstanceOf/asInstanceOf.
• You can match patterns of arrays, tuples, and case classes, and bind parts of the pattern to variables.
• In a for expression, nonmatches are silently skipped.
• A case class is a class for which the compiler automatically produces the methods that are needed 
  for pattern matching.
• The common superclass in a case class hierarchy should be sealed.
• Use the Option type for values that may or may not be present—it is safer than using null.
```
### Annotations
```text
The key points of this chapter are:
• You can annotate classes, methods, fields, local variables, parameters,expressions, 
  type parameters, and types.
• With expressions and types, the annotation follows the annotated item.
• Annotations have the form @Annotation, @Annotation(value), or @Annotation(name1 =value1, ...).
• @volatile, @transient, @strictfp, and @native generate the equivalent Java modifiers.
• Use @throws to generate Java-compatible throws specifications.
• The @tailrec annotation lets you verify that a recursive function uses tail call optimization.
• The assert function takes advantage of the @elidable annotation. You can optionally remove 
  assertions from your Scala programs.
• Use the @deprecated annotation to mark deprecated features.
```

### Future
```text
The key points of this chapter are:
• A block of code wrapped in a Future { ... } executes concurrently.
• A future succeeds with a result or fails with an exception.
• You can wait for a future to complete, but you don’t usually want to.
• You can use callbacks to get notified when a future completes, but that gets
tedious when chaining callbacks.
• Use methods such as map/flatMap, or the equivalent for expressions, to compose
futures.
• A promise has a future whose value can be set (once), which gives added
flexibility for implementing tasks that produce results.
• Pick an execution context that is suitable for the concurrent workload of your
computation.
```

### Implicits
```text
The key points of this chapter are:
• Implicit conversions are used to convert between types.
• You must import implicit conversions so that they are in scope.
• An implicit parameter list requests objects of a given type. They can be obtained from 
  implicit objects that are in scope, or from the companion object of the desired type.
• If an implicit parameter is a single-argument function, it is also used as an implicit conversion.
• A context bound of a type parameter requires the existence of an implicit object of the given type.
• If it is possible to locate an implicit object, this can serve as evidence that a type conversion is valid.
```

### Type Class

### CanBuildFrom