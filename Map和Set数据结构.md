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

向 Set 中加入值的时候，不会发生类型转换，所以 5 和 ‘5’ 是不同的值。Set 内部判别两个值是否相等的算法叫 ‘Same-value-zero equality’, 类似全等运算符（`===`），主要的区别是 `NaN` 等于自身。 

另外，两个对象总是不相等的。

```javascript
const set = new Set();

set.add({});
set.size // 1

set.add({});
set.size; // 2
```

## Set 实例的属性和方法

Set 结构有如下属性：

- `Set.prototype.constructor` : 构造函数，默认 Set 函数自身。
- `Set.prototype.size` : 返回 Set 实例的成员总数。

Set 结构的方法分为两类： 操作方法（用于操作数据）和遍历方法（用于遍历成员）。先介绍操作方法：

- `add(value)` : 添加某一个值，返回 Set 结构本身。
- `delete(value)` : 删除某个值，返回一个布尔值，表示删除是否成功。
- `has(value)`: 返回一个布尔值，表示该值是否为 Set 的成员
- `clear()` : 清除所有成员，没有返回值。

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

- `keys()` ：返回键名的遍历器。
- `values()` : 返回键值的遍历器。
- `entries()` : 返回键值对的遍历器。
- `forEach()` : 使用回调函数遍历每个成员。

需要特别注意的是，Set 的遍历顺序就是插入的顺序。

由于 Set 结构没有键名，只有键值，所以 `keys()` 和 `values` 方法完全一致。

```javascript
let set =   new Set(['red', 'green', 'blue']);

for (let item of Set) {
    console.log(item);
}
// 'red'
// 'green'
// 'blue'

for (let item of set) {
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

`entries` 方法：

```javascript
const set = new Set([1, 4, 9]);

set.forEach((key, value)) => console.log(key + ' : ' + values);)
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