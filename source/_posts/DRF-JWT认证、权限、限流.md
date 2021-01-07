---
title: DRF-JWT认证、权限、限流
date: 2020-03-15 23:22:25
categories:
  - 技术
  - python
  - web
tags:
  - JWT
  - 权限
  - 认证
  - 限流
  - DRF
---
## 用户模型类相关

### 模型类的定义

在此，我们借用django自带的用户认证系统实现目的

- 在`models.py`中定义用户类

    ```python
    from django.db import models
    from django.contrib.auth.models import AbstractUser

    class User(AbstractUser):
        phone = models.CharField(max_length=11, unique=True, verbose_name="手机号")

        class Meta:
            db_table = "tb_users"
            verbose_name = "用户"
            verbose_name_plural = "用户"

        def __str__(self):
            return self.username
    ```

### 系统模型类的指定

- 在项目的配置文件`settings.py`中指定

    ```python
    # AUTH_USER_MODEL = '应用名.模型类名'
    AUTH_USER_MODEL = 'users.user'
    ```

### 序列化器的实现

- 在 `serializer.py`中定义用户类对应的序列化器

    ```python
    from django.contrib.auth.hashers import make_password
    from rest_framework.serializers import ModelSerializer
    from .models import User

    class UserSerializer(ModelSerializer):
    
        def create(self, validated_data):
            # 重写创建用户方法，将密码进行加密
            validated_data["password"] = make_password(validated_data["password"])
            super().create(validated_data)
    
            return validated_data
    
        class Meta:
            model = User
            fields = ('id', 'username', 'password', 'phone')
            read_only_fields = ('id',)
    ```

## JWT

在用户注册或登录后，我们想记录用户的登录状态，或者为用户创建身份认证的凭证。我们不再使用Session认证机制，而使用JWT(Json Web Token)认证机制

### 安装配置

- 安装插件

    ```bash
    pip install djangorestframework-jwt django-restframework
    ```

- 重写JWT响应处理函数

    在应用users中新建一个`utils.py`文件

    ```python
    def jwt_response_payload_handler(token, user=None, request=None):
        """
        自定义jwt认证成功返回数据
        """
        return {
            'token': token,
            'id': user.id,
            'username': user.username
        }
    ```

- 配置JWT认证机制

    ```python
    REST_FRAMEWORK = {
        # 指定drf认证机制
        'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework_jwt.authentication.JSONWebTokenAuthentication', # JWT认证
            'rest_framework.authentication.SessionAuthentication', # session认证
            'rest_framework.authentication.BasicAuthentication', # 基本认证
        ),
    }

    JWT_AUTH = {
        # 指明token的有效期
        'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
        # 自定义jwt认证成功返回数据
        'JWT_RESPONSE_PAYLOAD_HANDLER': 'apps.users.utils.jwt_response_payload_handler',
    }
    ```

### 注册登录

- 定义视图

    > 注意：登录视图，使用JWT默认视图即可，只需实现注册视图

    ```python
    from rest_framework.generics import GenericAPIView
    from rest_framework.mixins import CreateModelMixin
    from .serializer import UserSerializer, User

    # Create your views here.
    class RegisterView(CreateModelMixin, GenericAPIView):
        queryset = User
        serializer_class = UserSerializer
    
        def post(self, request, *args, **kwargs):
            return self.create(request, *args, **kwargs)
    ```

- 配置路由

    ```python
    from django.urls import path
    from .views import RegisterView
    from rest_framework_jwt.views import obtain_jwt_token

    urlpatterns = [
        path('register/', RegisterView.as_view()),
        path('login/', obtain_jwt_token),
    ]
    ```

### 权限Permissions

权限控制可以限制用户对于视图的访问和对于具体数据对象的访问。

在执行视图的dispatch()方法前，会先进行视图访问权限的判断
在通过get_object()获取具体对象时，会进行对象访问权限的判断

