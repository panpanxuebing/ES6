# Proxy

## 概述

Proxy 用于修改某些操作的默认行为，属于一种“元编程”，即对编程语言进行编程。

Proxy 可以理解为，在目标对象前设置一层“拦截”，外界对该对象的访问,都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进程过滤和改写。

```javascript
var obj = new Proxy({}, {
    get: function (target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver);
    },
    set: function (target, key, value, receiver) {
        console.log(`setting ${key}!`);
        return Reflect.set(target, key, value, receiver);
    }
})

obj.count = 1;
// setting count!
obj.count++;
// getting count!
// setting count!
// 1
```

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```javascript
var proxy = new Proxy(target, handler);
```

其中，`new Proxy` 表示生成一个 `Proxy` 实例，`target` 参数表示要拦截的目标对象，`handler` 参数也是对象，用来制定拦截行为。Proxy 对象的所有用法,都是上面这种形式，不同的是 `handler` 参数的写法。

```javascript
var proxy = new Proxy({}, {
    get: function (target, property) {
        return 35;
    }
});

proxy.time; // 35
proxy.name; // 35
```

同一个拦截器函数，可以设置拦截多个操作。

```javascript
var handler = {
    get: function (target, name) {
        if (name === 'prototype') {
            return Object.prototype
        }
        return 'Hello ' + name
    },

    apply: function (target, thisBinding, args) {
        return args[0]
    },

    construct: function (traget, args) {
        return {value: args[1]}
    }
}

var fproxy = new Proxy(function (x, y) {
    return x + y
}, handler);

fproxy(1, 2); // 1
new fproxy(1, 2); // {value: 2}
fproxy.prototype === Object.prototype; // true
fproxy.foo // 'Hello foo'
```

## Proxy 实例的方法

### get()

`get` 方法用于某个属性的读取操作。

get(target, propKey, receiver)

- target ：目标对象
- propkey ：属性名
- receiver ：proxy 实例本身（可选） 

```javascript
var person = {
    name: '张三'
};

var proxy = new Proxy(person, {
    get: function (target, property) {
        if (property in target) {
            return target[property]
        } else {
            throw new ReferenceError(`Property '${property}' dose not exist.`)
        }
    }
});

proxy.name; // '张三'
proxy.age; // 报错
```
下面的例子使用 `get` 拦截，实现数组读取负数的索引。

```javascript
function createArray (...elements) {
    let handler = {
        get: function (target, key, receiver) {
            let index = Number(key);
            if (index < 0) {
                key = String(target.length + index);
            }
            return Reflect.get(target, key, receiver);
        }
    };

    return new Proxy(elements, handler);
}

let arr = createArray('a', 'b', 'c');
arr[-1];
```

利用 Proxy，可以将读取属性的操作（`get`），转变为执行某个操作，从而实现属性的链式操作。

```javascript
const pipe = ({
    return (value) => {
        let funcStack = [];
        const proxy = new Proxy({}, {
            get: function (target, fname) {
                if (fname === 'get') {
                    return funcStack.reduce((val, func) => {
                        return func(val)
                    }, value)
                }
                funcStack.push(window[fname]);
                return proxy;
            }
        });
        return proxy;
    }
})

const doule = n => n * 2;
const pow = n => n * n;
const reverseInt = n => n.toString().split("").reverse().join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63
```

下面的例子利用 `get` 拦截，实现一个生成各种 DOM 节点的通用函数 `dom`。

```javascript
const dom = new Proxy({}, {
    get (target, property) {
        return function (attr = {}, ...children) {
            let el = document.createElement(property);

            for(let prop of Object.keys(attr)) {
                el.setAttribute(prop, attr[prop]);
            }

            for(let child of children) {
                if(typeof child === 'string') {
                    child = document.createTextNode(child);
                }

                el.appendChild(child);
            }

            return el;
        }
    }
});

const el = dom.div({},
    'Hello, my name is ',
    dom.a({href: '//example'}, 'Mark'),
    '. I like',
    dom.ul({}, 
        dom.li({}, 'The web'),
        dom.li({}, 'Food'),
        dom.li({}, '...actually that \'s it')
    )
);
```

如果一个属性不可配置（configuration）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。

```javascript
const target = Object.defineProperties({}, {
    foo: {
        value: 123,
        writable: true,
        configuration: true
    }
});

const proxy = new Proxy(target, {
    get (property, value) {
        return 'abc'
    }
});

proxy.foo; // 报错
```

### set()

`set` 方法用来拦截某个属性的赋值操作，可以接受四个参数，依次是目标对象、属性名、属性值和 Proxy 对象本身，最后一个参数可选。

