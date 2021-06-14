---
title: Linux网络传输命令
date: 2021-06-13 10:11:52
categories:
	- 工具
	- Linux
tags:
	- ftp
	- scp
	- sftp
	- ssh
	- openssh

---

## 名词解释

### `FTP`

#### 介绍

`FTP`（File Transfer Protocol）: 是 TCP/IP 网络上两台计算机传送文件的协议，FTP 是在 TCP/IP 网络和 INTERNET 上最早使用的协议之一，它属于网络协议组的应用层。

FTP 客户机可以给服务器发出命令来下载文件，上载文件，创建或改变服务器上的目录。相比于 HTTP，FTP 协议要复杂得多。复杂的原因，是因为 FTP 协议要用到两个 TCP 连接：

-   一个是命令链路，用来在 FTP 客户端与服务器之间传递命令；
-   另一个是数据链路，用来上传或下载数据。

FTP 是基于 TCP 协议的，因此 iptables 防火墙设置中只需要放开指定端口（21 + **PASV** 端口范围）的 TCP 协议即可。

#### 工作模式

**PORT**（主动）方式的连接过程是：

-   客户端向服务器的 FTP 端口（默认是 21）发送连接请求，服务器接受连接，建立一条命令链路。
-   当需要传送数据时，客户端在命令链路上用 PORT 命令告诉服务器：“我打开了一个 1024 + 的随机端口，你过来连接我”。
-   于是服务器从 20 端口向客户端的 1024 + 随机端口发送连接请求，建立一条数据链路来传送数据。

**PASV**（Passive 被动）方式的连接过程是：

-   客户端向服务器的 FTP 端口（默认是 21）发送连接请求，服务器接受连接，建立一条命令链路。
-   当需要传送数据时，服务器在命令链路上用 PASV 命令告诉客户端：“我打开了一个 1024 + 的随机端口，你过来连接我”。
-   于是客户端向服务器的指定端口发送连接请求，建立一条数据链路来传送数据。

PORT 方式，服务器会主动连接客户端的指定端口，那么如果客户端通过代理服务器链接到 internet 上的网络的话，服务器端可能会连接不到客户端本机指定的端口，或者被客户端、代理服务器防火墙阻塞了连接，导致连接失败

PASV 方式，服务器端防火墙除了要放开 21 端口外，还要放开 PASV 配置指定的端口范围

### `SFTP`

`SFTP`（Secure File Transfer Protocol）：安全文件传送协议。

可以为传输文件提供一种安全的加密方法。sftp 与 ftp 有着几乎一样的语法和功能。

SFTP 为 SSH 的一部份，是一种传输文件到服务器的安全方式。

在 SSH 软件包中，已经包含了一个叫作 SFTP的安全文件传输子系统，SFTP 本身没有单独的守护进程，它必须使用 sshd 守护进程（端口号默认是 22）来完成相应的连接操作，所以从某种意义上来说，SFTP 并不像一个服务器程序，而更像是一个客户端程序。

SFTP 同样是使用加密传输认证信息和传输的数据，所以，使用 SFTP 是非常安全的。

但是，由于这种传输方式使用了加密 / 解密技术，所以传输效率比普通的 FTP 要低得多，如果您对网络安全性要求更高时，可以使用 SFTP 代替 FTP。

### `SCP`

`SCP`（Secure Copy）：scp 就是 `secure copy`，是用来进行远程文件复制的，并且整个复制过程是加密的。数据传输使用 ssh，并且和使用和 ssh 相同的认证方式，提供相同的安全保证。



**FTP & SCP/SFTP** 比较：

-    sftp/scp 传输协议是采用加密方式来传输数据的。

-    ftp 一般来说允许明文传输，当然现在也有带 SSL 的加密 ftp，有些服务器软件也可以设置成 “只允许加密连接”，但是毕竟不是默认设置需要我们手工调整，而且很多用户都会忽略这个设置。

-   普通 ftp 仅使用端口 21 作为命令传输，由服务器和客户端协商另外一个随机端口来进行数据传送。在 pasv 模式下，服务器端需要侦听另一个端口。假如服务器在路由器或者防火墙后面，端口映射会比较麻烦，因为无法提前知道数据端口编号，无法映射。

