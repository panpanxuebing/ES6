# 正则的扩展

## RegExp 构造函数

```javascript
// ES5
var regex = new RegExp('xyz', 'i');
var regex = new RegExp(/xyz/i);

// 等价于
var regex = /xyz/i;

var regex = new RegExp(/xyz/, 'i') // 报错

// ES6
var regex = new RegExp(/xyz/ig, 'i') // 不会报错，修饰符会被第二个参数替换掉
```

##  u 修饰符

含义为 ‘Unicode’ 模式，用来处理大于`uFFFF`的 Unicode 字符。

```javascript
/^\uD83D$/u.test('\uD83D\uDC2A') // false
/^\uD83D$/.test('\uD83D\uDC2A') // true
```

## RegExp.prototype.unicode 属性

正则实例对象新增`unicode`属性，表示是否设置了`u`修饰符

##  y 修饰符

“粘连”修饰符

和 g 修饰符类似，也是全局匹配，不同之处是 g 只要在剩余位置匹配就可，而 y 必须在从剩余的第一个位置开始。

```javascript
let s = 'aaa_aa_a';
let r1 = /a+/g;
let r2 = /a+/y;

r1.exec(s) // ['aaa']
r2.exec(s) // ['aaa']

r1.exec(s) // ['aa']
r2.exec(s) // null
```

单单一个`y`修饰符对`match`方法，只能返回第一个匹配，必须和`g`修饰符连用，才能返回所有匹配。

```javascript
'a1a2a3'.match(/a\d/y) // ['a1']
'a1a2a3'.match(/a\d/gy) // ['a1', 'a2', 'a3']
```

## RegExp.prototype.sticky 属性

正则实例对象新增`sticky`属性，表示是否设置了`y`修饰符

## RegExp.prototype.flags 属性

新增了`flags`属性，返回正则表达式的修饰符

## s 修饰符：dotAll模式

使得`.`可以匹配任意单个字符

```javascript
const re = /foo.bar/s;
re.test('foo\nbar') // true
re.dotAll // true
re.flags // 's'
```


