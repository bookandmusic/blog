---
title: Celery异步任务
date: 2021-02-28 21:39:31
categories:
  - 技术
  - python
  - Django
tags:
  - Celery
  - 单文件导入settings
  - 异步操作
---

**情景：** 用户发起request，并等待response返回。在本些views中，可能需要执行一段耗时的程序，那么用户就会等待很长时间，造成不好的用户体验，比如发送邮件、手机验证码等。

使用celery后，情况就不一样了。

**解决：** 将耗时的程序放到celery中执行。

Celery 是一个包含一系列的消息任务队列。您可以不用了解内部的原理直接使用，它的使用时非常简单的。

-   选择并且安装一个消息中间件（Broker）
-   安装 Celery 并且创建第一个任务
-   运行职程（Worker）以及调用任务
-   跟踪任务的情况以及返回值

[celery官方网站: http://www.celeryproject.org/](http://www.celeryproject.org/)

[celery中文文档: https://www.celerycn.io/](https://www.celerycn.io/)

Celery名词：

-   任务`task`：就是一个Python函数。
-   队列`queue`：将需要执行的任务加入到队列中。
-   工人`worker`：在一个新进程中，负责执行队列中的任务。
-   代理人`broker`：负责调度，在布置环境中使用redis。

## 选择中间人（Broker）

Celery 需要一个中间件来进行接收和发送消息，通常以独立的服务形式出现，成为 消息中间人（Broker）

以下有几种选择：

-   RabbitMQ：[RabbitMQ](https://www.rabbitmq.com/) 的功能比较齐全、稳定、便于安装。在生产环境来说是首选的。
-   Redis: [Redis](https://redis.io/) 功能比较全，但是如果突然停止运行或断电会造成数据丢失。

## 安装 Celery

可以用标准的 Python 工具，诸如 `pip` 或 `easy_install` 来安装：

```zsh
$ pip install celery
```

## Celery单应用

### 创建Celery实例对象及task任务

创建第一个 Celery 实例程序，我们把创建 Celery 程序成为 Celery 应用或直接简称 为 app，创建的第一个实例程序可能需要包含 Celery 中执行操作的所有入口点，例如创建任务、管理职程（Worker）等，所以必须要导入 Celery 模块。

在本教程中将所有的内容，保存为一个 app 文件中。针对大型的项目，可能需要创建 独立的模块。

首先创建 `utils/tasks.py`：

```python
import os
import sys
import django
from celery import Celery
from django.conf import settings
from django.core.mail import send_mail
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

project_path = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))  # 确定项目路径
sys.path.append(project_path)  # 将项目路径添加到系统搜索路径中
os.environ['DJANGO_SETTINGS_MODULE'] = 'django_test.settings'  # 设置项目的配置文件
django.setup()  # Django初始化

app = Celery('tasks', broker='amqp://guest@localhost//')


@app.task()
def task_send_email(user_id, username, email):
    # 加密用户信息，生成token
    info = {'confirm': user_id}
    serializer = Serializer(settings.SECRET_KEY, 3600)
    token = serializer.dumps(info).decode()

    # 发送邮件
    '''发送重置密码邮件'''
    reset_url = "{}/user/reset/password/?token={}".format(settings.HOST_URL, token)
    subject = "重置密码"
    message = '邮件正文'
    sender = settings.DEFAULT_FROM_EMAIL
    receiver = [email]
    html_message = '<h1>{}, 您好！</h1>请在1小时内点击下面链接重置密码:<br/><a href="{}">{}</a>'.format(username, reset_url, reset_url)
    send_mail(subject, message, sender, receiver, html_message=html_message)

```

-   第一个参数为当前模块的名称，只有在 `__main__` 模块中定义任务时才会生产名称。

-   第二个参数为中间人（Broker）的链接 URL ，实例中使用的 RabbitMQ（Celery默认使用的也是RabbitMQ）。

    更多相关的 Celery 中间人（Broker）的选择方案，可查阅上面的中间人（Broker）。

    例如，对于 RabbitMQ 可以写为 `amqp://localhost` ，使用 Redis 可以写为 `redis://localhost`。

-   创建了一个名称为 `task_send_email` 的任务，实现邮件的发送。

>   **注意**： 在 单文件中，想要导入 Django项目的`settings.py`配置文件

-   先确定Django项目所在的绝对路径
-   追加Django所在的路径到系统搜索路径
-   配置系统的Django环境变量

### 运行 Celery 子进程（Worker）服务

现在可以使用 `worker` 参数进行执行我们刚刚创建职程（Worker）：

注意： **此时运行celery，是在Django项目下的`utils`目录中**，`-A celery实例名`，即为当前文件的名

```zsh
$ celery -A task worker --loglevel=info
```

## Celery集成django使用

**注意:** 以前版本的Celery需要一个单独的库才能与Django一起使用，但是从3.1开始，情况不再如此。现在就已支持Django。

**注意:** Celery 4.0支持Django 1.8和更高版本。对于Django 1.8之前的版本，请使用Celery 3.1。

### 创建Celery实例对象

要将Celery与Django项目一起使用，必须首先定义Celery库的实例(称为 "app")

如果您拥有当前主流的Django项目结构，例如：

```python
- django_test/
  - manage.py
  - django_test/
    - __init__.py
    - settings.py
    - urls.py
```

那么建议的方法是创建一个新的`django_test/django_test/celery.py`模块，该模块定义`Celery`实例

**file**: `django_test/django_test/celery.py`

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery


# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'django_test.settings')


app = Celery('django_test')


# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')


# Load task modules from all registered Django app configs.
app.autodiscover_tasks()




@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

### Django加载Celery

然后，需要将此程序导入到`django_test/django_test/__ init__.py`模块中。这样可以确保在Django启动时加载应用程序，以便@shared_task装饰器将使用该应用程序：

**file**:`django_test/django_test/__init__.py:`

```python
from __future__ import absolute_import, unicode_literals


# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app


__all__ = ('celery_app',)
```

请注意，此示例项目结构适用于较大的项目，对于简单项目，可以使用一个包含的模块来定义应用程序和任务。

### 步骤解析

让我们分解一下第一个模块中发生的情况，首先我们从`__freture__`导入`absolute_import`导入`unicode_literals`，以使我们的`celery.py`模块不会与库冲突：

```python
from __future__ import absolute_import
```

然后，为celery命令行程序设置默认的`DJANGO_SETTINGS_MODULE`环境变量：

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'django_test.settings')
```

不需要此行，但可以避免始终将设置模块传递到celery程序中。它必须始终在创建应用程序实例之前出现，我们接下来要做的是：

```python
app = Celery('django_test')
```

这是我们的库实例，可以有很多实例，但是使用Django时可能没有理由。 我们还将Django设置模块添加为Celery的配置源。这意味着不必使用多个配置文件，而直接从Django设置中配置Celery；但也可以根据需要将它们分开。

```python
app.config_from_object('django.conf:settings', namespace='CELERY')
```

大写名称空间意味着所有Celery配置选项必须以大写而不是小写指定，并以`CELERY_`开头，因此例如[`task_always_eager`](https://docs.celeryproject.org/en/v4.3.0/userguide/configuration.html#std:setting-task_always_eager)设置变为`CELERY_TASK_ALWAYS_EAGER`，[`broker_url`](https://docs.celeryproject.org/en/v4.3.0/userguide/configuration.html#std:setting-broker_url)设置变为`CELERY_BROKER_URL`。

可以直接传递设置对象，但是使用字符串更好，因为这样一来，工作人员就不必序列化该对象。 `CELERY_`名称空间也是可选的，但建议使用（以防止与其他Django设置重叠）。

接下来，可重用应用程序的常见做法是在单独的task.py模块中定义所有任务，而Celery可以自动发现这些模块：

```python
app.autodiscover_tasks()
```

在上述行中，Celery将按照task.py约定自动发现所有已安装应用程序中的任务：

```python
- app1/
    - tasks.py
    - models.py
- app2/
    - tasks.py
    - models.py
```

这样，不必手动将各个模块添加到[`CELERY_IMPORTS`](https://docs.celeryproject.org/en/v4.3.0/userguide/configuration.html#std:setting-imports)设置中。 最后，debug_task示例是一个转储其自己的请求信息的任务。这是使用Celery 3.1中引入的新的`bind = True`任务选项来轻松引用当前任务实例。

### 使用@shared_task装饰器

编写的任务可能会存在于可重用应用程序中，并且可重用应用程序不能依赖于项目本身，因此不能直接导入应用程序实例。 @shared_task装饰器可让您创建任务而无需任何具体的应用程序实例：

 **file**: `user/tasks.py`:

```python
# Create your tasks here
from __future__ import absolute_import, unicode_literals
from celery import shared_task
from django.conf import settings
from django.core.mail import send_mail
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer


@shared_task
def task_send_email(user_id, username, email):
    # 加密用户信息，生成token
    info = {'confirm': user_id}
    serializer = Serializer(settings.SECRET_KEY, 3600)
    token = serializer.dumps(info).decode()

    # 发送邮件
    '''发送重置密码邮件'''
    reset_url = "{}/user/reset/password/?token={}".format(settings.HOST_URL, token)
    subject = "重置密码"
    message = '邮件正文'
    sender = settings.DEFAULT_FROM_EMAIL
    receiver = [email]
    html_message = '<h1>{}, 您好！</h1>请在1小时内点击下面链接重置密码:<br/><a href="{}">{}</a>'.format(username, reset_url,reset_url)
    send_mail(subject, message, sender, receiver, html_message=html_message)

```

### 运行 Celery 子进程（Worker）服务

现在可以使用 `worker` 参数进行执行我们刚刚创建职程（Worker）：

注意： **此时运行celery，是在Django项目下**，`-A celery实例名`，即为Django项目的同名目录

```zsh
$ celery -A django_test worker --loglevel=info
```

## 调用任务

需要调用我们创建的实例任务，可以通过 `delay()` 进行调用。

`delay()` 是 `apply_async()` 的快捷方法，可以更好的控制任务的执行（详情：[`调用任务：Calling Tasks`]()）：

```python
from django.contrib.auth import authenticate
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from user.tools.auth import JSONWebTokenAuthentication # 自定义token认证类
# from .task import task_send_email # 导入Django集成的task任务
from utils.task import task_send_email  # 导入 celery单应用的task任务

class ResetPassword(APIView):
    authentication_classes = [JSONWebTokenAuthentication]
    permission_classes = [IsAuthenticated]

    def post(self, request):
        user = request.user

        task_send_email.delay(user_id=user.id, username=user.username, email=user.email)
        return Response('发送成功')
```

该任务已经有职程（`Worker`）开始处理，可以通过控制台输出的日志进行查看执行情况。