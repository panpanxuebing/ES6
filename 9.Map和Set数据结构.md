# Map 和 Set 数据结构

## Set

ES6 提供新的数据结构 Set。它类似与数组，但它的成员的值都是唯一的，没有重复的值。Set 本身是一个构造函数，用来生成 Set 数据结构。

```javascript
const s = new Set();

[1, 2, 2, 3, 3, 4].map(x => s.add(x));

for (let i of s) {
    console.log(i)
}
// 1 2 3 4
```

上面的代码通过 `add` 向 Set 结构添加不同的值，但结果表明 Set 结构不会添加重复的值。

Set 函数可以接受一个数组（或者具有 iterator 接口的其他数据结构）作为参数，用来初始化。

```javascript
const set = new Set([1, 2, 2, 3, 5]);
[...set] // [1, 2, 3, 5]
set.size // 4
```

上面的代码也展示了一种去除数组重复成员的方法：

```javascript
[...new Set(array)]
```

向 Set 中加入值的时候，不会发生类型转换，所以 5 和 ‘5’ 是不同的值。Set 内部判别两个值是否相等的算法叫 ‘Same-value-zero equality’, 类似全等运算符（`===`），主要的区别是 `NaN` 等于自身。 

另外，两个对象总是不相等的。

```javascript
const set = new Set();

set.add({});
set.size // 1

set.add({});
set.size; // 2
```

## Set 实例的属性和方法

Set 结构有如下属性：

- `Set.prototype.constructor` : 构造函数，默认 Set 函数自身。
- `Set.prototype.size` : 返回 Set 实例的成员总数。

Set 结构的方法分为两类： 操作方法（用于操作数据）和遍历方法（用于遍历成员）。先介绍操作方法：

- `add(value)` : 添加某一个值，返回 Set 结构本身。
- `delete(value)` : 删除某个值，返回一个布尔值，表示删除是否成功。
- `has(value)`: 返回一个布尔值，表示该值是否为 Set 的成员
- `clear()` : 清除所有成员，没有返回值。

```javascript
const s = new Set();
s.add(1).add(2).add(2);

s.size; // 2

s.has(1); // true
s.delete(2) // true
s.has(2) // false
s.clear();
s.size // 0
```

`Array.from` 方法可以将 Set 数据结构转为数组。

```javascript
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);
// [1, 2, 3, 4, 5]
```
Set 操作方法：

- `keys()` ：返回键名的遍历器。
- `values()` : 返回键值的遍历器。
- `entries()` : 返回键值对的遍历器。
- `forEach()` : 使用回调函数遍历每个成员。

需要特别注意的是，Set 的遍历顺序就是插入的顺序。

由于 Set 结构没有键名，只有键值，所以 `keys()` 和 `values` 方法完全一致。

```javascript
let set =   new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
    console.log(item);
}
// 'red'
// 'green'
// 'blue'

for (let item of set.values()) {
    console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

Set 数据结构默认可遍历，它的默认遍历起就是它的 `values` 方法。

```javascript
Set.prototype[Symbol.iterator] === Set.prototype.values
```

`forEach` 方法：

```javascript
const set = new Set([1, 4, 9]);

set.forEach((key, value)) => console.log(key + ' : ' + values));
// 1 : 1
// 4 : 4
// 9 : 9
```

数组的 `filter` 和 `map` 方法可以间接用于 Set 数据结构。

```javascript
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2);
// 返回 Set 结构： {2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].map(x) => x % 2 === 0);
// 返回 Set 结构： {2, 4}
```

利用 Set 可以轻松实现并集，交集，差集（defference）。

```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 并集
let intersect = new Set([...a].filter(x => b.has(x)));
// Set {2, 3}

// 差集
let diffrence = new Set([...a].filter(x) => !b.has(x));
// Set {1}
```

如果想改变 Set 结构，又两种方法：

```javascript
// 方法一
let set = new Set([1, 2, 3]);

set = new Set([...set].map(x => x * 2));
// Set {2, 4, 6}

