# async 函数

## 含义

ES2017 标准引入了 async 函数，使得异步操作更为方便。

async 函数，简而言之就是 Generator 函数的语法糖。

看一个例子。

```js
const readFile = function (fileName) {
    return new Promise(function (resolve, reject) {
        fs.readFile(fileName, 'utf-8', function (err, data) {
            if (err) return reject(err);
            resolve(data);
        });
    });
}

const gen = function* () {
    const f1 = yield readFile('./test.json');
    const f2 = yield readFile('./dev.json');
    console.log(f1);
    console.log(f2);
}
```

写成 `async` 函数，就是下面这样。

```js
const asyncReadFile = async function () {
    const f1 = await readFile('./test.json');
    const f2 = await readFile('./dev.json');
    console.log(f1);
    console.log(f2);
}
```

一比较就会发现， `async` 函数只是将 Generator 函数的 `*` 号替换成 `async`，将 `yield` 替换成 `await`，仅此而已。

`async` 对 Generator 函数的改进有如下四点：

- 内置执行器
    
    Generator 函数的执行必须依靠执行器，所以才有了 `co` 模块，而 `async` 函数自带执行器。也就是说，`async` 函数的执行，与普通函数一杨，只需一行。

```js
asyncReadFile();
```

- 更好的语义

`async` 和 `await`，比起星号和 `yield`，语义更清楚了。`async` 表示函数里有异步操作，`await` 表示紧跟在后面的表达式需要等待结果。

- 更广的适用性
    `co` 模块规定，`yield` 命令后面只能是 Thunk 函数或者 Promise 对象，而 `async` 函数的 `await` 命令后面，可以使 Promise 对象和原始类型的值。

- 返回值是 Promise

    `async` 函数返回的是 Promise 对象，这比 Generator 函数返回值是 Iterator 对象方便多了。可以用 `then` 方法进行下一步操作。

进一步说，`async` 函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而 `await` 命令就是内部 `then` 命令的语法糖。

## 基本用法

`async` 函数返回一个 Promise 对象，可以使用 `then` 方法添加回调函数。当函数执行的时候，一旦遇到 `await` 就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

```js
const timeout = function (ms) {
    return new Promise(resolve => {
        setTimeout(resolve, ms);
    });
}

async function asyncPrint (content, ms) {
    console.log('please wait...');
    await timeout(ms);
    console.log(content);
}

asyncPrint('Hello World', 2000);
```

async 函数有多种使用形式。

```js
// 函数声明
async function foo () {}

// 函数表达式
const foo = async function () {}

// 对象的方法
const obj = { 
    async foo () {}
}

obj.foo().then(...)

// Class 的方法
class storage {
    async foo () {}
}

// 箭头函数
const foo = async () => {}
```

## 语法

`async` 函数的语法规则总体上比较简单，难点是错误处理机制。

### 返回 Promise 对象

`async` 函数返回一个 Promise 对象。

`async` 函数内部 `return` 语句返回的值，会成为 `then` 方法回调函数的参数。

`async` 函数内部抛出的错误，会导致返回的 Promise 对象为 `reject` 状态。抛出的错误对象会被 `catch` 方法的回调函数接收到。

```js
async function f () {
    throw new Error();
}

f().then(
    v => console.log(v),
    e => console.log(err)
)
// Error: 出错了
```

### Promise 对象的状态化

`async` 函数返回的 Promise 对象，必须等到内部所有的 `await` 命令后面的 Promise 对象执行完，才会发生状态的改变，除非遇到 `return` 语句或者抛出错误。也就是说，只有 `async` 函数内部的异步操作执行完，才会执行 `then` 方法指定的回调函数。

```js
async function getTitle (url) {
    let response = await fetch(url);
    let html = await response.text();
    return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}

getTitle('https://tc39.github.io/ecma262/').then(console.log);
```

上面代码中，函数 `getTitle` 内部有三个操作：抓取网页、取出文本、匹配页面标题。只有这三个操作全部完成，才会执行 `then` 方法里面的 `console.log`。

### await 命令

正常情况下，`await` 命令后面是一个 Promise 对象，如果不是，会被转成一个立即 `resolve` 的 Promise 对象。

```js
async function foo () {
    return await 123;
}
foo().then(console.log);

```

`await` 命令后面的 Promise 对象如果变为 `reject` 状态，则 `reject` 的参数会被 `catch` 方法的回调函数接收到。

```js
async function f () {
    await Promise.reject('出错了');
};

f()
.then(v => console.log(v))
.catch(err => console.log(err))
// 出错了
```

注意，上面的代码，`await` 前面并没有 `return`，但是 `reject` 方法依然会传入 `catch` 方法的回调函数。

只要一个 `await` 语句后面的 Promise 变为 `reject`，那么整个 `async` 函数都会中断执行。

```js
async function foo () {
    await Promise.reject('出错了');
    return await Promise.resolve('hello world'); // 不会执行
}
```

有时，我们希望前一个异步操作失败，也不要影响到后面的异步操作。这是可以将第一个 `await` 放在 `try...catch` 结构里面，这样不管这个异步操作是否成功，第二个 `await` 都会执行。

```js
async function foo () {
    try {
        await Promise.reject('出错了');
    } catch (err) {}
    return await Promise.resolve('hello world'); // 不会执行
}

foo()
.then(console.log)
.catch(console.log);
// hello world
```

另一种方法是 `await` 后面的 Promise 对象再跟一个 `catch` 方法,处理前面可能出现的错误。

```js
async function () {
    await Promise.reject('出错了')
        .catch(e => console.log(e))

    return await Promise.resolve('hello world');
}
// 出错了
// hello world
```

### 使用注意点

第一点，前面已经说过，`await` 命令后面的 Promise 对象，运行结果可能是 `rejected`，所以最好把 `await` 命令放在 `try...catch` 代码块中。

```js
async function myFunc () {
    try {
        await somethingThatReturnsAPromise();
    } catch (err) {
        console.log(err);
    }
}

// 另一种写法
async function myFunc () {
    await somethingThatReturnsAPromise()
        .catch(err => console.log(err));
}
```

第二点，多个 `await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。

```js
// bad 这种写法只有在 getFoo 执行完之后，才会执行 getBar，比较耗时。
let foo = await getFoo();
let bar = await getBar();

// good
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = barFoo();
let foo = await fooPromise;
let bar = await barPromise;
```

第三点，`await` 命令只能用在 `async` 函数中，用普通函数中会报错。

```js
async function doFuc (db) {
    let docs = [{}, {}, {}];

    // 报错
    docs.forEach(function (doc) {
        await db.post(doc);
    })
}

// 改为
function doFuc (db) {
    let docs = [{}, {}, {}];

    // 可能得到错误结果:因为这时三个 `db.post` 操作将是并发执行，即同时执行。正确的方法是改为 for 循环。
    docs.forEach(async function (doc) {
        await db.post(doc);
    })
}

// 正确的写法
async function doFuc (db) {
    let docs = [{}, {}, {}];

    for(let doc of docs) {
        await db.post(doc);
    }
}
```

如果确实希望多个请求并发执行，可以使用 `Promise.all` 方法。

## async 函数的实现原理

`async` 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。

```js
async function fn (args) {
    // ...
}

// 等同于
function fn (args) {
    return spawn(function* () {
        // ...
    })
}
```

所有的 `async` 函数都可以写成上面的第二种形式，其中的 `spawn` 函数就是自动执行器。

下面给出 `spawn` 函数的实现，基本就是前文自动执行器的翻版。

```js
function spawn (genF) {
    return new Promise(function(resolve, reject) {
        const gen = genF();

        function step (nextF) {
            let next;

            try {
                next = nextF();
            } catch (err) {
                return reject(err);
            }

            if (next.done) {
                return resolve(next.value);
            }

            Promise.resolve(next.value).then(
                v => step(function () { return gen.next(v); }),
                e => step(function () { return gen.throw(e) })
            )
        }

        step(function () {
            return gen.next(undefined);
        })
    });
}
```

## 与其他异步处理方法的比较

实际开发中，经常遇到一组异步操作，需要按照顺序完成。比如，依次远程读取一组 URL，然后按照读取的顺序输出结果。

Promise 的写法如下。

```js
function loginOrder (urls) {
    // 远程读取所有 url
    const textPromises = urls.map(url => {
        return fetch(url)
            .then(response => response.text());
    });

    // 按次序输出
    textPromise.reduce((chain, textPromise) => {
        return chian.then(() => textPromise)
            .then(text => console.log(text))
    }, Promise.resolve());
}
```

async 写法如下。

```js
async function loginOrder (urls) {
    for(let url of urls) {
        const response = await fetch(url);
        const text = await response.text();
        const title = text.match(/<title>([\s\S]+)<\/title>/i)[1]; 
        console.log(title);
        
    }
}
```

缺点是所有操作都是继发的，效率太差。下面的写法能解决这个问题。

```js
asfnc function loginOrder (urls) {
    // 并发读取远程 URL
    const textPromises = urls.map(async url => {
        const response = await fetch(url);
        const text = await response.text();
        const title = text.match(/<title>([\s\S]+)<\/title>/i)[1];
        return title;
    });

    // 用 for 循环按次序输出
    for(let textPromise of Promises) {
        const result = await textPromise;
        console.log(result);
    }
    
    // 或者用 Promise.all

    const result = Promise.all(textPromise);
    console.log(result);
}
```