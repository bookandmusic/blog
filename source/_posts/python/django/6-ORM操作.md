---
title: Django基础(六)-ORM操作
date: 2021-01-23 22:37:39
categories:
  - python
  - Django
tags:
  - ORM操作
---

## 模型类

```python
# 品牌
class Brand(models.Model):
    name = models.CharField(max_length=20, verbose_name='品牌名')

    class Meta:
        db_table = 'tb_brand'


# 分类
class Cate(models.Model):
    name = models.CharField(max_length=20, verbose_name='分类名')

    class Meta:
        db_table = 'tb_cate'


# 商品表： id、name、price、history、stock、 sales
class Goods(models.Model):
    cate = models.ForeignKey(null=True, to=Cate, on_delete=models.CASCADE, verbose_name='分类')
    brand = models.ForeignKey(null=True, to=Brand, on_delete=models.CASCADE, verbose_name='品牌')
    name = models.CharField(max_length=20, verbose_name='商品名')
    price = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='价格')
    history = models.IntegerField(default=0, verbose_name='浏览量')
    stock = models.IntegerField(default=0, verbose_name='库存')
    sales = models.IntegerField(default=0, verbose_name='销量')

    class Meta:
        db_table = 'tb_goods'

# 订单：订单id、创建时间、更新时间、用户、总数、总价、实付金额、订单状态
class Order(models.Model):
    status_choices = (
        (0, '未支付'),
        (1, '未收货'),
        (2, '未评价'),
        (3, '已完成'),
    )
    order_id = models.CharField(max_length=64, verbose_name='订单号', primary_key=True)
    create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True, verbose_name='更新时间')
    user = models.ForeignKey(to=User, on_delete=models.CASCADE, verbose_name='用户')
    total = models.IntegerField(default=0, verbose_name='商品总数')
    total_amount = models.DecimalField(default=0, max_digits=10, decimal_places=2, verbose_name='商品总价')
    pay_amount = models.DecimalField(default=0, max_digits=10, decimal_places=2, verbose_name='实付金额')
    status = models.SmallIntegerField(choices=status_choices, verbose_name='订单状态')

    class Meta:
        db_table = 'tb_order'


# 订单商品表：id、 name、price、num、goods_id(外键)、order_id(外键)、amount
class OrderGoods(models.Model):
    order = models.ForeignKey(to=Order, related_name='order_goods', on_delete=models.CASCADE, verbose_name='订单')
    goods = models.ForeignKey(to=Goods, related_name='orders', on_delete=models.CASCADE, verbose_name='商品')
    name = models.CharField(max_length=20, verbose_name='商品名')
    price = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='价格')
    num = models.IntegerField(verbose_name='数量')
    amount = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='小计')

    class Meta:
        db_table = 'tb_order_goods'

```

## 查询

### 聚合

>   聚合查询函数是对一组值执行计算，并返回单个值。
>
>   Django 使用聚合查询前要先从 `django.db.models` 引入 `Avg`、`Max`、`Min`、`Count`、`Sum`（首字母大写）。

```python
from django.db.models import Avg,Max,Min,Count,Sum  #   引入函数
```

聚合查询返回值的数据类型是字典。

```python
aggregate(别名 = 聚合函数名("属性名称"))
```

1.  查询所有订单的总价

    ```python
    Order.objects.aggregate(sum=Sum("total_amount"))
    ```

2.  查询所有商品的平均价格

    ```python
    Goods.objects.aggregate(avg=Avg("price"))
    ```

### 分组

分组查询一般会用到聚合函数，所以使用前要先从 `django.db.models` 引入 `Avg`,`Max`,`Min`,`Count`,`Sum`（首字母大写）

```python
from django.db.models import Avg,Max,Min,Count,Sum  #   引入函数
```

**返回值：**

-   分组后，用 `values` 取值，则返回值是 `QuerySet` ,数据类型里面为一个个字典；
-   分组后，用 `values_list` 取值，则返回值是 `QuerySet` ,数据类型里面为一个个元组。

MySQL 中的 `limit` 相当于 ORM 中的 `QuerySet` 数据类型的切片。

**注意：**

`annotate` 里面放聚合函数。

-   **values 或者 values_list 放在 annotate 前面：**values 或者 values_list 是声明以什么字段分组，annotate 执行分组。
-   **values 或者 values_list 放在annotate后面：** annotate 表示直接以当前表的pk执行分组，values 或者 values_list 表示查询哪些字段， 并且要将 annotate 里的聚合函数起别名，在 values 或者 values_list 里写其别名。

1.  查询每天的订单量、销售额

    ```python
    Order.objects.\
    annotate(day=TruncDay('create_time')).\  # 将订单对象，添加一个新字段
    # 按照日期分组，然后对 分组结果 进行聚合计算
    values('day').annotate(total=Count('id'),sum=Sum('total_amount')).\
    # 从聚合结果中，查询哪些字段
    values('day', 'total', 'sum')
    ```

2.  查询每天商品的销售额

    ```python
    OrderGoods.objects.annotate(day=TruncDay('order__create_time')).values('day').annotate(sum=Sum('amount')).values('day', 'sum')
    ```

3.  查询每个品牌商品的销售额

    ```python
    OrderGoods.objects.values('goods__brand__name').annotate(sum=Sum('amount'))
    ```

4.  查询每类商品的销售额

    ```python
    OrderGoods.objects.values('goods__cate__name').annotate(sum=Sum('amount'))
    ```

5.  查询每天用户新增量

    ```python
    User.objects.annotate(day=TruncDay('date_joined')).values('day').annotate(n=Count('id'))
    ```

    



