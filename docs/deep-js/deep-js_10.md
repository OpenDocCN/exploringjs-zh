# 7 更新数据的破坏性和非破坏性

> 原文：[`exploringjs.com/deep-js/ch_updating-destructively-and-nondestructively.html`](https://exploringjs.com/deep-js/ch_updating-destructively-and-nondestructively.html)

* * *

+   7.1 示例：破坏性和非破坏性地更新对象

+   7.2 示例：破坏性和非破坏性地更新数组

+   7.3 手动深层更新

+   7.4 实现通用的深层更新

* * *

在本章中，我们学习了两种不同的更新数据的方式：

+   *破坏性更新*会改变数据，使其具有所需的形式。

+   *非破坏性更新*会创建一个具有所需形式的数据副本。

后一种方式类似于首先制作副本，然后进行破坏性更改，但它同时进行了两者。

### 7.1 示例：破坏性和非破坏性地更新对象

以下代码展示了一个更新对象属性的函数，并在对象上使用它。

```js
function setPropertyDestructively(obj, key, value) {
 obj[key] = value;
 return obj;
}

const obj = {city: 'Berlin', country: 'Germany'};
setPropertyDestructively(obj, 'city', 'Munich');
assert.deepEqual(obj, {city: 'Munich', country: 'Germany'});
```

以下代码演示了对对象进行非破坏性更新：

```js
function setPropertyNonDestructively(obj, key, value) {
 const updatedObj = {};
 for (const [k, v] of Object.entries(obj)) {
 updatedObj[k] = (k === key ? value : v);
 }
 return updatedObj;
}

const obj = {city: 'Berlin', country: 'Germany'};
const updatedObj = setPropertyNonDestructively(obj, 'city', 'Munich');

// We have created an updated object:
assert.deepEqual(updatedObj, {city: 'Munich', country: 'Germany'});

// But we didn’t change the original:
assert.deepEqual(obj, {city: 'Berlin', country: 'Germany'});
```

扩展使`setPropertyNonDestructively()`更简洁：

```js
function setPropertyNonDestructively(obj, key, value) {
 return {...obj, [key]: value};
}
```

`setPropertyNonDestructively()`的两个版本都是*浅层*更新：它们只改变对象的顶层。

### 7.2 示例：破坏性和非破坏性地更新数组

以下代码展示了一个更新数组元素的函数，并在数组上使用它。

```js
function setElementDestructively(arr, index, value) {
 arr[index] = value;
}

const arr = ['a', 'b', 'c', 'd', 'e'];
setElementDestructively(arr, 2, 'x');
assert.deepEqual(arr, ['a', 'b', 'x', 'd', 'e']);
```

以下代码演示了对数组进行非破坏性更新：

```js
function setElementNonDestructively(arr, index, value) {
 const updatedArr = [];
 for (const [i, v] of arr.entries()) {
 updatedArr.push(i === index ? value : v);
 }
 return updatedArr;
}

const arr = ['a', 'b', 'c', 'd', 'e'];
const updatedArr = setElementNonDestructively(arr, 2, 'x');
assert.deepEqual(updatedArr, ['a', 'b', 'x', 'd', 'e']);
assert.deepEqual(arr, ['a', 'b', 'c', 'd', 'e']);
```

`.slice()`和扩展使`setElementNonDestructively()`更简洁：

```js
function setElementNonDestructively(arr, index, value) {
 return [
 ...arr.slice(0, index), value, ...arr.slice(index+1)];
}
```

`setElementNonDestructively()`的两个版本都是*浅层*更新：它们只改变数组的顶层。

### 7.3 手动深层更新

到目前为止，我们只是浅层更新了数据。让我们来解决深层更新的问题。以下代码展示了如何手动进行深层更新。我们正在更改名称和雇主。

```js
const original = {name: 'Jane', work: {employer: 'Acme'}};
const updatedOriginal = {
 ...original,
 name: 'John',
 work: {
 ...original.work,
 employer: 'Spectre'
 },
};

assert.deepEqual(
 original, {name: 'Jane', work: {employer: 'Acme'}});
assert.deepEqual(
 updatedOriginal, {name: 'John', work: {employer: 'Spectre'}});
```

### 7.4 实现通用的深层更新

以下函数实现了通用的深层更新。

```js
function deepUpdate(original, keys, value) {
 if (keys.length === 0) {
 return value;
 }
 const currentKey = keys[0];
 if (Array.isArray(original)) {
 return original.map(
 (v, index) => index === currentKey
 ? deepUpdate(v, keys.slice(1), value) // (A)
 : v); // (B)
 } else if (typeof original === 'object' && original !== null) {
 return Object.fromEntries(
 Object.entries(original).map(
 (keyValuePair) => {
 const [k,v] = keyValuePair;
 if (k === currentKey) {
 return [k, deepUpdate(v, keys.slice(1), value)]; // (C)
 } else {
 return keyValuePair; // (D)
 }
 }));
 } else {
 // Primitive value
 return original;
 }
}
```

如果我们将`value`视为正在更新的树的根，那么`deepUpdate()`只会深度更改单个分支（A 和 C 行）。所有其他分支都是浅层复制（B 和 D 行）。

使用`deepUpdate()`看起来像这样：

```js
const original = {name: 'Jane', work: {employer: 'Acme'}};

const copy = deepUpdate(original, ['work', 'employer'], 'Spectre');
assert.deepEqual(copy, {name: 'Jane', work: {employer: 'Spectre'}});
assert.deepEqual(original, {name: 'Jane', work: {employer: 'Acme'}});
```

[评论](https://github.com/rauschma/deep-js/issues/7)
