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

利用之前在 Promise 一章写的一个请求函数，我们来执行一个真实的异步任务。
```javascript
// 用 Promise 封装的一个请求函数
const getJson = function (url) {
    const promise = new Promise((resolve, reject) => {
        const handler = function () {
            if (this.readyState !== 4) {
                return;
            }

            if (this.status === 200) {
                let result = this.response;

                if (typeof result === 'string') {
                    result = JSON.parse(result);
                }

                resolve(result);
            } else {
                reject(new Error(this.statusText));
            }
        }
        const client = new XMLHttpRequest();
        client.open('get', url);
        client.onreadystatechange = handler;
        client.responseType = 'json';
        client.setRequestHeader('Accept', 'application/json');
        client.send();
    });

    return promise;
}

// Generator 函数
function* gen (url) {
    var result = yield getJson(url);
    console.log(result.bio);
}

// 执行
const g = gen('https://api.github.com/users/github');
const result = g.next(); // {value: Promise, done: false}

// 因为 getJson 函数返回的是一个 Promose 对象，所以要用 then 方法来调用下一个 next 方法。
result.value.then(data => {
    g.next(data);
});
// How people build software.
```

## Thunk 函数

Thunk 函数是自动执行 Generator 函数的一种方法。

### 参数的求值策略

求值策略：函数的参数该如何求值。

```javascript
var x = 1;

function f(m) {
    return m * 2;
}

// 这个表达式该如何求值呢？
f(x + 5)
```

一种意见是“传值调用”，即进入函数体之前，就计算参数的值，在将这个值传入函数。C 语言就是采取这种策略。

```javascript
f(x + 5)
// 传值调用时，等同于
f(6)
```

另一种意见是“传名调用”，即直接将表达式传入函数，只有用到它的时候求值。Haskkell 语言采取的就是这种策略。

```javascript
f(x + 5)
// 传名调用 等同于
(x +5) * 2
```

至于哪种比较好，众说纷纭。传值调用比较简单，但是对参数求值的时候，如果实际上没有用到这个参数，就有可能造成性能损失。所以有一些计算机学家倾向于“传名调用”，即在调用时求值。

### Thunk 函数的意义

编译器的“传名调用”的实现，往往是将参数放到一个临时函数里，再将这个临时函数传入函数体。这个临时函数就叫做 Thunk 函数。

```javascript
function f (m) {
    return m * 2
}

f(x + 5)

// 等同于

var thunk = function () {
    return x + 5
}

function f(thunk) {
    return thunk() * 2
}
```

上面的代码中，函数 f 的参数 `x + 5` 被一个函数替换了。凡是用到原参数的地方，对 Thunk 函数求值就好。

### JavaScript 语言的 Thunk 函数

在 JavaScript 语言中，Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数的作为参数的单参数函数。

```javascript
// 正常版本的 readFile （多参数版本）
fs.readFile(fileName, callback)

// Thunk 版本的 readFile （单参数版本）
var thunk = function (fileName) {
    return function (callback) {
        readFile(fileName, callback);
    }
}

var readFileThunk = thunk(fileName);
readFileThunk(callback);
```

上面代码中，`fs` 模块的 `readFile` 方法是一个多参数函数，两个参数分别为文件名和回调函数。经过转换器处理，它变成了一个单参数函数，只接受回调函数作为参数。这个单参数版本，就叫做 Thunk 函数。

任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式。

```javascript
// ES5 版本
var Thunk = function (fn) {
    return function () {
        var args = Array.prototype.slice.call(args);
        return function (callback) {
            args.push(callback);
            return fn.apply(this, args);
        }
    }
}

// ES6 版本
const Thunk = function (fn) {
    return function (...args) {
        return function (callback) {
            return fn(...args, callback);
        }
    }
}
```

下面是一个转换回调函数的例子。

```javascript
function f (a, callback) {
    callback(a);
}

const ft = Thunk(f);

ft(1)(console.log); // 1
```

### Thunkify 模块

生产环境的转换器，建议使用 Thunkify 模块。

首先是安装。

```javascript
$ npm i thunkify
```

使用释放如下。

```javascript
var thunkify = require('thunkify');
var fs = require('fs');

var read = thunkify(fs.readFile);
read('package.json')(function (err, data) {
    // ...
})
```

下面是 thunkify 模块的源码。

```javascript
function thunkify (fn) {
    return function () {
        var args = new Array(arguments.length);
        var ctx = this;

        for(var i =0, i < args.length, i++) {
            args[i] = arguments[i];
        }

        return function (done) {
            var called;

            args.push(function () {
                if (called) return;
                called = true;
                done.apply(null, arguments);
            });

            try {
                fn.apply(ctx, args);
            } catch (err) {
                done(err);
            }
        }
    }
}
```

它的源码主要多了一个检查机制，变量 `called` 确保回调函数只运行一次。这样的设计与下文的 Generator 函数相关。

### Generator 函数的流程管理

Thunk 函数可以用于 Generator 函数的自动流程管理。

Generator 函数可以自动执行。

```javascript
function* gen () {
    yield 1;
    yield 2;
    yield 3;
}

var g = gen();
var res = g.next();

while(!res.done) {
    console.log(res.value);
    res = g.next();
}
// 1
// 2
// 3
```

上面的代码中，Generator 函数回自动执行完所有步骤。

