---
title: 数据库笔记
categories:
  - 技术
  - 数据库
  - mysql
tags:
  - SQL语句
date: 2019-05-27 21:23:45
---

学习数据库安装后，最重要的就是学习SQL语句。

<!--more-->

### 1. 数据语句操作类型

SQL是操作数据库的核心，

结构化查询语言(Structured Query Language)简称SQL，是一种特殊目的的编程语言，是一种数据库查询和程序设计语言，用于存取数据以及查询、更新和管理关系数据库系统；同时也是数据库脚本文件的扩展名。

SQL是最重要的关系数据库操作语言，并且它的影响已经超出数据库领域，得到其他领域的重视和采用，如人工智能领域的数据检索等。

SQL是关系模型的数据库应用语言，由IBM在20世纪70年代为其关系型数据库 System R 所开发。

SQL 是1986年10 月由美国国家标准局（ANSI）通过的数据库语言美国标准，接着，国际标准化组织（ISO）颁布了SQL正式国际标准。1989年4月，ISO提出了具有完整性特征的SQL89标准，1992年11月又公布了SQL92标准。

虽然各个数据库系统略有不同，但是他们基本均遵循SQL 92标准。或者在SQL 92上做了一些简单的扩展和变化。

学好了MySQL 的SQL 语法，其他的SQL语法学习起来均是万变不离其中。

SQL语句按照其功能范围不同可分为3个类别：

1. **数据定义语言**(DDL ，Data Defintion Language)语句：数据定义语句，用于定义不同的数据段、数据库、表、列、索引等。常用的语句关键字包括create、drop、alter等。
2. **数据操作语言**(DML ， Data Manipulation Language)语句：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据的完整性。常用的语句关键字主要包括insert、delete、update和select等。
3. **数据控制语言**(DCL， Data Control Language)语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括grant、revoke等。

### 2. 管理数据库命令

#### 元数据

以下命令语句可以在 MySQL 的命令提示符使用，获取服务器元数据

命令| 描述
:---:|:---
select version( ) | 服务器版本信息
select database( )|当前数据库名 (或者返回空)
select user( )|当前用户名
show status|服务器状态
show variables|服务器配置变量

#### 创建数据库

- create database 数据库名:
  创建数据库

```bash
mysql> create database data_test;
Query OK, 1 row affected (0.01 sec)
```

#### 删除数据库

- drop database 数据库名:
 删除数据库

```bash
mysql> drop database data_test;
Query OK, 0 rows affected (0.01 sec)
```

#### 展示所有数据库

- show databases:
  列出 MySQL 数据库管理系统的数据库列表。

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| data_test          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)
```

#### 改变数据库

- use 数据库名 :
  选择要操作的Mysql数据库，使用该命令后所有Mysql命令都只针对该数据库。

```bash
mysql> use data_test;
Database changed
```

#### 展示当前数据库中所有的表

- show tables:
显示指定数据库的所有表，使用该命令前需要使用 use 命令来选择要操作的数据库。

```bash
mysql> show tables;
+---------------------+
| Tables_in_data_test |
+---------------------+
| customers           |
| orderitems          |
| orders              |
| productnotes        |
| products            |
| vendors             |
+---------------------+
6 rows in set (0.00 sec)
```

#### 展示特定表中每列的信息

- show columns from 数据表(desc 数据表):
显示数据表的属性，属性类型，主键信息 ，是否为 NULL，默认值等其他信息。

```bash
mysql> show columns from customers;
+--------------+-----------+------+-----+---------+----------------+
| Field        | Type      | Null | Key | Default | Extra          |
+--------------+-----------+------+-----+---------+----------------+
| cust_id      | int(11)   | NO   | PRI | NULL    | auto_increment |
| cust_name    | char(50)  | NO   |     | NULL    |                |
| cust_address | char(50)  | YES  |     | NULL    |                |
| cust_city    | char(50)  | YES  |     | NULL    |                |
| cust_state   | char(5)   | YES  |     | NULL    |                |
| cust_zip     | char(10)  | YES  |     | NULL    |                |
| cust_country | char(50)  | YES  |     | NULL    |                |
| cust_contact | char(50)  | YES  |     | NULL    |                |
| cust_email   | char(255) | YES  |     | NULL    |                |
+--------------+-----------+------+-----+---------+----------------+
9 rows in set (0.00 sec)
```

#### 展示数据表的详细索引信息

- show index from 数据表:
显示数据表的详细索引信息，包括PRIMARY KEY（主键）。

```bash
mysql> show index from customers;
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table     | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| customers |          0 | PRIMARY  |            1 | cust_id     | A         |           5 |  NULL    |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.01 sec)

