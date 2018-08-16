# Iterator 和 for...of 循环

## 含义

Iterator 是一种接口，为各种数据结构提供统一的访问机制。任何数据只要部署了 Iterator 接口，就可以完成遍历操作。

Iterator 的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是 ES6 创造了一种新的遍历命令 `for...of` 循环，Iterator 接口主要供 `for...of` 消费。

## 默认 Iterator 接口

一种数据只要部署了 Iterator 接口，就称九种数据结构是“可遍历的”（iterable）。

ES6 规定，默认的 Iterator 接口部署在数据结构的 `Symbol.iterater` 属性上。可以这样说，只要一个数据结构拥有 `Synbol.iterator` 属性，就称它是“可遍历的”。

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

ES6 的某些数据结构原生具备 Iterator 接口，即不用做任何处理，就可以被 `for...of` 循环遍历。原生具备 Iterator 接口的数据结构如下。

- Array
- Map
- Set
- String
- TypedArray
- 函数的 argumengs 对象
- NodeList 对象

下面的例子是数组的 `Symbol.Iterator` 属性。

```javascript
const arr = [1, 2, 3];
const iter = arr[Symbol.iterator];

iter.next(); // {value: 1, done: false}
iter.next(); // {value: 2, done: false}
iter.next(); // {value: 3, done: false}
iter.next(); //{value: undefined, done: true}
```

下面是一个为对象添加 Iteraror 接口的例子。

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

对于类数组（存在键名和 `length` 属性），部署 Iterator 接口，有一个简便方法，就是 `Symbol.iterator` 方法直接引用数组的 Iterator 接口。

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

有了遍历器接口，数据接口就可以用 `for...of` 或者 `while` 循环遍历。

```javascript
const $iterator = ITERATOR[Symbol.iterator]();
const $result = $iterator.next();

while (!$result.done) {
    let x = $result.value;
    // ...
    $iterator = $iterator.next();
}

```

## 调用 Iterator 的场景

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

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。

- for...of
- Array.from
- Map(), Set(), WeakMap(), WeakSet() (比如 `new map([['a', 1], ['b', 2]])`)
- Promise.all()
- Promise.race()

## Iterator 接口与 Generator 函数

`Symbol.iterator` 方法的最简直实现，还是使用 Generator 函数。

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