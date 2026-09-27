+++
title = 'Fastapi 开发笔记'
date = '2025-09-14T14:51:25+08:00'
categories = ['编程']
tags = ['code','python', 'fastapi']
toc = true
+++

最近使用一个 fastapi 后端应用遇到一些性能问题，借助 GPT 和文档学习了一些框架底层知识，记录以便温习。

<!--more-->
# FastAPI 中 sync 和 async 的工作原理

FastAPI 可以无缝支持 `sync` 和 `async` 两种风格的 API，但二者的执行模型不同，在数据库、HTTP 请求以及大模型等长耗时 I/O 场景中需要特别注意。

## 1. Sync API：通过线程池执行
FastAPI 中普通的同步 API：

```python
@router.get("/users")
def get_users():
    return query_users()
```

不会直接在 Event Loop 线程中执行，而是由 Starlette 通过  [AnyIO](https://github.com/Kludex/starlette/blob/9f16bf5c25e126200701f6e04330864f4a91a898/starlette/concurrency.py#L36-L42)放到线程池中执行，避免同步阻塞操作卡住 Event Loop。

Starlette 默认的 AnyIO thread limiter 为 **40 tokens**，同步 endpoint、同步 dependency 等需要进入线程池的任务会共享这个限制：

```python
import anyio

limiter = anyio.to_thread.current_default_thread_limiter()
print(limiter.total_tokens)

limiter.total_tokens = 100
```

需要注意，40 个 thread tokens **不等于 FastAPI 单实例最多只能支持 40 个并发请求**，它限制的是需要进入该线程池执行的并发任务数量，`async` 请求不受这个限制。

如果同步接口中存在大量长时间 I/O，例如：

```python
def chat():
    return requests.post(llm_url)
```

一次大模型请求可能持续数秒甚至数分钟，对应线程会一直被占用。高并发情况下线程池容易成为瓶颈，因此这类 I/O 密集型服务更适合使用 async。

数据库连接池大小也不应该简单按照线程池大小设置，而应该结合 Worker 数量、实例数量、事务时间以及数据库最大连接数综合设计。尤其不要在长时间 LLM Streaming 期间一直持有数据库连接。

## 2. Async API：通过 Event Loop 调度

FastAPI 的异步 API：

```python
@router.get("/users")
async def get_users():
    return await query_users()
```

由 Uvicorn Worker 的 Event Loop 调度执行。

当代码遇到真正的异步 I/O：

```python
result = await http_client.get(url)
```

Coroutine 会在等待 I/O 时让出 Event Loop：

```text
Request A
    │
    ├── 执行代码
    │
    └── await 网络 I/O
              │
              └─────── 让出 Event Loop
                          │
                          ├── Request B
                          ├── Request C
                          └── healthz
    网络数据到达
    │
    └── Event Loop 恢复 Request A
```

因此一个 Event Loop 线程可以同时管理大量处于 I/O 等待状态的请求，非常适合 HTTP、数据库、Redis、大模型调用、SSE 等 I/O 密集型场景。

## 3. Async 调用链中不能混入阻塞式 Sync I/O

这是 FastAPI 开发中特别需要注意的问题。

Python 允许：

```python
async def chat():
    result = requests.post(url)
    return result.json()
```

但 `async def` **不会自动把内部的同步函数变成异步函数**。

`requests.post()` 仍然是 blocking I/O：

```text
Event Loop
    │
    ▼
async chat()
    │
    ▼
requests.post()
    │
    │ 同步等待网络
    │
    │ Event Loop 被占用
    ▼
return
```

如果它阻塞 10 秒，这个 Worker 的 Event Loop 可能 10 秒无法正常调度其他 Coroutine，从而导致其他 API 甚至 healthz 超时。

因此 async API 应尽量保证整个 I/O 调用链都是 async：

| 场景 | Sync | Async |
|---|---|---|
| HTTP | `requests` | `httpx.AsyncClient` |
| SQLAlchemy | `Session` | `AsyncSession` |
| Redis | 同步客户端 | `redis.asyncio` |
| sleep | `time.sleep()` | `await asyncio.sleep()` |
| LangChain | `invoke()` | `ainvoke()` |
| LangChain Streaming | `stream()` | `astream()` |

如果由于历史 SDK 等原因必须调用同步方法，可以显式放入线程池：

```python
from starlette.concurrency import run_in_threadpool

result = await run_in_threadpool(sync_function)
```

---

## 4. 一个容易忽略的特殊情况：`requests + stream=True`

下面这种代码特别容易让人误以为 `requests` 已经变成了异步请求：

```python
response = requests.post(
    llm_url,
    stream=True,
)

for chunk in response:
    yield chunk
```

实际测试时会发现：

> 虽然 `requests` 是同步库，但前端确实可以看到大模型内容持续流式返回。

这是正常现象，因为 **Streaming 和 Async 是两个不同的概念**。

### `stream=True` 做了什么？

`stream=True` 不是 HTTP 协议参数，而是 `requests` 的行为配置。

默认：

```python
response = requests.post(url)
```

`requests` 会读取完整 Response Body 后再返回：

```text
收到 Response Headers
        ↓
读取整个 Body
        ↓
读取完成
        ↓
requests.post() 返回
```

而：

```python
response = requests.post(url, stream=True)
```

收到 Response Headers 后就可以返回 `Response` 对象，Response Body 后续再逐步读取：

```text
收到 Response Headers
        ↓
requests.post() 返回
        ↓
读取一块 Body
        ↓
yield chunk
        ↓
读取下一块 Body
        ↓
yield chunk
        ↓
...
```

所以同步 HTTP Client 完全可以实现流式响应。

### 问题在哪里？

问题在于：

```python
for chunk in response:
    yield chunk
```

仍然是**同步读取**。

可以粗略理解成：

```python
while True:
    chunk = socket.recv(...)
    yield chunk
```

如果 Socket Buffer 中已经有数据：

```text
next() → 很快得到 chunk → yield
```

所以多个请求看起来甚至可以同时正常 Streaming。

但如果某一次上游迟迟没有发送数据：

```text
next()
  │
  └── socket.recv()
          │
          │ 等待上游数据
          │ 5 秒
          ▼
       chunk
```

这 5 秒仍然是同步阻塞。

如果这段代码运行在 `async def` 的 Event Loop 线程中，就可能导致同一 Worker 上的其他 Coroutine 无法执行。

因此下面这种代码尤其隐蔽：

```python
@router.post("/chat")
async def chat():

    response = requests.post(
        llm_url,
        stream=True,
    )

    async def generator():
        for chunk in response:
            yield chunk

    return StreamingResponse(
        generator(),
        media_type="text/event-stream",
    )
```

外层虽然都是 `async`，但真正读取上游网络数据的：

```python
for chunk in response:
```

仍然是同步阻塞 I/O。

### 正确做法

对于大模型 Streaming，应该使用真正的异步 HTTP Client，例如 `httpx.AsyncClient`：

```python
async with httpx.AsyncClient() as client:
    async with client.stream("POST", llm_url, json=data) as response:
        async for chunk in response.aiter_bytes():
            yield chunk
```

此时如果上游暂时没有数据：

```text
await 下一块数据
       │
       ├── 让出 Event Loop
       │
       ├── 处理其他请求
       └── 处理 healthz

数据到达
       │
       └── 恢复当前 Coroutine
```

因此需要特别记住：

> **`requests(stream=True)` = Sync + Streaming，而不是 Async + Streaming。**

它确实可以产生流式返回效果，但等待下一块数据时仍然可能阻塞当前线程。在 FastAPI `async def` 中处理大模型等长时间 Streaming 请求，应优先使用 `httpx.AsyncClient` 等真正的异步 HTTP Client。

最后，要注意 fastapi 的 middleware 函数必须是 async 的，所以不能有任何同步方法，否则同样会导致应用卡顿明显。

# Depends 注解
fastapi 支持 Depends 依赖注入，常用于数据库连接或者认证信息注入，请求参数的合法性校验（id 是否存在等），可以有效简化公共逻辑。
```python
async def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(*, request: Request, db:Depends(get_db)):
    ...

# 或者定义一个单独类型注解
SessionDep = Annotated[SessionLocal, Depends(get_db)]
@app.get("/items/")
async def read_items(*, request: Request, db: SessionDep):
    ...
```
注意，get_db 是属于 api 内部的预处理逻辑，同样需要区分 sync 和 async，如果是 sync，同样会阻塞 async api。
更多[依赖注入文档](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/#always-raise-in-dependencies-with-yield-and-except)

同样这里需要注意 db session 底层会在查询的时候占用 db connection，如果 connection 在 llm 请求前后没有及时返回，那么也容易出现数据库 connection 耗尽的错误。所以 fastapi 的方法其实不是好的方法，简单的处理方式是及时 close，同时要注意 close 后不要继续使用 orm 对象，及时转换成 dto 使用。
```python
@app.get("/items")
async def read_items(db=Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    user_name = user.name
    # 1. 彻底关闭当前连接，把 Connection 完美归还给 Pool 注意 不要再用 user 了。
    db.close() 
    # 2. 此时连接池是安全的，LLM 慢慢等 5 秒，不占用任何数据库资源
    result = await call_llm(user_name)
    # 3. LLM 结束后，直接再次执行 SQL！
    # SQLAlchemy 发现 session 是关闭的，会自动去 Pool 里申请一个【新连接】开新事务
    new_item = db.query(Item).filter(Item.user_id == user_id).first()
    
    return {"result": result, "item": new_item}
```


# uvicorn 的 worker 设置
- uvicorn 的 worker 和 K8S 的 pod 实例数没有任何区别，所以在 K8S 环境中，无需设置多个 worker，增加 pod 即可。
- 是否应该增加 starlette 的线程池呢？如果可以增加 pod 数量，则一般情况不需要，假设 8 个 pod，那么就是 320 个线程，大部分情况下都是足够使用的，且也应该考虑数据库连接池数量的限制，很多 DBA 可能会限制连接数在 500 或者 1000 以内，所以遇到性能问题时更多应该从代码或者逻辑优化，或者将阻塞 api 转换成 async api。 

# 协程的 context var 问题
需要注意 fastapi 的中间件和 sync api 所在不是一个协程，中间件是异步的（must be async def or async def __call__)，而 sync api 会放在另一个 thread 内运行，无法看到中间件的信息。
所以如果 sync api 想要访问一些公共信息，可以考虑 Depends 依赖注入。或者将业务逻辑包装然后在前面增加一个 context_copy 逻辑，如下所示。

```python
# helper: run a sync function but preserving current ContextVars
def run_in_sync_with_context(func, *args, **kwargs):
    ctx = copy_context()
    return anyio.to_thread.run_sync(lambda: ctx.run(func, *args, **kwargs))

@app.get("/safe-sync")
async def safe_sync_endpoint():
    def logic():
        return {"user": user_var.get()}

    return await run_in_sync_with_context(logic)
```


# 其他
- 使用 pydantic 校验字段，使用 BaseSettings 读取环境变量配置并分组（redis 的配置，es 的配置）
- 用户登录信息可以放在 request.state 对象上。 