1. 全局配置

    可以在项目的配置文件`settings.py`中设置默认的权限管理类

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': (
            'rest_framework.permissions.IsAuthenticated',
        )
    }
    ```

    如果未指明，则采用如下默认配置

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': (
            'rest_framework.permissions.AllowAny',
        )
    }
    ```

2. 局部配置

    可以在具体的视图中通过permission_classes属性来设置，如

    ```python
    from rest_framework.generics import GenericAPIView
    from rest_framework.mixins import ListModelMixin
    from rest_framework.permissions import IsAuthenticated

    from .serializer import UserSerializer, User

    class UserView(GenericAPIView, ListModelMixin):
        queryset = User.objects.all()
        serializer_class = UserSerializer
        permission_classes = (IsAuthenticated,)

        def get(self, request, *args, **kwargs):
            return self.list(request, *args, **kwargs)
    ```

3. 提供的权限

    - `AllowAny`: 允许所有用户
    - `IsAuthenticated`: 仅通过认证的用户
    - `IsAdminUser`: 仅管理员用户
    - `IsAuthenticatedOrReadOnly`: 认证的用户可以完全操作，否则只能get读取

### 限流Throttling

可以对接口访问的频次进行限制，以减轻服务器压力。特别是限制爬虫的抓取。

> 可以在配置文件中，使用DEFAULT_THROTTLE_CLASSES 和 DEFAULT_THROTTLE_RATES进行全局配置

1. 针对用户进行限制

    - 全局配置

        ```python
        REST_FRAMEWORK = {
            'DEFAULT_THROTTLE_CLASSES': (
                # 限制所有匿名未认证用户，使用IP区分用户
                'rest_framework.throttling.AnonRateThrottle',
                # 限制认证用户，使用User id 来区分
                'rest_framework.throttling.UserRateThrottle'
            ),
            'DEFAULT_THROTTLE_RATES': {
                # 可以使用 second, minute, hour 或day来指明周期
                'anon': '3/minute',
                'user': '5/minute'
            }
        }
        ```

    - 局部配置

        视图中使用`throttle_classes`属性设置限流用户类型

        ```python
        from rest_framework.generics import GenericAPIView
        from rest_framework.mixins import ListModelMixin
        from rest_framework.throttling import UserRateThrottle, AnonRateThrottle

        from .serializer import UserSerializer, User

        class UserView(GenericAPIView, ListModelMixin):
            queryset = User.objects.all()
            serializer_class = UserSerializer
            throttle_classes = (UserRateThrottle, AnonRateThrottle)

            def get(self, request, *args, **kwargs):
                return self.list(request, *args, **kwargs)
        ```

        在项目配置文件中针对用户类型设置具体频率

        ```python
        REST_FRAMEWORK = {
            'DEFAULT_THROTTLE_RATES': {
                # 可以使用 second, minute, hour 或day来指明周期
                'anon': '3/minute',
                'user': '5/minute'
            }
        }
        ```

2. 针对视图限制

    - 视图中使用`throttle_scope`属性设置具体配置信息

        ```python
        from rest_framework.generics import GenericAPIView
        from rest_framework.mixins import ListModelMixin
        from rest_framework.permissions import IsAuthenticated

        from .serializer import UserSerializer, User

        class UserView(GenericAPIView, ListModelMixin):
            queryset = User.objects.all()
            serializer_class = UserSerializer
            throttle_scope = 'downloads'

            def get(self, request, *args, **kwargs):
                return self.list(request, *args, **kwargs)
        ```

    - 在项目配置文件中设置具体频率

        ```python
        'DEFAULT_THROTTLE_CLASSES': (
            # 限制用户对于每个视图的访问频次，使用ip或user id
            'rest_framework.throttling.ScopedRateThrottle',
        ),
        'DEFAULT_THROTTLE_RATES': {
            'downloads': '3/minute'
        }
        ```

