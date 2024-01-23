# 十、开始使用测验和练习

> 原文：[`exploringjs.com/impatient-js/ch_quizzes-exercises.html`](https://exploringjs.com/impatient-js/ch_quizzes-exercises.html)

* * *

+   10.1 测验

+   10.2 练习

    +   10.2.1 安装练习

    +   10.2.2 运行练习

+   10.3 JavaScript 中的单元测试

    +   10.3.1 典型测试

    +   10.3.2 Mocha 中的异步测试

* * *

在大多数章节中，都有测验和练习。这是一个付费功能，但提供了全面的预览。本章解释了如何开始使用它们。

### 10.1 测验

安装：

+   下载并解压`impatient-js-quiz.zip`

运行测验应用程序：

+   在 Web 浏览器中打开`impatient-js-quiz/index.html`

+   你将看到所有测验的目录。

### 10.2 练习

#### 10.2.1 安装练习

要安装练习：

+   下载并解压`impatient-js-code.zip`

+   按照`README.txt`中的说明操作

#### 10.2.2 运行练习

+   本书中通过路径引用练习。

    +   例如：`exercises/quizzes-exercises/first_module_test.mjs`

+   在每个文件中：

    +   第一行包含运行练习的命令。

    +   以下行描述了你需要做的事情。

### 10.3 JavaScript 中的单元测试

本书中的所有练习都是通过测试框架[Mocha](https://mochajs.org)运行的测试。本节简要介绍了这一点。

#### 10.3.1 典型测试

典型的测试代码分为两部分：

+   第 1 部分：要测试的代码。

+   第 2 部分：代码的测试。

例如，以下是两个文件：

+   `id.mjs`（要测试的代码）

+   `id_test.mjs`（测试）

##### 10.3.1.1 第 1 部分：代码

代码本身位于`id.mjs`中：

```js
export function id(x) {
 return x;
}
```

关键是：我们想要测试的所有内容都必须被导出。否则，测试代码无法访问它。

##### 10.3.1.2 第 2 部分：测试

！[](../Images/ec8e6930fbe484fc519f3aa7b812c3fd.png) **不要担心测试的确切细节**

你不需要担心测试的确切细节：它们总是为你实现的。因此，你只需要阅读它们，而不需要编写它们。

代码的测试位于`id_test.mjs`中：

```js
// npm t demos/quizzes-exercises/id_test.mjs
suite('id_test.mjs');

import * as assert from 'assert/strict'; // (A)
import {id} from './id.mjs'; // (B)

test('My test', () => { // (C)
 assert.equal(id('abc'), 'abc'); // (D)
});
```

这个测试文件的核心是 D 行 - 一个断言：`assert.equal()`指定`id('abc')`的预期结果是`'abc'`。

至于其他行：

+   开头的注释显示了运行测试的 shell 命令。

+   A 行：我们导入 Node.js 断言库（在*严格断言模式*下）。

+   B 行：我们导入要测试的函数。

+   C 行：我们定义一个测试。这是通过调用函数`test()`来完成的：

    +   第一个参数：测试的名称。

    +   第二个参数：通过箭头函数提供的测试代码。参数`t`使我们能够访问 AVA 的测试 API（断言等）。

要运行测试，我们在命令行中执行以下操作：

```js
npm t demos/quizzes-exercises/id_test.mjs
```

`t`是`test`的缩写。也就是说，这个命令的完整版本是：

```js
npm test demos/quizzes-exercises/id_test.mjs
```

！[](../Images/90f73f1851c5b1baf43cb746913c09e6.png) **练习：你的第一个练习**

以下练习让你初尝练习的滋味：

+   `exercises/quizzes-exercises/first_module_test.mjs`

#### 10.3.2 Mocha 中的异步测试

！[](../Images/ec8e6930fbe484fc519f3aa7b812c3fd.png) **阅读**

在学习异步编程的章节之前，你可能想推迟阅读本节。

编写异步代码的测试需要额外的工作：测试稍后接收其结果，并在返回时向 Mocha 发出信号表明它尚未完成。以下小节探讨了三种方法。

##### 10.3.2.1 通过回调实现异步

如果我们传递给 `test()` 的回调有一个参数（例如 `done`），Mocha 将切换到基于回调的异步性。当我们完成异步工作时，我们必须调用 `done`：

```js
test('divideCallback', (done) => {
 divideCallback(8, 4, (error, result) => {
 if (error) {
 done(error);
 } else {
 assert.strictEqual(result, 2);
 done();
 }
 });
});
```

这是 `divideCallback()` 的样子：

```js
function divideCallback(x, y, callback) {
 if (y === 0) {
 callback(new Error('Division by zero'));
 } else {
 callback(null, x / y);
 }
}
```

##### 10.3.2.2 通过 Promises 实现异步性

如果一个测试返回一个 Promise，Mocha 将切换到基于 Promise 的异步性。如果 Promise 被实现，测试被认为是成功的，如果 Promise 被拒绝，或者结算时间超过超时时间，测试被认为是失败的。

```js
test('dividePromise 1', () => {
 return dividePromise(8, 4)
 .then(result => {
 assert.strictEqual(result, 2);
 });
});
```

`dividePromise()` 的实现如下：

```js
function dividePromise(x, y) {
 return new Promise((resolve, reject) => {
 if (y === 0) {
 reject(new Error('Division by zero'));
 } else {
 resolve(x / y);
 }
 });
}
```

##### 10.3.2.3 异步函数作为测试“主体”

异步函数总是返回 Promises。因此，异步函数是实现异步测试的一种便捷方式。以下代码等同于之前的示例。

```js
test('dividePromise 2', async () => {
 const result = await dividePromise(8, 4);
 assert.strictEqual(result, 2);
 // No explicit return necessary!
});
```

我们不需要显式地返回任何东西：隐式返回的 `undefined` 被用来实现由这个异步函数返回的 Promise。如果测试代码抛出异常，那么异步函数会负责拒绝返回的 Promise。

[注释](https://github.com/rauschma/impatient-js/issues/2)