```

#### 展示数据库的性能及统计信息

- show table status [from db_name] [like 'pattern'] \G:
  该命令将输出Mysql数据库管理系统的性能及统计信息。

> 显示数据库 data_test 中所有表的信息

```bash
mysql> show table status from data_test;
+--------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
| Name         | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation       | Checksum | Create_options | Comment |
+--------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
| customers    | InnoDB |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |            0 |         0 |          10006 | 2019-05-26 10:15:02 | 2019-05-26 10:15:25 | NULL       | utf8_general_ci |     NULL |                |         |
| orderitems   | InnoDB |      10 | Dynamic    |   11 |           1489 |       16384 |               0 |        16384 |         0 |           NULL | 2019-05-26 10:15:02 | 2019-05-26 10:15:26 | NULL       | utf8_general_ci |     NULL |                |         |
| orders       | InnoDB |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |        16384 |         0 |          20010 | 2019-05-26 10:15:02 | 2019-05-26 10:15:25 | NULL       | utf8_general_ci |     NULL |                |         |
| productnotes | MyISAM |      10 | Dynamic    |   14 |            135 |        1892 | 281474976710655 |         6144 |         0 |            115 | 2019-05-26 10:15:02 | 2019-05-26 10:15:26 | NULL       | utf8_general_ci |     NULL |                |         |
| products     | InnoDB |      10 | Dynamic    |   14 |           1170 |       16384 |               0 |        16384 |         0 |           NULL | 2019-05-26 10:15:02 | 2019-05-26 10:15:25 | NULL       | utf8_general_ci |     NULL |                |         |
| vendors      | InnoDB |      10 | Dynamic    |    6 |           2730 |       16384 |               0 |            0 |         0 |           1007 | 2019-05-26 10:15:02 | 2019-05-26 10:15:25 | NULL       | utf8_general_ci |     NULL |                |         |
+--------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
6 rows in set (0.02 sec)
```

> 表名以cus开头的表的信息

```bash
mysql> show table status from data_test like "cus%";
+-----------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
| Name      | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation       | Checksum | Create_options | Comment |
+-----------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
| customers | InnoDB |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |            0 |         0 |          10006 | 2019-05-26 10:15:02 | 2019-05-26 10:15:25 | NULL       | utf8_general_ci |     NULL |                |         |
+-----------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+-----------------+----------+----------------+---------+
1 row in set (0.00 sec)
```

> 表名以cus开头的表的信息;
> 加上 \G，查询结果按列打印;\G后面不能再加分号;
> 因为\G在功能上等同于;
> 如果加了分号，那么就是;;(2个分号)，SQL语法错误`ERROR:No query specified`

```bash
mysql> show table status from data_test like "cus%"\G
*************************** 1. row ***************************
           Name: customers
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 3276
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 10006
    Create_time: 2019-05-26 10:15:02
    Update_time: 2019-05-26 10:15:25
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)
```

### 3. 数据类型

#### 数值类型

mysql支持所有标准sql数值数据类型。

这些类型包括严格数值数据类型（integer、smallint、decimal和numeric），以及近似数值数据类型（float、real和double precisi键字int是integer的同义词，关键字dec是decimal的同义词。

bit数据类型保存位字段值，并且支持myisam、memory、innodb和bdb表。

作为sql标准的扩展，mysql也支持整数类型tinyint、mediumint和bigint。下面的表显示了需要的每个整数类型的存储和范围。

|          类型          |     大小     | 范围（有符号）                                                                                                                      | 范围（无符号）                                                    |         用途         |
| :--------------------: | :----------: | :---------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------- | :------------------: |
|      **tinyint**       |    1 字节    | (-128，127)                                                                                                                         | (0，255)                                                          |       小整数值       |
|        smallint        |    2 字节    | (-32 768，32 767)                                                                                                                   | (0，65 535)                                                       |       大整数值       |
|       mediumint        |    3 字节    | (-8 388 608，8 388 607)                                                                                                             | (0，16 777 215)                                                   |       大整数值       |
| **int** 或 **integer** |    4 字节    | (-2 147 483 648，2 147 483 647)                                                                                                     | (0，4 294 967 295)                                                |       大整数值       |
|         bigint         |    8 字节    | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)                                                                             | (0，18 446 744 073 709 551 615)                                   |      极大整数值      |
|       **float**        |    4 字节    | (-3.402 823 466 e+38，-1.175 494 351 e-38)，0，(1.175 494 351 e-38，3.402 823 466 351 e+38)                                         | 0，(1.175 494 351 e-38，3.402 823 466 e+38)                       | 单精度<br/>浮点数值  |
|       **double**       |    8 字节    | (-1.797 693 134 862 315 7 e+308，-2.225 073 858 507 201 4 e-308)，0，(2.225 073 858 507 201 4 e-308，1.797 693 134 862 315 7 e+308) | 0，(2.225 073 858 507 201 4 e-308，1.797 693 134 862 315 7 e+308) | 双精度<br/> 浮点数值 |
|      **decimal**       | decimal(m,d) | 依赖于 m 和 d 的值                                                                                                                  | 依赖于 m 和 d 的值                                                |        小数值        |

* * *

#### 日期和时间类型

表示时间值的日期和时间类型为 datetime、date、timestamp、time 和 year。

每个时间类型有一个有效值范围和一个 "零" 值，当指定不合法的 mysql 不能表示的值时使用 "零" 值。

timestamp 类型有专有的自动更新特性，将在后面描述。

|     类型      | 大小(字节) | 范围                                                                                                                                   |        格式         |           用途           |
| :-----------: | :--------: | :------------------------------------------------------------------------------------------------------------------------------------- | :-----------------: | :----------------------: |
|   **date**    |     3      | 1000-01-01/9999-12-31                                                                                                                  |     yyyy-mm-dd      |          日期值          |
|   **time**    |     3      | '-838:59:59'/'838:59:59'                                                                                                               |      hh:mm:ss       |     时间值或持续时间     |
|     year      |     1      | 1901/2155                                                                                                                              |        yyyy         |          年份值          |
| **datetime**  |     8      | 1000-01-01 00:00:00/9999-12-31 23:59:59                                                                                                | yyyy-mm-dd hh:mm:ss |     混合日期和时间值     |
| **timestamp** |     4      | 1970-01-01 00:00:00/2038<br/>结束时间是第 **2147483647** 秒<br/>北京时间 **2038-1-19 11:14:07**<br/>格林尼治时间**2038-1-19 03:14:07** |   yyyymmdd hhmmss   | 混合日期和时间值，时间戳 |

* * *

#### 字符串类型

字符串类型指 char、varchar、binary、varbinary、blob、text、enum 和 set。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

| 类型        | 大小                 | 用途                            |
| :---------- | :------------------- | :------------------------------ |
| **char**    | 0-255 字节           | 定长字符串                      |
| **varchar** | 0-65535 字节         | 变长字符串                      |
| tinyblob    | 0-255 字节           | 不超过 255 个字符的二进制字符串 |
| tinytext    | 0-255 字节           | 短文本字符串                    |
| **blob**    | 0-65 535 字节        | 二进制形式的长文本数据          |
| **text**    | 0-65 535 字节        | 长文本数据                      |
| mediumblob  | 0-16 777 215 字节    | 二进制形式的中等长度文本数据    |
| mediumtext  | 0-16 777 215 字节    | 中等长度文本数据                |
| longblob    | 0-4 294 967 295 字节 | 二进制形式的极大文本数据        |
| longtext    | 0-4 294 967 295 字节 | 极大文本数据                    |

char 和 varchar 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

binary 和 varbinary 类似于 char 和 varchar，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

blob 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 blob 类型：tinyblob、blob、mediumblob 和 longblob。它们区别在于可容纳存储范围不同。

有 4 种 text 类型：tinytext、text、mediumtext 和 longtext。对应的这 4 种 blob 类型，可存储的最大长度不同，可根据实际情况选择。

> 注意：

1、一个汉字占多少长度与编码有关：

UTF－8：一个汉字＝3个字节

GBK：一个汉字＝2个字节

2、varchar(n) 表示 n 个字符，无论汉字和英文，Mysql 都能存入 n 个字符，仅是实际字节长度有所区别

3、MySQL 检查长度，可用 SQL 语言来查看：

```bash
select length(fieldname) from table_name
```

### 4. 创建数据表

创建MySQL数据表需要以下信息：

- 表名
- 表字段名
- 定义每个表字段

**语法**
以下为创建MySQL数据表的SQL通用语法：

```bash
create table table_name (column_name column_type);
```

**实例**
以下例子中我们将在 my_data 数据库中创建数据表students：

```bash
mysql> create table if not exists students(id int unsigned auto_increment, name varchar(40) not null, adress varchar(100) , birthday date not null, primary key(id)) engine=innodb default charset=utf8mb4;
Query OK, 0 rows affected (0.03 sec)

mysql> show columns from students;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id       | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name     | varchar(40)      | NO   |     | NULL    |                |
| adress   | varchar(100)     | YES  |     | NULL    |                |
| birthday | date             | NO   |     | NULL    |                |
+----------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

实例解析：

- 如果你不想字段为 null 可以设置字段的属性为 not null， 在操作数据库时如果输入该字段的数据为null ，就会报错。
- auto_increment定义列为自增的属性，一般用于主键，数值会自动加1。
- primary key关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。
- engine 设置存储引擎，charset 设置编码。

### 5. 删除数据表

MySQL中删除数据表是非常容易操作的， 但是你再进行删除表操作时要非常小心，因为执行删除命令后所有数据都会消失。

**语法**
以下为删除MySQL数据表的通用语法：

```bash
drop table table_name;
```

**实例**
以下实例中我们将删除 students 表:

```bash
mysql> drop table students;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
Empty set (0.00 sec)
```

### 6. 插入数据

MySQL 表中使用 insert into SQL语句来插入数据。

**语法**
以下为向MySQL数据表插入数据通用的 INSERT INTO SQL语法：

```bash
insert into table_name ( field1, field2,...fieldN ) values ( value1, value2,...valueN );
```

insert 插入多条数据

```bash
insert into table_name  (field1, field2,...fieldn)  values  (valuea1,valuea2,...valuean),(valueb1,valueb2,...valuebn),(valuec1,valuec2,...valuecn)......;
```

如果数据是字符型，必须使用单引号或者双引号，如："value"。

**实例**
以下实例中我们将向 students 表插入数据:

```bash
# 主键默认从1开始
mysql> insert into students (name,birthday) values("丽丽","1996-05-06");
Query OK, 1 row affected (0.00 sec)

# 主键设为0，即从最后一个id值自动增长
mysql> insert into students values(0,"李明",null,"1993-12-16");
Query OK, 1 row affected (0.01 sec)

mysql> insert into students values(10,"张磊",'北京市海淀区',"1995-04-12");
Query OK, 1 row affected (0.00 sec)

# 主键不设置，从最后一个id值自动增长
mysql> insert into students set name="孙雨",adress='河北省石家庄市',birthday="1989-07-18";
Query OK, 1 row affected (0.01 sec)

```

