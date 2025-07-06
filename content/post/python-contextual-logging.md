+++
title = 'Python 结构化日志设置[翻译]'
date = '2025-07-06T16:27:38+08:00'
categories = ['翻译']
tags = ['code','python']
toc = true
+++

在日常开发中，python 项目常用的 log 方法就是 `logger.info(f"xxx failed, {user=}, {filename=}")`, 这种文本日志具备一定的信息，在大多数情况下是够用的，但是在大型、可观测性要求较高的情况，还需要更多的上下文信息才能定位问题，例如，这个日志属于哪个请求，日志里面充满了相同用户的不同文件名称的日志，你也不知道上一步某个关键信息（缺乏关联）和这个日志属于同一个请求。这时候结构化日志就很有必要了。

<!--more-->

## 基础
每个 python log 方法都会创建一个`LogRecord`, 这个 record 会经过`Logger` `Filter` `Handler` `Formatter`四个组件处理。具体可以参考[logging how-to](https://docs.python.org/3/howto/logging.html#logging-howto)

在非脚本项目中，应该在 main.py 中设置 logging 基本设置。 
```py
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] [%(name)s] %(message)s"
)
```
然后在每个需要日志的文件中`logger = logging.getLogger(__name__)`。 

## 添加全局上下文
```python
import logging
import socket
import os

class ContextFilter(logging.Filter):
    def __init__(self, name=''):
        super().__init__(name)
        self.hostname = socket.gethostname()
        self.process_id = os.getpid()

    def filter(self, record):
        record.hostname = self.hostname
        record.process_id = self.process_id
        return True

# main.py
root_logger = logging.getLogger()
root_logger.addFilter(ContextFilter())
```

## 添加动态上下文
首先利用`contextvars`和`contextmanager`实现请求范围（scope）的上下文记录器。
```py
import logging
import socket
import os
import contextvars
from contextlib import contextmanager

_log_context = contextvars.ContextVar("log_context", default={})


class ContextFilter(logging.Filter):
    def __init__(self, name=""):
        super().__init__(name)
        self.hostname = socket.gethostname()
        self.process_id = os.getpid()

    def filter(self, record):
        record.hostname = self.hostname
        record.process_id = self.process_id

        context = _log_context.get()
        for key, value in context.items():
            setattr(record, key, value)

        return True


@contextmanager
def add_to_log_context(**kwargs):
    current_context = _log_context.get()
    new_context = {**current_context, **kwargs}

    # Set the new context and get the token for restoration
    token = _log_context.set(new_context)

    try:
        yield
    finally:
        _log_context.reset(token)
```
在 fastapi 可以借助 http 中间件在每个请求中在 log 上下文记录器注入需要的信息，例如请求 id, 在每个 api service 方法中同理，可以注入 user_id。 
利用 contextmanager 可以保证 add_to_log_context 只对当前内部 logger.xxx 方法有有效，不会污染其他 log 方法。
```py
# [...]
from log_filters import log_context, add_to_log_context
from fastapi import FastAPI, Request

# [...]

app = FastAPI()

@app.middleware("http")
async def add_request_context(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))

    with add_to_log_context(request_id=request_id):
        response = await call_next(request)
        return response


@app.get("/users/{user_id}")
async def get_user(user_id: str):
    with add_to_log_context(user_id=user_id):
        logger.info("User profile request received.")

    return {"user_id": user_id}
```


[原文：logging-in-python](https://www.dash0.com/guides/logging-in-python)
[python doc: logging-cookbook](https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook)