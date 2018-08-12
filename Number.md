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
})();co

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

## Number.isFinite(), Number.isNaN()

`Number.isFinite()` 来检查一个数值是否时有限（finite）的。

```javascript
Number.isFinite(5); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFInite('15'); // false
```

如果参数不是数值，`Number.isFinite()` 一律返回 `false`。

`Number.isNaN` 判断一个值是否为 `NaN`。

```javascript
Number.isNaN(NaN); // true
Number.isNaN(15); // false
Number.isNaN('true' / 0); // true
```

它们和传统的全局方法 `isFinite()` 和 `isNaN()` 的区别在于，传统方法会先调用 `Number()` 将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效。

## Number.parseInt() ,Number.parseFloat()

ES6 将全局方法 `parseInt()` 和 `parseFloat()` 移植到 `Number`  对象上，行为完全不变。

```javascript
// ES5 的写法
parseInt('12.34'); // 12
parseFloat('12.345#'); // 12.345

// ES6 的写法
Number.parseInt('12.34'); // 12
Number.parseFloat('12.345#'); // 12.345

Number.parseInt === parseInt; // true
Number.parseFloat === parseFloat; // true
```

这样做的目的是，逐步减少全局性方法，使得语言逐步模块化。

## Number.isInteger()

`Number.isInteget()` 来判断一个数值是否是整数。

```javascript
Number.isInteger(12); // true
Number.isInteger(12.1); // false
```

因为在 Javascript 内部，整数和浮点数用的是同样的存储方法，所以 25 和 25.0 被视为同一个值。

```javascript
Number.isInteger(25); // true
Number.isInteger(25.0); // true
```

如果参数不是数值，`Number.isInteget()` 返回 `false`。

如果精度超过限度，`Number.isInteger` 可能产生误判，所以精度要求较高的时候，不建议使用 `Number.isInteger()` 来判断一个数值是否为整数。

```javascript
// 小数位数超过 16 个十进制，转为二进制超过了 53 个二进制位，导致最后 2 被丢弃了。
Number.isInteger(3.0000000000000002); // true
```

## Number.EPSILON

ES6 在 `Number` 对象上面，增加了一个极小的常量，`Number.EPSILON` 。根据规格，它表示 1 与 大于 1 的最小浮点数之差。对于 64 位的浮点数来说，大于 1 的最小浮点数相当于二进制的 `1.00...1`，小数点后有连续 51 个零。这个值减去 1 之后，就等于 2 的 -52 次方。

```javascript
Number.ESPSILON === Math.pow(2, -52);
// true
```

`Number.EPSILON` 实际上表示的是 Javascript 的最小精度。误差如果小于这个值，就可以认为没有意义了，即不存在误差了。

引入一个这么小的量的目的，在于浮点数的运算，设置一个误差范围。我们知道浮点数是不精确的。

```javascript
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3
// 5.551115123125783e-17
```

`Number.EPSILON` 可以用来设置“能够接受的最小误差范围”。比如 2 的 -50 次方（即 `Number.EPSILON * Math.pow(2, 2)`）。假如两个浮点数的差小于这个值，我们就认为这两个数相等。

```javascript
5.551115123125783e-17 < Number.EPSILON * Math.pow(2, 2);
// true
```
因此，Number.EPSILON的实质是一个可以接受的最小误差范围。

```javascript
function withinErrorMargin (left, right) {
    return Math.abs(left - right) < Number.EPSILON * Math.pow(2, 2);
}

0.1 + 0.2 === 0.3;
// false
withinErrorMargin (0.1 + 0.2, 0.3);
// true
```

上面的代码为浮点数运算，部署了一个误差检查函数。

## 安全整数和 Number.isSafeInteger()

Javascript 能表示的整数范围在 `-2^53` 和 `2^53` 之间（不包含两个端点），超过这个范围，无法精确表示这两个值。

```javascript
Math.pow(2, 53); // 9007199254740992
9007199254740992; // 9007199254740992
9007199254740993; // 9007199254740992
Math.pow(2, 53) === Math.pow(2, 53) + 1;
// true
```

ES6 引入 `Number.MAX_SAFE_INTEGER` 和 `Number.MIN_SAFE_INTEGER` 这两个常量，来表示这个范围的上下限。

```javascript
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) -1; // true
Number.MAX_SAFE_INTEGER === 9007199254740991; // true
Number.MIN_SAFE_INTEGER === - Number.MAX_SAFE_INTEGER; // true
Number.MIN_SAFE_INTEGER === - 9007199254740991; // true
```

`Number.isSafeInteger()` 判断一个整数是否落在这个范围内。

