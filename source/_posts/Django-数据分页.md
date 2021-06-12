---
title: Django-数据分页
date: 2020-11-30 21:45:42
categories:
  - 技术
  - python
  - Django
tags:
  - 分页
---

当页面因需要展示的数据条目过多，导致无法在一个页面全部显示。这时，页面经常会采用分页形式进行展示，然后每页显示 20 或者 50 等条数据。分页经常在网站上随处可见，

分页的实现不仅提高了用户体验，还是减轻数据库读取数据的压力。Django 自带名为 Paginator 的分页工具， 方便我们实现分页功能。本文就讲解如何使用 Paginator 实现分页功能。

# Paginator

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

## 分页功能实现

### views函数

```python
class IndexView(View):
    def get(self, request):
        page_number = request.GET.get('page', 1)  # 获取客户端发送的页码，默认为1
        page_size = request.GET.get('page_size', 2)  # 获取客户端发送的每页数量，默认为1
        try:
            page_number = int(page_number)  # 处理页码， 过滤无效的数据
        except:
            page_number = 1

        goods_list = Goods.objects.all().order_by('id')

        paginator = Paginator(goods_list, page_size)  # 得到分页器对象
        page = paginator.get_page(page_number)  # 得到当前页码对象

        # 获取页码列表
        index = page.number - 1  # 当前页码对应的索引
        max_index = paginator.num_pages - 1  # 最大索引
        # 为了得到显示7个页码的列表，从当前索引向前数3个，向后数3个，加上本身，即7个页码
        start_index = index - 3 if index >= 3 else 0
        end_index = index + 3 if index <= max_index - 3 else max_index
        # 最后处理过的页码列表
        page_range = paginator.page_range[start_index:end_index + 1]
        return render(request, 'index.html', context={'goods': page, 'page_range': page_range})

```

### 模板实现

```html
<!-- 引入bootstrap样式文件 -->
<link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">

<!--数据遍历展示-->
<table class="table">
    <thead class="thead-light">
    <tr>
        <th scope="col">#</th>
        <th scope="col">名称</th>
        <th scope="col">单价</th>
        <th scope="col">数量</th>
        <th scope="col">图片</th>
    </tr>
    </thead>
    <tbody>
    {% for good in goods.object_list %}
        <tr>
            <th>{{ good.id }}</th>
            <td>{{ good.name }}</td>
            <td>{{ good.price }}</td>
            <td>{{ good.nums }}</td>
            <td>
                <img src="/media/{{ good.img }}" alt="" style="width: 100px;height: 130px">
            </td>
        </tr>
    {% endfor %}
    </tbody>
</table>

<!--底部分页按钮-->
<nav aria-label="Page navigation example" style="margin: 0 auto">
    <ul class="pagination">
        <li class="page-item">
            {% if goods.has_previous %}
                <a href="{% url 'goods:index' %}?page={{ goods.previous_page_number }}" class="page-link"
                   aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            {% else %}
                <a href="#" class="page-link" aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            {% endif %}
        </li>

        {% for page in page_range %}
            <li class="page-item">
                {% ifequal page goods.number %}
                    <a href="#" class="page-link">{{ page }}</a>
                {% else %}
                    <a href="{% url 'goods:index' %}?page={{ page }}" class="page-link">{{ page }}</a>
                {% endifequal %}
            </li>
        {% endfor %}

        <li class="page-item">
            {% if goods.has_next %}
                <a href="{% url 'goods:index' %}?page={{ goods.next_page_number }}" class="page-link" aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            {% else %}
                <a href="#" class="page-link" aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            {% endif %}
        </li>
    </ul>
</nav>
```

#### 路由配置

```python
urlpatterns = [
    path('', IndexView.as_view(), name='index'),  # /?page=1&page_size=2
]
```

