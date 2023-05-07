---
title: "Kotlin Coroutine"
date: "2023-04-30T11:02:29+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
---
    A coroutine is an instance of suspendable computation. 

协程是可被挂起的计算的实例. 换句话说协程是一个对象, 这个对象保存着一段可以切换线程的任务 + 当前执行的状态两部分信息. 
日常涉及协程的编码, 主要是描述协程的任务和管理多个协程的生命周期、异常处理等. 

Kotlin 使用堆栈帧管理要运行哪个函数以及所有局部变量. 挂起协程时, 系统会复制并保存当前的堆栈帧以供稍后使用. 恢复时, 会将堆栈帧从其保存位置复制回来, 然后函数再次开始运行. 即使代码可能看起来像普通的顺序阻塞请求, 协程也能确保网络请求避免阻塞主线程. 


<!--more-->

### 问题场景
假设现在有个场景, 根据用户id调用两个外部接口获取用户的姓名和公司名称, 拼接后返回. 
由于两个外部接口耗时较高, 直接的思路就是使用两个线程来发送请求然后等待请求全部响应后拼接响应值. 

#### 方式1 Java的Callable

```java
// 定义两个Callable来异步执行方法
Callable<String> getUserName = () -> {
    // 模拟调用耗时方法获取用户名
    Thread.sleep(1000); 
    return "John";
};

Callable<String> getCompany = () -> {
    // 模拟调用耗时方法获取公司名
    Thread.sleep(1000);
    return "Doe Corp."; 
};

// 使用ExecutorService执行两个Callable并获取Future
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<String> nameFuture = executor.submit(getUserName);
Future<String> companyFuture = executor.submit(getCompany);

// 在主线程中获取结果并合并
String name = nameFuture.get();    
String company = companyFuture.get();
String info = name + ", " + company;
System.out.println(info); // John, Doe Corp.

executor.shutdown();
```
#### 方式2 Java的CompletableFuture
```java
// 定义两个异步操作
CompletableFuture<String> getUserName = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(1000);
    return "John";
});

CompletableFuture<String> getCompany = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(1000);
    return "Doe Corp.";
});

// 使用thenCombine合并两个异步操作的结果
CompletableFuture<String> combined = getUserName.thenCombine(getCompany, (name, company) -> name + ", " + company);

// 获取最终结果
String result = combined.get(); 
System.out.println(result); // John, Doe Corp.
```

#### 方式3 Kotlin的Coroutine

```kotlin
suspend fun getUserName(): String {
    delay(1000)  // 模拟调用耗时方法
    return "John"
}

suspend fun getCompany(): String {
    delay(1000)  // 模拟调用耗时方法
    return "Doe Corp."
}

fun main() {
    // 使用async并发调用两个suspend函数
    val name = async { getUserName() }
    val company = async { getCompany() }
    
    // 使用await等待两个任务完成并获取结果
    val nameResult = name.await()  
    val companyResult = company.await()
    
    println("$nameResult, $companyResult") // John, Doe Corp. 
}

// 或者使用awaitAll
suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async { fetchDoc(1) },  // async returns a result for the first doc
            async { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
        // The awaitAll function should be preferred over map { it.await() }
    }
```

### suspend关键字
suspend函数是协程中的任务描述部分, suspend关键字只是一个语法提示, 告诉函数调用者该函数可能被切换线程, 同理, 也只能在suspend函数内部调用其他suspend函数, 例如上面的delay. 

编译器和IDE根据suspend关键字来做一个语法提示与校验. 

### coroutine builder
利用suspend fun只能描述任务/函数, 还需要使用coroutine builder来创建协程. 
`launch`函数会创建一个协程返回一个`Job`不包含协程结果信息. `async`函数也创建一个协程返回`Deferred`-类似Future包含协程的未来计算结果. 可以通过`Deferred`对象的await方法获取结果值. 
所有的coroutine builder都是`CoroutineScope`的扩展函数, 因为任何协程的生命周期都由对应的`CoroutineScope`对象管理。后面会看到有些方法会默认创建`CoroutineScope`对象。

```kotlin
val time = measureTimeMillis {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")
```
`start = CoroutineStart.LAZY`的async协程只有在被调用`start`或者`await`时才会启动. 

```kotlin
val time = measureTimeMillis {
    val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
    val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }

    one.start() // 不会阻塞，直接下一行执行
    two.start() 
    println("The answer is ${one.await() + two.await()}") //注意, 如果没有上面两个start的话, 那么这两个await是先后调用，导致两个协程顺序执行而不是异步
}
println("Completed in $time ms")
```

