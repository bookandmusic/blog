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


# 语言设置和时区设置
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
```

## 拓展User模型类

Django 用户认证系统提供了一个内置的 User 对象，用于记录用户的用户名，密码等个人信息。对于 Django 内置的 `User` 模型， 仅包含以下一些主要的属性：

-   username，即用户名
-   password，密码
-   email，邮箱
-   first_name，名
-   last_name，姓

对于一些网站来说，用户可能还包含有昵称、头像、个性签名等等其它属性，因此仅仅使用 Django 内置的 User 模型是不够。好在 Django 用户系统遵循可拓展的设计原则，我们可以方便地拓展 User 模型。

>   继承 AbstractUser 拓展用户模型

这是推荐做法。事实上，查看 `User` 模型的源码就知道，User 也是继承自 `AbstractUser` 抽象基类，而且仅仅就是继承了 `AbstractUser`，没有对 `AbstractUser` 做任何的拓展。

所以，如果我们继承 `AbstractUser`，将获得 `User` 的全部特性，而且还可以根据自己的需求进行拓展。

我们之前新建了一个 users 应用，通常我们把和数据库模型相关的代码写在 models.py 文件里。打开 `users/models.py` 文件，写上我们自定义的用户模型代码：

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

# Create your models here.
class User(AbstractUser):
    phone = models.CharField(max_length=11, unique=True, verbose_name="手机号")

    class Meta(AbstractUser.Meta):
        db_table = 'tb_users'  # 指定表名
        verbose_name = '用户'  # 后台显示表名
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.username
```

我们给自定义的用户模型新增了一个 phone（手机号）属性，用来记录用户的联系方式，设置 `unique=True` 的目的是让用户在注册时，一个手机号只能注册一次。根据你的需求可以自己进一步拓展，例如增加用户头像、个性签名等等，添加多少属性字段没有任何限制。

同时，我们继承了 `AbstractUser` 的内部类属性 `Meta` 。在这里继承 `Meta` 的原因是在你的项目中可能需要设置一些 `Meta` 类的属性值，不要忘记继承 `AbstractUser.Meta` 中已有的属性。

此外，`AbstractUser` 类又继承自 `AbstractBaseUser`，前者在后者的基础上拓展了一套用户权限（Permission）系统。因此如非特殊需要，尽量不要从 `AbstractBaseUser` 拓展，否则你需要做更多的额外工作。

为了让 Django 用户认证系统使用我们自定义的用户模型，必须在 `settings.py` 里通过 `AUTH_USER_MODEL` 指定自定义用户模型所在的位置，即需要如下设置：

```python
# 配置规则：AUTH_USER_MODEL = '应用名.模型类名'
AUTH_USER_MODEL = 'users.User'
```

设置好自定义用户模型后，生成数据库迁移文件，并且迁移数据库以生成各个应用必要的数据库表。即运行如下两条命令：

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

OK，现在 Django 用户系统使用的用户模型就是自定义的 `User` 模型了。

**注意：**一定要在设置好 `AUTH_USER_MODEL = 'users.User'` 后在第一次迁移数据库，即指定好自定义的用户模型后再执行数据库迁移命令。

## 创建基础模板

在项目根路径下,创建`templates`,在目录下创建`base.html`文件,以便后面复用

对于一个网站来说，比较好的用户体验是登录、注册和注销后跳转回用户之前访问的页面。否则用户在你的网站东跳转西跳转好不容易找到了想看的内容，结果他已登录给他跳转回了首页，这会使用户非常愤怒。接下来我们看看如何让登录、注册和注销后跳转回用户之前访问的页面。

其实现的原理是，在登录和注销的流程中，始终传递一个 `next` 参数记录用户之前访问页面的 URL。因此，我们需要做的就是在用户访问登录或者注销的页面时，在 URL 中传递一个 `next` 参数给视图函数，具体做法如下：

```html
{% block nav %}
    <p>
        <a href="{% url 'users:index' %}">Index</a>

        {% if user.is_authenticated %}
            Hello, <a href="{% url 'users:user' %}">{{ user.username }} </a> .
            <a href="{% url 'users:logout' %}?next={{ request.path }}">退出</a>
        {% else %}
            <a href="{% url 'users:register' %}?next={{ request.path }}">注册</a>
            <a href="{% url 'users:login' %}?next={{ request.path }}">登录</a>
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

可以看到，我们在注册、登录和注销的 URL 后加了 `next` 参数，其值为 `{{ request.path }}`。`request.path` 是用户当前访问页面的 URL。在 URL 中传递参数的方法就是在要传递的参数前加一个 ？然后写上传递的参数名和参数值，用等号链接。关于在 URL 中传递参数具体请 HTTP 的相关协议。

为了在整个登录流程中记录 next 的值，还需要在表单中增加一个表单控件，用于传递 next 值。

## 自定义认证后台

Django auth 应用默认支持用户名（username）进行登录。但是在实践中，网站可能还需要邮箱、手机号、身份证号等进行登录，这就需要我们自己写一个认证后台，用于验证用户输入的用户信息是否正确，从而对拥有正确凭据的用户进行登录认证。

Django 对用户登录的验证工作均在一个被称作认证后台（Authentication Backend）的类中进行。这个类是一个普通的 Python 类，它有一个 `authenticate` 方法，接收登录用户提供的凭据（如用户名或者邮箱以及密码）作为参数，并根据这些凭据判断用户是否合法（即是否是已注册用户，密码是否正确等）。

可以定义多个认证后台，Django 内部会逐一调用这些后台的 `authenticate` 方法来验证用户提供登录凭据的合法性，一旦通过某个后台的验证，表明用户提供的凭据合法，从而允许登录该用户。

### 自定义Backend

```python
from .models import UserModel
from django.contrib.auth.backends import ModelBackend

