---
title: Flask易错点
categories:
  - 技术
  - python
  - Flask
tags:
  - 易错点
date: 2019-03-22 22:58:58
---
#### 装饰器

```python
@app.route("/")
def index():
   	return "index page"
```

```python
def index():
   	return "index page"
app.route("/")(index)
```

装饰器不仅仅是定义时可以用，还可以在定义完再使用

#### 自定义正则转换器及蓝图

```python
from werkzeug.routing import BaseConverter

# 定义正则转换器
class ReConverter(BaseConverter):

    def __init__(self, url_map, regex):
        # 调用父类初始化方法
        super(ReConverter, self).__init__(url_map)
        # 重新赋值
        self.regex = regex
```

```python
  # 添加自定义的转换器
    app.url_map.converters["re"] = ReConverter
```



```python
from flask import Blueprint, current_app

html = Blueprint("web_html", __name__)

@html.route("/<re(r'.*'):file_name>")
def web_html(file_name):

    if not file_name:
        file_name = "index.html"

    if file_name != "favicon.ico":
        file_name = "html/" + file_name

    return current_app.send_static_file(file_name)
```

```python
# 注册蓝图
app.register_blueprint(html)
```

#### 登录装饰器

```python
# 定义验证登录状态的装饰器
def login_required(view_func):
    # wraps函数的作用是将wrapper内层函数的属性设置为被装饰函数view_func的属性
    @functools.wraps(view_func)
    def wrapper(*args, **kwargs):
        # 判断用户登录状态
        user_id = session.get("user_id")

        # 如果用户是登录状态，则执行登录状态
        if user_id is not None:
            # 将user_id保存到g对象中，在视图函数中，可以通过g对象获取保存数据
            g.user_id = user_id
            return view_func(*args, **kwargs)
        # 如果未登录，则返回未登录信息
        else:
            return jsonify(errno=RET.SESSIONERR, errmsg="用户未登录")
    return wrapper
```