-   当你的网络中还有一些 unix 系统的机器时，在它们上面自带了 scp/sftp 等客户端，不用再安装其它软件来实现传输目的。
-   scp/sftp 属于开源协议，我们可以免费使用不像 FTP 那样使用上存在安全或版权问题。
-   所有 scp/sftp 传输软件（服务器端和客户端）均免费并开源，方便我们开发各种扩展插件和应用组件。

>   **小提示：**当然在提供安全传输的前提下 sftp 还是存在一些不足的，例如: 他的帐号访问权限是严格遵照系统用户实现的，只有将该帐户添加为操作系统某用户才能够保证其可以正常登录 sftp 服务器

### `SSH`

`SSH`（Secure Shell）：由 IETF 的网络工作小组（Network Working Group）所制定；SSH 为建立在应用层和传输层基础上的安全协议。

SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。

SSH 是由客户端和服务端的软件组成的：

-   服务端是一个守护进程 (daemon)，他在后台运行并响应来自客户端的连接请求。

    服务端一般是 sshd 进程，提供了对远程连接的处理，一般包括公共密钥认证、密钥交换、对称密钥加密和非安全连接； 

-   客户端包含 ssh 程序以及像 scp（远程拷贝）、slogin（远程登陆）、sftp（安全文件传输）等其他的应用程序。从客户端来看，SSH 提供两种级别的安全验证：
    -   第一种级别（基于口令的安全验证）
    -    第二种级别（基于密匙的安全验证）

SSH 主要有三部分组成： 传输层协议 [`SSH-TRANS`] ；用户认证协议 [`SSH-USERAUTH`] ；连接协议 [`SSH-CONNECT`]。

### `OpenSSH`

`OpenSSH`: 是 SSH（Secure SHell）协议的免费开源实现。

SSH 协议族可以用来进行远程控制，或在计算机之间传送文件。

而实现此功能的传统方式，如 telnet(终端仿真协议)、 rcp ftp、 rlogin、rsh 都是极为不安全的，并且会使用明文传送密码。

OpenSSH 提供了服务端后台程序和客户端工具，用来加密远程控件和文件传输过程的中的数据，并由此来代替原来的类似服务。 

OpenSSH 是使用 SSH 透过计算机网络加密通讯的实现。它是取代由 `SSH Communications Security` 所提供的商用版本的开放源代码方案。

目前 OpenSSH 是 OpenBSD 的子计划。OpenSSH 常常被误认以为与 OpenSSL 有关联，但实际上这两个计划的有不同的目的，不同的发展团队，名称相近只是因为两者有同样的软件发展目标──提供开放源代码的加密通讯软件。

## 操作命令

### `ssh`

#### 环境配置

>   此处环境以 `Ubuntu18.04`为例

-   修改镜像源为中科大镜像

