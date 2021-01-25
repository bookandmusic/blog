---
title: GitBook教程
categories:
  - 教程
tags:
  - GitBook
date: 2019-03-31 11:58:35
---

> GitBook 是一个基于`Node.js`的命令行工具，可以使用`Markdown`来制作电子书，并利用`Git/Github`发布。

支持输出`静态站点`、`PDF`、`eBook`、`HTML网页`等格式。

安装 GitBook 需要 Node 环境，具体怎么安装 Node 这里就不多说了。

## 安装 GitBook

```bash
npm install -g gitbook-cli
# OR
yarn global add gitbook-cli
```

检查是否安装成功

```bash
gitbook -V
```

## 导出电子书

打开到 gitbook 的目录下

### 1、输出静态网页

```bash
$ gitbook serve .
Press CTRL+C to quit ...

Starting build ...
Successfuly built !

Starting server ...
Serving book on http://localhost:4000   
```

这时候就可以打开 [http://localhost:4000](http://localhost:4000/)：进行预览

同时在项目的目录中多了一个 `_book` 的文件夹，其中的文件就是生成的静态网页的内容。

### 2、导出 PDF

在项目的目录中执行

```bash
gitbook pdf .
```

项目目录下就会生成 `book.pdf`

### 3、导出 epub

在项目目录中执行

```bash
gitbook epub .  
```

项目目录下就会生成 `book.epub`

## 解决静态网页不能跳转问题

- 在导出的文件夹目录下找到gitbook->theme.js文件

- 找到下面的代码搜索`if(m)for(n.handler&&`

- 将if(m)改成if(false)
  
  ```js
  if(false)for(n.handler&&(i=n,n=i.handler,o=i.selector),o&&de.find.matchesSelector(Ye,o),n.guid||(n.guid=de.guid++),(u=m.events)||(u=m.events={}),(a=m.handle)||(a=m.handle=function(t){return"undefined"!=typeof de&&de.event.triggered!==t.type?de.event.dispatch.apply(e,arguments):void 0}),t=(t||"").match(qe)||[""],l=t.length;l--;)s=Ze.exec(t[l])||[],h=g=s[1],d=(s[2]||"").split(".").sort(),h&&(f=de.event.special[h]||{},h=(o?f.delegateType:f.bindType)||h,f=de.event.special[h]||{},c=de.extend({type:h,origType:g,data:r,handler:n,guid:n.guid,selector:o,needsContext:o&&de.expr.match.needsContext.test(o),namespace:d.join(".")},i),(p=u[h])||(p=u[h]=[],p.delegateCount=0,f.setup&&f.setup.call(e,r,d,a)!==!1||e.addEventListener&&e.addEventListener(h,a)),f.add&&(f.add.call(e,c),c.handler.guid||(c.handler.guid=n.guid)),o?p.splice(p.delegateCount++,0,c):p.push(c),de.event.global[h]=!0)}
  ```
