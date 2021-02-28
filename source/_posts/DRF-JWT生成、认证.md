---
title: JWT生成、认证
date: 2021-02-27 23:22:33
categories:
  - 技术
  - python
  - DRF
tags:
  - JWT生成
  - 认证
---

DRF框架的一系列功能：认证、权限、限流，都是依赖于JWT。

整个流程就是这样的:

-   客户端发送用户名和密码到服务端，
-   验证通过，生成JWT
-   返回JWT给客户端

-   下次，客户端发送请求时，携带JWT，一般是在请求头里加入`Authorization`，并加上`JWT`标注：

```js
{
  headers: {
    'Authorization': 'JWT ' + token
}
```

-   服务端会验证 token，如果验证通过就会返回相应的资源
-   在此基础上，可以实现权限和限流

流程图如下：

![jwt-diagram](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020/10/1821058-2e28fe6c997a60c9.png) 

而目前，在DRF项目开发过程中，token的生成、认证方式，主要有两种方式：`pyjwt`和`rest_framework_jwt`

## pyjwt

### 配置信息

```python
import datetime 

JWT_EXPIRATION_DELTA = datetime.timedelta(seconds=300)  # 指明jwt的过期时间

REST_FRAMEWORK = { # 可以全局配置，也可以局部配置
    # 指定视图权限
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.AllowAny',  # 默认每个视图，是只有认证用户可以访问
    ),
    # 指定drf认证机制
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'user.tools.auth.JSONWebTokenAuthentication',  # 指明 jwt认证类
        'rest_framework.authentication.SessionAuthentication',  # session认证
        'rest_framework.authentication.BasicAuthentication',  # 基本认证
    ),
}
```

### 工具类

自定义方法，生成token，解析token

```python
from datetime import datetime

import jwt
from django.conf import settings


def jwt_encode_handler(user):
   	"""
    :param user: 用户对象
    :return: 生成的token
    """
    payload = {
        'user_id': user.pk,
        'username': user.username,
        'exp': datetime.utcnow() + settings.JWT_EXPIRATION_DELTA
    }
    token = jwt.encode(payload=payload, key=settings.SECRET_KEY).decode('utf-8')

    return token


def jwt_decode_handler(token):
    """
    :param token: 生成的token
    :return: 解析后的数据
    """
    data = jwt.decode(token, key=settings.SECRET_KEY, algorithms='HS256')

    return data

```

自定义用户认证类

```python
from django.contrib.auth import get_user_model
from rest_framework.authentication import BaseAuthentication
from user.tools.jwt import jwt_decode_handler

USER = get_user_model()


class JSONWebTokenAuthentication(BaseAuthentication):
    def authenticate(self, request):
        token = request.META.get('HTTP_AUTHORIZATION', '')

        try:
            data = jwt_decode_handler(token)
        except Exception:
            return None, None

        user_id = data.get('user_id')
        try:
            user = USER.objects.get(id=user_id)
        except USER.DoesNotExist:
            return None, None

        return user, None
```

### 生成token

#### 视图

登录时，调用`authenticate`方法，认证用户，根据用户,自己生成jwt

```python
from django.contrib.auth import authenticate
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

from user.tools.jwt import jwt_encode_handler


# Create your views here.
# rest framework jwt
class LoginAPIView(APIView):
    def post(self, request):
        # 1. 获取用户信息
        username = request.data.get('username')
        password = request.data.get('password')

        # 2. 认证用户
        user = authenticate(request, username=username, password=password)

        # 3. 判断用户是否认证通过
        if user:
            # 使用pyjwt， 生成jwt
            token = jwt_encode_handler(user)

            return Response({
                'user': user.username,
                'token': token
            })

        else:
            return Response({'msg': '登录失败'}, status=400)
```

#### 路由

```python
from django.urls import path
from user.views import LoginAPIView

urlpatterns = [
    path('login/', LoginAPIView.as_view())
]

```

### 认证token

>   请求时，需要在请求头中添加参数 ,格式为： `{'Authorization' : token}`

#### 视图

用户认证时，自己校验jwt，重写认证类

```python
from django.contrib.auth import authenticate
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

from user.tools.auth import JSONWebTokenAuthentication


class UserInfoAPIView(APIView):
    # 针对视图类，局部配置认证类和权限类
    authentication_classes = [JSONWebTokenAuthentication]
    permission_classes = [IsAuthenticated]

    def get(self, request):
        user = request.user
        return Response({
            'username': user.username,
            'email': user.email,
        })

```

