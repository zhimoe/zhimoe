+++
title = 'Python @contextmanager 的使用'
date = '2023-11-05T21:45:24+08:00'
categories = ['编程']
tags = ['code','python']
toc = true
+++

日常开发中使用这个注解的情况比较少，今天发现其实有一个临时环境变量设置的使用方式。
```python
with set_environ(env_name, value):
    # 使用后自动清除 env_name 变量
    ...
# env_name 已失效

```

<!--more-->

### 可关闭资源的管理

```python
with self._operation_context() as conn:
    ...
    # 使用数据库 connection
```

```python
@contextlib.contextmanager
def _operation_context(self) -> Generator[Connection, None, None]:
    """Return a context that optimizes for multiple operations on a single
    transaction.

    This essentially allows connect()/close() to be called if we detected
    that we're against an :class:`_engine.Engine` and not a
    :class:`_engine.Connection`.

    """
    conn: Connection
    if self._op_context_requires_connect:
        conn = self.bind.connect()  # type: ignore[union-attr]
    else:
        conn = self.bind  # type: ignore[assignment]
    try:
        yield conn
    finally:
        if self._op_context_requires_connect:
            conn.close()
```

### 临时环境变量的设置与清除

```python
with set_environ('LOCAL_PROXY', value):
    # LOCAL_PROXY  now is value
    ...
# 环境变量中 LOCAL_PROXY 已恢复（删除或者前值）

```

```python
@contextlib.contextmanager
def set_environ(env_name, value):
    """Set the environment variable 'env_name' to 'value'

    Save previous value, yield, and then restore the previous value stored in
    the environment variable 'env_name'.

    If 'value' is None, do nothing"""
    value_changed = value is not None
    old_value = os.environ.get(env_name)
    if value_changed:
        os.environ[env_name] = value
    try:
        yield # 这里无需 yield 具体值，只是利用副作用
    finally:
    # 需要恢复原来的变量
        if value_changed:
            if old_value is None:
                del os.environ[env_name] 
            else:
                os.environ[env_name] = old_value
```