```javascript
Number.isSafeInteger('a'); // false
Number.isSageInteger(Infinity); // false

Number.isSafeInteger(3); // true
Number.isSafeInteger(3.1); // false

Number.isSafeInteger(9007199254740990); // true
Number.isSafeInteger(9007199254740992); // false

Number.isSafeInteger(Number.MAX_SAFE_INTEGER); // true
Number.isSageInteger(Number.MAX_SAFE_INTEGER + 1); // fasle 
```

实际运用这个函数时，不只要验证运算结果，还要同时验证参与运算的每一个值。

```javascript
Number.isSafeInteger(9007199254740993); // false
Number.isSafeInterger(900);
Number.isSafeInteger(9007199254740993 - 900); // true

9007199254740993 - 900
// 返回结果 9007199254740002
// 正确答案应该是 9007199254740003
```

因为 `9007199254740993` 不是一个安全整数，导致计算机内部，以 `9007199254740992` 的形式保存。

## Math 对象的拓展

### Math.trunc()

`Math.trunc` 方法用于去除一个数的证书部分，返回小数部分。

```javascript
Math.trunc(4.1); // 4
Math.trunc(-3.1); // -3
```

### Math.sign()

`Math.sign` 判断一个数到底是正数、负数还是零。对于非数值，会先转化为数值。

它会返回五个值：

- 参数为正数，返回 `1`
- 参数为负数，返回 `-1`
- 参数为 0，返回 `0`
- 参数为 -0，返回 `-0`
- 其他值，返回 `NaN`

```javascript
Number.sign(-5); // -1
Number.sign(4); // 1
Number.sign(0); // 0
Number.sign(-0); // -o
Number.sign(NaN); // NaN
```

对于对于没有部署这个方法的环境，可以通过一下代码模拟。

```javascript
Math.sign = Math.sign || function (x) {
    x = +x;

    if (x === 0 || isNaN(x)) {
        return x;
    }
    return x > 0 ? 1 : -1;
}
```

### Math.cbrt()

`Math.cbrt()` 用于计算一个数值的立方根。

对于非数值，先通过 `Number()` 先转为数值。

代码模拟：

```javascript
Math.cbrt = Math.cbrt || function (x) {
    var y = Math.pow(Math.abs(x), 1 / 3);
    return x > 0 ? y : -y;
}
```

### Math.clz32()

JavaScript 的整数使用 32 位二进制形式表示，Math.clz32方法返回一个数的 32 位无符号整数形式有多少个前导 0。

```javascript
Math.clz32(0); // 32
Math.clz32(1); // 31
```

左移运算符（`<<`）与 `Math.clz32()` 相关;

```javascript
Math.clz32(1); // 31
Math.clz32(1 << 1); // 30
```

### Math.imul()

`Math.imul` 方法返回两个数以32位带符号整数相乘的结果，返回的也是一个32位带符号的整数。

```javascript
Math.imul(2, 4); // 8
Math.imul(-2, 4); // -8
```

大部分情况，`Math.imul(a, b)` 与 `a * b` 的结果是相同的。之所以需要部署这个方法，是因为 JavaScript 有精度限制，超过 2 的 53 次方的值无法精确表示。这就是说，对于那些很大的数的乘法，低位数值往往都是不精确的，Math.imul方法可以返回正确的低位数值。

### Math.fround()

`Math.fround()` 方法返回一个数的32位单精度浮点数形式。
对于32位单精度格式来说，数值精度是24个二进制位（1 位隐藏位与 23 位有效位），所以对于 -2^24 至 2^24 之间的整数（不含两个端点），返回结果与参数本身一致。

```javascript
Math.fround(0); // 0
Math.fround(1); // 1
Math.fround(2 ** 24 + 1); // 16777217
```

如果参数的绝对值大于 2^24，返回的结果便开始丢失精度。

```javascript
Math.fround(2 ** 24); // 16777216
Math.fround(2 ** 24 + 1); // 16777216

// 未丢失有效精度
Math.fround(1.25); // 1.25
Math.fround(7,25); // 7,25

// 丢失精度
Math.fround(0.3); // 0.30000001192092896
Math.fround(0.7); // 0.699999988079071
Math.fround(1.0000000123); // 1
```

对于 NaN 和 Infinity，此方法返回原值。对于其它类型的非数值，Math.fround 方法会先将其转为数值，再返回单精度浮点数。

### 对数方法

### 双曲函数方法

### 指数运算符

ES6 新增了指数运算符 `**`。

```javascript
 2 ** 2 // 4
 2 ** 3 // 8

 let a = 2;
 a ** 2;
 // 4
```