// 方法二
let set = new Set([1, 2, 3]);
set =  new Set(Array.from(set, val => val * 2));
// Set {2, 4, 6}
```

## WeakSet

WeakSet 含义 和 Set 类似，也是不含重复的值。但是有两个区别：首先 WeakSet 的成员都是对象，不能是其他类型的值。其次 WeakSet 中的对象都是弱引用。即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
 
```javascript
const ws = new WeakSet();
ws.add(1); // 报错
ws.add(Symbol()); // 报错
```

作为构造函数 WeakSet 可以接受数组或类数组的对象作为参数,而且数组的成员也必须是对象。

```javascript
let a = [[1, 2], [3, 4]];
let ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}

let a = [3, 4];
let ws = new WeakSet(a); // 报错
```

WeakSet 有三个方法： `add()`, `has()`, `delete()`。

WeakSet 没有 `size` 属性和 `forEach` 方法，不能遍历。WeakSet 的一个用处时储存 DOM 节点，而不用担心这些节点被移除时，会引发内存泄漏。

```javascript
const foos = new WeakSet();
class Foo {
    constructor () {
        foos.add(this);
    }

    method () {
        if (!foos.has(this)) {
            throw new TypeError('Foo.prototype.method 只能在实例上调用！');
        }
    }
}
```

上面的代码保证了 `Foo` 的实例方法，只能在实例上调用。这里使用 `WeakSet` 的好处是，`foos` 对实例的引用，不会计入内存回收机制，所以删除实例的时候，不用考虑 `foos` ，也不会出现内存泄露。

## Map

### 含义和基本用法

Javascript 的对象（Object），本质上是键值对的集合（Hash 结构），但是传统上只能用字符串当做键，这给使用带来了很大的限制。为了解决这个问题，ES6 提供了 Map 数据结构。它类似于对象，但是它的“键”不限于字符串，各种类型的值都能当做键。即 Object 提供了“字符串-值”的对应，Map 结构提供了“值-值”的对应，是一种更完美的 Hash 结构实现。

```javascript
const m = new Map();
const o = {p: 'hello world'};

m.set(o, 'content');
m.get(o); // 'content'

m.has(o); // true
m.delete(o);
m.has(o); // false
```

Map 构造函数可以接受一个数组作为参数，该数组的成员是一个个表示键值对的数组。

```javascript
const map = new Map([
    ['name', '张三'],
    ['title', 'Author']
]);

map.size; // 2
map.has('name'); // true
map.get('name'); // '张三'
map.has('title'); // true
map.has('title'); // 'Author'
```

`Map` 结构接受数组作为参数，实际上是执行下面的算法。

```javascript
const items = [
    ['name', '张三'],
    ['title', 'Author']
];

const map = new Map();

items.forEach(
    ([key, value]) => map.set(key, value)
);
```

事实上，不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构，都可以当做 `Map` 构造函数的参数。`Set` 和 `Map` 都可以用来生成新的 Map。

```javascript
const set = new Set([
    ['foo', 1],
    ['bar', 2]
]);
const m1 = new Map(set);

m1.get('foo'); // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);

m3.get('baz'); // 3
```

如果对同一个键重新赋值，后面的值将覆盖前面的值。

```javascript
const map = new Map();

map
    .set(1, 'aaa')
    .set(1, 'bbb');

map.get(1); // 'bbb'
```

如果读取一个未知的键，返回 `undefined`。

注意：只有对同一个对象的引用，Map 结构才将其视为一个键。

```javascript
const map = new Map();

map.set(['a'], 'aaa');
map.get(['a']) // undefined
```

如果 Map 的键名是一个简单类型的值，则只要两个键严格相等，Map 将视为一个键。

### 实例的属性和方法

Map 结构有如下属性和方法：

- (1) size 属性

返回 Map 结构的成员总数。

- (2) set(key, value)

- (3) get(key)

- (4) has(key)

- (5) delete(key)

- (6) clear()

遍历方法：

- keys()

- values()

- entries()

- forEach()

Map 的遍历顺序就是插入顺序。

```javascript
const map = new Map([
    ['F', 'no'],
    ['T', 'yes']
]);

for(let item of map.keys()) {
    console.log(item)
}
// 'F'
// 'T'

for(let [key, value] of map) {
    console.log(key, value);
}
// 'F' 'no'
// 'T' 'yes'
```

Map 结构转为数组结构，比较快速的是使用扩展运算符（`...`）。

```javascript
const map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three']
]);

