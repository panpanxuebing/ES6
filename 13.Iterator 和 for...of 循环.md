# Iterator 和 for...of 循环

## 含义

Iterator 是一种接口，为各种数据结构提供统一的访问机制。任何数据只要部署了 Iterator 接口，就可以完成遍历操作。

Iterator 的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是 ES6 创造了一种新的遍历命令 `for...of` 循环，Iterator 接口主要供 `for...of` 消费。

## 默认 Iterator 接口

一种数据只要部署了 Iterator 接口，就称九种数据结构是“可遍历的”（iterable）。

ES6 规定，默认的 Iterator 接口部署在数据结构的 `Symbol.iterater` 属性上。可以这样说，只要一个数据结构拥有 `Synbol.iterator` 属性，就称它是“可遍历的”。

```javascript
const obj = {
    [Symbol.iterator]: function () {
        return {
            next: function () {
                return {
                    value: 1,
                    done: true
                }
            }
        }
    }
}
```

上面的代码中，`obj` 是可遍历的，因为它具有 `Symbol.iterator` 属性。

ES6 的某些数据结构原生具备 Iterator 接口，即不用做任何处理，就可以被 `for...of` 循环遍历。原生具备 Iterator 接口的数据结构如下。

- Array
- Map
- Set
- String
- TypedArray
- 函数的 argumengs 对象
- NodeList 对象

下面的例子是数组的 `Symbol.Iterator` 属性。

```javascript
const arr = [1, 2, 3];
const iter = arr[Symbol.iterator];

iter.next(); // {value: 1, done: false}
iter.next(); // {value: 2, done: false}
iter.next(); // {value: 3, done: false}
iter.next(); //{value: undefined, done: true}
```

下面是一个为对象添加 Iteraror 接口的例子。

```javascript
const obj = {
    data: ['hello', 'world'],
    [Symbol.iterator]: function () {
        const self = this;
        let index = 0;

        return {
            next: function () {
                if (index < self.data.length) {
                    return {
                        value: self.data[index++],
                        done: false
                    }
                } else {
                    return { value: undefined, done: true }
                }
            }
        }
    }
}

for (let i of obj) {
	console.log(i);
}
// hello
// world
```

对于类数组（存在键名和 `length` 属性），部署 Iterator 接口，有一个简便方法，就是 `Symbol.iterator` 方法直接引用数组的 Iterator 接口。

```javascript
let obj = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3,
    [Symbol.iterator]: [][Symbol.iterator]
};

for(let i of obj) {
    console.log(i);
}
// 'a' ,'b', 'c'
```

有了遍历器接口，数据接口就可以用 `for...of` 或者 `while` 循环遍历。

```javascript
const $iterator = ITERATOR[Symbol.iterator]();
const $result = $iterator.next();

while (!$result.done) {
    let x = $result.value;
    // ...
    $iterator = $iterator.next();
}

```

## 调用 Iterator 的场景

### 解构赋值

对数组和 Set 结构解构赋值时，会默认调用 `Iterator` 方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x, y] = set;
// x: 'a', y: 'b'

let [first, ...rest] = set;
// first: 'a', rest: ['b', 'c']
```

### 扩展运算符

```javascript
let str = 'hello';
[...str] // ["h", "e", "l", "l", "o"]

let arr = ['b', 'c'];
['a', ...arr, 'd']; // ["a", "b", "c", "d"]
```

任何部署了 Iterator 接口的数据接口都可以用扩展运算符转为数组。

```javascript
const arr = [...iterable]
```

### yield*

`yield*` 后面跟的是可遍历的结构，它会默认调用该结构的遍历器接口。

```javascript
let generator = function* () {
    yield 1;
    yield* [1, 2, 3];
    yield 4;
}

const iterator = generator();

iterator.next() // {value: 1, done: false}
iterator.next() // {value: 2, done: false}
iterator.next() // {value: 3, done: false}
iterator.next() // {value: 4, done: false}
iterator.next() // {value: undefined, done: true}
```

### 其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。

- for...of
- Array.from
- Map(), Set(), WeakMap(), WeakSet() (比如 `new map([['a', 1], ['b', 2]])`)
- Promise.all()
- Promise.race()

## Iterator 接口与 Generator 函数

`Symbol.iterator` 方法的最简直实现，还是使用 Generator 函数。

```javascript
let myIterator = {
    [Symbol.iterator]: function* () {
        yield 1;
        yield 2;
        yield 3;
    }
}

