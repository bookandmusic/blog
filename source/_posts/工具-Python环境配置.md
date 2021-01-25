---
title: python环境配置
date: 2020-04-11 00:00:27
categories:
  - 工具
  - python
tags:
  - virtualenv
  - pipenv
  - conda
  - 虚拟环境
  - pip
---

本篇文章将介绍如何在本地搭建Python开发环境。

Python可应用于多平台包括 Linux 和 Mac OS X。

你可以通过终端窗口输入 "python" 命令来查看本地是否已经安装Python以及Python的安装版本。

## Python环境配置

```bash
# ubuntu
sudo apt-get install python3-pip
sudo apt-get install python-pip

# windows
python3 -m pip install --upgrade pip --force-reinstall
python2 -m pip install --upgrade pip --force-reinstall

# mac
brew install python  # 这一步安装了python3和pip3
brew install python@2 # 这一步安装了python2和pip2

# ipython2
pip install ipython 
# ipython3
pip3 install ipython 
```

## pip镜像源

### 镜像源

- 清华：`https://pypi.tuna.tsinghua.edu.cn/simple`
- 阿里云：`http://mirrors.aliyun.com/pypi/simple/`
- 中国科技大学 `https://pypi.mirrors.ustc.edu.cn/simple/`
- 华中理工大学：`http://pypi.hustunique.com/`
- 山东理工大学：`http://pypi.sdutlinux.org/`
- 豆瓣：`http://pypi.douban.com/simple/`

### 文件修改

#### Linux/Mac

修改 `~/.pip/pip.conf` (没有就创建一个文件夹及文件。文件夹要加`.`，表示是隐藏文件夹)

> 内容如下

```bash
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

#### windows

直接在 user 目录中创建一个 pip 目录，如：`C:\Users\xx\pip`，新建文件`pip.ini`。内容同上。

### 终端修改

```bash
pip config set global.index-url http://mirrors.aliyun.com/pypi/simple/ # 终端使用命令设置pip镜像
pip install pip -U  # 升级pip包管理工具
```

## 虚拟环境之pipenv

1. 安装`pipenv`

   ```bash
    pip install pipenv
   ```

2. 使用`pipenv`创建虚拟环境
  
   ```bash
    # 尽量在一个项目目录下创建虚拟环境
    pip install
   ```

3. 激活虚拟环境
  
   ```bash
    # 在创建虚拟环境的位置运行命令
    pipenv shell
   ```

4. 修改虚拟环境的镜像源
  
    打开虚拟环境下的配置文件: `Pipfile`
   
    ```bash
        [[source]]
        name = "pypi"
        url = "https://pypi.org/simple"
        verify_ssl = true
    
        [dev-packages]
    
        [packages]
    
        [requires]
        python_version = "3.7"
    ```
    修改url为国内镜像源:
   
   - 清华: `https://pypi.tuna.tsinghua.edu.cn/simple`
   
   - 阿里云: `http://mirrors.aliyun.com/pypi/simple/`
   
   - 中国科技大学: `https://pypi.mirrors.ustc.edu.cn/simple/`
   
   - 华中理工大学: `http://pypi.hustunique.com/`
   
   - 山东理工大学: `http://pypi.sdutlinux.org/`
   
   - 豆瓣: `http://pypi.douban.com/simple/`

5. 在虚拟环境安装第三方包
  
   ```bash
    pipenv install django
   ```

6. 使用pipenv卸载第三方模块
  
   ```bash
    pipev uninstall django
   ```

7. 查看依赖
  
   ```bash
    pipenv graph
   ```

8. 将安装的模块打包到一个文件内
  
   ```bash
    pip freeze > requirements.txt
   ```

9. 将一个文件内的第三方扩展安装到虚拟环境中
  
   ```bash
    pip install -r requirements.txt
   ```

10. 退出虚拟环境
  
    ```bash
    exit
    ```

11. 删除虚拟环境
  
    ```bash
    pipenv --rm
    ```

12. 不激活虚拟环境，直接运行命令
  
    ```bash
    pipenv run django-amdin start project djangodemo
    ```

## 虚拟环境之virtualenv

1. 安装`virtualenv`
  
   ```bash
    pip install virtualenv # 虚拟环境
    pip install virtualenvwrappern # mac/linux系统
    pip install virtualenvwrapper-win # windows系统
   ```

2. 创建虚拟环境
  
   ```bash
    mkvirtualenv django
   ```

3. 激活虚拟环境
  
   ```bash
    # 在创建虚拟环境后会默认激活
    workon django  # 激活django虚拟环境
    workon  # 查看所有虚拟环境
   ```

