---
title: python自动化测试
date: 2021-05-15 18:48:25
categories:
  - 技术
  - python
  - web
tags:
  - unittest
  - django test
  - mock
---

>   写好了一个项目之后，要确定各个模块是不是稳定运行，有没有bug，比如一个项目有100个路由，可以手动一个一个URL的输入，查看响应有没有毛病，但这样不符合程序员的逼格。
>   由此，诞生了单元测试与集成测试。

## python unittest

[unittest](https://docs.python.org/3.7/library/unittest.html)是python内置的用于测试代码的模块，无需安装， 使用简单方便。

### unittest 简介

>   `unittest`中最核心的部分是：`TestFixture`、`TestCase`、`TestSuite`、`TestRunner`

`unittest case`的运行流程：

-   写好一个完整的`TestCase`
-   多个`TestCase` 由`TestLoder`被加载到`TestSuite`里面, `TestSuite`也可以嵌套`TestSuite`
-   由`TextTestRunner`来执行`TestSuite`，测试的结果保存在`TextTestResult`中
-   `TestFixture`指的是环境准备和恢复

#### Test Fixture

用于测试环境的准备和恢复还原, 一般用到下面几个函数。

-   `setUp()`：准备环境，执行每个测试用例的前置条件
-   `tearDown()`：环境还原，执行每个测试用例的后置条件
-   `setUpClass()`：必须使用`@classmethod`装饰器，所有`case`执行的前置条件，只运行一次
-   `tearDownClass()`：必须使用`@classmethod`装饰器，所有`case`运行完后只运行一次

------

#### Test Case

-   参数`verbosity`可以控制错误报告的详细程度：`默认为1`。
    -   `Verbosity=0`，表示不输出每一个用例的执行结果；
    -   `Verbosity=1`情况下`成功是 .`，失败是 F，出错是 E，跳过是 S；
    -   `Verbosity=2`情况下会打印测试的注释,显示详细的执行报告结；
-   测试的执行跟方法的顺序没有关系, **默认按字母顺序**
-   每个测试方法均以 `test_` 开头

---

#### Test Suite

-   一般通过`addTest()`或者`addTests()`向`suite`中添加
-   `case`的执行顺序与添加到Suite中的顺序是一致的
-   使用**装饰器**的方式来实现跳过测试与预计的失败，常用的主要有3种:
    -   `@unittest.skip()`：直接跳过测试用例；
    -   `@unittest.skipIf(condition,reason)`：当满足条件时跳过测试用例；
    -   `@unittest.skipUnless(condition,reason)`：只有满足某一条件时不跳过，其他的都跳过；

----

#### Test Loder

-   `TestLoadder`用来加载`TestCase`到`TestSuite`中

-   `loadTestsFrom*()`方法从各个地方寻找`testcase`，创建实例，然后`addTestSuite`，再返回一个`TestSuite`实例

-   `defaultTestLoader()` 与 `TestLoader()`功能差不多，复用原有实例

    ```python
    unittest.TestLoader().loadTestsFromTestCase(testCaseClass)
    unittest.TestLoader().loadTestsFromModule(module)
    unittest.TestLoader().loadTestsFromName(name, module=None)
    unittest.TestLoader().loadTestsFromNames(names, module=None)
    unittest.TestLoader().discover()
    ```

### 测试用例

>   定义实现**测试用例**，然后调用即可

```python
import unittest
import requests


class TestCaseDemo(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        print("this setupclass() method only called once.\n")

    @classmethod
    def tearDownClass(cls):
        print("this tear_down_class() method only called once too.\n")

    def setUp(self):
        print("do something before test : prepare environment.\n")

    def tearDown(self):
        print("do something after test : clean up.\n")

    def test_goods_list(self):
        r = requests.get('http://127.0.0.1:8000/goods/')

        # 断言
        self.assertEqual(200, r.status_code, '查询商品失败')

        result = r.json()
        self.assertEqual(list, type(result), '查询商品，结果不对')

    def test_goods_create(self):
        r = requests.post('http://127.0.0.1:8000/goods/')
        # 断言
        self.assertEqual(405, r.status_code, '不应该允许添加商品')


if __name__ == '__main__':
    unittest.main(verbosity=2)
```

### 测试套件

上述演示了一种比较基础、简单的测试用例的使用方法，但是这样比较固化，只能自动的去查找以*test*开头的测试方法，然后顺序的去执行测试方法，这样显然是有点僵化的，不能按照重要程度或者我们的意愿去执行测试方法，而且遇到多个测试用例是会比较混乱。
这里要讲的测试套件能够归档测试用例，能够让我们按照指定的顺序去执行测试方法。

-   创建测试实例
-   创建测试套件
-   加载测试实例
-   添加测试实例到测试套件
-   创建测试运行组件，加载测试套件
-   运行测试套件

```python
.
└── testcase
    ├── __init__.py
    ├── test.py  # 测试套件
    └── test_demo_class.py  # 测试实例
```



```python
import HTMLReport
from unittest import TestLoader, TestSuite, TextTestRunner

import test_demo_class  # 导入测试模块
from test_demo_class import TestCaseDemo

if __name__ == '__main__':
    # 2. 创建测试套件
    suite = TestSuite()

    # 3.加载测试实例
    # test1 = TestCaseDemo("test_goods_list")
    # test2 = TestCaseDemo('test_goods_create')
    tests = [
        TestCaseDemo("test_goods_list"), 
        TestCaseDemo("test_goods_create")
    ]
    # tests = TestLoader().loadTestsFromModule(test_demo_class)
    # tests = TestLoader().loadTestsFromTestCase(TestCaseDemo)
    # tests = TestLoader().loadTestsFromName('test_demo_class.TestCaseDemo.test_goods_list')
    # tests = TestLoader().loadTestsFromNames([
    #     'test_demo_class.TestCaseDemo.test_goods_list',
    #     'test_demo_class.TestCaseDemo.test_goods_create'
    # ])
    # tests = TestLoader().discover('.', 'test_*.py', top_level_dir=None)

    # 4. 添加测试实例到测试套件
    # suite.addTest(test1)
    # suite.addTest(test2)
    suite.addTests(tests)

    # 5. 创建运行套件，并运行
    runner = TextTestRunner(verbosity=2)  # 输出到终端

    # runner = HTMLReport.TestRunner(report_file_name='test',
    #                                 output_path='report',
    #                                 title='测试报告',
    #                                 description='测试描述',
    #                                 sequential_execution=True
    #                                 )  # 输出到HTML网页
    #
    runner.run(suite)

    # with open('ut_log.txt', 'a') as fp:  # 输出到txt文件
    #     runner = TextTestRunner(stream=fp, verbosity=2)
    #     runner.run(suite)
```

## django test

### 数据库配置

```python
DATABASES = {
    'default': {
        'ENGINE' : 'django.db.backends.mysql',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'django_db1', # 真实的数据库
        'USER': 'root',
        'PASSWORD': '123456',
        'TEST':{
            'NAME':'django_db_test' # 用于单元测试的数据库
        }
    }
}
```

### 单元测试文件

>   在每个APP里面，都有一个`test.py`文件，可以在里面去定义测试类，并可以直接运行

```python
from django.test import TestCase, Client
from .models import Goods


# Create your tests here.
class GoodsTestCase(TestCase):

    def test_goods_list(self):
        # 创建一个客户端
        client = Client()
        # 模拟客户端访问首页
        response = client.get('/goods/')

        # 测试，self.assertEqual接收3个参数(被测试对象，正确的测试结果，如果不正确输出什么）
        # 如果第一个参数的返回值不等于第二个参数，输出第三个参数
        self.assertEqual(
            response.status_code,
            200,
            '商品查询失败'
        )

        result = response.json()
        self.assertEqual(
            type(result),
            list,
            '商品查询结果不对'
        )


if __name__ == '__main__':
    userTestCase = GoodsTestCase()
    userTestCase.test_goods_list()
```

## mock

在单元测试进行的同时，就离不开`mock`模块的存在，初次接触这个概念的时候会有这样的疑问：把要测的东西都模拟掉了还测试什么呢？
但在，实际生产中的项目是非常复杂的，对其进行单元测试的时候，会遇到以下问题：

-   接口的依赖
-   外部接口调用
-   测试环境非常复杂

单元测试应该只针对当前单元进行测试, 所有的内部或外部的依赖应该是稳定的, 已经在别处进行测试过的。使用`mock` 就可以对外部依赖组件实现进行模拟并且替换掉, 从而使得单元测试将焦点只放在当前的单元功能。

因为在为代码进行单元测试的同时，会发现该模块依赖于其他的模块，例如数据库，网络，或者第三方模块的存在，而我们对一个模块进行单元测试的目的，是测试当前模块正常工作，这样就要避开对其他模块的依赖，而mock主要作用便在于，专注于待测试的代码。而在但与测试中，如何灵活的使用mock模块是核心所在。下面便以mock为核心，结合最近所写的代码，阐述mock模块的使用。

```python
import requests
from unittest import mock

def request_lemonfix():
    """

    :return:
    """
    res = requests.get('http://www.lemonfix.com')
    return res.status_code.encode('utf-8')


if __name__ == '__main__':
    request_lemonfix = mock.Mock(return_value="这里会显示论坛主页")
    print(request_lemonfix())
```

