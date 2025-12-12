# 5 本书使用的符号

> 原文：[`exploringjs.com/ts/book/ch_book-notation.html`](https://exploringjs.com/ts/book/ch_book-notation.html)

（广告，请勿拦截。）

1.  5.1 JavaScript 级别：`assert.*`

1.  5.2 类型级别：`assertType<T>(v)`

1.  5.3 类型级别：`Assert<B>`

1.  5.4 类型级别：`@ts-expect-error`

1.  5.5 这本书的符号是不是有点丑？

本章解释了在代码示例中用于解释结果和错误的函数。我们必须考虑两个层次：

+   JavaScript 级别：值（例如函数返回的值）和异常

+   类型级别：类型（例如由泛型类型构造的）和编译器错误

帮助我们的函数和泛型类型必须导入：用于导入的语句在本章中显示，但在本书的其他地方省略。

### 5.1 JavaScript 级别：`assert.*`

通过以下断言函数检查预期结果，这些函数来自 [Node.js 模块 `node:assert`](https://nodejs.org/api/assert.html)：

+   `assert.equal()` 通过 `===` 测试相等性。

+   `assert.deepEqual()` 通过深度比较嵌套对象（包括数组）来测试相等性。

+   `assert.throws()` 如果回调参数没有抛出异常，则会抱怨。

这是使用这些断言的一个例子：

```ts
import assert from 'node:assert/strict';

assert.equal(
 3 + ' apples',
 '3 apples'
);

assert.deepEqual(
 [...['a', 'b'], ...['c', 'd']],
 ['a', 'b', 'c', 'd']
);

assert.throws(
 () => Object.freeze({}).prop = true,
 /^TypeError: Cannot add property prop, object is not extensible/
);

```

在第一行，导入模块的指定器有后缀 `/strict`。这启用了 [严格断言模式](https://nodejs.org/api/assert.html#assert_strict_assertion_mode)，该模式使用 `===` 而不是 `==` 进行比较。

### 5.2 类型级别：`assertType<T>(v)`

函数 `assertType()` 由 TypeScript 库 `asserttt` 提供。

函数调用 `assertType<T>(v)` 断言（动态）值 `v` 具有类型 `T`：

```ts
import { assertType } from 'asserttt';

let value = 123;
assertType<number>(value);

```

### 5.3 类型级别：`Assert<B>`

`asserttt` 还提供了实用类型 `Assert<B>`，它断言类型 `B`（通常是实例化的泛型类型）为 `true`：

```ts
import { type Assert, type Equal, type Not } from 'asserttt';

type Pair<X> = [X, X];
type _ = [
 Assert<Equal<
 Pair<'a'>, ['a', 'a']
 >>,
 Assert<Not<Equal<
 Pair<'a'>, ['x', 'x']
 >>>,
];

```

`asserttt` 有几个 *谓词*（可以构造布尔值的泛型类型），我们可以与 `Assert<>` 一起使用。在上一个例子中，我们使用了：

+   `Equal<T1, T2>`

+   `Not<B>`

### 5.4 类型级别：`@ts-expect-error`

在本书中，`@ts-expect-error` 用于显示 TypeScript 编译器错误：

```ts
// @ts-expect-error: The value 'null' cannot be used here.
const value = null.myProp;

```

TypeScript 如何处理这样的指令？

+   如果在 `@ts-expect-error` 注释之后的某一行存在错误，则该错误将被忽略，编译成功。

+   如果没有错误，TypeScript 会抱怨：

    ```ts
    Unused '@ts-expect-error' directive.

    ```

换句话说：TypeScript 检查是否存在错误，但不检查错误的具体内容。`@ts-expect-error` 之后的所有文本（包括冒号）都将被忽略。

为了进行更彻底的检查，我使用了工具 `ts-expect-error`，该工具检查被抑制的错误消息是否与 `@ts-expect-error:` 之后的文字匹配。

### 5.5   这本书的标记法是不是有点丑？

当涉及到显示 TypeScript 代码的类型信息时，有一些非常漂亮的方法 – 例如 [Shiki Twoslash](https://shikijs.github.io/twoslash/)，它使用了 [twoslash 语法](https://github.com/microsoft/TypeScript-Website/tree/v2/packages/ts-twoslasher)。

尽管看起来不那么美观，这本书仍然使用代码中的检查（如上所述）。为什么？

+   这种标记法让你从测试的角度思考类型。这为你准备 计算类型 和 编码练习 – 其标记法类似。

+   这种标记法使得可以通过 [Markcheck](https://github.com/rauschma/markcheck) 工具自动测试代码示例，确保它们不包含错误。Twoslash 只指定要显示的类型；它并不检查这些类型是否符合预期。

+   对于印刷书籍，HTML 仍然不是我希望的样子。因此，我无法在那里使用 Shiki Twoslash。

+   Shiki Twoslash 的小缺点：你需要运行 TypeScript 类型检查器才能渲染一本书。使用我的标记法，我只有在检查代码示例时才需要运行它。
