---
title: haystack+Elasticsearch实现搜索引擎
date: 2021-03-02 20:24:18
categories:
  - 技术
  - python
  - DRF
tags:
  - haystack
  - Elasticsearch
  - DRF
---

## 搜索引擎原理

通过搜索引擎进行数据查询时，搜索引擎并不是直接在数据库中进行查询，而是搜索引擎会对数据库中的数据进行一遍预处理，单独建立起一份索引结构数据。

我们可以将索引结构数据想象成是字典书籍的索引检索页，里面包含了关键词与词条的对应关系，并记录词条的位置。

我们在通过搜索引擎搜索时，搜索引擎将关键字在索引数据中进行快速对比查找，进而找到数据的真实存储位置。

##  Elasticsearch

开源的 [Elasticsearch ](https://www.elastic.co/)是目前全文搜索引擎的首选。

它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 都采用它。

Elasticsearch 的底层是开源库 [Lucene](https://lucene.apache.org/)。但是，你没法直接用 Lucene，必须自己写代码去调用它的接口。Elastic 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。

Elasticsearch 是用Java实现的。

搜索引擎在对数据构建索引时，需要进行分词处理。分词是指将一句话拆解成多个单字或词，这些字或词便是这句话的关键词。如: `我在广州`

`我`、`在`、`广`、`州`、`广州` 等都可以是这句话的关键词。

Elasticsearch 不支持对中文进行分词建立索引，需要配合扩展**elasticsearch-analysis-ik**来实现中文分词处理。

## Docker安装Elasticsearch及其扩展

>   注意： haystack 支持 Elasticsearch 1.x、2.x、5.x，所以不能安装Elasticsearch7.x

创建文件夹  `elasticsearch`

下载中文分词插件 [elasticsearch-analysis-ik-5.6.16.zip](https://github.91chifun.workers.dev//https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.16/elasticsearch-analysis-ik-5.6.16.zip)， 注意：插件需要和elasticsearch版本对应

解压到当前文件夹，并将文件夹重命名为`elasticsearch-analysis-ik`

创建 `DockerFile`

```dockerfile
FROM elasticsearch:5.6.16
ADD elasticsearch-analysis-ik /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik
```

创建镜像

```zsh
docker build -f Dockerfile -t bookandmusic/elasticsearch-ik:5.6.16 .
```

查看镜像

```zsh
docker images
```

创建容器

```zsh
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name elasticsearch -e ES_JAVA_OPTS="-Xms64m -Xmx512m" -d bookandmusic/elasticsearch-ik:5.6.16

# docker run --name elasticsearch 创建一个es容器并起一个名字；
# -p 9200:9200 将主机的9200端口映射到docker容器的9200端口，用来给es发送http请求
# -p 9300:9300 9300是es在分布式集群状态下节点之间的通信端口  \ 换行符
# -e 指定一个参数，当前es以单节点模式运行
# *注意，ES_JAVA_OPTS非常重要，指定开发时es运行时的最小和最大内存占用为64M和512M，否则就会占用全部可用内存
# -d 后台启动服务
```

>   注意： 创建容器时，一定要指定内存，否则，则直接闪退

## 使用haystack对接Elasticsearch

Haystack为Django提供了模块化的搜索。它的特点是统一的，熟悉的API，可以让你在不修改代码的情况下使用不同的搜索后端（比如 Solr, Elasticsearch, Whoosh, Xapian 等等）。

我们在django中可以通过使用haystack来调用Elasticsearch搜索引擎。

### 安装依赖包

```zsh
pip install django-haystack
pip install drf-haystack 
pip install "elasticsearch>=5,<6"
```

[drf-haystack](https://drf-haystack.readthedocs.io/en/latest/index.html)是为了在REST framework中使用haystack而进行的封装（如果在Django中使用haystack，则安装[django-haystack](https://django-haystack.readthedocs.io/en/master/toc.html)即可）。

### 注册应用

```python
INSTALLED_APPS = [
    ...
    'haystack',
    ...
]
```

### 配置

在配置文件 `settings.py` 中配置haystack使用的搜索引擎后端

```python
# 全文搜索引擎haystack 配置
# 不同的搜索引擎，配置不同，详情见官方文档
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch5_backend.Elasticsearch5SearchEngine',
        'URL': 'http://127.0.0.1:9200/',  # 此处为elasticsearch运行的服务器ip地址和端口
        'INDEX_NAME': 'meiduo',  # 指定elasticserach建立的索引库名称
    },
}

# 搜索结果每页显示数量
# HAYSTACK_SEARCH_RESULTS_PER_PAGE = 5

# 当添加、修改、删除数据时,实时更新index
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```

### 数据模型类

假设有一个商品模型类SKU

```python
from django.db import models


class SKU(models.Model):
    """商品SKU"""
    name = models.CharField(max_length=50, verbose_name='名称')
    caption = models.CharField(max_length=100, verbose_name='副标题')
    # 当前SKU商品从属的分类
    # 隐藏字段category_id是关联的分类的主键id
    category = models.ForeignKey(GoodsCategory, related_name='sku', on_delete=models.PROTECT, verbose_name='从属类别')
    price = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='单价')
    stock = models.IntegerField(default=0, verbose_name='库存')
    sales = models.IntegerField(default=0, verbose_name='销量')
    comments = models.IntegerField(default=0, verbose_name='评价数')
    is_launched = models.BooleanField(default=True, verbose_name='是否上架销售')

    class Meta:
        db_table = 'tb_sku'

    def __str__(self):
        return '%s: %s' % (self.id, self.name)
```

### 索引类

通过创建索引类，来指明让搜索引擎对哪些字段建立索引，也就是可以通过哪些字段的关键字来检索数据。

在应用中新建`search_indexes.py`文件，用于存放索引类

```python
from haystack import indexes
from .models import SKU


class SKUIndex(indexes.SearchIndex, indexes.Indexable):
    """SKU索引数据模型类"""
    text = indexes.CharField(document=True, use_template=False)

    name = indexes.CharField(model_attr='name')

    caption = indexes.CharField(model_attr='caption')

    category = indexes.CharField(model_attr='category')

    def get_model(self):
        """返回建立索引的模型类"""
        return SKU

    def index_queryset(self, using=None):
        """返回要建立索引的数据查询集"""
        return self.get_model().objects.filter(is_launched=True)
```

### 序列化器

#### SkuSerializer

```python
from rest_framework import serializers

class SkuSerializer(ModelSerializer):
    category = StringRelatedField(read_only=True)

    class Meta:
        model = SKU
        fields = ['id', 'name', 'caption', 'price', 'stock', 'comments', 'category', 'default_image']
```

#### HaystackSerializer

`HaystackSerializer`:根据`SearchQueryset`中可用的模型填充字段,进行序列化解析;此时，API返回的数据仅包含 Elasticsearch检索出匹配关键词的搜索结果。

```python
from drf_haystack.serializers import HaystackSerializer
from goods.search_indexes import SKUIndex

class SKUHaystackSerializer(HaystackSerializer):
    class Meta:
        """
        SKU索引结果数据序列化器
        """
        index_classes = [SKUIndex]
        fields = ('text', 'name', 'caption', 'category')
```

但是Haystack通过`Elasticsearch`检索出匹配关键词的搜索结果后，还会在数据库中取出完整的数据库模型类对象，放到搜索结果的`object`属性中，并将结果通过`SKUIndexSerializer`序列化器进行序列化。所以我们可以通过声明搜索结果的`object`字段以`SkuSerializer`序列化的形式进行处理，明确要返回的搜索结果中每个数据对象包含哪些字段。

```python
from drf_haystack.serializers import HaystackSerializer
from goods.search_indexes import SKUIndex


class SKUHaystackSerializer(HaystackSerializer):
    object = SkuSerializer(read_only=True)

    class Meta:
        """
        SKU索引结果数据序列化器:
        """
        index_classes = [SKUIndex]
        fields = ('text', 'object')
```

#### HaystackSerializerMixin

`HaystackSerializerMixin`：可以将此`mixin`混入类添加到序列化器中，以将实际对象用作序列化的数据源，而不是使用存储在搜索索引字段中的数据。

```python 
from drf_haystack.serializers import HaystackSerializerMixin


class SKUHaystackSerializer(HaystackSerializerMixin, SkuSerializer):
    class Meta(SkuSerializer.Meta):
        search_fields = ["text"]
```

### 视图

```python
class Page(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'size'
    page_query_param = 'page'


class SKUSearchView(HaystackViewSet):
    index_models = [SKU]
    serializer_class = SKUHaystackSerializer
    pagination_class = Page
```

### 路由

```python
from django.urls import path, include
from rest_framework import routers
from goods.views import *

router = routers.DefaultRouter()
router.register(r"search", SKUSearchView, basename="search")

urlpatterns = [
    path('', include(router.urls))
]
```