> 如果添加过主键自增（PRINARY KEY AUTO_INCREMENT）第一列在增加数据的时候，可以写为0或者null，这样添加数据可以自增

### 7. 条件语句

- 查询语句中你可以使用一个或者多个表，表之间使用逗号, 分割，并使用where语句来设定查询条件。
- where 子句类似于程序语言中的 if 条件，根据 MySQL 表中的字段值来读取指定的数据。
- 以下为操作符列表，可用于 where 子句中。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以使用 and 或者 or 指定一个或多个条件。
- WHERE 子句也可以运用于 SQL 的 delete 或者 update 命令。

> 下表中实例假定 A 为 10, B 为 20

| 操作符 | 描述                                                                                     | 实例                 |
| :----: | :--------------------------------------------------------------------------------------- | :------------------- |
|   =    | 等号，检测两个值是否相等，如果相等返回true                                               | (A = B) 返回false。  |
|   !=   | 不等于，检测两个值是否相等，如果不相等返回true                                           | (A != B) 返回 true。 |
|   >    | 大于号，检测左边的值是否大于右边的值, 如果左边的值大于右边的值返回true                   | (A > B) 返回false。  |
|   <    | 小于号，检测左边的值是否小于右边的值, 如果左边的值小于右边的值返回true                   | (A < B) 返回 true。  |
|   >=   | 大于等于号，检测左边的值是否大于或等于右边的值, 如果左边的值大于或等于右边的值返回true   | (A >= B) 返回false。 |
|   <=   | 小于等于号，检测左边的值是否小于于或等于右边的值, 如果左边的值小于或等于右边的值返回true | (A <= B) 返回 true。 |


如果我们想在 MySQL 数据表中读取指定的数据，where 子句是非常有用的。

使用主键来作为 where 子句的条件查询是非常快速的。

如果给定的条件在表中没有任何匹配的记录，那么查询不会返回任何数据。

### 8. 查询数据

MySQL 数据库使用SQL select语句来查询数据。

**语法**
以下为在MySQL数据库中查询数据通用的 select 语法

```bash
select column_name,column_name
from table_name
[where Clause]
[limit N][ offset M]
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号(,)分割，并使用where语句来设定查询条件。
- select 命令可以读取一条或者多条记录。
- 你可以使用星号（*）来代替其他字段，select语句会返回表的所有字段数据
- 你可以使用 limit 属性来设定返回的记录数。
- 你可以通过offset指定select语句开始查询的数据偏移量。默认情况下偏移量为0。

**实例**
以下实例我们将通过 SQL select 命令来获取 MySQL 数据表 students 的数据：

```bash
mysql> select * from students;
+----+------+----------------+------------+
| id | name | adress         | birthday   |
+----+------+----------------+------------+
|  1 | 丽丽 | NULL           | 1996-05-06 |
|  2 | 李明 | NULL           | 1993-12-16 |
| 10 | 张磊 | 北京市海淀区    | 1995-04-12 |
| 11 | 孙雨 | 河北省石家庄市  | 1989-07-18 |
+----+------+----------------+------------+
4 rows in set (0.00 sec)

mysql> select id,name from students limit 2 offset 1;
+----+------+
| id | name |
+----+------+
|  2 | 李明 |
| 10 | 张磊 |
+----+------+
2 rows in set (0.00 sec)

mysql> select * from students where id>=10;
+----+------+----------------+------------+
| id | name | adress         | birthday   |
+----+------+----------------+------------+
| 10 | 张磊 | 北京市海淀区   | 1995-04-12 |
| 11 | 孙雨 | 河北省石家庄市 | 1989-07-18 |
+----+------+----------------+------------+
2 rows in set (0.00 sec)

mysql> select * from students where id>=10 limit 1 offset 1;
+----+------+----------------+------------+
| id | name | adress         | birthday   |
+----+------+----------------+------------+
| 11 | 孙雨 | 河北省石家庄市 | 1989-07-18 |
+----+------+----------------+------------+
1 row in set (0.00 sec)

mysql> select * from students where name="李明";
+----+------+--------+------------+
| id | name | adress | birthday   |
+----+------+--------+------------+
|  2 | 李明 | NULL   | 1993-12-16 |
+----+------+--------+------------+
1 row in set (0.00 sec)
```

### 9. 修改数据

如果我们需要修改或更新 MySQL 中的数据，我们可以使用 SQL update 命令来操作。

**语法**
以下是 update 命令修改 MySQL 数据表数据的通用 SQL 语法：

```bash
update table_name set field1=new-value1, field2=new-value2 [where clause]
```

当我们需要将字段中的特定字符串批量修改为其他字符串时，可已使用以下操作：

```bash
update table_name set field=replace(field, 'old-string', 'new-string') [where clause]
```

- 你可以同时更新一个或多个字段。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以在一个单独表中同时更新数据。
当你需要更新数据表中指定行的数据时 where 子句是非常有用的。

**实例**
以下我们将在 SQL update 命令使用 where 子句来更新 students 表中指定的数据：

```bash
mysql> update students set adress="北京市昌平区" where adress is null;
Query OK, 2 rows affected (0.01 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from students;
+----+------+----------------+------------+
| id | name | adress         | birthday   |
+----+------+----------------+------------+
|  1 | 丽丽 | 北京市昌平区   | 1996-05-06 |
|  2 | 李明 | 北京市昌平区   | 1993-12-16 |
| 10 | 张磊 | 北京市海淀区   | 1995-04-12 |
| 11 | 孙雨 | 河北省石家庄市 | 1989-07-18 |
+----+------+----------------+------------+
4 rows in set (0.00 sec)

mysql> update students set adress=replace(adress, "河北省石家庄","湖北省武汉") wh
ere id=11;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from students;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
| 10 | 张磊 | 北京市海淀区 | 1995-04-12 |
| 11 | 孙雨 | 湖北省武汉市 | 1989-07-18 |
+----+------+--------------+------------+
4 rows in set (0.00 sec)

mysql> update students set id=id-7 where id=10;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from students;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
| 11 | 孙雨 | 湖北省武汉市 | 1989-07-18 |
+----+------+--------------+------------+
4 rows in set (0.00 sec)
```

### 10. 删除数据

你可以使用 sql 的 delete from 命令来删除 mysql 数据表中的记录。

**语法**
以下是 sql delete 语句从 mysql 数据表中删除数据的通用语法：

```bash
delete from table_name [where clause]
```

- 如果没有指定 where 子句，mysql 表中的所有记录将被删除。
- 你可以在 where 子句中指定任何条件
- 您可以在单个表中一次性删除记录。
当你想删除数据表中指定的记录时 where 子句是非常有用的。

**实例**
这里我们将在 sql delete 命令中使用 where 子句来删除 mysql 数据表 students 所选的数据:

```bash
mysql> delete from students where id=11;
Query OK, 1 row affected (0.01 sec)

mysql> select * from students;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)
```

> delete，drop，truncate 都有删除表的作用，区别在于：

1、drop table table_name : 删除表全部数据和表结构，立刻释放磁盘空间，不管是 Innodb 和 MyISAM;

实例，删除学生表：

```bash
drop table students;
```

2、truncate table table_name : 删除表全部数据，保留表结构，立刻释放磁盘空间 ，不管是 Innodb 和 MyISAM;

实例，删除学生表：

```bash
truncate table students;
```

3、delete from table_name : 删除表全部数据，表结构不变，对于 MyISAM 会立刻释放磁盘空间，InnoDB 不会释放磁盘空间;

实例，删除学生表：

```bash
delete from students;
```

4、delete from table_name where xxx : 带条件的删除，表结构不变，不管是 innodb 还是 MyISAM 都不会释放磁盘空间;

实例，删除学生表中姓名为 "张三" 的数据：

```bash
delete from student where name = "张三";
```

5、delete 操作以后，使用 optimize table table_name 会立刻释放磁盘空间，不管是 innodb 还是 myisam;

实例，删除学生表中姓名为 "张三" 的数据：

```bash
delete from student where name = "张三";
```

实例，释放学生表的表空间：

```bash
optimize table students;
```

6、delete from 表以后虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以使用这部分空间。

### 11. 包含条件

我们知道在 mysql 中使用 sql select 命令来读取数据， 同时我们可以在 select 语句中使用 where 子句来获取指定的记录。

where 子句中可以使用等号 = 来设定获取数据的条件，如 "adress = '北京市昌平区'"。

但是有时候我们需要获取 adress 字段含有 "北京市" 字符的所有记录，这时我们就需要在 where 子句中使用 sql like 子句。

**语法**
以下是 sql select 语句使用 like 子句从数据表中读取数据的通用语法：

```bash
select field1, field2,...fieldn from table_name where field1 like condition1 [and [or]] filed2 = 'somevalue'
```

- sql like 子句中使用百分号 %字符来表示任意字符，类似于unix或正则表达式中的星号 *。
- 如果没有使用百分号 %, like 子句与等号 = 的效果是一样的。
- like 通常与 % 一同使用，类似于一个元字符的搜索。
- 可以使用 and 或者 or 指定一个或多个条件。
- 可以在 delete、select或 update 命令中使用 where...like 子句来指定条件。

**实例**
以下我们将在 sql select 命令中使用 where...like 子句来从mysql数据表 students 中读取数据。

```bash
mysql> select * from students where adress like "北京市%";
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)
```

> 在 where like 的条件查询中，SQL 提供了四种匹配方式。

- `%`：表示任意 0 个或多个字符。可匹配任意类型和长度的字符，有些情况下若是中文，请使用两个百分号（%%）表示。
- `_`：表示任意单个字符。匹配单个任意字符，它常用来限制表达式的字符长度语句。
- `[]`：表示括号内所列字符中的一个（类似正则表达式）。指定一个字符、字符串或范围，要求所匹配对象为它们中的任一个。
- `[^]`：表示不在括号所列之内的单个字符。其取值和 [] 相同，但它要求所匹配对象为指定字符以外的任一个字符。
- 查询内容包含通配符时,由于通配符的缘故，导致我们查询特殊字符 “%”、“_”、“[” 的语句无法正常实现，而把特殊字符用 “[ ]” 括起便可正常查询。

> like 匹配/模糊匹配，会与 % 和 _ 结合使用。

```bash
'%a'     //以a结尾的数据
'a%'     //以a开头的数据
'%a%'    //含有a的数据
'_a_'    //三位且中间字母是a的
'_a'     //两位且结尾字母是a的
'a_'     //两位且开头字母是a的
```

### 12. 关联查询

MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

**语法**
mysql union 操作符语法格式：

```bash
select expression1, expression2, ... expression_n
from tables
[where conditions]
union [all | distinct]
select expression1, expression2, ... expression_n
from tables
[where conditions];
```

**参数**

- expression1, expression2, ... expression_n: 要检索的列。

- tables: 要检索的数据表。

- where conditions: 可选， 检索条件。

- distinct: 可选，删除结果集中重复的数据。默认情况下 union 操作符已经删除了重复数据，所以 distinct 修饰符对结果没啥影响。

- all: 可选，返回所有结果集，包含重复数据。

**实例**
下面的 SQL 语句从 "students" 和 "teachers" 表中选取所有不同的adress（只有不同的值）：

```bash
mysql> select * from students;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)