```javascript
let validator = {
    set (target, property, value) {
        if (property === 'age') {
            if (!Number.isInteger(value)) {
                throw new Error('The age is not a integer.')
            }
            if (value > 200) {
                throw new RangeError('The age seems invalid');
            }
        }
        
        target[property] = value;
    }
};

const person = new Proxy({}, validator);
person.age = 100;

person.age; // 100
person.age = 'young'; // 报错
person.age = 200; // 报错
```

有时我们在对象上面设置内部属性，属性名的第一个字符使用下划线，表示这些属性不应该被外部使用。结合 `get` 和 `set` 方法，就可以实现防止这些内部属性被外部改写。

```javascript
const handler = {
    get (target, key, vakue) {
        invariant(key, 'get');
        return target[key];
    },
    set (target, key, value) {
        invariant(key, 'set');
        target[key] = value;
        // return true
    }
}

const invariant = function (key, action) {
    if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" porperty.`);
    }
}

const target = {};
const proxy = new Proxy(target, handler);

proxy._prop;
// Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c';
// Error: Invalid attempt to set private "_prop" property
```

注意，如果目标对象自身的某个属性，不可写且不可配置，那么 `set` 方法将不起作用。

### apply()

`apply` 方法拦截函数的调用、`call` 和 `apply` 操作。

`apply` 方法接受三个参数，分别是目标对象、目标对象的上下文对象（`this`）和目标对象的参数数组。

```javascript
var handler = {
    apply (target, ctx, args) {
        return Refelct.apply(...arguments);
    }
}
```

下面是一个栗子。

```javascript
const target = function () { return 'I am the target' };
var handler = {
    apply () {
        return 'I am the proxy'
    }
};

const p = new Proxy(target, handler);
p();
// "I am the proxy."
```

上面的代码中，变量 `p` 是 Proxy 的实例，当它作为函数调用时，就会被 `apply` 方法拦截，返回一个字符串。

下面是另外一个栗子。

```javascript
const handler = {
    apply (target, ctx, args) {
        return Reflect.apply(...arguments) * 2;
    }
}

const sum = function (left, right) {
    return left + right;
}

var proxy = new Proxy(sum, handler);

proxy(1, 3); // 8
proxy.apply(null, [1, 3]); // 8
proxy.call(null, 1, 3); // 8
```

另外，直接调用 `Reflect.apply` 方法，也会被拦截。

```javascript
Reflect.apply(proxy, null, [1, 3]); // 8
```

### has()

`has` 方法用来拦截 `HasProperty` 操作，即判断对象是否拥有某个属性时，这个方法会生效。典型的就是 `in` 操作符。

`has` 方法接受连个参数，分别是目标对象、需查询的属性名。

```javascript
const handler = {
    has (target, prop) {
        if (prop[0] === '_') {
            return false;
        }
        return prop in target;
    }
}

const target = {foo: 123, _foo: 'abc'}

var proxy = new Proxy(target, handler);

'foo' in proxy; // true
'_foo' in proxy; // false
```

如果原对象不可配置或者禁止扩展，这是 `has` 拦截会报错。

```javascript
var obj = {a: 10};
Object.preventExtensions(obj);

var p = new Proxy(obj, {
    has (target, prop) {
        return false;
    } 
});

'a' in p; // 报错
```

注意：`has` 方法拦截的是 `HasProperty` 操作，而不是 `hasOwnProperty` 操作，所以 `has` 方法不能判断一个属性是对象自身的属性，还是继承的属性。

另外，虽然 `for...in` 中也用到了 `in` 操作符，但是 `has` 拦截对 `for...in` 不起作用。

```javascript
let stu1 = {name: '张三', score: 59};
let stu2 = {name: '李四', score: 99};

let handler = {
    has (target, prop, value) {
        if (prop === 'score' && target[prop] < 60) {
            console.log(`${target.name}不及格`);
        }
        return prop in target;
    }
}

const oproxy1 = new Proxy(stu1, handler);
const oproxy2 = new Proxy(stu2, handler);

'score' in oproxy1;
// '张三不及格'
// true

'score' in oproxy2;
// true

for(let i in oproxy1) {
    console.log(oproxy1[i]);
}
// 张三
// 59
```

### construct

`construct` 方法用于拦截 `new` 命令，下面是拦截对象的写法。

```javascript
const handler = {
    construct (target, args, newTarget) {
        return new target(...args);
    }
}
```

`construct` 方法可以接受两个参数：

- `target`：目标对象

- `args`：构造函数的参数对象

- `newTarget`：创造实例对象时，`new` 命令作用的构造函数（下面例子的 `p` ）。

```javascript
const p = new Proxy(function () {}, {
    construct (target, args) {
        console.log('called: ' + args.join(','));
        return {value: args[0] * 10}
    }
});

