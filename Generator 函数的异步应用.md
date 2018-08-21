# Generator 函数的异步应用

异步编程对 JavaScript 语言太重要，JavaScript 语言的执行环境是“单线程”的，如果没有异步编程，根本没法用。

# 传统方法

ES6 诞生之前，异步编程的方法，大概有以下四种：

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象

Generator 函数将 Javascript 异步编程带入了一个全新的阶段。

# 基本概念

## 回调函数

JavaScript 语言对异步编程的实现，就是回调函数。

```javascript
fs.readFile('/etc/passwd', 'utf-8', function (err, data) {
    if (err) throw err;
    console.log(data);
})
```

## Promise

回调函数的问题在于，多个回调函数嵌套时，产生强耦合，一个操作修改，可能会影响到它的上层或下层回调函数。这种情况称之为“回调地狱”。

```javascript
fs.readFile(fileA, 'utf-8', function (err, data) {
    fs.readFile(fileB, 'utf-8', function (err, data) {
        // ...
    })
})
```

Promise 对象就是为避免这种情况。它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。

```javascript
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function(data) {
    console.log(data.toString());
})
.then(function(data) {
    return readFile(fileB)
})
.then(function (data) {
    console.log(data.toString());
})
.catch(function(err) {
    console.log(err);
})
```

Primose 提供 `then` 方法加载回调函数，`catch` 方法捕捉执行过程中的错误。

Promise 最大的问题是代码冗余，一眼看过去一堆的 `then`， 原来的语义变得很不清楚。

那么，没有用更好的写法呢？答案是 Generator。

## Generator 函数

### 协程

传统的编程语言，早有异步编程的解决方案（其实是多任务的解决方案）。其中有一种叫做"协程"（coroutine），意思是多个线程互相协作，完成异步任务。

协程有点像函数，又有点像线程。它的运行流程大致如下。

协程有点像函数，又有点像线程。它的运行流程大致如下。

- 第一步，协程 `A` 开始执行。
- 第二步，协程 `A` 执行到一半，进入暂停，执行权转移到协程 `B`。
- 第三步，（一段时间后）协程 `B` 交还执行权。
- 第四步，协程 `A` 恢复执行。

```javascript
function* asyncJob () {
    // 其他代码
    var f = yield readFile(fileA);
    // 其他代码
}
```

协程遇到 `yield` 命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作，如果去除 `yield` 命令，简直一模一样。

### 协程的 Generator 函数实现

整个 `Generator` 函数就是一个封装的异步任务，或者说是异步任务的容器。异步操作需要暂停的地方，都用 `yield` 语句注明。Generator 函数的执行方法如下。

```javascript
function* gen(x) {
    var y = yield x + 2;
    return y;
}

var g = gen(1);
g.next(); // {value: 3, done: false}
g.next(); // {value: undefined, done: true} 
```

### Generator 函数的数据交换和错误处理

Generator 函数能暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外，它还有两个特性，使它可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。

`next` 返回值的 `value` 属性，是 Generator 函数向外输出数据；`next` 方法还可以接受参数，向 Generator 函数体内输入数据。

```javascript
function* gen (x) {
    var y = yield x + 1;
    return y;
}

var g = gen(1);
g.next(); // {value: 2, done: false}
g.next(2); // {value: 2, done: true}
```

Generator 函数内部还可以部署错误处理机制，捕获函数体外抛出的错误。

```javascript
function* gen (x) {
    try {
        var y = yield x + 1;
        return y;
    } catch (err) {
        console.log(err);
    }
}

var g = gen(1);
g.next();
g.throw('出错了');
// 出错了
```

上面的代码中，Generator 函数体外，使用指针对象的 `throw` 方法抛出的错误，可以被函数的`try...catch` 捕获。这意味着，出错的代码和处理错误的代码，实现的时间和空间的分离，这对于异步编程来说是很重要的。

### 异步任务的封装

