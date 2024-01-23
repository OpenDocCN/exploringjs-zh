# 三十一、数组 (Array)

> 原文：[`exploringjs.com/impatient-js/ch_arrays.html`](https://exploringjs.com/impatient-js/ch_arrays.html)

* * *

+   31.1 速查表：数组

    +   31.1.1 使用数组

    +   31.1.2 数组方法

+   31.2 JavaScript 中使用数组的两种方式

+   31.3 基本数组操作

    +   31.3.1 创建、读取、写入数组

    +   31.3.2 数组的`.length`

    +   31.3.3 通过负索引引用元素

    +   31.3.4 清除数组

    +   31.3.5 [扩展到数组文字[ES6]](ch_arrays.html#spreading-into-array-literals)

    +   31.3.6 [数组：列出索引和条目[ES6]](ch_arrays.html#arrays-listing-indices-and-entries-es6)

    +   31.3.7 一个值是数组吗？

+   31.4 [`for-of`和数组[ES6]](ch_arrays.html#for-of-and-arrays-es6)

    +   31.4.1 `for-of`：遍历元素

    +   31.4.2 `for-of`：遍历索引

    +   31.4.3 [`for-of`：遍历[index, element]对](ch_arrays.html#for-of-iterating-over-index-element-pairs)

+   31.5 类数组对象

+   31.6 将可迭代对象和类数组值转换为数组

    +   31.6.1 通过扩展(`...`)将可迭代对象转换为数组

    +   31.6.2 通过`Array.from()`将可迭代对象和类数组对象转换为数组

+   31.7 创建和填充任意长度的数组

    +   31.7.1 我们需要创建一个空数组，然后稍后完全填充吗？

    +   31.7.2 我们需要创建一个填充有原始值的数组吗？

    +   31.7.3 我们需要创建一个填充有对象的数组吗？

    +   31.7.4 我们需要创建一个整数范围吗？

    +   31.7.5 如果元素都是整数或浮点数，则使用 Typed Array

+   31.8 多维数组

+   31.9 更多数组特性（高级）

    +   31.9.1 数组索引是（稍微特殊的）属性键

    +   31.9.2 数组是字典，可以有空隙

+   31.10 添加和移除元素（破坏性和非破坏性）

    +   31.10.1 在元素和数组之前添加元素

    +   31.10.2 追加元素和数组

    +   31.10.3 移除元素

+   31.11 方法：迭代和转换（`.find()`, `.map()`, `.filter()`等）

    +   31.11.1 迭代和转换方法的回调

    +   31.11.2 搜索元素：`.find()`, `.findIndex()`

    +   31.11.3 `.map()`: 复制并给元素赋予新值

    +   31.11.4 `.flatMap()`: 映射到零个或多个值

    +   31.11.5 `.filter()`: 仅保留一些元素

    +   31.11.6 `.reduce()`: 从数组中派生值（高级）

+   31.12 `.sort()`: 排序数组

    +   31.12.1 自定义排序顺序

    +   31.12.2 排序数字

    +   31.12.3 排序对象

+   31.13 快速参考：`Array`

    +   31.13.1 `new Array()`

    +   31.13.2 `Array`的静态方法

    +   31.13.3 `Array.prototype`的方法

    +   31.13.4 来源

* * *

### 31.1 数组速查表

JavaScript 数组是一种非常灵活的数据结构，用作列表、堆栈、队列、元组（例如对）等。

一些与数组相关的操作会破坏性地改变数组。其他操作会非破坏性地产生对原始内容副本应用了更改的新数组。

#### 31.1.1 使用数组

创建数组，读取和写入元素：

```js
// Creating an Array
const arr = ['a', 'b', 'c']; // Array literal
assert.deepEqual(
 arr,
 [ // Array literal
 'a',
 'b',
 'c', // trailing commas are ignored
 ]
);

// Reading elements
assert.equal(
 arr[0], 'a' // negative indices don’t work
);
assert.equal(
 arr.at(-1), 'c' // negative indices work
);

// Writing an element
arr[0] = 'x';
assert.deepEqual(
 arr, ['x', 'b', 'c']
);
```

数组的长度：

```js
const arr = ['a', 'b', 'c'];
assert.equal(
 arr.length, 3 // number of elements
);
arr.length = 1; // removing elements
assert.deepEqual(
 arr, ['a']
);
arr[arr.length] = 'b'; // adding an element
assert.deepEqual(
 arr, ['a', 'b']
);
```

通过`.push()`破坏性地添加元素：

```js
const arr = ['a', 'b'];

arr.push('c'); // adding an element
assert.deepEqual(
 arr, ['a', 'b', 'c']
);

// Pushing Arrays (used as arguments via spreading (...)):
arr.push(...['d', 'e']);
assert.deepEqual(
 arr, ['a', 'b', 'c', 'd', 'e']
);
```

通过扩展（`...`）非破坏性地添加元素：

```js
const arr1 = ['a', 'b'];
const arr2 = ['c'];
assert.deepEqual(
 [...arr1, ...arr2, 'd', 'e'],
 ['a', 'b', 'c', 'd', 'e']
);
```

清除数组（删除所有元素）：

```js
// Destructive – affects everyone referring to the Array:
const arr1 = ['a', 'b', 'c'];
arr1.length = 0;
assert.deepEqual(
 arr1, []
);

// Non-destructive – does not affect others referring to the Array:
let arr2 = ['a', 'b', 'c'];
arr2 = [];
assert.deepEqual(
 arr2, []
);
```

循环遍历元素：

```js
const arr = ['a', 'b', 'c'];
for (const value of arr) {
 console.log(value);
}

// Output:
// 'a'
// 'b'
// 'c'
```

循环遍历索引-值对：

```js
const arr = ['a', 'b', 'c'];
for (const [index, value] of arr.entries()) {
 console.log(index, value);
}

// Output:
// 0, 'a'
// 1, 'b'
// 2, 'c'
```

在无法使用数组文字（例如因为我们事先不知道它们的长度或者它们太大）时创建和填充数组：

```js
const four = 4;

// Empty Array that we’ll fill later
assert.deepEqual(
 new Array(four),
 [ , , , ,] // four holes; last comma is ignored
);

// An Array filled with a primitive value
assert.deepEqual(
 new Array(four).fill(0),
 [0, 0, 0, 0]
);

// An Array filled with objects
// Why not .fill()? We’d get single object, shared multiple times.
assert.deepEqual(
 Array.from({length: four}, () => ({})),
 [{}, {}, {}, {}]
);

// A range of integers
assert.deepEqual(
 Array.from({length: four}, (_, i) => i),
 [0, 1, 2, 3]
);
```

#### 31.1.2 数组方法

本节简要概述了数组 API。本章末尾有更全面的快速参考。

从现有数组派生新数组：

```js
> ['■','●','▲'].slice(1, 3)
['●','▲']
> ['■','●','■'].filter(x => x==='■') 
['■','■']

> ['▲','●'].map(x => x+x)
['▲▲','●●']
> ['▲','●'].flatMap(x => [x,x])
['▲','▲','●','●']
```

在给定索引处删除数组元素：

```js
// .filter(): remove non-destructively
const arr1 = ['■','●','▲'];
assert.deepEqual(
 arr1.filter((_, index) => index !== 1),
 ['■','▲']
);
assert.deepEqual(
 arr1, ['■','●','▲'] // unchanged
);

// .splice(): remove destructively
const arr2 = ['■','●','▲'];
arr2.splice(1, 1); // start at 1, delete 1 element
assert.deepEqual(
 arr2, ['■','▲'] // changed
);
```

计算数组的摘要：

```js
> ['■','●','▲'].some(x => x==='●')
true
> ['■','●','▲'].every(x => x==='●')
false

> ['■','●','▲'].join('-')
'■-●-▲'

> ['■','▲'].reduce((result,x) => result+x, '●')
'●■▲'
> ['■','▲'].reduceRight((result,x) => result+x, '●')
'●▲■'
```

反转和填充：

```js
// .reverse() changes and returns `arr`
const arr = ['■','●','▲'];
assert.deepEqual(
 arr.reverse(), arr
);
// `arr` was changed:
assert.deepEqual(
 arr, ['▲','●','■']
);

// .fill() works the same way:
assert.deepEqual(
 ['■','●','▲'].fill('●'),
 ['●','●','●']
);
```

`.sort()`也修改数组并返回它：

```js
// By default, string representations of the Array elements
// are sorted lexicographically:
assert.deepEqual(
 [200, 3, 10].sort(),
 [10, 200, 3]
);

// Sorting can be customized via a callback:
assert.deepEqual(
 [200, 3, 10].sort((a,b) => a - b), // sort numerically
 [ 3, 10, 200 ]
);
```

查找数组元素：

```js
> ['■','●','■'].includes('■')
true
> ['■','●','■'].indexOf('■')
0
> ['■','●','■'].lastIndexOf('■')
2
> ['■','●','■'].find(x => x==='■')
'■'
> ['■','●','■'].findIndex(x => x==='■')
0
```

在开头或结尾添加或删除元素：

```js
// Adding and removing at the start
const arr1 = ['■','●'];
arr1.unshift('▲');
assert.deepEqual(
 arr1, ['▲','■','●']
);
arr1.shift();
assert.deepEqual(
 arr1, ['■','●']
);

// Adding and removing at the end
const arr2 = ['■','●'];
arr2.push('▲');
assert.deepEqual(
 arr2, ['■','●','▲']
);
arr2.pop();
assert.deepEqual(
 arr2, ['■','●']
);
```

### 31.2 在 JavaScript 中使用数组的两种方式

在 JavaScript 中有两种使用数组的方式：

+   固定布局数组：以这种方式使用，数组具有固定数量的索引元素。每个元素可以具有不同的类型。

+   序列数组：以这种方式使用，数组具有可变数量的索引元素。每个元素具有相同的类型。

实际上，这两种方式经常混合使用。

值得注意的是，序列数组非常灵活，我们可以将它们用作（传统的）数组、堆栈和队列。稍后我们会看到。

### 31.3 基本数组操作

#### 31.3.1 创建、读取、写入数组

创建数组的最佳方式是通过*数组文字*：

```js
const arr = ['a', 'b', 'c'];
```

数组文字以方括号`[]`开始和结束。它创建一个包含三个*元素*：'a'，'b'和'c'的数组。

数组文字中允许并忽略尾随逗号：

```js
const arr = [
 'a',
 'b',
 'c',
];
```

要读取数组元素，我们将索引放在方括号中（索引从零开始）：

```js
const arr = ['a', 'b', 'c'];
assert.equal(arr[0], 'a');
```

要更改数组元素，我们将索引分配给数组：

```js
const arr = ['a', 'b', 'c'];
arr[0] = 'x';
assert.deepEqual(arr, ['x', 'b', 'c']);
```

数组索引的范围为 32 位（不包括最大长度）：[0, 2³²−1]

#### 31.3.2 数组的`.length`

每个数组都有一个属性`.length`，可用于读取和更改(!)数组中的元素数量。

数组的长度始终是最高索引加一：

```js
> const arr = ['a', 'b'];
> arr.length
2
```

如果我们在长度的索引处写入数组，我们将添加一个元素：

```js
> arr[arr.length] = 'c';
> arr
[ 'a', 'b', 'c' ]
> arr.length
3
```

另一种（破坏性地）附加元素的方法是通过数组方法`.push()`：

```js
> arr.push('d');
> arr
[ 'a', 'b', 'c', 'd' ]
```

如果我们设置`.length`，我们正在修剪数组，删除元素：

```js
> arr.length = 1;
> arr
[ 'a' ]
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：通过`.push()`删除空行**

`exercises/arrays/remove_empty_lines_push_test.mjs`

#### 31.3.3 通过负索引引用元素

几种数组方法支持负索引。如果索引为负数，则将其添加到数组的长度以产生可用索引。因此，`.slice()`的以下两个调用是等效的：它们都从最后一个元素开始复制`arr`。

```js
> const arr = ['a', 'b', 'c'];
> arr.slice(-1)
[ 'c' ]
> arr.slice(arr.length - 1)
[ 'c' ]
```

##### 31.3.3.1 `.at()`: 读取单个元素（支持负索引）[ES2022]

数组方法`.at()`返回给定索引处的元素。它支持正数和负数索引（`-1`指最后一个元素，`-2`指倒数第二个元素等）：

```js
> ['a', 'b', 'c'].at(0)
'a'
> ['a', 'b', 'c'].at(-1)
'c'
```

相比之下，方括号运算符`[]`不支持负索引（并且不能更改，因为那样会破坏现有的代码）。它将它们解释为非元素属性的键：

```js
const arr = ['a', 'b', 'c'];

arr[-1] = 'non-element property';
// The Array elements didn’t change:
assert.deepEqual(
 Array.from(arr), // copy just the Array elements
 ['a', 'b', 'c']
);

assert.equal(
 arr[-1], 'non-element property'
);
```

#### 31.3.4 清除数组

清除（清空）一个数组，我们可以将其`.length`设置为零：

```js
const arr = ['a', 'b', 'c'];
arr.length = 0;
assert.deepEqual(arr, []);
```

或者我们可以将一个新的空数组赋给存储数组的变量：

```js
let arr = ['a', 'b', 'c'];
arr = [];
assert.deepEqual(arr, []);
```

后一种方法的优点是不会影响指向相同数组的其他位置。但是，如果我们确实想要为所有人重置共享的数组，那么我们需要前一种方法。

#### 31.3.5 扩展到数组文字[ES6]

在数组文字中，*扩展元素*由三个点（`...`）后跟一个表达式组成。它导致表达式被评估然后被迭代。每个迭代的值都成为额外的数组元素 - 例如：

```js
> const iterable = ['b', 'c'];
> ['a', ...iterable, 'd']
[ 'a', 'b', 'c', 'd' ]
```

这意味着我们可以使用扩展来创建数组的副本并将可迭代对象转换为数组：

```js
const original = ['a', 'b', 'c'];

const copy = [...original];

const iterable = original.keys();
assert.deepEqual(
 [...iterable], [0, 1, 2]
);
```

然而，对于前面的两种用例，我发现`Array.from()`更具有自我描述性，并更喜欢它：

```js
const copy2 = Array.from(original);

assert.deepEqual(
 Array.from(original.keys()), [0, 1, 2]
);
```

扩展也方便将数组（和其他可迭代对象）连接到数组中：

```js
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

const concatenated = [...arr1, ...arr2, 'e'];
assert.deepEqual(
 concatenated,
 ['a', 'b', 'c', 'd', 'e']);
```

由于扩展使用迭代，只有当值可迭代时才有效：

```js
> [...'abc'] // strings are iterable
[ 'a', 'b', 'c' ]
> [...123]
TypeError: 123 is not iterable
> [...undefined]
TypeError: undefined is not iterable
```

![](img/0ac255e56dc93a43365d8502301c8688.png) **扩展和`Array.from()`产生浅层副本**

通过扩展或通过`Array.from()`复制数组是浅层的：我们在新数组中获得新条目，但值与原始数组共享。浅层复制的后果在[§28.4“扩展到对象字面量（`...`）[ES2018]”](ch_objects.html#spreading-into-object-literals)中得到了证明。

#### 31.3.6 数组：列出索引和条目[ES6]

方法`.keys()`列出数组的索引：

```js
const arr = ['a', 'b'];
assert.deepEqual(
 Array.from(arr.keys()), // (A)
 [0, 1]);
```

`.keys()`返回一个可迭代对象。在 A 行，我们将该可迭代对象转换为数组。

列出数组索引与列出属性不同。前者产生数字；后者产生字符串化的数字（除了非索引属性键）：

```js
const arr = ['a', 'b'];
arr.prop = true;

assert.deepEqual(
 Object.keys(arr),
 ['0', '1', 'prop']);
```

方法`.entries()`列出数组的内容为[索引，元素]对：

```js
const arr = ['a', 'b'];
assert.deepEqual(
 Array.from(arr.entries()),
 [[0, 'a'], [1, 'b']]);
```

#### 31.3.7 是否值为数组？

以下是检查值是否为数组的两种方法：

```js
> [] instanceof Array
true
> Array.isArray([])
true
```

`instanceof`通常很好。如果值可能来自另一个*领域*，我们需要`Array.isArray()`。粗略地说，领域是 JavaScript 全局范围的一个实例。一些领域彼此隔离（例如，浏览器中的 Web Workers），但也有一些领域之间可以移动数据 - 例如，浏览器中的同源 iframe。`x instanceof Array`检查`x`的原型链，因此如果`x`是来自另一个领域的数组，则返回`false`。

`typeof`将数组归类为对象：

```js
> typeof []
'object'
```

### 31.4 `for-of`和数组[ES6]

我们已经在本书的早期遇到了`for-of`循环。本节简要回顾了如何在数组中使用它。

#### 31.4.1 `for-of`：遍历元素

以下`for-of`循环遍历数组的元素：

```js
for (const element of ['a', 'b']) {
 console.log(element);
}
// Output:
// 'a'
// 'b'
```

#### 31.4.2 `for-of`：遍历索引

这个`for-of`循环遍历数组的索引：

```js
for (const element of ['a', 'b'].keys()) {
 console.log(element);
}
// Output:
// 0
// 1
```

#### 31.4.3 `for-of`：遍历[索引，元素]对

以下`for-of`循环遐射[索引，元素]对。解构（稍后描述）为我们提供了在`for-of`头部设置`index`和`element`的便捷语法。

```js
for (const [index, element] of ['a', 'b'].entries()) {
 console.log(index, element);
}
// Output:
// 0, 'a'
// 1, 'b'
```

### 31.5 类似数组的对象

一些适用于数组的操作仅需要最低限度：值必须只是*类似数组*。类似数组的值是具有以下属性的对象：

+   `.length`：保存类似数组对象的长度。

+   `[0]`：保存索引 0 处的元素（等等）。请注意，如果我们使用数字作为属性名称，它们总是被强制转换为字符串。因此，`[0]`检索其键为`'0'`的属性的值。

例如，`Array.from()`接受类数组对象并将其转换为数组：

```js
// If we omit .length, it is interpreted as 0
assert.deepEqual(
 Array.from({}),
 []);

assert.deepEqual(
 Array.from({length:2, 0:'a', 1:'b'}),
 [ 'a', 'b' ]);
```

Array-like 对象的 TypeScript 接口是：

```js
interface ArrayLike<T> {
 length: number;
 [n: number]: T;
}
```

！[](../Images/b666ba365e94edaf0ef510fd7e12c7de.png) **类数组对象在现代 JavaScript 中相对较少**

在 ES6 之前，类数组对象很常见；现在我们很少见到它们。

### 31.6 将可迭代对象和类数组值转换为数组

将可迭代对象和类数组值转换为数组有两种常见的方法：

+   扩展为数组

+   `Array.from()`

我更喜欢后者-我觉得它更加自解释。

#### 31.6.1 通过扩展（`...`）将可迭代对象转换为数组

在数组字面量内，通过`...`扩展将任何可迭代对象转换为一系列数组元素。例如：

```js
// Get an Array-like collection from a web browser’s DOM
const domCollection = document.querySelectorAll('a');

// Alas, the collection is missing many Array methods
assert.equal('map' in domCollection, false);

// Solution: convert it to an Array
const arr = [...domCollection];
assert.deepEqual(
 arr.map(x => x.href),
 ['https://2ality.com', 'https://exploringjs.com']);
```

转换起作用是因为 DOM 集合是可迭代的。

#### 31.6.2 通过`Array.from()`将可迭代对象和类数组对象转换为数组

`Array.from()`可以以两种模式使用。

##### 31.6.2.1 `Array.from()`的模式 1：转换

第一种模式具有以下类型签名：

```js
.from<T>(iterable: Iterable<T> | ArrayLike<T>): T[]
```

`Iterable`接口显示在同步迭代章节。`ArrayLike`接口出现在本章前面。

使用单个参数，`Array.from()`将任何可迭代对象或类数组转换为数组：

```js
> Array.from(new Set(['a', 'b']))
[ 'a', 'b' ]
> Array.from({length: 2, 0:'a', 1:'b'})
[ 'a', 'b' ]
```

##### 31.6.2.2 `Array.from()`的模式 2：转换和映射

`Array.from()`的第二种模式涉及两个参数：

```js
.from<T, U>(
 iterable: Iterable<T> | ArrayLike<T>,
 mapFunc: (v: T, i: number) => U,
 thisArg?: any)
 : U[]
```

在这种模式下，`Array.from()`做了几件事：

+   它遍历`iterable`。

+   它调用`mapFunc`与每个迭代的值。可选参数`thisArg`指定了`mapFunc`的`this`。

+   它将`mapFunc`应用于每个迭代的值。

+   它将结果收集到一个新数组中并返回它。

换句话说：我们从类型为`T`的可迭代元素转换为类型为`U`的数组。

这是一个例子：

```js
> Array.from(new Set(['a', 'b']), x => x + x)
[ 'aa', 'bb' ]
```

### 31.7 创建和填充任意长度的数组

创建数组的最佳方式是通过数组字面量。但是，我们并不总是能够使用数组字面量：数组可能太大，我们可能在开发过程中不知道其长度，或者我们可能希望保持其长度的灵活性。那么我推荐以下技术来创建和可能填充数组。

#### 31.7.1 我们需要创建一个稍后完全填充的空数组吗？

```js
> new Array(3)
[ , , ,]
```

请注意，结果有三个*holes*（空槽） - 数组字面量中的最后一个逗号总是被忽略。

#### 31.7.2 我们需要创建一个填充有原始值的数组吗？

```js
> new Array(3).fill(0)
[0, 0, 0]
```

注意：如果我们使用`.fill()`与一个对象，那么每个数组元素将引用此对象（共享它）。

```js
const arr = new Array(3).fill({});
arr[0].prop = true;
assert.deepEqual(
 arr, [
 {prop: true},
 {prop: true},
 {prop: true},
 ]);
```

下一小节将解释如何修复这个问题。

#### 31.7.3 我们需要创建一个填充有对象的数组吗？

```js
> new Array(3).fill(0)
[0, 0, 0]
```

对于大尺寸，临时数组可能会消耗大量内存。以下方法没有这个缺点，但是不够自描述：

```js
> Array.from({length: 3}, () => ({}))
[{}, {}, {}]
```

我们使用临时的类数组对象而不是临时数组。

#### 31.7.4 我们需要创建一个整数范围吗？

```js
function createRange(start, end) {
 return Array.from({length: end-start}, (_, i) => i+start);
}
assert.deepEqual(
 createRange(2, 5),
 [2, 3, 4]);
```

这是一个替代的、稍微巧妙的创建以零开头的整数范围的技术：

```js
/** Returns an iterable */
function createRange(end) {
 return new Array(end).keys();
}
assert.deepEqual(
 Array.from(createRange(4)),
 [0, 1, 2, 3]);
```

这是因为`.keys()`将*holes*视为`undefined`元素并列出它们的索引。

#### 31.7.5 如果元素都是整数或浮点数，使用 Typed Array

在处理整数或浮点数数组时，我们应该考虑*Typed Arrays*，这是为此目的而创建的。

### 31.8 多维数组

JavaScript 没有真正的多维数组；我们需要求助于其元素为数组的数组：

```js
function initMultiArray(...dimensions) {
 function initMultiArrayRec(dimIndex) {
 if (dimIndex >= dimensions.length) {
 return 0;
 } else {
 const dim = dimensions[dimIndex];
 const arr = [];
 for (let i=0; i<dim; i++) {
 arr.push(initMultiArrayRec(dimIndex+1));
 }
 return arr;
 }
 }
 return initMultiArrayRec(0);
}

const arr = initMultiArray(4, 3, 2);
arr[3][2][1] = 'X'; // last in each dimension
assert.deepEqual(arr, [
 [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
 [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
 [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
 [ [ 0, 0 ], [ 0, 0 ], [ 0, 'X' ] ],
]);
```

### 31.9 更多数组特性（高级）

在本节中，我们将研究在使用数组时不经常遇到的现象。

#### 31.9.1 数组索引是（稍微特殊的）属性键

你可能会认为数组元素很特殊，因为我们是通过数字访问它们的。但是用于这样做的方括号运算符`[]`与用于访问属性的运算符相同。它将任何值（不是符号）强制转换为字符串。因此，数组元素（几乎）是正常属性（A 行），并且使用数字或字符串作为索引并不重要（B 和 C 行）：

```js
const arr = ['a', 'b'];
arr.prop = 123;
assert.deepEqual(
 Object.keys(arr),
 ['0', '1', 'prop']); // (A)

assert.equal(arr[0], 'a');  // (B)
assert.equal(arr['0'], 'a'); // (C)
```

更让事情变得更加混乱的是，这只是语言规范定义事物的方式（如果你愿意的话，这是 JavaScript 的理论）。大多数 JavaScript 引擎在幕后进行优化，并确实使用实际的整数来访问数组元素（如果你愿意的话，这是 JavaScript 的实践）。

用于数组元素的属性键（字符串！）称为[*索引*](https://tc39.github.io/ecma262/#integer-index)。如果将字符串`str`转换为 32 位无符号整数，然后再转换回原始值，则该字符串是一个索引。写成一个公式：

```js
ToString(ToUint32(str)) === str
```

##### 31.9.1.1 列出索引

在列出属性键时，索引被特殊对待 – 它们总是首先出现，并且像数字一样排序（`'2'`出现在`'10'`之前）：

```js
const arr = [];
arr.prop = true;
arr[1] = 'b';
arr[0] = 'a';

assert.deepEqual(
 Object.keys(arr),
 ['0', '1', 'prop']);
```

请注意，`.length`，`.entries()`和`.keys()`将数组索引视为数字，并忽略非索引属性：

```js
assert.equal(arr.length, 2);
assert.deepEqual(
 Array.from(arr.keys()), [0, 1]);
assert.deepEqual(
 Array.from(arr.entries()), [[0, 'a'], [1, 'b']]);
```

我们使用`Array.from()`将`.keys()`和`.entries()`返回的可迭代对象转换为数组。

#### 31.9.2 数组是字典，可以有空洞

我们在 JavaScript 中区分两种数组：

+   如果所有索引`i`，其中 0 ≤ `i` < `arr.length`，都存在，则数组`arr`是*密集的*。也就是说，索引形成一个连续的范围。

+   如果索引范围中有*空洞*，则数组是*稀疏的*。也就是说，一些索引是缺失的。

JavaScript 中的数组可以是稀疏的，因为数组实际上是从索引到值的字典。

推荐：避免空洞

到目前为止，我们只看到了密集的数组，确实建议避免空洞：它们使我们的代码更加复杂，并且数组方法处理不一致。此外，JavaScript 引擎优化了密集数组，使它们更快。

##### 31.9.2.1 创建空洞

当分配元素时，我们可以通过跳过索引来创建空洞：

```js
const arr = [];
arr[0] = 'a';
arr[2] = 'c';

assert.deepEqual(Object.keys(arr), ['0', '2']); // (A)

assert.equal(0 in arr, true); // element
assert.equal(1 in arr, false); // hole
```

在 A 行，我们使用`Object.keys()`，因为`arr.keys()`将空洞视为`undefined`元素，并且不会显示它们。

创建空洞的另一种方法是跳过数组文字中的元素：

```js
const arr = ['a', , 'c'];

assert.deepEqual(Object.keys(arr), ['0', '2']);
```

我们也可以删除数组元素：

```js
const arr = ['a', 'b', 'c'];
assert.deepEqual(Object.keys(arr), ['0', '1', '2']);
delete arr[1];
assert.deepEqual(Object.keys(arr), ['0', '2']);
```

##### 31.9.2.2 数组操作如何处理空洞？

遗憾的是，数组操作处理空洞的方式有很多不同。

一些数组操作会删除空洞：

```js
> ['a',,'b'].filter(x => true)
[ 'a', 'b' ]
```

一些数组操作会忽略空洞：

```js
> ['a', ,'a'].every(x => x === 'a')
true
```

一些数组操作会忽略但保留空洞：

```js
> ['a',,'b'].map(x => 'c')
[ 'c', , 'c' ]
```

一些数组操作将空洞视为`undefined`元素：

```js
> Array.from(['a',,'b'], x => x)
[ 'a', undefined, 'b' ]
> Array.from(['a',,'b'].entries())
[[0, 'a'], [1, undefined], [2, 'b']]
```

`Object.keys()`的工作方式与`.keys()`不同（字符串 vs. 数字，空洞没有键）：

```js
> Array.from(['a',,'b'].keys())
[ 0, 1, 2 ]
> Object.keys(['a',,'b'])
[ '0', '2' ]
```

这里没有规则需要记住。如果数组操作如何处理空洞很重要，最好的方法是在控制台中进行快速测试。

### 31.10 添加和删除元素（破坏性和非破坏性）

JavaScript 的`Array`非常灵活，更像是数组、堆栈和队列的组合。本节探讨了添加和删除数组元素的方法。大多数操作都可以进行*破坏性*（修改数组）和*非破坏性*（生成修改后的副本）。

#### 31.10.1 前置元素和数组

在下面的代码中，我们破坏性地将单个元素前置到`arr1`，并将数组前置到`arr2`：

```js
const arr1 = ['a', 'b'];
arr1.unshift('x', 'y'); // prepend single elements
assert.deepEqual(arr1, ['x', 'y', 'a', 'b']);

const arr2 = ['a', 'b'];
arr2.unshift(...['x', 'y']); // prepend Array
assert.deepEqual(arr2, ['x', 'y', 'a', 'b']);
```

扩展让我们将一个数组推入`arr2`。

通过扩展元素进行非破坏性的前置：

```js
const arr1 = ['a', 'b'];
assert.deepEqual(
 ['x', 'y', ...arr1], // prepend single elements
 ['x', 'y', 'a', 'b']);
assert.deepEqual(arr1, ['a', 'b']); // unchanged!

const arr2 = ['a', 'b'];
assert.deepEqual(
 [...['x', 'y'], ...arr2], // prepend Array
 ['x', 'y', 'a', 'b']);
assert.deepEqual(arr2, ['a', 'b']); // unchanged!
```

#### 31.10.2 附加元素和数组

在下面的代码中，我们破坏性地将单个元素附加到`arr1`，并将数组附加到`arr2`：

```js
const arr1 = ['a', 'b'];
arr1.push('x', 'y'); // append single elements
assert.deepEqual(arr1, ['a', 'b', 'x', 'y']);

const arr2 = ['a', 'b'];
arr2.push(...['x', 'y']); // (A) append Array
assert.deepEqual(arr2, ['a', 'b', 'x', 'y']);
```

扩展（`...`）让我们将一个数组推入`arr2`（A 行）。

通过扩展元素进行非破坏性的附加：

```js
const arr1 = ['a', 'b'];
assert.deepEqual(
 [...arr1, 'x', 'y'], // append single elements
 ['a', 'b', 'x', 'y']);
assert.deepEqual(arr1, ['a', 'b']); // unchanged!

const arr2 = ['a', 'b'];
assert.deepEqual(
 [...arr2, ...['x', 'y']], // append Array
 ['a', 'b', 'x', 'y']);
assert.deepEqual(arr2, ['a', 'b']); // unchanged!
```

#### 31.10.3 删除元素

这些是移除数组元素的三种破坏性方式：

```js
// Destructively remove first element:
const arr1 = ['a', 'b', 'c'];
assert.equal(arr1.shift(), 'a');
assert.deepEqual(arr1, ['b', 'c']);

// Destructively remove last element:
const arr2 = ['a', 'b', 'c'];
assert.equal(arr2.pop(), 'c');
assert.deepEqual(arr2, ['a', 'b']);

// Remove one or more elements anywhere:
const arr3 = ['a', 'b', 'c', 'd'];
assert.deepEqual(arr3.splice(1, 2), ['b', 'c']);
assert.deepEqual(arr3, ['a', 'd']);
```

`.splice()`在本章末尾的快速参考中有更详细的介绍。

通过剩余元素的解构，我们可以非破坏性地从数组的开头移除元素（解构在后面有介绍）。

```js
const arr1 = ['a', 'b', 'c'];
// Ignore first element, extract remaining elements
const [, ...arr2] = arr1;

assert.deepEqual(arr2, ['b', 'c']);
assert.deepEqual(arr1, ['a', 'b', 'c']); // unchanged!
```

遗憾的是，剩余元素必须在数组中的最后。因此，我们只能用它来提取后缀。

！[](../Images/90f73f1851c5b1baf43cb746913c09e6.png) **练习：通过数组实现队列**

`exercises/arrays/queue_via_array_test.mjs`

### 31.11 方法：迭代和转换（`.find()`，`.map()`，`.filter()`等）

在本节中，我们将介绍用于迭代数组和转换数组的数组方法。

#### 31.11.1 迭代和转换方法的回调

所有迭代和转换方法都使用回调。前者将所有迭代值传递给它们的回调；后者询问它们的回调如何转换数组。

这些回调的类型签名如下：

```js
callback: (value: T, index: number, array: Array<T>) => boolean
```

也就是说，回调有三个参数（可以自由忽略其中任何一个）：

+   `value`是最重要的。此参数保存当前正在处理的迭代值。

+   `index`还可以告诉回调迭代值的索引。

+   `array`指向当前数组（方法调用的接收者）。有些算法需要引用整个数组，例如搜索答案。此参数使我们能够为这些算法编写可重用的回调。

回调预期返回的内容取决于传递给它的方法。可能的情况包括：

+   `.map()`用其回调返回的值填充其结果：

    ```js
    > ['a', 'b', 'c'].map(x => x + x)
    [ 'aa', 'bb', 'cc' ]
    ```

+   `.find()`返回其回调返回`true`的第一个数组元素：

    ```js
    > ['a', 'bb', 'ccc'].find(str => str.length >= 2)
    'bb'
    ```

这两种方法稍后将更详细地描述。

#### 31.11.2 搜索元素：`.find()`，`.findIndex()`

`.find()`返回其回调返回真值的第一个元素（如果找不到任何内容，则返回`undefined`）：

```js
> [6, -5, 8].find(x => x < 0)
-5
> [6, 5, 8].find(x => x < 0)
undefined
```

`.findIndex()`返回其回调返回真值的第一个元素的索引（如果找不到任何内容，则返回`-1`）：

```js
> [6, -5, 8].findIndex(x => x < 0)
1
> [6, 5, 8].findIndex(x => x < 0)
-1
```

`.findIndex()`可以实现如下：

```js
function findIndex(arr, callback) {
 for (const [i, x] of arr.entries()) {
 if (callback(x, i, arr)) {
 return i;
 }
 }
 return -1;
}
```

#### 31.11.3 `.map()`：在给元素新值的同时复制

`.map()`返回接收者的修改副本。副本的元素是将`map`的回调应用于接收者的元素的结果。

所有这些都可以通过示例更容易理解：

```js
> [1, 2, 3].map(x => x * 3)
[ 3, 6, 9 ]
> ['how', 'are', 'you'].map(str => str.toUpperCase())
[ 'HOW', 'ARE', 'YOU' ]
> [true, true, true].map((_x, index) => index)
[ 0, 1, 2 ]
```

`.map()`可以实现如下：

```js
function map(arr, mapFunc) {
 const result = [];
 for (const [i, x] of arr.entries()) {
 result.push(mapFunc(x, i, arr));
 }
 return result;
}
```

！[](../Images/90f73f1851c5b1baf43cb746913c09e6.png) **练习：通过`.map()`编号行**

`exercises/arrays/number_lines_test.mjs`

#### 31.11.4 `.flatMap()`：映射到零个或多个值

`Array<T>.prototype.flatMap()`的类型签名是：

```js
.flatMap<U>(
 callback: (value: T, index: number, array: T[]) => U|Array<U>,
 thisValue?: any
): U[]
```

`.map()`和`.flatMap()`都接受一个函数`callback`作为参数，控制如何将输入数组转换为输出数组：

+   使用`.map()`，每个输入数组元素都被转换为一个输出元素。也就是说，`callback`返回一个单一值。

+   使用`.flatMap()`，每个输入数组元素都被转换为零个或多个输出元素。也就是说，`callback`返回一个值的数组（也可以返回非数组值，但这很少见）。

这是`.flatMap()`的实际应用：

```js
> ['a', 'b', 'c'].flatMap(x => [x,x])
[ 'a', 'a', 'b', 'b', 'c', 'c' ]
> ['a', 'b', 'c'].flatMap(x => [x])
[ 'a', 'b', 'c' ]
> ['a', 'b', 'c'].flatMap(x => [])
[]
```

在探讨如何实现此方法之前，我们将首先考虑使用情况。

##### 31.11.4.1 用例：同时过滤和映射

数组方法`.map()`的结果始终与调用它的数组长度相同。也就是说，它的回调不能跳过它不感兴趣的数组元素。`.flatMap()`具有这样做的能力，在下一个示例中非常有用。

我们将使用以下函数`processArray()`创建一个数组，然后通过`.flatMap()`进行过滤和映射：

```js
function processArray(arr, callback) {
 return arr.map(x => {
 try {
 return { value: callback(x) };
 } catch (e) {
 return { error: e };
 }
 });
}
```

接下来，我们通过`processArray()`创建一个数组`results`：

```js
const results = processArray([1, -5, 6], throwIfNegative);
assert.deepEqual(results, [
 { value: 1 },
 { error: new Error('Illegal value: -5') },
 { value: 6 },
]);

function throwIfNegative(value) {
 if (value < 0) {
 throw new Error('Illegal value: '+value);
 }
 return value;
}
```

现在我们可以使用`.flatMap()`来提取`results`中的值或错误：

```js
const values = results.flatMap(
 result => result.value ? [result.value] : []);
assert.deepEqual(values, [1, 6]);

const errors = results.flatMap(
 result => result.error ? [result.error] : []);
assert.deepEqual(errors, [new Error('Illegal value: -5')]);
```

##### 31.11.4.2 用例：将单个输入值映射到多个输出值

数组方法`.map()`将每个输入数组元素映射到一个输出元素。但是如果我们想要将其映射到多个输出元素呢？

这在以下示例中变得必要：

```js
> stringsToCodePoints(['many', 'a', 'moon'])
['m', 'a', 'n', 'y', 'a', 'm', 'o', 'o', 'n']
```

我们想要将字符串数组转换为 Unicode 字符（代码点）数组。以下函数通过`.flatMap()`实现了这一点：

```js
function stringsToCodePoints(strs) {
 return strs.flatMap(str => Array.from(str));
}
```

##### 31.11.4.3 一个简单的实现

我们可以如下实现`.flatMap()`。注意：这个实现比内置版本更简单，例如，执行了更多的检查。

```js
function flatMap(arr, mapFunc) {
 const result = [];
 for (const [index, elem] of arr.entries()) {
 const x = mapFunc(elem, index, arr);
 // We allow mapFunc() to return non-Arrays
 if (Array.isArray(x)) {
 result.push(...x);
 } else {
 result.push(x);
 }
 }
 return result;
}
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：`.flatMap()`**

+   `exercises/arrays/convert_to_numbers_test.mjs`

+   `exercises/arrays/replace_objects_test.mjs`

#### 31.11.5 `.filter()`: 仅保留一些元素

数组方法`.filter()`返回一个数组，其中收集了回调返回真值的所有元素。

例如：

```js
> [-1, 2, 5, -7, 6].filter(x => x >= 0)
[ 2, 5, 6 ]
> ['a', 'b', 'c', 'd'].filter((_x,i) => (i%2)===0)
[ 'a', 'c' ]
```

`.filter()`可以如下实现：

```js
function filter(arr, filterFunc) {
 const result = [];
 for (const [i, x] of arr.entries()) {
 if (filterFunc(x, i, arr)) {
 result.push(x);
 }
 }
 return result;
}
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：通过`.filter()`删除空行**

`exercises/arrays/remove_empty_lines_filter_test.mjs`

#### 31.11.6 `.reduce()`: 从数组中派生值（高级）

方法`.reduce()`是计算数组`arr`的“摘要”的强大工具。摘要可以是任何类型的值：

+   一个数字。例如，`arr`的所有元素的总和。

+   一个数组。例如，`arr`的副本，其中每个元素都是原始元素的两倍。

+   等等。

`reduce`在函数式编程中也被称为`foldl`（“从左向右折叠”），并且在那里很受欢迎。一个警告是它可能使代码难以理解。

`.reduce()`具有以下类型签名（在`Array<T>`中）：

```js
.reduce<U>(
 callback: (accumulator: U, element: T, index: number, array: T[]) => U,
 init?: U)
 : U
```

`T`是数组元素的类型，`U`是摘要的类型。这两者可能相同，也可能不同。`accumulator`只是“summary”的另一个名称。

为了计算数组`arr`的摘要，`.reduce()`将所有数组元素逐个传递给它的回调函数：

```js
const accumulator_0 = callback(init, arr[0]);
const accumulator_1 = callback(accumulator_0, arr[1]);
const accumulator_2 = callback(accumulator_1, arr[2]);
// Etc.
```

`callback`将先前计算的摘要（存储在其参数`accumulator`中）与当前数组元素组合，并返回下一个`accumulator`。`.reduce()`的结果是最终的累加器 - 在访问所有元素后`callback`的最后结果。

换句话说：`callback` 承担了大部分工作；`.reduce()` 只是以一种有用的方式调用它。

我们可以说回调将数组元素折叠到累加器中。这就是为什么这个操作在函数式编程中被称为“折叠”的原因。

##### 31.11.6.1 第一个例子

让我们看一个`.reduce()`的示例：函数`addAll()`计算数组`arr`中所有数字的总和。

```js
function addAll(arr) {
 const startSum = 0;
 const callback = (sum, element) => sum + element;
 return arr.reduce(callback, startSum);
}
assert.equal(addAll([1,  2, 3]), 6); // (A)
assert.equal(addAll([7, -4, 2]), 5);
```

在这种情况下，累加器保存了`callback`已经访问过的所有数组元素的总和。

在 A 行中，结果`6`是如何从数组中派生出来的？通过以下对`callback`的调用：

```js
callback(0, 1) --> 1
callback(1, 2) --> 3
callback(3, 3) --> 6
```

注：

+   第一个参数是当前累加器（从`.reduce()`的参数`init`开始）。

+   第二个参数是当前数组元素。

+   结果是下一个累加器。

+   `callback`的最后结果也是`.reduce()`的结果。

或者，我们可以通过`for-of`循环来实现`addAll()`：

```js
function addAll(arr) {
 let sum = 0;
 for (const element of arr) {
 sum = sum + element;
 }
 return sum;
}
```

很难说哪种实现“更好”：基于`.reduce()`的实现更加简洁，而基于`for-of`的实现可能更容易理解 - 尤其是对于不熟悉函数式编程的人来说。

##### 31.11.6.2 通过`.reduce()`查找索引的示例

以下函数是数组方法`.indexOf()`的实现。它返回给定的`searchValue`在数组`arr`中第一次出现的索引：

```js
const NOT_FOUND = -1;
function indexOf(arr, searchValue) {
 return arr.reduce(
 (result, elem, index) => {
 if (result !== NOT_FOUND) {
 // We have already found something: don’t change anything
 return result;
 } else if (elem === searchValue) {
 return index;
 } else {
 return NOT_FOUND;
 }
 },
 NOT_FOUND);
}
assert.equal(indexOf(['a', 'b', 'c'], 'b'), 1);
assert.equal(indexOf(['a', 'b', 'c'], 'x'), -1);
```

`.reduce()`的一个限制是我们无法提前完成（在`for-of`循环中，我们可以`break`）。在这里，一旦找到结果，我们总是立即返回结果。

##### 31.11.6.3 例子：数组元素加倍

函数`double(arr)`返回`inArr`的副本，其中的每个元素都乘以 2：

```js
function double(inArr) {
 return inArr.reduce(
 (outArr, element) => {
 outArr.push(element * 2);
 return outArr;
 },
 []);
}
assert.deepEqual(
 double([1, 2, 3]),
 [2, 4, 6]);
```

我们通过将其推入来修改初始值`[]`。`double()`的非破坏性、更加功能性的版本如下：

```js
function double(inArr) {
 return inArr.reduce(
 // Don’t change `outArr`, return a fresh Array
 (outArr, element) => [...outArr, element * 2],
 []);
}
assert.deepEqual(
 double([1, 2, 3]),
 [2, 4, 6]);
```

这个版本更加优雅，但也更慢，使用的内存更多。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：`.reduce()`**

+   通过`.reduce()`进行`map()`：`exercises/arrays/map_via_reduce_test.mjs`

+   通过`.reduce()`进行`filter()`：`exercises/arrays/filter_via_reduce_test.mjs`

+   通过`.reduce()`计算`countMatches()`：`exercises/arrays/count_matches_via_reduce_test.mjs`

### 31.12 `.sort()`: 排序数组

`.sort()`具有以下类型定义：

```js
sort(compareFunc?: (a: T, b: T) => number): this
```

默认情况下，`.sort()`对元素的字符串表示进行排序。这些表示通过`<`进行比较。这个操作符进行*词典顺序*比较（第一个字符最重要）。我们可以看到在对数字进行排序时：

```js
> [200, 3, 10].sort()
[ 10, 200, 3 ]
```

在对人类语言字符串进行排序时，我们需要意识到它们是根据它们的代码单元值（字符代码）进行比较的：

```js
> ['pie', 'cookie', 'éclair', 'Pie', 'Cookie', 'Éclair'].sort()
[ 'Cookie', 'Pie', 'cookie', 'pie', 'Éclair', 'éclair' ]
```

所有无重音的大写字母都排在所有无重音的小写字母之前，后者排在所有重音字母之前。如果我们想要人类语言的正确排序，我们可以使用`Intl`，[JavaScript 国际化 API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)。

`.sort()`就地排序；它会更改并返回其接收者：

```js
> const arr = ['a', 'c', 'b'];
> arr.sort() === arr
true
> arr
[ 'a', 'b', 'c' ]
```

#### 31.12.1 自定义排序顺序

我们可以通过参数`compareFunc`自定义排序顺序，该参数必须返回一个数字，即：

+   如果`a < b`则为负

+   如果`a === b`则为零

+   如果`a > b`则为正

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **记住这些规则的提示**

负数*小于*零（等等）。

#### 31.12.2 排序数字

我们可以使用这个辅助函数来排序数字：

```js
function compareNumbers(a, b) {
 if (a < b) {
 return -1;
 } else if (a === b) {
 return 0;
 } else {
 return 1;
 }
}
assert.deepEqual(
 [200, 3, 10].sort(compareNumbers),
 [3, 10, 200]);
```

以下是一个快速而肮脏的替代方法。

```js
> [200, 3, 10].sort((a,b) => a - b)
[ 3, 10, 200 ]
```

这种方法的缺点是：

+   这是神秘的。

+   如果`a-b`变成一个很大的正数或负数，就会有数值溢出或下溢的风险。

#### 31.12.3 排序对象

如果我们想要对对象进行排序，我们还需要使用比较函数。例如，以下代码显示了如何按年龄对对象进行排序。

```js
const arr = [ {age: 200}, {age: 3}, {age: 10} ];
assert.deepEqual(
 arr.sort((obj1, obj2) => obj1.age - obj2.age),
 [{ age: 3 }, { age: 10 }, { age: 200 }] );
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：按名称对对象进行排序**

`exercises/arrays/sort_objects_test.mjs`

### 31.13 `Array`快速参考

传奇：

+   `R`：方法不改变数组（非破坏性）。

+   `W`：方法改变了数组（破坏性）。

#### 31.13.1 `new Array()`

`new Array(n)`创建一个包含`n`个空位的长度为`n`的数组：

```js
// Trailing commas are always ignored.
// Therefore: number of commas = number of holes
assert.deepEqual(new Array(3), [,,,]);
```

`new Array()`创建一个空数组。但是，我建议始终使用`[]`。

#### 31.13.2 `Array`的静态方法

+   `Array.from<T>(iterable: Iterable<T> | ArrayLike<T>): T[]` ^([ES6])

+   `Array.from<T,U>(iterable: Iterable<T> | ArrayLike<T>, mapFunc: (v: T, k: number) => U, thisArg?: any): U[]` ^([ES6])

    将可迭代对象或类数组对象转换为数组。可选地，输入值可以在添加到输出数组之前通过`mapFunc`进行转换。

    例子：

    ```js
    > Array.from(new Set(['a', 'b'])) // iterable
    [ 'a', 'b' ]
    > Array.from({length: 2, 0:'a', 1:'b'}) // Array-like object
    [ 'a', 'b' ]
    ```

+   `Array.of<T>(...items: T[]): T[]` ^([ES6])

    这个静态方法主要用于`Array`的子类，它作为自定义数组文字：

    ```js
    class MyArray extends Array {}

    assert.equal(
     MyArray.of('a', 'b') instanceof MyArray, true);
    ```

#### 31.13.3 `Array.prototype`的方法

+   `.at(index: number): T | undefined` ^([R, ES2022])

    返回`index`处的数组元素。如果`index`为负，则在使用之前将其添加到`.length`（`-1`变为`this.length-1`等）。

    ```js
    > ['a', 'b', 'c'].at(0)
    'a'
    > ['a', 'b', 'c'].at(-1)
    'c'
    ```

+   `.concat(...items: Array<T[] | T>): T[]` ^([R, ES3])

    返回一个新的数组，该数组是接收者和所有`items`的连接。非数组参数（例如以下示例中的`'b'`）将被视为具有单个元素的数组。

    ```js
    > ['a'].concat('b', ['c', 'd'])
    [ 'a', 'b', 'c', 'd' ]
    ```

+   `.copyWithin(target: number, start: number, end=this.length): this` ^([W, ES6])

    将索引范围从（包括）`start`到（不包括）`end`的元素复制到以`target`开头的索引。重叠部分被正确处理。

    ```js
    > ['a', 'b', 'c', 'd'].copyWithin(0, 2, 4)
    [ 'c', 'd', 'c', 'd' ]
    ```

    如果`start`或`end`为负数，则将其添加到`.length`。

+   `.entries(): Iterable<[number, T]>` ^([R, ES6])

    返回一个可迭代的[index, element]对。

    ```js
    > Array.from(['a', 'b'].entries())
    [ [ 0, 'a' ], [ 1, 'b' ] ]
    ```

+   `.every(callback: (value: T, index: number, array: Array<T>) => boolean, thisArg?: any): boolean` ^([R, ES5])

    如果`callback`对每个元素返回一个真值，则返回`true`。否则，返回`false`。一旦接收到一个假值，它就会停止。这个方法对应于数学中的全称量化（“对于所有”，`∀`）。

    ```js
    > [1, 2, 3].every(x => x > 0)
    true
    > [1, -2, 3].every(x => x > 0)
    false
    ```

    相关方法：`.some()`（“存在”）。

+   `.fill(value: T, start=0, end=this.length): this` ^([W, ES6])

    将`value`分配给`start`和`end`之间（包括`start`但不包括`end`）的每个索引。

    ```js
    > [0, 1, 2].fill('a')
    [ 'a', 'a', 'a' ]
    ```

    注意：不要使用这个方法来用对象`obj`填充数组；然后每个元素将引用`obj`（共享它）。在这种情况下，最好使用`Array.from()`。

+   `.filter(callback: (value: T, index: number, array: Array<T>) => any, thisArg?: any): T[]` ^([R, ES5])

    返回一个只包含`callback`返回真值的元素的数组。

    ```js
    > [1, -2, 3].filter(x => x > 0)
    [ 1, 3 ]
    ```

+   `.find(predicate: (value: T, index: number, obj: T[]) => boolean, thisArg?: any): T | undefined` ^([R, ES6])

    结果是`predicate`返回真值的第一个元素。如果没有这样的元素，则结果是`undefined`。

    ```js
    > [1, -2, 3].find(x => x < 0)
    -2
    > [1, 2, 3].find(x => x < 0)
    undefined
    ```

+   `.findIndex(predicate: (value: T, index: number, obj: T[]) => boolean, thisArg?: any): number` ^([R, ES6])

    结果是`predicate`返回真值的第一个元素的索引。如果没有这样的元素，则结果是`-1`。

    ```js
    > [1, -2, 3].findIndex(x => x < 0)
    1
    > [1, 2, 3].findIndex(x => x < 0)
    -1
    ```

+   `.flat(depth = 1): any[]` ^([R, ES2019])

    “展平”一个数组：它会进入嵌套在输入数组中的数组，并创建一个副本，其中它发现的所有值都被移动到顶层，直到达到`depth`级别或更低级别。

    ```js
    > [ 1,2, [3,4], [[5,6]] ].flat(0) // no change
    [ 1, 2, [3,4], [[5,6]] ]

    > [ 1,2, [3,4], [[5,6]] ].flat(1)
    [1, 2, 3, 4, [5,6]]

    > [ 1,2, [3,4], [[5,6]] ].flat(2)
    [1, 2, 3, 4, 5, 6]
    ```

+   `.flatMap<U>(callback: (value: T, index: number, array: T[]) => U|Array<U>, thisValue?: any): U[]` ^([R, ES2019])

    结果是通过调用原始数组的每个元素的`callback()`并连接它返回的数组产生的。

    ```js
    > ['a', 'b', 'c'].flatMap(x => [x,x])
    [ 'a', 'a', 'b', 'b', 'c', 'c' ]
    > ['a', 'b', 'c'].flatMap(x => [x])
    [ 'a', 'b', 'c' ]
    > ['a', 'b', 'c'].flatMap(x => [])
    []
    ```

+   `.forEach(callback: (value: T, index: number, array: Array<T>) => void, thisArg?: any): void` ^([R, ES5])

    为每个元素调用`callback`。

    ```js
    ['a', 'b'].forEach((x, i) => console.log(x, i))

    // Output:
    // 'a', 0
    // 'b', 1
    ```

    通常使用`for-of`循环更好：它更快，支持`break`，并且可以迭代任意可迭代对象。

+   `.includes(searchElement: T, fromIndex=0): boolean` ^([R, ES2016])

    如果接收器有一个值为`searchElement`的元素，则返回`true`，否则返回`false`。搜索从索引`fromIndex`开始。

    ```js
    > [0, 1, 2].includes(1)
    true
    > [0, 1, 2].includes(5)
    false
    ```

+   `.indexOf(searchElement: T, fromIndex=0): number` ^([R, ES5])

    返回第一个严格等于`searchElement`的元素的索引。如果没有这样的元素，则返回`-1`。从索引`fromIndex`开始搜索，接着访问更高的索引。

    ```js
    > ['a', 'b', 'a'].indexOf('a')
    0
    > ['a', 'b', 'a'].indexOf('a', 1)
    2
    > ['a', 'b', 'a'].indexOf('c')
    -1
    ```

+   `.join(separator = ','): string` ^([R, ES1])

    通过连接所有元素的字符串表示来创建一个字符串，用`separator`分隔它们。

    ```js
    > ['a', 'b', 'c'].join('##')
    'a##b##c'
    > ['a', 'b', 'c'].join()
    'a,b,c'
    ```

+   `.keys(): Iterable<number>` ^([R, ES6])

    返回接收器的键的可迭代对象。

    ```js
    > Array.from(['a', 'b'].keys())
    [ 0, 1 ]
    ```

+   `.lastIndexOf(searchElement: T, fromIndex=this.length-1): number` ^([R, ES5])

    返回最后一个严格等于`searchElement`的元素的索引。如果没有这样的元素，则返回`-1`。从索引`fromIndex`开始搜索，接着访问更低的索引。

    ```js
    > ['a', 'b', 'a'].lastIndexOf('a')
    2
    > ['a', 'b', 'a'].lastIndexOf('a', 1)
    0
    > ['a', 'b', 'a'].lastIndexOf('c')
    -1
    ```

+   `.map<U>(mapFunc: (value: T, index: number, array: Array<T>) => U, thisArg?: any): U[]` ^([R, ES5])

    返回一个新的数组，其中每个元素都是将`mapFunc`应用于接收器的相应元素的结果。

    ```js
    > [1, 2, 3].map(x => x * 2)
    [ 2, 4, 6 ]
    > ['a', 'b', 'c'].map((x, i) => i)
    [ 0, 1, 2 ]
    ```

+   `.pop(): T | undefined` ^([W, ES3])

    移除并返回接收器的最后一个元素。也就是说，它将接收器的末尾视为一个堆栈。与`.push()`相反。

    ```js
    > const arr = ['a', 'b', 'c'];
    > arr.pop()
    'c'
    > arr
    [ 'a', 'b' ]
    ```

+   `.push(...items: T[]): number` ^([W, ES3])

    向接收器的末尾添加零个或多个`items`。也就是说，它将接收器的末尾视为一个堆栈。返回值是更改后接收器的长度。与`.pop()`相反。

    ```js
    > const arr = ['a', 'b'];
    > arr.push('c', 'd')
    4
    > arr
    [ 'a', 'b', 'c', 'd' ]
    ```

    我们可以通过展开（`...`）数组来推送一个数组作为参数：

    ```js
    > const arr = ['x'];
    > arr.push(...['y', 'z'])
    3
    > arr
    [ 'x', 'y', 'z' ] 
    ```

+   `.reduce<U>(callback: (accumulator: U, element: T, index: number, array: T[]) => U, init?: U): U` ^([R, ES5])

    此方法生成接收者的摘要：它将所有数组元素提供给`callback`，`callback`将当前摘要（在参数`accumulator`中）与当前数组元素组合并返回下一个`accumulator`：

    ```js
    const accumulator_0 = callback(init, arr[0]);
    const accumulator_1 = callback(accumulator_0, arr[1]);
    const accumulator_2 = callback(accumulator_1, arr[2]);
    // Etc.
    ```

    `.reduce()`的结果是在访问所有数组元素后`callback`的最后结果。

    ```js
    > [1, 2, 3].reduce((accu, x) => accu + x, 0)
    6
    > [1, 2, 3].reduce((accu, x) => accu + String(x), '')
    '123'
    ```

    如果没有提供`init`，则使用索引 0 处的数组元素，并首先访问索引 1 处的元素。因此，数组的长度必须至少为 1。

+   `.reduceRight<U>(callback: (accumulator: U, element: T, index: number, array: T[]) => U, init?: U): U` ^([R, ES5])

    与`.reduce()`类似，但是以相反的顺序访问数组元素，从最后一个元素开始。

    ```js
    > [1, 2, 3].reduceRight((accu, x) => accu + String(x), '')
    '321'
    ```

+   `.reverse(): this` ^([W, ES1])

    重新排列接收者的元素，使其按相反顺序排列，然后返回接收者。

    ```js
    > const arr = ['a', 'b', 'c'];
    > arr.reverse()
    [ 'c', 'b', 'a' ]
    > arr
    [ 'c', 'b', 'a' ]
    ```

+   `.shift(): T | undefined` ^([W, ES3])

    删除并返回接收者的第一个元素。与`.unshift()`相反。

    ```js
    > const arr = ['a', 'b', 'c'];
    > arr.shift()
    'a'
    > arr
    [ 'b', 'c' ]
    ```

+   `.slice(start=0, end=this.length): T[]` ^([R, ES3])

    返回一个新的数组，其中包含接收者的索引位于`start`和`end`之间（包括`start`但不包括`end`）的元素。

    ```js
    > ['a', 'b', 'c', 'd'].slice(1, 3)
    [ 'b', 'c' ]
    > ['a', 'b'].slice() // shallow copy
    [ 'a', 'b' ]
    ```

    允许负索引，并添加到`.length`：

    ```js
    > ['a', 'b', 'c'].slice(-2)
    [ 'b', 'c' ]
    ```

+   `.some(callback: (value: T, index: number, array: Array<T>) => boolean, thisArg?: any): boolean` ^([R, ES5])

    如果`callback`对至少一个元素返回真值，则返回`true`。否则，它返回`false`。一旦收到真值，它就会停止。此方法对应于数学中的存在量词（“存在”，`∃`）。

    ```js
    > [1, 2, 3].some(x => x < 0)
    false
    > [1, -2, 3].some(x => x < 0)
    true
    ```

    相关方法：`.every()`（“对于所有”）。

+   `.sort(compareFunc?: (a: T, b: T) => number): this` ^([W, ES1])

    对接收者进行排序并返回。默认情况下，它对元素的字符串表示进行排序。它按字典顺序和字符的代码单元值（字符代码）进行排序：

    ```js
    > ['pie', 'cookie', 'éclair', 'Pie', 'Cookie', 'Éclair'].sort()
    [ 'Cookie', 'Pie', 'cookie', 'pie', 'Éclair', 'éclair' ]
    > [200, 3, 10].sort()
    [ 10, 200, 3 ]
    ```

    我们可以通过`compareFunc`自定义排序顺序，它返回一个数字：

    +   如果`a < b`，则为负数

    +   如果`a === b`，则为零

    +   如果`a > b`，则为正数

    对于对数字进行排序的技巧（存在数值溢出或下溢的风险）：

    ```js
    > [200, 3, 10].sort((a, b) => a - b)
    [ 3, 10, 200 ]
    ```

    ![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **`.sort()`是稳定的**

    自 ECMAScript 2019 以来，排序保证是稳定的：如果元素被排序视为相等，则排序不会改变这些元素的顺序（相对于彼此）。

+   `.splice(start: number, deleteCount=this.length-start, ...items: T[]): T[]` ^([W, ES3])

    在索引`start`处，它删除`deleteCount`个元素并插入`items`。它返回已删除的元素。

    ```js
    > const arr = ['a', 'b', 'c', 'd'];
    > arr.splice(1, 2, 'x', 'y')
    [ 'b', 'c' ]
    > arr
    [ 'a', 'x', 'y', 'd' ]
    ```

    `start`可以为负数，如果是负数，则将其添加到`.length`：

    ```js
    > ['a', 'b', 'c'].splice(-2, 2)
    [ 'b', 'c' ]
    ```

+   `.toString(): string` ^([R, ES1])

    通过`String()`将所有元素转换为字符串，用逗号分隔它们并返回结果。

    ```js
    > [1, 2, 3].toString()
    '1,2,3'
    > ['1', '2', '3'].toString()
    '1,2,3'
    > [].toString()
    ''
    ```

+   `.unshift(...items: T[]): number` ^([W, ES3])

    将`items`插入到接收者的开头，并在此修改后返回其长度。

    ```js
    > const arr = ['c', 'd'];
    > arr.unshift('e', 'f')
    4
    > arr
    [ 'e', 'f', 'c', 'd' ]
    ```

+   `.values(): Iterable<T>` ^([R, ES6])

    返回接收者的值的可迭代对象。

    ```js
    > Array.from(['a', 'b'].values())
    [ 'a', 'b' ]
    ```

#### 31.13.4 来源

+   [TypeScript 的内置类型](https://github.com/Microsoft/TypeScript/blob/master/lib/)

+   [JavaScript 的 MDN Web 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

+   [ECMAScript 语言规范](https://tc39.github.io/ecma262/)

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/22)
