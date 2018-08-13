# Class 的基本语法

## 简介

Javascript 语言中，生成实例对象的传统方法是使用构造函数。

```javascript
function Point (x, y) {
    this.x = x;
    this.y = y;
}

Point.prototype.toString = function () {
    return '(' + this.x + ', ' + this.y + ')';
}

var p = new Point(1, 2);
```

上面这种写法和传统的面向对象的语言差异很大，很容易让新学习这门语言的程序员感到困惑。

ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。

基本上， ES6 的 `class` 可以看做只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 `class` 写法只是让对象原型的写法更加清晰、跟像面对对象编程的语法而已。

```javascript
class Point {
    constructor (x, y) {
        this.x = x;
        this.y = y;
    }

    toString () {
        return '(' + this.x + ', ' + this.y + ')';
    }
}
```

`constructor` 方法，就是构造方法，而 `this` 关键字则代表实例对象。即 ES5 的构造方法，对应 ES6 的 `Point` 类的构造方法。

定义类的方法时，不需要加上 `function` 这个关键字；另外方法之间不需要用逗号分隔，加了反而报错。

```javascript
class Point {}

typeof Point; // function
Point === Point.prototype.constructor; // true
```

上面的代码表示，类的数据类型就是函数，类的本身就指向构造函数。

使用的时候，用 `new` 命令，和构造函数一样。

类上的所有方法都定义在类的 `prototype` 属性上。在类的实例上调用方法，其实就是调用原型上的方法。

`prototype` 的 `constructor` 属性，指向“类”本身。这与 ES5 一致。

```javascript
Point.prototype.constructor === Point; // true
```

另外，类的内部所以定义的方法，都是不可枚举的。

```javascript
Object.keys(Point.prototype);
// []
Object.getOwnPropertyNames(Point.prototype);
// ['constructor', 'toString']
```

## 严格模式

类和模块的内部，默认j就是严格模式。

## constructor 方法

`consturctor` 方法是类的默认方法，通过 `new` 生成实例，自动调用该方法。一个类必须有 `constructor` 方法，如果没有显示定义，一个空的 `constructor` 方法会被默认添加。

`constructor` 方法默认返回实例对象，完全可以指定返回另外的对象。

```javascript
class Foo {
    constructor () {
        return Object.create(null);
    }
}

new Foo() instanceof Foo;
// false
```

类必须使用 `new` 调用，否则会报错。

### Class 表达式

与函数一样，类也可以使用表达式的形式定义。

```javascript
const MyClass = class me {
    getClassName () {
        return Me.name;
    }
}

// 这个类的名字是 Myclass 而不是 me, me 只能在代码内部使用,指代当前类。
let inst = new MyClass();
inst.getClassName(); // me
me.name; // ReferenceError: Me is not defined
```

采用 Class 表达式，可以写出立即执行的 class。

```javascript
let person = new class {
    constructor (name) {
        this.name = name;
    }

    sayName () {
        console.log(this.name);
    }
}('张三');

person.sayName();
// "张三"
```

## 不存在变量提升

类不存在变量提升（hoist），这一点与 ES5 完全不同。

## this 的指向

类的方法内部如果含有 `this`，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。

```javascript
class Logger {
    printName (name = 'world') {
        this.print(`Hello ${name}`);
    }

    print (text) {
        console.log(text);
    } 
}

const logger = new Logger();
const { printName } = logger;
printName();
// TypeError: Cannot read property 'print' of undefined
```

上面的代码中, `printName` 方法中的 `this`，默认指向 `Logger` 实例。但是，如果将这个方法取出来单独使用，`this` 会指向该方法运行时所在的环境，因为找不到 `print` 方法而报错。

一个比较简单的解决方法是，在构造函数中绑定 `this`。

```javascript
class Logger {
    constrctor (name) {
        this.printName = this.printName.bind(this);
    }
    
    // ...
}
```

另一种解决方法是使用箭头函数。

```javascript
class Logger {
    printName = (name = 'world') => {
        this.print(`Hello ${name}`);
    }

    // ...
}
```

还有一种解决方法是使用 `Proxy`，获取方法的时候，自动绑定 `this`。

```javascript
const cache = new WeakMap();
const selfish = function (target) {
    const handler = {
        get (target, key) {
            const value = Reflect.get(target, key);

            if(typeof value !== 'function') {
                return value;
            }

            if (!cache.has(value)) {
                cache.set(value, value.bind(target));
            }
            return cache.get(value);
        }
    }
    const proxy = new Proxy(target, handler);
    return proxy;
}

const logger = selfisg(new Logger());
```

## Class 的取值函数（getter）和存执函数（setter）

与 ES5 一样，在类的内部可以使用 `get` 和 `set` 关键字，对某个属性设置存值和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
    constructor () {}
    
    get prop () {
        return 'getter'
    }

    set prop (value) {
        console.log('setter: ' + value);
    }
}

let inst = new MyClass();
inst.prop = 123;
// setter: 123
inst.prop;
// 'getter'
```

存值和取值函数是设置在属性的 Descriptor 对象上的。

```javascript
var descriptor = Object.getOwnPropertyDescriptor(MyClass.prototype, 'prop');

'get' in descriptor; // true
'set' in descriptor; // true
```