mysql> select * from teachers;
+----+--------+--------------+----------+
| id | name   | adress       | birthday |
+----+--------+--------------+----------+
|  1 | 刘老师 | 北京市海淀区 | NULL     |
|  2 | 孙老师 | 北京市朝阳区 | NULL     |
|  3 | 常老师 | 北京市昌平区 | NULL     |
+----+--------+--------------+----------+
3 rows in set (0.00 sec)

mysql> select adress from students union select adress from teachers ;
+--------------+
| adress       |
+--------------+
| 北京市昌平区 |
| 北京市海淀区 |
| 北京市朝阳区 |
+--------------+
3 rows in set (0.00 sec)

mysql> select name from students where adress like "%昌平区" union select name from teachers where adress like "%昌平区" ;
+--------+
| name   |
+--------+
| 丽丽   |
| 李明   |
| 常老师 |
+--------+
3 rows in set (0.00 sec)
```

> UNION 语句：用于将不同表中相同列中查询的数据展示出来；（不包括重复数据）
> UNION ALL 语句：用于将不同表中相同列中查询的数据展示出来；（包括重复数据）

### 13. 排序

我们知道从 mysql 表中使用 sql select 语句来读取数据。

如果我们需要对读取的数据进行排序，我们就可以使用 mysql 的 order by 子句来设定你想按哪个字段哪种方式来进行排序，再返回搜索结果。

**语法**
以下是 sql select 语句使用 order by 子句将查询数据排序后再返回数据：

```bash
select field1, field2,...fieldn table_name1, table_name2... order by field1 [asc [desc][默认 asc]], [field2...] [asc [desc][默认 asc]]
```

- 你可以使用任何字段来作为排序的条件，从而返回排序后的查询结果。
- 你可以设定多个字段来排序。
- 你可以使用 asc 或 desc 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。
- 你可以添加 where...like 子句来设置条件。

**实例**
尝试以下实例，结果将按升序及降序排列。

```bash
mysql> select * from students where adress like "北京市%" order by birthday;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)

mysql> select * from students where adress like "北京市%" order by birthday desc;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)

mysql> select * from students where adress like "北京市%" order by birthday asc;
+----+------+--------------+------------+
| id | name | adress       | birthday   |
+----+------+--------------+------------+
|  2 | 李明 | 北京市昌平区 | 1993-12-16 |
|  3 | 张磊 | 北京市海淀区 | 1995-04-12 |
|  1 | 丽丽 | 北京市昌平区 | 1996-05-06 |
+----+------+--------------+------------+
3 rows in set (0.00 sec)
```

### 14. 分组

group by 语句根据一个或多个列对结果集进行分组。
在分组的列上我们可以使用 count, sum, avg等函数。

**语法**
以下是gruop by语句的语法示例：

```bash
select column_name, function(column_name) from table_name where column_name operator value group by column_name;
```

**实例**
接下来我们使用 group by 语句 将数据表按名字进行分组，并统计每个商品有多少条记录：

```bash
mysql> select * from goods;
+----+--------+------+
| id | name   | nums |
+----+--------+------+
|  1 | 西瓜   |   50 |
|  2 | 甜瓜   |   15 |
|  3 | 甜瓜   |   15 |
|  4 | 苹果   |   25 |
|  5 | 西瓜   |   25 |
|  6 | 西瓜   |   63 |
+----+--------+------+
6 rows in set (0.00 sec)

mysql> select name, count(name) from goods group by name;
+--------+-------------+
| name   | count(name) |
+--------+-------------+
| 甜瓜   |           2 |
| 苹果   |           1 |
| 西瓜   |           3 |
+--------+-------------+
3 rows in set (0.00 sec)

```

with rollup 可以实现在分组统计数据基础上再进行相同的统计（sum,avg,count…）。

例如我们将以上的数据表按商品名称进行分组，再统计每类商品的总数量，或者求其均值：

```bash
mysql> select name, sum(nums) as count_num from goods group by name;
+--------+-----------+
| name   | count_num |
+--------+-----------+
| 甜瓜   |        30 |
| 苹果   |        25 |
| 西瓜   |       138 |
+--------+-----------+
3 rows in set (0.00 sec)

mysql> select name, avg(nums) as avg_num from goods group by name with rollup;
+--------+---------+
| name   | avg_num |
+--------+---------+
| 甜瓜   | 15.0000 |
| 苹果   | 25.0000 |
| 西瓜   | 46.0000 |
| NULL   | 32.1667 |
+--------+---------+
4 rows in set (0.00 sec)