[...myIterator] // [1, 2, 3]

let obj = {
    *[Symbol.iterator] () {
        yield 'hello';
        yield 'world';
    }
}

for (let i of obj) {
    console.log(i);
}
// hello
// world
```

## 遍历器对象的 return(), throw()

`return` 方法的使用场合是，如果 `for...of` 循环提前退出（通常是因为出错，或者有 `break` 语句），就会调用 `return` 方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署 `return` 方法。

```javascript
function readLineSync() {
    return {
        [Symbol.iterator] () {
            return {
                next () {
                    return { done: false }
                },
                return () {
                    file.close();
                    return { done: true }
                } 
            }
        }
    }
}

// 情况一
for (let line of readLinesSync(fileName)) {
    console.log(line);
    break;
}

// 情况二
for (let line of readLinesSync(fileName)) {
    console.log(line);
    throw new Error();
}
```

`throw` 方法主要配合 Generator 函数使用，详情请见 《Generator 函数》这一章。

## for...of 循环

一个数据结构只要部署了 Iterator 接口，就可以用 `for...of` 循环遍历它的成员。也就是说，`for...of` 内部调用的是数据结构的 `Symbol.iterator` 方法。

### 数组

```javascript
const arr = ['a', 'b', 'c'];

for(let i of arr) {
    console.log(i);
}
// 'a'
// 'b'
// 'c

const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for(let i of obj) {
    console.log(i);
}
// 'a'
// 'b'
// 'c'
```

上面的代码中，空对象部署了数组 `arr` 的 `Symbol.iterator` 属性，结果 `arr` 的 `for...of` 循环，产生了与 `arr` 完全一样的结果。

`for...of` 循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。这点和 `for...in` 不一样。

```javascript
let arr = [1, 3, 4];
arr.foo = 'abc';

for(let i in arr) {
    console.log(arr[i]);
}
// 1 3 4 abc

for(let i of arr) {
    console.log(i);
}
// 1 3 4
```

Set 和 Map 结构

Set 和 Map 结构也原生具有 Iterator 接口，可以直接使用 `for...of`。

```javascript
const engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);

for (let i of engines) {
    console.log(i);
}
// Gecko
// Trident
// Webkit

const es6 = new Map();
es6.set('edition', 6);
es6.set('committee', 'TC39');
es6.set('standard', 'EMCA-262');

for(let [name, value] of es6) {
    console.log(name + ' :' + value);
}
// edition :6
// committee :TC39
// standard :EMCA-262
```

值得注意的是，Set 遍历返回的是一个值，Map 遍历返回的是一个数组，数组成员是当前 Map 成员的键名和键值。

### 计算生成的数据对象

数组、Set，Map 结构通过调用下面三种方法，返回的都是遍历器对象。

- keys 
- values
- entries

```javascript
let arr = ['a', 'b', 'c'];

for(let i of arr.keys()) {
    console.log(i);
}
// 0
// 1
// 2
```

### 类似数组的对象

例如字符串，DOM NodeList 对象，`arguments` 对象。

对于类似数组而不具有 Iterator 接口的对象可以通过 `Array.from()` 来将其转为数组。

### 对象

对象不能直接使用 `for...of` 循环，会报错，必须部署了 Iterator 接口后才能使用。我们通常使用 `for...in` 去遍历。

一种解决方法是使用 `Object.keys` 方法将对象的键名生成一个数组，然后遍历这个数组。

另一种方法是使用 Generator 函数将对象重新包装一下。

```javascript
const obj = {
    name: '张三',
    age: 30
};

function* entries (obj) {
    for(let key of Object.keys(obj)) {
        yield [key, obj[key]];
    }
}

for(let [key, value] of entries(obj)) {
    console.log(key, '->', value);
}
// name -> 张三
// age -> 30
```

### 与其他遍历器语法的比较

- `for` 循环：这种方法比较麻烦。

- 数组的 `forEach` 方法：中途无法跳出循环，`break` 或者 `return` 命令都不行。

- `for...in`：不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键，某些情况下会以任意顺序遍历键名。

相比之下，`for...of` 有以下优点。

-  有着 `for...in` 一样简便的语法，但是又没 `for...in` 的那些缺点。

- 不用于 `forEach` 语法，它可以与 `break`、`continue` 和 `return` 使用。

- 提供了遍历所有数据结构的统一操作接口。