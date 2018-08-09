# ES6 有关的其他知识点

## Object.create()

创建一个具有指定原型且可选择性的包含指定属性的对象。

语法：

Object.create(prototype, descriptors);

参数：

- `prototype` 必需，要用做原型的对象，可以为null

- `decriptors` 可选,h包含一个或多个属性描述符的对象

## Object.defineProperty()

## apply(), call()

apply 方法能劫持另一个对象的方法，继承另外一个对象的属性。

### 用法

Function.apply(obj, args)

- obj ：这个对象将替代 Function 类里的 this 对象

- args : 这是个数组，它将作为参数传给 Function 

call 和 apply 的作用差不多，只不过参数l列表不一样。

```javascript
function Person (name, age) {
    this.name = name;
    this.age = age;
}

function Student (name, age, grade) {
    Person.apply(this, arguments);
    // 或者
    Person.call(this, name, age);

    this.grade = grade;
}

var student = new Student('张三', '12', '一年级');

student.name; // 张三
student.age; // 12
student.grade; // 一年级
```

### apply 方法的妙用

上面的例子中，第一个参数是 `this`，第二个参数是一个类数组对象，在调用 Person 的时候，它需要的不是一个数组，但是它依然可以讲数组解析为一个一个的参数。这就是 `apply` 的妙用：它可以将一个数组默认的转换为一个参数列表。利用这个特性，可以提高一些程序的效率。

```javascript
// Max.max 实现获取数组中的最大值
var max = Math.max.apply(null, [1, 2, 3]);
// 3

// Array.prototype.push 实现两个数组合并
var arr1 = [1, 2, 3];
var arr2 = [4, 5, 6];

Array.prototype.push.apply(arr1, arr2);
arr1 // [1, 2, 3, 4, 5, 6]
```