# 三十八、同步生成器（高级）

> 原文：[`exploringjs.com/impatient-js/ch_sync-generators.html`](https://exploringjs.com/impatient-js/ch_sync-generators.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

* * *

+   38.1 什么是同步生成器？

    +   38.1.1 生成器函数返回可迭代对象并通过`yield`填充它们

    +   38.1.2 `yield`暂停生成器函数

    +   38.1.3 `yield`为什么会暂停执行？

    +   38.1.4 示例：在可迭代对象上进行映射

+   38.2 从生成器中调用生成器（高级）

    +   38.2.1 通过`yield*`调用生成器

    +   38.2.2 示例：遍历树

+   38.3 背景：外部迭代 vs. 内部迭代

+   38.4 生成器的用例：重用遍历

    +   38.4.1 可重用的遍历

    +   38.4.2 内部迭代（推送）

    +   38.4.3 外部迭代（拉取）

+   38.5 生成器的高级特性

* * *

### 38.1 什么是同步生成器？

同步生成器是函数定义和方法定义的特殊版本，它们总是返回同步可迭代对象：

```js
// Generator function declaration
function* genFunc1() { /*···*/ }

// Generator function expression
const genFunc2 = function* () { /*···*/ };

// Generator method definition in an object literal
const obj = {
 * generatorMethod() {
 // ···
 }
};

// Generator method definition in a class definition
// (class declaration or class expression)
class MyClass {
 * generatorMethod() {
 // ···
 }
}
```

星号(`*`)标记函数和方法为生成器：

+   函数：伪关键字`function*`是关键字`function`和星号的组合。

+   方法：`*`是一个修饰符（类似于`static`和`get`）。

#### 38.1.1 生成器函数返回可迭代对象并通过`yield`填充它们

如果我们调用一个生成器函数，它会返回一个可迭代对象（实际上是一个同时也是可迭代的迭代器）。生成器通过`yield`操作符填充该可迭代对象：

```js
function* genFunc1() {
 yield 'a';
 yield 'b';
}

const iterable = genFunc1();
// Convert the iterable to an Array, to check what’s inside:
assert.deepEqual(
 Array.from(iterable), ['a', 'b']
);

// We can also use a for-of loop
for (const x of genFunc1()) {
 console.log(x);
}
// Output:
// 'a'
// 'b'
```

#### 38.1.2 `yield`暂停生成器函数

使用生成器函数涉及以下步骤：

+   调用它会返回一个迭代器`iter`（也是可迭代的）。

+   重复迭代`iter`会调用`iter.next()`。每次，我们都会进入生成器函数的主体，直到出现返回值的`yield`。

因此，`yield`不仅仅是向可迭代对象添加值，它还会暂停并退出生成器函数：

+   与`return`一样，`yield`退出函数体并返回一个值（通过`.next()`）。

+   与`return`不同，如果我们重复调用（`.next()`），执行将直接在`yield`之后恢复。

让我们通过以下生成器函数来检查这意味着什么。

```js
let location = 0;
function* genFunc2() {
 location = 1; yield 'a';
 location = 2; yield 'b';
 location = 3;
}
```

为了使用`genFunc2()`，我们必须首先创建迭代器/可迭代对象`iter`。`genFunc2()`现在被暂停在其主体“之前”。

```js
const iter = genFunc2();
// genFunc2() is now paused “before” its body:
assert.equal(location, 0);
```

`iter`实现了迭代协议。因此，我们通过`iter.next()`来控制`genFunc2()`的执行。调用该方法会恢复暂停的`genFunc2()`并执行，直到出现`yield`。然后执行暂停，`.next()`返回`yield`的操作数：

```js
assert.deepEqual(
 iter.next(), {value: 'a', done: false});
// genFunc2() is now paused directly after the first `yield`:
assert.equal(location, 1);
```

请注意，`yield`的值`'a'`被包装在一个对象中，这就是迭代器始终传递其值的方式。

我们再次调用`iter.next()`，执行将在之前暂停的地方继续。一旦遇到第二个`yield`，`genFunc2()`被暂停，`.next()`返回`'b'`。

```js
assert.deepEqual(
 iter.next(), {value: 'b', done: false});
// genFunc2() is now paused directly after the second `yield`:
assert.equal(location, 2);
```

我们再次调用`iter.next()`，执行将继续，直到离开`genFunc2()`的主体：

```js
assert.deepEqual(
 iter.next(), {value: undefined, done: true});
// We have reached the end of genFunc2():
assert.equal(location, 3);
```

这一次，`.next()`的结果的`.done`属性为`true`，这意味着迭代器已经完成。

#### 38.1.3 `yield`为什么会暂停执行？

`yield`暂停执行的好处是什么？为什么它不像数组方法`.push()`那样简单地填充可迭代对象而不暂停？

由于暂停，生成器提供了许多*协程*的功能（考虑协作式多任务处理的进程）。例如，当我们请求可迭代对象的下一个值时，该值是*惰性计算*的（按需计算）。以下两个生成器函数演示了这意味着什么。

```js
/**
 * Returns an iterable over lines
 */
function* genLines() {
 yield 'A line';
 yield 'Another line';
 yield 'Last line';
}

/**
 * Input: iterable over lines
 * Output: iterable over numbered lines
 */
function* numberLines(lineIterable) {
 let lineNumber = 1;
 for (const line of lineIterable) { // input
 yield lineNumber + ': ' + line; // output
 lineNumber++;
 }
}
```

请注意，`numberLines()`中的`yield`出现在`for-of`循环内。`yield`可以在循环内使用，但不能在回调内使用（稍后会详细介绍）。

让我们将两个生成器组合起来产生可迭代对象`numberedLines`：

```js
const numberedLines = numberLines(genLines());
assert.deepEqual(
 numberedLines.next(), {value: '1: A line', done: false});
assert.deepEqual(
 numberedLines.next(), {value: '2: Another line', done: false});
```

在这里使用生成器的主要好处是一切都是逐步进行的：通过`numberedLines.next()`，我们只要求`numberLines()`提供一个编号行。反过来，它只要求`genLines()`提供一个未编号行。

如果，例如，`genLines()`从大型文本文件中读取其行，则此增量主义将继续工作：如果我们要求`numberLines()`提供编号行，则只要`genLines()`从文本文件中读取了第一行，我们就会得到一个编号行。

如果没有生成器，`genLines()`将首先读取所有行并返回它们。然后`numberLines()`将对所有行进行编号并返回它们。因此，我们必须等待更长的时间才能获得第一行编号。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：将普通函数转换为生成器**

`exercises/sync-generators/fib_seq_test.mjs`

#### 38.1.4 示例：在可迭代对象上进行映射

以下函数`mapIter()`类似于数组方法`.map()`，但它返回一个可迭代对象，而不是一个数组，并且按需生成其结果。

```js
function* mapIter(iterable, func) {
 let index = 0;
 for (const x of iterable) {
 yield func(x, index);
 index++;
 }
}

const iterable = mapIter(['a', 'b'], x => x + x);
assert.deepEqual(
 Array.from(iterable), ['aa', 'bb']
);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：过滤可迭代对象**

`exercises/sync-generators/filter_iter_gen_test.mjs`

### 38.2 从生成器调用生成器（高级）

#### 38.2.1 通过`yield*`调用生成器

`yield`只能直接在生成器内部使用 - 到目前为止，我们还没有看到将产出委托给另一个函数或方法的方法。

让我们首先检查什么*不*起作用：在以下示例中，我们希望`foo()`调用`bar()`，以便后者为前者产生两个值。遗憾的是，朴素的方法失败了：

```js
function* bar() {
 yield 'a';
 yield 'b';
}
function* foo() {
 // Nothing happens if we call `bar()`:
 bar();
}
assert.deepEqual(
 Array.from(foo()), []
);
```

为什么这不起作用？函数调用`bar()`返回一个可迭代对象，我们忽略了它。

我们希望`foo()`产生`bar()`产生的所有内容。这就是`yield*`运算符的作用：

```js
function* bar() {
 yield 'a';
 yield 'b';
}
function* foo() {
 yield* bar();
}
assert.deepEqual(
 Array.from(foo()), ['a', 'b']
);
```

换句话说，前面的`foo()`大致相当于：

```js
function* foo() {
 for (const x of bar()) {
 yield x;
 }
}
```

请注意，`yield*`适用于任何可迭代对象：

```js
function* gen() {
 yield* [1, 2];
}
assert.deepEqual(
 Array.from(gen()), [1, 2]
);
```

#### 38.2.2 示例：遍历树

`yield*`让我们在生成器中进行递归调用，这在迭代递归数据结构（如树）时非常有用。例如，以下是用于二叉树的数据结构。

```js
class BinaryTree {
 constructor(value, left=null, right=null) {
 this.value = value;
 this.left = left;
 this.right = right;
 }

 /** Prefix iteration: parent before children */
 * [Symbol.iterator]() {
 yield this.value;
 if (this.left) {
 // Same as yield* this.left[Symbol.iterator]()
 yield* this.left;
 }
 if (this.right) {
 yield* this.right;
 }
 }
}
```

方法`[Symbol.iterator]()`添加了对迭代协议的支持，这意味着我们可以使用`for-of`循环来迭代`BinaryTree`的实例：

```js
const tree = new BinaryTree('a',
 new BinaryTree('b',
 new BinaryTree('c'),
 new BinaryTree('d')),
 new BinaryTree('e'));

for (const x of tree) {
 console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
// 'd'
// 'e'
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：遍历嵌套数组**

`exercises/sync-generators/iter_nested_arrays_test.mjs`

### 38.3 背景：外部迭代 vs. 内部迭代

为了准备下一节，我们需要了解两种不同的迭代对象“内部”值的风格：

+   外部迭代（拉取）：您的代码通过迭代协议向对象请求值。例如，`for-of`循环基于 JavaScript 的迭代协议：

    ```js
    for (const x of ['a', 'b']) {
     console.log(x);
    }
    // Output:
    // 'a'
    // 'b'
    ```

+   内部迭代（推送）：我们将回调函数传递给对象的方法，该方法将值提供给回调。例如，数组具有方法`.forEach()`：

    ```js
    ['a', 'b'].forEach((x) => {
     console.log(x);
    });
    // Output:
    // 'a'
    // 'b'
    ```

下一节中有两种迭代风格的示例。

### 38.4 生成器的用例：重用遍历

生成器的一个重要用例是提取和重用遍历。

#### 38.4.1 重用的遍历

例如，考虑以下遍历文件树并记录它们路径的函数（它使用[Node.js API](https://nodejs.org/en/docs/)来实现）：

```js
function logPaths(dir) {
 for (const fileName of fs.readdirSync(dir)) {
 const filePath = path.resolve(dir, fileName);
 console.log(filePath);
 const stats = fs.statSync(filePath);
 if (stats.isDirectory()) {
 logPaths(filePath); // recursive call
 }
 }
}
```

考虑以下目录：

```js
mydir/
    a.txt
    b.txt
    subdir/
        c.txt
```

让我们记录`mydir/`内部的路径：

```js
logPaths('mydir');

// Output:
// 'mydir/a.txt'
// 'mydir/b.txt'
// 'mydir/subdir'
// 'mydir/subdir/c.txt'
```

我们如何重用这个遍历并做一些除了记录路径之外的事情呢？

#### 38.4.2 内部迭代（推）

重用遍历代码的一种方式是通过*内部迭代*：每个遍历的值都传递给一个回调函数（A 行）。

```js
function visitPaths(dir, callback) {
 for (const fileName of fs.readdirSync(dir)) {
 const filePath = path.resolve(dir, fileName);
 callback(filePath); // (A)
 const stats = fs.statSync(filePath);
 if (stats.isDirectory()) {
 visitPaths(filePath, callback);
 }
 }
}
const paths = [];
visitPaths('mydir', p => paths.push(p));
assert.deepEqual(
 paths,
 [
 'mydir/a.txt',
 'mydir/b.txt',
 'mydir/subdir',
 'mydir/subdir/c.txt',
 ]);
```

#### 38.4.3 外部迭代（拉）

重用遍历代码的另一种方式是通过*外部迭代*：我们可以编写一个生成器，它会产生所有遍历的值（A 行）。

```js
function* iterPaths(dir) {
 for (const fileName of fs.readdirSync(dir)) {
 const filePath = path.resolve(dir, fileName);
 yield filePath; // (A)
 const stats = fs.statSync(filePath);
 if (stats.isDirectory()) {
 yield* iterPaths(filePath);
 }
 }
}
const paths = Array.from(iterPaths('mydir'));
```

### 38.5 生成器的高级特性

*Exploring ES6*中的[生成器章节](https://exploringjs.com/es6/ch_generators.html)涵盖了本书范围之外的两个特性：

+   `yield`也可以通过`.next()`的参数*接收*数据。

+   生成器也可以`return`值（不仅仅是`yield`它们）。这样的值不会成为迭代值，但可以通过`yield*`来检索。

[评论](https://github.com/rauschma/impatient-js/issues/43)