[...map.keys] // [1, 2, 3]

[...map.values] // ['one', 'two', 'three']

[...map.entries] // [[1, 'one'], [2, 'two'], [3, 'three']]

[...map] // [[1, 'one'], [2, 'two'], [3, 'three']]
```

结构数组的 `Map` 和 `filter` 方法，可以实现 Map 的遍历和过滤（Map 本身没有这两个方法）。

```javascript
const map0 = new Map()
    .set(1, 'a')
    .set(2, 'b')
    .set(3, 'c');

const map1 = new Map(
    [...map0].filter(([k, v]) => k > 1)
);
// Map {2 => 'a', 3 => 'c'}

const map2 = new Map(
    [...map0].map(([k, v]) => [v * 2, '_' + v])
)
// Map {2 => '_a', 4 => '_b', 6 => '_c'}
```

### Map 与其他数据结构互相转换

- (1) Map 转为数组

使用扩展运算符 (`...`)

```javascript
const myMap = new Map()
    .set(true, 7)
    .set({foo: 3}, ['abc']);

[...myMap]
// [[true, 7], [{foo: 3}, ['abc']]]
```

- (2) 数组转为 Map

将数组传入 Map 构造函数，就可以转为 Map

```javascript
new Map([
    [true, 7],
    [{foo: 3}, ['abc']]
]);

// Map {
//     true => 7,
//     {foo: 3} => ['abc']
// }
```

- (3) Map 转为对象

如果所有 Map 的键都是字符串，就可以无损的转为对象。

```javascript
function strMaptoObj (strMap) {
    const obj = Object.create(null);

    for(let [k, v] of strMap) {
        obj[k] = v;
    }

    return obj;
}

const myMap = new Map([
    ['yes', true],
    ['no', false]
])

strMapToObj(myMap);
```

- 对象转为 Map

```javascript
function objToStrMap(obj) {
    const myMap = new Map();

    for (let [k, v] of Object.entries()) {
        myMap.set(k, v);
    }

    return myMap;
}

const obj = {['yes', true], ['no', false]};
objToStrMap(obj);
```

- (5) Map 转为 Json

```javascript
// 情况一： Map 的键名都是字符串，可以转为对象字符串
function strMapToJson(strMap) {
    return JSON.stringify(strMapToObj(strMap));
}

const strMap = new Map([["yes", true], ["no", false]]);
strMapToJson(strMap);
// '{"yes": true, "no": false}'

// 情况二： Map 的键名有非字符串，可以转为数组字符串
function mapToArrayJson(map) {
    return JSON.stringify([...map]);
}
```

- (6) Json 转为 Map

```javascript
// json 所有键名都是字符串
function jsonToStrMap(jsonStr) {
    return objToStrMap(JSON.parse(jsonStr));
}

// json 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。
function jsonToMap(jsonStr) {
   return new Map(JSON.parse(jsonStr)); 
}
```

## WeakMap

与 Map 类似，也是用于生成键值对的集合。

与 Map 的区别，WeakMap 只接受对象作为键名，且 WeakMap 的键名所指向的对象，不计入垃圾回收机制。

注意：WeakMap 弱引用的只是键名，键值还是正常引用。

```javascript
const wm = new WeakMap();
const key = {};
const obj = {foo: 3};

wm.set(key, obj);
obj = null;
wm.get(key);
// {foo: 3}
```

 WeakMap 的用途

 ```javascript
// DOM 节点作为键名
let myElement = document.getElementById('logo');
let myWeakMap = new WeakMap();

myWeakMap.set(myElement, {timesClicked: 0});

myElemen.addEventListener('click', function () {
    let logData = myWeakMap.get(myElement);
    logData.timesClicked ++;
}, false);

// 部署私有属性
const _count = new WeakMap();
const _action = new WeakMap();

class CountDown {
    constructor (count, action) {
        _counter.set(this, conter);
        _action.set(this, action);
    }

    dec () {
        let counter = _counter.get(this);
        
        if(counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
            _action.get(this)();
        }
    }
}

const c = new CountDown(2, () => console.log('DONE'));

c.dec()
c.dec();
// 'DONE'
 ```