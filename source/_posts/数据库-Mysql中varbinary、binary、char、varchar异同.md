---
title: Mysql中varbinary、binary、char、varchar异同
date: 2019-06-01 09:12:16
categories:
  - 技术
  - 数据库
  - mysql
tags:
  - varbinary
  - binary
  - char
  - varchar
---

binary 与 varbinary 类型和char与varchar类型是相似的，只是他们存储的是二进制数据，也就是说他们是包含字节流而不是字符流，他们有二进制字符的集合和顺序，他们的对比，排序是基于字节的数值进行的

binary与varbinary的最大长度和char与varchar是一样的，只不过他们是定义**字节长度**，而char和varchar对应的是**字符长度**。

#### 存储和取出时对尾部空格的处理

- char(N)用来存储非二进制字符串，插入时，对于少于N个字符的会自动在尾部加空格，查询时，尾部的空格就会被丢弃掉

- vachar(N) 用来存储非二进制字符串，插入时，对于少于N个字符的不填补空格，查询时，尾部的空格不会被丢弃掉

- binary(N)存储二进制字符串，插入进，少于N个字节的会自动在尾部加0x00，取出时，所有的字节都保留，返回定义长度的字节长度，在比较的时候，所有的字节都是有效的，并且0x00<space (space对应的是0x20)

- varbinary在插入不会去填补0x00字节，查询的时候也不会丢弃任何字节，在比较的时候，所有的字节都是有效的，并且0x00<space (space对应的是0x20)

#### 大小比较时

char与varchar的字符比较中，是忽略大小写与最后的空格的，如：

```bash
mysql> select 'a'='a ' , 'a'='A' , 'a'='A ';

+----------+---------+----------+
| 'a'='a ' | 'a'='A' | 'a'='A ' |
+----------+---------+----------+
|        1 |       1 |        1 |
+----------+---------+----------+
1 row in set (0.00 sec)
```

而binary及varbinary的字节比较中，所有的信息都不会被忽略，如：

```bash
mysql> create table t (c BINARY(3));
Query OK, 0 rows affected (0.01 sec)

mysql> insert into t set c = 'a';
Query OK, 1 row affected (0.01 sec)

mysql> select hex(c), c = 'a', c = 'a\0\0' from t;
+--------+---------+-------------+
| HEX(c) | c = 'a' | c = 'a\0\0' |
+--------+---------+-------------+
| 610000 |       0 |           1 |
+--------+---------+-------------+
1 row in set (0.08 sec)
```

