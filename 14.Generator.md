# Generator 函数的语法

## 简介

### 基本概念

Generator 函数是 ES6 提供的一种解决异步编程的方案。Generator 函数有多种理解角度。语法上，可以理解成一个状态机，封装了多个内部状态。

执行 Generator 函数会返回一个遍历器对象。也就是说，Generator 函数除了状态机，还是一个遍历器生成函数。它返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

形式上，Generator 函数是一个普通的函数，但有两个特征：一是，`function` 关键字与函数名之间有一个星号；二是，函数体内部使用 `yield` 关键字，定义不同的内部状态（`yield` 在英语里的意思就是“产出”）。

```javascript
function* helloWorldGenerator () {
    yield 'Hello';
    yield 'World';
    return 'ending';
}

const hw = helloWorldGenerator();
```

上面这个函数有个三状态：hello, world 和 return 语句。

Generator 函数的调用之后和普通函数也不相同，该函数并不执行，返回的是一个指向内部状态的指针对象，即遍历器对象（Iterator）。

下一步，必须调用遍历器对象的 `next` 方法，使得指针移向一下个状态。每次调用 `next` 方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个 `yield` 表达式（或 `return` 语句）。换言之，Generator 函数是分段执行的，`yield` 是暂停执行的标记，而 `next` 方法可以恢复执行。

## yield 表达式

遍历器对象的 `next` 方法的运行逻辑如下。

- 遇到 `yield` 表达式，就暂停执行后面的操作，并将紧跟在 `yield` 后面的那个表达式的值，作为返回的对象的 `value` 属性值。

- 下一次调用 `next` 方法时，在往下继续执行，直到遇到下一个 `yield` 表达式。

- 如果没有再遇到 `yield` 表达式，就一直运行到函数结束，直到 `return` 语句为止，并将 `return` 语句后面表达式的值，作为返回的对象的 `value` 属性值。

- 如果函数没有 `return` 语句，则返回的对象的 `value` 属性值为 `undefined`。


Generator 函数可以没有 `yield` 表达式，这样就变成了一个单纯的暂缓执行函数。

```javascript
function* f() {
    console.log('执行了');
}

var generator = f();

setTimeout(() => generator.next(), 1000)
// 执行了
```

另外，需要注意的是，`yield` 表达式只能用在 Generator 函数中，在其他地方都会报错。

```javascript
const arr = [1, [[2, 3], 4], [5, 6]];

const flat = function* (a) {
    a.forEach(item => {
        if (typeof item !== 'number') {
            yield* flat(item);
        } else {
            yield item; // 报错
        }
    });
}
```

上面的代码会报错，因为 `forEach` 方法的参数是一个普通函数，但在里面使用了 `yield` 表达式。这种情况下可以用 `for` 循环代替。

```javascript
const arr = [1, [[2, 3], 4], [5, 6]];

const flat = function* (a) {
    var len = a.length;

    for(let i = 0; i < len; i++) {
        const item = a[i];

        if (typeof item !== 'number') {
            yield* flat(item);
        } else {
            yield item;
        }
    }
}

for(let i of flat(arr)) {
    console.log(i);
}
// 1 2 3 4 5 6
```

另外，`yield` 表达式如果用在另一个表达式中，必须放在圆括号里面。

```javascript
function* demo () {
    console.log('hello' + (yield));
    console.log('hello' + (yield 123));
}
```

`yield` 表达式用作函数参数或者放在赋值表达式右边，不用加括号。

```javascript
functio* demo () {
    foo(yield 'a', yield 'b');
    let input = yield 'c';
}
```

## 与 Iterator 的关系

任意一个对象的 `Symbol.iterator` 方法，等于该对象的遍历器生成函数，调用该函数就会返回该对象的一个遍历器对象。

由于 Generator 函数就是遍历器生成器，因此可以把 Generator 赋值给对象的 `Symbol.iterator` 属性，从而使该对象具有 Iterator 接口。

```javascript
const myIterator = {};

myIterator[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
}

[...myIterator] // [1, 2, 3]
```

Generator 函数执行后，返回一个遍历器对象。该对象本身也具有 `Symbol.iterator` 属性，执行后返回自身。

```javascript
function* gen () {}

const g = gen();

g === g[Symbol.iterator] // true
```

## next 方法的参数

`yield` 表达式本身没有参数，或者总返回 `undefined`。`next` 方法可以带一个参数，该参数就会当做上一个 `yield` 表达式的返回值。

```javascript
function* f() {
    for(let i = 0; true; i++) {
        let reset = yield i;
        if(reset) { i = -1 }
    }
}
var g = f();

g.next(); // { value: 0, done: false }
g.next(); // { value: 1, done: false }
g.next(true) // // { value: 0, done: false }
```

我们可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数的行为。

再举一个栗子~

```javascript
function* foo (x) {
	const y = 2 * (yield (x + 1));
	const z = yield (y / 3);
	return (x + y + z);
}

const g = foo(5);

// x = 5, y = 6, z = 4
g.next(); // {value: 6, done: false}
g.next(3); // {value: 2, done: false}
g.next(4); // {value: 15, done}
```

V8 引擎直接忽略第一次使用 `next` 方法时的参数，只有从第二次使用 `next` 方法开始，参数才是有效的。从语义上讲，第一个 `next` 方法用来启动遍历器对象，所以不用带有参数。