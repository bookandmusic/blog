---
title: Flask图片上传与展示
date: 2021-06-09 00:00:28
categories:
 - python
 - Flask
tags:
 - 图片上传
---

## 项目配置

```python
import os


class Config(object):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'mysql://root:mysql@127.0.0.1:3306/day08'
    SQLALCHEMY_TRACK_MODIFICATIONS = False

    # 构建项目所在的 绝对路径，也就是 day08 的绝对路径
    BASE_DIR = os.path.abspath(os.path.dirname(os.path.dirname(__file__)))
    # 静态资源存放路径
    STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
    # 自定义的 图片上传路径
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
```

## 图片上传与展示

### static静态资源方式

>   采用此种方式：
>
>   -   创建APP时，一定要指明静态资源的路径
>   -   然后在上传图片时，将上传的图片存放到上一步指明的静态资源路径中
>   -   就可以使用static内置的服务，展示图片资源

#### app创建

>   创建app时,指明静态资源目录

```python
app = Flask(__name__, static_folder=Config.STATIC_ROOT, static_url_path='/static')
```

-   `static_folder`： 静态资源路径
-   `static_url_path`： 静态资源展示的url前缀

#### 图片上传

```python
import os

from uuid import uuid4
from flask_restful import Resource, reqparse
from werkzeug.datastructures import FileStorage

from app.config import Config
from .models import *

def img_upload(img):
    if not img:
        return None

    # 将图片名按照 .  进行切分， 找到最后一个元素，也就是  文件的后缀名
    end_name = img.filename.rsplit('.')[-1]

    # 通过文件的后缀名判断 身份为 合法的  图片
    if end_name not in ['jpg', 'png', 'gif', 'jpeg']:
        return None

    filename = str(uuid4()) + '.' + end_name  # 为了生成一个不重复的 文件名
    img_path = os.path.join(Config.STATIC_ROOT, filename)  # 将路径和文件名拼接在一起，方便保存文件

    img.save(img_path)  # 将图片对象保存到 本地

    return filename


class NewsView(Resource):
    def post(self):
        # 1. 创建解析参数的对象
        parser = reqparse.RequestParser()
        # 2. 指明需要解析的参数
        parser.add_argument('title', type=str, location='form', required=True)
        parser.add_argument('img', type=FileStorage, location='files')

        # 3. 获取具体的参数
        args = parser.parse_args()

        title = args.get('title')
        img = args.get('img')

        # 利用自定义函数，将图片保存到本地
        filename = img_upload(img)

        # 4. 创建对象, 注意：图片存储的只是 从media之后的  图片路径
        news = News(title=title, img=filename)
        # 5. 添加到 事务中
        db.session.add(news)
        # 6. 提交事务
        try:
            db.session.commit()
        except:
            return {
                       'msg': '添加失败'
                   }, 500
        # 7. 返回响应
        return {
                   'id': news.id,
                   'title': news.title,
                   'img': news.img
               }, 201
```

#### 图片访问

>   通过 `http://127.0.0.1:5000/static/filename`就可以访问图片

-   `static`是app创建时，配置的`url`前缀

### 自定义方式

>   自定义方式实现：
>
>   -   app创建时，不需要指明静态资源有关的任何参数
>   -   图片的存放路径，可以是 任何一个合适的路径
>   -   展示图片时，需要自己定义视图和路由，实现图片展示

#### 图片上传

>   和上面的图片上传的流程一样，只是 图片存放的路径可以自由指定，不必一定存放到 某个特定路径

#### 图片访问

>   图片访问时，可以自己读取文件内容，也可以调用flask内置的 方法，返回响应

```python
import os
from flask import make_response
from app.config import Config


class ImgView(Resource):
    def get(self, filename):
        # 1. 拼接除图片的完成路径
        img_path = os.path.join(Config.MEDIA_ROOT, filename)

        # 2. 按照二进制方式打开文件，读到的内容为 二进制文件流，方便接下来的网络传输
        try:
            with open(img_path, 'rb') as f:
                img = f.read()
        except FileNotFoundError:
            return None, 404

        # 3. 自定义响应
        resp = make_response(img)
        # 4. 声明响应体的类型 为  图片
        resp.headers['Content-Type'] = 'image/png'
        # 5. 返回响应
        return resp
```

```python
import os
from flask import send_from_directory

from app.config import Config


class ImgView(Resource):
    def get(self, filename):
		# 2. 使用flask 内置的函数提供响应
        return send_from_directory(Config.MEDIA_ROOT, filename)

```

>   通过 `http://127.0.0.1:5000/media/filename`就可以访问图片

-   `media`是自己配置的路由前缀