但是，这不适合异步操作。这时，Thunk 函数就能派上用场。

```javascript
var fs = require('fs'),
    thunkify = require('thunkify'),
    readFileThunkify = thunkify(fs.readFile);

var gen = function* () {
    var r1 = yield readFileThunkify('./package.json', 'utf-8');
    var r2 = yield readFileThunkify('./test.json', 'utf-8');
    console.log(JSON.parse(r2.name));
}
```

上面的代码中，`yiled` 命令用于将程序的执行权移除 Generator 函数，那么久需要一种方法，将执行权再交还给 Generator 函数。

这种方法就是 Thunk 函数，因为它可以在回调函数里，将执行权再交还给 Generator 函授。为了方便理解，我们先手动执行上面这个 Generator 函数。

```javascript
var g = gen();

// 开始第一次异步请求
var r1 = g.next();

// r1.value 属性后面跟的就是回调函数，用于交还执行权。
r1.value(function (err, data) {
    // 第一次请求成功后的回调处理
    if(err) return;
    console.log(JSON.parse(r1).version);

    // next() 交还执行权，进行第二次异步请求
    var r2 = g.next(data);

    // 进行第二次异步请求
    r2.value(function(err, data) {
        // 第二次请求成功后的回调处理
        if(err) return;
        console.log(JSON.parse(data));
        g.next(data);
    })
});
```

我们发现，Generator 函数的执行过程，其实是将同一个回调函数，反复传入 `next` 方法的 `value` 属性。这样我们可以用递归来自动完成这个过程。

### Thunk 函数的自动管理流程

下面是一个基于 Thunk 函数的自动 Generator 函数执行器。

```javascript
function run (fn) {
    var gen = fn();

    function next (err, data) {
        var result = gen.next(data);
        if (result.done) return;
        result.value(next);
    }

    next();
}

function* g () {
    // ...
}

run(g);
```

有了这个执行器，执行 GEenerator 函数就方便多了。不管内部有多少个异步操作，直接把 Generator 函数传入 run 函数即可。当前提前是每一个异步操作，都要是 Thunk 函数，也就是说，跟在 `yield` 命令后面的必须是 Thunk 函数。

```javascript
var g = function* () {
    var f1 = yield readFileThunk('fileA');
    var f2 = yield readFileThunk('fileB');
    // ...
    var fn = yield readFileThunk('fileN');
}

run(g);
```

Thunk 函数并不是 Generator 函数自动执行的唯一方案。因为自动执行的关键是，必须有一种机制，自动控制 Generator 函数的流程，接收和交还程序的执行权。回调函数可以做到这一点，Promise 对象也可以做到这一点。

## co 模块

co 模块是著名程序员 TJ Holowaychuk 于 2013 年 6 月发布的一个小工具，用于 Generator 函数的自动执行。

```javascript
var co = require('co');
var gen = function* () {
    var r1 = yield readfile('./dev.json', 'utf-8');
    var r2 = yield readfile('./test.json', 'utf-8');
    console.log(JSON.parse(r1));
    console.log(JSON.parse(r2));
}

co(gen).then(function () {
    console.log('Generator 函数执行完成');
}).catch(function (err) {
    console.log(err);
})
```

只要将 Generator 函数传入 co 函数，就会自动执行。

co 函数返回一个 Promise 对象，可以使用 `then` 方法添加回调函数。

### co 模块的原理

为什么 co 可以自动执行 Generator 函数？

前面说过，Generator 就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

两种方法可以做到这一点。

（1）回调函数。将异步操作包装成 Thunk 函数，在回调函数里面交回执行权。

（2）Promise 对象。将异步操作包装成 Promise 对象，用 `then` 方法交回执行权。

co 模块其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个模块。使用 co 的前提条件是，Generator 函数的 `yield` 命令后面，只能是 Thunk 函数或 Promise 对象。如果数组或对象的成员，全部都是 Promise 对象，也可以使用 co，详见后文的例子。

### 基于 Promise 对象的自动执行

首先，我们把 `fs` 模块的 `readFile` 方法包装成一个 Promise 对象。

```javascript
var fs = require('fs');
var readFile = function (fileName) {
    return new Primise(function (resolve, reject) {
        fs.readFile(fileName, 'utf-8', function(err, data) {
            if (err) return reject(err);
            resolve(data);
        });
    });
}

var gen = function* () {
    var f1 = yield readFile('file1');
    var f2 = yield readFile('file2');
}
```

手动执行上面的函数。

```javascript
var g = gen();
g.next().value.then(function (data) {
    g.next(data).value.then(function (data) {
        g.next(data);
    });
}).catch(function (err) {
    console.log(err);
})
```

手动执行其实就是使用 `then` 方法，层层添加回调函数。所以可以写出一个自动执行器。

```javascript
function run (fn) {
    var g = fn();

    function next (data) {
        var result = g.next(data);

        if (result.done) return result.value;
        result.value.then(function (data) {
            next(data);
        });
    }

    next();
}

run(gen);
```

### 处理并发的异步操作

co 支持并发的异步操作，即允许某些操作同时进行，等到它们全部完成，才进行下一步。

这时，要把并发的操作都放在数组或对象里面，跟在 `yield` 语句后面。

```javascript
// 数组的写法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 对象的写法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res);
}).catch(onerror);
```

