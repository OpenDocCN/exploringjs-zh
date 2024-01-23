# 33 地图（Map）

> 原文：[`exploringjs.com/impatient-js/ch_maps.html`](https://exploringjs.com/impatient-js/ch_maps.html)

* * *

+   33.1 使用地图

    +   33.1.1 创建地图

    +   33.1.2 复制地图

    +   33.1.3 处理单个条目

    +   33.1.4 确定地图的大小并清除它

    +   33.1.5 获取地图的键和值

    +   33.1.6 获取地图的条目

    +   33.1.7 按插入顺序列出：条目、键、值

    +   33.1.8 在地图和对象之间转换

+   33.2 示例：计算字符

+   33.3 关于地图键的一些细节（高级）

    +   33.3.1 哪些键被视为相等？

+   33.4 地图操作缺失

    +   33.4.1 映射和过滤地图

    +   33.4.2 组合地图

+   33.5 快速参考：`Map<K,V>`

    +   33.5.1 构造函数

    +   33.5.2 `Map<K,V>.prototype`：处理单个条目

    +   33.5.3 `Map<K,V>.prototype`：处理所有条目

    +   33.5.4 `Map<K,V>.prototype`：迭代和循环

    +   33.5.5 本节的来源

+   33.6 常见问题：地图

    +   33.6.1 何时应该使用地图，何时应该使用对象？

    +   33.6.2 何时我会使用对象作为地图中的键？

    +   33.6.3 地图为什么保留插入顺序？

    +   33.6.4 地图为什么有`.size`，而数组有`.length`？

* * *

在 ES6 之前，JavaScript 没有字典的数据结构，并且（滥用）对象作为从字符串到任意值的字典。ES6 引入了地图，这是从任意值到任意值的字典。

### 33.1 使用地图

`Map`的实例将键映射到值。单个键值映射称为*条目*。

#### 33.1.1 创建地图

有三种常见的创建地图的方法。

首先，您可以使用没有任何参数的构造函数创建一个空地图：

```js
const emptyMap = new Map();
assert.equal(emptyMap.size, 0);
```

其次，您可以将可迭代对象（例如数组）传递给构造函数的键值“对”（具有两个元素的数组）：

```js
const map = new Map([
 [1, 'one'],
 [2, 'two'],
 [3, 'three'], // trailing comma is ignored
]);
```

第三，`.set()`方法向地图添加条目，并且是可链接的：

```js
const map = new Map()
 .set(1, 'one')
 .set(2, 'two')
 .set(3, 'three');
```

#### 33.1.2 复制地图

正如我们将在后面看到的，地图也是键值对的可迭代对象。因此，您可以使用构造函数创建地图的副本。该副本是*浅层*的：键和值是相同的；它们不是重复的。

```js
const original = new Map()
 .set(false, 'no')
 .set(true, 'yes');

const copy = new Map(original);
assert.deepEqual(original, copy);
```

#### 33.1.3 处理单个条目

`.set()`和`.get()`用于写入和读取值（给定键）。

```js
const map = new Map();

map.set('foo', 123);

assert.equal(map.get('foo'), 123);
// Unknown key:
assert.equal(map.get('bar'), undefined);
// Use the default value '' if an entry is missing:
assert.equal(map.get('bar') ?? '', '');
```

`.has()`检查地图是否具有具有给定键的条目。`.delete()`删除条目。

```js
const map = new Map([['foo', 123]]);

assert.equal(map.has('foo'), true);
assert.equal(map.delete('foo'), true)
assert.equal(map.has('foo'), false)
```

#### 33.1.4 确定地图的大小并清除它

`.size`包含地图中的条目数。`.clear()`删除地图的所有条目。

```js
const map = new Map()
 .set('foo', true)
 .set('bar', false)
;

assert.equal(map.size, 2)
map.clear();
assert.equal(map.size, 0)
```

#### 33.1.5 获取地图的键和值

`.keys()`返回地图的键的可迭代对象：

```js
const map = new Map()
 .set(false, 'no')
 .set(true, 'yes')
;

for (const key of map.keys()) {
 console.log(key);
}
// Output:
// false
// true
```

我们使用`Array.from()`将`.keys()`返回的可迭代对象转换为数组：

```js
assert.deepEqual(
 Array.from(map.keys()),
 [false, true]);
```

`.values()`的工作原理类似于`.keys()`，但是针对值而不是键。

#### 33.1.6 获取地图的条目

`.entries()`返回地图的条目的可迭代对象。

```js
const map = new Map()
 .set(false, 'no')
 .set(true, 'yes')
;

for (const entry of map.entries()) {
 console.log(entry);
}
// Output:
// [false, 'no']
// [true, 'yes']
```

`Array.from()` 将 `.entries()` 返回的可迭代对象转换为数组：

```js
assert.deepEqual(
 Array.from(map.entries()),
 [[false, 'no'], [true, 'yes']]);
```

地图实例也是条目的可迭代对象。在下面的代码中，我们使用 解构 来访问 `map` 的键和值：

```js
for (const [key, value] of map) {
 console.log(key, value);
}
// Output:
// false, 'no'
// true, 'yes'
```

#### 33.1.7 按插入顺序列出：条目、键、值

地图记录了条目创建的顺序，并在列出条目、键或值时遵守该顺序：

```js
const map1 = new Map([
 ['a', 1],
 ['b', 2],
]);
assert.deepEqual(
 Array.from(map1.keys()), ['a', 'b']);

const map2 = new Map([
 ['b', 2],
 ['a', 1],
]);
assert.deepEqual(
 Array.from(map2.keys()), ['b', 'a']);
```

#### 33.1.8 在地图和对象之间转换

只要地图只使用字符串和符号作为键，就可以将其转换为对象（通过 `Object.fromEntries()`）：

```js
const map = new Map([
 ['a', 1],
 ['b', 2],
]);
const obj = Object.fromEntries(map);
assert.deepEqual(
 obj, {a: 1, b: 2});
```

您还可以使用字符串或符号键将对象转换为地图（通过 `Object.entries()`）：

```js
const obj = {
 a: 1,
 b: 2,
};
const map = new Map(Object.entries(obj));
assert.deepEqual(
 map, new Map([['a', 1], ['b', 2]]));
```

### 33.2 示例：计算字符

`countChars()` 返回一个将字符映射到出现次数的地图。

```js
function countChars(chars) {
 const charCounts = new Map();
 for (let ch of chars) {
 ch = ch.toLowerCase();
 const prevCount = charCounts.get(ch) ?? 0;
 charCounts.set(ch, prevCount+1);
 }
 return charCounts;
}

const result = countChars('AaBccc');
assert.deepEqual(
 Array.from(result),
 [
 ['a', 2],
 ['b', 1],
 ['c', 3],
 ]
);
```

### 33.3 关于地图键的一些细节（高级）

任何值都可以是一个键，甚至是一个对象：

```js
const map = new Map();

const KEY1 = {};
const KEY2 = {};

map.set(KEY1, 'hello');
map.set(KEY2, 'world');

assert.equal(map.get(KEY1), 'hello');
assert.equal(map.get(KEY2), 'world');
```

#### 33.3.1 哪些键被认为是相等的？

大多数地图操作需要检查一个值是否等于其中一个键。它们通过内部操作 [SameValueZero](http://www.ecma-international.org/ecma-262/6.0/#sec-samevaluezero) 来进行，它的工作方式类似于 `===`，但认为 `NaN` 等于它自己。

因此，您可以像任何其他值一样在地图中使用 `NaN` 作为键：

```js
> const map = new Map();

> map.set(NaN, 123);
> map.get(NaN)
123
```

不同的对象总是被认为是不同的。这是无法改变的事情（但是 - 配置键相等性在 TC39 的长期路线图上）。

```js
> new Map().set({}, 1).set({}, 2).size
2
```

### 33.4 缺失的地图操作

#### 33.4.1 映射和过滤地图

您可以对数组进行 `.map()` 和 `.filter()`，但对于地图没有这样的操作。解决方法是：

1.  将地图转换为 [键，值] 对的数组。

1.  地图或过滤数组。

1.  将结果转换回地图。

我将使用以下地图来演示它是如何工作的。

```js
const originalMap = new Map()
.set(1, 'a')
.set(2, 'b')
.set(3, 'c');
```

映射 `originalMap`：

```js
const mappedMap = new Map( // step 3
 Array.from(originalMap) // step 1
 .map(([k, v]) => [k * 2, '_' + v]) // step 2
);
assert.deepEqual(
 Array.from(mappedMap),
 [[2,'_a'], [4,'_b'], [6,'_c']]);
```

过滤 `originalMap`：

```js
const filteredMap = new Map( // step 3
 Array.from(originalMap) // step 1
 .filter(([k, v]) => k < 3) // step 2
);
assert.deepEqual(Array.from(filteredMap),
 [[1,'a'], [2,'b']]);
```

`Array.from()` 将任何可迭代对象转换为数组。

#### 33.4.2 合并地图

没有合并地图的方法，这就是为什么我们必须使用类似于上一节的解决方法。

让我们合并以下两个地图：

```js
const map1 = new Map()
 .set(1, '1a')
 .set(2, '1b')
 .set(3, '1c')
;

const map2 = new Map()
 .set(2, '2b')
 .set(3, '2c')
 .set(4, '2d')
;
```

要合并 `map1` 和 `map2`，我们创建一个新数组，并将 `map1` 和 `map2` 的条目（键值对）扩展（`...`）到其中（通过迭代）。然后我们将数组转换回地图。所有这些都在 A 行中完成：

```js
const combinedMap = new Map([...map1, ...map2]); // (A)
assert.deepEqual(
 Array.from(combinedMap), // convert to Array for comparison
 [ [ 1, '1a' ],
 [ 2, '2b' ],
 [ 3, '2c' ],
 [ 4, '2d' ] ]
);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：合并两个地图**

`exercises/maps/combine_maps_test.mjs`

### 33.5 快速参考：`Map<K,V>`

注意：为了简洁起见，我假装所有键都具有相同的类型 `K`，所有值都具有相同的类型 `V`。

#### 33.5.1 构造函数

+   `new Map<K, V>(entries?: Iterable<[K, V]>)` ^([ES6])

    如果您不提供参数 `entries`，则会创建一个空地图。如果您提供一个 [键，值] 对的可迭代对象，则这些对将作为条目添加到地图中。例如：

    ```js
    const map = new Map([
     [ 1, 'one' ],
     [ 2, 'two' ],
     [ 3, 'three' ], // trailing comma is ignored
    ]);
    ```

#### 33.5.2 `Map<K,V>.prototype`: 处理单个条目

+   `.get(key: K): V` ^([ES6])

    返回此地图中将 `key` 映射到的 `value`。如果在此地图中没有键 `key`，则返回 `undefined`。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    assert.equal(map.get(1), 'one');
    assert.equal(map.get(5), undefined);
    ```

+   `.set(key: K, value: V): this` ^([ES6])

    将给定的键映射到给定的值。如果已经有一个键是 `key` 的条目，它将被更新。否则，将创建一个新条目。此方法返回 `this`，这意味着您可以链接它。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    map.set(1, 'ONE!')
     .set(3, 'THREE!');
    assert.deepEqual(
     Array.from(map.entries()),
     [[1, 'ONE!'], [2, 'two'], [3, 'THREE!']]);
    ```

+   `.has(key: K): boolean` ^([ES6])

    返回此地图中是否存在给定的键。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    assert.equal(map.has(1), true); // key exists
    assert.equal(map.has(5), false); // key does not exist
    ```

+   `.delete(key: K): boolean` ^([ES6])

    如果有一个键是 `key` 的条目，它将被移除并返回 `true`。否则，什么也不会发生，并返回 `false`。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    assert.equal(map.delete(1), true);
    assert.equal(map.delete(5), false); // nothing happens
    assert.deepEqual(
     Array.from(map.entries()),
     [[2, 'two']]);
    ```

#### 33.5.3 `Map<K,V>.prototype`: 处理所有条目

+   `get .size: number` ^([ES6])

    返回此地图有多少条目。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    assert.equal(map.size, 2);
    ```

+   `.clear(): void` ^([ES6])

    从此地图中删除所有条目。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    assert.equal(map.size, 2);
    map.clear();
    assert.equal(map.size, 0);
    ```

#### 33.5.4 `Map<K,V>.prototype`: 迭代和循环

迭代和循环都是按照条目添加到地图的顺序进行的。

+   `.entries(): Iterable<[K,V]>` ^([ES6])

    返回一个可迭代对象，其中每个条目都有一个[key, value]对。这些对是长度为 2 的数组。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    for (const entry of map.entries()) {
     console.log(entry);
    }
    // Output:
    // [1, 'one']
    // [2, 'two']
    ```

+   `.forEach(callback: (value: V, key: K, theMap: Map<K,V>) => void, thisArg?: any): void` ^([ES6])

    第一个参数是一个回调，对此地图中的每个条目调用一次。如果提供了`thisArg`，则对每次调用都将其设置为`this`。否则，将`this`设置为`undefined`。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    map.forEach((value, key) => console.log(value, key));
    // Output:
    // 'one', 1
    // 'two', 2
    ```

+   `.keys(): Iterable<K>` ^([ES6])

    返回此地图中所有键的可迭代对象。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    for (const key of map.keys()) {
     console.log(key);
    }
    // Output:
    // 1
    // 2
    ```

+   `.values(): Iterable<V>` ^([ES6])

    返回此地图中所有值的可迭代对象。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    for (const value of map.values()) {
     console.log(value);
    }
    // Output:
    // 'one'
    // 'two'
    ```

+   `[Symbol.iterator](): Iterable<[K,V]>` ^([ES6])

    迭代地图的默认方式。与`.entries()`相同。

    ```js
    const map = new Map([[1, 'one'], [2, 'two']]);
    for (const [key, value] of map) {
     console.log(key, value);
    }
    // Output:
    // 1, 'one'
    // 2, 'two'
    ```

#### 33.5.5 本节的来源

+   [TypeScript 的内置类型](https://github.com/Microsoft/TypeScript/blob/master/lib/)

### 33.6 常见问题：地图

#### 33.6.1 何时应该使用地图，何时应该使用对象？

如果您需要一个类似字典的数据结构，其键既不是字符串也不是符号，那么您别无选择，必须使用地图。

然而，如果您的键是字符串或符号，则必须决定是否使用对象。一个粗略的一般指导原则是：

+   是否有一组在开发时已知的键？

    然后使用对象`obj`，并通过固定键访问值：

    ```js
    const value = obj.key;
    ```

+   键的集合是否可以在运行时更改？

    然后使用地图`map`，并通过存储在变量中的键访问值：

    ```js
    const theKey = 123;
    map.get(theKey);
    ```

#### 33.6.2 何时应该将对象用作地图中的键？

通常希望地图键通过值进行比较（如果两个键具有相同的内容，则认为它们相等）。这排除了对象。但是，对象作为键有一个用例：将数据外部附加到对象上。但是，WeakMaps 更适合这种用例，其中条目不会阻止键被垃圾回收（有关详细信息，请参阅下一章）。

#### 33.6.3 为什么地图保留插入条目的顺序？

原则上，地图是无序的。排序条目的主要原因是列出条目、键或值的操作是确定性的。例如，这有助于测试。

#### 33.6.4 为什么地图有`.size`，而数组有`.length`？

在 JavaScript 中，可索引的序列（例如数组和字符串）具有`.length`，而不可索引的集合（例如地图和集合）具有`.size`：

+   `.length`基于索引；它始终是最高索引加一。

+   `.size`计算集合中元素的数量。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

参见测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/35)
