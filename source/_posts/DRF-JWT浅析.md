---
title: JWT浅析
date: 2020-10-24 20:38:26
categories:
  - 技术
  - python
  - web
tags:
  - JWT
  - DRF
---

 JWT起源
-------

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于 JSON 的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)). 该 token 被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT 的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该 token 也可直接被用于认证，也可被加密。

说起 JWT，我们应该来谈一谈基于 token 的认证和传统的 session 认证的区别。

### 传统的 session 认证

我们知道，http 协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据 http 协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为 cookie, 以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了, 这就是传统的基于 session 认证。

但是这种基于 session 的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于 session 认证应用的问题就会暴露出来.

#### 基于 session 认证所显露的问题

**Session**: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言 session 都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。

**扩展性**: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上, 这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

**CSRF**: 因为是基于 cookie 来进行用户识别的, cookie 如果被截获，用户就会很容易受到跨站请求伪造的攻击。

### 基于 token 的鉴权机制

基于 token 的鉴权机制类似于 http 协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于 token 认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

流程上是这样的：

*   用户使用用户名密码来请求服务器
*   服务器进行验证用户的信息
*   服务器通过验证发送给用户一个 token
*   客户端存储 token，并在每次请求时附送上这个 token 值
*   服务端验证 token 值，并返回数据

这个 token 必须要在每次请求时传递给服务端，它应该保存在请求头里， 另外，服务端要支持`CORS(跨来源资源共享)`策略，一般我们在服务端这么做就可以了`Access-Control-Allow-Origin: *`。

那么我们现在回到 JWT 的主题上。

JWT 构成
---------

JWT 是由三段信息构成的，将这三段信息文本用`.`链接一起就构成了 Jwt 字符串。就像这样:

```python
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

第一部分我们称它为头部（header), 第二部分我们称其为载荷（payload, 类似于飞机上承载的物品)，第三部分是签证（signature).

### header（头信息）

jwt 的头部承载两部分信息：

*   令牌类型（即：JWT）
*   散列算法（HMAC、RSASSA、RSASSA-PSS等）

完整的头部就像下面这样的 JSON：

```python
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

然后将头部进行 base64 加密（该加密是可以对称解密的), 构成了第一部分.

```python
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

### Payload（有效载荷）

载荷就是存放有效信息的地方，其中包含claims。claims是关于实体（常用的是用户信息）和其他数据的声明，claims有三种类型：

- **Registered claims（注册的声明）：** 这些是一组预定义的claims，非强制性的，但是推荐使用， iss（发行人）， exp（到期时间）， sub（主题）， aud（观众）等；
- **Public claims（公共的声明）:** 自定义claims，注意不要和JWT注册表中属性冲突，[这里可以查看JWT标准注册表](https://www.iana.org/assignments/jwt/jwt.xhtml)
- **Private claims（私有的声明）:** 这些是自定义的claims，用于在同意使用这些claims的各方之间共享信息，它们既不是Registered claims，也不是Public claims。

#### 标准中注册的声明

> **建议但不强制使用**

*   **iss**: jwt 签发者
*   **sub**: jwt 所面向的用户
*   **aud**: 接收 jwt 的一方
*   **exp**: jwt 的过期时间，这个过期时间必须要大于签发时间，注意，这个值是秒数，而不是毫秒数。
*   **nbf**: 定义在什么时间之前，该 jwt 都是不可用的.
*   **iat**: jwt 的签发时间
*   **jti**: jwt 的唯一身份标识，主要用来作为一次性 token, 从而回避重放攻击。
*   **name**：用户全名

#### 公共的声明

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息. 但不建议添加敏感信息，因为该部分在客户端可解密。

#### 私有的声明 

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为 base64 是对称解密的，意味着该部分信息可以归类为明文信息。

> 在官网有详细的属性说明，尽量使用里面提到的 *Registered Claim Names*，这样可以提高阅读性

自定义一个 payload:

```python
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后将其进行 base64 加密，得到 Jwt 的第二部分。

```python
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

### signature

jwt 的第三部分是一个签证信息，这个签证信息由三部分组成：

*   header (base64 后的)
*   payload (base64 后的)
*   secret

这个签名的计算跟第一部分中的 alg 属性有关，假如是 HS256，那么服务端需要保存一个私钥，比如 secret 。然后，把第一部分和第二部分生成的两个字符串用 `.` 连接之后，用 HS256 进行加盐`secret`加密，然后就构成了 jwt 的第三部分。

```python
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

将这三部分用`.`连接成一个完整的字符串, 构成了最终的 jwt:

```python
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

> **注意：secret 是保存在服务器端的，jwt 的签发生成也是在服务器端的，secret 就是用来进行 jwt 的签发和 jwt 的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个 secret, 那就意味着客户端是可以自我签发 jwt 了。**

## 如何应用

一般是在请求头里加入`Authorization`，并加上`JWT`标注：

```
fetch('api/user/1', {
  headers: {
    'Authorization': 'JWT ' + token
  }
})
```

服务端会验证 token，如果验证通过就会返回相应的资源。整个流程就是这样的:

![jwt-diagram](https://gitee.com/liushaofeng2018/imgs/raw/master/uPic/2020/10/1821058-2e28fe6c997a60c9.png) 



###  示例1/2：以 Django 作为服务端

#### 环境搭建

```pythhon
pip install django  django-cors-headers djangorestframework djangorestframework-jwt
```

#### 用户模型类

> `users/models.py`

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class UserModel(AbstractUser):
    phone = models.CharField(max_length=11, unique=True, verbose_name='手机')

    class Meta:
        ordering = ['id']
        db_table = 'db_user'
        verbose_name = '用户'
        verbose_name_plural = '用户'

    def __str__(self):
        return self.username
```

#### 序列化类

> `users/ser.py`

```python
from rest_framework import serializers
from .models import UserModel
from django.contrib.auth.hashers import make_password
import re


class UserSerializer(serializers.ModelSerializer):
    def create(self, validated_data):
        return UserModel.objects.create_user(**validated_data)

    def update(self, instance, validated_data):
        instance.username = validated_data.get('username', instance.username)
        instance.phone = validated_data.get('phone', instance.phone)
        instance.email = validated_data.get('email', instance.email)

        password = validated_data['password']
        if not password:
            instance.password = make_password(password)

        instance.save()

        return instance

    def validate_password(self, value):
        """
        验证密码不能全是小写字母，不能全是大写字母，也不能全是数字
        """
        if not re.match(r'(?!^\d*$)(?![a-z]$)(?![A-Z]$).{6,}$', value):
            raise serializers.ValidationError('密码等级不够')
        return value

    def validate_phone(self, value):
        """
        校验手机号是否合法
        """
        if not re.match(r'1[3-7]\d{9}$', value):
            raise serializers.ValidationError('手机号不合法')
        return value

    class Meta:
        model = UserModel
        fields = ('id', 'username', 'phone', 'password', 'email')
        read_only_fields = ('id',)
        extra_kwargs = {
            'password': {
                'write_only': True
            }
        }
```

#### 自定义JWT payload 

> `users/utils.py`

```python
def jwt_response_payload_handler(token, user=None, request=None):
    """
    自定义jwt认证成功返回数据
    """
    return {
        'token': token,
        'id': user.id,
        'username': user.username,
        'phone': user.phone,
        'email': user.email,
    }

```

#### 自定义Django登录验证

> `users/utils.py`

```python
from .models import UserModel
from django.contrib.auth.backends import ModelBackend

class JWTModelBackend(ModelBackend):
    """user验证"""

    def authenticate(self, request, username=None, password=None, **kwargs):
        try:
            if re.match(r'^1[3-9]\d{9}$', username):
                user = UserModel.objects.get(phone=username)
            else:
                user = UserModel.objects.get(username=username)
        except UserModel.DoesNotExist:
            return None
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
```

#### 项目配置信息

> `djangodemo/settings.py`

```python
INSTALLED_APPS = [
   	...,
    'corsheaders',
    'users'
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 跨域参数
CORS_ORIGIN_ALLOW_ALL = True

# 自定义用户模型类
AUTH_USER_MODEL = 'users.UserModel'

# 多种方式登录验证
AUTHENTICATION_BACKENDS = ['users.utils.JWTModelBackend']

REST_FRAMEWORK = {
    # 指定drf认证机制
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # 默认JWT认证
        # 'rest_framework.authentication.SessionAuthentication',  # session认证
        # 'rest_framework.authentication.BasicAuthentication',  # 基本认证
    ),
  
  	# 全局权限配置
  	'DEFAULT_PERMISSION_CLASSES': (
         'rest_framework.permissions.IsAuthenticated',
     )
}

import datetime

JWT_AUTH = {
    # 指明token的有效期， 默认5分
    'JWT_EXPIRATION_DELTA': datetime.timedelta(minutes=5),
    'JWT_ALLOW_REFRESH': True,
    # 在多久间隔内可以用它来刷新以便获取新的token，默认是7天
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),
    # 客户端首先调用obtain_jwt_token进行登录操作，
    # 之后必须每隔小于5分钟就刷新一次token，才能保证不掉线。
    # 然而即使一直保持在线，上限也只有7天，7天过后必须重新登录，这才是5mins + 7days的确切含义

    # 自定义jwt认证成功返回数据
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'users.utils.jwt_response_payload_handler',
}
```

#### 权限验证

> 权限控制可以限制用户对于视图的访问和对于具体数据对象的访问。
>
> - 在项目的配置文件中，实现全局的权限配置
> - 在视图类中指定权限

权限分为四类：

- `AllowAny`: 允许所有用户， 默认权限
- `IsAuthenticated`: 仅通过认证的用户
- `IsAdminUser`: 仅管理员用户
- `IsAuthenticatedOrReadOnly`: 认证的用户可以完全操作，否则只能`get`读取

#### 创建视图

> `users/views.py`

