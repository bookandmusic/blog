---
title: JWS的python实现方案
date: 2021-06-10 23:00:11
categories:
  - python
tags:
  - JWT
  - JWS
  - itsdangerous
  - pyjwt
---

## JWS

在python中的 jws实现方案有以下两种：

### `pyjwt`

>   很多框架的 token认证方案，都是对`pyjwt`的进一步封装， 如：DRF的 `restframework-jwt`、Flask的 `flask-jwt-extend`

```python
import jwt
from datetime import datetime, timedelta

expiry = datetime.utcnow() + timedelta(seconds=60*60)
secret = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9'

payload = {
    'exp': datetime.utcnow() + timedelta(seconds=30),   # 过期时间
    'uid': 1,
    'username': 'admin'
}

# 加密, 得到 token字符串
token = jwt.encode(payload, secret, algorithm='HS256')
print(token)
# head部分.payload载荷部分.签名部分

# 解密，校验签名
payload = jwt.decode(encoded_jwt, secret,  algorithms=['HS256'])
print(payload)
# 如果 token 被修改过后，则会抛出 DecodeError: Signature verification failed
# 如果 token 过期后，则会抛出 ExpiredSignatureError: Signature has expired
```

生成的token：

```json
b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MjMzMzk0MjQsInVpZCI6MSwidXNlcm5hbWUiOiJhZG1pbiJ9.c__sgvA9VOv54T1R5Nm8Z4scFBMEgHo46xVF3PxtaRw'
```

进行base64解码：

```python
In [1]: import base64
    
In [2]: base64.decodebytes(b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9')
Out[2]: b'{"typ":"JWT","alg":"HS256"}'
    
In [3]: base64.decodebytes(b'eyJleHAiOjE2MjMzMzk0MjQsInVpZCI6MSwidXNlcm5hbWUiOiJhZG1pbiJ9')
Out[3]: b'{"exp":1623337821,"uid":1,"username":"admin"}' 
```

###  `itsdangerous`

>   `ItsDangerous`默认使用`HMAC`和`SHA-512`进行签名, 生成**临时身份令牌**

有时您只想将一些数据发送到不受信任的环境。但是如何安全地做到这一点？诀窍就是签名。只要知道一个密钥，您就可以对数据进行加密签名并将其移交给其他人。当您取回数据时，可以轻松确保没有人篡改数据。使用`itsdangerous`可以实现此种方案。

例如： 想通过邮件让用户注册激活的时候地址当中带有用户的信息。但是信息一般都是敏感信息，而且还想让它具有时效性，所以就可以选择`itsdangerous`模块

```python
from itsdangerous import TimedJSONWebSignatureSerializer
from itsdangerous import SignatureExpired, BadSignature, BadData

expiry = 60
secret = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9'
payload = {
    'uid': 1,
    'username': 'admin'
}

# 创建序列化器对象
serializer = TimedJSONWebSignatureSerializer(secret, expires_in=expiry)

# 加密
token = serializer.dumps(payload)
print(token)
# head部分.payload载荷部分.签名部分

# 解密，校验签名
payload = serializer.loads(token)
print(payload )
# 如果 token 被修改过后，则会抛出 BadSignature: Signature b'xxx' does not match
# 如果 token 过期后，则会抛出 SignatureExpired: Signature expired
```

生成的token：

```json
b'eyJhbGciOiJIUzUxMiIsImlhdCI6MTYyMzMzODcwNiwiZXhwIjoxNjIzMzM4NzY2fQ.eyJ1aWQiOjEsInVzZXJuYW1lIjoiYWRtaW4ifQ._cpzkgKX-u7w8r57BGbwMHEg-EkHdJKjC74x_gPvrSGkPq5mrjmAXWFMSit0l_RGV-DzUJA09kEL6uX-ReCL0g'
```

进行base64解码：

```python
In [1]: import base64
    
In [2]: base64.decodebytes(b'eyJhbGciOiJIUzUxMiIsImlhdCI6MTYyMzMzODcwNiwiZXhwIjoxNjIzMzM4NzY2fQ==')
Out[2]: b'{"alg":"HS512","iat":1623338706,"exp":1623338766}'
    
In [3]: base64.decodebytes(b'eyJ1aWQiOjEsInVzZXJuYW1lIjoiYWRtaW4ifQ==')
Out[3]: b'{"uid":1,"username":"admin"}' 
```

## Base64

>   对字符串进行base64 解码时，有时候会出现 `Incorrect padding`:

因为输入的base64编码字符串必须符合base64的padding规则：

-   当原数据长度不是3的整数倍时, 如果最后剩下两个输入数据，在编码结果后加1个“=”；

-   如果最后剩下一个输入数据，编码结果后加2个“=”；

-   如果没有剩下任何数据，就什么都不要加，这样才可以保证资料还原的正确性。

因此，在对上面的 `itsdangerous` 生成的token进行base64 解码时，需要补上对应的 `=`

