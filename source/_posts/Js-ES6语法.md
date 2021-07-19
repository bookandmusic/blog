---
title: ES6语法
date: 2020-04-12 23:13:05
categories:
- JS
- ES6
tags:
- ES6
- var
- let
- const
- 箭头函数
---

## 变量声明

### 1. `let`

- `let`作用域只局限于当前代码块

     ```javascript
     {
        var a = 10
        let b = 20
     }

     console.log(a)  // 输出 10
     console.log(b)  // 报错 b is not defined
     ```

- 使用`let`声明的变量作用域不会提前

     ```javascript
     console.log(a)  // 输出 undefined
     console.log(b)  // 报错 Cannot access 'b' before initialization

     var a = 10
     let b = 20
     ```

- 在相同的作用域下不能声明相同的变量

     ```javascript
           {
          let a = 10
      }
      let a = 20
     ```

     ```javascript
      var a = 10
      let a = 20  // 报错 Identifier 'a' has already been declared
     ```

### 2. const

- 声明一个只读的常量。一旦声明，常量的值就不能改变

- `const`只声明，不赋值，就会报错，这意味着`const`一旦声明常量，就必须立即初始化

- `const`作用域与`let`相同，只在所在的块级作用域内有效

- `const`声明的常量不支持提升，和`let`一样，只能在声明后使用

- `const`声明的常量，也与`let`一样， 不可重复声明

- `const`实际保证的并不是常量的值不变，而是常量指向的内存地址不得改变
  
  - 对于简单的数据类型(数字、字符串、布尔值),值就保存在常量指向的地址，因此等同常量

  - 对于复合类型的数据(数组、对象)，常量指向的内存地址保存的只是一个指针，const只能保证指针是固定的，但是指针指向的数据结构是不是可变的，并不能保证

    ```javascript
    const a = 5
    // a = 10 // 报错 Assignment to constant variable
    
    const obj = {}
    obj.a = 20 // 可执行
    console.log(obj) // 可执行 {a: 20}
    // obj = {} // 报错
    
    const arry = []
    arry.push(1) // 可执行
    arry.length = 0 // 可执行
    // arry = [] // 报错 Assignment to constant variable
    ```

### 3. 实例演示

- 使用var声明循环变量，会导致全局共用一个变量

  ```html
  <button>按钮</button>
  <button>按钮</button>
  <button>按钮</button>
  <button>按钮</button>
  <button>按钮</button>

  <script>
      var btns = document.querySelectorAll('button')
      for (var i = 0; i < btns.length; i++) {
          btns[i].onclick = function () {
              console.log(`这是第${i}个按钮`)
          }
      }
      // 此时点击任何按钮，输出都是"这是第5个按钮"

      for (let j = 0; j < btns.length; j++) {
          btns[j].onclick = function () {
              console.log(`这是第${j}个按钮`)
          }
      }
      // 把循环中的var改为let,则为正确的结果，即输出对应的按钮序号
  </script>
  ```

- 下面的例子也是for循环中全局共用同一变量的弊端

    ```javascript
    for(var i=1;i<10;i++){
        setTimeout(()=>{
            console.log(i) // 输出 10个9
        }, 100)
    }
    console.log(i) // 9

    for(let j=1;j<10;j++){
        setTimeout(()=>{
            console.log(j) // 输出 0-9
        }, 100)
    }
    console.log(j) // 报错 j is not defined
    ```

    > 解释

- var是全局作用域，有变量提升的作用，所以在for中定义一个变量，全局可以使用，循环中的每一次给变量i赋值都是给全局变量i赋值。

- let是块级作用域,只能在代码块中起作用，在js中一个{}中的语句我们也称为叫一个代码块，每次循环会产生一个代码块，每个代码块中的都是一个新的变量j

### 4. 小结

> `let`、`const`使用场景

- `const`一般用在需要一个模块时使用或者定义一些全局常量时使用
- `let`限制了变量的作用域，保证变量不会影响全局变量，所以尽量将`var`改为`let`

## 箭头函数

### 1. 箭头函数

ES6标准新增了一种新的函数：Arrow Function（箭头函数）。

为什么叫Arrow Function？因为它的定义用的就是一个箭头

```javascript
x => x * x
```

上面的箭头函数相当于：

```javascript
function (x) {
    return x * x;
}
```

箭头函数相当于匿名函数，并且简化了函数定义。箭头函数有两种格式。一种像上面的，只包含一个表达式，连`{ ... }`和`return`都省略掉了。还有一种可以包含多条语句，这时候就不能省略`{ ... }`和`return`：

```javascript
x => {
    if (x > 0) {
        return x * x;
    }
    else {
        return - x * x;
    }
}
```

如果参数不是一个，就需要用括号()括起来：

```javascript
// 两个参数:
(x, y) => x * x + y * y

// 无参数:
() => 3.14

// 可变参数:
(x, y, ...rest) => {
    var i, sum = x + y;
    for (i=0; i<rest.length; i++) {
        sum += rest[i];
    }
    return sum;
}
```