```python
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import CreateModelMixin
from users.ser import UserSerializer, UserModel


# 创建注册视图类
class RegisterView(GenericAPIView, CreateModelMixin):
    queryset = UserModel.objects.all()
    serializer_class = UserSerializer

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

# rest_framework_jwt 已经提供登录签发JWT的视图函数：obtain_jwt_token

# rest_framework_jwt 提供在有效期内刷新token的视图函数：refresh_jwt_token

# 创建允许管理员查询所有用户的视图类
class UserView(GenericAPIView, ListModelMixin):
  	"""查询所有注册用户"""
    queryset = UserModel.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)  # 指定权限验证,限制管理员才可以查询所有用户

    def get(self, request, *args, **kwrags):
        return self.list(request, *args, **kwrags)


```

#### 路由配置

> `djangodemo/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('users/', include('users.urls'))
]
```

> `users/urls.py`

```python
from django.urls import path
from .views import *
from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token

urlpatterns = [
    path('register/', RegisterView.as_view()), # 注册路由
    path('login/', obtain_jwt_token), # 登录路由
    path('refresh/', refresh_jwt_token), # 刷新token
    path('', UserView.as_view()) # 权限验证，查询所有用户
]
```

### 示例 2/2：以 Vue 作为前端

#### 路由前置守卫

> `router/index.js`

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Login from '@/components/Login'
import Users from '@/components/Users'

Vue.use(Router);

let router = new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/login',
      name: 'Login',
      component: Login
    },
    {
      path: '/Users',
      name: 'Users',
      component: Users
    }
  ]
})

router.beforeEach((to, from, next) => {
  const isLogin = localStorage.getItem('token') ? true : false;
  if (to.path == '/login' || to.path == '/register') {
    //'login'和'register'相当于是路由白名单
    localStorage.setItem("preRoute", router.currentRoute.fullPath);
    next();
  } else {
    //如果token存在，就正常跳转，如果不存在，则说明未登陆，则跳转到'login'
    isLogin ? next() : next("/login");
  }
})

export default router
```

#### axios拦截器

> `utils/axios.js`

```javascript
import axios from 'axios'
import router from '../router/index.js'

const instance = axios.create({
  baseURL: 'http://127.0.0.1:8000/',
  timeout: 10000,
});

// 添加请求拦截器
instance.interceptors.request.use(function (config) {
  // 在发送请求之前, 添加 token 到请求头
  if (localStorage.token) {
    config.headers['Authorization'] = 'JWT ' + localStorage.getItem('token');
    config.headers['Accept'] = 'application/json';
  }
  return config;
}, function (error) {
  // 对请求错误做些什么
  return Promise.reject(error);
});

// 自定义的 axios 响应拦截器
instance.interceptors.response.use((response) => {
  // 判断一下响应中是否有 token，如果有就直接使用此 token 替换掉本地的 token。你可以根据你的业务需求自己编写更新 token 的逻辑
  var token = response.data.token;

  if (token) {
    localStorage.setItem('token', token);
  }
  return response
}, (error) => {
  if (error.response) {
    switch (error.response.status) {
      case 401:
        // 这里写清除token的代码
        console.log("401")
        localStorage.removeItem('token');
        /* 普通401拦截直接返回到登录页面 */
        router.push('/login');
    }
  }
  return Promise.reject(error)
});

export default instance
```

#### 登录页面

```html
<template>
  <div>
    用户名: <input type="text" v-model="userInfo.username"/> <br>
    密码: <input type="text" v-model="userInfo.password"/> <br>
    <button @click="login">登录</button>
  </div>
</template>

<script>
export default {
  name: "login",
  data() {
    return {
      userInfo: {
        username: "",
        password: "",
      },
    };
  },
  methods: {
    login() {
      this.$axios
        .post("users/login/", this.userInfo)
        .then((resp) => {
          const curr = localStorage.getItem('preRoute');
          if (curr == null) {
            this.$router.push({path: "/user_center"});
          } else {
            this.$router.push({path: curr});
          }
          this.$router.push({path: decodeURIComponent(url)});
        })
        .catch((err) => {
        });
    },
  },
};
</script>
```

#### 用户展示页面

```html
<template>
  <div>
    <table>
      <tr v-for="user in userList">
        <td> {{ user.id }}</td>
        <td> {{ user.username }}</td>
        <td> {{ user.phone }}</td>
      </tr>
    </table>
  </div>

</template>

<script>
export default {
  data() {
    return {
      userList: ""
    }
  },
  methods: {
    getData() {
      this.$axios.get('/users/')
        .then(resp => {
          console.log(resp.data)
          this.userList = resp.data
        })
    }
  },
  mounted() {
    this.getData();
  }
}
</script>
```



总结
--

### 优点

*   因为 json 的通用性，所以 JWT 是可以进行跨语言支持的，像 JAVA,JavaScript,NodeJS,Python 等很多语言都可以使用。
*   因为有了 payload 部分，所以 JWT 可以在自身存储一些其他业务逻辑所必要的非敏感信息。
*   便于传输，jwt 的构成非常简单，字节占用很小，所以它是非常便于传输的。
*   它不需要在服务端保存会话信息, 所以它易于应用的扩展。

### 安全相关

*   不应该在 jwt 的 payload 部分存放敏感信息，因为该部分是客户端可解密的部分。
*   保护好 secret 私钥，该私钥非常重要。
*   如果可以，请使用 https 协议