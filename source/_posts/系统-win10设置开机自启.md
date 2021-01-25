---
title: win10设置开机自启
categories:
  - 系统
  - Windows
tags:
  - 开机自启
date: 2019-05-25 10:45:27
---
刚开始接触 win10 的朋友肯定不知道在哪里把自己常用的软件设置成开机启动，因为你根本找不到前面的 xp、win7、win8，等里面的启动文件夹。

## 工具 / 原料

* win10 系统电脑一台

## 方法 / 步骤

1. 如果想要实现应用程序在所有的用户登录系统后都能自动启动，就把该应用程序的快捷方式放到 “系统启动文件夹” 里；
    ![20190526103017-win10开始_01](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2005/20190526103017-win10开始_01%20.png)
  
2. 上面的方法有的朋友可能找不到路径，没有关系，你可以把上面的路径直接复制到地址栏里面打开即可。如下图
    ![20190526103044-win10开始_02](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2005/20190526103044-win10开始_02%20.png)

3. 同样也可以用系统命令来打开 “启动文件夹”。在运行里面输入：`shell:startup`
    ![20190526103052-win10开始_03](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2005/20190526103052-win10开始_03%20.png)

4. 或者输入：`%programdata%\Microsoft\Windows\Start Menu\Programs\Startup`
    ![20190526103102-win10开始_04](https://gitee.com/bookandmusic/imgs/raw/master/uPic/2020%2005/20190526103102-win10开始_04%20.png)

5. 上面那种命令都可以打开系统启动文件夹的；同样，打开之后把要启动的软件放进去即可。
