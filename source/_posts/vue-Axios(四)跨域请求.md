---
title: Axios(四):跨域请求
date: 2020-03-03 17:55:30
categories:
    - 技术
    - vue
tags:
    - vue
    - axios
    - 跨域
---
在前后端开发过程中一直会伴随我们的问题就是跨域问题，因为这时候前端和后端的代码是在不同机器上运行的，两个地址不在一个域名下，这个时候前端脚本在进行axios访问的时候浏览器就会报跨域相关的错误。

原因：浏览器的同源策略不允许跨域访问，所谓同源策略是指协议、域名、端口相同。

## 1. 设置代理服务器,实现跨域请求转发

方案：采用proxyTable解决。

1. proxyTable是什么？

    vue-cli提供的解决vue开发环境下跨域问题的方法，proxyTable的底层使用了[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware),它是http代理中间件，它依赖node.js，基本原理是用服务端代理解决浏览器跨域

    实现的过程就是在我们前端的本地起一个服务，然后我们前端的所有ajax访问首选访问我们本地的服务，本地的服务不会对来的请求做加工处理，只是将请求转发到我们真实的后台服务上去。我们本地的服务其实你就是一个中转站。这种解决方案就是利用**后端之间访问是不存在跨域的问题**。

2. 具体流程

    1. 首先在`config/index.js`里面找到`proxyTable:{}`,然后在里面加入

        ```javascript
        "/api":{
            target: 'https://www.runoob.com',
            changeOrigin: true,
            pathRewrite:{
                '^/api':''
            }
        }
        ```

        >注意:

        - `/api`: 是自定义的，写成什么都可以。

        - `target`: 设置要调用的接口域名和端口号。

        - `^/api`: 代替`target`里面的地址，后面组件中我们调接口时直接用`/api`代替

    2. 在组件中实现跨域请求

        - 比如要访问`'https://www.runoob.com/ajax/json_demo.json'`，直接写`/api/ajax/json_demo.json`即可。

            ```javascript
            import axios from 'axios'

            axios.get('/api/ajax/json_demo.json')
                .then(resp=>{
                    console.log(resp.data)
                })
                .cath(err=>{
                    console.log(err)
                })
            ```

        - 然而我们可以在`src/main.js`设置一个基础路径，这样你调用接口的时候可以不写`api`,直接写`/ajax/json_demo.json`即可。

            - 在`src/main.js`设置`axios.defaults.baseURL="/api"`;

                ```javascript
                import axios from 'axios'
                axios.defaults.baseURL="/api";
                ```

            - 在组件中直接调用接口即可。

                ```javascript
                import axios from 'axios'

                axios.get('/ajax/json_demo.json')
                    .then(resp=>{
                        console.log(resp.data)
                    })
                    .cath(err=>{
                        console.log(err)
                    })
                ```

## 2. 在后台django中配置跨域请求允许

但是今天我们这里不讲这种方式，有兴趣的可以在我的另一篇博客看到