### 结构化并发
还是上面方式3的两个suspend函数, 如果其中一个方法异常, 另一个方法也就没有必要继续执行了, 在Java多线程目前难以做到（JEP 428已经实现, 参考jdk19的StructuredTaskScope类）, 
而在kotlin协程中, 只需要将两个线程放在同一个`CoroutineScope`即可实现:

```kotlin
fun main() = runBlocking<Unit> {
    try {
        failedConcurrentSum()
    } catch(e: ArithmeticException) {
        println("Computation failed with ArithmeticException")
    }
}

suspend fun failedConcurrentSum(): Int = coroutineScope { //coroutineScope函数创建一个新的scope 
    val one = async<Int> { 
        try {
            delay(Long.MAX_VALUE) // Emulates very long computation
            42
        } finally {
            println("First child was cancelled")
        }
    }
    val two = async<Int> { 
        println("Second child throws an exception")
        throw ArithmeticException()
    }
    one.await() + two.await()
}
```
结构化并发是kotlin协程的核心优势之一, 只有在你遇到复杂的场景时才能感受到结构化并发的威力与优雅. 

### Dispatcher
CoroutineDispatcher用来决定哪个（或几个）线程来运行该协程, 可以将协程的执行限制在一个线程或者某个线程池, 或者不限制. 自带的几个dispatcher: 
`Dispatchers.Main`: A coroutine dispatcher that is confined to the Main thread operating with UI objects. Usually such dispatcher is single-threaded.
`Dispatchers.Default`: The default CoroutineDispatcher that is used by all standard builders like launch, async, etc. if no dispatcher nor any other ContinuationInterceptor is specified in their context.
`Dispatchers.IO`: The CoroutineDispatcher that is designed for offloading blocking IO tasks to a shared pool of threads.
`Dispatchers.Unconfined`: A coroutine dispatcher that is not confined to any specific thread. It executes initial continuation of the coroutine in the current call-frame and lets the coroutine resume in whatever thread that is used by the corresponding suspending function, without mandating any specific threading policy.

注意, 即使是同一个函数内的不同行代码也不一定在同一个线程上面执行. 

注意, Dispatcher实现了CoroutineContext接口, 所以会看到`withContext(Dispatchers.IO) {}`用法. 

### CoroutineContext
协程执行时总有带有一个CoroutineContext, 可以理解为就是一个元信息Map, 保存了Job、coroutine dispatcher 等信息:
Job: 控制协程的生命周期. 
CoroutineDispatcher: 将工作分派到适当的线程. 
CoroutineName: 协程的名称, 可用于调试. 
CoroutineExceptionHandler: 处理未捕获的异常. 

coroutine builder（async、launch）接收可选的CoroutineContext对象参数. CoroutineContext最常见的用途就是指定协程的dispatcher. 

在kotlin中, `CoroutineContext`表示协程的context, 包含了多个元素. 而`CoroutineContext.Element`表示context的一个元素. 类似map和kv的关系. 
但是`CoroutineContext.Element`继承了`CoroutineContext`, 即一个element也是一个context. 这种抽象可以简化一些API设计, 例如,withContext函数的参数类型是CoroutineContext,但是我们常常会传入一个CoroutineContext.Element的实现类如Dispatchers. 由于后者继承了前者,所以这样的使用方式也是被允许的. 
由于实现了plus操作符方法，`Job() + Dispatchers.Main`也表示一个`CoroutineContext`。

### CoroutineScope
CoroutineScope是协程最重要也是最难理解的点. CoroutineScope 给每个协程都定义了一个scope,用来组织和管理一组协程的生命周期. 
async和launch也是CoroutineScope的扩展函数. 很多教程里面直接调用async函数其实是使用了GlobalScope对象. 
获取独立的scope对象最佳实践是通过 `CoroutineScope()` 和 `MainScope()` 工厂函数. 一般不建议自己实现`CoroutineScope`接口. 

`suspend withContext`和`suspend coroutineScope`函数也叫scoping function. 
`withContext`: Calls the specified suspending block with a given coroutine context, suspends until it completes, and returns the result.
`coroutineScope`: Creates a CoroutineScope and calls the specified suspend block with this scope. The provided scope inherits its coroutineContext from the outer scope, but overrides the context's Job.
`withContext`比`coroutineScope`多了一个context:CoroutineContext参数. 

`withContext`几个使用示例

