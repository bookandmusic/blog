---
title: Git操作
categories:
	- 工具
	- Git
date: 2019-03-31 12:15:27
tags:
	- Git
	- 密钥
	- 远程仓库关联
---

Git是目前世界上最先进的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。也就是用来管理你的hexo博客文章，上传到GitHub的工具。Git非常强大，开发人员必备技能。

## 1. 安装Git

- windows：到git官网上下载, [Download](https://gitforwindows.org/) git,下载后会有一个Git Bash的命令行工具，以后就用这个工具来使用git。

- linux：对linux来说实在是太简单了，因为最早的git就是在linux上编写的，只需要一行代码

  ```shell
  sudo apt-get install git
  ```

  安装好后，用`git --version` 来查看一下版本

## 2. 基本操作

![Git操作](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2005/Git操作%20.jpg)


## 3. 远程仓库密钥配置

### 1. GitHub密钥配置

#### 1.1. 设置身份信息

git设置身份信息

```shell
git config --global user.name "yourname"

git config --global user.email "your@email.com"
```

#### 1.2. 删除.ssh文件夹

（直接搜索该文件夹）下的known_hosts(手动删除即可，不需要git）

#### 1.3. 创建密钥

终端输入命令

```shell
ssh-keygen -t rsa -C "your@email.com"（请填你设置的邮箱地址）
```

接着出现：

```shell
Generating public/private rsa key pair.

Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):
```

请直接按下回车

然后系统会自动在.ssh文件夹下生成两个文件，id_rsa和id_rsa.pub，用记事本打id_rsa.pub

将全部的内容复制

#### 1.4. 添加公钥

打开[https://github.com/](https://github.com/)，登陆你的账户，进入设置

进入ssh设置

点击 `New SSH key`

在key中将刚刚复制的粘贴进去

最后点击`Add SSH key`

#### 1.5. 测试

在终端输入

```shell
ssh -T git@github.com
```

你将会看到：

```shell
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```

输入 yes

```shell
Hi humingx! You've successfully authenticated, but GitHub does not provide shell access.
```

如果看到Hi后面是你的用户名，就说明成功了。

### 2. Gitee仓库密钥配置类似

> 重装系统之后,git push的时候会出现问题:`The authenticity of host 'github.com (13.229.188.59)' can't be established.`，原因是本地仓库和远程的SSH不匹配
>
> 解决办法：重新配置仓库密钥即可。

## 4. 本地仓库关联远程仓库

1. 已有仓库

    ```bash
    cd existing_git_repo
    git remote add origin https://gitee.com/bookandmusic/test.git
    ```
    添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
    ```bash
    git push -u origin master
    ```
    把本地库的内容推送到远程，用git push命令，实际上是把当前分支source推送到远程。

    由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

2. 新建git仓库，并关联远程仓库

    ```bash
    mkdir test
    cd test
    git init
    touch README.md
    git add README.md
    git commit -m "first commit"
    git remote add origin https://gitee.com/bookandmusic/test.git
    git push -u origin master
    ```

3. 用git进行push操作的时候，报`fatal: TaskCanceledException encountered.`的解决方法
    解决方法如下：
    ```shell
    git config --global credential.helper manager
    ```
    之后再push一切正常