4. 修改虚拟环境的镜像源
  
   ```bash
   pip config set global.index-url http://mirrors.aliyun.com/pypi/simple/ # 终端使用命令设置pip镜像
   pip install pip -U  # 升级pip包管理工具
   ```

5. 在虚拟环境安装第三方包
  
   ```bash
    pip install django  # 先激活虚拟环境
   ```

6. 卸载第三方模块
  
   ```bash
    pip uninstall django # 先激活虚拟环境
   ```

7. 将安装的模块打包到一个文件内
  
   ```bash
    pip freeze > requirements.txt
   ```

8. 将一个文件内的第三方扩展安装到虚拟环境中
  
   ```bash
    pip install -r requirements.txt
   ```

9. 退出虚拟环境
  
    ```bash
    exit
    ```

10. 删除虚拟环境
  
    ```bash
    rmvirtualenv django
    ```

## 虚拟环境之conda

### 1. Anaconda简介

- Anaconda是一个方便的python包管理和环境管理软件，一般用来配置不同的项目环境。

- Anaconda通过管理工具包、开发环境、Python版本，大大简化了你的工作流程。不仅可以方便地安装、更新、卸载工具包，而且安装时能自动安装相应的依赖包，同时还能使用不同的虚拟环境隔离不同要求的项目。

- Anaconda 镜像使用帮助
  - Anaconda 是一个用于科学计算的 Python 发行版，支持 Linux, Mac, Windows, 包含了众多流行的科学计算、数据分析的 Python 包。

  - Anaconda 安装包可以到 [https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/) 下载。

- Miniconda 镜像使用帮助
  - Miniconda 是一个 Anaconda 的轻量级替代，默认只包含了 python 和 conda，但是可以通过 pip 和 conda 来安装所需要的包。

  - Miniconda 安装包可以到 [https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/) 下载。

### 2. Anaconda安装

- Windows安装

  - 点击安装文件`Anaconda3-2019.07-Windows-x86_64.exe`,选择安装路径,如`D:\python\Anaconda`,然后一直next即可。

  - **配置环境变量:** 将安装的根路径,如`D:\python\Anaconda`和`scripts`文件夹路径`D:\python\Anaconda\scripts`添加到电脑环境变量之中

- Linux/Mac安装

  - 将安装文件`Anaconda3-2019.07-Linux-x86_64.sh`移动到用户家目录

  - 在用户家目录,打开终端,执行`./Anaconda3-2019.07-Linux-x86_64.sh`,然后输入yes,一路回车即可。

  - **配置环境变量:**

    - 打开.bashrc 文件,在终端执行如下命令:

      ```bash
      vi ~/.bashrc
      ```

    - 输入G，跳转到文件末尾,在文件最后一行新增环境变量

      ```bash
      export PATH=~/anaconda3/bin:$PATH
      ```

    - 修改完成,先按esc键进入命令行模式，然后按`shift+：`进入末行模式，输入`wq`,保存退出

    - 在终端执行如下命令,使其立即生效

      ```bash
      source ~/.bashrc
      ```

    > 注意：在Mac中，修改文件`.bash_profile`,其余和Linux操作一样

### 3. 修改Anaconda镜像源

- Anaconda默认访问国外服务器，网速较慢，故需要修改默认镜像

- TUNA 还提供了 Anaconda 仓库与第三方源（conda-forge、msys2、pytorch等，查看完整列表）的镜像，各系统都可以通过修改用户目录下的 .condarc 文件:

  ```bash
  channels:
    - defaults
  show_channel_urls: true
  channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
  default_channels:
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
  custom_channels:
    conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  ```

  即可添加 Anaconda Python 免费仓库。Windows 用户无法直接创建名为 `.condarc` 的文件，可先执行 `conda config --set show_channel_urls yes` 生成该文件之后再修改。

### 4. Anconda基本使用

#### 管理环境

- 检查conda版本

  ```bash
  conda --version
  ```

- 升级当前版本conda

  ```bash
  conda update conda
  ```

- 管理（虚拟）环境

  ```bash
  # 创建一个名为python37的环境，指定Python版本是3.7（不用管是3.7.x，conda会为我们自动寻找3.7.x中的最新版本）
  conda create --name python37 python=3.7
  
  # 安装好后，使用activate激活某个环境
  activate python37 # for Windows
  source activate python37 # for Linux & Mac
  # 激活后，会发现terminal输入的地方多了python37的字样
  
  # 如果想返回默认的python环境，运行
  deactivate python37 # for Windows
  source deactivate python37 # for Linux & Mac
  
  # 删除一个已有的环境
  conda remove --name python37 --all
  
  # 另外，我们可以使用conda命令替换source命令用来激活和关闭环境
  conda activate python37
  conda deactivate
  
  # 取消每次打开终端，默认激活bash环境
  conda config --set auto_activate_base false
  
  # 重新激活每次打开终端，默认进入base环境
  conda config --set auto_activate_base true
  
  ```

  新的开发环境会被默认安装在你conda目录下的envs文件目录下。

  如果我们没有指定安装python的版本，conda会安装我们最初安装conda时所装的那个版本的python。