1. 切换到IO上下文执行IO操作
```kotlin
suspend fun doSomething() {
    withContext(Dispatchers.IO) {
        // 在IO上下文中执行IO密集型代码
        doNetworkRequest() 
    }
}
```
2. 切换到主线程更新UI
```kotlin
suspend fun doSomething() {
    val result = withContext(Dispatchers.Default) {
        // 在默认上下文中进行计算
        calculateResult() 
    }
    withContext(Dispatchers.Main) {
        // 在主线程中更新UI
        updateUI(result)
    }
}
```
3. 同时在两个不同上下文中执行任务
```kotlin
suspend fun doSomething() {
    val job1 = GlobalScope.launch(Dispatchers.IO) {
        // ...
    } 
    val job2 = GlobalScope.launch(Dispatchers.Main) {
        // ...
    }
    withContext(Dispatchers.IO) {
        job1.join()   // 等待IO上下文的任务结束
    }
    withContext(Dispatchers.Main) {
        job2.join()   // 等待主线程的任务结束 
    } 
}
```
4. 取消上下文切换
```kotlin
suspend fun doSomething() {
    withContext(NonCancellable) { // 使用NonCancellable上下文
        // 这里的代码块不会被取消
        doSomething()
    }
    // ...
}
```
#### withContext vs async
看上去除了返回值不一样, 两者的功能非常相似,都是接收context和block参数. 
```kotlin
// async
fun asyncDemo() = runBlocking {
    println("I am working")
    val opOne = async(IO) { operationOne() }.await() //注意 这里会阻塞等到operationOne返回才能继续下一行执行
    val opTwo = async(IO) { operationTwo() }.await()
    println("Done working.")
    println("The multiplied result is ${opOne * opTwo}")
}

// withContext
fun withContextDemo() = runBlocking {
    println("I am working")
    val opOne = withContext(IO) { operationOne() }
    val opTwo = withContext(IO) { operationTwo() }
    println("Done working.")
    println("The multiplied result is ${opOne * opTwo}")
}
```
其实async是用于并发异步编程的, 上面的async使用方式是不推荐的, 因为在创建一个协程后立即调用await会阻塞当前线程, 所以上面opOne和opTwo是顺序执行. 
withContext只是用于Context切换. 上面的代码其实也可以写成
```kotlin
val result = withContext(IO) { operationOne() + operationTwo() }
```

### Flow
```kotlin
fun simple(): Flow<Int> = flow { // flow builder, no suspend keyword before fun
    for (i in 1..3) {
        delay(1000) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

// Collect the flow
simple().collect { value -> println(value) }
// We can replace delay with Thread.sleep in the body of simple's flow { ... } and see that the main thread is blocked in this case.
```
Flow只有在collect调用时才计算, 也可以中途取消:
```kotlin
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
// only collected 1 2 
```
除了 `flow` 还有 `flowOf(1,2,3)` 、 `coll.asFlow()` 等flow builder函数. 
operator: `transform` `take` `collect` `toList/toSet` `first` `reduce` `fold - reduce with initial value`

`flowOn` change the context of a flow:

```ktolin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    simple().collect { value ->
        log("Collected $value") 
    } 
}            
```
如果collect函数比flow的emit还慢的话, 可以使用`buffer`将flow提前生成
```kotlin
val time = measureTimeMillis {
    simple() // 100ms for each element
        .buffer() // buffer emissions, don't wait
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
}   
println("Collected in $time ms")
```
### Channel
```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close() // we're done sending
}
// here we print received values using `for` loop (until the channel is closed)
for (y in channel) println(y)
println("Done!")
```

### 其他常用函数

`runBlocking`的签名`actual fun <T> runBlocking(context: CoroutineContext = EmptyCoroutineContext, block: suspend CoroutineScope.() -> T): T`
看着和withContext非常相似, 但是withContext是suspend函数, runBlocking不是. 
runBlocking 运行一个新的协程, 并可中断地阻塞当前线程, 直到协程完成. 此函数不应在协程中使用. 它旨在将常规的阻塞代码与挂起风格编写的库连接起来, 以便在main函数和测试中使用. 

前面说过，所有的协程都应该在一个CoroutineScope下面被管理。在`runBlocking {}`大括号内部写代码时IDE会提示你当前this的type是CoroutineScope，这个scope实际是runBlocking方法内构建的BlockingCoroutine对象。
由于`AbstractCoroutine`接口继承了`CoroutineScope`,所以BlockingCoroutine也是一个CoroutineScope实例。


`kotlin.system.measureTimeMillis` Executes the given block and returns elapsed time in milliseconds.

`delay` Delays coroutine for a given time without blocking a thread and resumes it after a specified time.

`suspend fun yield()` Yields the thread (or thread pool) of the current coroutine dispatcher to other coroutines on the same dispatcher to run if possible.

### Coroutine.start函数
```kotlin
public fun <R> start(start: CoroutineStart, receiver: R, block: suspend R.() -> T) {
    start(block, receiver, this) //这里实际调用的是CoroutineStart.invoke方法。this指的是当前coroutine
}
```

