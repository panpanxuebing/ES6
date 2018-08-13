# Reflect

## 概述

`Reflect` 对象和 `Proxy` 对象一样，也是 ES6 为了操作对象而提供的 API。 `Reflect` 的设计目的是：

- 将 `Object` 对象的一些明显属于语言内部的方法（比如 `Object.defineProperty`），放到 `Reflect` 对象上。也就是说从 `Reflect` 上能拿到语言内部的方法。

- 修改某些 `Object` 方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)` 无法定义属性时，会抛出一个错误，而 `Reflect.defineProperty(obj, name, desc)` 则会返回 `false`。

- 让 `Object` 操作变为函数行为。某些 `Object` 操作是命令式，比如 `name in obj` 和 `delete obj[name]`，而  `Reflect.has(obj, name)` 和 `Reflect.deleteProperty(obj, name)` 则让它们变成了函数行为。

- `Reflect` 对象的方法于 `Proxy` 对象的方法一一对应，只要是 `Proxy` 对象的方法，就能在 `Reflect` 对象上找到对应的方法。这就让 `Proxy` 对象可以方便地调用对应的 `Reflect` 方法，完成默认行为，作为修改行为的基础。也就是说，不管 `Proxy` 怎么修改默认行为，你总可以在 `Reflect` 上获取默认行为。

```javascript
Proxy(target, {
    set (target, name, value, receiver) {
        var success = Reflect.set(target, name, value, receiver);

        if (success) {
            log('property ' + name + ' on ' + target + ' set to ' + value);
        }

        return success;
    }
});
```