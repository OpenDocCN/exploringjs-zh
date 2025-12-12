# 39 测试类型

> 原文：[`exploringjs.com/ts/book/ch_testing-types.html`](https://exploringjs.com/ts/book/ch_testing-types.html)

（广告，请勿拦截。）

1.  39.1 在类型级别进行断言

1.  39.2 如何检查类型是否为 `any`？

1.  39.3 如何检查两个类型是否相等？

    1.  39.3.1 第一次尝试：`TMutuallyAssignable`

    1.  39.3.2 禁用分发

    1.  39.3.3 确保任何类型仅等于自身

1.  39.4 如何断言某物必须是 `true`？

    1.  39.4.1 `Not`：用于布尔取反的实用类型

1.  39.5 断言错误

    1.  39.5.1 一个更复杂的例子

1.  39.6 断言值的类型

1.  39.7 在普通代码中使用类型级别断言的用例

1.  39.8 运行类型级别测试

1.  39.9 进一步阅读

在本章中，我们探讨了我们如何测试复杂的 TypeScript 类型是否按预期工作。为此，我们需要类型级别的断言和其他工具。

### 39.1 在类型级别进行断言

编写更复杂的类型就像在不同的级别上进行编程：

+   在程序级别，我们使用 JavaScript – 例如：

    +   值

    +   带有参数的函数

+   在类型级别，我们使用（非 JavaScript）TypeScript – 例如：

    +   类型

    +   带有类型参数的泛型类型

在程序级别，我们可以使用断言，如 `assert.deepEqual()`（[`nodejs.org/api/assert.html#assertdeepequalactual-expected-message`](https://nodejs.org/api/assert.html#assertdeepequalactual-expected-message)）来测试我们的代码：

```ts
const pair = (x) => [x, x];
const result = pair('abc');
assert.deepEqual(
 result, ['abc', 'abc']
);

```

因此，我们如何测试类型级别的代码 – 这对于复杂的类型非常重要？我们还需要断言 – 例如：

```ts
type Pair<T> = [T, T];
type Result = Pair<'abc'>;
type _ = Assert<Equal<
 Result, ['abc', 'abc']
>>;

```

泛型类型 `Assert` 和 `Equal` 是我 npm 包 `asserttt` 的一部分。在本章中，我们将以两种方式使用此包：

+   一方面，我们重新实现其 API 以查看其工作原理。

+   另一方面，我们使用其 API 来检查我们所实现的功能是否按预期工作。

为了避免混淆，我们类型的名称总是以 `T` 开头：

+   `asserttt` 类型：`MutuallyAssignable`

+   我们的版本：`TMutuallyAssignable`

### 39.2 如何检查类型是否为 `any`？

在本节中，我们探讨如何检查给定的类型是否为 `any` 类型。这将有助于我们稍后定义 `TEqual<X, Y>`。

我们如何仅通过检查通过 `extends` 的可赋性来检测 `any` 类型？需要考虑的特殊类型有：

+   `any` 可以赋值给任何类型，也可以从任何类型赋值 – 唯一的例外是它不能赋值给 `never`。

+   任何类型都可以赋值给 `unknown`。`unknown` 不能赋值给任何类型（除了 `unknown` 和 `any`）。

+   `never` 可以赋值给任何类型。没有类型（除了 `never`）可以赋值给 `never`；甚至 `any` 也不能赋值给 `never`。

因此，一个类型 `T` 是 `any` 如果且仅当：

1.  `T` 可以赋值给数字字面量类型 `1`。

1.  数字字面量类型 `2` 可以赋值给 `T`。

在检查 1 之后，`T` 可以是 `1`、`never` 或 `any`。检查 2 排除了 `1` 和 `never`。这是两个检查的实现：

```ts
type TIsAny<T> = [T, 2] extends [1, T] ? true : false;
type _ = [
 Assert<Equal<
 TIsAny<any>, true
 >>,
 Assert<Equal<
 TIsAny<unknown>, false
 >>,
 Assert<Equal<
 TIsAny<never>, false
 >>,
 Assert<Equal<
 TIsAny<1>, false
 >>,
 Assert<Equal<
 TIsAny<2>, false
 >>,
];

```

[一个更简洁的解决方案](https://stackoverflow.com/questions/49927523/disallow-call-with-any/49928360#49928360) 由 Joe Calzaretta 提出：

```ts
type TIsAny<T> = 0 extends (1 & T) ? true : false;

```

这段代码是如何工作的？

+   条件 `0 extends 1` 失败。它检查 `0` 是否是 `1` 的子集（如果 `0` 可以赋值给 `1`）。

+   对于所有正常类型 `T`，交集 `1 & T` 的大小与类型 `1` 或更小。因此，检查的结果保持不变。

+   只有类型 `any` 是不同的，因为 `1 & any` 是 `any`，而 `0` 是 `any` 的子集。

    ```ts
    type _X = Assert<Equal<
     1 & any,
     any
    >>;

    ```

### 39.3 如何检查两个类型是否相等？

类型级断言 API 最重要的部分是检查两个类型是否相等。结果证明，这出奇地困难。

#### 39.3.1 第一次尝试：`TMutuallyAssignable`

两个 `X` 和 `Y` 相等实际上意味着什么？一个合理的定义是：

+   `X` 可以赋值给 `Y`，

+   `Y` 可以赋值给 `X`。

我们通过 `extends` 来检查赋值性：

```ts
type TMutuallyAssignable<X, Y> =
 X extends Y
 ? (Y extends X ? true : false)
 : false
;
type _ = [
 Assert<Equal<
 TMutuallyAssignable<'hello', 'hello'>, // (A)
 true
 >>,
 Assert<Equal<
 TMutuallyAssignable<'yes', 'no'>, // (B)
 false
 >>,
 Assert<Equal<
 TMutuallyAssignable<string, 'yes'>, // (C)
 false
 >>,
 Assert<Equal<
 TMutuallyAssignable<'a', 'a'|'b'>, // (D)
 true | false
 >>,
];

```

测试用例开始得很好：第 A 行和第 B 行产生了预期的结果，甚至第 C 行的更复杂的检查也正确无误。

啊，在第 D 行，结果既是 `true` 也是 `false`。为什么是这样？`TMutuallyAssignable` 是使用条件类型定义的，而这些类型在联合类型上是分配的（更多信息）。

#### 39.3.2 禁用分配

为了让 `TMutuallyAssignable` 如预期那样工作，我们需要关闭分配。一个条件类型只有在 `extends` 的左侧是一个裸类型变量时才是分配的（更多信息）。因此，我们可以通过将 `extends` 的两边都变成单元素元组来禁用分配：

```ts
type TMutuallyAssignable<X, Y> =
 [X] extends [Y]
 ? ([Y] extends [X] ? true : false)
 : false
;

```

我们可以通过同时进行两个 `extends` 检查来稍微简化这个 `TMutuallyAssignable`（方括号仍然确保不会发生分配）：

```ts
export type TMutuallyAssignable<X, Y> =
 [X, Y] extends [Y, X] ? true : false
 ;

```

让我们测试 `TMutuallyAssignable` 的非分配版本：

```ts
type _ = [
 Assert<Equal<
 TMutuallyAssignable<'hello', 'hello'>,
 true
 >>,
 Assert<Equal<
 TMutuallyAssignable<'yes', 'no'>,
 false
 >>,
 Assert<Equal<
 TMutuallyAssignable<string, 'yes'>,
 false
 >>,
 Assert<Equal<
 TMutuallyAssignable<'a', 'a'|'b'>, // (A)
 false
 >>,
 Assert<Equal<
 TMutuallyAssignable<any, 123>, // (B)
 true
 >>,
];

```

现在，我们也可以正确处理联合类型（第 A 行）。然而，一个问题仍然存在（第 B 行）：`any` 等于任何其他类型：

+   如果期望的类型是 `any`，那么所有实际类型都与它相等。

+   如果我们期望一个特定的类型，那么实际的类型 `any` 也总是相等的。

#### 39.3.3 确保 `any` 只等于自身

我们可以使用 `TMutuallyAssignable` 来定义一个泛型类型 `TEqual`，以解决 `any` 的问题（我们使用之前提到的实用工具 `TIsAny`）：

```ts
type TEqual<X, Y> =
 [TIsAny<X>, TIsAny<Y>] extends [true, true] ? true
 : [TIsAny<X>, TIsAny<Y>] extends [false, false] ? MutuallyAssignable<X, Y>
 : false
 ;

```

我们的方法是：

+   如果 `X` 和 `Y` 都是 `any`，它们是相等的。

+   如果 `X` 和 `Y` 都不是 `any`，它们如果可以相互赋值，则它们是相等的。

+   否则，只有 `X` 或只有 `Y` 是 `any`，它们不相等。

`TEqual`通过了以下所有测试：

```ts
type _ = [
 Assert<Equal<
 TEqual<'hello', 'hello'>,
 true
 >>,
 Assert<Equal<
 TEqual<'yes', 'no'>,
 false
 >>,
 Assert<Equal<
 TEqual<string, 'yes'>,
 false
 >>,
 Assert<Equal<
 TEqual<'a', 'a'|'b'>,
 false
 >>,
 Assert<Equal<
 TEqual<any, 123>, // (A)
 false
 >>,
 Assert<Equal<
 TEqual<any, any>, // (B)
 true
 >>,
];

```

`TEqual`通过了 A 行测试，而`TMutuallyAssignable`失败了。B 行包含了对`any`的附加检查，以确保一切正常工作。

### 39.4 我们如何断言某物必须是`true`？

在程序/JavaScript 级别，如果断言失败，我们可以抛出异常：

```ts
function assert(condition) {
 if (condition === false) {
 throw new Error('Assertion failed');
 }
}
function equal(x, y) {
 return x === y;
}

assert(equal(3, 4)); // throws an exception

```

然而，在 TypeScript 中，没有在编译时失败的方法。如果有，它可能看起来像这样：

```ts
type AssertType1<B extends boolean> = B extends true ? void : fail;

```

如果`B`为`true`，则此类型具有与`void`函数相同的结果。如果它是`false`，则通过发明的指令`fail`在类型级别失败。TypeScript 当前最接近`fail`的是返回`never`。然而，`never`不会立即导致类型检查错误。

幸运的是，有一个相当不错的解决方案，基本上满足了我们的需求：

```ts
type AssertType2<_B extends true> = void; // (A)
type _ = [
 AssertType2<true>, // OK
 // @ts-expect-error: Type 'false' does not satisfy
 // the constraint 'true'.
 AssertType2<false>,
];

```

当我们将参数传递给泛型类型时，所有参数的`extends`约束都必须得到满足。否则，我们将得到类型级别的失败。这就是我们在 A 行中使用的方法：传递给`AssertType2`的值必须是`true`，否则类型检查器会报错。

这个解决方案有其局限性。以下功能只能通过`fail`实现：

```ts
type AssertEqual<X, Y> = Equal<X, Y> extends true ? void : fail;

```

#### 39.4.1 `Not`：布尔否定实用类型

一旦我们有了`Equal`和`Assert`，我们就可以实现更多的辅助类型——例如`Not<B>`：

```ts
type TNot<B extends boolean> = [B] extends [true] ? false : true;

```

`B`周围的方括号防止了分配。`TNot`使我们能够断言一个类型不等于另一个类型：

```ts
type _ = Assert<TNot<Equal<
 'yes', 'no'
>>>;

```

结果为`true`或`false`的泛型类型被称为*谓词*。这些类型可以与`Assert`一起使用。`Equal`和`Not`是谓词。但可以设想和使用的谓词更多——例如：

```ts
/**
 * Is type `Target` assignable from type `Source`?
 */
type TAssignable<Target, Source> = [Source] extends [Target] ? true : false;

type _ = [
 Assert<TAssignable<number, 123>>,
 Assert<TAssignable<123, 123>>,

 Assert<Not<TAssignable<123, number>>>,
 Assert<Not<TAssignable<number, 'abc'>>>,
];

```

`asserttt`定义了[更多谓词](https://github.com/rauschma/asserttt/blob/main/src/asserttt.ts)。

### 39.5 断言错误

有时，我们需要测试在预期的地方发生错误。在 JavaScript 级别，我们可以使用如`assert.throws()`之类的函数：

```ts
assert.throws(
 () => null.prop,
 {
 name: 'TypeError',
 message: "Cannot read properties of null (reading 'prop')",
 }
);

```

在类型级别，我们可以使用`@ts-expect-error`：

```ts
// @ts-expect-error: The value 'null' cannot be used here.
null.prop;

```

默认情况下，`@ts-expect-error`只检查*存在*错误，而不是错误的类型。要检查后者，我们可以使用像[ts-expect-error](https://www.npmjs.com/package/ts-expect-error)这样的工具。

#### 39.5.1 一个更复杂的例子

为了娱乐，让我们比较一下 JavaScript 级别的测试与其类型级别的对应项。这是 JavaScript 代码：

```ts
function upperCase(str) {
 if (typeof str !== str) {
 throw new TypeError('Not a string: ' + str);
 }
 return str.toUpperCase();
}
assert.throws(
 () => upperCase(123),
 {
 name: 'TypeError',
 message: 'Not a string: 123',
 }
);

```

对于类型级别的代码，我省略了运行时类型检查——即使这通常仍然有意义。

```ts
function upperCase(str: string) {
 return str.toUpperCase();
}

// @ts-expect-error: Argument of type 'number' is not assignable to
// parameter of type 'string'.
upperCase(123);

```

### 39.6 断言值的类型

在测试类型时，我们还想检查一个值是否具有给定的类型——如通过推断或泛型类型提供。这样做的一种方法是通过`typeof`运算符（A 行）：

```ts
const pair = <T>(x: T): [T, T] => [x, x];
const value = pair('a' as const);
type _ = Assert<Equal<
 typeof value, ['a', 'a'] // (A)
>>;

```

另一个选项是通过辅助函数`assertType()`：

```ts
assertType<['a', 'a']>(value);

```

使用程序级函数可以使这个检查不那么冗长，因为我们可以直接接受程序级值，我们不需要通过 `typeof` 将它们转换为类型级值。这就是 `assertType()` 的样子：

```ts
function assertType<T>(_value: T): void { }

```

我们对参数 `_value` 没有做任何事情；我们只静态检查它是否可以赋值给类型参数 `T`。`assertType()` 的一个限制是它只检查可赋性；它不检查类型相等性。例如，我们无法检查一个值是否具有 `string` 类型而不是更具体的类型：

```ts
const value_string: string = 'abc';
const value_abc: 'abc' = 'abc';

assertType<string>(value_string);
assertType<string>(value_abc);

```

在行 A 中，`value_abc` 的类型可以赋值给 `string`，但它并不等于 `string`。

相比之下，`Equal` 允许我们检查一个值是否不具有比 `string` 更具体的类型：

```ts
type _ = [
 Assert<Equal<
 typeof value_string, string
 >>,
 Assert<Not<Equal<
 typeof value_abc, string
 >>>,
];

```

### 39.7   正常代码中类型级断言的一个用例

有时，类型级断言在常规（非测试）代码中也非常有用。例如，考虑以下枚举模式：

```ts
const OutputFormat = {
 html: 'HTML',
 epub: 'EPUB',
 pdf: 'PDF',
} as const;
type OutputFormatType = (typeof OutputFormat)[keyof typeof OutputFormat];
type OutputFormatKey = keyof (typeof OutputFormat);

```

假设我们想使用 Zod 验证一个 JSON 属性，其值是 `OutputFormat` 的键之一。然后我们需要一个可以传递给 `z.enum()` 的数组（[更多信息](https://zod.dev/?id=zod-enums)）：

```ts
const OUTPUT_FORMAT_KEYS = ['html', 'epub', 'pdf'] as const;

```

我们为什么不通过 `Object.keys()` 创建 `OUTPUT_FORMAT_KEYS`？Zod 需要一个元组以便它可以推断静态类型。

为了确保 `OUTPUT_FORMAT_KEYS` 与 `OutputFormatKey` 一致，我们可以使用以下类型级断言：

```ts
type _ = Assert<Equal<
 (typeof OUTPUT_FORMAT_KEYS)[number], // (A)
 OutputFormatKey
>>;

```

在行 A 中，我们使用索引访问类型 `T[K]` 将数组 `OUTPUT_FORMAT_KEYS` 转换为其元素的类型联合（更多信息）。

### 39.8   运行类型级测试

要运行用 TypeScript 编写的常规测试，我们运行编译后的 JavaScript。如果测试包含类型级断言，我们还需要额外进行类型检查。有两种选择：

+   首先运行 JavaScript。然后通过 `tsc` 对测试进行类型检查。

+   通过像 [tsx](https://tsx.is) 这样的工具运行测试，这些工具在运行代码之前进行类型检查。

### 39.9   进一步阅读

+   各种类型测试方法的概述：2ality 博客文章 “在 TypeScript 中测试静态类型”（[更多信息](https://2ality.com/2022/11/testing-static-types-typescript.html)）
