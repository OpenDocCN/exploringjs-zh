# 6 本书中使用的符号

> 原文：[`exploringjs.com/tackling-ts/ch_book-notation.html`](https://exploringjs.com/tackling-ts/ch_book-notation.html)

* * *

+   6.1 测试断言（动态）(ch_book-notation.html#test-assertions-dynamic)

+   6.2 类型断言（静态）

* * *

本章解释了代码示例中使用的功能，但不是 TypeScript 本身的一部分。

### 6.1 测试断言（动态）

本书中显示的代码示例通过单元测试自动测试。操作的预期结果通过来自[Node.js 模块`assert`](https://nodejs.org/api/assert.html)的以下断言函数进行检查：

+   `assert.equal()`通过`===`测试相等性

+   `assert.deepEqual()`通过深度比较嵌套对象（包括数组）来测试相等性。

+   `assert.throws()`如果回调参数*没有*抛出异常，则会报错。

这是使用这些断言的一个例子：

```ts
import {strict as assert} from 'assert';

assert.equal(3 + ' apples', '3 apples');

assert.deepEqual(
 [...['a', 'b'], ...['c', 'd']],
 ['a', 'b', 'c', 'd']);

assert.throws(
 () => eval('null.myProperty'),
 TypeError);
```

第一行的 import 语句使用了严格断言模式（使用`===`而不是`==`）。通常在代码示例中会省略这一行。

### 6.2 类型断言（静态）

您还会看到静态类型断言。

`%inferred-type`只是普通 TypeScript 中的一个注释，描述了 TypeScript 为下一行推断的类型：

```ts
// %inferred-type: number
let num = 123;
```

`@ts-expect-error`在 TypeScript 中抑制静态错误。在本书中，抑制的错误总是会被提及。这在普通的 TypeScript 中既不是必需的，也不会有任何作用。

```ts
assert.throws(
 // @ts-expect-error: Object is possibly 'null'. (2531)
 () => null.myProperty,
 TypeError);
```

请注意，我们以前需要使用`eval()`来避免 TypeScript 的警告。

[评论](https://github.com/rauschma/tackling-ts/issues/6)
