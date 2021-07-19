---
title: vue-router
date: 2020-04-14 21:44:51
categories:
    - vue
tags:
    - Vue
    - router
---

Vue.js官方提供了一套专用的路由工具库 Vue Router,其使用和配置都非常简单。

## 安装路由

可以使用npm包的方式来安装路由

```bash
cnpm install vue-router --save
```

> 其实通过Vue-cli脚手架快速搭建项目时，命令行会询问是否安装路由功能，从而并不需要额外安装路由功能

通过Vue-cli脚手架搭建的项目，在`src/router`文件夹内部有`index.js`文件，打开index.js文件会看到以下引用。

```javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)
```

在入口文件`src/main.js`中导入`src/router/index.js`文件，即可使用路由配置的信息

```javascript
import router from './router'
//等效于
//import router from './router/index.js'
```

## 建立路由模块

先建立一个路由器模块,主要用来配置和绑定相关信息。

在`src/router/index.js`文件中使用`new Router`命令创建一个路由,一个路由是一个对象。

一条路由的实现需要三部分:`name` 、`path`和`component`。`name`是命名,`path`是路径,`component`是组件

```javascript
// src/router/index.js

import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
// 把对应的组件或页面引入进来
import Login from '@/pages/login' // @指src根路径

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/', // 这里默认是跳转到HelloWorld 组件， 后期会改为项目首页
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path:'/login',
      name: 'Login',
      component: Login  // 后续如果还有页面要跳转，就按照这样的格式继续书写
    }
  ]
})
```

> 导入`HelloWorld`组件的时候，实际上导入的是`src/components/HelloWorld.vue`.因为在`Webpack`中配置了，所以可以省略vue后缀。`*.vue`文件是Vue的单文件组件格式，可以同时包括模板定义、样式定义和组件模块定义。

## 启动路由器