```

我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce 语法：

```bash
select coalesce(a,b,c);
```

参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。

以下实例中如果名字为空我们使用总数代替：

```bash
mysql> select coalesce(name, '总计') as 名称, sum(nums) as 数量 from goods group by name with rollup;
+--------+--------+
| 名称   | 数量   |
+--------+--------+
| 甜瓜   |     30 |
| 苹果   |     25 |
| 西瓜   |    138 |
| 总计   |    193 |
+--------+--------+
4 rows in set (0.00 sec)

```

### 15. 连接

在前面的章节中，我们已经学会了如何在一张表中读取数据，这是相对简单的，但是在真正的应用中经常需要从多个数据表中读取数据。

本章节我们将向大家介绍如何使用 mysql 的 join 在两个或多个表中查询数据。

你可以在 select, update 和 delete 语句中使用 mysql 的 join 来联合多表查询。

join 按照功能大致分为如下三类：

- inner join（内连接,或等值连接）：获取两个表中字段匹配关系的记录。
- left join（左连接）：获取左表所有记录，即使右表没有对应匹配的记录。
- right join（右连接）： 与 left join 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

**实例**
首先创建表，并添加测试数据

```bash
mysql> create table web_counts(id int unsigned auto_increment primary key, web_name varchar(20) not null, nums int unsigned default 0) default charset=utf8;
Query OK, 0 rows affected (0.10 sec)

mysql> create table web_info(id int unsigned auto_increment primary key, web_name varchar(20) not null, web_info varchar(50), create_time date not null) default charset=utf8;
Query OK, 0 rows affected (0.00 sec)

mysql> desc web_info;
+-------------+------------------+------+-----+---------+----------------+
| Field       | Type             | Null | Key | Default | Extra          |
+-------------+------------------+------+-----+---------+----------------+
| id          | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| web_name    | varchar(20)      | NO   |     | NULL    |                |
| web_info    | varchar(50)      | YES  |     | NULL    |                |
| create_time | date             | NO   |     | NULL    |                |
+-------------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

mysql> desc web_counts;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id       | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| web_name | varchar(20)      | NO   |     | NULL    |                |
| nums     | int(10) unsigned | YES  |     | 0       |                |
+----------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

mysql> insert into web_counts values(0, " 淘宝",1000);
Query OK, 1 row affected (0.03 sec)

mysql> insert into web_counts values(0, "百度", 3000);
Query OK, 1 row affected (0.00 sec)

mysql> insert into web_counts values(0, "腾讯", 2000);
Query OK, 1 row affected (0.00 sec)

mysql> insert into web_counts values(0, "网易", 900);
Query OK, 1 row affected (0.00 sec)

mysql> select * from web_counts;
+----+----------+------+
| id | web_name | nums |
+----+----------+------+
|  1 | 淘宝     | 1000 |
|  2 | 百度     | 3000 |
|  3 | 腾讯     | 2000 |
|  4 | 网易     |  900 |
+----+----------+------+
4 rows in set (0.00 sec)

mysql> insert into web_info values(0, "百度", "搜索网站","1989-9-01");
Query OK, 1 row affected (0.00 sec)

mysql> insert into web_info values(0, "腾讯", "社交网站","1991-02-05");
Query OK, 1 row affected (0.00 sec)

mysql> insert into web_info values(0, "网易", "门户网站","1990-08-09");
Query OK, 1 row affected (0.00 sec)

mysql> insert into web_info values(0, "新浪", "社交网站","1993-05-12");
Query OK, 1 row affected (0.00 sec)

mysql> select * from web_info;
+----+----------+--------------+-------------+
| id | web_name | web_info     | create_time |
+----+----------+--------------+-------------+
|  1 | 百度     | 搜索网站     | 1989-09-01  |
|  2 | 腾讯     | 社交网站     | 1991-02-05  |
|  3 | 网易     | 门户网站     | 1990-08-09  |
|  4 | 新浪     | 社交网站     | 1993-05-12  |
+----+----------+--------------+-------------+
4 rows in set (0.01 sec)

```

接下来我们就使用mysql的inner join(也可以省略 inner 使用 join，效果一样)来连接以上两张表来读取web_counts表中所有web_name字段在web_info表对应的字段值：

```bash
mysql> select b.id, a.web_name, a.nums,b.web_info, b.create_time from web_counts a inner join web_info b on a.web_name=b.web_name;
+----+----------+------+--------------+-------------+
| id | web_name | nums | web_info     | create_time |
+----+----------+------+--------------+-------------+
|  1 | 百度     | 3000 | 搜索网站     | 1989-09-01  |
|  2 | 腾讯     | 2000 | 社交网站     | 1991-02-05  |
|  3 | 网易     |  900 | 门户网站     | 1990-08-09  |
+----+----------+------+--------------+-------------+
3 rows in set (0.00 sec)

```

以上 SQL 语句等价于：

```bash
mysql> select b.id, a.web_name, a.nums,b.web_info, b.create_time from web_counts a, web_info b where a.web_name=b.web_name;
+----+----------+------+--------------+-------------+
| id | web_name | nums | web_info     | create_time |
+----+----------+------+--------------+-------------+
|  1 | 百度     | 3000 | 搜索网站     | 1989-09-01  |
|  2 | 腾讯     | 2000 | 社交网站     | 1991-02-05  |
|  3 | 网易     |  900 | 门户网站     | 1990-08-09  |
+----+----------+------+--------------+-------------+
3 rows in set (0.00 sec)

```

mysql left join 与 join 有所不同。 mysql left join 会读取左边数据表的全部数据，即便右边表无对应数据。

```bash
mysql> select a.id, a.web_name, a.nums,b.web_info, b.create_time from web_counts a left join web_info b on a.web_name=b.web_name order by a.id;
+----+----------+------+--------------+-------------+
| id | web_name | nums | web_info     | create_time |
+----+----------+------+--------------+-------------+
|  1 | 淘宝     | 1000 | NULL         | NULL        |
|  2 | 百度     | 3000 | 搜索网站     | 1989-09-01  |
|  3 | 腾讯     | 2000 | 社交网站     | 1991-02-05  |
|  4 | 网易     |  900 | 门户网站     | 1990-08-09  |
+----+----------+------+--------------+-------------+
4 rows in set (0.00 sec)

```

mysql right join 会读取右边数据表的全部数据，即便左边边表无对应数据。

```bash
mysql> select b.id, b.web_name, a.nums,b.web_info, b.create_time from web_counts a right join web_info b on a.web_name=b.web_name order by b.id;
+----+----------+------+--------------+-------------+
| id | web_name | nums | web_info     | create_time |
+----+----------+------+--------------+-------------+
|  1 | 百度     | 3000 | 搜索网站     | 1989-09-01  |
|  2 | 腾讯     | 2000 | 社交网站     | 1991-02-05  |
|  3 | 网易     |  900 | 门户网站     | 1990-08-09  |
|  4 | 新浪     | NULL | 社交网站     | 1993-05-12  |
+----+----------+------+--------------+-------------+
4 rows in set (0.01 sec)

