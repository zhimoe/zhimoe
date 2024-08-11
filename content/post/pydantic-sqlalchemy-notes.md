+++
title = 'Pydantic Sqlalchemy 项目开发总结'
date = '2024-08-10T22:32:37+08:00'
categories = ['编程']
tags = ['code','python']
toc = true
+++

最近使用 fastapi pydantic sqlalchemy 写了一个两千行左右的 API 项目，这是第一次面向 class 写 python 项目，和以前使用 requests、pandas 写数据处理脚本有很大区别，特别是 sqlalchemy 第一次使用，看文档内容非常多，在和 pydantic 的 schema 相互转换遇到很多问题，所以做一个笔记。

<!--more-->

## .venv 在项目 root dir 中必要设置
If you store the .venv inside the project, right-click it in the project tree -> Mark directory as -> Excluded.
Or go to Settings -> Editor -> File Types -> Ignored Files and Folders, and ignore `.venv` folder globally.
否则 pycharm 会将.venv 看成 project file，每次重构时都会检索一遍。

## Pydantic 技巧
### 自定义 datetime 字段 json string 格式

可以通过`@field_serializer`注解自定义 datetime 字段在 json 序列化时的格式，默认格式是 datetime(X,X,X) 的 str，不适合给前端展示。
```python
from datetime import datetime
from pydantic import BaseModel, field_serializer

def dt_format(dt: datetime) -> str:
    return dt.strftime('%Y-%m-%d %H:%M:%S') if dt else dt

class Message(BaseModel):
    """对话消息"""
    message_id: int
    conversation_id: str
    user_id: str
    user_name: str | None
    user_type: str | None  # human or ai
    created_at: datetime | None = None

    @field_serializer(*['created_at'])
    def dt_format(self, dt: datetime):
        return dt_format(dt)
```

### model 和 dict 相互转换
- 获取 model 的 fields name: `msg.model_fields()`
- `m.model_dump_json` vs `m.model_dump`区别
```
d = {"user_id": "zhi", "born_at": "1990-01-01 00:00:01"}
u = User(**d)
print(repr(u.model_dump()))
print(repr(u.model_dump_json()))
#output:
#> {'user_id': 'zhi', 'born_at': '1990-01-01 00:00:01'}
#> '{"user_id":"zhi","born_at":"1990-01-01 00:00:01"}'

```
- `m.dict` return raw value of super-model field, like:
```json
{'banana': 3.14, 'foo': 'hello', 'bar': BarModel(whatever=123)}
```
- field 的校验
```python
from pydantic import Field

class BookModel:
    name: str = Field(..., min_length=3, max_length=50, alias="book_name")
    publisher: Publisher
    price: float = Field(..., gt=0)
    isbn: str = Field(..., regex="^(?=(?:\D*\d){10}(?:(?:\D*\d){3})?$)[\d-]+$")
    published_date: datetime.date = Field(..., gt=datetime.date(1800, 1, 1))
```

[Pydantic Tips & Tricks](https://www.bhavaniravi.com/python/advanced-python/pydantic-tips-tricks)
[Advanced Pydantic Usage Guide](https://gist.github.com/shiningflash/f17eabef18b38a70a38fb510130be58b)