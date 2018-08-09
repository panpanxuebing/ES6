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