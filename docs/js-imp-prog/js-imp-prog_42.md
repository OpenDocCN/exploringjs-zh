# 三十五、集合（Set）

> 原文：[`exploringjs.com/impatient-js/ch_sets.html`](https://exploringjs.com/impatient-js/ch_sets.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   35.1 使用集合

    +   35.1.1 创建集合

    +   35.1.2 添加、删除、检查成员资格

    +   35.1.3 确定集合的大小并清除它

    +   35.1.4 遍历集合

+   35.2 使用集合的示例

    +   35.2.1 从数组中移除重复项

    +   35.2.2 创建一个 Unicode 字符（代码点）的集合

+   35.3 哪些集合元素被视为相等？

+   35.4 缺失的集合操作

    +   35.4.1 并集 (`a` ∪ `b`)

    +   35.4.2 交集 (`a` ∩ `b`)

    +   35.4.3 差集 (`a` \ `b`)

    +   35.4.4 映射集合

    +   35.4.5 过滤集合

+   35.5 快速参考：`Set<T>`

    +   35.5.1 构造函数

    +   35.5.2 `Set<T>.prototype`：单个 Set 元素

    +   35.5.3 `Set<T>.prototype`：所有 Set 元素

    +   35.5.4 `Set<T>.prototype`：迭代和循环

    +   35.5.5 与`Map`的对称性

+   35.6 常见问题：集合

    +   35.6.1 为什么集合有`.size`，而数组有`.length`？

* * *

在 ES6 之前，JavaScript 没有集合的数据结构。而是使用了两种解决方法：

+   对象的键被用作字符串的集合。

+   数组被用作任意值的集合。缺点是检查*成员资格*（数组是否包含一个值）较慢。

自 ES6 以来，JavaScript 有了数据结构`Set`，它可以包含任意值并快速执行成员资格检查。

### 35.1 使用集合

#### 35.1.1 创建集合

有三种常见的创建集合的方法。

首先，您可以使用没有任何参数的构造函数创建一个空的 Set：

```js
const emptySet = new Set();
assert.equal(emptySet.size, 0);
```

其次，您可以将可迭代对象（例如数组）传递给构造函数。迭代的值成为新 Set 的元素：

```js
const set = new Set(['red', 'green', 'blue']);
```

第三，`.add()` 方法向 Set 中添加元素，并且可以链式调用：

```js
const set = new Set()
.add('red')
.add('green')
.add('blue');
```

#### 35.1.2 添加、删除、检查成员资格

`.add()` 向 Set 中添加一个元素。

```js
const set = new Set();
set.add('red');
```

`.has()` 检查一个元素是否是 Set 的成员。

```js
assert.equal(set.has('red'), true);
```

`.delete()` 从 Set 中移除一个元素。

```js
assert.equal(set.delete('red'), true); // there was a deletion
assert.equal(set.has('red'), false);
```

#### 35.1.3 确定集合的大小并清除它

`.size` 包含集合中元素的数量。

```js
const set = new Set()
 .add('foo')
 .add('bar');
assert.equal(set.size, 2)
```

`.clear()` 移除 Set 的所有元素。

```js
set.clear();
assert.equal(set.size, 0)
```

#### 35.1.4 遍历集合

集合是可迭代的，`for-of`循环的工作方式与您期望的一样：

```js
const set = new Set(['red', 'green', 'blue']);
for (const x of set) {
 console.log(x);
}
// Output:
// 'red'
// 'green'
// 'blue'
```

正如您所看到的，集合保留了*插入顺序*。也就是说，元素总是按照它们被添加的顺序进行迭代。

鉴于集合是可迭代的，您可以使用`Array.from()`将其转换为数组：

```js
const set = new Set(['red', 'green', 'blue']);
const arr = Array.from(set); // ['red', 'green', 'blue']
```

### 35.2 使用集合的示例

#### 35.2.1 从数组中移除重复项

将数组转换为 Set，然后再转换回来，可以从数组中移除重复项：

```js
assert.deepEqual(
 Array.from(new Set([1, 2, 1, 2, 3, 3, 3])),
 [1, 2, 3]);
```

#### 35.2.2 创建一个 Unicode 字符（代码点）的集合

字符串是可迭代的，因此可以作为`new Set()`的参数使用：

```js
assert.deepEqual(
 new Set('abc'),
 new Set(['a', 'b', 'c']));
```

### 35.3 哪些集合元素被视为相等？

与 Map 键一样，Set 元素的比较方式类似于`===`，唯一的例外是`NaN`等于它自己。

```js
> const set = new Set([NaN, NaN, NaN]);
> set.size
1
> set.has(NaN)
true
```

与`===`一样，两个不同的对象永远不会被视为相等（目前没有办法改变这一点）：

```js
> const set = new Set();

> set.add({});
> set.size
1

> set.add({});
> set.size
2
```

### 35.4 缺失的集合操作

集合缺少一些常见的操作。这样的操作通常可以通过实现：

+   通过扩展为数组文字将输入的 Set 转换为数组。

+   在数组上执行操作。

+   将结果转换为 Set 并返回。

#### 35.4.1 并集（`a` ∪ `b`）

计算两个 Set `a` 和 `b` 的并集意味着创建一个包含`a`和`b`元素的 Set。

```js
const a = new Set([1,2,3]);
const b = new Set([4,3,2]);
// Use spreading to concatenate two iterables
const union = new Set([...a, ...b]);

assert.deepEqual(Array.from(union), [1, 2, 3, 4]);
```

#### 35.4.2 交集（`a` ∩ `b`）

计算两个 Set `a` 和 `b` 的交集意味着创建一个包含`a`和`b`中都有的元素的 Set。

```js
const a = new Set([1,2,3]);
const b = new Set([4,3,2]);
const intersection = new Set(
 Array.from(a).filter(x => b.has(x))
);

assert.deepEqual(
 Array.from(intersection), [2, 3]
);
```

#### 35.4.3 差集（`a` \ `b`）

计算两个 Set `a` 和 `b` 之间的差异意味着创建一个包含`a`中不在`b`中的元素的 Set。这个操作有时也被称为*减*（−）。

```js
const a = new Set([1,2,3]);
const b = new Set([4,3,2]);
const difference = new Set(
 Array.from(a).filter(x => !b.has(x))
);

assert.deepEqual(
 Array.from(difference), [1]
);
```

#### 35.4.4 对 Set 进行映射

集合没有一个名为`.map()`的方法。但是我们可以借用数组的方法：

```js
const set = new Set([1, 2, 3]);
const mappedSet = new Set(
 Array.from(set).map(x => x * 2)
);

// Convert mappedSet to an Array to check what’s inside it
assert.deepEqual(
 Array.from(mappedSet), [2, 4, 6]
);
```

#### 35.4.5 对 Set 进行过滤

我们不能直接使用`.filter()`来过滤 Set，所以我们需要使用相应的数组方法：

```js
const set = new Set([1, 2, 3, 4, 5]);
const filteredSet = new Set(
 Array.from(set).filter(x => (x % 2) === 0)
);

assert.deepEqual(
 Array.from(filteredSet), [2, 4]
);
```

### 35.5 快速参考：`Set<T>`

#### 35.5.1 构造函数

+   `new Set<T>(values?: Iterable<T>)` ^([ES6])

    如果你不提供参数`values`，那么将创建一个空的 Set。如果你提供了参数，那么迭代的值将被添加为 Set 的元素。例如：

    ```js
    const set = new Set(['red', 'green', 'blue']);
    ```

#### 35.5.2 `Set<T>.prototype`: 单个 Set 元素

+   `.add(value: T): this` ^([ES6])

    将`value`添加到这个 Set 中。这个方法返回`this`，这意味着它可以被链接。

    ```js
    const set = new Set(['red']);
    set.add('green').add('blue');
    assert.deepEqual(
     Array.from(set), ['red', 'green', 'blue']
    );
    ```

+   `.delete(value: T): boolean` ^([ES6])

    从这个 Set 中移除`value`。如果有东西被删除，则返回`true`，否则返回`false`。

    ```js
    const set = new Set(['red', 'green', 'blue']);
    assert.equal(set.delete('red'), true); // there was a deletion
    assert.deepEqual(
     Array.from(set), ['green', 'blue']
    );
    ```

+   `.has(value: T): boolean` ^([ES6])

    检查`value`是否在这个 Set 中。

    ```js
    const set = new Set(['red', 'green']);
    assert.equal(set.has('red'), true);
    assert.equal(set.has('blue'), false);
    ```

#### 35.5.3 `Set<T>.prototype`: 所有 Set 元素

+   `get .size: number` ^([ES6])

    返回这个 Set 中有多少个元素。

    ```js
    const set = new Set(['red', 'green', 'blue']);
    assert.equal(set.size, 3);
    ```

+   `.clear(): void` ^([ES6])

    从这个 Set 中移除所有元素。

    ```js
    const set = new Set(['red', 'green', 'blue']);
    assert.equal(set.size, 3);
    set.clear();
    assert.equal(set.size, 0);
    ```

#### 35.5.4 `Set<T>.prototype`: 迭代和循环

+   `.values(): Iterable<T>` ^([ES6])

    返回这个 Set 的所有元素的可迭代对象。

    ```js
    const set = new Set(['red', 'green']);
    for (const x of set.values()) {
     console.log(x);
    }
    // Output:
    // 'red'
    // 'green'
    ```

+   `[Symbol.iterator](): Iterable<T>` ^([ES6])

    迭代 Set 的默认方式。与`.values()`相同。

    ```js
    const set = new Set(['red', 'green']);
    for (const x of set) {
     console.log(x);
    }
    // Output:
    // 'red'
    // 'green'
    ```

+   `.forEach(callback: (value: T, key: T, theSet: Set<T>) => void, thisArg?: any): void` ^([ES6])

    将这个 Set 的每个元素传递给`callback()`。`value`和`key`都包含当前元素。这种冗余是为了使这个`callback`的类型签名与`Map.prototype.forEach()`的`callback`相同。

    您可以通过`thisArg`指定`callback`的`this`。如果省略，`this`为`undefined`。

    ```js
    const set = new Set(['red', 'green']);
    set.forEach(x => console.log(x));
    // Output:
    // 'red'
    // 'green'
    ```

#### 35.5.5 与`Map`的对称性

以下两种方法主要是为了使 Set 和 Map 具有类似的接口。每个 Set 元素都被处理，就好像它是一个键和值都是元素的 Map 条目一样。

+   `Set.prototype.entries(): Iterable<[T,T]>` ^([ES6])

+   `Set.prototype.keys(): Iterable<T>` ^([ES6])

`.entries()`使您可以将 Set 转换为 Map：

```js
const set = new Set(['a', 'b', 'c']);
const map = new Map(set.entries());
assert.deepEqual(
 Array.from(map.entries()),
 [['a','a'], ['b','b'], ['c','c']]
);
```

### 35.6 常见问题：Set

#### 35.6.1 为什么 Set 有`.size`，而数组有`.length`？

这个问题的答案在§33.6.4 “为什么 Map 有`.size`，而数组有`.length`？”中给出。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/37)
