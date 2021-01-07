---
title: Deepin OS 安装 NVIDIA驱动
date: 2020-09-03 18:20:35
categories:
    - 系统
    - linux
tags:
    - deepin
    - nvidia
---

## 工具 / 原料

- 一台电脑
- Deepin15.10.2 系统
- N 卡官网驱动

<!--more-->

## 方法 / 步骤

### 一、驱动下载

本教程以 `GTX1050` 为例，前往 [N 卡官网](<https://www.nvidia.cn/geforce/drivers/>)下载对应的驱动 :

![20200826215347](https://gitee.com/liushaofeng2018/imgs/raw/master/uPic/2020/08/20200826215347.png)

然后选择一个下载即可，我下载的是`NVIDIA-Linux-x86_64-430.26.run`，最好不要下载最新的驱动，有可能有 Bug。(为可方便起见，建议更改文件名为 `001.run`，千万别忘了`. run`，该文件名只是为了好敲入命令。)

### 二、禁用 nouveau 驱动

1. 如果之前在 Deepin 中安装过 NVIDIA 驱动，请将其全部删除：

   在终端执行命令: `sudo apt autoremove  nvidia`(没有可以跳过)

2. 在终端执行命令:

   ```shell
   sudo dedit /etc/modprobe.d/blacklist.conf
   ```

   然后在将以下内容复制到文件中

   ```shell
   blacklist nouveau

   blacklist lbm-nouveau

   options nouveau modeset=0

   alias nouveau off

   alias lbm-nouveau off
   ```

	 保存退出

4. 接下来在终端执行命令:

    ```
    sudo update-initramfs -u
    ```

5. 重启系统，再次进入系统，可能会发现分  辨率异常。(分辨率异常就说明成功禁用 nouveau 驱动重启系统，重启后查看是否生效，

   ```shell
   lsmod |grep -i nouveau
   ```


### 三、NVIDIA 安装过程

1. 使用快捷键 `CTRL+ALT+F2` 进入终端，登录自己的账号 (就是用户名和密码)。
2. 暂时关闭图形界面：

    ```bash
    sudo service lightdm stop
    ```

3. 给下载好的 nvidia 驱动文件设置执行权限 (文件默认在 `/home/用户名/Downloads/` 目录下，用户名为你自己的用户名，如果你改文件名了，就填你改后的文件名，千万别填 `NVIDIA-Linux-x86_64-430.26.run`)。

    ```bash
    sudo chmod a+x  /home/用户名/Downloads/NVIDIA-Linux-x86_64-430.26.run
    ```

4. 驱动安装：

    ```bash
    sudo  sh  /home / 用户名 / Downloads/NVIDIA-Linux-x86_64-430.26.run
    ```

	(一系列 yes，还有一个界面选择 `install and cover`，意为安装和覆盖。)

5. 重启系统：

    ```bash
    sudo reboot
    ```

    安装完成后重启，执行 `nvidia-smi`

	![20190922111147368](https://gitee.com/liushaofeng2018/imgs/raw/master/uPic/2020/08/20190922111147368.png)
	
	发现这时候其实 NVIDIA 的显卡并没有工作，**显存一点都没占用**。主要是由于我的电脑是双显卡，这时候其实依然是 intel 集成显卡在工作，所以还要做下面的工作。

### 四、设置默认 nvidia 显卡工作

```shell
lspci | egrep 'VGA|3D'
```

执行上述命令获取 `nvidia` 显卡设备 `BusID`，例如: `01:00.0` 填写 `PCI:1:0:0`，

![20190922111306609](https://gitee.com/liushaofeng2018/imgs/raw/master/uPic/2020/08/20190922111306609.png)

 然后编辑 `/etc/X11/xorg.conf`，注意其中 PCI 部分填写 `PCI:1:0:0`，

```shell
Section "Module"
    Load "modesetting"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:X:X:X"       
    Option "AllowEmptyInitialConfiguration"
EndSection
```

编辑`~/.xinitrc`，

```shell
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
xrandr --dpi 96
```

编辑`/etc/lightdm/display_setup.sh`，

```
#!/bin/sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
xrandr --dpi 96
```

然后执行

```shell
sudo chmod +x  /etc/lightdm/display_setup.sh
```

编辑 `/etc/lightdm/lightdm.conf` 在 `[Seat:*]` 行下添加，

```shell
display-setup-script=/etc/lightdm/display_setup.sh
```

重启动后，查看是否生效，`nvidia-smi`

![20190922111512515](https://gitee.com/liushaofeng2018/imgs/raw/master/uPic/2020/08/20190922111512515.png)

发现已经生效。