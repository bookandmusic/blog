---
title: DRF-序列化与反序列化
date: 2020-07-08 10:26:57
categories:
  - 技术
  - python
  - web
tags:
  - 序列化
  - 反序列化
  - DRF
---

DRF内置的序列化类，可以对整个模型类进行序列化与反序列化，简化了繁杂的工作。

<!--more-->

## 内置字段序列化

### 模型类

```python
from django.db import models


# Create your models here.
class Authors(models.Model):
    name = models.CharField(max_length=10, verbose_name='姓名')

    def __str__(self):
        return self.name

    class Meta:
        db_table = 'db_authors'
        verbose_name = '作者'
        verbose_name_plural = '作者'


class Books(models.Model):
    title = models.CharField(max_length=10, verbose_name='标题')
    price = models.DecimalField(max_digits=7, decimal_places=2, verbose_name='单价')
    publish = models.CharField(max_length=20, verbose_name='出版社')
    author = models.ForeignKey(Authors, on_delete=models.CASCADE)

    def __str__(self):
        return self.title

    class Meta:
        db_table = 'db_books'
        verbose_name = '图书'
        verbose_name_plural = '图书'

```

### 序列化类

#### 1. 全部字段序列化

```python
from rest_framework.serializers import ModelSerializer
from .models import Books, Authors
from rest_framework import serializers


class AuthorsSerializer(ModelSerializer):

    class Meta:
        model = Authors
        fields = '__all__'

        
class BooksSerializer(ModelSerializer):
   
    class Meta:
        model = Books
        fields = '__all__'
        
```

#### 2. 指定字段序列化

##### <1> 外键字段内置方式序列化

**图书类序列化**

```python
from rest_framework.serializers import ModelSerializer
from .models import Books, Authors
from rest_framework import serializers


class BooksSerializer(ModelSerializer):
    # author = serializers.PrimaryKeyRelatedField(read_only=True)  # 内置关联字段，默认序列化方式，关联对象的主键
    
    # author = serializers.StringRelatedField(read_only=True)  # 内置关联字段，此字段将被序列化为关联对象的字符串表示方式（即__str__方法的返回值）
    
    # author = serializers.HyperlinkedRelatedField(view_name='authors-detail', read_only=True)  # 内置关联字段，此字段被序列化为关联对象的路由地址

    # author = serializers.SlugRelatedField(read_only=True, slug_field='id')  # 内置关联字段，指定关联对象的字段
    
    # author = serializers.CharField(source='author.name', read_only=True)  # 指定关联对象的字段
    
    # author = AuthorsSerializer()  # 将关联对象所有字段全部序列化
    class Meta:
        model = Books
        fields = '__all__'
```

**作者类序列化**

```python
from rest_framework.serializers import ModelSerializer
from .models import Books, Authors
from rest_framework import serializers


class AuthorsSerializer(ModelSerializer):
    # books_set = serializers.PrimaryKeyRelatedField(read_only=True, many=True)
    # books_set = serializers.StringRelatedField(many=True)
    # books_set = serializers.HyperlinkedRelatedField(view_name='books-detail', read_only=True, many=True)
    # books_set = BookSerializer(many=True)
    # books_set = BookRelateField(read_only=True, many=True)  # 自定义关联字段

    class Meta:
        model = Authors
        fields = '__all__'

```

##### <2> 外键字段自定义方式序列化

```python
from rest_framework.serializers import ModelSerializer
from .models import Books, Authors
from rest_framework import serializers


class BookRelateField(serializers.RelatedField):
    def to_representation(self, value):
        return 'Author: {}'.format(value)


class AuthorsSerializer(ModelSerializer):
    books_set = BookRelateField(read_only=True, many=True)  # 自定义关联字段

    class Meta:
        model = Authors
        fields = '__all__'

class AuthorRelateField(serializers.RelatedField):
    def to_representation(self, value):
        return 'Author: {} {}'.format(value.id, value.name)


class BooksSerializer(ModelSerializer):
    author = AuthorRelateField(read_only=True) # 自定义关联字段
    
    class Meta:
        model = Books
        fields = '__all__'
       
```

以上序列化方式，任选其一均可，但是注意，此时只能保证**序列化成功**，不能保证反序列化

## 自定义字段序列化

> 最大好处, 在于将序列化字段与反序列化字段分离，互不影响

### <1> 修改模型类

需要在模型类中用`@property`来实现，可插拔

#### 模型类

```python
class Books(models.Model):
    title = models.CharField(max_length=10, verbose_name='标题')
    price = models.DecimalField(max_digits=7, decimal_places=2, verbose_name='单价')
    publish = models.CharField(max_length=20, verbose_name='出版社')
    author = models.ForeignKey(Authors, on_delete=models.CASCADE)

    @property  # 插拔字段 - 默认为read_only(不需要考虑反序列化)，且不能修改
    def author_name(self):
        return self.author.name

    @property
    def author_info(self):
        from .serializers import AuthorsSerializer
        return AuthorsSerializer(self.author).data

    def __str__(self):
        return self.title

```

#### 序列化类

```python
class BooksSerializer(ModelSerializer):

    class Meta:
        model = Books
        fields = ('id', 'title', 'price', 'author', 'author_name', 'author_info')

        extra_kwargs = {
            'author': {
                'write_only': True
            }
        }

```

### <2>修改序列化类

> 模型类不变，只在序列化器中添加序列化字段

#### 序列化类

```python
class BooksSerializer(ModelSerializer):
    author_url = serializers.HyperlinkedIdentityField(view_name='authors-detail', read_only=True)  # 内置序列化方式，关联对象的路由地址
    
    author_name = serializers.SerializerMethodField()
    # author_name 字段在数据库中不能存在，下面写方法的时候前面加 get_  就可以，这样就得到我们需要的数据了。

    @staticmethod
    def get_author_name(obj):
        """
        固定写法,obj代表Books实例对象,模型类配置了反向引用author代表作者对象
        """
        return obj.author.id

    class Meta:
        model = Books
        fields = ('id', 'title', 'price', 'author', 'author_name', 'author_url')

        extra_kwargs = {
            'author': {
                'write_only': True
            }
        }

```

