---
title: Django-缓存
date: 2020-12-01 21:49:38
categories:
  - python
  - Django
tags:
  - 缓存
---

## 缓存简介

在动态网站中,用户所有的请求,服务器都会去数据库中进行相应的增,删,查,改,渲染模板,执行业务逻辑,最后生成用户看到的页面.

当一个网站的用户访问量很大的时候,每一次的的后台操作,都会消耗很多的服务端资源,所以必须使用**缓存**来减轻后端服务器的压力.

缓存是将一些常用的数据保存到**内存**或者**memcache**中,在一定的时间内有人来访问这些数据时,则不再去执行数据库及渲染等操作,而是直接从内存或memcache的缓存中去取得数据,然后返回给用户。

## Django缓存

### 缓存方式

>   Django提供了6种缓存方式

-   开发调试缓存
-   内存缓存
-   文件缓存
-   数据库缓存
-   Memcache缓存(使用`python-memcached`模块)
-   Memcache缓存(使用`pylibmc`模块)

经常使用的有**文件缓存**和**Mencache**缓存, 而选用不同的缓存方式，实际就是需要缓存的数据保存的位置不同。只需要修改django项目的配置文件`settings.py`，即可实现不同缓存方式的切换。



#### 开发调试

>   此模式为开发调试使用,实际上不执行任何操作

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',  # 缓存后台使用的引擎
        'TIMEOUT': 300,  # 缓存超时时间（默认300秒，None表示永不过期，0表示立即过期）
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        },
    }
}
```

#### 内存缓存

>   将缓存内容保存至内存区域中

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',  # 指定缓存使用的引擎
        'LOCATION': 'unique-snowflake',  # 写在内存中的变量的唯一值 
        'TIMEOUT': 300,  # 缓存超时时间(默认为300秒,None表示永不过期)
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        }
    }
}
```

#### 文件缓存

>   把缓存数据存储在文件中

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',  # 指定缓存使用的引擎
        'LOCATION': '/var/tmp/django_cache',  # 指定缓存的路径
        'TIMEOUT': 300,  # 缓存超时时间(默认为300秒,None表示永不过期)
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        }
    }
}
```

#### 数据库缓存

>   把缓存数据存储在数据库中

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',  # 指定缓存使用的引擎
        'LOCATION': 'cache_table',  # 数据库表    
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        }
    }
}
```

注意,创建缓存的数据库表使用的语句:

```python
python manage.py createcachetable
```

#### Memcached缓存

Memcached是Django原生支持的缓存系统.要使用Memcached,需要下载Memcached的支持库`python-memcached`或`pylibmc`.

>   使用`python-memcached`模块连接memcache

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',  # 指定缓存使用的引擎
        'LOCATION': '192.168.10.100:11211',  # 指定Memcache缓存服务器的IP地址和端口
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        }
    }
}
```

>   使用`pylibmc`模块连接memcache

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache',  # 指定缓存使用的引擎
        'LOCATION': '192.168.10.100:11211',  # 指定本机的11211端口为Memcache缓存服务器
        'OPTIONS': {
            'MAX_ENTRIES': 300,  # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3,  # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
        },
    }
}

```

Memcached是基于内存的缓存,数据存储在内存中.所以如果服务器死机的话,数据就会丢失,所以Memcached一般与其他缓存配合使用

### 缓存应用

> 在使用缓存之前，必选先选择一种缓存方式，按照上面的配置信息在 `settings.py`中实现对应配置。

#### 全站使用缓存

> 如果想要对整个项目的**所有**`GET`和`HEAD`方法的正常返回结果进行缓存，可以通过**中间件**实现

- 用户的请求通过中间件，经过一系列的认证等操作，如果请求的内容在缓存中存在，则使用`FetchFromCacheMiddleware`获取内容并返回给用户
- 当返回给用户之前，判断缓存中是否已经存在，如果不存在，则`UpdateCacheMiddleware`会将缓存保存至Django的缓存之中，以实现全站缓存
- 请求来了，是从上到下入走中间件；响应的时候是从下到上走中间件。因此，获取页面放在最后，保存页面放最前面

