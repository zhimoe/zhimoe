+++
title = 'Pydantic v2 Sqlalchemy v2 项目开发总结'
date = '2024-08-10T22:32:37+08:00'
categories = ['编程']
tags = ['code','python']
toc = true
+++

最近使用 fastapi pydantic(v2) sqlalchemy(v2)  写了一个两千行左右的 API 项目，这是第一次面向 class 写 python 项目，和以前使用 requests、pandas 写数据处理脚本有很大区别，特别是 sqlalchemy 第一次使用，看文档内容非常多，在和 pydantic 的 schema 相互转换遇到很多问题，所以做一个笔记。

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
> use .model_dump() instead of .dict() if you can use Pydantic v2.

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
### 校验 list
通过 TypeAdapter 可以校验一个 list：
```python
import pathlib
from typing import List

from pydantic import BaseModel, EmailStr, PositiveInt, TypeAdapter

class Person(BaseModel):
    name: str
    age: PositiveInt
    email: EmailStr

person_list_adapter = TypeAdapter(List[Person])  

json_string = pathlib.Path('people.json').read_text()
people = person_list_adapter.validate_json(json_string)
print(people)
#> [Person(name='John Doe', age=30, email='john@example.com'), Person(name='Jane Doe', age=25, email='jane@example.com')]

# or jsonl file with for expression
json_lines = pathlib.Path('people.jsonl').read_text().splitlines()
people_list = [Person.model_validate_json(line) for line in json_lines]
```

## Sqlalchemy 技巧
### 数据库连接池的超时设置:pool_recycle
容器云环境一般会有一个 http 连接自动断开时间，平台的连接断开应用是无法感知的，会导致你的应用中连接池出现失效，所以应用自身需要做连接池中连接主动失效或者使用前检测。在 sqlalchemy 中最简单的做法就是设置`pool_recycle`时间小于平台的断开时间。
```python
# 假设容器平台 300s 会断开未使用的 http 连接
engine = sqlalchemy.create_engin(db_url,pool_recycle=290)
```
### pydantic schema -> sqlalchemy model
从 pydantic schema 构建  model:

```python
# 将 pydantic object 导出成 dict 然后通过 **kwargs dict 结构传递给 sqlalchemy model 构建函数
db_orm = models.ItemOrm(**item.model_dump())
```
如果两边字段不一致那么需要使用 model_dump 方法的 include 参数
```python
table_cols = {col.name for col in MsgOrm.__table__.columns}
db_msg = MsgModel(**msg.model_dump(include=table_cols))
```

### sqlalchemy model -> pydantic model
Pydantic 也给了[一些建议](https://docs.pydantic.dev/latest/examples/orms/#sqlalchemy)

简单粗暴的做法是通过 dict 转换：
```python
msg = Msg(**msg_orm.__dict__)
```
官方的做法是通过[Arbitrary class instances](https://docs.pydantic.dev/2.8/concepts/models/#arbitrary-class-instances)
注意，在 v1 版本这个配置叫 orm_mode，实际上不准确，因为 pydantic 支持任意 class，不只是 orm 的 model。
```python
class Company(BaseModel):
    # 如果没有 from_attributes=True, 则 model_validate 只能接收 dict 或者 Company 实例，其他类型会报错
    model_config = ConfigDict(from_attributes=True)
    id: int
    public_key: Annotated[str, StringConstraints(max_length=20)]

# co_orm is sqlalchemy model instance
com = Company.model_validate(co_orm)

# 或者省略 model_config = ConfigDict(from_attributes=True)，而是在 model_validate 调用中设置
com1 = Company.model_validate(co_orm, from_attributes=True)
```
通过 dict 转换 和 `from_attributes=True` 没有区别，因为使用构建函数也是有 validate 动作的。 

[Pydantic Tips & Tricks](https://www.bhavaniravi.com/python/advanced-python/pydantic-tips-tricks)
[Advanced Pydantic Usage Guide](https://gist.github.com/shiningflash/f17eabef18b38a70a38fb510130be58b)