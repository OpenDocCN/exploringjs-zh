# 30 同步迭代

> 原文：[`exploringjs.com/impatient-js/ch_sync-iteration.html`](https://exploringjs.com/impatient-js/ch_sync-iteration.html)

* * *

+   30.1 同步迭代是什么？

+   30.2 核心迭代构造：可迭代对象和迭代器

+   30.3 手动迭代

    +   30.3.1 通过`while`迭代可迭代对象

+   30.4 实践中的迭代

    +   30.4.1 遍历数组

    +   30.4.2 遍历集合

+   30.5 同步迭代的快速参考

    +   30.5.1 可迭代数据源

    +   30.5.2 同步迭代语言构造

* * *

### 30.1 同步迭代是什么？

同步迭代是一种*协议*（接口加上使用规则），它连接 JavaScript 中的两组实体：

+   **数据源：** 一方面，数据以各种形状和大小出现。在 JavaScript 的标准库中，您有线性数据结构数组，有序集合 Set（元素按添加时间排序），有序字典 Map（条目按添加时间排序）等。在库中，您可能会找到树形数据结构等。

+   **数据消费者：** 另一方面，您有一整类构造和算法，它们只需要*顺序*访问它们的输入：一次一个值，直到所有值都被访问。例如，`for-of`循环和展开到函数调用（通过`...`）。

迭代协议通过接口`Iterable`连接这两组：数据源通过它顺序地“传递其内容”；数据消费者通过它获取其输入。

![图 18：数据消费者如 for-of 循环使用接口 Iterable。数据源如数组实现该接口。](img/04fc345e457a6c010f8b61f6ba201e81.png)

图 18：数据消费者如`for-of`循环使用接口`Iterable`。数据源如`Arrays`实现该接口。

图 18 展示了迭代的工作原理：数据消费者使用接口`Iterable`；数据源实现它。

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **实现接口的 JavaScript 方式**

在 JavaScript 中，如果一个对象具有描述的所有方法，它就会*实现*一个接口。本章提到的接口只存在于 ECMAScript 规范中。

数据的来源和消费者都从这种安排中受益：

+   如果您开发了一个新的数据结构，您只需要实现`Iterable`，就可以立即应用一系列工具。

+   如果您编写使用迭代的代码，它将自动适用于许多数据源。

### 30.2 核心迭代构造：可迭代对象和迭代器

两个角色（由接口描述）构成了迭代的核心（图 19）：

+   *可迭代对象*是其内容可以按顺序遍历的对象。

+   *迭代器*是用于遍历的指针。

![图 19：迭代有两个主要接口：Iterable 和 Iterator。前者有一个返回后者的方法。](img/d4254788bb2bb7f5a88144606f3d4d28.png)

图 19：迭代有两个主要接口：`Iterable`和`Iterator`。前者有一个返回后者的方法。

这些是迭代协议接口的类型定义（在 TypeScript 的表示法中）：

```js
interface Iterable<T> {
 [Symbol.iterator]() : Iterator<T>;
}

interface Iterator<T> {
 next() : IteratorResult<T>;
}

interface IteratorResult<T> {
 value: T;
 done: boolean;
}
```

这些接口的使用如下：

+   您可以通过其键为`Symbol.iterator`的方法向`Iterable`请求迭代器。

+   `Iterator`通过其方法`.next()`返回迭代的值。

+   值不是直接返回的，而是包装在具有两个属性的对象中：

    +   `.value`是迭代的值。

    +   `.done`指示是否已经到达迭代的末尾。在最后一个迭代值之后为`true`，之前为`false`。

### 30.3 手动迭代

这是使用迭代协议的示例：

```js
const iterable = ['a', 'b'];

// The iterable is a factory for iterators:
const iterator = iterable[Symbol.iterator]();

// Call .next() until .done is true:
assert.deepEqual(
 iterator.next(), { value: 'a', done: false });
assert.deepEqual(
 iterator.next(), { value: 'b', done: false });
assert.deepEqual(
 iterator.next(), { value: undefined, done: true });
```

#### 30.3.1 通过`while`迭代可迭代对象

以下代码演示了如何使用`while`循环来迭代可迭代对象：

```js
function logAll(iterable) {
 const iterator = iterable[Symbol.iterator]();
 while (true) {
 const {value, done} = iterator.next();
 if (done) break;
 console.log(value);
 }
}

logAll(['a', 'b']);
// Output:
// 'a'
// 'b'
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：手动使用同步迭代**

`exercises/sync-iteration-use/sync_iteration_manually_exrc.mjs`

### 30.4 实践中的迭代

我们已经看到了如何手动使用迭代协议，这相对来说比较麻烦。但是协议并不是直接使用的 - 它是通过在其之上构建的高级语言构造来使用的。本节展示了这是什么样子。

#### 30.4.1 遍历数组

JavaScript 的数组是可迭代的。这使我们能够使用`for-of`循环：

```js
const myArray = ['a', 'b', 'c'];

for (const x of myArray) {
 console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
```

通过数组模式解构（稍后解释）也在内部使用迭代：

```js
const [first, second] = myArray;
assert.equal(first, 'a');
assert.equal(second, 'b');
```

#### 30.4.2 遍历集合

JavaScript 的 Set 数据结构是可迭代的。这意味着`for-of`可以工作：

```js
const mySet = new Set().add('a').add('b').add('c');

for (const x of mySet) {
 console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
```

数组解构也是如此：

```js
const [first, second] = mySet;
assert.equal(first, 'a');
assert.equal(second, 'b');
```

### 30.5 快速参考：同步迭代

#### 30.5.1 可迭代数据源

以下内置数据源是可迭代的：

+   数组

+   字符串

+   地图

+   集合

+   （浏览器：DOM 数据结构）

迭代对象的属性时，您需要诸如`Object.keys()`和`Object.entries()`之类的辅助工具。这是必要的，因为属性存在于与数据结构级别无关的不同级别。

#### 30.5.2 同步迭代语言构造

本节列出了使用同步迭代的构造。

##### 30.5.2.1 迭代的语言构造

+   通过数组模式解构：

    ```js
    const [x,y] = iterable;
    ```

+   通过`...`扩展到函数调用和数组文字中：

    ```js
    func(...iterable);
    const arr = [...iterable];
    ```

+   `for-of`循环：

    ```js
    for (const x of iterable) { /*···*/ }
    ```

+   `yield*`：

    ```js
    function* generatorFunction() {
     yield* iterable;
    }
    ```

##### 30.5.2.2 将可迭代对象转换为数据结构

+   `Object.fromEntries()`:

    ```js
    const obj = Object.fromEntries(iterableOverKeyValuePairs);
    ```

+   `Array.from()`:

    ```js
    const arr = Array.from(iterable);
    ```

+   `new Map()`和`new WeakMap()`：

    ```js
    const m  = new Map(iterableOverKeyValuePairs);
    const wm = new WeakMap(iterableOverKeyValuePairs);
    ```

+   `new Set()`和`new WeakSet()`：

    ```js
    const s  = new Set(iterableOverElements);
    const ws = new WeakSet(iterableOverElements);
    ```

##### 30.5.2.3 杂项

+   Promise 组合函数：`Promise.all()`等。

    ```js
    const promise1 = Promise.all(iterableOverPromises);
    const promise2 = Promise.race(iterableOverPromises);
    const promise3 = Promise.any(iterableOverPromises);
    const promise4 = Promise.allSettled(iterableOverPromises);
    ```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/21)
