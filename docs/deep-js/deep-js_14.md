# 十、保护对象免受更改

> 原文：[`exploringjs.com/deep-js/ch_protecting-objects.html`](https://exploringjs.com/deep-js/ch_protecting-objects.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   10.1 保护级别：防止扩展、封闭、冻结

+   10.2 防止对象扩展

    +   10.2.1 检查对象是否可扩展

+   10.3 封闭对象

    +   10.3.1 检查对象是否已封闭

+   10.4 冻结对象

    +   10.4.1 检查对象是否已冻结

    +   10.4.2 冻结是浅层的

    +   10.4.3 实现深冻结

+   10.5 进一步阅读

* * *

在本章中，我们将探讨如何保护对象免受更改。例如：防止添加属性和防止更改属性。

![](img/6b17215a2abf7f098e996c026fba60fd.png)  **必需知识：属性特性**

在本章中，您应该熟悉属性特性。如果您不熟悉，请查看§9“属性特性：简介”。

### 10.1 保护级别：防止扩展、封闭、冻结

JavaScript 有三个级别的对象保护：

+   *防止扩展*使得无法向对象添加新属性。但我们仍然可以删除和更改属性。

    +   方法：`Object.preventExtensions(obj)`

+   *封闭*防止扩展并使所有属性*不可配置*（大致上：我们无法再改变属性的工作方式）。

    +   方法：`Object.seal(obj)`

+   *冻结*在使所有属性不可写后封闭对象。也就是说，对象不可扩展，所有属性都是只读的，没有办法改变。

    +   方法：`Object.freeze(obj)`

### 10.2 防止对象扩展

```js
Object.preventExtensions<T>(obj: T): T
```

该方法的工作方式如下：

+   如果`obj`不是一个对象，则返回它。

+   否则，它会改变`obj`，使得我们无法再添加属性，并返回它。

+   类型参数`<T>`表示结果与参数具有相同的类型。

让我们在一个例子中使用`Object.preventExtensions()`：

```js
const obj = { first: 'Jane' };
Object.preventExtensions(obj);
assert.throws(
 () => obj.last = 'Doe',
 /^TypeError: Cannot add property last, object is not extensible$/);
```

但我们仍然可以删除属性：

```js
assert.deepEquals(
 Object.keys(obj), ['first']);
delete obj.first;
assert.deepEquals(
 Object.keys(obj), []);
```

#### 10.2.1 检查对象是否可扩展

```js
Object.isExtensible(obj: any): boolean
```

检查`obj`是否可扩展 - 例如：

```js
> const obj = {};
> Object.isExtensible(obj)
true
> Object.preventExtensions(obj)
{}
> Object.isExtensible(obj)
false
```

### 10.3 封闭对象

```js
Object.seal<T>(obj: T): T
```

该方法的描述：

+   如果`obj`不是一个对象，则返回它。

+   否则，它会防止`obj`的扩展，使得所有属性*不可配置*，并返回它。属性不可配置意味着它们无法再被更改（除了它们的值）：只读属性保持只读，可枚举属性保持可枚举，等等。

以下示例演示了封闭使对象不可扩展且其属性不可配置。

```js
const obj = {
 first: 'Jane',
 last: 'Doe',
};

// Before sealing
assert.equal(Object.isExtensible(obj), true);
assert.deepEqual(
 Object.getOwnPropertyDescriptors(obj),
 {
 first: {
 value: 'Jane',
 writable: true,
 enumerable: true,
 configurable: true
 },
 last: {
 value: 'Doe',
 writable: true,
 enumerable: true,
 configurable: true
 }
 });

Object.seal(obj);

// After sealing
assert.equal(Object.isExtensible(obj), false);
assert.deepEqual(
 Object.getOwnPropertyDescriptors(obj),
 {
 first: {
 value: 'Jane',
 writable: true,
 enumerable: true,
 configurable: false
 },
 last: {
 value: 'Doe',
 writable: true,
 enumerable: true,
 configurable: false
 }
 });
```

但我们仍然可以更改属性`.first`的值：

```js
obj.first = 'John';
assert.deepEqual(
 obj, {first: 'John', last: 'Doe'});
```

但我们无法更改其属性：

```js
assert.throws(
 () => Object.defineProperty(obj, 'first', { enumerable: false }),
 /^TypeError: Cannot redefine property: first$/);
```

#### 10.3.1 检查对象是否已封闭

```js
Object.isSealed(obj: any): boolean
```

检查`obj`是否已封闭 - 例如：

```js
> const obj = {};
> Object.isSealed(obj)
false
> Object.seal(obj)
{}
> Object.isSealed(obj)
true
```

### 10.4 冻结对象

```js
Object.freeze<T>(obj: T): T;
```

+   该方法立即返回`obj`，如果它不是一个对象。

+   否则，它会使所有属性不可写，封闭`obj`并返回它。也就是说，`obj`不可扩展，所有属性都是只读的，没有办法改变。

```js
const point = { x: 17, y: -5 };
Object.freeze(point);

assert.throws(
 () => point.x = 2,
 /^TypeError: Cannot assign to read only property 'x'/);

assert.throws(
 () => Object.defineProperty(point, 'x', {enumerable: false}),
 /^TypeError: Cannot redefine property: x$/);

assert.throws(
 () => point.z = 4,
 /^TypeError: Cannot add property z, object is not extensible$/);
```

#### 10.4.1 检查对象是否已冻结

```js
Object.isFrozen(obj: any): boolean
```

检查`obj`是否被冻结 - 例如：

```js
> const point = { x: 17, y: -5 };
> Object.isFrozen(point)
false
> Object.freeze(point)
{ x: 17, y: -5 }
> Object.isFrozen(point)
true
```

#### 10.4.2 冻结是浅层的

`Object.freeze(obj)`只会冻结`obj`及其属性。它不会冻结这些属性的值 - 例如：

```js
const teacher = {
 name: 'Edna Krabappel',
 students: ['Bart'],
};
Object.freeze(teacher);

// We can’t change own properties:
assert.throws(
 () => teacher.name = 'Elizabeth Hoover',
 /^TypeError: Cannot assign to read only property 'name'/);

// Alas, we can still change values of own properties:
teacher.students.push('Lisa');
assert.deepEqual(
 teacher, {
 name: 'Edna Krabappel',
 students: ['Bart', 'Lisa'],
 });
```

#### 10.4.3 实现深冻结

如果我们想要深冻结，我们需要自己实现它：

```js
function deepFreeze(value) {
 if (Array.isArray(value)) {
 for (const element of value) {
 deepFreeze(element);
 }
 Object.freeze(value);
 } else if (typeof value === 'object' && value !== null) {
 for (const v of Object.values(value)) {
 deepFreeze(v);
 }
 Object.freeze(value);
 } else {
 // Nothing to do: primitive values are already immutable
 } 
 return value;
}
```

重新审视上一节的示例，我们可以检查`deepFreeze()`是否真的深度冻结：

```js
const teacher = {
 name: 'Edna Krabappel',
 students: ['Bart'],
};
deepFreeze(teacher);

assert.throws(
 () => teacher.students.push('Lisa'),
 /^TypeError: Cannot add property 1, object is not extensible$/);
```

### 10.5 进一步阅读

+   有关防止数据结构被更改的模式的更多信息：§15 “不可变集合的包装器”

[评论](https://github.com/rauschma/deep-js/issues/10)
