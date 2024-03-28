+++
title = "Python Tips for Impatient Dev"
date = "2022-01-31T21:45:45+08:00"
categories = ["编程", ]
tags = ["code", "python", ]
toc = "true"
+++

## Python tricks

### f-string 的妙用

py3.6 开始，推荐使用 f-string，不要使用` %s`或者 `"".format()`.如果接收用户输入，使用 Template 做安全校验。
在 python f-string 中可以通过变量或者表达式后面加=实现打印变量名或者表达式：

```python
print(f'{v=}') # 等价 print(f'v={v}')
print(f'{(len(arr),v)=}') 
```
参考：调式时`icecream`比`print` `log`更好。

<!--more-->
### python 注解

Python 对注解所做的唯一的事情，就是将它们存储在函数的 __annotations__ 属性中
```python
>>> def foo(a: 'x', b: 5 + 6, c: list) -> max(2, 9): ...
>>> foo.__annotations__
{'a': 'x', 'b': 11, 'c': <class 'list'>, 'return': 9}
```

### 单例模式

参考[creating-a-singleton-in-python](https://stackoverflow.com/questions/6760685/creating-a-singleton-in-python)
,建议是 metaclass 模式，里面很多回答是错误的。

```python
# 下面其实不是单例，因为可以再次调用 Foo(), 只是一个全局变量
class Foo(object):
     pass

some_global_variable = Foo()
# 利用缓存可以实现单例模式，前提是__init__没有参数，因为 cache 是根据参数列表生成缓存的 key
from functools import cache

@cache
class CustomClass(object):

    def __init__(self): #这里不能有参数，因为会导致 key 不一样
        self.instance =  ... 


# 另外一种方法是 metaclass, 和上面的@cache 原理近似，只不过一个是外部缓存，一个是 mataclass 内缓存，同理，__init__不能有参数
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
        
class ESClient(metaclass=Singleton):
    pass

assert id(ESClient()) == id(ESClient())


# 一种取巧的方式
def _singleton(c): c()

@_singleton
class ESClient(object):

    def __init__(self):
        self.client =  Elasticsearch([{'host': es_config.host, 'port': es_config.port}])
    def get_instance(self):
        """get es client instance"""
        return self.client
# 原理是通过装饰器改写后得到的 wat 实际是一个对象，不是一个 class:
wat = _singleton(wat) # 这样就阻止你通过 another_instance = wat() 获取新实例
# 当然你可以通过 another_instance = wat.__class__() 创建新的实例
# 当然这种方式也要求__init__不能有其他参数，对于无参的单例模式其实挺巧妙

# 补充阅读 python es client 是否应该全局变量：
# https://elasticsearch-py.readthedocs.io/en/7.x/#thread-safety
```

### 枚举类 Enum 略去 value 方法

假设你想要获得下面`Color`的`#000`, 需要使用`Color.WHITE.value`。但是可以通过`StrEnum`省去这个`.value`

```python
class Color(Enum):
    WHITE = "#000"

# tips
from enum import StrEnum

class Directions(StrEnum):
    NORTH = 'north',    
    SOUTH = 'south',     # notice the trailing comma, it's ok and recommend

print(Directions.NORTH) # no need .value
# 缺点：StrEnum 成员不能有 int 类型
```

### 使用 with 管理需要关闭的资源，无需多个 with

```python
with open(file_path, 'w') as file, get_connection(config) as conn:
    file.write('Hello, World!')
```

### 简化 if 的一些技巧

```python
if variable == a or variable == b:
    res = do_something(variable)

# better  
if variable in {a, b}:
    res = do_something(variable)

if 'a' in v or 'b' in v or 'c' in v:
  pass

# better
if any(char in v for char in ('a', 'b', 'c')):
    pass


if b > 10:
    a = 0
else:
    a = 5
# better 
a = 0 if b > 10 else 5

if data:
    lst = data
else:
    lst = [0, 0, 0]
    
# better, only use this when data type is collection, 
# cuz if data is int, then data=0 will make lst = [0,0,0]
lst = data or [0, 0, 0]


```

### Instead of asking for permission, ask for forgiveness

Python 的异常比较轻量，所以一般推荐的写法是与其提前判断条件是否满足，不如在 try catch 中处理异常

```python
try:
    with open(filename, 'w') as file:
        file.write('Hello, World!')

except FileNotFoundError:
    print('File does not exist.')

except PermissionError:
    print('You dont have write permission.')

except OSError as exc:
    print(f'An OSError has occurred:\n{exc}')
```

### isinstance

isinstance 可以一次判断多个 Class 类型：

```python
# no need or conditions
isinstance(foo, (Class1, Class2, ...))
```

### 海象运算符 (walrus operator)

python 3.8 引入海象运算符，解决一个场景：获取一个值，检查它是否为非零，然后使用它。在 python3.8 之前需要三行：

```python
count = fresh_fruit.get('lemon', 0)
if count:
    make_lemonade(count)
else:
    out_of_stock()
```

上面的代码会引入一个看上去非常重要的变量`count`，但实际情况并非如此，count 只在一个分支里面使用到。使用海象运算符可以解决这个问题。

```python
if count := fresh_fruit.get('lemon', 0):
    make_lemonade(count)
else:
    out_of_stock()
```

虽然只是减少了一行代码，但是可读性提升了巨大，可以清楚地知道 count 只有在 if 成立时才使用到。甚至还可以在 if 中判断

```python
if (count := fresh_fruit.get('apple', 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()
```

另一个常见的场景是 do-while，例如处理一个分页请求，直到请求返回为 None，由于 python 不支持 do-while 语法，一般有两种写法：

```python
page_data = get_page()
while page_data:
    process_page(page_data)
    page_data = get_page()

# 方法 2 loop-and-a-half
while True:
    page_data = get_page()
    if not page_data:
        break
    process_page(page_data)

```

海象运算符完美解决了这个问题：

```python
while page_data := get_page():
    process_page(page_data)
```

读写文件时也经常遇到这个场景：

```python
fp = open("test.txt", "r")
while line := fp.readline():
    print(line.strip())
```

### 多进程的使用

```python
from multiprocessing import Pool

requests = [req1, req2]

with Pool as p:
    results = p.map(process_request, requests)
```

### python 的 dict 中关于 equal 和 hash 计算方式会有意外的效果

```python
['no', 'yes'][True] # output?
{True: 'yes', 1: 'no', 1.0: 'maybe'} # output?
```

> “布尔类型是整数类型的子类型，布尔值在几乎所有环境中的行为都类似于值 0 和 1，但在转换为字符串时，分别得到的是字符串 False
> 或 True.”  
> -- The Standard Type Hierarchy

由于 True,1, 1.0 的__eq__和__hash__都一样，所以出现了神奇的结果。

### `(1) != (1,)` 第一个是 int，第二个是 tuple

### 避免可变的默认参数，例如：

```python
def fun(count=[]):
  count.append(2) #这里 count 两次调用如果都使用默认参数的话，则是同一个数组，非常危险！
  return count
fun()   #[2]
fun()   #[2,2]
```

同理，需要避免在 tuple 中放入可变类型元素，例如 list。

```text
flat seq: str, bytes, bytesarray, memeoryview, array.array
container seq: list, tuple, collections.deque

immutable: bytes, str, tuple
mutable: list dict set bytesarray
```

## staticmethod classmethod

> staticmethod 和 classmethod 都可以通过 Cls.m() 或 instance.m() 方式访问，都可以被继承，都可以访问全局变量。区别是
> classmethod 访问的 class 变量信息会自动在 Derive 子类中改变，而 staticmethod 因为缺少第一个 cls 参数，所以访问的全局变量始终是父类的变量。
> staticmethod 可以理解为 Java 的 StringUtils 类，只是和 Cls 放在一起方便代码阅读和组织。
> classmethod 则是可以通过 cls 参数访问到当前类信息的。

## 容器方法

### 合并字典

```python
d1.update(d2) # 遍历 d2，更新到 d1
d = dict(**profile, **ext_info) #解构重新创建 dict，右边的优先级高
d = dict(profile.items() | ext_info.items()) #同理
d = d1 | d2 # 新语法
{k: v for d in [profile, ext_info] for k, v in d.items()} # 推导式
```

py3.6 开始 dict 默认插入有序，如果只是希望插入有序则无需使用 OrderDict

### dict get and pop

```python
d.get(k,default) 
d.setdefault(k,default) 
d.pop(k) #删除不存在的键时，使用 del d[k] 有异常抛出，pop 则不会
```

### 自定义 dict

> 自定义自己的 dict 不能继承 dict，而是 collections.abc.MutableMapping, 因为自带的 list 和 dict 有一些特殊行为无法覆盖

### sort dict by key:

`print(sorted(dic, key=dic.get)) #output: key in asc order`

### 带索引遍历

`for idx, item in enumerate(x):`

### 浅复制

list(l) 等方式构建的 list dict set 属于浅复制，如果容器的元素还是容器，那么元素属于引用。
深度复制需要使用 copy module

```python
import copy
xs = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
zs = copy.deepcopy(xs)
# 对象同样可以使用 copy，还有__copy__等魔法方法可以探索
```   

### 使用 namedtuple

namedtuple 可以让代码更容易阅读，当然现在可以是所有 dataclass，特性更丰富

```python
from collections import namedtuple
class Car:
  """you do not modify the name and date attributes """

  def __init__(self, name, date):
      self._name = name
      self._date = date


# use namedtuple
Car = namedtuple('Car', 'name date')  # 注意，这里多个属性可以一次性传入，使用空格分割
# 然后你可以将 Car 作为 data class 使用

#还有一个 type hint 的版本
from typing import NamedTuple
```

### list vs array

list 的元素可以不同，更为紧凑的单一类型是 array 类型。
list 通过 append pop 两个方法可以实现 stack 效果。

### Counter

`Counter(string).most_common(3)`

### deque

```python
from collections import deque
names = deque(['raymond', 'rachel', 'matthew', 'roger',
             'betty', 'melissa', 'judith', 'charlie'])
names.popleft()
names.appendleft('mark')
``` 

### 一些不常用的容器

CountedObject,ChainMap,MappingProxyType,frozenset,defaultdict

### 在遍历中修改 list

使用 for i in range(len(a)) 或者 for i, v in enumerate(a) 都是危险的。

```python
# 方法 1 将 list 拷贝一下，遍历新数组的过程中，修改原 list:
num_list = [1, 2, 3, 4, 5]
print(num_list)
for item in num_list[:]:  # 这里 list[:] 是对原数组的拷贝，trick!!!
    if item == 2: 
        num_list.remove(item)
    else:
        print(item)
print(num_list)

# 方法 2 如果数组很大，那应该使用倒序遍历：
for i in range(len(num_list)-1, -1, -1):
  if num_list[i] == 2:
      num_list.pop(i)
  else:
      print(num_list[i])
      
# 方法 3 还是使用 for 推导式
[4 if x==3 else x for x in num_list]
```

打印容器可以使用 pprint.pprint

## str 和 bytes

### bytes 和 bytesarray

bytes 是不可变的数组，每个元素必须在 0～255 之间。
bytearray 是可变的，可以修改，增加，删除元素。
转换 `bytes(ba)`

### 多行 string

```python
my_very_big_string = (
  "For a long time I used to go to bed early. Sometimes, "
  "when I had put out my candle, my eyes would close so quickly "
  "that I had not even time to say “I’m going to sleep.”"
)
```

多行文本去除缩进可以使用 `textwrap.deden("""\your text""")`

### str 重复 n 次

`print(s * n);`

### int/string intern

小整数池是[-5,256], string 也有 string intern

### str.partition/str.translate

有时使用 str.partition 方法拆分 str 或者 str.translate 批量 replace 子串更方便

### NamedTuple, typing.NamedTuple, dataclass

```python
from collections import namedtuple
Coordinate = namedtuple('Coordinate', 'lat long')
issubclass(Coordinate, tuple) # True
moscow = Coordinate(55.756, 37.617)
moscow == Coordinate(lat=55.756, long=37.617)  # True __eq__

# typing.NamedTyple also is subclass tuple, not new type 
import typing
Coordinate = typing.NamedTuple('Coordinate', [('lat', float), ('long', float)])
# or with type: Coordinate = typing.NamedTuple('Coordinate', lat=float, long=float)
issubclass(Coordinate, tuple) # True
Coordinate.__annotations__
# {'lat': <class 'float'>, 'long': <class 'float'>}”
issubclass(Coordinate, typing.NamedTuple)
# False
issubclass(Coordinate, tuple)
# True

### class style
from typing import NamedTuple

class Coordinate(NamedTuple):

    lat: float
    long: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.long >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.long):.1f}°{we}'

### dataclass
from dataclasses import dataclass

@dataclass(frozen=True) # frozen=True: exception if re-assign fields after instance initialized
class Coordinate:

    lat: float
    long: float

    def __str__(self):
        ...

dataclasses.asdict(c)

```

## 重要标准库
```python
functools contextlib atexit pathlib collections itertools
inspect: e.g. inspect the function signature
bisect: 二分查找
```

### `pathlib` vs `os.path`
use `pathlib` over `os.path`. 后者方法不全。

```shell
def project_root() -> Path:
    return Path(__file__).parent.parent.parent


def file_abspath(relative_path: str) -> str:
    """get file absolute path from project root"""
    return (project_root() / relative_path).as_posix()
```

### os vs sys

> os module is for system, sys this module provides access to some variables used or maintained by
> the interpreter and to functions that interact strongly with the interpreter.

### 迭代器

- 迭代器：`__iter__` 和 `__next__` 两个方法
- 可迭代对象：`__iter__` 方法
- 如果希望可迭代对象可以重复使用，应该在 `__iter__` 每次返回新的迭代器对象。
- yield 生成器也是迭代器。itertools 简化循环，例如 product

## pythonic 的代码

- 使用 for 推导式，不要 for..in 遍历，也少用 map,filter
- 多使用 destructing，这点在 Java/Go 都不支持，可以在方法内部省很多代码

  ```python
    long_list = [x for x in range(100)]
    a, b, *c, d, e, f = long_list #e==98 f=99
  ```
- 使用`if x is None`,而不是 `if x == None `

- 异常处理
  清楚 except 和 assert 场合，logging.error('xxxxxxx', exc_info=True).
  自定义异常必须重写__init__() 和 __str__()

- use `reversed(lis)` over `lis[::-1]`
- @property make a method to class's property. can not use () when access the method cuz it is prop.
- use assert protect your code. 不要使用 assert 检查数据，断言可能被全局禁用，导致数据检查（或者更恐怖的权限检查）被跳过
- @functools.wraps(func)
- `__var` 在 class 环境中会被改写
- 使用 abc 模块可以避免抽象类只有在未实现方法被调用时才抛出 NotImplementedError
- 理解 python 的 dunder 方法，可以写出超级简洁的方法：
    ```python
    import collections
    
    Card = collections.namedtuple('Card', ['rank', 'suit'])
    class FrenchDeck:
        ranks = [str(n) for n in range(2, 11)] + list('JQKA')
        suits = 'spades diamonds clubs hearts'.split()
    
        def __init__(self):
            self._cards = [Card(rank, suit) for suit in self.suits
                                            for rank in self.ranks]
    
        def __len__(self):
            return len(self._cards)
    
        def __getitem__(self, position):
            return self._cards[position]
    
    deck = new FrenchDeck()
    from random import choice
    choice(deck)
    # Card(rank='3', suit='hearts')
    ```
  上面的例子我们使用了__len__方法和__getitem__方法，好处就是你可以使用 python 的 len(deck),deck[1]
  这种语法，也就是说`FrenchDeck`几乎就是一个容器类型，还可以使用 for 遍历。
  更多查看<fluent python 2nd>.

## 其他

- 数据校验`pydantic`
- 日志 `loguru`
- 异常现场`stackprinter`，打印上下文`icecream`
- 命令行`click` 或者`defopt`
- 时间处理 `arrow`
- 查看字节码`dis`
- 使用 `help(),dir()` 获取信息
- python 中的每个函数都有`__code__`属性，包含字节码信息
- 使用 dis 模块的 dis 函数可以查看更容易阅读的汇编 (dis == disassembler)
- sys.getsizeof(x) 获取对象大小
- `...`和`pass`几乎等效的，这是一个 ellipsis type 的单例。
- 无限大 float('inf') float('-inf')
- `==`会被`__eq__`方法改变，判断是否 None 时应该使用 is 判断 id 是否一致
- python 的 try 可以配合 else:当没有任何异常或者 try 里面没有 return break，才执行 else 部分。这个和 finally 有很重要的不同
- 使用 with 需要实现`__enter__` `__exit__`两个方法
- with 语句可以同时打开多个文件，不要嵌套 with，更多功能查看`contextlib`
- 不要自己手动做数据校验，使用`pydantic`这个库
- 不要使用 assert 校验参数合法性，因为可以通过-O 参数跳过
- 代码执行可视化 https://pythontutor.com/
- .pyi 文件 存根文件，i 可以理解成 interface。在 PyCharm 中，如果某一行的左边有 * 号标识，则说明这一行（可以是类、属性或函数）在存根文件中有定义，你可以点击 * 号跳转到该文件对应的存根文件，通常是存放在 Python 库文件的 Typeshed Stubs 目录中，文件名以 .pyi 后缀结尾。

## 参考资料

[Python 工匠系列](https://github.com/piglei/one-python-craftsman)
[RealPython 学习路径](https://realpython.com/learning-paths/)
[一份非常详尽的 Python 小抄](https://github.com/gto76/python-cheatsheet)
[Fluent Python 2nd](https://book.douban.com/subject/34990079/)