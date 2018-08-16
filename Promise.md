# Promise

## 含义

Promise 是一步编程的一种解决方案，比传统的解决方案--回调函数和事件--更为合理和强大。Promise 是一个对象，从它可以获取异步操作的消息。

`Promise` 对象有两个特点。

- 对象的状态不受外界影响。`Promise` 对象的异步操作有三种状态：`pending` （进行中），`fulfilled`（已成功）和 `rejected` （已失败）。只有异步操作的结果，可以决定当前是哪一种操作，其他任何操作都无法改变这个状态。

- 一旦状态改变，就不会再变，任何时候都会得到这个结果。

## 基本用法

ES6 规定，`Promise` 对象是一个构造函数，用来生成 `Promise` 实例。

```javascript
const promise = new Promise(function (resolve, reject) {
		// ...some code

		if (/* 异步操作成功 */) {
				reslove(value);
		} else {
				reject(error)
		}
});
```

`then` 方法可以接受两个回调函数作为参数。

```javascript
function timeout(ms) {
	return new Promise((resolve, reject) => {
		setTimeout(resolve, ms, 'done');
	});
}

timeout(100).then((value) => {
	console.log(value);
});
// 'done'
```

Promise 新建后会立即执行。

```javascript
let promise = new Promise((resolve, reject) => {
	console.log('Promise');
	resolve();
});

promise.then(() => {
	console.log('resolved');
})

console.log('Hi');

// 'Promise'
// 'Hi'
// 'resolved'
```

下面是异步加载图片的例子。

```javascript
function loadImageAsync(url) {
	return new Promise(function(resolve, reject) {
		const image = new Image();

		image.onload = function() {
			resolve(image);
		};

		image.onerror = function() {
			reject(new Error('Could not load image at ' + url));
		};

		image.src = url;
	});
}

loadImageAsync('1.jpg').then((image) => {
	document.getElementById('root').appendChild(image);
});
```

下面是一个用 `Promise` 对象实现 Ajax 操作的例子。

```javascript
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

getJson('include/js/mock-up/json/success.json').then((json) => {
	console.log(json.msg);
}, (error) => {
	console.log('出错了', error);
});
```

如果调用 `resolve` 函数和 `reject` 函数时带有参数，那么他们的参数会被传递给回调函数。`reject` 函数的参数通常 `Error` 对象的实例，表示抛出的错误；`resolve` 函数的参数除了正常的值以外，还可能是另一个 Promise 实例。

```javascript
const p1 = new Promise((resolve, reject) => {
	setTimeout(() => reject(new Error('Failed')), 3000);
});

const p2 = new Promise((resolve, reject) => {
	setTimeout(() => resolve(p1), 1000);
});

p2
	.then(result => console.log(result)
	.then(error => console.log(error));
// Error: Failed;
```

## Promise.prototype.then

Promise 实例具有 `then` 方法。它的作用是为 Promise 实例添加状态改变时的回调函数。`then` 的第一个参数是 `resolved` 状态的回调函数，第二个参数是 `rejected` 状态的回调函数。

`then` 方法返回的是一个新的 `Promise` 实例。因此可以采用链式的写法。

```javascript
getJSON('/post/1.json').then(
	post => getJSON(post.commentUrl)
).then(
	comments => console.log('resolved', comments),
	err => console.log('rejected', err)
)
```

## Promise.prototype.catch

`Promise.prototype.catch` 是 `.then(null, rejection)` 的别名，用于指定发生错误是的回调函数。

```javascript
getJSON('/post/1.json').then(
	// ...
).catch(
	// 处理 getJSON 和前一个回调函数运行时发生的错误。
	err => console.log('发生错误', err);
)
```

一般来说，不要在 `then` 方法里面定义 Reject 状态的回调函数（即第二个参数），而采用 `catch` 方法比较好。

```javascript
// bad
promise
	.then(function (data) {
		// success
	}, function (err) {
		// error
	});

// good
promise
	.then(function (data) {
		// success
	})
	.catch(function (err) {
		// error
	});
```

上面的两种写法中，建议使用第二种。理由是第二种可以捕获前面 `then` 方法中执行的错误，也更接近同步的写法( `try/catch` )。

跟传统的 `try/catch` 代码块不同的是，如果没有使用 `catch` 方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码。

```javascript
const someAsyncThing = function () {
	return new Promise(function (resolve, reject) {
		// 下面一个代码报错，因为 x 没有声明
		resolve(x + 1);
	})
}

someAsyncThing().then(
	() => console.log('everything is great')
);

console.log('I\'m still work out.');
// 报错
// I'm still work out.
```

一般情况下报错后，脚本会终止执行，但在 Promise 中的错误不会影响到 Promise 外部的代码。

## Promise.prototype.finally

类似于 `try/catch/finally`

## Promise.all

`Primise.all` 方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```javascript
const p = Promise.all([p1, p2, p3]);
```

上面的例子中， p 的状态由 p1，p2，p3 决定，分成两种情况。

- 只有 p1，p2，p3 的状态都为变为 `fulfilled`，p 的状态才会变为 `fulfilled`，此时 p1，p2，p3 的返回值组成一个数组，传递给 p 的回调函数。

- 只要有 p1，p2，p3 中的一个被 `rejected`，p 的状态就变为 `rejected`，此时第一个被 `rejected` 的实例的返回值，会传递给 p 的回调函数。

```javascript
const promises = [1, 2].map(id => {
	return Main.get('include/js/mock-up/json/success.json')
});

Promise.all(promises).then(
	result => console.log(result)
).catch(
	err => console.log(err)
);
```

下面是另一个栗子。

```javascript
const databasePromise = connectDatabase();

const booksPromise = databasePromise
	.then(findAllBooks);

const usePromise = databasePromise()
	.then(getCurrentUser);

Promise.all([
	booksPromise,
	usePromise
])
.then([books, user]) => pickTopRecomentations(books, users);
```

注意，如果作为参数的 Promise 实例，自己定义了 `catch` 方法，那么它一旦被 `rejected`，并不会触发 `Promise.all` 的 `catch` 方法。

```javascript
const p1 = new Promise((resolve, reject) => {
	resolve('hello');
})
	.then(result => result)
	.catch(err => err);

const p2 = new Promise((resolve, reject) => {
	throw new Error('报错了');
})
	.then(result => result)
	.catch(err => err);

Promise.all([p1, p2])
	.then(result => console.log(result))
	.catch(err => console.log(err));
// ["hello", Error: 报错了]	
```

如果 p2 中没有自己的 `catch` 方法，就会调用 `Promise.all()` 的 `catch` 方法。

## Promise.race()

`Promise.race` 方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

```javascript
const p = Promise.race([p1, p2, p3]);
```

上面的代码中，只要 p1，p2，p3 中有一个实例先改变状态，p 的状态就会跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。就像字面意思一样，race， 进程之间的赛跑，就看谁最先跑完。

```javascript
const p = Promise.race([
	fetch('/resource-that-may-take-a-while'),
	new Promise((resolve, reject) => {
		setTimeout(() => reject(new Error('request timeout')), 5000)
	})
]);

p
.then(console.log)
.catch(console.error);
```

上面代码中，如果 5 秒之内 `fetch` 方法无法返回结果，变量 p 的状态就会变为 `rejected`，从而触发 `catch` 方法指定的回调函数。

## Promise.resolve()

`Promise.resolve` 方法可以将现有对象转为 Promise 对象。

```javascript
const jsPromise = Promise.resolve($.ajax('/whatever.json'))
```

`Promise.resolve` 等价于下面的写法。

```javascript
Promise.resolve('foo');
// 等价于
new Promise(resolve => resolve('foo'))
```

`Promise.resolve` 的参数可以分为四种情况。

- 参数是一个 Promise 实例

如果参数是 Promise 实例，那么 `Promise.resolve` 将不做任何更改，原封不动的返回这个实例。

- 参数是一个 `thenable` 对象

`thenable` 对象指的是具有 `then` 方法的对象，比如下面这个对象。

```javascript
let thenable = {
	then: function (resolve, rejet) {
		resolve('foo')
	}
}
```

`Promise.resolve` 方法会将这个对象转为 Promise 对象，然后立即执行 `thenable` 对象的 `then` 方法。

```javascript
let thenable = {
	then: function (resolve, rejet) {
		resolve('foo')
	}
};

let p1 = Promise.resolve(thenable);
p1.then(
	value => console.log(value); // 'foo'
);
```

- 参数不是具有 `then` 方法的对象，或者根本不是对象。

如果参数是一个原始值，或者是一个不具有 `then` 方法的对象，则 `Promise.resolve` 方法返回一个新的 Promise 对象，状态为 `resolved`。这个参数也将会为回调函数的参数。

```javascript
const p = Promise.resolve({});

p.then(
	s => console.log(s) // 'Hello'
)
```

- 不带任何参数

`Promise.resolve` 允许不带参数，直接返回一个 `resolved` 状态的 Promise 对象。

## 应用

Generator 函数和 Promise 的结合

使用 Generator 函数管理流程，遇到异步操作的时候，通常返回一个 Promise 对象。

```javascript
function getFoo () {
	return new Promise((resolve, reject) => {
		resolve('foo');
	})
}

const g = function* () {
	try {
		const foo = yield getFoo();
		console.log(foo);
	} catch (err) {
		console.log(err)
	}
}

function run (generator) {
	const it = generator();

	const go = function (result) {
		if (result.done) return result.value;

		return result.value
			.then(
				value => go(it.next(value)),
				err => go(it.throw(error))
			)
	}

	go(it.next());
}

run(g);
```