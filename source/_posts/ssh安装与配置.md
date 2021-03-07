---
title: SSH安装与配置
date: 2021-03-06 09:56:09
categories:
tags:
---

## ssh服务器

### 安装ssh-server

```bash
$ sudo apt-get install openssh-server vim -y
```

### 设置密码

由于`SSH`使用的就是账号的登录密码，所以不存在单独的`SSH`密码。如果你需要修改`SSH`远程登录密码，需要修改这台服务器的账号的登录密码。

```bash
$ passwd
```

### 修改配置文件

#### 配置ssh客户端

```bash
$ vi /etc/ssh/ssh_config
```

去掉`PasswordAuthentication yes`前面的`#`号，保存退出

```shell
Host *
#   ForwardAgent no
#   ForwardX11 no
#   ForwardX11Trusted yes
PasswordAuthentication yes  # 找不到，在命令行模式下，/模板字符串  搜索即可
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
```

#### 配置ssh服务器

```bash
$ vi /etc/ssh/sshd_config
```

把`PermitRootLogin prohibit-password`改成`PermitRootLogin yes`，保存退出

```shell
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

### 重启服务

```bash
/etc/init.d/ssh restart
```

查看服务是否开启

```bash
$ ps -e | grep ssh
```

## ssh客户端

### 安装ssh-client

```bash
$ sudo apt install ssh
```

### 登录服务器

#### 密码登录

```bash
ssh -p 22  root@192.168.1.9  
```

#### 密钥登录

##### 制作密钥对

首先在客户端执行如下命令, 制作密钥对。

```bash
[root@host ~]$ ssh-keygen # <== 建立密钥对
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): # <== 按 Enter
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): # <== 输入密钥锁码，或直接按 Enter 留空
Enter same passphrase again:  # <== 再输入一遍密钥锁码

Your identification has been saved in /root/.ssh/id_rsa. # <== 私钥
Your public key has been saved in /root/.ssh/id_rsa.pub. # <== 公钥

The key fingerprint is:
0f:d3:e7:1a:1c:bd:5c:03:f1:19:f1:22:df:9b:cc:08 root@host

# 密钥锁码在使用私钥时必须输入，这样就可以保护私钥不被盗用。当然，也可以留空，实现无密码登录。

# 现在，在 root 用户的家目录中生成了一个 .ssh 的隐藏目录，内含两个密钥文件。id_rsa 为私钥，id_rsa.pub 为公钥。
```

##### 上传公钥到服务器

###### 方法一(推荐)

直接将公钥内容添加到服务器的 授权密钥文件`~/.ssh/authorized_keys`中

```bash
$ ssh-copy-id  -p 22 -i ~/.ssh/id_rsa.pub root@192.168.1.9
```

执行命令了会要求输入远程机器的密码，输入密码即可。

注：`ssh-copy-id`默认端口是22，如果您的SSH端口不是22，也就是远程服务器端口修改成其他的了，那就要得加上 -p +端口。

###### 方法二

先在客户端使用`scp`命令将公钥文件上传至服务器

```bash
$ scp ~/.ssh/id_rsa.pub root@192.168.1.9:~/.ssh/
```

使用密码登录服务器

```bash
$ ssh -p 22  root@192.168.1.9  
```

在服务器将公钥文件添加到授权密钥文件`~/.ssh/authorized_keys`中

```bash
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

##### 登录服务器

```bash
$ ssh root@192.168.1.9
```

