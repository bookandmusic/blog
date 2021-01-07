---
title: 变量进阶
date: 2020-06-06 11:10:17
categories:
- 技术
- python
- 函数
tags:
- LEGB
- 引用计数
- global
- nonlocal
---

### 变量的作用域

- 在Python程序中创建、改变、查找变量名时，都是在一个保存变量名的空间中进行，我们称之为命名空间，也被称之为作用域。

<!--more-->

- Python的作用域是静态的，在源代码中变量名被赋值的位置决定了该变量能被访问的范围。即Python变量的作用域由变量所在源代码中的位置决定。

### 作用域的产生

- 只有当变量在`Module(模块)`、`Class(类)`、`def(函数)`中定义的时候，才会有作用域的概念。

- 在作用域中定义的变量，一般只在该作用域中有效。

- 在`if-elif-else`、`for-else`、`while`、`try-except/try-finally`等关键字的语句块中并不会产生作用域

#### 代码分析1

```python
def func():
    variable = 100
    print(variable)
print(variable)
```

> 代码执行到第二个`print(variable)`时会报错 `NameError: name 'variable' is not defined`
>
>变量`variable`只在函数`func()`的定义域内有效！

#### 代码分析2

```python
if True:
    variable = 100
    print(variable)
print("******")
print(variable)
```

输出结果为：

```shell
100     
******
100
```

程序没有报错，因为if语句不产生作用域

### 变量作用域的类型

- L(local)局部作用域
  - √ 函数内的命名空间
- E(enclosing)嵌套作用域
  - √ 外部嵌套函数的命名空间
- G(global)全局作用域
  - √ 所在模块（文件）的命名空间
- B(built-in)内置作用域
  - √ Python内置模块的命名空间

#### 实例解析

```python
globalVar = 100  # 全局作用域     


def test_scope():
    enclosingVar = 200  # 嵌套作用域

    def func():
        localVar = enclosingVar + 1  # 局部作用域
        print(localVar)


print(__name__)  # 内置作用域
```

### 变量名解析LEGB法则

当在函数中使用未确定的变量名时，`Python`会按照优先级依次搜索4个作用域，以此来确定该变量名的意义。

- 首先搜索局部作用域(L)，

- 之后是上一层嵌套结构中`def`或`lambda`函数的嵌套作用域(E)，

- 之后是全局作用域(G)，

- 最后是内置作用域(B)。

- 按这个查找原则，在第一处找到的地方停止。如果没有找到，则会出发`NameError`错误。

### 搜索变量名的优先级：

- 局部作用域 > 嵌套作用域 > 全局作用域 > 内置作用域

#### 实例1

```python
def func():
    variable = 300
    print(variable)  #打印局部作用域的variable

variable = 100
func()
print(variable) #打印全局作用域的variable
```

运行结果：

```shell
300
100
```

#### 实例2

```python
def test_scopt():
    variable = 200
    print(variable)
    def func():
        print(variable) #此处打印的是上层嵌套函数
    func()              #test_scopt()中的variable
variable = 100
test_scopt()
print(variable)
```

运行结果：

```shell
200
200
100
```

#### 实例3

```python
variable = 300
def test_scopt():
    print(variable) #此处的变量variable绑定的是下一行  
    variable = 200  #语句中所声明的局部变量，但未被赋值所以报错

test_scopt()
print(variable)
```

运行结果：

```shell
UnboundLocalError: local variable 'variable' referenced before assignment
```

#### 实例4

```python
variable = 300
def test_scopt():
    print(variable)  #此处打印的是全局变量variable
    #variable = 200  #如果这行未被注释，那么上一句print打印的应是
                     #此处定义的局部变量     
test_scopt()
print(variable)
```

运行结果：

```shell
300
300
```

仔细体会实例4和实例3的区别！

### global与nonlocal关键字

- global适用于函数内部修改全局变量的值

- nonlocal适用于嵌套函数中内部函数修改外部变量的值

#### 实例1（global关键字）

```python
spam = 99   # <--
def tester():
    def nested():
        global spam   #绑定到了第一行定义的spam
        print('current=',spam)
        spam = 200
    return nested
tester()()
print(spam)
```

运行结果：

```shell
current= 99
200
```

#### 实例2（nonlocal关键字）

```python
def outer():
    count = 10 # <--
    def inner():
        nonlocal count #绑定到了第二行定义的count
        count = 20
        print(count)
    inner()
    print(count)
outer()
```

运行结果：

```shell
20
20
```
