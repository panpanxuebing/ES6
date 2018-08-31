# Moudle 的语法

## export 命令

`export` 用于规定模块的对外接口，`import` 用于输入其他模块的功能。

一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，必须使用 `export` 关键字输出变量。

```js
// profile.js
// bad
export var firstName = 'Michael';
export var lastName = 'Jaskson';
export var year = 1958;

// good
var firstName = 'Michael';
var lastName = 'Jaskson';
var year = 1958;

export {fistName, lastName, year};
```

需要特别注意的是，`export` 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```js
// 报错
export 1;

// 报错
var m = 1;
export m;
```

上面两种写法报错，是因为没有提供对外的接口。第一种方法直接输出 1，第二种写法通过变量 m，还是直接输出 1。1 只是一个值，不是接口。正确的写法是下面这样。

```js
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

上面的写法都是正确的，规定了对外的接口 m。其他脚本可以通过这个接口，取到它的值 1。它们的实质是，在接口名和模块内部变量之间，建立了一一对应关系。

另外，`export` 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 1000);
```

上面代码输出变量 `foo`，值为 `bar`，500 毫秒之后变成 `baz`。

最后，export命令可以出现在模块的任何位置，只要处于模块顶层就可以。

## import 命令

使用 `export` 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 `import` 命令加载这个模块。

```js
// main.js
import {firstName, lastName} from './profile.js';
```

如果想为输入的变量重新取一个名字，`import` 命令要使用 `as` 关键字，将输入的变量重命名。

```js
import {lastName as surName} from './profile.js';
```

`import` 命令输入的变量都是只读的，因为它的本质是输入接口。也就是说，不允许在加载模块的脚本里面，改写接口。

```js
import {a} from './xxx.js';
a = {};
// Syntax Error: 'a' is read-only.
```

但是，如果 `a` 是一个对象，改写 `a` 的属性是允许的，并且其他模块也可以读到改写后的值。这种写法很难差错，建议凡是量输入的变量，轻易不要改变它的属性。

注意，`import` 命令具有提升效果，会提升到整个模块的头部，首先执行。

```js
foo();

import {foo} from 'my_module';
```

上面的代码不会报错，因为 `import` 的执行早于 foo 的调用。这种行为的本质是，`import` 命令是编译阶段执行的，在代码运行之前。

由于 `import` 是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构。

```js
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let moudle = 'my_module';
import {foo} from module;

// 报错
if (x === 1) {
    import {foo} from 'module1'
} else {
    import {foo} from 'module2';
}
```

上面三种写法都会报错，因为它们用到了表达式、变量和 `if` 结构。在静态分析阶段，这些语法都是没法得到值的。

如果多次重复执行同一句 `import` 语句，那么只会执行一次，而不会执行多次。

## 模块的整体加载

除了指定加载某个输出值，还可以使用整体加载，即用星号（`*`）指定一个对象，所有输出值都加载在这个对象上面。

```js
// circle.js
export function area(radius) {
    return Math.PI * radius * radius;
}

export function circumference(radius) {
    return 2 * Math.PI * radius;
}

// main.js
import * as circle from './circle.js';

console.log(circle.area(4));
onsole.log(circle.circumference(16));
```

注意，模块整体加载所在的那个对象（上例是 `circle` ），应该是可以静态分析的，所以不允许运行时改变。下面的写法都是不允许的。

```js
import * as circle from './circle';

// 下面两行被是不被允许的
circle.foo = '123';
circle.area = function () {}
```

## export default 命令

`export default` 命令，可以为模块指定默认输出。

```js
// export-default.js
export default function () {
    console.log('foo');
}

// import-default.js
import customName from './export-default.js';

customName(); // 'foo'
```

其他模块加载该模块时，`import` 命令可以为该匿名函数指定任意名字。而且这时 `import` 命令后面，不使用大括号。

本质上，`export default` 就是输出一个叫做 `default` 的变量或方法，然后系统允许你为它取任意名字。

正是因为 `export default` 命令其实只是输出一个叫做 `default` 的变量，所以它后面不能跟变量声明语句。

```js
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;
```

