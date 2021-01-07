---
title: Json模块
date: 2019-11-25 14:30:27

categories: 
- 技术
- python
- 模块
tags:
  - Json
  - Python
  - 字典
  - 列表
---

### JSON介绍
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。

#### JSON 语法规则
JSON是一个标记符的序列。
JSON是一个序列化的对象或数组。

#### JSON 与 JS 对象的关系
很多人搞不清楚 JSON 和 JS 对象的关系，甚至连谁是谁都不清楚。其实，可以这么理解：
**JSON 是 JS 对象的字符串表示法，它使用文本表示一个 JS 对象的信息，本质是一个字符串。**
如
```javascript
var obj = {a: 'Hello', b: 'World'}; //这是一个对象，注意键名也是可以使用引号包裹的

var json = '{"a": "Hello", "b": "World"}'; //这是一个 JSON 字符串，本质是一个字符串
```
#### JSON 和 JS 对象互转
要实现从JSON字符串转换为JS对象，使用 JSON.parse() 方法：
```javascript
var obj = JSON.parse('{"a": "Hello", "b": "World"}'); //结果是 {a: 'Hello', b: 'World'}
```
要实现从JS对象转换为JSON字符串，使用 JSON.stringify() 方法：
```javascript
var json = JSON.stringify({a: 'Hello', b: 'World'}); //结果是 '{"a": "Hello", "b": "World"}'
```

#### JSON 和 PYTHON 对象互转

要实现从JSON字符串转换为Python对象，使用 json.loads() 方法：
```python
import json

a_dict = json.loads('{"a": "Hello", "b": "World"}') //结果是 {'a': 'Hello', 'b': 'World'}
b_list = json.loads('[1, 2, 3, 4]') //结果是 [1, 2, 3, 4]
```
要实现从Python对象转换为JSON字符串，使用 json.dumps() 方法：
```python
import json

a_obj = json.dumps({a: 'Hello', b: 'World'}) //结果是 '{"a": "Hello", "b": "World"}'
b_obj = json.dumps([1, 2, 3, {"a":7}]) //结果是 '[1, 2, 3, {"a": 7}]'
```

要实现从Python对象转换为JSON字符串，并写入到json文件中，使用 json.dump() 方法：
```python
import json

a_obj = '[12, 34, 56]'
with open("1.json", "w") as f:
	json.dump(a_obj,f)
```

要实现从json文件中读取JSON字符串，并转换为Python对象，使用 json.load() 方法：
```python
import json

with open('1.json', "r") as f:
	a_str = json.load(f)
	print(a_str)
```


