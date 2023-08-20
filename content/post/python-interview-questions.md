+++
title = "Python 4道笔试题"
date = "2020-06-04T21:31:01+08:00"
categories = [ "编程",]
tags = [ "code", "python",]
toc = "true"
+++

4道常见的python面试题和解答,以及一些python陷阱的链接.

## 问题

1. 题目1

```python 
    def change(v):
        v[1] = 4
        return v
        
        
    a = [1, 2, 3]
    print(change(a))
    print(a)    
```
<!--more-->

2. 题目2

```python
def append1(x=[]):
    x.append(1)
    return x
    
    
def now(n=time.time()):
    time.sleep(1)
    return n
    
print(append1(), append1()) #?
print(now(), now()) #?
    
```

3. 题目3

```python
def arr_multi():
    x = [[0] * 3] * 3
    x[0][0] = 42
    return x
    
print(arr_multi())
```

4. 题目4

```python
def fn_for():
    f = [lambda x: x * i for i in range(3)]
    print(f[0](1), f[1](1), f[2](1))
    
print(fn_for())
```
5. 题目5
```python
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
run, what happen next?

A. `t` becomes `(1, 2, [30, 40, 50, 60])`.
B. `TypeError` is raised with the message `'tuple' object does not support item assignment`.
C. Neither.
D. Both *A* and *B*.

```

## 解答

1. 

```python
[1, 4, 3]
[1, 4, 3]

# 就是简单的引用传递,但是很多人不自信,在选择题里面频频出错.
# python中所有的都是对象, id(obj)会返回地址. 
# 但是如果新建对象是short string,int [-5,256],不可变的空集合(empty tuples) 等情况不会真的创建新对象.

from copy import copy, deepcopy

arr1 = [1,2,3,[4,5,6]]
arr2 = copy(arr1) # shallow copy, new id, but elements in array is same id

id(arr1[0]) == id(arr2[0]) 

#deepcopy
arr3 = deepcopy(arr1) # elements id is new 

```

2.

```python
# 结果:
[1, 1] [1, 1]
1590544209.9695618 1590544209.9695618

# 不少人认为是: [1] [1, 1].其实还是没有深入理解引用的原理,
# 翻译一下就很好理解了:
y = append1()  # id(y) == id(x), y=[1]
y = append1()  # id(y) == id(x), y=[1,1]
print(y,y)


# 最好不要使用[]作为默认参数,使用下面的形式:
def my_func(working_list=None):
    if working_list is None: 
        working_list = []

    working_list.append("a")
    print(working_list)

# 或者
def fun(count=[]):
    count.append(2) #这里count两次调用如果都使用默认参数的话,则是同一个数组,非常危险!
    return count
fun()   #[2]
fun()   #[2,2]
   

```

3.
```py
[[42, 0, 0], [42, 0, 0], [42, 0, 0]]
# list 是mutable, []*3表示是引用复制三次.

# 赋值后为什么只改变列的值？

```
4. 

```text
2 2 2
None
```

本意其实是想得到一个函数列表[0*x,1*x,2*x],
但是 **Python’s closures are *late binding*. This means that the values of variables used in closures are looked up at the time the inner function is called.**
解决方案是偏函数partial

```python
from functools import partial
def fix_fn_for():
    f = [partial(lambda y, x: y * x, x=i) for i in range(3)]
    print(f[0](1), f[1](1), f[2](1))
```
或：

```python
fl=[lambda x, i=i: x*i for i in range(3)]
```

5.
```python
答案是D
可以使用dis.dis('s[a] += b')查看执行码
大概得流程是将s[a]加载到stack(tos: top of stack)， 执行tos + b，最后是赋值 s[a] = tos， 但是因为s是tuple，属于不可变类型，抛错。

1. 永远不在默认参数，tuple内等使用可变参数，同题目2。 可变类型包括：list set dict bytesarray四种
2. += 中赋值不是原子操作
3. dis模块其实没有那么复杂，尝试使用发现问题

```

[常见python陷阱](https://docs.python-guide.org/writing/gotchas/)
[The 10 Most Common Mistakes in Python](https://www.toptal.com/python/top-10-mistakes-that-python-programmers-make)
[Some Common Gotchas in Python](https://8thlight.com/blog/shibani-mookerjee/2019/05/07/some-common-gotchas-in-python.html)