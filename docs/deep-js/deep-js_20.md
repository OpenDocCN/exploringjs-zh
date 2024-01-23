# 15 不可变集合的包装器

> 原文：[`exploringjs.com/deep-js/ch_immutable-collection-wrappers.html`](https://exploringjs.com/deep-js/ch_immutable-collection-wrappers.html)

* * *

+   15.1 包装对象

    +   15.1.1 通过包装使集合变为不可变

+   15.2 Map 的不可变包装器

+   15.3 数组的不可变包装器

* * *

通过包装集合的不可变包装器使该集合变为不可变。在本章中，我们将探讨其工作原理以及其有用之处。

### 15.1 包装对象

如果有一个我们想要减少接口的对象，我们可以采取以下方法：

+   创建一个新对象，将原始对象存储在私有字段中。新对象被称为*包装器*，原始对象被称为*被包装对象*。

+   包装器只将它接收到的一些方法调用转发给被包装对象。

包装的样子如下：

```js
class Wrapper {
 #wrapped;
 constructor(wrapped) {
 this.#wrapped = wrapped;
 }
 allowedMethod1(...args) {
 return this.#wrapped.allowedMethod1(...args);
 }
 allowedMethod2(...args) {
 return this.#wrapped.allowedMethod2(...args);
 }
}
```

相关软件设计模式：

+   包装与[四人帮设计模式*Facade*](https://en.wikipedia.org/wiki/Facade_pattern)有关。

+   我们使用*转发*来实现*委托*。委托意味着一个对象让另一个对象（*委托*）处理它的一些工作。这是共享代码的继承的替代方法。

#### 15.1.1 通过包装使集合变为不可变

要使集合变为不可变，我们可以使用包装并从其接口中删除所有破坏性操作。

这种技术的一个重要用例是一个具有内部可变数据结构的对象，它希望安全地导出而不复制它。导出是“活动的”也可能是一个目标。对象可以通过包装内部数据结构并使其不可变来实现其目标。

接下来的两节展示了 Map 和数组的不可变包装器。它们都有以下限制：

+   它们只是草图。需要更多的工作使它们适用于实际使用：更好的检查，支持更多的方法等。

+   它们的工作是浅层的：每个都使被包装对象不可变，但不影响其返回的数据。这可以通过包装一些方法返回的结果来修复。

### 15.2 不可变的 Map 包装器

类`ImmutableMapWrapper`生成 Map 的包装器：

```js
class ImmutableMapWrapper {
 static _setUpPrototype() {
 // Only forward non-destructive methods to the wrapped Map:
 for (const methodName of ['get', 'has', 'keys', 'size']) {
 ImmutableMapWrapper.prototype[methodName] = function (...args) {
 return this.#wrappedMapmethodName;
 }
 }
 }

 #wrappedMap;
 constructor(wrappedMap) {
 this.#wrappedMap = wrappedMap;
 }
}
ImmutableMapWrapper._setUpPrototype();
```

原型的设置必须由静态方法执行，因为我们只能从类内部访问私有字段`.#wrappedMap`。

这是`ImmutableMapWrapper`的实际应用：

```js
const map = new Map([[false, 'no'], [true, 'yes']]);
const wrapped = new ImmutableMapWrapper(map);

// Non-destructive operations work as usual:
assert.equal(
 wrapped.get(true), 'yes');
assert.equal(
 wrapped.has(false), true);
assert.deepEqual(
 [...wrapped.keys()], [false, true]);

// Destructive operations are not available:
assert.throws(
 () => wrapped.set(false, 'never!'),
 /^TypeError: wrapped.set is not a function$/);
assert.throws(
 () => wrapped.clear(),
 /^TypeError: wrapped.clear is not a function$/);
```

### 15.3 不可变的数组包装器

对于数组`arr`，普通的包装是不够的，因为我们不仅需要拦截方法调用，还需要拦截属性访问，比如`arr[1] = true`。[JavaScript 代理](https://exploringjs.com/es6/ch_proxies.html)使我们能够做到这一点：

```js
const RE_INDEX_PROP_KEY = /^[0-9]+$/;
const ALLOWED_PROPERTIES = new Set([
 'length', 'constructor', 'slice', 'concat']);

function wrapArrayImmutably(arr) {
 const handler = {
 get(target, propKey, receiver) {
 // We assume that propKey is a string (not a symbol)
 if (RE_INDEX_PROP_KEY.test(propKey) // simplified check!
 || ALLOWED_PROPERTIES.has(propKey)) {
 return Reflect.get(target, propKey, receiver);
 }
 throw new TypeError(`Property "${propKey}" can’t be accessed`);
 },
 set(target, propKey, value, receiver) {
 throw new TypeError('Setting is not allowed');
 },
 deleteProperty(target, propKey) {
 throw new TypeError('Deleting is not allowed');
 },
 };
 return new Proxy(arr, handler);
}
```

让我们来包装一个数组：

```js
const arr = ['a', 'b', 'c'];
const wrapped = wrapArrayImmutably(arr);

// Non-destructive operations are allowed:
assert.deepEqual(
 wrapped.slice(1), ['b', 'c']);
assert.equal(
 wrapped[1], 'b');

// Destructive operations are not allowed:
assert.throws(
 () => wrapped[1] = 'x',
 /^TypeError: Setting is not allowed$/);
assert.throws(
 () => wrapped.shift(),
 /^TypeError: Property "shift" can’t be accessed$/);
```

[评论](https://github.com/rauschma/deep-js/issues/15)
