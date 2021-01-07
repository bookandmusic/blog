---
title: 匿名函数
date: 2020-06-06 11:14:42
categories:
- 技术
- python
- 函数
tags:
- lambda
- 匿名函数
---

- 匿名函数的另一个别称是`lambda表达式`

<!--more-->

- lambda表达式的语法：
  ```python
  lambda argument_list: expression
  ```
  - `argument_list`是参数列表。它的结构与Python中函数(`function`)的参数列表是一样的。
  - `expression`是一个关于参数的表达式。表达式中出现的参数需要在`argument_list`中有定义，并且表达式只能是单行的。
  
  ```python
  # lambda x: x*x
  # 这个lambda表达式实际等同于：
  # def f(x):
  #   return x*x
  In [15]: f = lambda x: x * x

  In [16]: f
  Out[16]: <function __main__.<lambda>(x)>

  In [17]: f(5)
  Out[17]: 25
      
  In [24]: x2 = lambda *args: [x**2 for x in list(args)]
  
  In [25]: x2(1, 2, 3, 4, 5, 6, 7)
  Out[25]: [1, 4, 9, 16, 25, 36, 49]
  ```

- 匿名函数作为参数

  ```python
  In [7]: s = [x for x in range(10)]
  
  In [8]: s
  Out[8]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  
  In [9]: s.sort(key=lambda x : abs(x-5))
  
  In [10]: s
  Out[10]: [5, 4, 6, 3, 7, 2, 8, 1, 9, 0]
  ```

  ```python
  In [32]: stus = [
  ...:     {"name": "zhangsan", "age": 18},
  ...:     {"name": "lisi", "age": 19},
  ...:     {"name": "wangwu", "age": 17}
  ...: ]
  
  In [33]: stus.sort(key=lambda x : x["name"])
  
  In [34]: stus
  Out[34]:
  [{'name': 'lisi', 'age': 19},
   {'name': 'wangwu', 'age': 17},
   {'name': 'zhangsan', 'age': 18}]
  
  In [35]: stus.sort(key=lambda x : x["age"])
  
  In [36]: stus
  Out[36]:
  [{'name': 'wangwu', 'age': 17},
   {'name': 'zhangsan', 'age': 18},
   {'name': 'lisi', 'age': 19}]
  ```