```python
MIDDLEWARE = [
 'django.middleware.cache.UpdateCacheMiddleware',  # 响应HttpResponse中设置几个headers
 'django.middleware.security.SecurityMiddleware',
 'django.contrib.sessions.middleware.SessionMiddleware',
 'django.middleware.common.CommonMiddleware',
 'django.middleware.csrf.CsrfViewMiddleware',
 'django.contrib.auth.middleware.AuthenticationMiddleware',
 'django.contrib.messages.middleware.MessageMiddleware',
 'django.middleware.clickjacking.XFrameOptionsMiddleware',
 'django.middleware.cache.FetchFromCacheMiddleware',  # 用来缓存通过GET和HEAD方法获取的状态码为200的响应
]
 
CACHE__MIDDLEWARE_SECONDS=15  # 设定超时时间为15秒
```

此时在中间件中已经设置过缓存，因此其余代码不变，视图中正常执行逻辑，即可实现全站缓存。

```python
from django.http.response import HttpResponse
import time

def index(request):
    print('视图函数被调用')
    result = 0
    for i in range(10):
        result += i
        time.sleep(0.5)
    ctime = time.strftime('%H:%M:%S', time.localtime())
    return HttpResponse("时间{}的结果是:{}".format(ctime, result))
```



#### 单独视图

>   如果不想对所有的查询结果缓存，而只是为**单个视图**函数添加缓存，就可以使用**装饰器**实现

```python
from django.views.decorators.cache import cache_page
from django.http.response import HttpResponse
import time

@cache_page(10)  # 缓存10秒
def index(request):
    print('视图函数被调用')
    result = 0
    for i in range(10):
        result += i
        time.sleep(0.5)
    ctime = time.strftime('%H:%M:%S', time.localtime())
    return HttpResponse("时间{}的结果是:{}".format(ctime, result))
```

因为缓存的原因,不停的刷新浏览器时会发现：第一次请求页面时，会等待5秒，计算完全部数据数据， 才可以返回结果；在下一次请求时，利用缓存结果，可以很快得到响应数据；直到10秒之后，缓存过期，重新进行该过程。

#### 模板缓存

> 如果并不想对页面的其余部分做任何处理，只是相对页面的**一部分** 进行缓存， 就可以使用 **模板缓存** 实现

如果想对缓存进行更多的控制，可以使用 `cache`模板标签来缓存模板的一个片段。

`views`视图函数

```python
from django.shortcuts import render
import time

def index(request):
    print('视图函数被调用')
    ctime = time.strftime('%H:%M:%S', time.localtime())
    return render(request, 'page.html', {'ctime': ctime})
```

 前端网页

```python
{% load cache %}  # 放在缓存片段的上面
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>模板缓存</title>
</head>
<body>
<p>这里是未缓存的时间:{{ ctime }}</p>

# 将按给定的时间缓存包含块中的内容。 
# 它最少需要两个参数：缓存时间（以秒为单位）；给缓存片段起的名称,该名称将被视为是字符串，而不是变量
# 设定超时时间为10秒, 给缓存模板片段起名为 "time"
{% cache 10 time %}  
    <p>这里是缓存的时间:{{ ctime }}</p>
{% endcache %}
</body>
</html>
```

刷新浏览器可以看到,第一个时间实时变化,后面一个时间每10秒钟变化一次

#### 手动缓存

> 如果不想为一个方法（页面）进行缓存，只想缓存**某个字段**的话，可以 **手动缓存**

```python
from django.core.cache import cache
from django.http.response import HttpResponse
import time


def index(request):
    print('视图函数被调用')
    result = cache.get('result')
    if not result:
        result = 0
        for i in range(10):
            result += i
            time.sleep(0.5)
        cache.set('result', result, 10)

    ctime = time.strftime('%H:%M:%S', time.localtime())
    return HttpResponse("时间{}的结果是:{}".format(ctime, result))
```

## Redis缓存

[参考](https://www.cnblogs.com/believepd/p/10275384.html#二、redis缓存)