---
title: Vue引入本地资源
date: 2020-10-23 22:28:48
categories:
    - vue
tags:
    - Vue
    - 图片
    - 样式
---

简单介绍一下在Vue项目中引入本地资源的实现方式: <!--more-->

## 引入本地图片

### 使用 `@`引入：

这是在组件内直接引用和普通的 html 方法一样，代码如下

```html
<img src="@/assets/test.png" alt="test.png">
```

### 使用 vue 的方法引入：

这是典型的 vue 思想，使用数据来操纵 dom； 首先在组件内使用 import ... from 引入

```js
import imgUrl from '../assets/test.png';
```

然后在 data 里面声明

```js
data: function () {
      return {
                  imgSrc: imgUrl
            }
       }
```

最后绑定数据

```js
<img :src="imgSrc" alt="imgSrc">
```

## 引入样式文件

在项目的 `src` 文件下，新建一个 `style` 文件夹，存放 `css` 文件。

### 全局引入

将外部的 css 文件放到 style 文件下，引入外部文件只需在 `main.js`文件中

```js
import './style/reset.css'
```

### 局部引入

```js
<style scoped>
  @import '../assets/iconfont/iconfont.css'; // 这个分号一定要写，要不会报错
</style>
```
