---
title: Flask易错点
categories:
  - 技术
  - python
  - web
tags:
  - web
  - flask
date: 2019-03-22 22:58:58
---
#### 1 with上下文管理器

常用：

```python
with open("file_name","wb") as f:
	f.write("hello flask")
```

自定义：

```python
class Foo(gbiect):
	def __enter__(self):
		"""进入with语句的时候被with调用"""
		print("enter called")
	def __exit_(self, exc_type, exc_val, exc_tb):
		"""离开with语句的时候被with调用"""
        print("exit called")
        print("exc_type:%s" % exc_type)
        print("exc_val:9%s" % exc_val)
		print("exc_tb:%s"%exc_tb)
		
with Foo() as foo:
    print("helto python")
    a=1/0
    print("hello end")
```

运行结果：

> ```bash
> enter called 
> Traceback (most recent call last): 
> hello python 
> 	File"/Users/delron/Desktop/code/03 with. py", line 39, in <module>
> exit called
>      a=1/0
> ZeroDivisionError: integer division or modulo by zero 
> exc_type:<type ' exceptions. ZeroDivisionError'>
> exc_val: integer division or modulo by zero 
> exc_tb:<traceback object at 0x1097bc440>
> 
> Process finished with exit code 1
> ```

#### 2 json模块

dumps  —> 可以将字典转换为字符串

```python
import json
data = {"name": "python", "age": 18}
json_str = json.dumps(data)
print(type(json_str), json_str)
```

运行结果：

```python
str      {"age": 18, "name": "python"}
```

loads ——> 将字符串转换为字典

```python
import json
a = '{"city": "sz", "country": "china"}'
b = json.loads(a)
print(type(b), b)
```

运行结果：

```python
dict      {"city": "sz", "country": "china"}
```

#### 3 xss攻击

当前段传送过来的数据默认进行转义，否则，则会默认执行前端传送的数据，则称为xss攻击

#### 4 flask 和mysql

**Linux：**

*flask使用mysql数据库需要：*

1. **pymysql**
2. **sqlalchemy**
3. **flask_sqlalchemy**

**windows：**

***Flask利用pymysql出现Warning：1366的解决办法***

*flask使用mysql数据库需要：*

1. **mysql-connector-python**

2. **sqlalchemy**

3. **flask_sqlalchemy**

   ```python
   SQLALCHEMY_DATABASE_URI = "mysql+mysqlconnector://root:mysql@localhost/ihome01"
   ```

#### 5 装饰器

```python
@app.route("/")
def index():
   	return "index page"
```

```python
def index():
   	return "index page"
app.route("/)(index)
```

装饰器不仅仅是定义时可以用，还可以在定义完再使用

#### 6 自定义正则转换器及蓝图

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

#### 7 登录装饰器

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

#### 8 视图函数 

1. 路由匹配不能出现相同的地址，即同一地址，不能出现两个视图函数
2. 路由匹配不能出现不同的函数，即不同的地址，不能出现相同的函数名

#### 9 参数获取

1. 直接从request中获取json数据，并将其转换为字典

   ```python
   house_data = request.get_json()
   ```

2. 从request中获取文件

   ```python
   image_file = request.files.get("house_image")
   ```

3. 从request中的form表单中获取键值对

   ```python
   house_id = request.form.get("house_id")
   ```

#### 10 数据库操作

​	在同一视图函数中，可以对对象多次修改，只提交一次即可

```python
db.session.add(house_image)
db.session.add(house)

try:
    db.session.commit()
except Exception as e:
    current_app.logger.error(e)
    db.session.rollback()
```

