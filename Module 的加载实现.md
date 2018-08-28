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


