# Symbol

## 概述

ES6引入了一种新的原始类型数据 `Symbol`，表示独一无二的值。它是 Javascript 语言的第七种数据类型。前六种是：`undefined`、`null`、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。

```javascript
let s = Symbol();

typeof s // "symbol"
```

Symbol通过`Symbol`函数生成。凡是属性名属于Symbol类型，都是独一无二的，不会和其他属性产生冲突。

Symbol函数可以接受一个字符串作为参数，表示对Symbol类型的描述。

```javascript
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // 'symbol(foo)'
s2.toString() // 'symbol(bar)'
```

如果 Symbol 函数的参数是一个对象，则会先调用对象的 `toString` 方法，将其转为字符串，再生成一个 Symbol 值。

```javascript
let obj = {
    toString: function () {
        return 'abc'
    }
}

let sym = Symbol(obj); // Symbol(abc)
```

注意，Symbol 函数的参数只是用来对当前 Symbol 值的描述，因此拥有相同参数的 Symbol 值也是不相等的。

```javascript
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

Symbol 值不能和其他类型的值进行运算，否则会报错。

但是 Symbol 可以显示的转为字符串。另外 Symbol 值还可以转为布尔值，但不能转为数值。

```javascript
let sym = Symbol('My symbol');

'your symbol is' + sym // 报错

String(Sym) // 'Symbol(My symbol)
Boolean(Sym) // true
Number(Sym) // 报错
```

## 作为属性名的Symbol

由于每个 Symbol 值都是不同的，这意味这 Symbol 值可以作为标识符，用于对象的属性名，就能保证不出现同名的属性。

```javascript
let mySymbol = Symbol();

//  第一种写法
let a = {};
a[mySymbol] = 'Hello';

// 第二种写法
let a = {
    [mySymbol]: 'Hello'
}

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello' });

// 以上的写法都能得到同样的结果
a[mySymbol] // 'Hello'
```

注意，Symbol 值作为对象属性值时，不能用点运算符。

```javascript
const mySymbol = Symbol();
const a = {};

a.mySymbol = 'Hello';
a[mySymbol] // undefined
a['mySymbol'] // 'Hello'
```

上面的代码中，因为点运算符后面总是字符串，所以不会读取 Symbol 作为标识符指代 的那个值，导致a的属性名实际是一个字符串，而不是一个 Symbol 值。所以，在对象的内部，用 Symbol 定义属性时，Symbol 值必须放在括号里面。

```javascript
let a = Symbol();
let obj = {
    [s]: function () {
        console.log(arguments);
    }
}

obj[a](123)
```
Symbol 类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

```javascript
const log = {};

log.levels = {
    DEBUG: Symbol('debug'),
    INFO: Symbol('info'),
    WARN: Symbol('warn')
}

console.log(log.levels.DEBUG, 'debug message');
console.log(log.levels.INFO, 'info message');
```

需要注意的一点是，Symbol值作为属性名时，该属性还是公开属性，不是私有属性。

## 实例：消除魔术字符串

```javascript
function getArea (shape, options) {
    let area = 0;

    switch (shape) {
        case 'Triange':
            area = .5 * options.width * options.height;
            break;
        /* more code */
    }

    return area;
}

getArea('Triange', {width: 100, height: 50});
```

上面的代码中`Triange`便是魔术字符串。它的多次出现，与代码形成“强耦合”，不利于代码的管理和维护。

常用的消除魔术字符串的方法，就是把它写成一个变量。

```javascript
const shapeType = {
    triange: 'triange'
}

// 修改为Symbol

const shapeType = {
    triange: Symbol()
}

function getArea (shape, options) {
    let area = 0;

    switch (shape) {
        case shapeType.triange:
            area = .5 * options.width * options.height;
            break
        /* more code */
    }

    return area;


getArea(shapeType.triange, {width: 100, height: 50});
```

## 属性名的遍历

Symbol作为属性名，不会出现在 `for...in` 、`for...of` 循环中，也不会被 `Object.keys()` 、 `Object.getOwnPtopertyNames()` 、`Json.stringify()` 返回。但是它不是私有属性，有一个 `Object.getOwnPropertySymbols()` 方法，可以获取指定对象的所有 Symbol 属性名。

```javascript
const obj = {};
let a = Symbol('a')
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'world';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```

下面一个例子是 `Object.getOwnPropertySymbols()` 与 `for...in` 循环， `Object.getOwnPropertyNames()` 方法进行对比。

```javascript
const obj = {};
let foo = Symbol('foo');

Object.defineProperty(obj, foo, {
    value: 'foobar'
});

for(let i in obj) {
    console.log(i); // 无输出
}

Object.getOwnPropertyNames(obj); // []

Object.getOwnPropertySymbols(obj) // [Symbol(foo)]
```

另外一个新的API，`Reflect.ownKeys()` 方法可以返回对象所有类型的键名，包括常规键名和 Symbol 键名。

```javascript
const obj = {
    [Symbol('my_key')]: 1,
    enum: 2,
    nonEnum: 3
}

Reflect.ownKeys(obj)
// [Symbol(my_key), enum, nonEnum]

    
```