如果想在一条 `import` 语句中，同时输入默认方法和其他接口，可以写成下面这样。

```js
import _, { each, each as forEach } from 'lodash';
```

## export 与 import 的复合写法

如果在一个模块之中，先输入后输出同一个模块，`import` 语句可以与 `export` 语句写在一起。

```js
export {foo, bar} from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
```

但是需要注意的是，写成一行后，`foo` 和 `bar` 模块实际并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用 `foo` 和 `bar`。

模块的接口改名和整体输出，也可以采用这种写法。

```js
// 接口改名
export {foo as myFoo} from 'my_moudle';

// 整体输出
export * from 'my_module';
```

## 模块的继承

模块之间也可以继承。

假设有一个 `circleplus` 模块，继承了 `circle` 模块。

```js
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function (x) {
    return Math.exp(x);
}
```
上面的代码中的 `export *` ，表示再输出 `circle` 模块的所有属性和方法。注意，`export *` 会忽略 `circle` 模块的 `default` 方法。然后，上面代码又输出了自定义的 `e` 变量和默认方法。

## 跨模块常量

本书介绍 `const` 命令的时候说过，`const` 声明的常量只在当前代码块有效。

本书介绍 `const` 命令的时候说过，`const` 声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法。

```js
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一个专门的 constants 目录，将各种常量写在不同的文件里面，保存在该目录下。

```js
// constants/db.js
export const db = {
    url: 'http://my.couchdbserver.local:5984',
    admin_username: 'admin',
    admin_password: 'admin password'
}

// constant/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然后，将这些文件输出的常量，合并在 `index.js` 里面。

```js
// constants/index.js
export { db } from './db';
export { users } from './users';
```

使用的时候，直接加载 `index.js` 就可以了。

```js
// main.js
import { db, users } from './constants/index';
```

## import()

`import` 命令只会被 JavaScript 静态解析，所以无法在运行时在家模块。

```js
const path = './' + fileName;
const myMoudle = require(path);
```

上面的语句就是动态加载，`require` 到底加载哪一个模块，只有运行时才知道。`import` 命令做不到这一点。

所以引入 `import()` 函数，完成动态加载。

```js
import(specifier);
```

`import()` 返回一个 Promise 对象。

```js
const main = document.querySelector('main');

import(`./section-module/${someVariable}.js`)
    .then(module => {
        module.loadPageInfo(main);
    })
    .catch(err => {
        main.textContent = err.message;
    })
```

### 适用场合

(1) 按需加载

`import` 可以在需要的时候，再加载某个模块。

```js
button.addEventListener('click', event => {
    import('./dialogBox.js')
        .then(dialogBox => {
            dailogBox.open();
        })
        .catch(err => {
            console.log(err);
        })
})
```

(2) 条件加载

`import()` 可以放在 `if` 代码块，根据不同的情况，加载不同的模块。

```js
if (condition) {
    import('moduleA').then(...);
} else {
    import('moduleB').then(...);
}
```

(3) 动态的模块路径

`import()` 允许模块路径动态生成。

```js
import(f())
    .then(...)
```

### 注意点

`import()` 加载成功以后，这个模块会作为一个对象，当做 `then` 方法的参数。因此可以采用结构赋值的语法，获取输出接口。

```js
import('./my-module')
    .then({ export1, export2 } => {
        // ...
    })
```

如果模块有 `default` 输出接口，可以用参数直接获得。

```js
import('./myModule')
    .then(myModule => {
        console.log(myModule.default);
    });

// 或者

import('./myModule')
    .then(({ default: theDefault }) => {
        console.log(theDefault);
    });
```

如果想加载多个模块，可以采用以下方法。

```js
Promise.all([
    import('./module1'),
    import('./module2'),
    import('./module3')
])
.then(([module1, module2, module3]) => {
        // ...
    });
```

`import()` 还可以用在 async 函数中。

```js
async function main () {
    const myModule = await import('./myModule');
    const { export1, export2 } = await import('./myModule');
    const [module1, module2, module3] = await Promise.all([
        import('./module1'),
        import('./module2'),
        import('./module3')
    ]);
}

main();
```