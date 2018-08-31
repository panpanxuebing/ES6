# Module 的加载实现

本章介绍如何在浏览器和 node 中加载 ES6 模块。

## 浏览器加载

### 传统方法

HTML 网页中，浏览器通过 `<script>` 标签加载 JavaScript 脚本。

```html
<!-- 页面嵌套的脚本 -->
<script type="application/javascript">
    // module code
</script> 

<!-- 外部脚本 -->
<script type="application/javascript" src="path/to/myMoudle.js"></script>
```

上面的代码中，由于浏览器脚本的默认语言是 JavaScript。因此 `type="application/javascript"` 可以省略。

默认情况下，浏览器同步加载 JavaScript 脚本，即渲染引擎遇到 `<script>` 标签就会停下来，等到执行完脚本，再继续往下渲染。如果是外部脚本，还必须加入脚本加载的时间。

如果脚本体积很大，下载和执行的时间就会很长，因此造成浏览器堵塞，用户会感觉到浏览器“卡死”了，没有任何响应。这显然是很不好的体验，所以浏览器允许脚本异步加载，下面就是两种异步加载的语法。

```html
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

上面代码中，`<script>` 标签打开 `defer` 或 `async` 属性，脚本就会异步加载。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的命令。

`defer` 与 `async` 的区别是: `defer` 是“渲染完再执行”，`async` 是“下载完就执行”。

### 加载规则

浏览器加载 ES6 模块，也使用 `<script>` 标签，但是要加入 `type="module"` 属性。

```html
<script type="module" src="./foo.js"></script>
```

## ES6 模块与 CommonJS 模块的差异

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

第二个差异是因为 CommonJS 加载的是一个对象（即 `module.exports` 属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口是一种静态定义，在代码静态解析阶段就会生成。

下面解释第一个差异。

CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。请看下面这个模块文件 `lib.js` 的例子。

```js
// lib.js
var count = 3;
function couter () {
    count++;
}

module.exports = {
    count,
    counter
}

// main.js
var mod = require('./lib.js');
console.log(mod.count); // 3
mod.counter();
console.log(count); // 3
```

上面的代码说明，`lib.js` 模块加载后，它的内部变化就影响不到输出的 `mod.count` 了。这是因为 `mod.count` 是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

```js
// lib.js
var count = 3;
function couter () {
    count++;
}

module.exports = {
    get count () {
		return count;
    },
    counter
}

$ node main.js
// 3
// 4 
```

ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令 `import`，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里去取值。所以，原始值变了，`import` 加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

```js
// lib.js
const let = 3;
const counter = function () {
    count++;
}

export { count, counter };

// mian.js
import { count, counter } from './lib.js';

console.log(count); // 3
counter();
console.log(count); // 4
```

上面的代码说明，ES6 模块输入的变量 `count` 是活的，完全反应其所在模块 `lib.js` 内部的变化。

由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它赋值会报错。

最后，`export` 通过接口，输出的是同一个值，不同的脚本加载这个接口，得到的是同样的实例。

```js
// lib.js
function C () {
    this.sum = 0;
    this.add = function () {
        this.sum++;
    }
    this.show = function () {
        console.log(this.sum);
    }
}

export let c = new C();
```

上面的脚本输出的是一个 `C` 的实例。不同的脚本加载这个模块，得到的都是同一个实例。

```js
// x.js
import {c} from './lib';
c.add();

// y.js
import {c} from './lib';
c.show();

// index.js
import './x';
import './y'; 
```

执行 `index.js`，输出的是 1。

```bash
$ babel-node index.js
// 1
```



