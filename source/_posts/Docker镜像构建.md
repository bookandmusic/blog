---
title: Docker镜像构建
date: 2021-03-02 20:24:18
categories:
tags:
---

## 构建Elasticsearch Docker镜像

创建文件夹  es

下载 [elasticsearch-analysis-ik-5.6.16.zip](https://github.91chifun.workers.dev//https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.16/elasticsearch-analysis-ik-5.6.16.zip)

解压到当前文件夹，并将文件夹重命名为`elasticsearch-analysis-ik`

创建 DockerFile

```dockerfile
FROM elasticsearch:5.6.16
ADD elasticsearch-analysis-ik /usr/bin/elasticsearch/plugins/elasticsearch-analysis-ik
```

创建镜像

```zsh
docker build -f Dockerfile -t bookandmusic/elasticsearch-ik:5.6.16 .
```

查看镜像

```zsh
docker images
```

创建容器

```zsh
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name elasticsearch -e ES_JAVA_OPTS="-Xms64m -Xmx512m" 
-v ~/elasticsearch/data:/usr/share/elasticsearch/data 
-v ~/elasticsearch/plugins:/usr/share/elasticsearch/plugins 
-d bookandmusic/elasticsearch-ik:5.6.16

# docker run --name elasticsearch 创建一个es容器并起一个名字；
# -p 9200:9200 将主机的9200端口映射到docker容器的9200端口，用来给es发送http请求
# -p 9300:9300 9300是es在分布式集群状态下节点之间的通信端口  \ 换行符
# -e 指定一个参数，当前es以单节点模式运行
# *注意，ES_JAVA_OPTS非常重要，指定开发时es运行时的最小和最大内存占用为64M和512M，否则就会占用全部可用内存
# -v 挂载命令，将主机中的路径和docker中的路径进行关联
# -d 后台启动服务
```

>   注意： 创建容器时，一定要指定内存，否则，则直接闪退