class EmailBackend(ModelBackend):
    """user验证"""

    def authenticate(self, request, username=None, password=None, **kwargs):
        try:
            user = UserModel.objects.get(email=username)
        except UserModel.DoesNotExist:
                return None

        # 校验用户密码，且用户为激活用户
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
```

### 配置 Backend

接下来就要告诉 Django，需要使用哪些 Backends 对用户的凭据信息进行验证，这需要在 `settings.py` 中设置：

```python
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'users.backends.EmailBackend',
)
```

第一个 Backend 是 Django 内置的 Backend，当用户提供的是用户名和正确的密码时该 Backend 会通过验证；第二个 Backend 是刚刚自定义的 Backend，当用户提供的是 Email 和正确的密码时该 Backend 会通过验证。

## 注册

用户注册就是创建用户对象，将用户的个人信息保存到数据库里。回顾一下 Django 的 MVT 经典开发流程，对用户注册功能来说，首先创建用户模型（M），这一步我们已经完成了。编写注册视图函数（V），并将为视图函数绑定对应的 URL。编写注册模板（T），模板中提供一个注册表单给用户。Django 用户系统内置了登录、修改密码、找回密码等视图，但是唯独用户注册的视图函数没有提供，这一部分需要我们自己来写。

### 视图

首先来分析一下注册函数的逻辑。

-   首先用户请求注册表单，然后服务器的视图函数给用户提供注册表单；

-   用户在注册表单里填写注册信息，然后通过表单将这些信息提交给服务器。
-   视图函数从用户提交的数据提取用户的注册信息，然后验证这些数据的合法性。
-   如果数据合法，就新建一个用户对象，将用户的数据保存到数据库，否则就将错误信息返回给用户，提示用户对提交的信息进行修改。
-   注册成功，跳转到登录页面

过程就是这么简单，下面是对应的代码（视图函数的代码通常写在 `views.py` 文件里）：

```python
from django.contrib.auth.hashers import make_password

from django.views import View
from django.shortcuts import render, reverse, redirect
from django.http import HttpResponseForbidden
from .models import User

class RegisterView(View):

    def get(self, request):
        next = request.GET.get('next')
        
        # 判断用户是否登录(属性) user.is_authenticated
        if request.user.is_authenticated:
            if next:
                return redirect(next)
            else:
            	return redirect(reverse('users:index'))
        
        
        # 1. 提供注册页面
        return render(request, 'register.html', context={'next': next})

    def post(self, request):
        # 2.1 获取用户的注册信息
        username = request.POST.get("username")
        password = request.POST.get("password")
        password2 = request.POST.get("password2")
        phone = request.POST.get("phone")
        next = request.POST.get('next', '')
		
        # 2.2 验证数据的合法性
        if not all([username, password, password2, phone]):
            return render(request, 'index.html',context={'next': next, 'err_msg': '缺少必要参数'})
            

        if password != password2:
            return render(request, 'index.html', context={'next': next, 'pwd_msg': '密码不一致'})
        
        if not re.match('^(13\d|14[5|7]|15\d|166|17[3|6|7]|18\d)\d{8}$'):
            return render(request, 'index.html', context={'next': next, 'phone_msg': '手机号不合法'})
		
        # 2.3 新建用户对象
        
        # 此时 手动加密密码
        # hash_password = make_password(password)
        # try:
        #   user = User(username=username, password=hash_password, phone=phone)
        #   user.save()
        # except Exception as e:
        #     return HttpResponseForbidden("创建失败")

        try:
            # Django认证系统用户模型类提供的 create_user() 方法创建新的用户。
            # create_user() 方法中封装了 set_password() 方法加密密码
            user = User.objects.create_user(username=username, password=password, phone=phone)
        except Exception as e:
            return render(request, 'index.html', context={'next': next, 'err_msg': '创建失败'})


        # 2.4 注册成功，跳转回注册前页面
        if next:
            return redirect(next)
        else:
            return redirect('/')
