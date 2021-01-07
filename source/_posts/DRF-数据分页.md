---
title: DRF-数据分页
date: 2020-03-16 20:23:36
categories:
  - 技术
  - python
  - web
tags:
  - DRF
  - 分页

---

REST framework提供了分页的支持。有以下两种方式实现:

## 全局配置

- 修改`settings.py`文件
  
  ```python
  REST_FRAMEWORK = {
      ...
      'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.PageNumberPagination',
      'PAGE_SIZE': 100  # 每页数目
      ...
  }
  ```

## 类视图中定义

自定义分页器,有两种分页器可选

### PageNumberPagination

- 前端访问形式

  ```python
  GET  http://127.0.0.1:8000/books/?page=1&page_size=2
  ```

- 属性

  - page_size 每页数目

  - page_query_param 前端发送的页数关键字名，默认为"page"

  - page_size_query_param 前端发送的每页数目关键字名，默认为None

  - max_page_size 前端最多能设置的每页数量

- 实例展示

  ```python
  from rest_framework.pagination import PageNumberPagination

  class StandardPageNumberPagination(PageNumberPagination):
      page_size_query_param = 'page_size'  # 前端发送的每页数目关键字名，默认为None
      max_page_size = 10  # 前端最多能设置的每页数量
      page_size = 3  # 每页数目
      page_query_param = 'page'  # 前端发送的页数关键字名，默认为"page"

  class BookListView(ListAPIView):
      queryset = BookInfo.objects.all().order_by('id')
      serializer_class = BookInfoSerializer
      pagination_class = StandardPageNumberPagination

      def get(self, request, *args, **kwargs):
          return self.list(request, *args, **kwargs)
  ```

### LimitOffsetPagination

- 前端访问形式

  ```python
  GET  http://127.0.0.1:8000/books/?limit=2&offset=0
  ```

- 属性

  - `default_limit`: 默认限制，默认值与PAGE_SIZE设置一致
  - `limit_query_param`: `limit`参数名，默认'limit'
  - `offset_query_param`: `offset`参数名，默认'offset'
  - `max_limit`: 最大`limit`限制，默认None

- 实例展示

  ```python
  class LimitOffsetNumberPagination(LimitOffsetPagination):
      default_limit = 2  # 默认限制，默认值与PAGE_SIZE设置一直
      limit_query_param = 'limit'  # limit参数名，默认'limit'
      offset_query_param = 'offset'  # offset参数名，默认'offset'
      max_limit = 3  # 最大limit限制，默认None

  class GoodsView(GenericAPIView, CreateModelMixin, ListModelMixin):
      queryset = Goods.objects.all().order_by('id')
      serializer_class = GoodsSerializer
      pagination_class = LimitOffsetNumberPagination

      def get(self, request, *args, **kwargs):
          return self.list(request, *args, **kwargs)
  ```
