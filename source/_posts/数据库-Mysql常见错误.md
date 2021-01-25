---
title: Mysql常见错误
date: 2020-04-21 22:27:42
categories: 
    - 技术
    - 数据库
    - mysql
tags:
    - 错误说明
---

简单对常见错误进行总结、归纳。

## Ubuntu系统

### 1698 只能root用户登录Mysql

#### 一、问题描述

如果mysql安装时，没有提示输入密码，则会随机分配密码，直接mysql -u root -p则无法登录，报错：

```shell
ERROR 1698 (28000): Access denied for user 'root'@'localhost
```

#### 二、解决方案

1. 用管理员权限进入数据库
  
   ```shell
    sudo mysql -uroot -p
   ```

2. 修改加密方式和密码
  
   ```sql
    alter user 'username'@'host' identified with mysql_native_password BY 'password';
   ```

3. 刷新
  
   ```sql
    flush privileges;
   ```

4. 最后重启终端，就可通过 `mysql -u root -p` 免sudo登录mysql啦！

## Windows系统

### 1042服务启动异常

#### 一、问题描述

mysql已经发展到了8.0阶段，但是很多人在下载了安装了mysql8.0后，在快接近完成的阶段下出现了异常：

```bash
error 1042：Unable to connect to any of the specified MySQL hosts
```

上述异常直接导致mysql无法正常Finish，如图所示：

![Mysql-1042](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020/08/Mysql-1042.png)

#### 二、解决方案

1. windows端使用`Win+R` --> 运行 `"services.msc"` --> 打开service服务管理器，找到刚才安装mysql的服务名称

2. 右键 --> 属性 --> 登录，更改成"本地系统账户" --> 确定
  
    ![Mysql登录身份修改](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020/08/Mysql登录身份修改.png)

3. 回到安装程序，在次点击Execute，会发现已经可以成功到Finish界面

### 1045用户密码错误

#### 一、问题描述

在数据库安装之后，使用`mysql -uroot -p`链接数据库时，出现以下异常:

```bash
1045    Access denied for user 'root'@'localhost' (using password:YES)
```

这个意思是说：用户“root”@本地主机的访问被拒绝

那为什么会出现这种错误呢？

答案是这样：这种问题的本质是用户密码出现错误。

#### 二、解决方案

1. 打开命令窗口cmd，停止MySQL服务，输入命令：
  
   ```bash
    net stop mysql
   ```

2. 开启跳过密码验证登录的MySQL服务,输入命令
  
   ```bash
    mysqld --console --skip-grant-tables --shared-memory
   ```

3. 再打开一个新的cmd，无密码登录MySQL，输入登录命令
  
   ```bash
    mysql -u root -p
   ```

4. 重置用户名对应的密码，命令如下：
  
   ```sql
    use mysql;
   
    update user set authentication_string='' where user='root'; --修改密码为空
   
    flush privileges; --刷新权限
   ```

5. 退出mysql

6. 关闭以`--console --skip-grant-tables --shared-memory` 启动的MySQL服务，

7. 打开命令框，启动MySQL服务。输入
  
   ```bash
    net start mysql
   ```

8. 再次登录无密码登录：
  
   ```bash
    mysql -u root -p
   ```

9. 正确修改root密码
  
   ```sql
   alter user 'root'@'host' identified with mysql_native_password BY 'mysql';
   
   flush privileges;
   ```

10. 退出，再次成功登录，到此，重置密码结束。

### Mysql数据库初始化

当Mysql数据库链接失败， 跳过用户名验证也失败， 需要先删除 安装目录下的`data`文件夹, 然后重新初始化,生成初始化密码

1. 以管理员的身份打开cmd窗口跳转路径到`X:\xxx\mysql-8.0.11-winx64\bin`
  
   ```bash
    mysqld --initialize --user=mysql --console
   ```

2. 按照上面的流程，就可以跳过用户名验证，重新设置mysql密码




