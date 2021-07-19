---
title: Axios(三):GET和POST提交数据
date: 2020-03-03 17:05:24
categories:
    - vue
tags:
    - vue
    - axios
    - 参数发送
---

## GET 请求传递参数

### 1、直接在 URL 上添加参数

```javascript
import axios from 'axios'

axios.get('/api/goods/add_info/?ID=12345&firstName=Fred&lastName=Flintstone')
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});
```

### 2、可以通过 params 设置参数

```javascript
import axios from 'axios'

axios.get('/api/goods/add_info/', {
    params: {
        ID: 12345,
        firstName: 'Fred',
        lastName: 'Flintstone',
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});
```

## POST 请求传递参数

### 1、Content-Type: application/json

```javascript
import axios from 'axios'

let data = {"code":"1234","name":"yyyy"};
axios.post(`${this.$url}/test/testRequest`,data)
.then(res=>{
    console.log('res=>',res);
})
```

### 2、Content-Type: multipart/form-data

```javascript
import axios from 'axios'
let data = new FormData();
data.append('code','1234');
data.append('name','yyyy');
axios.post(`${this.$url}/test/testRequest`,data)
.then(res=>{
    console.log('res=>',res);
})
```

### 3、Content-Type: application/x-www-form-urlencoded

```javascript
import axios from 'axios'
import qs from 'Qs'
let data = {"code":"1234","name":"yyyy"};
axios.post(`${this.$url}/test/testRequest`,qs.stringify({
    data
}))
.then(res=>{
    console.log('res=>',res);
})
```

总结：  
1、上面三种方式会对应后台的请求方式，这个也要注意，比如django POST参数获取
