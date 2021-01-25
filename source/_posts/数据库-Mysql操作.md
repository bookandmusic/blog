---
title: Mysql操作
date: 2019-12-12 08:33:14
categories: 
    - 技术
    - 数据库
    - mysql
tags:
    - 用户管理
    - mysql软件操作
---
介绍如何在 Ubuntu系统上 安装mysql软件、以及如何创建用户信息。

### Mysql软件操作

以下所有操作以Ubuntu为例

#### mysql安装

```bash
sudo apt install mysql-server # 安装mysql
```

#### mysql删除

```bash
sudo apt remove --upgrade mysql-* -y # 卸载mysql

dpkg -l|grep ^rc|awk '{print$2}'|sudo xargs dpkg -P #清除配置
```

#### mysql服务

```bash
service mysql start # 启动服务
service mysql stop  # 停止服务
service mysql restart  # 重启服务
```

### mysql用户操作

#### 一、创建用户

```sql
create user 'username'@'host' identified with mysql_native_password BY 'password';
```

- `username`：你将创建的用户名
- `host`：指定该用户在哪个主机上可以登陆，如果是本地用户可用`localhost`，如果想让该用户可以从任意远程主机登陆，可以使用通配符`%`
- `password`：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
- `mysql_native_password`：用户加密方式，该方式为mysql8.0之前的加密方式，如果不写，则是mysql 8.0之后的加密方式`caching_sha2_password`

#### 二、 修改用户信息

```sql
alter user 'username'@'host' identified with mysql_native_password BY 'password';
```

- 参数同上

#### 三、删除用户

```sql
drop user 'username'@'host';
```

- 参数同上

#### 四、授权

```sql
grant privileges on databasename.tablename to 'username'@'host';
```

- privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
- databasename：数据库名
- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*
- 其余参数同上

#### 五、删除权限

```sql
revoke privileges on databasename.tablename to 'username'@'host';
```

- 参数同上

#### 六、演示

```sql
# 创建一个可以从任意远程主机登陆的用户test
create user 'test'@'%' identified with mysql_native_password BY '123456';

# 创建数据库mydata
create database mydata charset=utf8;

# 对用户test授权，只能操作数据库mydata
grant all on mydata.* to 'test'@'%';

# 刷新配置，立即启用修改
flush privileges;
```
