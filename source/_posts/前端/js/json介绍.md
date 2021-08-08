---
title: Json介绍
date: 2019-11-25 14:30:27
categories: 
  - Js
tags:
  - Json
---

### JSON介绍

JSON是JavaScript Object Notation的缩写，意思是JavaScript 对象表示法

是存储和交换文本信息的语法。类似 XML，不过它比 XML 更小、更快，更易解析

官网的介绍 http://www.json.org/json-zh.html

JSON(JavaScript Object Notation) 是一种**轻量级的数据交换格式**。 易于人阅读和编写。同时也易于机器解析和生成。 它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。 JSON采用**完全独立于语言的文本格式**，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。 这些特性使JSON成为理想的数据交换语言。

### JSON 语法规则
JSON建构于两种结构：

>   "名称/值"对的集合 (键值对)

不同的语言中，它被理解为对象（object）、纪录（record）、结构（struct）、字典（dictionary）、哈希表（hash table）、有键列表（keyed list）、或者关联

对象是一个无序的**'名称/值'对**集合。一个对象以`{`（左括号）开始，`}`（右括号）结束。每个**名称**后跟一个`:`（冒号）；**'名称/值'对** 对之间使用`,`（逗号）分隔

```json
{"name":"张三","age":23,"salary":12.3,"sex":true}
```

>   数组

值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

数组是值（value）的有序集合。一个数组以`[`（左中括号）开始，`]`（右中括号）结束。值之间使用`,`（逗号）分隔

```json
["hello","boy",12,12,3,true]
```

值（value）可以是双引号括起来的字符串（`string`）、数值(`number`)、`true`、`false`、 `null`、对象（`object`）或者数组（`array`）。这些结构可以嵌套。

### JSON 与 JS 对象的关系
很多人搞不清楚 JSON 和 JS 对象的关系，甚至连谁是谁都不清楚。其实，可以这么理解：
**JSON 是 JS 对象的字符串表示法，它使用文本表示一个 JS 对象的信息，本质是一个字符串。**
如

```javascript
var obj = {a: 'Hello', b: 'World'}; //这是一个对象，注意键名也是可以使用引号包裹的

var json = '{"a": "Hello", "b": "World"}'; //这是一个 JSON 字符串，本质是一个字符串
```
### JSON 和 JS 对象互转
要实现从JSON字符串转换为JS对象，使用 JSON.parse() 方法：
```js
var obj = JSON.parse('{"a": "Hello", "b": "World"}'); //结果是 {a: 'Hello', b: 'World'}
```
要实现从JS对象转换为JSON字符串，使用 JSON.stringify() 方法：
```js
var json = JSON.stringify({a: 'Hello', b: 'World'}); //结果是 '{"a": "Hello", "b": "World"}'
```

