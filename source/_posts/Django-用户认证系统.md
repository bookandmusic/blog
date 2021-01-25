---
title: Django-用户认证
categories:
  - 技术
  - python
  - Django
tags:
  - auth
  - 模板
date: 2020-03-10 21:34:46
---
## 项目框架

1. 项目结构

    ```bash
    .
    ├── users
    │    ├── __init__.py
    │    ├── admin.py
    │    ├── apps.py
    │    ├── migrations
    │    │   └── __init__.py
    │    ├── models.py
    │    ├── tests.py
    │    ├── urls.py
    │    └── views.py
    ├── db.sqlite3
    ├── manage.py
    ├── django_demo
    │    ├── __init__.py
    │    ├── settings.py
    │    ├── urls.py
    │    └── wsgi.py
    └── templates
        ├── base.html
        ├── index.html
        ├── login.html
        ├── register.html
        └── user.html

    ```

## 准备工作

### 创建项目

通过终端命令,创建项目

```bash
django_admin startproject django_demo
```

### 生成应用

通过终端命令,生成users应用

```bash
# 此时在项目文件夹之下执行命令
python manage.py startapp users
```

### 修改配置信息

在`settings.py`修改配置

```python
# 注册app
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users.apps.UsersConfig' # 注册users应用
]

# 配置模板路径
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

### 创建模板

在项目根路径下,创建`templates`,在目录下创建`base.html`文件,以便后面复用

```html
{% block nav %}
    <p>
        <a href="{% url 'users:index' %}">Index</a>

        {% if user.is_authenticated %}
            Hello, <a href="{% url 'users:user' %}">{{ user.username }} </a> .
            <a href="{% url 'users:logout' %}">log out</a>
        {% else %}
            <a href="{% url 'users:register' %}">register</a>
            <a href="{% url 'users:login' %}">log in</a>
        {% endif %}
    </p>
{% endblock %}

{% block err %}
    {% if account_msg %}
        <p>{{account_msg}}. Please try again.</p>
    {% endif %}

{% endblock %}

{% block body %}

{% endblock %}
```

### 指定用户模型类

1. 在`users/models.py`中创建模型类

    ```python
    from django.db import models
    from django.contrib.auth.models import AbstractUser
    
    # Create your models here.
    class UserProfile(AbstractUser):
        phone = models.CharField(max_length=11, unique=True, verbose_name="手机号")
    
        class Meta:
            db_table = 'tb_users'  # 指定表名
            verbose_name = '用户'  # 后台显示表名
            verbose_name_plural = verbose_name
    
        def __str__(self):
            return self.username
    ```

2. 在`settings.py`文件中,指定本项目用户模型类
    Django用户模型类是通过全局配置项 AUTH_USER_MODEL 决定的

    ```python
    # 配置规则：AUTH_USER_MODEL = '应用名.模型类名'
    AUTH_USER_MODEL = 'users.UserProfile'
    ```

3. 通过终端命令,生成迁移文件和迁移表

    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```

## 逻辑实现

### 实现注册逻辑

#### 定义视图

```python
from django.contrib.auth.hashers import make_password
from django.contrib.auth import login

from django.views import View
from django.shortcuts import render
from django.http import JsonResponse, HttpResponseForbidden
from .models import UserProfile
  
class RegisterView(View):

  def get(self, request):
      return render(request, 'register.html')

  def post(self, request):
      username = request.POST.get("username")
      password = request.POST.get("password")
      password2 = request.POST.get("password2")
      phone = request.POST.get("phone")

      if not all([username, password, password2, phone]):
          return JsonResponse("缺少必要参数")

      if password != password2:
          return HttpResponseForbidden("密码不一致")

      # 此时 手动加密密码
      # hash_password = make_password(password)
      # try:
      #   user = UserProfile(username=username, password=hash_password, phone=phone)
      #   user.save()
      # except Exception as e:
      #     return HttpResponseForbidden("创建失败")

      try:
          # Django认证系统用户模型类提供的 create_user() 方法创建新的用户。
          # create_user() 方法中封装了 set_password() 方法加密密码
          user = UserProfile.objects.create_user(username=username, password=password, phone=phone)
      except Exception as e:
          return HttpResponseForbidden("创建失败")

      # 状态保持
      login(request, user)

      return JsonResponse({"msg": "ok", "code": 200})
```

#### 实现模板

