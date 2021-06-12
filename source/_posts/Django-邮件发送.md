---
title: Django-邮件发送
date: 2020-12-03 22:07:31
categories:
  - 技术
  - python
  - Django
tags:
  - 邮件
---

我们常常会用到一些发送邮件的功能，比如：有人注册网站之后，需要向其邮箱中发送激活链接，只有点击激活链接，激活账户之后，才允许登录。

## 配置相关参数

在 settings.py 的最后面加上类似这些

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_USE_TLS = False   # 是否使用TLS安全传输协议(用于在两个通信应用程序之间提供保密性和数据完整性。)
EMAIL_USE_SSL = True    # 是否使用SSL加密，qq企业邮箱要求使用
EMAIL_HOST = 'smtp.163.com'   # 发送邮件的邮箱 的 SMTP服务器，这里用了163邮箱
EMAIL_PORT = 465     # 发件箱的SMTP服务器端口
EMAIL_HOST_USER = 'xxxxx@xmdaren.com'    # 发送邮件的邮箱地址
EMAIL_HOST_PASSWORD = '*********'         # 发送邮件的邮箱密码(这里使用的是授权码)
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER  # 默认的发送方
```

DEFAULT_FROM_EMAIL 还可以写成这样：

```python
DEFAULT_FROM_EMAIL = 'mac <mac@163.com>'
```

这样别人收到的邮件中就会有你设定的名称。

下面是一些常用的邮箱：

[163 邮箱](http://help.163.com/09/1223/14/5R7P3QI100753VB8.html)   [126 邮箱](http://www.126.com/help/client_04.htm)  [QQ 邮箱](https://service.mail.qq.com/cgi-bin/help?subtype=1&&no=166&&id=28)

其它邮箱参数可以登陆邮箱寻找帮助信息，也可以尝试在搜索引擎中搜索："SMTP 邮箱名称"，比如："163 SMTP" 进行查找。

## 发送邮件

 发送普通邮件可以使用`django.core.mail`模块下的`send_mail`函数进行

```python
send_mail(subject, message, from_email, recipient_list, fail_silently=False, html_message=None)
# 将邮件发送至recipient_list中的每一个收件人
'''
subject: 发送邮件标题
message: 发送邮件正文
from_email: 发件人邮箱地址
recipient_list: 一个字符串列表，每一个数据为接收者的邮箱地址
html_message: 如果指定该值，则发送的内容类型为text/html为一个html邮件内容
'''
```



```python
from django.conf import settings
from django.core.mail import send_mail


def send_regiser_active_email(to_email, username, token):
    '''发送激活邮件'''
    active_url = "{}/user/active/{}".format(settings.HOST_URL, token)
    subject = "xxx欢迎信息"
    message = '邮件正文'
    sender = settings.DEFAULT_FROM_EMAIL
    receiver = [to_email]
    html_message = '<h1>{}, 欢迎您成为xxx注册会员</h1>请点击下面链接激活您的账户<br/><a href="{}">{}</a>'.format(username, active_url,
                                                                                            active_url)
    send_mail(subject, message, sender, receiver, html_message=html_message)
    
```

## 邮件激活

注册成功，发送激活链接, **注意：**为了安全考虑，可以使用`itsdangerous`，根据用户信息加密生成 token

```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
    
    
# 加密用户信息，生成token
info = {'confirm': user.id}
serializer = Serializer(settings.SECRET_KEY, 3600)
token = serializer.dumps(info).decode()

# 发送邮件
send_regiser_active_email(email, username, token)

```

用户点击激活链接，进行账户激活

```python
# 进行解密， 获取要激活的用户信息
serializer = Serializer(settings.SECRET_KEY, 3600)
try:
    # 获取用户id
    info = serializer.loads(token)  # 在路由匹配中，获取token
except (SignatureExpired, BadSignature) as error:
    return redirect(reverse("user:activeemail"))  # 激活失败，重写跳转到邮箱激活页面

else:
    user_id = info.get("confirm")
    # 获取用户信息
    user = User.objects.get(id=user_id)
    user.is_active = 1
    user.save()

    # 跳转到登录界面
    return redirect(reverse('user:login'))  # 激活成功，跳转到登录页面      
```

