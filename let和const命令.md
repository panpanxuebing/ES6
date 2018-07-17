# let 和 const 命令

## let 命令

### 基本用法

用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效。

```javascript
{
    let a = 10;
    var b = 1;
}

a // ReferenceError: a is not defined
b // 1
```

### 不存在变量提示

```javascript
// var 的情况
console.log(foo); // 输出undefined
var f00 = 2;

// let 的情况
console.log(bar); // 输入 ReferenceError
let bar = 2;
```

### 暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

### 不允许重复声明  

`let` 不允许在相同作用域内，重复声明同一个变量

## 快级作用域

### ES6 的快级作用域

外层作用域无法读取内层作用域的变量

```javascript
function f1 () {
    let n = 5;
    {
        let n = 10;
    }
    console.log(n); // 5
}
```

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE 写法
(function () {
    var tmp = ...;
    ...
})();

// 快级作用域写法
{
    let map = ...;
    ...
}
```

### 块级作用域和函数声明

为了兼容以往代码，在ES6中规定

- 允许在快级作用域内声明函数
- 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部
- 同时，函数声明还会提升到所在快级作用域的头部

注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作`let`处理。

```javascript
// 浏览器的 ES6 环境
function f () { consle.log('I am outside!') };

(function () {
    if (false) {
        function f () { console.log('I am inside!') }
    }

    f();
}());
// Uncaught TypeError： f is not a function
```

上面的代码在 ES6 浏览器中，都会报错，因为实际运行的代码是：

```javascript
// 浏览器的 ES6 环境
function f () { console.log('I am outside!') };

(function () {
    var f = undefined;
    if (false) {
        function f () { console.log('I am inside!') };
    }

    f();
}());
```

考虑到环境导致的行为差异太大，应该避免在快级作用域内声明函数。如果确实需要，也应写函数表达式，而不是函数声明语句。

```javascript
// 函数声明语句
{
    let a = 'secret';
    function f () {
        return a;
    }
}

// 函数表达式
{
    let a = 'secret';
    let f = function () {
        return a;
    }
}
```

另外，ES6 的快级作用域的规则，只在使用大括号内情况下成立，如果没有大括号，就会报错。

```javascript
// 不报错
'use strict'
if (true) {
    function f () {}
}

// 报错
'use strict'
if (true)
    function f () {}
```

## const 命令

### 基本用法

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。
这就意味着`const`一旦声明变量，就必须立即初始化，不能留到以后赋值。

```javascript
const foo;
// SyntaxError: Missing initializer in const declaration
```

`const`的作用域于`let`命令相同：只在声明所在的快级作用域内有效。

```javascript
if (true) {
    const MAX = 5;
}
MAX // Uncaught ReferenceError: MAX is not defined
```

`const`命令声明的变量也是不提示，同样存在暂时性死区，只能在声明的位置后面使用。
和`let`一样不可以重复声明。


### 本质
`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。
对于简单类型的数据（数值、字符换、布尔值），值就是保存在变量指向的那个内存地址，
因此等同于常量。对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存
的只是一个指针，`const`只能保证这个指针是固定的，至于它指向的数据结构是不是可变
的，就完全不能控制了。

```javascript
const foo = {};
// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {} // 报错

const a = [];
a.push('hello'); // 可执行
a.length = 0; // 可执行
a = ['Dave']; // 报错
```

如果真的想把对象冻结，应该使用`Obejct.freeze`方法
```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;

const foo = Object.freeeze({name: ['foo']});
// 但是对象的属性不会被冻结
foo.name.push('bar');
foo.name // ['foo','bar']
```

下面是一个将对象彻底冻结的函数。

```javascript
const constantize = (obj) => {
    Object.freeze(obj);
    Object.keys(obj).forEach(key => {
        if(typeof obj[key] === 'object') {
            constantize(obj[key]);
        }
    })
}
```

### 声明变量的六种方法

- ES5: var,function
- ES6: let, const, import, class

### 顶层对象的属性

顶层对象，在浏览器指的是`window`对象，在Node指的是`global对象`。ES5中，顶层对象的属性和全局变量是等价的。

```javascript
window.a = 1;
a // 1

a = 2;
window.a // 2
```
从ES6开始，全局变量将逐步与顶层对象脱钩

```javascript
let a = 1;
window.a // undefined
```

