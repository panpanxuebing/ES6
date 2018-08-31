# Class 的继承

## 简介

Class 可以通过 `extends` 关键字来继承。

```javascript
class Point {}

class ColorPoint extends Point {}
```

在类中，`super` 关键字表示父元素的构造函数。子类必须在 `constructor` 中调用 `super()` 方法，否则实例新建的时候回报错。这是因为子类自己的 `this` 对象，必须通过父类的构造函数来完成塑造，得到与父类同样的实例属性和方法，然后对其进行加工，加上子类自己的实例属性和方法。如果不调用 `super` 方法，子类就得不到 `this` 对象。

```javascript
class Point {
    constructor (width, height) {
        this.width = width;
        this.hieght = height;
    }

    toString () {
        return this.width + ' ' + this.hieght
    }
}

class ColorPoint extends Point {
    constructor (x, y, color) {
        super(x, y);  // 调用父类的constructor(width, height)
        this.color = color;
    }

    toString () {
        return this.color + ' ' + super.toString(); // 调用父类的toString()
    }
}
```

## Mixin 模式的实现

Mixin 指的是多个对象合成为一个对象，新对象拥有各个成员的接口。它的最简单实现如下。

```javascript
const a = {
    a: 'a'
};
const b = {
    b: 'b'
};
const c = {...a, ...b}; 
```

下面是一个更完备的实现。

```javascript
function mix (...mixins) {
    class Mix {}

    for (let mixin of mixins) {
        copyProperties(Mix.prototype, mixin); // 拷贝实例属性
        copyProperties(Min.prototype, Reflect.getPrototypeOf(mixin)); // 拷贝原型属性
    }
}

function copyProperties (target, source) {
    for (let key of Reflect.ownKeys(srouce)) {
        if (key !== 'constructor'
            && key !== 'prototype'
            && key !== 'name'
        ) {
            let desc = Reflect.getOwnPropertyDescriptor(srouce, key);
            Reflect.defineProperty(target, key, desc);
        }
    }
}
```

上面的 `mix` 函数，可以将多个对象合成为一个类。使用时，只要继承这个类即可。

```javascript
class combine extends mix(...mixins) {
    // ...
}
```