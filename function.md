# 函数的拓展

## 函数参数的默认值

### 基本用法

ES6允许为函数设定默认值，即直接写在参数定义的后面

```javascript
function Point (x = 0, y = 0) {
    this.x = x;
    this.y = y;
}

const p = new Point();
p // {x: 0, y: 0}
```

### 与解构赋值默认值结合使用

```javascript
function foo ({x, y = 5}) {
    console.log(x, y);
}

foo(); // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // 报错

function foo ({x, y = 5} = {}) {
    console.log(x, y);
}
foo() // undefined 5
```

下面两种设置默认值的写法中，建议采用第一种：

```javascript
// 写法一
function m1 ({x: 0, y: 0} = {}) {
    return [x, y];
}

// 写法而
function m2 ({x, y} = {x: 0, y: 0}) {
    return [x, y];
}

m1() // [0, 0]
m2() // [0, 0]

m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

m1({}) // [0, 0]
m2({}) // [undefined, undefined]
```

### 函数的 length 属性

指定了默认值后，函数的`length`属性,将返回没有指定默认值的参数个数。也就是说，指定了默认值后，`length`属性将失真。且设定的不是尾参数，那么`length`属性也不计入后面的参数了。

```javascript
(function (a = 1) {}).length // 0
(function (a = 1, b, c){}).length // 0
```

### 应用

利用参数默认值，可以指定默认参数不能省略,如果省略就跑出一个错误。

```javascript
function throwIfMissing () {
    throw new Error('Missing parameter')
}

function foo (mustProvided = throwIfMissing()) {
    return mustProvided;
}

foo();
// Error: Missing parameter
```

可以将参数默认值设为`undefined`，表明这个参数是可省略的。

```javascript
function foo (optional = undefined) { 
    //... 
}
```

## rest 参数

ES6引入 rest 参数(形式为`...变量名`)，用于获取函数的多余参数，这样就不需要`arguments`对象了。

```javascript
function add (...values) {
    let sum = 0;
    
    for(var val of values) {
        sum += val;
    }

    return sum;
}

add(2, 5, 3) // 10
```

下面是一个 rest 参数代替`arguments`对象的例子

```javascript
// arguments 变量写法
function sortNumbers () {
    return Array.prototype.slice.call(arguments).sort();
}

// rest 参数写法
function sortNumbers = (...numbers) => numbers.sort();
```

注意：rest 参数后不能有其他参数。

## 箭头函数

### 基本用法

```javascript
var f = v => v;

// 等同于
var f = function (v) {
    return v;
}
```

如果箭头函数的参数不存在或者不需要参数，就是用一个圆括号代表参数部分。

```javascript
var f = () => 5;
```

如果箭头函数的代码块部分多余一条语句，就要用大括号将它们括起来，并使用`return`语句返回。

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。

```javascript
// 报错
let getTempItem = id => {id, name: 'Temp'};

//不报错
let getTempItem = id => ({id, name: 'Temp'});
```

箭头函数简化回调函数。

```javascript
// 正常函数写法
[1, 2, 3]map(function (x) {
    return x * x;
})

// 箭头函数
[1, 2, 3].map(x => x * x);
```

rest 函数和箭头函数结合：

```javascript
const numbers = (...numbers) => nums;

numbers(1, 2, 3, 4, 5) // [1, 2, 3, 4, 5]

const headAndTail = (head, ...tail) => [head, tail]

headAndTail(1, 2, 3, 4, 5); // [1, [2, 3, 4, 5]]
```

### 使用注意点

箭头函数有几个使用注意点：

- 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。
- 不可以当做构造函数，即不可以使用`new`命令，否则会抛出一个错误。
- 不可以使用`arguments`对象，该对象在箭头函数内不存在，可以用`rest`参数代替。
- 不可以使用`yield`命令，因此箭头函数不能用作Generator函数。

特别要注意第一点：

```javascript
function foo () {
    setTimeout( () => {
        console.log(this.id);
    })
}

var id = 21;

foo.call({id: 42})
// 42
```
箭头函数可以让`setTiemout`中的`this`，绑定定义时所在的作用域，而不是运行时所在的作用域。

```javascript
function Timer () {
    this.s1 = 0;
    this.s2 = 0;

    setInterval(() => this.s1++, 1000)
    setInterval(() => this.s2++, 1000)
}

var timer = new Timer();

setTimeout(() => console.log('s1: ' + timer.s1), 3100); // 3
setTimeout(() => console.log('s2: ' + timer.s2), 3100); // 0
```

箭头函数可以让`this`指向固定化，这种特性有利于封装回调函数。

```javascript
var handler = {
    id: '123',
    init: function () {
        // this在这里指向handler对象，而在普通函数中，this会指向document对象
        document.addEventListener('click',
        () => this.doSomething(event.type), false);
    },
    doSomething: function (type) {
        console.log('Handing ' + type + ' for ' + this.id);
    }
}
```

所以，箭头函数转成 ES5 的代码如下：

```javascript
// ES6
function foo () {
    setTimeout(() => {
        console.log('id: ' + this.id)
    }, 100);
}

// ES5
function foo () {
    var _this = this;
    setTimeout(function() {
        console.log('id: ' + _this.id);
    }, 100)
}
```

### 嵌套箭头函数

箭头函数内部，可以嵌套箭头函数。

```javascript
// ES5
function insert (value) {
    return {into: function (array) {
        return {after: function (afterValue) {
            array.splice(array.indexOf(afterValue) + 1, 0, value);
            return array;
        }}
    }}
}

// ES6
let insert = value => ({
    into: array => ({
        after: afterValue => {
            array.splice(array.indexOf(afterValue) + 1, 0, value);
            return array;
        }
    })
})
```

下面是一个部署管道机制（pipeline）的例子，即前一个函数的输出是后一个函数的输入。

```javascript
// ES5
function pipeline (...funcs) {
    return function (val) {
        return funcs.reduce(function (a, b) {
            return b(a);
        }, val);
    } 
}

// ES6
const pipeline = (...funcs) =>
    val => funcs.reduce((a, b) => b(a), val);

const plus1 = a => a + 1;
const mult2 = a => a * 2;

const addThenMult = pipeline(plus1, mult2);

addThenMult(5); // 12
```

上面的写法可读性较差，可以采用下面的写法

```javascript
const plus1 = a => a + 1;
const mults = a => a * 2;

plus1(mults(5)) // 12
```

## 双冒号运算符

函数绑定运算符是并排的两个冒号（`::`），冒号左边是对象，右边是函数。该运算符会自动将左边的对象，作为上下文环境（即`this`对象），绑定到右边的函数上面。

```javascript
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments)
// 等同于
bar.apply(foo, arguments)

foo
```

## 尾调用优化

### 尾调用

尾调用时函数式编程的一个重要概念，指在某个函数的最后一步是调用另一个函数

```javascript
function f (x) {
    return g(x);
}
```

上面的例子中函数`f`的最后一步是调用函数`g`，这就是尾调用。

下面三种情况都不属于尾调用：

```javascript
// 情况一： 调用函数g后，还有赋值操作
function f (x) {
    let y = g(x);
    return y;
}

// 情况二：同情况一，调用后还有操作
function f (x) {
    return g(x) + 1
}

// 情况三
function f (x) {
    g(x)
}

// 等同于
function f (x) {
    g(x);
    return undefined;
}
```

尾调用不一定出现在函数尾部，只要最后一步操作即可。

```javascript
function f (x) {
    if (x > 0) {
        return m(x);
    }
    return n(x);
}
```

上面的`m`和`n `都属于尾调用，因为他们都是函数`f`的最后一步操作。


