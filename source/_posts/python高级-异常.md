---
title: python异常
categories:
  - python
tags:
  - 异常
date: 2019-03-22 22:38:25
---
![20190517210629-异常类型](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2006/20190517210629-异常类型%20.png)

### 自定义异常类型
```python
# 自定义错误类型
class ArgsException(Exception):
    def __init__(self, num, num_type):
        self.num = num  # 用来描述参数个数
        self.num_type = num_type  # 用户描述参数类型


num1 = input("输入数字:")
num2 = input("输入数字:")

try:
    if num1.isdigit() is False or num2.isdigit() is False:
             # 错误类型的实例对象
        raise ArgsException(2, "int")  # raise 异常类型  -> 主动抛出异常
except ArgsException as ret:
    print("需要%d个%s参数" % (ret.num, ret.num_type))

```