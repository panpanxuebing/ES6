# Reflect

## 概述

`Reflect` 对象和 `Proxy` 对象一样，也是 ES6 为了操作对象而提供的 API。 `Reflect` 的设计目的是：

- 将 `Object` 对象的一些明显属于语言内部的方法（比如 `Object.defineProperty`），放到 `Reflect` 对象上。也就是说从 `Reflect` 上能拿到语言内部的方法。

- 修改某些 `Object` 方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)` 无法定义属性时，会抛出一个错误，而 `Reflect.defineProperty(obj, name, desc)` 则会返回 `false`。

- 让 `Object` 操作变为函数行为。某些 `Object` 操作是命令式，比如 `name in obj` 和 `delete obj[name]`，而  `Reflect.has(obj, name)` 和 `Reflect.deleteProperty(obj, name)` 则让它们变成了函数行为。

- `Reflect` 对象的方法于 `Proxy` 对象的方法一一对应，只要是 `Proxy` 对象的方法，就能在 `Reflect` 对象上找到对应的方法。这就让 `Proxy` 对象可以方便地调用对应的 `Reflect` 方法，完成默认行为，作为修改行为的基础。也就是说，不管 `Proxy` 怎么修改默认行为，你总可以在 `Reflect` 上获取默认行为。

```javascript
Proxy(target, {
    set (target, name, value, receiver) {
        var success = Reflect.set(target, name, value, receiver);

        if (success) {
            log('property ' + name + ' on ' + target + ' set to ' + value);
        }

        return success;
    }
});
```

下面是另一个例子：

```javascript
var obj = {foo: 123};
var loggedObj = new Proxy(obj, {
    get (target, name) {
        console.log('get', target, name);
        return Reflect.get(target, name);
    },
    deleteProperty (target, name) {
        console.log('delete ' + name);
        return Reflect.deleteProperty(target, name);
    },
    has (target, name) {
        console.log('has ' + name);
        return Reflect.has(target, name);
    }
});

loggedObj.foo;
// get {foo: 123} foo
// 123
'foo' in loggedObj;
// has foo
// true
delete loggedObj.foo
// delete foo
// true
```

## 静态方法

`Reflect` 对象一共有 13 个静态方法。

- `Reflect.apply(target, thisArg, args)`
- `Reflect.construct(target, args)`
- `Reflect.get(target, name, receiver)`
- `Reflect.set(target, name, value, receiver)`
- `Reflect.defineProperty(target, name, desc)`
- `Reflect.deleteProperty(target, name)`
- `Reflect.has(target, name)`
- `Reflect.ownKeys(target)`
- `Reflect.isExtensible(target)`
- `Reflect.preventExtensions(target)`
- `Reflect.getOwnPropertyDescriptor(target, name)`
- `Reflect.getPrototypeOf(target)`
- `Reflect.setPrototypeOf(target, prototype)`

上面这些方法的作用，大部分与 `Object` 对象的同名方法的作用都是相同的，而且它与Proxy对象的方法是一一对应的。下面是对它们的解释。

### Reflect.get(target, name, receiver)

`Reflect.get` 方法查找并返回 `target` 对象的 `name` 属性，如果没有该属性，则返回 `undefined`。

```javascript
var myObject = {
    foo: 1,
    bar: 2,
    get baz () {
        return this.foo + this.bar;
    }
}

Reflect.get(myObject, 'foo'); // 1
Reflect.get(myObject, 'bar'); // 2
Reflect.get(myObject, 'baz'); // 3
```

如果 `name` 属性部署了读取函数（getter），则读取函数的 `this` 绑定 `receiver`。

```javascript
var myObject = {
    foo: 1,
    bar: 2,
    get baz () {
        return this.foo + this.bar;
    }
}

var myReceiverObject = {
    foo: 4,
    bar: 4
}

Reflect.get(myObject, 'baz', myReceiverObject); // 8    
```

### Reflect.set(target, name, value, receiver)

`Reflect` 方法设置 `target` 对象的 `name` 属性等于 `value`。

```javascript
var myObject = {
    foo: 1,
    set baz (value) {
        return this.foo = value
    }
}

var myReceiverObject = {
    foo: 0
}

