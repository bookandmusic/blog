---
title: Django基础(一)-项目流程
categories:
  - 技术
  - python
  - Django
tags:
  - 入门教程
date: 2019-03-22 22:45:46
---

## 基本流程

使用命令`django-admin startproject djangodemo`,创建Django项目

```python
|-- djangodemo
|    |-- __init__.py  # 说明booktest是一个python包。
|    |-- settings.py # 项目的配置文件。
|    |-- urls.py  # 进行url路由的配置。
|    |-- wsgi.py  # web服务器和Django交互的入口。
|
|-- manage.py  # 项目的管理文件。

```

使用命令`python manage.py startapp books`, 创建DjangoApp

```python
|- books
    |- __init__.py  # 说明目录是一个Python模块。
    |- models.py  # 写和数据库项目的内容, 设计模型类。
    |- views.py  # 接收请求，进行处理，与M和T进行交互，返回应答。定义处理函数，视图函数。
    |- tests.py  # 写测试代码的文件。
    |- admin.py  # 网站后台管理相关的文件。
```

## 项目配置

> 修改项目根目下的`settings.py`文件

### 注册App

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'books', # 应用名
  ]
```

### 配置数据库

```python
DATABASES = {
      'default': {
          # 'ENGINE': 'django.db.backends.sqlite3',
          # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),

          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'djangodemo',  # 使用的数据库名字，数据库必须手动创建
          'USER': 'root',  # 连接mysql的用户名
          'PASSWORD': 'mysql',  # 用户名对应的密码
          'HOST': 'localhost',  # 指定mysql数据库所在电脑的IP
          'PORT': 3306,  # mysql服务器的端口号
      }
  }
```

> **注意:**  mysql数据库**链接驱动**，django2.x之后，官方建议使用 `mysqlclient`,若非要使用 `pymysql`，还需要如下配置

在与`settings.py`同级的`__init__.py`文件中添加:

```python
import pymysql
pymysql.install_as_MySQLdb()
```

### 配置模板目录

```python
# 在项目根目录下, 新建模板目录 templates
TEMPLATES = [
  {
      'BACKEND': 'django.template.backends.django.DjangoTemplates',
      'DIRS': [os.path.join(BASE_DIR, "templates")],
      'APP_DIRS': True,
      'OPTIONS': {
          'context_processors': [
              'django.template.context_processors.debug',
              'django.template.context_processors.request',
              'django.contrib.auth.context_processors.auth',
              'django.contrib.messages.context_processors.messages',
          ],
      },
  },
]
```

### 配置静态资源

```python
# 静态资源展示的url前缀
STATIC_URL = '/static/'
# 静态资源存放的目录，需要先创建static目录
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
```

### 本地化配置

```python
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_TZ = True # 设置为True， 数据库存储默认使用UTC时区，设置为False， 使用本地时区
```

### 配置HOST地址

```python
ALLOWED_HOSTS = ["*"] # 允许客户端通过服务器的任何ip(本地ip和网络ip)访问服务器
```

## 创建后台超级管理员

- 进入manage.py目录下，使用命令`python manage.py createsuperuser`来进行创建

- 接下来输入用户名称 : `Username(leave bkank to use 'administrator'): root`

- 然后是输入邮箱（QQemail等都可以）: `Email address：xxxx@xxx.com`

- 输入密码（输入密码时不会显示出来，并且长度必须超过八位才行）: `Password：********`

- 确认密码 : `Password(again)：********`

- 当两次密码输入相同且超过八位的时候，就会提示创建超级用户成功 : `Superuser created successfully`

## 数据模型

### 创建模型类

在应用`books`目录下的`models.py`文件中创建模型类

```python
from django.db import models

# Create your models here.
class BookInfo(models.Model):
    title = models.CharField(max_length=20)
    pub_date = models.DateField()
    read = models.IntegerField(default=0)
    comment = models.IntegerField(default=0)
    # 删除标记
    isDelete = models.BooleanField(default=False)

    class Meta:
        db_table = 'tb_books'  # 指定表名
        verbose_name = '图书'  # 后台显示表名
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.title  # 查询对象时，显示信息

class HeroInfo(models.Model):
    name = models.CharField(max_length=20)
    gender = models.BooleanField(default=False)
    comment = models.CharField(max_length=200)
    book = models.ForeignKey('BookInfo', on_delete=models.CASCADE)
    isDelete = models.BooleanField(default=False)
		
    class Meta:
        db_table = 'tb_heros'  # 指定表名
        verbose_name = '英雄'  # 后台显示表名
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name  # 查询对象时，显示信息
```

### 模型类生成表

- 生成迁移文件`python manage.py makemigrations`

- 迁移文件生成表`python manage.py migrate`,生成表名的默认格式：应用名_模型类名小写,可以指定表名

### Admin后台管理

在应用book下的admin.py中注册模型类。

告诉djang框架根据注册的模型类来生成对应表管理页面。

```python
from book.models import BookInfo
from book.models import HeroInfo

# 后台管理相关文件
# Register your models here.
class BookInfoAdmin(admin.ModelAdmin):
    """图书模型管理类"""
    list_display = ['id', 'book_title', 'book_pub_date']

class HeroInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'hero_name', 'hero_sex', 'hero_comment', 'hero_book']

# 注册模型类
admin.site.register(BookInfo, BookInfoAdmin)
admin.site.register(HeroInfo, HeroInfoAdmin)
```

## 模板

在 `templates`目录下 创建  `hero_list.html`, 展示所有 英雄信息

```python
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>英雄列表</title>
</head>
<body>
<table>
    {% for hero in heroes %}   # 类似python的for循环和if判断
        {% if not hero.isDelete %}
            <tr>
                <td>{{ hero.id }}</td>
                <td>{{ hero.name }}</td>
                <td>{{ hero.gender }}</td>
                <td>{{ hero.comment }}</td>
                <td>{{ hero.book }}</td>
            </tr>
        {% endif %}
    {% endfor %}
</table>
</body>
</html>
```

## 视图

在 `books` APP目录下的 `views.py`中创建自己的 视图函数

```python
from django.shortcuts import render
from books.models import *

def hero_list_view(request):
    heros = HeroInfo.objects.all()  # 利用模型类查询全部数据
    return render(request, 'hero_list.html', context={'heros': heros})  # 利用render函数渲染模板
```

## 路由

在 项目同名目录的 `urls.py`中配置路由

```python
from django.urls import path
from book.views import *


urlpatterns = [
    path('heroes/', hero_list_view)
]
```

## 项目启动

- 只能通过本地ip访问 : `python manage.py runserver` 

  浏览器输入`127.0.0.1:8000`访问服务器

- 可以通过网络ip访问 : `python manage.py runserver 0.0.0.0:8000`

  浏览器输入`网络ip:8000`访问服务器，同时需要搭配 `settings.py`中的配置参数使用
