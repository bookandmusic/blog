---
title: Vue事件绑定以及事件修饰符
date: 2020-05-16 12:50:50
categories:
    - vue
tags:
    - vue
    - v-on
    - Js事件
    - 事件修饰符
    - 事件绑定
---
## 事件

要理解事件绑定，就得先了解事件。

浏览器是事件驱动型的，根据用户的行为触发不同的事件，根据事件执行相应的操作。我们较为熟悉的事件有三大类型：

### 鼠标键盘事件

鼠标键盘事件|事件介绍
:---|:--
onclick | 鼠标点击某个对象
ondbclick| 鼠标双击某个对象
onmousedown | 某个鼠标按键被按下
onmouseup | 某个鼠标按键被松开
onmousemove | 鼠标被移动
onmouseover | 鼠标被移到某元素之上
onmouseout | 鼠标从某元素移开
onkeypress | 某个键盘的键被按下或按住
onkeydown | 某个键盘的键被按下
onkeyup | 某个键盘的键被松开

### 页面事件

页面事件 | 事件介绍
:---|:--
onload | 某个页面或图像被完成加载
onunload | 用户退出页面
onresize | 窗口或框架被调整尺寸	
onerror | 当加载文档或图像时发生某个错误
onabort | 图像加载被中断


### 表单相关事件

表单相关事件 | 事件介绍
:---|:---
onblur | 元素失去焦点
onfocus | 元素获得焦点
onchange | 用户改变域的内容
onreset | 重置按钮被点击
onsubmit | 提交按钮被点击
onselect | 文本被选定


> 需要注意的是事件处理程序中的变量`event`保留着事件对象的信息，包括比如`click`事件，事件属性里有点击位置相对于浏览器，以及页面的坐标信息，事件的类型（`click`）,触发事件的DOM节点信息等;可以将`evenet`作为参数传递，在函数内部获取具体的evenet对象信息。


## 事件绑定

- 在Vue.js中`v-on`指令用来监听`DOM`事件，并在触发事件时运行一些`JavaScript`代码;当然`v-on`也可以简写为`@`

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
        <meta charset="UTF-8">
        <title>事件绑定</title>
        <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
    </head>

    <body>

        <div id='app'>
            <button v-on:click='func'> 显示 </button>
            <h1> {{msg}} </h1>
        </div>

        <script>
            var vm = new Vue({
                el: '#app',
                data() {
                    return {
                        msg: '',
                    }
                },
                methods: {
                    func() {
                        this.msg = '这是一个大大的H1'
                    }
                },
            })
        </script>
    </body>

    </html>
    ```

## 事件修饰符

> `Vue.js` 为 `v-on` 提供了事件修饰符来处理 `DOM` 事件细节;`Vue.js`通过由点(`.`)表示的指令后缀来调用修饰符。

### 事件修饰符分类

Vue.js提供的事件修饰符主要针对两类情况:

- 冒泡机制修饰符
    - `.stop`
    - `.capture`
    - `.self`

- 事件本身修饰符
    - `.prevent`
    - `.once`

### 冒泡机制修饰符

#### 事件冒泡

`DOM`中，树状结构决定了子元素肯定在父元素里，所以点击子元素，就同时点击了子元素和父元素，以及父元素的父元素，以此类推，当然最终的根节点都是文档，以及`window`。

试想，当一个子元素被点击的时候，不仅仅这个元素本身被点击了，因为这个元素也在其上一级父元素中(属于父级元素的地盘)，所以相当于其父元素也被点击了，以此类推，一层一层往外推，最终整个文档也是被点击了，如果每个层级的节点元素都绑定了`click`事件，那么每个节点的`click`事件函数都会被执行。举个形象的例子，一个村里的人被打了（`click`），首先就要按照村里的规矩处理，同时这个村属于某个乡镇，当然也是相当于这个乡镇的人被打了，那么也要按照这个乡镇的规矩处理，以此一层一层往上报。这个例子不准确的地方就是，现实中一个人因为一个事件只会被处理一次，不会因为同一件事情多次处理。

#### 冒泡带来的烦恼

当上层（以及上上层，直至`body`元素）父级有子元素同样的方法，但你子元素的事件后，所有父级元素的同名函数也会从下到上，由里往外，挨个执行，但是大多数情况下，我们只希望子当事元素事件执行，不希望层层执行，这就要想办法阻止这种冒泡的情况发生。比如我们点击`Child Span`的时候只显示 `Child Span`的内容。结合刚刚的例子就是，村里发生了打人事件，在村里解决了，就没必要一层一层往上报，在层层处理了。

#### 事件修饰符

在Vue.js中针对Js事件本身的冒泡机制提供一些事件修饰符以便使用

- `.stop`: 阻止冒泡事件

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
        <meta charset="UTF-8">
        <title>冒泡机制</title>
        <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
        <style>
            #outer {
                width: 400px;
                height: 200px;
                background-color: aqua;
                position: absolute;
            }

            #inner {
                width: 200px;
                height: 100px;
                background-color: darkcyan;
                margin: 50px auto;
            }

            #btn {
                width: 40px;
                height: 30px;
                background-color: gold;
                margin: 35px 80px;
            }
        </style>
    </head>

    <body>
        <div id='app'>
            <div id="outer" @click='outer'>
                <div id="inner" @click='inner'>
                    <button id="btn" @click.stop='btn'>按钮 </button>
                </div>
            </div>
        </div>
        <script>
            var vm = new Vue({
                el: '#app',
                methods: {
                    inner() {
                        console.log("inner触发了")
                    },
                    outer() {
                        console.log("outer触发了")
                    },
                    btn() {
                        console.log("btn触发了")
                    }
                },
            })
        </script>
    </body>

    </html>
    ```

    > 当点击按钮时，只会触发`button`本身的`click`事件,不会继续传递