- 列出所有环境

  ```bash
  conda info -e
  conda info –-envs
  ```

  > 注意:conda会对当前活动的环境追加星号标记。

  ```bash
  macdeMacBook-Pro:~ mac$ conda info -e
  # conda environments:
  #
  base                  *  /Users/mac/anaconda3
  myenv                    /Users/mac/anaconda3/envs/myenv
  py3                      /Users/mac/anaconda3/envs/py3
  ```

- 复制一个环境
  通过克隆来复制一个环境。这儿将通过克隆py3来创建一个称为py32的副本。

  ```bash
  conda create -n py32 --clone py3
  ```

  通过

  ```bash
  conda info –-envs
  ```

  来检查环境。

- 重命名env

  conda是没有重命名环境的功能的, 要实现这个基本需求, 只能通过愚蠢的克隆-删除的过程。

  切记不要直接mv移动环境的文件夹来重命名, 会导致一系列无法想象的错误的发生!

  ```bash
  conda create --name newname --clone oldname      # 克隆环境
  conda remove --name oldname --all      # 彻底删除旧环境
  ```

- 分享环境

  如果你想把你当前的环境配置与别人分享，这样ta可以快速建立一个与你一模一样的环境（同一个版本的python及各种包）来共同开发/进行新的实验。

  **一个分享环境的快速方法就是给ta一个你的环境的.yml文件。**

  首先通过activate target_env要分享的环境target_env，然后输入下面的命令会在当前工作目录下生成一个environment.yml文件

  ```bash
  conda env export > environment.yml
  ```

  小伙伴拿到environment.yml文件后，将该文件放在工作目录下，可以通过以下命令从该文件创建环境

  ```bash
  conda env create -f environment.yml
  ```

#### 管理包

- conda安装和管理python包非常方便，可以在指定的python环境中安装包，且自动安装所需要的依赖包，避免了很多拓展包冲突兼容问题。

- **不建议使用easy_install安装包**。大部分包都可以使用conda安装，无法使用conda和anaconda.org安装的包可以通过pip命令安装

- 使用合适的源可以提升安装的速度

- 查看已安装包

  ```bash
  conda list
  ```

  使用这条命令来查看哪个版本的python或其他程序安装在了该环境中，或者确保某些包已经被安装了或被删除了。

- 向指定环境安装包
  我们在指定环境中安装requests包，有两种方式:

  - 直接通过-n选项指定安装环境的名字

    ```bash
    conda install --name py3 requests
    ```

    > 提示：你必须告诉conda你要安装环境的名字（-n py3）否则它将会被安装到当前环境中。

  - 激活py3环境，再使用conda install命令。

    ```bash
    conda activate py3
    conda install requests
    ```

- 通过pip命令

  对于那些无法通过conda安装或者从Anaconda.org获得的包，我们通常可以用pip命令来安装包。

  可以上pypi网站查询要安装的包，查好以后输入pip install命令就可以安装这个包了。

  我们激活想要放置程序的python环境，然后通过pip安装一个叫“PyMysql”的包。

  ```bash
  # Linux, OS X
  source activate bunnies
  
  # Windows
  activate py3
  # 安装
  pip install pymysql
  ```

  pip只是一个包管理器，所以它不能为你管理环境。pip甚至不能升级python，因为它不像conda一样把python当做包来处理。但是它可以安装一些conda安装不了的包。

  > 小技巧：在任何时候你可以通过在命令后边跟上-help来获得该命令的完整文档。很多跟在–后边常用的命令选项，可以被略写为一个短线加命令首字母。所以–name选项和-n的作用是一样的。通过conda -h或conda –-help来看大量的缩写。

#### 移除包、环境、或者conda

- 移除包

  假设你决定不再使用包pymysql。你可以在py3环境中移除它。

  ```bash
  conda remove -n py3 pymysql
  ```

- 移除环境

  我们不再需要snakes环境了，可以输入以下命令：

  ```bash
  conda remove -n myenv --all
  ```

- 删除conda

  - Linux/OS X：

    移除Anaconda 或 Miniconda 安装文件夹

    ```bash
    rm -rf ~/miniconda
    # OR
    rm -rf ~/anaconda
    ```

  - Windows：
    去控制面板，点击“添加或删除程序”，选择“Python2.7/3.6（Anaconda）”或“Python2.7/3.6（Miniconda）”并点击删除程序。