Reflect.set(myObject, 'baz', 4);
myObject.foo; // 4

// 如果 name 属性设置了赋值函数，则赋值函数的 this 绑定 receiver。

Reflect.set(myObject, 'baz', 1, myReceiverObject);
myObject.foo; // 4
myReceiverObject.foo; // 1
```

### Reflect.has()

`Reflect.has` 方法对应 `name in obj` 里面的 `in` 运算符。

```javascript
var myObject = {
    foo: 1,
}

// 旧写法
'foo' in myObject // true

// 新写法
Refelct.has(myObject, 'foo'); // true
```

### Reflect.deleteProperty(obj, name)

`Reflect.deleteProperty` 方法等同于 `delete obj[name]`，用于删除对象的属性。

```javascript
var myObject = {
    foo: 1,
};

// 旧写法
delete myObject.foo;

// 新写法
Refelct.deleteProperty(myObject, 'foo');
```

### Reflect.construct(target, args)

`Reflect.constrcut` 方法等同于 `new target(...args)`，这提供了一种不使用 `new`，来调用构造函数的方法。

```javascript
function Greeting () {
    this.name = name;
}

// 旧的写法
const instance = new Greeting('张三');

// 新的写法
const instance = Reflect.construct(Greeting, ['张三']);
```

### Reflect.getPrototypeOf(obj)

`Reflect.getPrototypeOf` 方法用于读取对象的 `__proto__` 属性，对应 `Object.getPrototypeOf`。它们的区别是，如果参数不是对象，前者会报错，而后者会将参数转化为对象，然后再运行。

### Reflect.setPrototypeOf(obj, newProto)

`Reflect.setPrototypeOf` 方法用于设置目标对象的原型，对应 `Object.setPrototypeOf(obj, newProto)` 方法。返回一个布尔值，表示是否设置成功。

如果第一个参数不是对象，`Object.setPrototypeOf` 会返回第一个参数本身，`Reflect.setPrototypeOf` 会报错。

如果第一个参数是 `undefined` 或 `null`，两者都会报错。

### Reflect.apply()

`Reflect.apply` 方法等同于 `Function.prototype.apply.call(func, thisArg, args)`，用于绑定 `this` 对象后执行给定函数。

```javascript
const ages = [1, 2, 3, 4, 5];

// 旧写法
const youngest = Math.min.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngset, []);
```

### Reflect.defineProperty(target, propertyKey, attributes)

`Reflect.defineProperty` 方法基本同 `Object.defineProperty`，用来为对象定义属性。未来，后者将逐渐被前者取代，所以从现在开始就使用 `Reflect.defineProperty` 来替代它。

### Reflect.getOwnPropertyDescriptor(target, propertyKey)

`Reflect.getOwnPropertyDescriptor` 基本等同于 `Object.getOwnPropertyDescriptor`，用于得到指定属性的描述对象，将来会替代掉后者。

### Reflect.isExtensible(target)

`Reflect.isExtensible` 方法对应 `Object.isExtensible`，返回一个布尔值，表示当前对象是否可扩展。

### Reflect.preventExtensions(target)

`Reflect.preventExtensions` 方法对应 `Object.preventExtensions` 方法，用来让一个对象变为不可扩展。

### Reflect.ownKeys()

`Reflect.ownKeys` 方法用于返回对象的所有属性，基本等于 `Object.getOwnPropertyNames` 与 `Object.getOwnPropertySymbols` 之和。

### 实例：使用 Proxy 实现观察者模式

观察者模式（Observer mode）指的是函数自动观察数据对象，一旦对象有变化，函数就会自动执行。

```javascript
const person = observable({
    name: '张三',
    age: 20
});

function print () {
    console.log(`${person.name}, ${person.age}`)
}

abserver(pirnt);
person.name = '李四';
// 李四，20
```

思路是 `observable` 函数返回一个原始对象的 Proxy 代理，拦截赋值操作，触发充当观察者的各个函数。

```javascript
const queuedObserable = new Set();

const abserve = fn => queuedObserable.add(fn);
const abservable = obj => new Proxy(obj, {set});

function set (target, name, value, receiver) {
    const result = Reflect.set(target, name, value, receiver);
    queuedObserable.forEach(abserver => abserver());
    return result;
}
```
