---
title: DRF-数据分页
date: 2020-03-16 20:23:36
categories:
  - 技术
  - python
  - DRF
tags:
  - 分页

---

> 当需要展示的数据过多时，就需要将全部数据分批显示，这就需要用到 **分页** 功能

## Django内置分页

在DRF提供的API视图中，并不支持 `pagination_class` 分页属性，因此，需要分页，只能使用django内置的分页器 `Paginator` 。

Paginator 类的作用是将我们需要分页的数据分割成若干份。当我们实现化一个 Paginator 类的实例时，需要给 Paginator 传入两个参数。第一个参数是数据源，可以是**一个列表、元组、以及查询结果集 QuerySet**。第二个参数**需要传入一个整数，表示每页显示数据条数**。具体用法如下：

### 导入分页模块

```python
from django.core.paginator import Paginator
```

### 获取queryset对象

```python
goods_list = Goods.objects.all().order_by('id')
```

### 实例化分页类对象

```python
paginator = Paginator(goods_list, 2)
```

#### Paginator类对象的属性

| 序号 | 属性名     | 说明                 |
| ---- | ---------- | -------------------- |
| 1    | num_pages  | 返回分页之后的总页数 |
| 2    | page_range | 返回分页后的页码列表 |
| 3    | count      | 需要分页的总记录数   |

#### Paginator类对象的方法

| 序号 | 方法名                 | 说明                           |
| ---- | ---------------------- | ------------------------------ |
| 1    | get_page(self, number) | 返回第number页的page类实例对象 |

#### Page实例对象的属性

| 序号 | 属性名      | 说明                          |
| ---- | ----------- | ----------------------------- |
| 1    | number      | 返回当前页的页码              |
| 2    | object_list | 返回当前页的数据查询集        |
| 3    | paginator   | 返回对应的**Paginator**类对象 |

#### page实例对象的方法

| 序号 | 方法名               | 说明                   |
| ---- | -------------------- | ---------------------- |
| 1    | has_previous         | 判断当前页是否有前一页 |
| 2    | has_next             | 判断当前页是否有下一页 |
| 3    | previous_page_number | 返回前一页的页码       |
| 4    | next_page_number     | 返回下一页的页码       |

### 分页功能实现

```python
from rest_framework.response import Response
from rest_framework.views import APIView
from school.serializers import *


class StudentsAPIView(APIView):
    def get(self, request):
        page_number = request.GET.get('page', 1)  # 获取客户端发送的页码，默认为1
        page_size = request.GET.get('page_size', 2)  # 获取客户端发送的每页数量，默认为1
        try:
            page_number = int(page_number)  # 处理页码， 过滤无效的数据
        except:
            page_number = 1

        goods_set = Student.objects.all().order_by('id')

        paginator = Paginator(goods_set, page_size)  # 得到分页器对象
        page = paginator.get_page(page_number)  # 得到当前页码对象

        ser = StudentSerializer(page.object_list, many=True)

        return Response(ser.data)
```



## DRF 分页

>   REST框架包括对可定制的分页样式的支持。

### 分页类

#### PageNumberPagination：

可以进行直接的页码处理，返回某一页分页数据

```python
http://127.0.0.1:8000/user/?page=2
# page：当前的页码
```

> 除了默认的分页样式之外，还可以通过继承分页类的方式重写分页样式

```python
from rest_framework.pagination import PageNumberPagination


class class PagerPagination(PageNumberPagination):
    max_page_size = 3  # 每页最大数目
    page_size = 1  # 默认每页数目
    page_query_param = 'page'  # 页码关键字名， 默认为"page"
    page_size_query_param = 'size'  # 每页数目关键字名, ，默认为None
```

#### LimitOffsetPagination

可以通过连接可选的参数进行分页单页数量大小的控制，分页数据偏移的选择

```python
http://127.0.0.1:8000/path/?limit=2&offset=2
# limit：每页的数据大小
# offset：从某一个数据位置开始偏移
```

> 除了默认的分页样式之外，还可以通过继承分页类的方式重写分页样式

```python
from rest_framework.pagination import LimitOffsetPagination

class LimitPagination(LimitOffsetPagination):
    max_limit = 2  # 最大limit限制，默认None
    default_limit = 1  # 默认限制，和page_size作用一样
    limit_query_param = 'limit'  # 参数名，默认limit
    offset_query_param = 'offset'  # 参数名，默认offset
```



#### 自定义分页类

>   可以继承父类，并重写响应对象

```python
from collections import OrderedDict
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response
from rest_framework.utils.urls import replace_query_param


class Pagination(PageNumberPagination):
    def get_num_pages_link(self):
        url = self.request.build_absolute_uri()
        num_pages = self.page.paginator.num_pages
        return [replace_query_param(url, self.page_query_param, page) for page in range(1, num_pages + 1)]

    def get_paginated_response(self, data):
        return Response(OrderedDict([
            ('count', self.page.paginator.count),
            ('num_pages', self.page.paginator.num_pages),  # 分页的总页码
            ('next', self.get_next_link()),
            ('previous', self.get_previous_link()),
            ('pages', self.get_num_pages_link()),  # 所有分页的链接
            ('results', data)
        ]))

```

### 分页应用

#### 全局分页

> **注意**：需要修改Django的全局配置文件 `settings.py`

分页样式可以使用`DEFAULT_PAGINATION_CLASS`和`PAGE_SIZE`设置键进行全局设置。

例如，要使用内置的限制/偏移分页，可以执行以下操作：

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS':
        'rest_framework.pagination.LimitOffsetPagination',  # 指明分页类
        # 'rest_framework.pagination.PageNumberPagination',
        # 'utils.pagination.Pagination', # 自定义分页类
    
    'PAGE_SIZE': 2  # 每页数目
}
```

请注意，您需要同时设置分页类和应使用的页面大小。

#### 局部分页

您还可以使用`pagination_class`属性在单个视图上设置分页类。配置完了，如果不想用，那么就直接在视图类下使用如下属性关闭全局分页功能

```python
from rest_framework.generics import ListCreateAPIView
from .serializer import BookSerializer
from .models import BookModel

class BookListView(ListCreateAPIView):
    serializer_class = BookSerializer
    queryset = BookModel.objects.all()
    pagination_class = None  # 指定分页类，为None，即当前视图不分页
```




