---
title: python高阶函数
date: 2020-06-06 11:13:12
categories:
- 技术
- python
- 基础
tags:
- 高阶函数
- sorted
- map
- reduce
- filter
---

## 高阶函数介绍

### 定义

当一个函数 接收 另一个函数作为参数， 它就是 `高阶函数`

### 示例

```python
def func(a, x, y):
    a(x, y)  # a就是传递的 函数参数f，  执行 a 相当于 执行  f

def f(x, y):
    print(x + y)

func(f, 1, 2)  # 3， 在函数f执行过程中，输出 3
```

## 四大高阶函数

### `map(func, iterable)`

>将 **可迭代对象** 中的每个值 按照 第一个参数( **函数**)的 规则 依次 处理， 得到一个 新的 **迭代器** 对象


```python
def func(alist):
    """
    :param alist: 接收列表作为参数
    :return: 将列表中每个数字，求平方之后，返回一个新列表
    """

    def sqr(x):
        return x * x

    r = map(sqr, alist) # r是迭代器对象

    return list(r)  # 将迭代器对象转换为列表


r = func([11, 22, 33, 44, 55])
print(r)
```



```python
# 定义函数， 接收 列表 作为参数 ， 返回一个 包含 绝对值  的新列表
# abs()：python内置函数，求一个数的绝对值
def func(alist):
    # map 高阶函数需要的参数，不用管是内置的，还是自定义的
    r = map(abs, alist)
    return list(r)


r = func([-12, 34, 56, -98])

print(r)  # [12, 34, 56, 98]
```



```python
# 定义函数，接收 一个 包含 单词的字符串， 返回 一个 每个单词 首字母大写 的字符串
def func(astr):
    # 将字符串切分为单词列表
    wlist = astr.split()

    def toUpper(x):
        return x.title()

    r = map(toUpper, wlist)

    # 将单词拼接为字符串
    s = " ".join(r)

    return s


r = func("a good  fruit apple")

print(r)  # " A Good  Fruit Apple"

# 提示： 可以考虑 将 字符串按照 单词 切分， map处理之后， 再 重新拼接为字符串
```

### `reduce(func, iterable)`

> 将可迭代对象中所有的元素，按照第一个 函数的规则， 连续操作，得到一个  **具体的数据**

```python
from functools import reduce


# 自定义一个Mysum()函数，接收一个列表作为参数， 返回列表中所有元素之和
def mysum(alist):
    def add(x, y):
        return x + y

    r = reduce(add, alist) # reduce 的结果是一个具体的值

    return r


r = mysum([1, 2, 3, 4, 5])
print(r)  # 15
```

### `filter(func, iterable)`

> 按照第一个 函数的 **返回值** 对 可迭代对象 中 所有的数据 进行 过滤， 得到一个 包含过滤后结果的 **迭代器对象**

```python
# 自定义一个filterNumber()函数，接收一个列表作为参数， 返回一个 包含偶数 的列表

def filterNumber(alist):
    def is_even(x):
        """如果函数的返回结果为True, 对应的元素可以得到"""
        if x % 2 == 0:
            return True

    r = filter(is_even, alist)  # r也是迭代器对象

    return list(r)  # 通过类型转换，将迭代器中数据取出


r = filterNumber([1, 2, 3, 4, 5, 6])
print(r)  # [2, 4, 6]]
```

```python
# 自定义一个函数 sevenNumber()函数，接收一个列表作为参数， 返回一个 包含 "逢7必过" 数字的新列表
def sevenNumber(alist):
    def f(x):
        # 判断为7的倍数，返回True
        if x % 7 == 0 or '7' in str(x):
            return True

    r = filter(f, alist)  # r是迭代器对象

    return list(r)


r = sevenNumber(range(1, 101))
print(r)
```



### `sorted(iterable, key=func, reverse=False)`

> 默认将列表 按照 升序 排序

```python
alist = [21, 14, -67, -34, 19]

# 按照 数字大小 升序 排序
r = sorted(alist)
print(r)  # [-67, -34, 14, 19, 21]

# 按照 数字大小 降序 排序
r = sorted(alist, reverse=True)
print(r)  # [21, 19, 14, -34, -67]

# 按照每 个数字的绝对值 升序 排序
r = sorted(alist, key=abs)
print(r)  # [14, 19, 21, -34, -67]

# 按照 每个数字的绝对值 降序 排序
r = sorted(alist, key=abs, reverse=True)
print(r)  # [-67, -34, 21, 19, 14]
```
