---
title: Axios(二):API说明
date: 2020-03-03 13:30:05
categories:
    - 技术
    - vue
tags:
    - vue
    - axios
    - API
---

## Axios API

1. 可以通过向 axios 传递相关配置来创建请求

    - axios配置全部参数

        ```javascript
        axios(config)
        ```

        ```javascript
        // 发送 POST 请求
        axios({
            method: 'post',
            url: '/user/12345',
            data: {
            firstName: 'Fred',
            lastName: 'Flintstone'
            }
        });
        ```

    - axios中指定url和参数配置

        ```javascript
        axios(url[, config])
        ```

        ```javascript
        // 发送 GET 请求（默认的方法）
        axios('/user/12345');
        ```

        ```javascript
        // 发送 POST 请求
        axios('/user/12345', {
            method: 'post',
            data: {
            firstName: 'Fred',
            lastName: 'Flintstone'
            }
        });
        ```

2. 请求方法的别名

    - 为方便起见，为所有支持的请求方法提供了别名

        ```javascript
        axios.request(config)
        axios.get(url[, config])
        axios.delete(url[, config])
        axios.head(url[, config])
        axios.post(url[, data[, config]])
        axios.put(url[, data[, config]])
        axios.patch(url[, data[, config]])
        ```

        > NOTE:在使用别名方法时， url、method、data 这些属性都不必在配置中指定。

    - axios请求方法

        ```javascript
        // 发送 POST 请求
        axios.post('/user/12345',{
            firstName: 'Fred',
            lastName: 'Flintstone'
        });
        ```

3. 并发

    - 处理并发请求的助手函数

        ```javascript
        axios.all(iterable)
        axios.spread(callback)
        ```

    - 执行多个并发请求

        ```javascript
        function getUserAccount() {
            return axios.get('/user/12345');
        }

        function getUserPermissions() {
            return axios.get('/user/12345/permissions');
        }

        axios.all([getUserAccount(), getUserPermissions()])
        .then(axios.spread(function (acct, perms) {
            // 两个请求现在都执行完成
        }));
        ```
