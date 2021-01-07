---
title: 正则匹配
categories:
  - 技术
  - python
  - 高级
tags:
  - 正则
date: 2019-05-23 20:51:32
---

## 正则匹配

1. 题目描述：

现公司要开发一个业务管理系统，要求注册环节的密码需要提示用户其安全等级，密码按如下规则进行计分，并根据不同的得分为密码进行安全等级划分；此外，密码的组成可以由字母，数字，以及符号构成。

以下为密码分数判断选项，每项判断只能拿一个分项：

- 密码长度:
  - 5 分: 小于等于4个字符
  - 10 分: 5到7字符
  - 25 分: 大于等于8个字符
- 字母:
  - 0 分: 没有字母
  - 10 分: 全都是小（大）写字母
  - 20 分: 大小写混合字母
- 数字:
  - 0 分: 没有数字
  - 10 分: 1个数字
  - 20 分: 大于1个数字

- 符号:
  - 0 分: 没有符号
  - 10 分: 1个符号
  - 25 分: 大于1个符号

- 奖励:
  - 2 分: 字母和数字
  - 3 分: 字母、数字和符号
  - 5 分: 大小写字母、数字和符号

最后的评分标准:

  大于等于90: 非常安全
  大于等于80: 安全
  大于等于70: 非常强
  大于等于 60: 强
  大于等于 50: 一般
  大于等于 25: 弱
  大于等于0:  非常弱

当用户输入一个密码字符串后，判断密码等级，并输出

输入描述：

```shell
b12A3%$123
```

输出：

```shell
非常安全
```

代码实现如下：

```python
import re


def s1(password):
    if re.match(r".{,4}$", password):
        return 5
    if re.match(r".{5,7}$", password):
        return 10

    if re.match(r".{8,}$", password):
        return 25


def s2(password):
    if re.match(r"[^a-zA-Z]*$", password):
        return 0
    if re.match(r"([a-z]*$)|([A-Z]*$)", password):
        return 10

    if re.match(r".*(?=.*[a-z])(?=.*[A-Z]).*$", password):
        return 20


def s3(password):
    if re.match(r"[^\d]*$", password):
        return 0
    if re.match(r"[^\d]*[\d][^\d]*$", password):
        return 10
    if re.match(r".*[\d]+.*[\d]+.*$", password):
        return 20


def s4(password):
    if re.match(r"[^!@#$%^&*?]*$", password):
        return 0
    if re.match(r"[^!@#$%^&*?]*[!@#$%^&*?][^!@#$%^&*?]*$", password):
        return 10
    if re.match(r".*[!@#$%^&*?]+.*[!@#$%^&*?]+.*$", password):
        return 20


def s5(password):
    if re.match(r".*(?=.*\d.*$)(?=.*[!@#$%^&*?].*$)(?=.*[a-z].*$)(?=.*[A-Z].*$).*$", password):
        return 5

    if re.match(r"(?=.*\d.*$)(?=.*[!@#$%^&*?].*$)(?=.*[a-zA-Z].*$).*$", password):
        return 3

    if re.match(r".*(?=.*[\d].*$)(?=.*[a-zA-Z].*$).*$", password):
        return 2


password = input("请输入密码：")
ret = sum((s1(password), s2(password), s3(password), s4(password), s5(password)))

if ret >= 90:
    print("非常安全")
elif ret >= 80:
    print("安全")
elif ret >= 70:
    print("非常强")
elif ret >= 60:
    print("强")
elif ret >= 50:
    print("一般")
elif ret >= 25:
    print("弱")
else:
    print("非常弱")

```