如果要返回一个对象，就要注意，如果是单表达式，这么写的话会报错：

```javascript
// SyntaxError:
x => { foo: x }
```

因为和函数体的{ ... }有语法冲突，所以要改为：

```javascript
// ok:
x => ({ foo: x })
```

### 2. 箭头函数与匿名函数异同

箭头函数看上去是匿名函数的一种简写，但实际上，箭头函数和匿名函数有个明显的区别:

箭头函数的提出，主要是为了解决多层函数嵌套中this指代混乱的问题；箭头函数内部的this是词法作用域，由上下文确定。

- 使用普通函数嵌套会产生this指代混乱的问题

    ```javascript
    var person = {
        age: 18,
        getBirth: function () {
            var bth = function () {
              // 多层嵌套函数中，this优先寻找父级作用域对象，不存在，即指向全局window对象
                return new Date().getFullYear() - this.age  // this.age   undefined
            }
            return bth()
        }
    }
    console.log(person.getBirth()) // NaN
    ```

- 使用箭头函数即可解决问题

    ```javascript
    var person = {
        age: 18,
        getBirth: function () {
            var bth = ()=>{
              // 在箭头函数中 this会寻找父级作用域的对象，找到为止
                return new Date().getFullYear() - this.age  // this.age  18
            }
            return bth()
        }
    }
    console.log(person.getBirth()) // 2002
    ```

### 3. 箭头函数不适用的场景

1. 作为对象的属性

    ```javascript
    var obj = {
        a: () => {
            console.log(this); // window
        }
    };
    ```

    作为对象的属性时，this的指向则不再是对象本身了，这就造成了意想不到的结果

2. 不可以作为构造函数来使用

    ```javascript
    var Person = (name) => { // Uncaught TypeError: Person is not a constructor
        this.name = name;
    }
    
    var person = new Person('Jack');
    ```

    这个特性很容易测试，如果上一条明白的话也很容易理解： 箭头函数压根就没有this，当然不能作为构造函数（如果明白构造函数new的过程的话，插一句： new的过程其实就是创建一个对象，将this指向该对象，然后执行代码初始化这个对象，最后返回）
3. 不可以作为原型方法

    同样的规则适用于原型方法（prototype method）的定义，使用箭头函数会导致运行时的执行上下文错误

    ```javascript
    function Cat(name) {
        this.name = name;
    }

    Cat.prototype.sayCatName = () => {
        console.log(this === window); // => true
        return this.name;
    };

    const cat = new Cat('Mew');
    cat.sayCatName(); // => undefined
    ```

    使用传统的函数表达式就能解决问题：

    ```javascript
    function Cat(name) {
        this.name = name;
    }

    Cat.prototype.sayCatName = function () {
        console.log(this === cat); // => true
        return this.name;
    };

    const cat = new Cat('Mew');
    cat.sayCatName(); // => 'Mew'
    ```

    sayCatName 变成普通函数之后，被调用时的执行上下文就会指向新创建的 cat 实例。

4. 不绑定arguments（如果有要使用arguments的时候可以使用rest参数代替）

    ```javascript
    var foo = (val) => {
        console.log(arguments); // Uncaught ReferenceError: arguments is not defined
    };
    foo();
    ```

    这个特性也很好测试，但是实在要使用`arguments`对象要怎么办呢？我们可以使用es6的另一个新特性rest参数，完美替代

    ```javascript
    var foo = (...args) => {
        console.log(args); // [1, 2, 3]
    };
    foo(1, 2, 3);
    ```

## JS对象嵌套函数

在 `js` 的语法规则中，如果一个对象内存在方法，则该方法 `this` 指向当前对象。
如下所示：

```js
let obj = {
    age: 10,
    getAge: function () { 
        console.log(this.age)
    }
}
obj.getAge();// 10
```

若方法中再次包含函数方法，则该嵌套函数指向全局。不能获取到本对象中的属性。 
如下所示：

```js
let obj = {
    age: 10,
    getAge: function () { 
        setTimeout(function(){ alert(this.age)}, 1000)
    }
}
obj.getAge();// undefined
```

可以通过如下几种办法来解决此问题：  
（1）定义变量，保存对象 `this`

```js
getAge: function () { 
		let self = this;
        setTimeout(function(){ alert(self.age)}, 1000)
    }
```

(2) 通过箭头函数的方式 

由于箭头函数具有默认指向父级调用对象 obj 的特点，因此也可以解决本问题

```js
getAge: function () { 
		setTimeout(() => { alert(this.age)}, 1000)
}
```

## 解构赋值

### 什么是解构赋值？

解构赋值允许你使用类似数组或对象字面量的语法将数组和对象的属性赋给各种变量。这种赋值语法极度简洁，同时还比传统的属性访问方法更为清晰。

通常来说，你很可能这样访问数组中的前三个元素：

```js
var first = someArray[0];
var second = someArray[1];
var third = someArray[2];
```

