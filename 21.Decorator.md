# 修饰器

## 类的修饰

修饰器（Decorator）函数，用来修改类的行为。

```js
@testable
class MyTestableClass {}

function testable (target) {
    target.isTestable = true;
}

MyTestableClass.isTestable; // true
```

上面代码中，`@testable` 就是一个修饰器，它修饰了 `MyTestableClass` 这个类的行为，为它加上静态属性 `isTestable`。`testable` 函数的参数 `target` 是 `MyTestableClass` 类本身。

```js
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

如果觉得一个参数不够用，可以在修饰器外再包一层函数。

```js
function testable (isTestable) {
    return function (target) {
        target.isTestable = isTestable;
    }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestale; // true

@testable(false)
class MyTestableClass{}
MyTestableClass.isTestable; // false
```

注意，修饰器对类的行为的改变，是在代码编译是发生的，而不是在运行时。这意味着，修饰器能在编译时运行代码。也就是说，修饰器本质是编译时执行的函数。

通过修饰器添加实例属性。

```js
function testable (target) {
    target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable; // true
```

实际开发中，React 和 Redux 库结合时，常常需要写成这样。

```js
class MyReactComponent extends React.Component {}
export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);

// 用修饰器，可以改写成这样
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```

## 方法的修饰

修饰器不仅可以修饰类，还可以修饰类的属性。

```js
class Person {
    @readonly
    name () { return `${this.first} ${this.last}` }
}
```

上面的代码中，修饰器 `readonly` 用来修饰“类”的 `name` 方法。

修饰器函数 `readonly` 一共接受三个参数。

```js
function readonly (target, name, descriptor) {
    // descriptor 对象原来的值如下
    // {
    //     value: specifiedFunction,
    //     enumerable: false,
    //     configurable: true,
    //     writable: true
    // }
    descriptor.writable = false;
    return descriptor;
}
```

下面是另一个例子，修改属性描述对象的 `enumerable` 属性，使得该属性可遍历（原型中的属性默认不可遍历）。

```js
class Person {
    @nonenumerable
    get kidCount () { return this.children.length }
}

function nonenumerable (target, name, descriptor) {
    descriptor.enumerable = true;
    return descriptor;
}

Object.keys(Person.prototype); // ['kidCount']
```

上面的代码，`@log` 修饰器的作用就是在执行原始的操作之前，执行一次 `console.log`，从而达到输出日志的作用。

如果同一个方法有多个修饰器，会像剥洋葱一样，先从外到内进入，然后由内到外执行。

```js
function dec (id) {
    console.log(id);
    return (target, name, descriptor) => { console.log(id) }
}

class Example {
    @dec(1)
    @dec(2)
    method () {}
}

const e = new Example();
e.method();

// 1
// 2
// 2
// 1
```

上面代码中，外层修饰器 `@dec(1)` 先进入，但是内层修饰器 `@dec(2)` 先执行。

## 为什么修饰器不能用于函数

修饰器只能用于类和类的方法，不能用于函数，因为函数存在变量提升。

```js
var counter = 0;
var add = function () {
    counter++;
}

@add
function fun () {}

// 由于函数存在提升，实际执行的是
@add
function () {}

var counter = 0;
var add = function () {
    counter++;
}
```

如果一定要修饰函数，可以采用高阶函数的形式执行。

```js
function doSomething (name) {
    console.log('Hello ', name);
}

function logDecorator (wrapped) {
    return function () {
        console.log('Starting');
        var result  = wrapped(...arguments);
        console.log('Finished');
        return result;
    }
}

var wrapped = logDecorator(doSomething)('world');
// Starting
// Hello  world
// Finished
```

## code-decarator.js

`code-decarator` 是一个第三方模块，提供了几个常见的修饰器，通过它可以更好的理解修饰器。

- @autobind

`autobild` 修饰器使得方法中的 `this` 对象，绑定原始对象。

```js
import { autobind } from 'core-decorators';

class Person {
    @autobind
    getPerson () {
        return this;
    }
}

let p = new Person();
let getPerson = p.getPerson;

console.log(getPerson() === p)
// true
```

- @readonly

`readonly` 修饰器使得属性或方法不可写。

- @override

`override` 修饰器检查子类的方法，是否正确覆盖了父类的同名方法，如果不正确会报错。

## Mixin

在修饰器的基础上，可以实现 `Mixin` 模式。所谓 `Mixin` 模式，就是对象继承的一种替代方案，中文译为“混入”（mix in），意为在一个对象之中混入另外一个对象的方法。

```js
const Foo = {
    foo () {
        console.log('foo');
    }
}

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo();
```

上面的代码是“混入”模式的一个简单实现。

我们可以将 Mixin 写成一个修饰器。

```js
// mixin.js
export function mixins (...list) {
    return function (target) {
        Object.assign(target.prototype, ...list);
    }
}

// index.js
import { mixin } from './mixin.js';

const Foo = {
    foo () { console.log('foo'); }
}

@mixin(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo();
```