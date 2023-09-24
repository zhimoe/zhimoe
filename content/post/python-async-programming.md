+++
title = 'Python异步编程'
date = '2023-09-24T08:47:44+08:00'
categories = ['编程']
tags = ['async','python']
toc = true
+++

异步编程很难，但却是最近十年所有编程语言在发力的方向。
在面向CPU计算的场景下，多线程基本都能吃满CPU资源。但是在IO场景下，多线程并不能解决问题，大部分时间线程都在等待IO调用的返回。
实际上python的[官方教程](https://docs.python.org/3/tutorial/index.html)里面并没有async编程的内容，而是在[std lib doc中网络编程章节](https://docs.python.org/3/library/ipc.html)介绍了asyncio这个lib,实际上这也是异步编程的最佳使用场景。

此外经常会看到“Use async sparingly”,因为异步编程存在染色问题，一旦使用async，会要求你全链路全部为async，否则在block时cpu并无法让出线程资源。
大多数情况，如果出于性能原因不需要异步，线程通常是更简单的替代方案。

<!--more-->

### generator

generator示例：
```python
>>> def multi_yield():
...     yield_str = "This will print the first string"
...     yield yield_str
...     yield_str = "This will print the second string"
...     yield yield_str
...
>>> multi_obj = multi_yield()
>>> print(next(multi_obj))
This will print the first string
>>> print(next(multi_obj))
This will print the second string
>>> print(next(multi_obj))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

```python
def infinite_sequence():
    num = 0
    while True:
        yield num
        num += 1
```


### coroutine and future and task
- coroutines declared with the async/await syntax, is awaitable object.
  an object is an awaitable object if it can be used in an `await` expression. Many asyncio APIs are designed to accept awaitable objects. 
  There are three main types of awaitable objects: `coroutine`, `Task`, and `Future`.
- a coroutine function: an async def function; a coroutine object: an object returned by calling a coroutine function.
- The `asyncio.create_task()` function to run coroutines concurrently as asyncio Tasks.
   Tasks are used to schedule coroutines concurrently.
- A Future is a special low-level awaitable object that represents an eventual result of an asynchronous operation.
- Normally there is no need to create Future objects at the application level code.
  Future objects in asyncio are needed to allow callback-based code to be used with async/await.(重要)
- A good example of a low-level function that returns a Future object is `loop.run_in_executor()`.


coroutine简单示例：
```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

```

future和coroutine同时使用：
```python
async def main():
    await function_that_returns_a_future_object()

    # this is also valid:
    await asyncio.gather(
        function_that_returns_a_future_object(),
        some_python_coroutine()
    )
```

常用的api：
```python
coroutine asyncio.sleep(delay, result=None)
awaitable asyncio.gather(*aws, return_exceptions=False)
loop = asyncio.get_running_loop()
awaitable asyncio.shield(aw): Protect an awaitable object from being cancelled.
asyncio.timeout(delay)/timeout_at(when)/wait_for(aw, timeout):

async def main():
    async with asyncio.timeout(10):
        await long_running_task()
```

### runner
runners are built on top of an `event loop` with the aim to simplify async code usage for common wide-spread scenarios.
```python
asyncio.run(coro, *, debug=None)

# class asyncio.Runner(*, debug=None, loop_factory=None)
# Runner is a context manager that simplifies multiple async function calls in the same context 
async def main():
    await asyncio.sleep(1)
    print('hello')

with asyncio.Runner() as runner:
    runner.run(main())


```
### future
Future objects are used to bridge low-level callback-based code with high-level async/await code.

The rule of thumb is to never expose Future objects in user-facing APIs, and the recommended way to create a Future object is to call `loop.create_future()`. 
This way alternative event loop implementations can inject their own optimized implementations of a Future object.

实际项目中使用future的例子：
```python
# TBD

```


### event loop
The event loop is the core of every asyncio application. Event loops run asynchronous tasks and callbacks, perform network IO operations, and run subprocesses.
```python
def get_event_loop():
    """Return an asyncio event loop.

    When called from a coroutine or a callback (e.g. scheduled with call_soon
    or similar API), this function will always return the running event loop.

    If there is no running event loop set, the function will return
    the result of `get_event_loop_policy().get_event_loop()` call.
    """
    # NOTE: this function is implemented in C (see _asynciomodule.c)
    current_loop = _get_running_loop()
    if current_loop is not None:
        return current_loop
    return get_event_loop_policy().get_event_loop()
```

### stream
Streams are high-level async/await-ready primitives to work with network connections. Streams allow sending and receiving data without using callbacks or low-level protocols and transports.
可以理解成channel。主要包含 `asyncio.open_connection` `asyncio.start_server` `asyncio.open_unix_connection` `asyncio.start_unix_server`

### 多线程并发

#### threading version
```python
import concurrent.futures
import requests
import threading
import time


thread_local = threading.local()


def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session


def download_site(url):
    session = get_session()
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")


def download_all_sites(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)


if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")
```
#### multiprocessing version
```python

import multiprocessing
import time


def cpu_bound(number):
    return sum(i * i for i in range(number))


def find_sums(numbers):
    with multiprocessing.Pool() as pool:
        pool.map(cpu_bound, numbers)


if __name__ == "__main__":
    numbers = [5_000_000 + x for x in range(20)]

    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print(f"Duration {duration} seconds")
```