```python
sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

-   升级镜像源

```zsh
apt update
```

-   安装 `openssh-server`、 `vim`

```zsh
apt install openssh-server vim -y
```

#### 密码验证登录

-   打开`ssh`客户端配置文件`/etc/ssh/ssh_config`，修改 `ssh` 客户端配置

```zsh
# PasswordAuthentication yes
PasswordAuthentication yes  # 允许客户端使用密码登录
```

-   打开`ssh`服务器配置文件`/etc/ssh/sshd_config`，修改 `ssh` 服务器配置

```zsh
# PermitRootLogin prohibit-password
PermitRootLogin yes # 允许以root用户登录
```

-   重启ssh服务

```zsh
/etc/init.d/ssh start
```

>   在vi的命令模式下 `/关键词`即可实现搜索效果

配置当前`root`用户密码

```zsh
$ passwd
Enter new UNIX password:  # 输入新密码
Retype new UNIX password:  # 输入确认密码
passwd: password updated successfully  # 密码设置成功
```

#### 服务连接

>   在上一步中，服务器端已经配置过`ssh`服务端，现在就可以在本地电脑上，使用 `ssh`客户端链接远程的服务器

```zsh
$ ssh root@127.0.0.1 -p 2222  # ssh服务的端口号默认是22，如果不是，就需要使用 -p 指明
```

命令执行完成后，会提示输入密码，注意： `密码是远程服务器的密码，也就是上一步最后设置的密码`

#### 密钥验证登录

>   此时每次链接远程服务都需要输入密码，比较麻烦；可以通过公钥验证身份

-   打开`ssh`服务器配置文件`/etc/ssh/sshd_config`，修改 `ssh` 服务器配置

```zsh
PubkeyAuthentication yes  # 打开该配置，允许使用密钥验证
AuthorizedKeysFile   .ssh/authorized_keys # 指明公钥位置，多个公钥，可以在文件末尾追加
```

-   重启服务

```
/etc/init.d/ssh start
```

-   将本地的公钥文件上传到服务器，并移动到指定位置，并重命名

### `scp`

>   本地文件复制到远程服务器, **当远程ssh服务的端口不是22，通过 `-P` 指明**

```zsh
# scp -P 端口 本地文件地址  用户名@远程服务器地址:/文件路径
# 注意：远程服务器的文件路径必须存在
scp -P 2222 ~/.ssh/id_rsa.pub root@127.0.0.1:/root/.ssh/authorized_keys
```

将本地公钥 复制到远程服务器上， 此时在复制时，还需要 输入root用户的密码

复制成功，之后的所有操作 就可以通过密钥实现验证，不需要再输入密码

>   将远程服务器文件复制到本地

```zsh
# scp -P 端口 用户名@远程服务器地址:/文件路径 本地路径 
scp -P 2222 root@127.0.0.1:/root/.ssh/authorized_keys  ~/Documents
```

### `sftp`

链接远程服务器

```zsh
# sftp user@ip, 当远程ssh服务的端口不是22，通过 -P 指明
sftp -P 2222 root@127.0.0.1 
```

执行命令，进入交互环境

```zsh
sftp> 
```

以下命令都是在交互环境下执行

-   查看帮助信息

```zsh
sftp> help  # 执行help, 可以看看sftp支持哪些命令。
```

-   查看路径

```zsh
sftp> pwd  # 查看远程服务器的当前路径
Remote working directory: /root

sftp> lpwd  # 查看本地的当前路径
Local working directory: /Users/lsf/Documents
```

-   查看目录下的文件信息

```zsh
sftp> ls  # 查看远程服务器当前目录下的文件列表
authorized_keys  id_rsa.pub
sftp> lls  # 查看本地电脑当前目录下的文件列表
$RECYCLE.BIN	GitNote 	gitbook		hexo
```

-   下载文件

```zsh
# 下载远程服务器的文件 到本地
sftp> get id_rsa.pub /Users/lsf/Documents
Fetching /root/.ssh/id_rsa.pub to /Users/lsf/Documents/id_rsa.pub
/root/.ssh/id_rsa.pub                                                                                        100%  575   213.3KB/s   00:00


# 下载远程服务器的 .ssh 文件夹 到 本地目录， 需要参数 -r
sftp> get -r  .ssh/ /Users/lsf/Documents  
Fetching /root/.ssh/ to /Users/lsf/Documents/.ssh
Retrieving /root/.ssh
/root/.ssh/authorized_keys                                                                                   100%  575   262.3KB/s   00:00
/root/.ssh/id_rsa.pub
```

-   上传文件

```zsh
# 上传 本地文件到 远程服务器的当前目录
sftp> put random.html .
Uploading random.html to /root/.ssh/./random.html
random.html

# 上传 本地文件夹到 远程服务器的当前目录，需要参数 -r 
sftp> put -r GitNote  .
Uploading GitNote/ to /root/.ssh/./GitNote
Entering GitNote/
Entering GitNote/Note
GitNote/Note/Shadowsocks配置.md                                                                            100% 1107     1.0MB/s   00:00
GitNote/Note/Docker镜像配置.md                                                                           100%   84    66.4KB/s   00:00
```

-   `command`, 此时命令都是在 远程服务器执行

```zsh
sftp> rm id_rsa.pub
Removing /root/.ssh/id_rsa.pub
sftp> mkdir upload
sftp> ls
GitNote          authorized_keys  random.html      upload
```

-   `!command`,  此时命令是在本地电脑上执行

```zsh
sftp> !ls
$RECYCLE.BIN	GitNote		authorized_keys	desktop.ini	gitbook
sftp> !rm authorized_keys
sftp> !ls
$RECYCLE.BIN	GitNote		desktop.ini	gitbook
sftp> lls
$RECYCLE.BIN	GitNote		desktop.ini	gitbook
```

-   退出命令： `exit`、`bye`、`quit`