1. 在`main.js`入口文件中启用该路由器。
   
    `main.js`作为入口文件，使用`import`可以把所有要用到的包都导入到这里，然后在Vue项目中去运用它们。
   
    创建和挂载根实例，通过`router`配置参数注入路由，从而让整个应用都有路由功能
   
   ```javascript
    // src/main.js
   
    import Vue from 'vue'
    import App from './App'
    import router from './router'
   
    new Vue({
    el: '#app',
    router,
    components: { App },
    template: '<App/>'
    })
   ```
   
   > 浅析 导包方式
   
    这个不是vue的规定而是node加载模块的方式,当require('./router')(import会被转为require),node是这样的寻找目标的:
   
   1. 首先寻找目录下有没有router.js或者router.node,如果有就导入
   
   2. 如果没有看是否有router目录,如果没有就require失败,抛出异常"Cannot find module './router'"
   
   3. 如果有router目录会在其下寻找package.json文件,如果有则按照package的配置来导入
   
   4. 如果没有package.json,看是否有index.js或者index.node,如果有就导入没有就失败
      
       详细见node文档：[https://nodejs.org/api/modules.html#modules_folders_as_module](https://nodejs.org/api/modules.html#modules_folders_as_module)

2. 配置之后，路由匹配到的组件将会渲染到`App.vue`的`＜router-view></router-view>`
   
   ```javascript
    // src/App.vue
   
    <template>
    <div id="app">
        <router-view/>
    </div>
    </template>
   ```

3. 最后App.vue 会渲染到`index.html`
   
   ```html
    <body>
    <div id=”app”></div>
    </body>
   ```
   
    这样就会把渲染出来的页面挂载到id为app的div里了

## 路由重定向

1. 项目应用通常都会有一个首页，默认首先打开的是首页，要跳转到哪个页面都可以设置路由路径发生跳转。重定向也是通过`routes`配置来完成的
   
   ```javascript
    routes: [
        {
        path: '/index',
        name: 'index',
        component: HelloWorld
        },
   
        // 从"/"重定向到"/index"
        {
        path:"/",
        redirect:'/index'
        },
   
        // 重定向的目标也可以是一个命名的路由
        {
        path:"/p1",
        redirect: { name:"index" }
        },
   
        // 重定向的目标甚至可以是一个方法，动态返回重定向目标
        {
        path:"/p2",
        redirect:to=>{
            // 方法接收 "目标路由" 作为参数
            // return 重定向的 "字符串路径／路径对象"
   
            // return '/index'
            return {name: 'index'}
        }
        }
    ]
   ```

## `<router-link>`

1. `<router-link＞`组件支持用户在具有路由功能的应用中单击导航。
   
    通过`to`属性可以指定目标地址,默认渲染成带有正确链接的`＜a＞`标签,通过配置`tag`属性可以生成别的标签。
   
    另外,当目标路由成功激活时,链接元素会自动设置一个表示激活的`css`类名。
   
   ```html
    <!-- 直接给to指定属性值, 不提倡使用，尽量使用属性绑定 -->
    <router-link to='/goods'>商品</router-link>
   
    <!-- 使用v-bind 绑定 JS表达式-->
    <router-link :to="'/goods'">商品</router-link>
   
    <!-- 绑定data中的变量path -->
    <router-link :to='path'>商品</router-link>
   
    <!-- 绑定对象，自定义网址，携带查询参数 ==> /goods?id=1 -->
    <router-link :to="{'path':'/goods', , query:{'id':1}}}">商品</router-link>
   
    <!-- 绑定 对象，命名路由，携带params参数 ==> /goods/1 -->
    <router-link :to="{name:'goods', params:{'id':1}}">商品</router-link>
   ```

## 路由对象属性

> `route`和`router`区别:`vue-router`中经常会操作的两个对象`router`和`route`两个

1. `route`是一个跳转的路由对象，每一个路由都会有一个`route`对象，是一个局部的对象，可以获取对应的`name`,`path`,`params`,`query`等
   
    下面列出了常用的路由信息对象属性
   
   - `$route.path`: 字符串，对应当前路由的路径，总是解析为绝对路径,如`"/goods"`
   - `$route.params`: 一个`key/value`对象, 包含了动态片段和全匹配片段，如果没有路由参数，就为空对象。
   - `$route.query`: 一个`key/value`对象, 表示URL查询参数。例如: 对于路径`/goods?cate_id=1`,则有`$route.query.cate_id == 1`；如果没有查询参数,则为空对象
   - `$route.hash`: 当前路由的`hash`值(不带#)，如果没有`hash`值，则为空字符串。
   - `$route.ful1Path`: 完成解析后的URL，包含查询参数和hash的完整路径。
   - `$route.matched`: 一个数组，包含当前路由的所有嵌套路径片段的路由记录。路由记录就是`routes`配置数组中的对象副本(还有一些在children数组)。

2. `router`是`VueRouter`的一个对象，通过`Vue.use(VueRouter)`和`VueRouter`构造函数得到一个`router`的实例对象，这个对象中是一个全局的对象，他包含了所有的路由包含了许多关键的对象和属性。
   
    下面列出了常用的路由信息对象方法
   
   - `router.push(location)`: 要导航到不同的URL，则使用`router.push`方法。该方法会向`history`栈添加一个新的记录，当用户单击浏览器的后退按钮时，回到之前的URL。
     
     ```javascript
       router.push('/goods') // 字符串
       router.push({path: '/goods'}) // 对象
     
       // 自定义网址，携带查询参数
       router.push({ path: "/goods", query: { id: 1 } }) // -> /goods?id=1
     
       // 命名路由，携带params参数
       router.push({ name: "goods", params: { id: 1 } }) // -> /goods/1
     ```
     
       由一个页面跳转到另一个页面时，需要携带一些数据，这时就需要用到这种带参数的路由跳转方式了。
     
     > 当单击`＜router-link＞`时，会在内部调用`router.push(...)`方法，所以说单击`＜router-link:to＝"...">`等同于调用`router.push(...)`。
     
     ```javascript
       // 当跳转之前，两种方式可以携带params参数
       // <router-link :to="{name:'goods', params:{'id':1}}">商品</router-link>
       // router.push({ name: "goods", params: { id: 1 } })
     
       // 对应的路由配置有两种方式:
       // 1. 将参数获取直接体现在网址上
       routes: [
           {
           path: '/goods/:id',
           name: 'goods',
           component: goods
           }
       ]
       // 此时会导致之前的params参数拼接在网址上， 为"/goods/1"，刷新不丢失数据
     
       // 2. 不在网址上体现
       routes: [
           {
           path: '/goods',
           name: 'goods',
           component: goods
           }
       ]
       // 此时会导致之前的参数不能表现出来，网址仍为"/goods"，刷新之后，参数丢失
     
       // 但是不管怎么配置路由，参数获取方式相同
     ```
   
   - `router.replace(location)`: `router.replace`与`router.push`很像，唯一的不同是它不会向`history`栈添加新记录，而是跟它的方法名一样只替换掉当前的history 记录。`router.replace(...)`等价于`＜router-link :to＝"..." replace＞`
   
   - `router.go(n)`: 参数是一个整数，表示在`history`记录中向前进多少步或向后退多少步
     
     ```javascript
       // 在浏览器记录中前进一步，等同于history.forward()
       router.go(1)
       // 后退一步，等同于history.back()
       router.go(-1)
       // 前进三步
       router.go(3)
       // 如果history记录不够，就会失败
       router.go(-100)
       router.go(100)
     ```