如果使用解构赋值的特性，将会使等效的代码变得更加简洁并且可读性更高：

```js
var [first, second, third] = someArray;
```

`SpiderMonkey`（Firefox 的 JavaScript 引擎）已经支持解构的大部分功能，但是仍不健全。你可以通过 [bug 694100](https://bugzilla.mozilla.org/show_bug.cgi?id=694100) 跟踪解构和其它 ES6 特性在 `SpiderMonkey` 中的支持情况。

### 数组与迭代器的解构

以上是数组解构赋值的一个简单示例，其语法的一般形式为：

```js
[ variable1, variable2, ..., variableN ] = array;
```

这将为 variable1 到 variableN 的变量赋予数组中相应元素项的值。如果你想在赋值的同时声明变量，可在赋值语句前加入`var`、`let`或`const`关键字，例如：

```js
var [ variable1, variable2, ..., variableN ] = array;
let [ variable1, variable2, ..., variableN ] = array;
const [ variable1, variable2, ..., variableN ] = array;
```

事实上，用`变量`来描述并不恰当，因为你可以对任意深度的嵌套数组进行解构：

```js
var [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo);  // 1
console.log(bar);  // 2
console.log(baz);  // 3
```

此外，你可以在对应位留空来跳过被解构数组中的某些元素：

```js
var [,,third] = ["foo", "bar", "baz"];
console.log(third);  // "baz"
```

而且你还可以通过 “[不定参数](http://www.infoq.com/cn/articles/es6-in-depth-rest-parameters-and-defaults)” 模式捕获数组中的所有尾随元素：

```js
var [head, ...tail] = [1, 2, 3, 4];
console.log(tail); // [2, 3, 4]
```

当访问空数组或越界访问数组时，对其解构与对其索引的行为一致，最终得到的结果都是：`undefined`。

```js
console.log([][0]);   // undefined
var [missing] = [];
console.log(missing); // undefined
```

请注意，数组解构赋值的模式同样适用于任意迭代器：

```js
function* fibs() {
      var a = 0;
      var b = 1;
      while (true) {
        yield a;
        [a, b] = [b, a + b];
      }
}
var [first, second, third, fourth, fifth, sixth] = fibs();
console.log(sixth);  // 5
```

### 对象的解构

通过解构对象，你可以把它的每个属性与不同的变量绑定，首先指定被绑定的属性，然后紧跟一个要解构的变量。

```js
var robotA = { name: "Bender" };
var robotB = { name: "Flexo" };
var { name: nameA } = robotA;
var { name: nameB } = robotB;

console.log(nameA);  // "Bender"
console.log(nameB);  // "Flexo"
```

当属性名与变量名一致时，可以通过一种实用的句法简写：

```js
var { foo, bar } = { foo: "lorem", bar: "ipsum" };

console.log(foo);  // "lorem"
console.log(bar);  // "ipsum"
```

与数组解构一样，你可以随意嵌套并进一步组合对象解构：

```js
var complicatedObj = {
  arrayProp: [
    "Zapp",
    { second: "Brannigan" }
  ]
};
var { arrayProp: [first, { second }] } = complicatedObj;
console.log(first);
// "Zapp"
console.log(second);
// "Brannigan"
```

当你解构一个未定义的属性时，得到的值为`undefined`：

```js
var { missing } = {};
console.log(missing);
// undefined
```

请注意，当你解构对象并赋值给变量时，如果你已经声明或不打算声明这些变量（亦即赋值语句前没有`let`、`const`或`var`关键字），你应该注意这样一个潜在的语法错误：

```js
{ blowUp } = { blowUp: 10 };
// Syntax error 语法错误
```

为什么会出错？这是因为 JavaScript 语法通知解析引擎将任何以 {开始的语句解析为一个块语句（例如，`{console}`是一个合法块语句）。解决方案是将整个表达式用一对小括号包裹：

```js
({ safe } = {});
// No errors 没有语法错误
```

### 解构值不是对象、数组或迭代器

当你尝试解构`null`或`undefined`时，你会得到一个类型错误：

```js
var {blowUp} = null;
// TypeError: null has no properties（null没有属性）
```

然而，你可以解构其它原始类型，例如：`布尔值`、`数值`、`字符串`，但是你将得到`undefined`：

```js
var {wtf} = NaN;
console.log(wtf);
// undefined
```

你可能对此感到意外，但经过进一步审查你就会发现，原因其实非常简单。当使用对象赋值模式时，被解构的值[需要被强制转换为对象](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-requireobjectcoercible)。大多数类型都可以被转换为对象，但`null`和`undefined`却无法进行转换。当使用数组赋值模式时，被解构的值一定要[包含一个迭代器](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-getiterator)。

### 默认值

当你要解构的属性未定义时你可以提供一个默认值：

```js
var [missing = true] = [];
console.log(missing);
// true
var { message: msg = "Something went wrong" } = {};
console.log(msg);
// "Something went wrong"
var { x = 3 } = {};
console.log(x);
// 3
```