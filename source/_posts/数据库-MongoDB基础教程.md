---
title: MongoDB基础教程
date: 2021-06-16 18:49:51
categories:
	- 技术
	- 数据库
	- MongoDB
tags:
	- MongoDB
	- 基础教程
---

## 基本概念

> 在`mongodb`中基本的概念是文档、集合、数据库

| SQL术语/概念    | MongoDB术语/概念 | 解释/说明                   |
| ----------- | ------------ | ----------------------- |
| database    | database     | 数据库                     |
| table       | collection   | 数据库表/集合                 |
| row         | document     | 数据记录行/文档                |
| column      | field        | 数据字段/域                  |
| index       | index        | 索引                      |
| table joins |              | 表连接,MongoDB不支持          |
| primary key | primary key  | 主键,MongoDB自动将_id字段设置为主键 |

通过下图实例，我们也可以更直观的了解Mongo中的一些概念：

![1myWPG](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2021/06/1myWPG.png)

## 库的操作

```shell
> use test  # use 库名， 创建库
switched to db test

> db  # 当前正在使用的库
test

> show dbs  # 没有刚创建的库，因为该库中没有任何数据
admin   0.000GB
config  0.000GB
local   0.000GB

> db.dropDatabase()  # 删除当前正在使用的库
{ "dropped" : "test", "ok" : 1 }
```

## 集合的操作

>   类似于MySQL的数据表

```shell
> db.createCollection("class")  # 创建一个叫 class 的集合
{ "ok" : 1 }

> show collections;  # 查看所有集合
class

> db.class.drop()  # 指明删除特定的集合
true
```

## 数据的操作

### 插入数据

>   -   一条数据在MongoDB中，被称为 **文档**, MySQL中被称为 **记录**
>
>   -   一列数据在MongoDB中，被称为 **域**, MySQL中被称为 **字段**

```shell
> db.class.insert({name:"H2009A", number:42}) # 插入一条数据，不指定 _id,根据ObjectId自动生成
WriteResult({ "nInserted" : 1 })

> db.class.insert({_id: "212312313", name:"H2103A", number:28}) # 插入一条数据,指定 _id, 重复，会抛出异常
WriteResult({ "nInserted" : 1 })

> db.class.insertMany([{name:"H2102A", number:18}, {name:"H2101A", number:54}])  # 一次插入多条数据
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("60c9e2e7f08e1d893ea18d4b"),
		ObjectId("60c9e2e7f08e1d893ea18d4c")
	]
}

```

以上实例中 `class` 是我们的集合名，如果该集合不在该数据库中， MongoDB 会自动创建该集合并插入文档。

### 查询数据

>   `find()`:类似于MySQL的 `all()`，查询全部

```shell
> db.class.find()
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : "212312313", "name" : "H2103A", "number" : 28 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
```

>   `findOne(条件, {字段:1, 字段: 0})`: 根据条件查询单个文档对象，指明文档对象中的哪个字段显示和不显示, 其中 `_id`字段,默认显示，不显示，指定为0即可。

```json
> db.students.findOne({name:"小名"})
{
	"_id" : ObjectId("60d13f5f9f3bfc393f9489a5"),
	"name" : "小名",
	"score" : 90,
	"class" : "H2009A"
}

> db.students.findOne({name:"小名"}, {name:1, class:1})
{
	"_id" : ObjectId("60d13f5f9f3bfc393f9489a5"),
	"name" : "小名",
	"class" : "H2009A"
}
> db.students.findOne({name:"小名"}, {_id:0, name:1, class:1})
{ "name" : "小名", "class" : "H2009A" }
```



### 更新数据

>   `update（条件, {$set: 文档}）`: 根据条件，更新特定字段的值，字段不存在，则新增

```shell
> db.class.update({ "_id" : "60c9dd57f08e1d893ea18d49"}, {$set:{number:34}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

### 删除数据

>   `remove(条件)`

```shell
> db.class.remove({_id: "60c9dd57f08e1d893ea18d49"})
WriteResult({ "nRemoved" : 1 })
```

## 高级查询

### 比较查询

```shell
> db.class.find({number:{$lt:32}})  # 小于
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }

> db.class.find({number:{$lte:42}})  # 小于等于
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }

> db.class.find({number:{$gt:42}})  # 大于
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }

> db.class.find({number:{$gte:42}})  #大于等于
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }

> db.class.find({number:42})  # 等于
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }

> db.class.find({number:{$ne:42}})  # 不等于
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
```

### 逻辑关系

```shell
> db.class.find({$or:[{number:{$gt:42}}, {number:{$lt:32}}]}) # 大于42 或 小于32
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }

> db.class.find({number:{$lt:42}, name: "H2008A"})  # 两个条件且，也就是同时成立
```

### 范围查询

```shell
> db.class.find({number:{$lt:52,$gt:32}})  # 大于32且小于52
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }

> db.class.find({number:{$in:[32, 34, 42]}})  # 任意一个值皆可
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }

> db.class.find({number:{$nin:[32, 54, 42]}})  # 不是任意一个值
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
```

### 分页

```shell
> db.class.find()  # 查询全部
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
> db.class.find().limit(2)  # 限制查询两条，也就是每页2条
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
> db.class.find().limit(2).skip(2)  # 每页2条，第2页
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
```

### 排序

```shell
> db.class.find().sort({number:1})  # 指定按照 key 升序排序
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
> db.class.find().sort({number:-1})  # 按照key降序排序
{ "_id" : ObjectId("60c9dd57f08e1d893ea18d49"), "name" : "H2008A", "number" : 54 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4c"), "name" : "H2101A", "number" : 54 }
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
{ "_id" : ObjectId("60c9e2e7f08e1d893ea18d4b"), "name" : "H2102A", "number" : 18 }
```

### 模糊查询

```shell
# 使用正则表达式匹配
> db.class.find({name:/^.*09.*$/})
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
> db.class.find({name:{$regex:'^H.*09A$'}})
{ "_id" : ObjectId("60c9e0c6f08e1d893ea18d4a"), "name" : "H2009A", "number" : 42 }
```

### 聚合

```json
// _id 字段， 指明 分组的字段， 当 _id 为 null时，默认不进行分组，直接聚合计算，有 max、min、sum、avg
// 对所有班级进行聚合计算
> db.class.aggregate([{
    $group:{
        _id:null,
        num_avg:{
            $avg:'$number'
        }, 
        num_total:{
            $sum: '$number'
        }, 
        max_num:{
            $max: '$number'
        }, 
        min_num:{
            $min: '$number'
        }
    }
}])

{ 
    "_id" : null, 
    "num_avg" : 42, 
    "num_total" : 168, 
    "max_num" : 54, 
    "min_num" : 18 
}

// 先按照班级分组，再进行聚合计算
> db.students.aggregate({$group: {_id:"$class", avg_score:{$avg: "$score"},max_score:{$max: "$score"}, min_score:{$min: "$score"} }})
{ "_id" : "H2008A", "avg_score" : 83.33333333333333, "max_score" : 100, "min_score" : 70 }
{ "_id" : "H2009A", "avg_score" : 95, "max_score" : 100, "min_score" : 90 }
```

## 数据关系

关系数据库（`RDBMS`）维护表之间的关系，以有意义的方式组织数据。而`MongoDB` 并没有像 `RDBMS` 那样的关系。但是，虽然文档数据库不需要与关系数据库相同的预定义结构，但这并不意味着它们不支持。实际上，`MongoDB` 允许通过嵌入式和引用式方法对文档之间的关系建立联系。

MongoDB 中的关系可以是：

-   1对1：一位作者对应一篇文章；
-   1对多：一位作者对应多篇文章；
-   多对1：多位作者对应一篇文章；
-   多对多：一位作者对应多篇文章，一篇文章对应多位作者；

现在，我们根据以上对应关系，来看下作者跟文章的文档结构。

下面是表示作者“jack”的文档结构：

```json
{ 
    "_id" : ObjectId("5e046286b1432f3ebcfd57d5"),
    "name" : "jack",
    "age" : 18,
    "gender" : "boy"
}
```

表示书籍信息的文档结构：

```json
{
    "_id" : ObjectId("5e0463f9b1432f3ebcfd57d7"),
    "book_name" : "第一本书",
    "published_date" : 2019,
    "price" : 99
}
```

### 嵌入式关系

现在使用嵌入式的方法，将书籍的文档嵌入到所属的作者文档中：

```json
> db.authors.find().pretty()
{
	"_id" : ObjectId("5e046866b1432f3ebcfd57d8"),
	"name" : "jack",
	"age" : "18",
	"gender" : "boy",
	"books" : [
		{
			"book_name" : "第一本书",
			"published_date" : "2019",
			"price" : "99"
		},
		{
			"book_name" : "第二本书",
			"published_date" : "2020",
			"price" : "199"
		}
	]
}
```

可以看到，现在多本书籍都跟其作者保存在同一个文档中。例如可以使用以下方式直接查询出这位作者的书籍信息：

```json 
> db.authors.findOne({name:"jack"}).books
[
	{
		"book_name" : "第一本书",
		"published_date" : "2019",
		"price" : "99"
	},
	{
		"book_name" : "第二本书",
		"published_date" : "2020",
		"price" : "199"
	}
]
```

该数据结构的优缺点：

-   **优点:** 快速、高效、简单
-   **缺点:** 如果数据量不断变大，会影响读写性能。需要定期去更新用户信息，该频度不好把握。（需要考虑原子性）
-   **使用场景:** 小的子文档、数据不经常改变、当最终一致性是可以接受的、文档增长小、经常需要进行二次查询来获取数据、读快。

### 引用关系

引用式关系是设计数据库时经常用到的方法，该方法把作者的数据文档和书籍的数据文档分开，再通过引用文档的“`_id`”字段来建立关系。

```json 
> db.authors.find().pretty()
{
    "_id" : ObjectId("5e046866b1432f3ebcfd57d8"),
    "name" : "jack",
    "age" : "18",
    "gender" : "boy",
    "books_ids" : [
        ObjectId("5e0463f9b1432f3ebcfd57d7"),
        ObjectId("5e046ba3b1432f3ebcfd57dd")
    ]
}
```

以上文档的`books_ids`字段保存每个书籍文档的对`_id`数组，而我们可以通过这些对象id获取每本书籍的信息。

该方法需要两次查询，首先找出作者的对象id，将其值先赋予到result上

```json
var result = db.author_book.findOne({name:"jack"}, {books_ids:1})
```

再根据查询出来的结果，在books数据库中再次进行查询，使用`$in`方法，查询出_id对象存在列表`result["books_ids"]`中书籍数据。

```json
var book = db.books.find({_id:{"$in": result["books_ids"]}})
```

该数据结构的优缺点：

-   **优点:** 文档大小更易于管理，两个文档的数据同步更新。
-   **缺点:** 查询耗时。
-   **使用场景:** 大的子文档、非易失性数据、当实时一致性是必要的、文档增长大、经常需要从结果中排除数据、写快。