#### 路由

```python
from django.urls import path
from user.views import UserInfoAPIView

urlpatterns = [
    path('userinfo/', UserInfoAPIView.as_view())
]
```

## rest_framework_jwt

### 配置信息

```python
# 针对 drf 的配置信息， 全局配置 drf的视图的认证和权限
REST_FRAMEWORK = {
    # 指定视图权限
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.AllowAny',  # 默认每个视图，允许任何用户访问
    ),
    # 指定drf认证机制
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # rest_framework_jwt认证， 也可以在每个视图中指明认证类
    )
}

# 针对 rest_framework_jwt 的配置信息
JWT_AUTH = {
    'JWT_RESPONSE_PAYLOAD_HANDLER':
    	# 'rest_framework_jwt.utils.jwt_response_payload_handler', # 默认jwt认证成功返回数据 
      	'user.utils.jwt_response_payload_handler', # 自定义jwt认证成功返回数据
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),  # 指明token的有效期， 默认5分
    'JWT_ALLOW_REFRESH': True, # 允许刷新
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),  # 在多久间隔内可以用旧token来刷新以便获取新的token，默认是7天
}
```

### 生成token

#### 默认方式

登录时，调用 封装好的 `obtain_jwt_token`，默认返回 `{'token': token}`; 如果还想要返回其他数据，需要自定义响应返回事件

##### 自定义响应

```python
def jwt_response_payload_handler(token, user=None, request=None):
    return {
        'token': token,
        'username': user.username
    }

# 自定义响应之后，需要在 rest_framework_jwt 的配置信息中 指明响应事件
```

##### 视图类

```python
# 登录视图，不需要自己实现
```

##### 路由

```python
from django.urls import path
from rest_framework_jwt.views import obtain_jwt_token

urlpatterns = [
    path('login/', obtain_jwt_token),
]
```

#### 自定义token

有时可能需要手动生成令牌，可以调用内置的方法 `jwt_payload_handler`  和 `jwt_encode_handler`来产生token

##### 视图类

```python
from django.contrib.auth import authenticate
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.settings import api_settings

jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

# 自定义登录视图
class LoginAPIView(APIView):
    def post(self, request):
        # 1. 获取用户信息
        username = request.data.get('username')
        password = request.data.get('password')

        # 2. 认证用户
        user = authenticate(request, username=username, password=password)

        # 3. 判断用户是否认证通过
        if user:
            # 生成jwt
            payload = jwt_payload_handler(user)
            token = jwt_encode_handler(payload)
       
            return Response({
                'user': user.username,
                'token': token
            })

        else:
            return Response({'msg': '登录失败'}, status=400)
```

##### 路由

```python
from django.urls import path
from user.views import LoginAPIView

urlpatterns = [
    path('login/', LoginAPIView.as_view())
]
```

### 认证token

>   请求时，需要在请求头中添加参数 ,格式为： `{'Authorization' : 'JWT' + ' ' + token}`

用户认证时，直接调用`rest_framework_jwt`内置的 `JSONWebTokenAuthentication` 认证类即可

#### 视图类

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework_jwt.authentication import JSONWebTokenAuthentication


# 个人中心
class UserInfoAPIView(APIView):
    # 认证类和权限类可以全局指明，也可以局部指定
    authentication_classes = [JSONWebTokenAuthentication] # token认证
    permission_classes = [IsAuthenticated] # 权限限制，只有认证用户可以访问

    def get(self, request):
        user = request.user
        return Response({
            'username': user.username,
            'email': user.email,
        })
```

#### 路由

```python
from django.urls import path
from user.views import UserInfoAPIView

urlpatterns = [
    path('userinfo/', UserInfoAPIView.as_view())
]

```

### 刷新token

当token将要过期时，允许客户端提供旧token,换取新token；当然刷新token之前，需要在 配置中指明

#### 视图

```python
# 当然，刷新token的视图，也不需要自己实现;但是，客户端需要使用post方式提交 {"token": token}来换取新token
```

#### 路由

```python
from django.urls import path
from rest_framework_jwt.views import refresh_jwt_token

urlpatterns = [
    path('refresh/', refresh_jwt_token),
]
```

