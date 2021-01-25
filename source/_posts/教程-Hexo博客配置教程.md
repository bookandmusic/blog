---
title: Hexo博客配置教程
categories:
  - 教程
tags:
  - Hexo
date: 2019-03-23 21:51:51
---
实现Hexo + GitHub 搭建博客。

## Hexo简介

Hexo是一款基于Node.js的静态博客框架，依赖少易于安装使用，可以方便的生成静态网页托管在GitHub和Coding上，是搭建博客的首选框架。大家可以进入hexo官网进行详细查看，因为Hexo的创建者是台湾人，对中文的支持很友好，可以选择中文进行查看。

> 注意: 查看[文档](https://hexo.io/docs/)以获取更多信息。如果使用Hexo时遇到任何问题，可以在[故障排除](https://hexo.io/docs/troubleshooting.html)中找到答案，或者可以在[GitHub](https://github.com/hexojs/hexo/issues)上询问。

## hexo教程

1. git和nodejs安装好后，就可以安装hexo了

  ```shell
  npm install hexo-cli -g
  npm install hexo --save
  ```
  依旧用hexo -v查看一下版本

  至此就全部安装完了。

2. 接下来初始化一下hexo
  ```shell
  hexo init
  ```

3. 创建一个新帖子

  ``` bash
  $ hexo new "我的新帖子"
  ```

  更多信息：[写作](https://hexo.io/docs/writing.html)

4. 运行服务器

  ``` bash
  $ hexo server
  ```

  更多信息：[服务器](https://hexo.io/docs/server.html)

5. 生成静态文件

  ``` bash
  $ hexo generate
  ```

  更多信息：[生成](https://hexo.io/docs/generating.html)

6. 部署到远程站点

  ``` bash
  $ hexo deploy
  ```

  更多信息：[部署](https://hexo.io/docs/deployment.html)

## Hexo中添加本地图片

1. 把主页配置文件`_config.yml` 里的`post_asset_folder:`这个选项设置为`true`

2. 在你的hexo目录下执行这样一句话`npm install hexo-asset-image --save`，这是下载安装一个可以上传本地图片的插件

3. 等待一小段时间后，再运行`hexo n "xxxx"`来生成md博文时，`/source/_posts`文件夹内除了`xxxx.md`文件还有一个同名的文件夹

4. 最后在`xxxx.md`中想引入图片时，先把图片复制到`xxxx`这个文件夹中，然后只需要在`xxxx.md`中按照markdown的格式引入图片：

   ```markdown
   ![你想输入的替代文字](xxxx/图片名.jpg)
   ```

   > `xxxx`是这个md文件的名字，也是同名文件夹的名字。只需要有文件夹名字即可，不需要有什么绝对路径。你想引入的图片就只需要放入`xxxx`这个文件夹内就好了，很像引用相对路径。