```

### 16. 正则表达式

mysql 正则表达式
在前面的章节我们已经了解到mysql可以通过 `like ...%` 来进行模糊匹配。

mysql 同样也支持其他正则表达式的匹配， mysql中使用 regexp 操作符来进行正则表达式匹配。

下表中的正则模式可应用于 `regexp` 操作符中。

模式| 描述
 :---: | :---
 `^`     | 匹配输入字符串的开始位置。如果设置了 regexp 对象的 multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。
`$`     | 匹配输入字符串的结束位置。如果设置了regexp 对象的 multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。
 `.`     | 匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式。
`[...]`   | 字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。
`[^...]`  | 负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。
`p1|p2`| 匹配 p1 或 p2。例如，`z|food` 能匹配 "z" 或 "food"。`(z | f)ood` 则匹配 "zood" 或 "food"。
`*`     | 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。
`+`     | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。
`{n}`    | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "bob" 中的 'o'，但是能匹配 "food" 中的两个 o。
`{n,m}`   | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。

**实例**
了解以上的正则需求后，我们就可以根据自己的需求来编写带有正则表达式的sql语句。以下我们将列出几个小实例(表名：person_tbl )来加深我们的理解：

- 查找name字段中以'st'为开头的所有数据：

```bash
mysql> select name from person_tbl where name regexp '^st';
```

- 查找name字段中以'ok'为结尾的所有数据：

```bash
mysql> select name from person_tbl where name regexp 'ok$';
```

- 查找name字段中包含'mar'字符串的所有数据：

```bash
mysql> select name from person_tbl where name regexp 'mar';
```

- 查找name字段中以元音字符开头或以'ok'字符串结尾的所有数据：

```bash
mysql> select name from person_tbl where name regexp '^[aeiou]|ok$';
```

### 17. NULL 值处理

为了处理这种情况，mysql提供了三大运算符:

- is null: 当列的值是 null,此运算符返回 true。
- is not null: 当列的值不为 null, 运算符返回 true。
- <=>: 比较操作符（不同于=运算符），当比较的的两个值为 null 时返回 true。

> 关于 null 的条件比较运算是比较特殊的。你不能使用 = null 或 != null 在列中查找 null 值 。

> 在 mysql 中，null 值与任何其它值的比较（即使是 null）永远返回 false，即 null = null 返回false 。

mysql 中处理 null 使用 is null 和 is not null 运算符。

**实例**
在数据库my_data中创建student表，并插入相应数据

```bash
mysql> create table student(ID int unsigned auto_increment primary key,name varchar(10) not null,age int unsigned) default charset=utf8;
Query OK, 0 rows affected (0.25 sec)

mysql> insert into student values(0, "李华",25);
Query OK, 1 row affected (0.04 sec)

mysql> insert into student values(0, "敏柔",null);
Query OK, 1 row affected (0.04 sec)

mysql> insert into student values(0, "赵强",null);
Query OK, 1 row affected (0.04 sec)

