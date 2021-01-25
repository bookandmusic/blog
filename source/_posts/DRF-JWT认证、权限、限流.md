---
title: DRF-JWT认证、权限、限流
date: 2020-03-15 23:22:25
categories:
  - 技术
  - python
  - DRF
tags:
  - JWT
  - 权限
  - 认证
  - 限流
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



## 以 Django 作为服务端

### 环境搭建

```python
pip install django  django-cors-headers djangorestframework djangorestframework-jwt
```

### 项目配置信息

> `djangodemo/settings.py`

```python
INSTALLED_APPS = [
   	...,
    'rest_framework',
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

# 跨域参数,允许所有源访问
CORS_ORIGIN_ALLOW_ALL = True

# 自定义用户模型类
AUTH_USER_MODEL = 'users.UserModel'
```

### 用户模型类

> `users/models.py`

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class UserModel(AbstractUser):
    """继承django内置的用户类，并重写，注意：必须在settings.py中指明"""
    phone = models.CharField(max_length=11, unique=True, verbose_name='手机')

    class Meta:
        ordering = ['id']
        db_table = 'db_user'
        verbose_name = '用户'
        verbose_name_plural = '用户'

    def __str__(self):
        return self.username
```

### 序列化类

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

### 注册

#### 视图

>   `users/views.py`

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
```

#### 路由

```python
from django.urls import path
from .views import *

urlpatterns = [
    path('register/', RegisterView.as_view()), # 注册路由  
]
```

### 登录

#### 自定义Django登录验证

默认django只支持用户名和密码登录，想要支持多种方式登录，必须重写认证类

> `users/utils.py`

```python
from .models import UserModel
from django.contrib.auth.backends import ModelBackend

class UserModelBackend(ModelBackend):
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

#### 自定义返回数据

> 登录成功，需要生成JWT，返回给客户端，之后的认证通过生成的jwt实现

```python 
# rest_framework_jwt插件已经内置了登录视图，登录成功，返回JWT,注意，默认登录成功只会返回token，如果想要其他用户信息，必须自定义返回数据
rest_framework_jwt.views.obtain_jwt_token 

# rest_framework_jwt插件已经内置了刷新jwt视图，在 jwt失效之前，返回新的jwt
rest_framework_jwt.views.refresh_jwt_token
```

> `users/utils.py`

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

#### 配置信息

```python
import datetime

# 多种方式登录验证
AUTHENTICATION_BACKENDS = ['users.utils.UserModelBackend']

# JWT配置信息
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

#### 路由

```python
from django.urls import path
from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token

urlpatterns = [
    path('login/', obtain_jwt_token), # 登录路由
    path('refresh/', refresh_jwt_token), # 刷新token
]
```

### 认证

当客户端请求时，会携带上一步生成的`token`，因此，需要在 服务端验证`token`，从而判断是否允许进行下一步操作

#### 全局配置

```python
REST_FRAMEWORK = {
    # 指定drf认证机制
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # JWT认证
        # 'rest_framework.authentication.SessionAuthentication',  # session认证
        # 'rest_framework.authentication.BasicAuthentication',  # 基本认证
    ]
}
```

#### 局部配置

可以在具体的视图中通过`authentication_classes`属性来设置

```python
# 个人信息    
class UserInfoAPIView(APIView):
    authentication_classes = [TokenAuthentication]  # 指明认证类

    def get(self, request):
        user = request.user  # 只要认证成功，请求对象中就会存在 user对象
        ser = UserInfoSerializer(user)
        return Response(ser.data)
```

### 权限

上面的视图，仍然存在一个问题：客户端发送jwt，服务器可以正常解析出登录用户；但是，如果没有发送jwt，再通过`request.user`获取用户对象，就会出错。因此，需要对该功能进行权限验证，只有登录用户才可以访问，否则，就拒绝访问

总而言之，权限控制可以限制用户对于视图的访问和对于具体数据对象的访问。

在执行视图的`dispatch()`方法前，会先进行视图访问权限的判断
在通过`get_object()`获取具体对象时，会进行对象访问权限的判断

权限分为四类：

- `AllowAny`: 允许所有用户， 默认权限
- `IsAuthenticated`: 仅通过认证的用户
- `IsAdminUser`: 仅管理员用户
- `IsAuthenticatedOrReadOnly`: 认证的用户可以完全操作，否则只能`get`读取

#### 全局配置

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

#### 局部配置

可以在具体的视图中通过`permission_classes`属性来设置

```python
# 个人信息    
class UserInfoAPIView(APIView):
    authentication_classes = [JSONWebTokenAuthentication] # 指明认证类
    permission_classes = [IsAuthenticated]  # 指明只有认证用户可以访问

    def get(self, request):
        user = request.user  # 只要认证成功，且权限校验通过，请求对象中就一定会存在 user对象
        ser = UserInfoSerializer(user)
        return Response(ser.data)
    
# 创建允许管理员查询所有用户的视图类
class UserView(GenericAPIView, ListModelMixin):
  	"""查询所有注册用户"""
    queryset = UserModel.objects.all()
    serializer_class = UserSerializer
    authentication_classes = [JSONWebTokenAuthentication] # 指明认证类
    permission_classes = [IsAdminUser]  # 指定权限验证,限制管理员才可以查询所有用户

    def get(self, request, *args, **kwrags):
        return self.list(request, *args, **kwrags)
```



### 限流Throttling

可以对接口访问的频次进行限制，以减轻服务器压力。特别是限制爬虫的抓取。

#### 针对用户进行限制

>   可以在配置文件中，使用`DEFAULT_THROTTLE_CLASSES` 和 `DEFAULT_THROTTLE_RATES`进行全局配置

##### 全局配置

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

##### 局部配置

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

#### 针对视图限制

视图中使用`throttle_scope`属性设置具体配置信息

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

在项目配置文件中设置具体频率

```python
'DEFAULT_THROTTLE_CLASSES': (
    # 限制用户对于每个视图的访问频次，使用ip或user id
    'rest_framework.throttling.ScopedRateThrottle',
),
'DEFAULT_THROTTLE_RATES': {
    'downloads': '3/minute'
}
```

## 以 Vue 作为前端

#### 路由前置守卫

> `router/index.js`

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import Login from '@/components/Login'
import Users from '@/components/Users'

Vue.use(Router);

let router = new Router({
  routes: [
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

> `main.js`

```javascript
import Vue from 'vue'
import App from './App'
import router from './router'
import axios from 'axios'

axios.defaults.baseURL = 'http://127.0.0.1:8000/'

// 添加请求拦截器
axios.interceptors.request.use(function (config) {
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
axios.interceptors.response.use((response) => {
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

Vue.prototype.$axios = axios
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
      <tr>
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
      user: ""
    }
  },
  methods: {
    getData() {
      this.$axios.get('/userinfo/')
        .then(resp => {
          console.log(resp.data)
          this.user = resp.data
        })
    }
  },
  mounted() {
    this.getData();
  }
}
</script>
```
