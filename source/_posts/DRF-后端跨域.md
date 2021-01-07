---
title: DRF-后端跨域
date: 2020-03-17 18:19:28
categories:
  - 技术
  - python
  - web
tags:
  - DRF
  - 跨域
---

前后端分离开发时，经常会遇到一个问题，那就是`跨域问题`,因为这时候前端和后端的代码是在不同机器上运行的，两个地址不在一个域名下，这个时候前端脚本在进行ajax访问的时候浏览器就会报跨域相关的错误。

原因：浏览器的同源策略不允许跨域访问，所谓同源策略是指协议、域名、端口相同。

为了解决这个问题，在前端和后台配置均可，此时，以后台django配置为例

## 1. 安装插件

```bash
pip install django-cors-headers
```

## 2. 注册应用

```python
INSTALLED_APPS = [
    ...
    'corsheaders',
    ...
]
```

## 3. 中间件设置

```python
MIDDLEWARE = [
    ...
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',  # 注意，位置有限制
    'django.middleware.common.CommonMiddleware',
    ...
]
```

## 4. 配置跨域参数

```python
CORS_ORIGIN_ALLOW_ALL = True  # 允许所有host访问
# 添加白名单
CORS_ORIGIN_WHITELIST = (
    '127.0.0.1:8080',
    'localhost:8080',
)
# 以上配置二选一即可

CORS_ALLOW_CREDENTIALS = True  # 允许携带cookie,不需要可以不设置
```