(new p(1)).value;
// 'called: 1'
// 10
```

`construct` 必须返回一个对象，否则会报错。

### deleteProperty()

`deleteProperty` 用于拦截 `delete` 操作，如果这个这个方法抛出错误或者返回 `false`，当前属性就无法被删除。

```javascript
const handler = {
    deleteProperty (target, prop) {
        invariant(prop, 'delete');
        return true;
    }
}

const invariant = function (key, action) {
    if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" porperty.`);
    }
}

var target = {_prop: 'foo'};

var proxy = new Proxy(target, handler);

delete proxy._prop;
// Error: Invalid attempt to delete private "_prop" property
```

上面的代码中，`deleteProperty` 方法拦截了 `delete` 操作符，删除第一个字符为下划线的属性就会报错。

注意，目标对象自身的不可配置（configurable）的属性，不能被deleteProperty方法删除，否则报错。

### defineProperty()

defineProperty方法拦截了Object.defineProperty操作。

```javascript
var handler = {
    defineProperty(target, key, descriptor) {
        return false;
    }
}

var target = {};
var proxy = new Proxy(target, handler);

proxy.foo = 'bar';
// 不会生效
```

### getOwnPropertyDescriptor()

`getOwnPropertyDescriptor` 方法拦截 `Object.getOwnPropertyDescriptor()`，返回一个属性描述对象或者 `undefined`。

### getPrototypeOf()

`getPrototypeOf` 方法主要用来拦截获取对象的原型。具体有以下操作：

- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceOf`

```javascript
const proto = {};
const p = new Proxy({}, {
    getPrototypeOf(target) {
        return proto;
    }
})

Object.getPrototypeOf(p) === proto; // true
```

### isExtensible()

`isExtensible` 方法拦截 `Object.isExtensible` 操作。

### ownKeys()

`ownKey` 方法用于拦截对自身属性的读取操作。具体有以下操作：

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `for...in` 循环

最后必须返回数组或者对象，返回的是对象也会默认转换为数组。

下面是拦截 `Object.keys()` 的例子。

```javascript
let target = {
    a: 1,
    b: 2,
    c: 3
}

let handler = {
    ownKeys (target) {
        return ['a']
    }
}

let proxy = new Proxy(target, handler);

Object.keys(proxy);
// ['a']
```

注意，使用 `Object.keys()` 方法时，有三类属性会被 `ownKeys` 方法自动过滤掉，不会返回：

- 目标对象上不存在的属性
- 属性名为 Symbol 值
- 不可遍历的属性

```javascript
let target = {
    a: 1,
    b: 2,
    c: 3,
    [Symbol.for('secret')]: 4
}

Object.defineProperty(target, 'key', {
    value: 'static',
    enumerable: false,
    writable: true,
    configurable: true
});

let handler = {
    ownKeys (target) {
        return ['a', 'd', Symbol.for('secret'), 'key']
    }
}

const proxy = new Proxy(target, handler);

Object.keys(proxy);
// ['a']
```

上面的代码中， `ownkeys` 方法中，显式返回不存在属性、Symbol 值，不可遍历属性，结果都被过滤掉了。

`ownkeys` 返回的数组成员，只能是字符串或者 Symbol 值。如果有其他类型的值，或者返回的根本不是数组，就会报错。

如果目标对象中包含不可配置的属性，则该属性必须被 `ownKey` 返回，否则会报错。

```javascript
let obj = {};

Object.defineProperty(obj, 'a', {
    configurable: false,
    enumerable: true,
    writable: true,
    value: 10
});

const proxy = new Proxy(obj, {
    ownKeys(target) {
        return ['b']
    } 
});

Object.keys(proxy); // 报错
```

### preventExtensions()

### setPrototypeOf()

## Proxy.revocable()

`Proxy.revocable` 方法返回一个可取消的 Proxy 实例。

`Proxy.revocable` 方法返回一个对象，该对象时的 `proxy` 属性是 Proxy 的实例，`revoke` 属性是一个函数，可以取消 `Proxy` 实例。

```javascript
const target = {};
const handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo; // 123

revoke();
proxy.foo; // TypeError: Revoked
```

`Proxy.revocable` 的一个使用场景是，目标对象只能通过代理访问，一旦访问结束，就收回代理权，不允许访问。

## this 问题

在 Proxy 代理的情况下，目标对象内部的 `this` 关键字会指向 Proxy 代理。

```javascript
const _name = new WeakMap();

class Person {
    constructor (name) {
        _name.set(this, name);
    }

    get name () {
        return _name.get(this)
    }
}
const jane = new Person('Jane');
jane.name; // 'Jane'

const proxy = new Proxy(jane, {});
proxy.name // undefined
```

这时需要 `this` 绑定原始对象，就可以解决这个问题。