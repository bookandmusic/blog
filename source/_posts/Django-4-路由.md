---
title: Django基础(四)-路由
categories:
  - 技术
  - python
  - Django
tags:
  - namespace
  - app_name
  - reverse
  - django基础
date: 2019-05-25 22:45:46
---
> Django2.0 新增了在 urls.py 中 `app_name` 来指定 namespace。我们之前通过 `reverse` 函数来反向获取 url

## reverse 语法

```python
reverse("<namespace>:<url-name>", kwargs={"<kwarg>": "<val>"})
```

## 流程

### 路由定义

在项目的总路由中，可以通过指定namespace来确定应用

```python
urlpatterns = [
    path(r'users/', include(('users.urls', 'userss')))
]
```

更进一步,把 namespace 定义到被 include 的  子路由`users/urls.py` 中去使用 app_name 定义名称空间

```python
from django.urls import re_path, path
from users.views import RegisterView, LoginView, DetailView, IndexView

app_name = 'users'

urlpatterns = [
  path('detail/<int:uid>/', DetailView.as_view(), name="detail"),
  path("", IndexView, name="index")
]
```

### 反向解析

现在我们仍然可以用 reverse 函数和模板中的 url 获取 URL

```python
reverse("users:index")
reverse("users:detail", kwargs={"uid": 2020})
{% url "users:index" %}
{% url "users:detail" uid=2020 %}
```

