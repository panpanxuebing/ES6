# Number

## 二进制和八进制表示法

ES6 提供了二进制和八进制的新写法，分别使用前缀 `0b` (或 `0B` )和 `0o` (或 `0O` )表示。

```javascript
0b111110111 === 504; // true
0o767 === 503; // true
```

从 ES5 开始，在严格模式中，八进制就不再允许使用前缀 `0` 表示，ES6 进一步明确，要使用前缀 `0o` 表示。

```javascript
// 非严格模式
(function (){ 
    console.log(0o11 === 011);
    // true
})();

// 严格模式
(function () {
    'use strict';
    console.log(0o11 === 011);
    // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
})();
```

如果要将 `0b` 和 `0o` 前缀的字符串数值转为十进制，要使用 `Number` 方法。

```javascript
Number('0b111'); // 7
Number('0o11'); // 8
```