mysql> insert into student values(0, "罗晴",23);
Query OK, 1 row affected (0.03 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
+----+--------+------+
4 rows in set (0.00 sec)

```

以下实例中你可以看到 = 和 != 运算符是不起作用的：

```bash
mysql> select id,name,age=null from student;
+----+--------+----------+
| id | name   | age=null |
+----+--------+----------+
|  1 | 李华   |     NULL |
|  2 | 敏柔   |     NULL |
|  3 | 赵强   |     NULL |
|  4 | 罗晴   |     NULL |
+----+--------+----------+
4 rows in set (0.00 sec)

mysql> select id,name,age!=null from student;
+----+--------+-----------+
| id | name   | age!=null |
+----+--------+-----------+
|  1 | 李华   |      NULL |
|  2 | 敏柔   |      NULL |
|  3 | 赵强   |      NULL |
|  4 | 罗晴   |      NULL |
+----+--------+-----------+
4 rows in set (0.00 sec)
```

查找数据表中 age 列是否为 null，必须使用 <=> 、is null 和 is not null，如下实例：

```bash
mysql> select id,name,age is null from student;
+----+--------+-------------+
| id | name   | age is null |
+----+--------+-------------+
|  1 | 李华   |           0 |
|  2 | 敏柔   |           1 |
|  3 | 赵强   |           1 |
|  4 | 罗晴   |           0 |
+----+--------+-------------+
4 rows in set (0.00 sec)

mysql> select id,name,age<=>null from student;
+----+--------+------------+
| id | name   | age<=>null |
+----+--------+------------+
|  1 | 李华   |          0 |
|  2 | 敏柔   |          1 |
|  3 | 赵强   |          1 |
|  4 | 罗晴   |          0 |
+----+--------+------------+
4 rows in set (0.00 sec)

```

### 18.事务

mysql 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

- 在 mysql 中只有使用了 innodb 数据库引擎的数据库或表才支持事务。
- 事务处理可以用来维护数据库的完整性，保证成批的 sql 语句要么全部执行，要么全部不执行。
- 事务用来管理 insert,update,delete 语句

一般来说，事务是必须满足4个条件（acid）：：原子性（atomicity，或称不可分割性）、一致性（consistency）、隔离性（isolation，又称独立性）、持久性（durability）。

- **原子性**：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

- **一致性**：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

- **隔离性**：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable）。

- **持久性**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

> 在 mysql 命令行的默认设置下，事务都是自动提交的，即执行 sql 语句后就会马上执行 commit 操作。因此要显式地开启一个事务务须使用命令 begin 或 start transaction，或者执行命令 set autocommit=0，用来禁止使用当前会话的自动提交。

#### 事务控制语句：

- begin 或 start transaction 显式地开启一个事务；

- commit 也可以使用 commit work，不过二者是等价的。commit 会提交事务，并使已对数据库进行的所有修改成为永久性的；

- rollback 也可以使用 rollback work，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

- savepoint identifier，savepoint 允许在事务中创建一个保存点，一个事务中可以有多个 savepoint；

- release savepoint identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

- rollback to identifier 把事务回滚到标记点；

- set transaction 用来设置事务的隔离级别。innodb 存储引擎提供事务的隔离级别有read uncommitted、read committed、repeatable read 和 serializable。

#### mysql 事务处理主要有两种方法：

1、用 begin, rollback, commit来实现

- begin 开始一个事务
- rollback 事务回滚
- commit 事务确认

2、直接用 set 来改变 mysql 的自动提交模式:

- set autocommit=0 禁止自动提交
- set autocommit=1 开启自动提交

**实例**
下面具体演示MySQL事务的使用

```bash
mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
+----+--------+------+
4 rows in set (0.00 sec)

mysql> begin; # 开启事务
Query OK, 0 rows affected (0.00 sec)

mysql> insert into student values(0,"寒梅",28); # 插入数据
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
|  5 | 寒梅   |   28 |
+----+--------+------+
5 rows in set (0.00 sec)

mysql> rollback; # 回滚
Query OK, 0 rows affected (0.04 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
+----+--------+------+
4 rows in set (0.00 sec)

mysql> insert into student values(0,"寒梅",28); # 插入数据
Query OK, 1 row affected (0.15 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
|  6 | 寒梅   |   28 |
+----+--------+------+
5 rows in set (0.00 sec)

mysql> insert into student values(5,"李磊",26);
Query OK, 1 row affected (0.04 sec)

mysql> insert into student values(0,"张雷",20);
Query OK, 1 row affected (0.04 sec)

mysql> set autocommit=0; # 禁止自动提交
Query OK, 0 rows affected (0.00 sec)

mysql> insert into student values(0,"韩美美",20);
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+----+-----------+------+
| ID | name      | age  |
+----+-----------+------+
|  1 | 李华      |   25 |
|  2 | 敏柔      | NULL |
|  3 | 赵强      | NULL |
|  4 | 罗晴      |   23 |
|  5 | 李磊      |   26 |
|  6 | 寒梅      |   28 |
|  7 | 张雷      |   20 |
|  8 | 莉莉      |   20 |
|  9 | 韩美美    |   20 |
+----+-----------+------+
9 rows in set (0.00 sec)

mysql> rollback; # 回滚
Query OK, 0 rows affected (0.16 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   | NULL |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
|  5 | 李磊   |   26 |
|  6 | 寒梅   |   28 |
|  7 | 张雷   |   20 |
|  8 | 莉莉   |   20 |
+----+--------+------+
8 rows in set (0.00 sec)

mysql> update student set age=23 where name="敏柔"; # 修改数据
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   |   23 |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
|  5 | 李磊   |   26 |
|  6 | 寒梅   |   28 |
|  7 | 张雷   |   20 |
|  8 | 莉莉   |   20 |
+----+--------+------+
8 rows in set (0.00 sec)

mysql> savepoint point1; # 创建保存点
Query OK, 0 rows affected (0.00 sec)

mysql> update student set age=22 where name="赵强";
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   |   23 |
|  3 | 赵强   |   22 |
|  4 | 罗晴   |   23 |
|  5 | 李磊   |   26 |
|  6 | 寒梅   |   28 |
|  7 | 张雷   |   20 |
|  8 | 莉莉   |   20 |
+----+--------+------+
8 rows in set (0.00 sec)

mysql> rollback to point1; # 回滚到保存点
Query OK, 0 rows affected (0.00 sec)

mysql> select * from student;
+----+--------+------+
| ID | name   | age  |
+----+--------+------+
|  1 | 李华   |   25 |
|  2 | 敏柔   |   23 |
|  3 | 赵强   | NULL |
|  4 | 罗晴   |   23 |
|  5 | 李磊   |   26 |
|  6 | 寒梅   |   28 |
|  7 | 张雷   |   20 |
|  8 | 莉莉   |   20 |
+----+--------+------+
8 rows in set (0.00 sec)

mysql> commit; # 事务提交
Query OK, 0 rows affected (0.04 sec)

```

### 19. ALTER命令

当我们需要修改数据表名或者修改数据表字段时，就需要使用到MySQL alter命令。

#### 删除表字段

如下命令使用了 alter 命令及 drop 子句来删除以上创建表的 age 字段：

```bash
mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| ID    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | YES  |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

mysql> alter table table_name  drop field_name;
Query OK, 0 rows affected (0.92 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| ID    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)      | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)

```

如果数据表中只剩余一个字段则无法使用drop来删除字段。

#### 增加表字段

mysql 中使用 add 子句来向数据表中添加列，如下实例在表 student 中添加 age 字段，并定义数据类型:

```bash
mysql> alter table student add age int unsigned not null;
Query OK, 0 rows affected (0.53 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| ID    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

```

执行以上命令后，field_name 字段会自动添加到数据表字段的末尾。

如果你需要指定新增字段的位置，可以使用mysql提供的关键字 first (设定位第一列)， after 字段名（设定位于某个字段之后）。

尝试以下 alter table 语句, 在执行成功后，使用 show columns 查看表结构的变化：

```bash
mysql> desc student;
+-------+------------------+------+-----+---------+-------+
| Field | Type             | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+-------+
| name  | varchar(10)      | NO   |     | NULL    |       |
| age   | int(10) unsigned | NO   |     | NULL    |       |
+-------+------------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> alter table student add id int unsigned auto_increment primary key first;
Query OK, 0 rows affected (0.56 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

mysql> alter table student add class int unsigned after id;
Query OK, 0 rows affected (0.54 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class | int(10) unsigned | YES  |     | NULL    |                |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```

first 和 after 关键字可用于 add 与 modify 子句，所以如果你想重置数据表字段的位置就需要先使用 drop 删除字段然后使用 add 来添加字段并设置位置。

#### 修改表字段

如果需要修改字段类型及名称, 你可以在alter命令中使用 modify 或 change 子句 。

例如，把字段 class 的类型从 int unsigned 改为 varchar(10)，可以执行以下命令:

```bash
mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class | int(10) unsigned | YES  |     | NULL    |                |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

mysql> alter table student modify class varchar(10);
Query OK, 8 rows affected (0.69 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> desc student;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class | varchar(10)      | YES  |     | NULL    |                |
| name  | varchar(10)      | NO   |     | NULL    |                |
| age   | int(10) unsigned | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```

使用 change 子句, 语法有很大的不同。 在 change 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型。

当你修改字段时，你可以指定是否包含值或者是否设置默认值。

如果你不设置默认值，mysql会自动设置该字段默认为 null。

```bash
mysql> alter table student change class class_room varchar(20) default "203";
Query OK, 0 rows affected (0.10 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class_room | varchar(20)      | YES  |     | 203     |                |
| name       | varchar(10)      | NO   |     | NULL    |                |
| age        | int(10) unsigned | NO   |     | NULL    |                |
+------------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```

你可以使用 alter 来修改字段的默认值，尝试以下实例：

```bash
mysql> alter table student alter class_room set default "205";
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class_room | varchar(20)      | YES  |     | 205     |                |
| name       | varchar(10)      | NO   |     | NULL    |                |
| age        | int(10) unsigned | NO   |     | NULL    |                |
+------------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```

你也可以使用 alter 命令及 drop子句来删除字段的默认值，如下实例：

```bash
mysql> alter table student alter class_room drop default;
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc student;
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| class_room | varchar(20)      | YES  |     | NULL    |                |
| name       | varchar(10)      | NO   |     | NULL    |                |
| age        | int(10) unsigned | NO   |     | NULL    |                |
+------------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

#### 修改数据表

修改数据表类型，可以使用 alter table 命令来完成。

尝试以下实例，我们将表 student 的类型修改为 MyISAM ，然后再修改为 InnoDB：

```bash
mysql> alter table student engine=myisam;
Query OK, 8 rows affected (0.28 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> show table status like "student"\G
*************************** 1. row ***************************
           Name: student
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 8
 Avg_row_length: 20
    Data_length: 160
Max_data_length: 281474976710655
   Index_length: 2048
      Data_free: 0
 Auto_increment: 9
    Create_time: 2019-06-09 17:47:23
    Update_time: 2019-06-09 17:47:23
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

mysql> alter table students engine=innodb;
Query OK, 8 rows affected (0.76 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> show table status where name="students"\G
*************************** 1. row ***************************
           Name: students
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 8
 Avg_row_length: 2048
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 9
    Create_time: 2019-06-09 17:54:53
    Update_time: 2019-06-09 17:54:53
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

```

如果需要修改数据表的名称，可以在 alter table 语句中使用 rename 子句来实现。

尝试以下实例将数据表 student 重命名为 students：

```bash
mysql> alter table student rename to students;
Query OK, 0 rows affected (0.00 sec)

mysql> show tables;
+-------------------+
| Tables_in_my_data |
+-------------------+
| students          |
+-------------------+
1 row in set (0.00 sec)

```

### 20. 临时表

MySQL 临时表在我们需要保存一些临时数据时是非常有用的。临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。

如果你使用了其他MySQL客户端程序连接MySQL数据库服务器来创建临时表，那么只有在关闭客户端程序时才会销毁临时表，当然你也可以手动销毁。

#### 创建临时表

使用temporary关键字创建临时表

```bash
mysql> create temporary table class_room(room_id int unsigned primary key, class varchar(20) ) default charset=utf8;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into class_room values(205,"九年级一班");
Query OK, 1 row affected (0.00 sec)

mysql> select * from class_room;
+---------+-----------------+
| room_id | class           |
+---------+-----------------+
|     205 | 九年级一班      |
+---------+-----------------+
1 row in set (0.00 sec)

mysql> desc class_room;
+---------+------------------+------+-----+---------+-------+
| Field   | Type             | Null | Key | Default | Extra |
+---------+------------------+------+-----+---------+-------+
| room_id | int(10) unsigned | NO   | PRI | NULL    |       |
| class   | varchar(20)      | YES  |     | NULL    |       |
+---------+------------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> show tables;
+-------------------+
| Tables_in_my_data |
+-------------------+
| students          |
+-------------------+
1 row in set (0.00 sec)

```

当你使用 show tables命令显示数据表列表时，你将无法看到 class_room表。

如果你退出当前mysql会话，再使用 select命令来读取原先创建的临时表数据，那你会发现数据库中没有该表的存在，因为在你退出时该临时表已经被销毁了。

#### 删除临时表

默认情况下，当你断开与数据库的连接后，临时表就会自动被销毁。当然你也可以在当前MySQL会话使用 DROP TABLE 命令来手动删除临时表。

以下是手动删除临时表的实例：

```bash
mysql> drop table class_room;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from class_room;
ERROR 1146 (42S02): Table 'my_data.class_room' doesn't exist

```

### 21. 复制表

如果我们需要完全的复制MySQL的数据表，包括表的结构，索引，默认值等。 如果仅仅使用create table ... select 命令，是无法实现的。

**实例**
尝试以下实例来复制表 runoob_tbl 。

步骤一：

获取数据表的完整结构。

```bash
mysql> show create table students\G;
*************************** 1. row ***************************
       Table: students
Create Table: CREATE TABLE `students` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL,
  `class_room` varchar(20) DEFAULT '205',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

ERROR:
No query specified

```

步骤二：

修改SQL语句的数据表名，并执行SQL语句。

```bash
mysql> create table `student` (`id` int(10) unsigned not null auto_increment,`name` varchar(10) not null,`class_room` varchar(20) default '205', primary key (`id`)) engine=innodb auto_increment=9 default charset=utf8;
Query OK, 0 rows affected (0.59 sec)

```

步骤三：

执行完第二步骤后，你将在数据库中创建新的克隆表 student。 如果你想拷贝数据表的数据你可以使用 insert into... select 语句来实现。

```bash
mysql> insert into student(id,name,class_room) select * from students;
Query OK, 8 rows affected (0.00 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> select * from student;
+----+--------+------------+
| id | name   | class_room |
+----+--------+------------+
|  1 | 李华   | 205        |
|  2 | 敏柔   | 205        |
|  3 | 赵强   | 205        |
|  4 | 罗晴   | 205        |
|  5 | 李磊   | 205        |
|  6 | 寒梅   | 205        |
|  7 | 张雷   | 205        |
|  8 | 莉莉   | 205        |
+----+--------+------------+
8 rows in set (0.00 sec)

```

### 22. 处理重复数据

有些 MySQL 数据表中可能存在重复的记录，有些情况我们允许重复数据的存在，但有时候我们也需要删除这些重复的数据。

#### 防止表中出现重复数据

你可以在 MySQL 数据表中设置指定的字段为 PRIMARY KEY（主键） 或者 UNIQUE（唯一） 索引来保证数据的唯一性。

如果你想设置表中字段 id，name 数据不能重复，你可以设置双主键模式来设置数据的唯一性， 如果你设置了双主键，那么那个键的默认值不能为 NULL，可设置为 NOT NULL。如下所示：

```bash
mysql> create table teachers(id int unsigned auto_increment, name varchar(10), gender varchar(5), primary key(id,name));
Query OK, 0 rows affected (0.26 sec)

mysql> desc teachers;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name   | varchar(10)      | NO   | PRI | NULL    |                |
| gender | varchar(5)       | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

```

如果我们设置了唯一索引，那么在插入重复数据时，SQL 语句将无法执行成功,并抛出错。

insert ignore into 与 insert into 的区别就是 insert ignore 会忽略数据库中已经存在的数据，如果数据库没有数据，就插入新的数据，如果有数据的话就跳过这条数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的。

```bash
mysql> insert into teachers values(1, "李磊", "男");
Query OK, 1 row affected (0.00 sec)

mysql> insert into teachers values(1, "李磊", "男");
ERROR 1062 (23000): Duplicate entry '1-李磊' for key 'PRIMARY'

mysql> insert ignore into teachers values(1, "李磊", "男");
Query OK, 0 rows affected, 1 warning (0.00 sec)

```

INSERT IGNORE INTO 当插入数据时，在设置了记录的唯一性后，如果插入重复数据，将不返回错误，只以警告形式返回。 而 REPLACE INTO 如果存在 primary 或 unique 相同的记录，则先删除掉。再插入新记录。

```bash
mysql> replace into teachers values(1, "李磊", "女");
Query OK, 2 rows affected (0.00 sec)

mysql> select * from teachers;
+----+--------+--------+
| id | name   | gender |
+----+--------+--------+
|  1 | 李磊   | 女     |
+----+--------+--------+
1 row in set (0.00 sec)

```

另一种设置数据的唯一性方法是添加一个 UNIQUE 索引，如下所示：

```bash
mysql> create table person(first_name char(20) not null, last_name char(20) not null, gender char(10),unique (last_name, first_name)) charset=utf8;
Query OK, 0 rows affected (0.25 sec)

mysql> insert ignore into person values("李", "雷", "男");
Query OK, 1 row affected (0.00 sec)

mysql> insert ignore into person values("李", "雷", "男");
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> insert ignore into person values("李", "丽", "女");
Query OK, 1 row affected (0.00 sec)

mysql> select * from person;
+------------+-----------+--------+
| first_name | last_name | gender |
+------------+-----------+--------+
| 李         | 丽        | 女     |
| 李         | 雷        | 男     |
+------------+-----------+--------+
2 rows in set (0.00 sec)

```

#### 统计重复数据

以下我们将统计表中 first_name 和 last_name的重复记录数：

- 创建person_info表，并插入3条重复数据

```bash
mysql> create table person_info(id int unsigned primary key auto_increment,first_name char(20) not null, last_name char(20) not null, gender char(10)) charset=utf8;
Query OK, 0 rows affected (0.29 sec)

mysql> insert ignore into person_info values(0,"李", "丽", "女");
Query OK, 1 row affected (0.00 sec)

mysql> insert ignore into person_info values(0,"李", "丽", "女");
Query OK, 1 row affected (0.00 sec)

mysql> insert ignore into person_info values(0,"李", "丽", "女");
Query OK, 1 row affected (0.00 sec)

mysql> select * from person_info;
+----+------------+-----------+--------+
| id | first_name | last_name | gender |
+----+------------+-----------+--------+
|  1 | 李         | 丽        | 女     |
|  2 | 李         | 丽        | 女     |
|  3 | 李         | 丽        | 女     |
+----+------------+-----------+--------+
3 rows in set (0.00 sec)
```

- 查询重复数据

```bash
mysql> select count(*) as repetitions,last_name,first_name from person_info group by last_name, first_name having repetitions > 1;
+-------------+-----------+------------+
| repetitions | last_name | first_name |
+-------------+-----------+------------+
|           3 | 丽        | 李         |
+-------------+-----------+------------+
1 row in set (0.00 sec)

```

以上查询将返回 person_info 表中重复的记录数。 一般情况下，查询重复的值，请执行以下操作：

- 确定哪一列包含的值可能会重复。
- 在列选择列表使用count(*)列出的那些列。
- 在group by子句中列出的列。
- having子句设置重复数大于1。

#### 过滤重复数据

如果你需要读取不重复的数据可以在 select 语句中使用 distinct 关键字来过滤重复数据。

```bash
mysql> select distinct first_name,last_name from person_info;
+------------+-----------+
| first_name | last_name |
+------------+-----------+
| 李         | 丽        |
+------------+-----------+
1 row in set (0.00 sec)

```

你也可以使用 group by 来读取数据表中不重复的数据：

```bash
mysql> select first_name, last_name  from person_info group by last_name, first_name;
+------------+-----------+
| first_name | last_name |
+------------+-----------+
| 李         | 丽        |
+------------+-----------+
1 row in set (0.01 sec)

```

#### 删除重复数据

如果你想删除数据表中的重复数据，你可以使用以下的SQL语句：

##### 方法一

先创建临时表tab,新表tab中的数据时从person_info表中分组查询出来的

```bash
mysql> create table tmp select last_name, first_name, gender from person_info  group by last_name, first_name, gender;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0
```

在表的第一位添加主键

```bash
alter table tmp add column `id` bigint(20) primary key not null auto_increment comment 'id' first;
```

删除原表

```bash
mysql> drop table person_info;
Query OK, 0 rows affected (0.01 sec)
```

重命名为person_info

```bash
mysql> alter table tmp rename to person_info;
Query OK, 0 rows affected (0.01 sec)
```

