+++
title = 'Fastapi 开发笔记'
date = '2025-09-14T14:51:25+08:00'
categories = ['编程']
tags = ['code','python', 'fastapi']
toc = true
+++

最近使用一个 fastapi 后端应用遇到一些性能问题，借助 GPT 和文档学习了一些框架底层知识，记录以便温习。

<!--more-->

## sync 和 async 的工作原理
fastapi 可以无缝支持 sync 和 async 两种风格的 api，但是选择哪种风格有很多需要考量。sync api 是通过[anyio 在线程池里面执行](https://github.com/Kludex/starlette/blob/9f16bf5c25e126200701f6e04330864f4a91a898/starlette/concurrency.py#L36-L42)，starlette(fastapi 底层 http web 组件）默认是开启 40 个线程池。所以单个实例理论最多支持 40 个并发，那么 mysql 的 engine 设置 max_overflow 必须是 0 或者 >40，否则可能会遇到数据库连接池不够用的情况。你可以使用下面的代码检查或者设置 fastapi/starlette 的线程池大小。
```python
limiter = anyio.to_thread.current_default_thread_limiter()
limiter.total_tokens = 100
```
目前很多大模型 agent 应用使用 python 开发，大模型请求时耗时往往非常高，可能超过 3~5s，那么这个线程以及数据库连接会被长时间占用，高峰期很容易遇到整个应用无法接受新的请求的情况。这时候就需要通过 async 提升系统吞吐量。
fastapi 的 async api 是通过一个 event loop 执行的（uvicorn 的底层 event loop 和 nodejs 相同的——libuv），在遇到 io 场景可以让出 event loop 所在 thread 执行其他任务。需要注意的是，语法上同步方法内无法执行 async 方法，所以不会出错，但是 async 方法内可以执行同步方法（当成非阻塞代码看待），而一个 uvicorn worker 只有一个 event loop，所以一旦某个 async 方法链上某个同步方法阻塞等待，则整个应用都无法接收新的请求，看上去应用就无任何响应，客户端连接超时，这会导致应用吞吐量受到极大影响，所以需要确保 async api 整个方法链全部是 async 的，例如 langchain 的 stream 和 invoke 方法都有对应的 async 版本：astream 和 ainvoke。sqlalchemy 1.4+ 以上支持 async，redis 需要使用 redis.asyncio 客户端，http 请求需要使用 httpx 的 AsyncClient（langchain 也是用这个）或者 aiohttp（不建议），不能使用 requests 库。

## Depends 注解
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

## uvicorn 的 worker 设置
- uvicorn 的 worker 和 K8S 的 pod 实例数没有任何区别，所以在 K8S 环境中，无需设置多个 worker，增加 pod 即可。
- 是否应该增加 starlette 的线程池呢？如果可以增加 pod 数量，则一般情况不需要，假设 8 个 pod，那么就是 320 个线程，大部分情况下都是足够使用的，且也应该考虑数据库连接池数量的限制，很多 DBA 可能会限制连接数在 500 或者 1000 以内，所以遇到性能问题时更多应该从代码或者逻辑优化，或者将阻塞 api 转换成 async api。 

## 协程的 context var 问题
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


## 其他
- 使用 pydantic 校验字段，使用 BaseSettings 读取环境变量配置并分组（redis 的配置，es 的配置）
- 用户登录信息可以放在 request.state 对象上。 