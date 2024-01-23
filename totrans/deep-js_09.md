# 6 复制对象和数组

> 原文：[`exploringjs.com/deep-js/ch_copying-objects-and-arrays.html`](https://exploringjs.com/deep-js/ch_copying-objects-and-arrays.html)

* * *

+   6.1 浅复制 vs. 深复制

+   6.2 JavaScript 中的浅复制

    +   6.2.1 通过扩展复制普通对象和数组

    +   6.2.2 通过`Object.assign()`进行浅复制（可选）

    +   6.2.3 通过`Object.getOwnPropertyDescriptors()`和`Object.defineProperties()`进行浅复制（可选）

+   6.3 JavaScript 中的深复制

    +   6.3.1 通过嵌套扩展手动进行深复制

    +   6.3.2 技巧：通过 JSON 进行通用深复制

    +   6.3.3 实现通用深复制

+   6.4 进一步阅读

* * *

在本章中，我们将学习如何在 JavaScript 中复制对象和数组。

### 6.1 浅复制 vs. 深复制

数据可以以两种“深度”进行复制：

+   *浅复制*只会复制对象和数组的顶层条目。条目的值在原始和副本中仍然相同。

+   *深复制*还会复制值的条目的条目等。也就是说，它遍历了以要复制的值为根的完整树，并复制了所有节点。

接下来的部分涵盖了两种复制方式。不幸的是，JavaScript 只内置了对浅复制的支持。如果我们需要深复制，就需要自己实现。

### 6.2 JavaScript 中的浅复制

让我们看看浅复制数据的几种方法。

#### 6.2.1 通过扩展复制普通对象和数组

我们可以将[扩展到对象字面量](https://exploringjs.com/impatient-js/ch_single-objects.html#spreading-into-object-literals)和[扩展到数组字面量](https://exploringjs.com/impatient-js/ch_arrays.html#spreading-into-array-literals)中进行复制：

```js
const copyOfObject = {...originalObject};
const copyOfArray = [...originalArray];
```

遗憾的是，扩展存在一些问题。这些将在下一小节中介绍。其中一些是真正的限制，另一些只是特殊情况。

##### 6.2.1.1 对象扩展不会复制原型

例如：

```js
class MyClass {}

const original = new MyClass();
assert.equal(original instanceof MyClass, true);

const copy = {...original};
assert.equal(copy instanceof MyClass, false);
```

请注意以下两个表达式是等价的：

```js
obj instanceof SomeClass
SomeClass.prototype.isPrototypeOf(obj)
```

因此，我们可以通过给副本设置与原始相同的原型来解决这个问题：

```js
class MyClass {}

const original = new MyClass();

const copy = {
 __proto__: Object.getPrototypeOf(original),
 ...original,
};
assert.equal(copy instanceof MyClass, true);
```

或者，我们可以在创建副本后通过`Object.setPrototypeOf()`设置其原型。

##### 6.2.1.2 许多内置对象具有特殊的“内部插槽”，对象扩展不会复制它们

此类内置对象的示例包括正则表达式和日期。如果我们复制它们，就会丢失其中存储的大部分数据。

##### 6.2.1.3 对象扩展只会复制自有（非继承）属性

考虑到[原型链](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#prototype-chains)的工作方式，这通常是正确的方法。但我们仍然需要意识到这一点。在下面的例子中，原型的继承属性`.inheritedProp`在`copy`中不可用，因为我们只复制自有属性，而不保留原型。

```js
const proto = { inheritedProp: 'a' };
const original = {__proto__: proto, ownProp: 'b' };
assert.equal(original.inheritedProp, 'a');
assert.equal(original.ownProp, 'b');

const copy = {...original};
assert.equal(copy.inheritedProp, undefined);
assert.equal(copy.ownProp, 'b');
```

##### 6.2.1.4 对象扩展只会复制可枚举属性

例如，数组实例的自有属性`.length`不可枚举，也不会被复制。在下面的例子中，我们通过对象扩展（行 A）复制了数组`arr`：

```js
const arr = ['a', 'b'];
assert.equal(arr.length, 2);
assert.equal({}.hasOwnProperty.call(arr, 'length'), true);

const copy = {...arr}; // (A)
assert.equal({}.hasOwnProperty.call(copy, 'length'), false);
```

这也很少是一个限制，因为大多数属性都是可枚举的。如果我们需要复制不可枚举的属性，我们可以使用`Object.getOwnPropertyDescriptors()`和`Object.defineProperties()`来复制对象（如何做到这一点稍后会解释）：

+   它们考虑所有属性（不仅仅是`value`），因此正确地复制了 getter、setter、只读属性等。

+   `Object.getOwnPropertyDescriptors()`检索可枚举和不可枚举的属性。

有关可枚举性的更多信息，请参见§12“属性的可枚举性”。

##### 6.2.1.5 对象扩展并不总是忠实地复制属性

不考虑属性的*属性*，它的副本始终是一个可写和可配置的数据属性。

例如，在这里，我们创建了属性`original.prop`，其属性`writable`和`configurable`都是`false`：

```js
const original = Object.defineProperties(
 {}, {
 prop: {
 value: 1,
 writable: false,
 configurable: false,
 enumerable: true,
 },
 });
assert.deepEqual(original, {prop: 1});
```

如果我们复制`.prop`，那么`writable`和`configurable`都是`true`：

```js
const copy = {...original};
// Attributes `writable` and `configurable` of copy are different:
assert.deepEqual(
 Object.getOwnPropertyDescriptors(copy),
 {
 prop: {
 value: 1,
 writable: true,
 configurable: true,
 enumerable: true,
 },
 });
```

因此，getter 和 setter 也不会被忠实地复制：

```js
const original = {
 get myGetter() { return 123 },
 set mySetter(x) {},
};
assert.deepEqual({...original}, {
 myGetter: 123, // not a getter anymore!
 mySetter: undefined,
});
```

前面提到的`Object.getOwnPropertyDescriptors()`和`Object.defineProperties()`总是以完整的属性传输自有属性（如后面所示）。

##### 6.2.1.6 复制是浅的

副本中有原始每个键值条目的新版本，但原始的值本身并没有被复制。例如：

```js
const original = {name: 'Jane', work: {employer: 'Acme'}};
const copy = {...original};

// Property .name is a copy: changing the copy
// does not affect the original
copy.name = 'John';
assert.deepEqual(original,
 {name: 'Jane', work: {employer: 'Acme'}});
assert.deepEqual(copy,
 {name: 'John', work: {employer: 'Acme'}});

// The value of .work is shared: changing the copy
// affects the original
copy.work.employer = 'Spectre';
assert.deepEqual(
 original, {name: 'Jane', work: {employer: 'Spectre'}});
assert.deepEqual(
 copy, {name: 'John', work: {employer: 'Spectre'}});
```

我们将在本章后面讨论深拷贝。

#### 6.2.2 通过`Object.assign()`进行浅复制（可选）

`Object.assign()`在大多数情况下像扩展到对象中。也就是说，以下两种复制方式大部分是等价的：

```js
const copy1 = {...original};
const copy2 = Object.assign({}, original);
```

使用方法而不是语法的好处是，它可以在旧的 JavaScript 引擎上通过库进行填充。

`Object.assign()`并不完全像扩展，它在一个相对微妙的地方有所不同：它以不同的方式创建属性。

+   `Object.assign()`使用*赋值*来创建副本的属性。

+   扩展*定义*了副本中的新属性。

除其他事项外，赋值会调用自有和继承的 setter，而定义不会（有关赋值与定义的更多信息）。这种差异很少会被注意到。以下代码是一个例子，但它是刻意的：

```js
const original = {['__proto__']: null}; // (A)
const copy1 = {...original};
// copy1 has the own property '__proto__'
assert.deepEqual(
 Object.keys(copy1), ['__proto__']);

const copy2 = Object.assign({}, original);
// copy2 has the prototype null
assert.equal(Object.getPrototypeOf(copy2), null);
```

通过在 A 行使用计算属性键，我们创建了`.__proto__`作为自有属性，并且没有调用继承的 setter。然而，当`Object.assign()`复制该属性时，它会调用 setter。（有关`.__proto__`的更多信息，请参见[“JavaScript for impatient programmers”](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#proto)。）

#### 6.2.3 通过`Object.getOwnPropertyDescriptors()`和`Object.defineProperties()`进行浅复制（可选）

JavaScript 允许我们通过*属性描述符*创建属性，这些属性描述了属性的属性。例如，通过`Object.defineProperties()`，我们已经看到了它的作用。如果我们将该方法与`Object.getOwnPropertyDescriptors()`结合使用，我们可以更忠实地复制：

```js
function copyAllOwnProperties(original) {
 return Object.defineProperties(
 {}, Object.getOwnPropertyDescriptors(original));
}
```

这消除了通过扩展复制对象的两个问题。

首先，自有属性的所有属性都被正确地复制。因此，我们现在可以复制自有的 getter 和 setter：

```js
const original = {
 get myGetter() { return 123 },
 set mySetter(x) {},
};
assert.deepEqual(copyAllOwnProperties(original), original);
```

其次，由于`Object.getOwnPropertyDescriptors()`，不可枚举的属性也被复制了：

```js
const arr = ['a', 'b'];
assert.equal(arr.length, 2);
assert.equal({}.hasOwnProperty.call(arr, 'length'), true);

const copy = copyAllOwnProperties(arr);
assert.equal({}.hasOwnProperty.call(copy, 'length'), true);
```

### 6.3 JavaScript 中的深拷贝

现在是时候解决深拷贝的问题了。首先，我们将手动进行深拷贝，然后我们将研究通用的方法。

#### 6.3.1 通过嵌套扩展手动进行深拷贝

如果我们嵌套扩展，我们会得到深拷贝：

```js
const original = {name: 'Jane', work: {employer: 'Acme'}};
const copy = {name: original.name, work: {...original.work}};

// We copied successfully:
assert.deepEqual(original, copy);
// The copy is deep:
assert.ok(original.work !== copy.work);
```

#### 6.3.2 技巧：通过 JSON 进行通用深拷贝

这是一个技巧，但在紧急情况下，它提供了一个快速解决方案：为了深拷贝一个对象`original`，我们首先将其转换为 JSON 字符串，然后解析该 JSON 字符串：

```js
function jsonDeepCopy(original) {
 return JSON.parse(JSON.stringify(original));
}
const original = {name: 'Jane', work: {employer: 'Acme'}};
const copy = jsonDeepCopy(original);
assert.deepEqual(original, copy);
```

这种方法的重大缺点是，我们只能复制由 JSON 支持的键和值的属性。

一些不支持的键和值会被简单地忽略：

```js
assert.deepEqual(
 jsonDeepCopy({
 // Symbols are not supported as keys
 [Symbol('a')]: 'abc',
 // Unsupported value
 b: function () {},
 // Unsupported value
 c: undefined,
 }),
 {} // empty object
);
```

其他会引发异常：

```js
assert.throws(
 () => jsonDeepCopy({a: 123n}),
 /^TypeError: Do not know how to serialize a BigInt$/);
```

#### 6.3.3 实现通用的深拷贝

以下函数通用地深拷贝一个值`original`：

```js
function deepCopy(original) {
 if (Array.isArray(original)) {
 const copy = [];
 for (const [index, value] of original.entries()) {
 copy[index] = deepCopy(value);
 }
 return copy;
 } else if (typeof original === 'object' && original !== null) {
 const copy = {};
 for (const [key, value] of Object.entries(original)) {
 copy[key] = deepCopy(value);
 }
 return copy;
 } else {
 // Primitive value: atomic, no need to copy
 return original;
 }
}
```

该函数处理三种情况：

+   如果`original`是一个数组，我们创建一个新的数组，并将`original`的元素深拷贝到其中。

+   如果`original`是一个对象，我们使用类似的方法。

+   如果`original`是一个原始值，我们不需要做任何事情。

让我们试试`deepCopy()`：

```js
const original = {a: 1, b: {c: 2, d: {e: 3}}};
const copy = deepCopy(original);

// Are copy and original deeply equal?
assert.deepEqual(copy, original);

// Did we really copy all levels
// (equal content, but different objects)?
assert.ok(copy     !== original);
assert.ok(copy.b   !== original.b);
assert.ok(copy.b.d !== original.b.d);
```

请注意，`deepCopy()`只修复了扩展的一个问题：浅拷贝。其他问题仍然存在：原型不会被复制，特殊对象只会被部分复制，不可枚举的属性会被忽略，大多数属性特性会被忽略。

通用地实现完全的复制通常是不可能的：并非所有数据都是树状结构，有时我们不想复制所有属性，等等。

##### 6.3.3.1 更简洁的`deepCopy()`版本

如果我们使用`.map()`和`Object.fromEntries()`，我们可以使我们先前的`deepCopy()`实现更加简洁：

```js
function deepCopy(original) {
 if (Array.isArray(original)) {
 return original.map(elem => deepCopy(elem));
 } else if (typeof original === 'object' && original !== null) {
 return Object.fromEntries(
 Object.entries(original)
 .map(([k, v]) => [k, deepCopy(v)]));
 } else {
 // Primitive value: atomic, no need to copy
 return original;
 }
}
```

### 6.4 进一步阅读

+   第 14 节“复制类的实例：`.clone()` vs. copy constructors”解释了基于类的复制模式。

+   [“扩展到对象文字”部分](https://exploringjs.com/impatient-js/ch_single-objects.html#spreading-into-object-literals)在“JavaScript for impatient programmers”中

+   [“扩展到数组文字”部分](https://exploringjs.com/impatient-js/ch_arrays.html#spreading-into-array-literals)在“JavaScript for impatient programmers”中

+   [“原型链”部分](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#prototype-chains)在“JavaScript for impatient programmers”中

[评论](https://github.com/rauschma/deep-js/issues/6)
