# 36 WeakSets (WeakSet) (advanced)

> 原文：[`exploringjs.com/impatient-js/ch_weaksets.html`](https://exploringjs.com/impatient-js/ch_weaksets.html)

* * *

+   36.1 示例：将对象标记为可与方法一起使用

+   36.2 WeakSet API

* * *

WeakSets 类似于 Sets，具有以下区别：

+   它们可以在不阻止这些对象被垃圾回收的情况下持有对象。

+   它们是黑匣子：我们只有在拥有 WeakSet 和一个值的情况下才能从 WeakSet 中获取任何数据。支持的唯一方法是`.add()`、`.delete()`、`.has()`。请参阅 WeakMaps 作为黑匣子一节，了解为什么 WeakSets 不允许迭代、循环和清除。

鉴于我们无法遍历它们的元素，WeakSets 的用例并不那么多。它们确实使我们能够标记对象。

### 36.1 示例：将对象标记为可与方法一起使用

以下代码演示了一个类如何确保其方法仅应用于由它创建的实例（基于[Domenic Denicola 的代码](https://mail.mozilla.org/pipermail/es-discuss/2015-June/043027.html)）：

```js
const instancesOfSafeClass = new WeakSet();

class SafeClass {
 constructor() {
 instancesOfSafeClass.add(this);
 }

 method() {
 if (!instancesOfSafeClass.has(this)) {
 throw new TypeError('Incompatible object!');
 }
 }
}

const safeInstance = new SafeClass();
safeInstance.method(); // works

assert.throws(
 () => {
 const obj = {};
 SafeClass.prototype.method.call(obj); // throws an exception
 },
 TypeError
);
```

### 36.2 WeakSet API

`WeakSet`的构造函数和三个方法与它们的`Set`等效方法的工作方式相同（参见 ch_sets.html#quickref-sets）：

+   `new WeakSet<T>(values?: Iterable<T>)` ^([ES6])

+   `.add(value: T): this` ^([ES6])

+   `.delete(value: T): boolean` ^([ES6])

+   `.has(value: T): boolean` ^([ES6])

[评论](https://github.com/rauschma/impatient-js/issues/38)