- `.self`:  只当事件在该元素本身（而不是子元素）触发时触发回调函数

    ```html
    <div id="outer" @click='outer'>
        <div id="inner" @click.self='inner'>
            <button id="btn" @click='btn'>按钮 </button>
        </div>
    </div>
    ```

    > 当点击按钮时, 继续触发冒泡机制，因此`outter`盒子同样会触发对应的回调函数,但是`inner`盒子并不会触发回调函数，只有点击`inner`本身时，才会正常执行回调函数

- `.capture`: 添加事件侦听器时使用事件捕获模式,即拥有该事件修饰符的元素会优先触发对应事件

    ```html
    <div id="outer" @click.capture='outer'>
        <div id="inner" @click='inner'>
            <button id="btn" @click='btn'>按钮 </button>
        </div>
    </div>
    ```

    > 当点击按钮时, 继续触发冒泡机制，但是会优先触发`outer`的回调函数,其次，按照正常的冒泡顺序，由内向外

### 事件本身修饰符

- `.prevent`:阻止默认行为

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
        <meta charset="UTF-8">
        <title>VueDemo</title>
        <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
        <style>
            form {
                width: 210px;
                height: 300px;

                position: relative;
                margin: 0 auto;
            }

            .user {
                width: 200px;
                height: 20px;
                display: block;
                margin: 5px auto;
            }

            #btn {
                width: 70px;
                float: right;
            }
        </style>
    </head>

    <body>

        <div id='app'>

            <form action="/userinfo" method="POST" v-on:submit.prevent='onsubmit'>
                <input type="text" name="user" class="user" v-model='user.name' placeholder="请输入用户名。。。">
                <input type="password" name="user" class="user" v-model='user.pwd' placeholder="请输入密码。。。">

                <input type="submit" id="btn" value="提交">
            </form>

        </div>

        <script>
            var vm = new Vue({
                el: '#app',
                data() {
                    return {
                        user: {
                            name: "",
                            pwd: ""
                        }
                    }
                },
                methods: {
                    onsubmit() {
                        console.log(`表单信息:${this.user.name}和${this.user.pwd}提交`)
                    }
                },
            })
        </script>
    </body>

    </html>
    ```

    > 表单本身的提交行为会进行页面跳转，现在使用`.prevent`修饰符之后，只执行绑定的方法，不跳转页面;类似的还有超链接等的默认行为都可以使用该修饰符阻止

- `.once`: 事件只能触发一次

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
        <meta charset="UTF-8">
        <title>VueDemo</title>
        <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
    </head>

    <body>
        <div id='app'>
            <button @click.once='btn'> 你只能评论一次 </button>
        </div>
        <script>
            var vm = new Vue({
                el: '#app',
                methods: {
                    btn() {
                        alert("已评论")
                    }
                },
            })
        </script>
    </body>

    </html>
    ```
