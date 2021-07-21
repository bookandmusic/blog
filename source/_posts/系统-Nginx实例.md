---
title: Nginx实例
date: 2021-07-21 09:11:42
categories:
    - 系统
    - Linux
tags:
    - 反向代理
    - 负载均衡
    - 子路径部署服务
---

## 部署静态网站

>   以下以`hexo`静态博客为例，演示部署流程

-   将生成的静态资源文件夹放到 `/var/www/`中，起名为`blog`

-   打开 `nginx`配置文件，添加 `server` 模块

    ```json
    server {
        listen 80;
        server_name localhost:80;
    
        location / {
            alias /usr/local/var/www/blog/;
            index index.html;
        }
    }
    ```

-   `listen`指的是 后期可以在哪个端口中，访问博客，`http`协议默认端口是 80，hexo博客部署的也是顶级路径，也就是 如果服务器域名是 `http://www.bookandmusic.cn/`, hexo服务的主页路径也是 `http://www.bookandmusic.cn/`

## 部署动态网站

>   以下以 `django`项目为例，演示 部署流程

-   确定 django项目中的静态资源路径以及 `supervisor` 启动的django服务的地址

-   打开 `nginx`配置文件，添加 `server` 模块

    ```json
    server {
        listen 443;
        server_name localhost:7003;
        # 静态文件配置
        location /static/ {
            alias /Users/lsf/PycharmProjects/DjangoBlog/collectedstatic/;
            expires max;
            access_log        off;
            log_not_found    off;
        }
        location /media/ {
            # 静态文件配置
            alias /Users/lsf/PycharmProjects/DjangoBlog/uploads/;
            expires max;
        }
        location ~ \.py$ {
            return 403;
        }
        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_set_header X-NginX-Proxy true;
            proxy_redirect off;
            if (!-f $request_filename) {
                proxy_pass http://127.0.0.1:8000;
                break;
            }
        }
    }
    ```

-   `https`协议默认的端口是443，此时默认部署的django项目的路径为 顶级路径，也就是 如果服务器域名是 `https://www.bookandmusic.cn/`, django服务的主页路径也是 `https://www.bookandmusic.cn/`

>   部署 `https://`协议对应的网站，需要指明 `ssl`证书和解密私钥

## 反向代理

### 静态网站

现在有三个不同的服务：

-   `http://127.0.0.1:7000/`: 博客服务

-   `http://127.0.0.1:7001/`: django文档服务

-   `http://127.0.0.1:7002/`: flask文档服务

此时，想要在同样的域名 `http://www.bookandmusic.cn/`下，通过不同的子路径来体现不同的服务

```json
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:7000/;
    }

    location /drf/ {
        proxy_pass http://127.0.0.1:7001/;
    }

    location /flask/ {
        proxy_pass http://127.0.0.1:7002/;
    }
}
```

### 动态网站

>   如果想要将 django动态服务，部署到 子路径`/django/`下

-   首先需要修改`django`项目的 主路由，在 所有的 路由地址前面，添加 `/django/`路由前缀
-   其次，修改 静态资源的 `url`前缀，也就是 `MEDIA_URL` 和 `STATIC_URL`，添加 `/django/`前缀
-   最后，才是 修改 `nginx`配置，添加前缀

```json
server {
    listen 7003;
    server_name localhost:7003;
    location /django/static/ {
        alias /Users/lsf/PycharmProjects/DjangoBlog/collectedstatic/;
        expires max;
        access_log        off;
        log_not_found    off;
    }
    location /django/media/ {
        # 静态文件配置
        alias /Users/lsf/PycharmProjects/DjangoBlog/uploads/;
        expires max;
    }
    location ~ \.py$ {
        return 403;
    }
    location /django/ {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
        if (!-f $request_filename) {
            proxy_pass http://127.0.0.1:8000;
            break;
        }
    }
}


server {
    listen 80;
    server_name localhost;

   	
    location /django/ {
        proxy_pass http://127.0.0.1:7003/;
    }
}
```

-   此时，就可以通过 `https://www.bookandmusic.cn/django/` 访问项目

## 负载均衡

django项目，现在部署了多个服务

-   `http://127.0.0.1:7000/`

-   `http://127.0.0.1:7001/`

-   `http://127.0.0.1:7002/`

此时，为了分担压力，在访问 `https://www.bookandmusic.cn/`时，需要 分发到不同的服务中，也就是所谓的负载均衡

```json
upstream django {
    # 此时配置服务时，不需要要 http或https
    server 127.0.0.1:7000;
    server 127.0.0.1:7001;
    server 127.0.0.1:7002;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://django/;
    }
}
```