```

### 模板

在表单中增加了一个隐藏的 `input` 控件，其值为 `{{ next }}`，即之前通过 `URL` 参数传递给注册视图函数的，然后注册视图函数又将该值传递给了 `index.html` 模板。这样在整个注册流程中，始终有一个记录着用户在注册前页面 URL 的变量 `next` 在视图和模板间来回传递，知道用户注册成功后再跳转回 `next` 记录的页面 URL。

```html
{% extends "base.html" %}

{% block body %}
    <form action="{% url 'users:register' %}" method="post">
        用户名:<input type="text" name="username"> <br>
        密码:<input type="password" name="password">  <br>
        确认密码:<input type="password" name="password2"> {{pwd_msg}} <br>
        手机号: <input type="phone" name="phone"> {{phone_msg}}  <br>
        <input type="hidden" name="next" value="{{ next }}"/>
        <p>
            {{err_msg}}
        </p>
        <input type="submit" value="提交">
    </form>
{% endblock %}
```

## 登录

### 视图

```python
class LoginView(View):
    def get(self, request):
		next = request.GET.get('next')
        
        # 判断用户是否登录(属性) user.is_authenticated
        if request.user.is_authenticated:
            if next:
                return redirect(next)
            else:
            	return redirect(reverse('users:login'))

        return render(request, "login.html", context={'next': next})

    def post(self, request):
        username = request.POST.get("username")
        password = request.POST.get("password")
        remembered = request.POST.get("remembered")
        next = request.POST.get('next', '')

        if not all([username, password]):
            # return JsonResponse({"account_msg": "缺少必要参数", "code": 403})
            return render(request, 'login.html', {'account_msg': '缺少必要参数', "next": next})

        # 校验用户信息，成功返回user对象，否则为None
        user = authenticate(username=username, password=password)

        if user is None:
            return render(request, 'login.html', {'account_msg': '用户名或密码错误', "next": next})

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
        if next:
            return redirect(next)
        else:
            return redirect(reverse('users:index'))
```

### 模板

类似注册流程中，有next变量始终保留跳转前的地址，在模板和视图之间传递

```html
{% extends "base.html" %}

{% block body %}


    <form method="post" action="{% url 'users:login' %}">
        {% csrf_token %}
        用户名:<input type="text" name="username">
        密码:<input type="password" name="password">
        记住登录 <input type="checkbox" name="remembered" value='on'>
        <button name="submit">login</button>
        <input type="hidden" name="next" value="{{ next }}"/>
    </form>

{% endblock %}
```

## 注销

#### 视图

```python
class LogoutView(View):

    def get(self, request):
        index = reverse('users:index')
        next = request.GET.get('next', index)
        
        """实现退出登录逻辑"""
        # 清理session
        logout(request)

        # 退出登录，重定向到首页
        response = redirect(next)

        # 退出登录时清除cookie中的username
        response.delete_cookie('username')
        return response
```

## 用户信息

#### 视图

```python
class UserView(View):

    def get(self, requset):
        # 如果用户没有登陆就访问本应登陆才能访问的页面时会直接跳转到登陆页面
        user = requset.user
        if user.is_authenticated:
            return render(requset, 'user.html', context={'user': user})
        else:
            return redirect(reverse('users:login', kwargs={"next": request.path}))

    def post(self, request):
        user = requset.user
        if not user.is_authenticated:
            return redirect(reverse('users:login', kwargs={"next": request.path}))
        
        old_password = request.POST.get("old_password")
        new_password = request.POST.get("new_password")
        conf_password = request.POST.get("conf_password")

        if not all([old_password, new_password, conf_password]):
            return render(request, 'user.html', {'user': user, 'account_msg': '缺少必要参数'})

        if new_password != conf_password:
            return render(request, 'user.html', {'user': user, 'account_msg': '密码不一致'})

        # 校验密码  check_password()
        if not request.user.check_password(old_password):
            return render(request, 'user.html', {'user': user, 'account_msg': '密码不正确'})

        user = request.user

        # 修改密码
        user.set_password(new_password)
        # user.password = make_password(new_password)

        # 保存修改
        user.save()

        # 状态保持
        login(request, user)

        return render(requset, 'user.html', context={'user': user})

```

#### 模板

```html
{% extends 'base.html' %}

{% block body %}
	<p>
       用户名： {{user.username}}
	</p>
	<p>
       邮箱： {{user.email}}
	</p>
	<p>
       手机号： {{user.phone}}
	</p>
    <form action="{% url 'users:user' %}" method="post">
        旧密码: <input type="password" name="old_password">
        新密码: <input type="password" name="new_password">
        确认密码: <input type="password" name="conf_password">
        <button name="submit">确认修改</button>
        <input type="hidden" name="next" value="{% url 'users:index' %}"/>
    </form>
{% endblock %}
```

## 首页

#### 视图

```python
class IndexView(View):

def get(self, request):
    return render(request, 'index.html')
```

#### 模板

```html
{% extends 'base.html' %}

{% block body %}
    <h1>欢迎来到首页</h1>
{% endblock %}
```

## 路由

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