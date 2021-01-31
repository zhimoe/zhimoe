---
title: "Py Tips for Impatient Dev"
date: "2021-01-31T21:45:45+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - python
---

python在看似简单的语法中，有很多tricks。

## tricks
1. python的dict中关于equal和hash计算方式会有意外的效果
```python
['no', 'yes'][True] # output?
{True: 'yes', 1: 'no', 1.0: 'maybe'} # output?
```
由于True，1， 1.0的__eq__和__hash__都一样，所以出现了神奇的结果。
2. `(1) != (1,)`
3. 避免可变的默认参数, 例如:
   ```python
   def fun(count=[]):
       count.append(2) #这里count两次调用如果都使用默认参数的话,则是同一个数组,非常危险!
       return count
   fun()   #[2]
   fun()   #[2,2]
   ```
## staticmethod classmethod
```python
# staticmethod和classmethod都可以通过Cla.m()或instance.m()方式访问，都可以被继承，都可以访问全局变量。区别是
# classmethod访问的class变量信息会自动在Derive子类中改变，而staticmethod因为缺少第一个cls参数，所以访问的全局变量始终是父类的变量。
# staticmethod可以理解为Java的StringUtils类，只是和Cls放在一起方便代码阅读和组织。
# classmethod则是可以通过cls参数访问到当前类信息的。

```

## str和bytes
1. 使用f-string,不要使用 %s或者 "".format()。如果接收用户输入，使用Template做安全校验
2. 多行string
```python
my_very_big_string = (
    "For a long time I used to go to bed early. Sometimes, "
    "when I had put out my candle, my eyes would close so quickly "
    "that I had not even time to say “I’m going to sleep.”"
)
```
3. bytes 是不可变的数组，每个元素必须在0～255之间。
   bytearray是可变的，可以修改，增加，删除元素。
   转换 `bytes(ba)`

## pythonic的代码
1.  使用for推导式,不要for..in遍历, 也少用map，filter
2. 多使用destructing,这点在Java/Go都不支持,可以在方法内部省很多代码
```python
long_list = [x for x in range(100)]
a, b, *c, d, e, f = long_list #e==98 f=99
```
3. with语句可以同时打开多个文件,不要嵌套with，更多功能查看contextlib
4. 使用`if x is None`,而不是 `if x == None `
5. 避免可变的默认参数, 例如:
```python
def fun(count=[]):
    count.append(2) #这里count两次调用如果都使用默认参数的话,则是同一个数组,非常危险!
    return count
fun()   #[2]
fun()   #[2,2]
```
6.  新手多使用格式化,多看pycharm的提示,多在逻辑复杂处使用空行
7. 异常处理: 清楚except和assert场合,logging.error('xxxxxxx', exc_info=True).
   自定义异常必须重写__init__() 和 __str__()
8. 使用for i in range(len(a)):或者for i, v in enumerate(a):都是危险的。
```python
# 方法1
# 将list拷贝一下,遍历新数组的过程中,修改原list:
num_list = [1, 2, 3, 4, 5] 
print(num_list) 
for item in num_list[:]:  # 这里list[:]是对原数组的拷贝,trick!!!
    if item == 2: num_list.remove(item) 
    else: 
        print(item) 
print(num_list)

# 方法2
# 如果数组很大,那应该使用倒序遍历:
for i in range(len(num_list)-1, -1, -1):
    if num_list[i] == 2:
        num_list.pop(i)
    else:
        print(num_list[i])
		
# 方法3 还是使用for推导式
[4 if x==3 else x for x in num_list]
```
9. 打印容器可以使用pprint.pprint
10. `x if x < 10 else y`
11. use `reversed(lis)` over `lis[::-1]`
12. use pathlib over os.pathlib。 pathlib的方法不全。
13.  os module is for system, sys this module provides access to some variables used or maintained by the interpreter and to functions that interact strongly with the interpreter.
14.  @property make a method to class's property. can not use () when access the method cuz it is prop.
15. use assert protect your code. 不要使用assert检查数据， 断言可能被全局禁用，导致数据检查（或者更恐怖的权限检查）被跳过

## 容器
1. dict.get()可以指定missing-key-value
2. 判断key是否存在: `if k in d:`
3. 带索引遍历: `for idx, item in enumerate(x):`
4. list(l)等方式构建的list dict set属于浅复制，如果容器的元素还是容器，那么元素属于引用。
   深度复制需要使用copy module
```python
import copy
xs = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
zs = copy.deepcopy(xs)
# 对象同样可以使用copy,还有__copy__等魔法方法可以探索
```   
5. 使用namedtuple可以让代码更容易阅读
```python
from collections import namedtuple
class Car:
    """you do not modify the name and date attributes """

    def __init__(self, name, date):
        self._name = name
        self._date = date


# use namedtuple
Car = namedtuple('Car', 'name date')  # 注意，这里多个属性可以一次性传入，使用空格分割
# 然后你可以将Car作为data class使用

#还有一个type hint的版本
from typing import NamedTuple
```
6. 不常用的一些容器
```python
class CountedObject:
    num_instances = 0

    def init(self):
        self.__class__.num_instances += 1

from types import MappingProxyType #对外提供视图
from collections import defaultdict
from collections import ChainMap

frozenset
```
7. list的元素可以不同，更为紧凑的单一类型是array 类型
8. `Counter(string).most_common(3)`
9. sort map by key: print(sorted(dic, key=dic.get)), output: key in asc order
10. deque
```python
from collections import deque
names = deque(['raymond', 'rachel', 'matthew', 'roger',
               'betty', 'melissa', 'judith', 'charlie'])
names.popleft()
names.appendleft('mark')
``` 
11. 合并dict
```python
system = dict("path", "c:/", "env", "prd")
user = dict("path", "d:/", "env", "dev")
system.update(user) # 遍历user，更新到system
final = {**system, **user}  # 右边的优先级高

``` 
  
## dunder方法,函数,oop
1. @functools.wraps(func)
2. `__var` 在class环境中会被改写
3. 使用abc模块可以避免抽象类只有在未实现方法被调用时才抛出NotImplementedError

## 调试与编译
1. 使用help(),dir()获取信息
2. python中的每个函数都有__code__属性，包含字节码信息
3. 使用dis模块的dis函数可以查看更容易阅读的汇编(dis == disassembler)
4. sys.getsizeof(x)获取对象大小