```html
{% extends "base.html" %}

{% block body %}
    <form action="{% url 'users:register' %}" method="post">
        用户名:<input type="text" name="username">
        密码:<input type="password" name="password">
        确认密码:<input type="password" name="password2">
        手机号: <input type="phone" name="phone">
        <input type="submit" value="提交">
    </form>
{% endblock %}
```

### 实现登录逻辑

#### 定义视图

```python
class LoginView(View):
    def get(self, request):

        # 判断用户是否登录(属性) user.is_authenticated
        if request.user.is_authenticated:
            return redirect(reverse('users:index'))

        return render(request, "login.html")

    def post(self, request):
        username = request.POST.get("username")
        password = request.POST.get("password")
        remembered = request.POST.get("remembered")

        if not all([username, password]):
            # return JsonResponse({"account_msg": "缺少必要参数", "code": 403})
            return render(request, 'login.html', {'account_msg': '缺少必要参数', "code": 403})

        # 校验用户信息，成功返回user对象，否则为None
        user = authenticate(username=username, password=password)

        if user is None:
            return render(request, 'login.html', {'account_msg': '用户名或密码错误', "code": 403})

        # 实现状态保持
        login(request, user)

        # 设置状态保持的周期
        if remembered != 'on':
            # 没有记住用户：浏览器会话结束就过期
            request.session.set_expiry(0)
        else:
            # 记住用户：None表示两周后过期
            request.session.set_expiry(None)

        # 响应登录结果
        return redirect(reverse('users:index'))
```

#### 实现模板

```html
{% extends "base.html" %}

{% block body %}


    <form method="post" action="{% url 'users:login' %}">
        {% csrf_token %}
        用户名:<input type="text" name="username">
        密码:<input type="password" name="password">
        <button name="submit">login</button>
        <input type="hidden" name="next" value="{% url 'users:index' %}"/>
    </form>

{% endblock %}
```

### 注销

#### 定义视图

```python
class LogoutView(View):

    def get(self, request):
        """实现退出登录逻辑"""
        # 清理session
        logout(request)

        # 退出登录，重定向到首页
        response = redirect(reverse('users:index'))

        # 退出登录时清除cookie中的username
        response.delete_cookie('username')
        return response
```

### 用户信息

#### 定义视图

```python
class UserView(View):

    def get(self, requset):
        # 如果用户没有登陆就访问本应登陆才能访问的页面时会直接跳转到登陆页面
        if requset.user.is_authenticated:
            return render(requset, 'user.html')
        else:
            return redirect(reverse('users:login'))

    def post(self, request):
        old_password = request.POST.get("old_password")
        new_password = request.POST.get("new_password")
        conf_password = request.POST.get("conf_password")

        if not all([old_password, new_password, conf_password]):
            return render(request, 'user.html', {'account_msg': '缺少必要参数', "code": 403})

        if new_password != conf_password:
            return render(request, 'user.html', {'account_msg': '密码不一致', "code": 403})

        # 校验密码  check_password()
        if not request.user.check_password(old_password):
            return render(request, 'user.html', {'account_msg': '密码不正确', "code": 403})

        user = request.user

        # 修改密码
        user.set_password(new_password)
        # user.password = make_password(new_password)

        # 保存修改
        user.save()

        # 状态保持
        login(request, user)

        return redirect(reverse('users:index'))

```

#### 实现模板

```html
{% extends 'base.html' %}

{% block body %}

    <form action="{% url 'users:user' %}" method="post">
        旧密码: <input type="password" name="old_password">
        新密码: <input type="password" name="new_password">
        确认密码: <input type="password" name="conf_password">
        <button name="submit">确认修改</button>
        <input type="hidden" name="next" value="{% url 'users:index' %}"/>
    </form>

{% endblock %}
```

### 首页

#### 定义视图

```python
class IndexView(View):

def get(self, request):
    return render(request, 'index.html')
```

#### 实现模板

```html
{% extends 'base.html' %}

{% block body %}
    <h1>欢迎来到首页</h1>
{% endblock %}
```

## 配置路由

`django_demo\urls.py`中配置总路由

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('users/', include("users.urls","users"))
]
```

`users\urls.py`中配置子路由

```python
from django.urls import path
from .views import RegisterView, LoginView, IndexView, LogoutView, UserView

app_name = "users"

urlpatterns = [
    path('register/', RegisterView.as_view(), name="register"),
    path('login/', LoginView.as_view(), name="login"),
    path('logout/', LogoutView.as_view(), name="logout"),
    path("user/", UserView.as_view(), name='user'),
    path('', IndexView.as_view(), name="index")
]
```