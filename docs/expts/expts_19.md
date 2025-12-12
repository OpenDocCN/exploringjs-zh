# 15 最低类型 `never`

> 原文：[`exploringjs.com/ts/book/ch_never.html`](https://exploringjs.com/ts/book/ch_never.html)

（广告，请勿拦截。）

1.  15.1 `never` 是一个底类型

1.  15.2 `never` 是空集

1.  15.3 `never` 的用例：过滤联合类型

1.  15.4 `never` 的用例：编译时进行完备性检查

    1.  15.4.1 完备性检查和 `if`

1.  15.5 `never` 的用例：禁止属性

1.  15.6 返回 `never` 的函数

1.  15.7 本章来源

在本章中，我们探讨特殊的 TypeScript 类型 `never`，它大致上表示永远不会发生的事情的类型。正如我们将看到的，它有令人惊讶的众多应用。

### 15.1 `never` 是一个底类型

如果我们将类型解释为值的集合，那么：

+   类型 `Sub` 是类型 `Sup` 的子类型（`Sub <: Sup`）。

+   如果 `Sub` 是 `Sup` 的子集（`Sub ⊂ Sup`）。

两种类型是特殊的：

+   顶级类型 `T` 包含所有值，所有类型都是 `T` 的子类型。

+   底类型 `B` 是空集，并且是所有类型的子类型。

在 TypeScript 中：

+   `any` 和 `unknown` 是顶级类型，在“顶级类型 `any` 和 `unknown`”（§14）中进行了解释。

+   `never` 是一个底类型。

### 15.2 `never` 是空集

当使用类型进行计算时，类型联合有时用来表示（类型级别的）值的集合。那么空集由 `never` 表示：

```ts
type _ = [
 Assert<Equal<
 keyof {a: 1, b: 2},
 'a' | 'b' // set of types
 >>,
 Assert<Equal<
 keyof {},
 never // empty set
 >>,
];

```

同样，如果我们使用类型运算符 `&` 来交集两个没有共同元素的类型，我们得到空集：

```ts
type _ = Assert<Equal<
 boolean & symbol,
 never
>>;

```

如果我们使用类型运算符 `|` 来计算类型 `T` 和 `never` 的联合，结果是 `T`：

```ts
type _ = Assert<Equal<
 'a' | 'b' | never,
 'a' | 'b'
>>;

```

### 15.3 `never` 的用例：过滤联合类型

我们可以使用条件类型来过滤联合类型：

```ts
type KeepStrings<T> = T extends string ? T : never;
type _ = [
 Assert<Equal<
 KeepStrings<'abc'>, // normal instantiation
 'abc'
 >>,
 Assert<Equal<
 KeepStrings<123>, // normal instantiation
 never
 >>,
 Assert<Equal<
 KeepStrings<'a' | 'b' | 0 | 1>, // distributed instantiation
 'a' | 'b'
 >>,
];

```

我们使用两种现象来实现这一点：

+   当我们将条件类型应用于联合类型时，它是 *分配的* – 应用到联合的每个元素。

+   在类型的结果联合中，`KeepStrings` 的假分支返回的 `never` 类型消失了（参见上一节）。

更多信息：“通过条件返回 `never` 过滤联合类型”（§34.3）

### 15.4 `never` 的用例：编译时进行完备性检查

使用类型推断，TypeScript 会跟踪变量仍然可能具有的值 – 例如：

```ts
function f(x: boolean): void {
 assertType<false | true>(x); // (A)
 if (x === true) {
 return;
 }
 assertType<false>(x); // (B)
 if (x === false) {
 return;
 }
 assertType<never>(x); // (C)
}

```

在行 A 中，`x` 仍然可以有 `false` 和 `true` 的值。在我们返回 `x` 有 `true` 值之后，它仍然可以有 `false` 的值（行 B）。在我们返回 `x` 有 `false` 值之后，这个变量不再有其他值，这就是为什么它有 `never` 类型（行 C）。

这种行为对于像枚举一样使用的枚举和联合特别有用，因为它使 *完备性检查*（检查我们是否已全面处理所有情况）成为可能：

```ts
enum Color { Red, Green }

```

以下模式对 JavaScript 很有效，因为它在运行时检查 `color` 是否有意外值：

```ts
function colorToString(color: Color): string {
 switch (color) {
 case Color.Red:
 return 'RED';
 case Color.Green:
 return 'GREEN';
 default:
 throw new UnexpectedValueError(color);
 }
}

```

我们如何从类型级别支持这种模式，以便在意外未考虑枚举 `Color` 的所有成员时发出警告？（返回类型 `string` 也使我们保持安全，但使用我们即将看到的技巧，即使没有返回值，我们也能得到保护。此外，我们还在运行时受到非法值的保护。）

让我们先看看当我们添加案例时，`color` 的推断值是如何变化的：

```ts
function exploreSwitch(color: Color) {
 switch (color) {
 default:
 assertType<Color.Red | Color.Green>(color);
 }
 switch (color) {
 case Color.Red:
 break;
 default:
 assertType<Color.Green>(color);
 }
 switch (color) {
 case Color.Red:
 break;
 case Color.Green:
 break;
 default:
 assertType<never>(color);
 }
}

```

再次强调，类型记录了 `color` 仍然可以具有的值。

以下 `UnexpectedValueError` 类的实现要求其实际参数的类型为 `never`：

```ts
class UnexpectedValueError extends Error {
 constructor(
 // Type enables type checking
 value: never,
 // Only solution that can stringify undefined, null, symbols, and
 // objects without prototypes
 message = `Unexpected value: ${{}.toString.call(value)}`
 ) {
 super(message)
 }
}

```

现在如果我们忘记了一个案例，因为我们没有消除 `color` 可以具有的所有值，我们会得到一个编译时警告：

```ts
function colorToString(color: Color): string {
 switch (color) {
 case Color.Red:
 return 'RED';
 default:
 assertType<Color.Green>(color);
 // @ts-expect-error: Argument of type 'Color.Green' is not
 // assignable to parameter of type 'never'.
 throw new UnexpectedValueError(color);
 }
}

```

#### 15.4.1 关于 `if` 的完备性检查

如果我们通过 `if` 处理案例，完备性检查也适用：

```ts
function colorToString(color: Color): string {
 assertType<Color.Red | Color.Green>(color);
 if (color === Color.Red) {
 return 'RED';
 }
 assertType<Color.Green>(color);
 if (color === Color.Green) {
 return 'GREEN';
 }
 assertType<never>(color);
 throw new UnexpectedValueError(color);
}

```

### 15.5 `never` 的用例：禁止属性

由于没有其他类型可以赋值给 `never`，我们可以用它来禁止属性——例如具有字符串键的属性：

```ts
type EmptyObject = Record<string, never>;

// @ts-expect-error: Type 'number' is not assignable to type 'never'.
const obj1: EmptyObject = { prop: 123 };
const obj2: EmptyObject = {}; // OK

```

如需更多信息，请参阅“通过 `never` 禁止属性”（§18.6）。

### 15.6 返回 `never` 的函数

`never` 还可以作为永远不会返回的函数的标记——例如：

```ts
function throwError(message: string): never {
 throw new Error(message);
}

```

如果我们调用这样的函数，TypeScript 就知道执行结束，并相应地调整推断的类型。有关更多信息，请参阅“返回类型 `never`：不返回的函数”（§27.4）。

### 15.7 本章来源

+   在“宣布 TypeScript 3.7”中，丹尼尔·罗森沃瑟的章节[“对 `never` 返回函数的更好支持”](https://devblogs.microsoft.com/typescript/announcing-typescript-3-7/#better-support-for-never-returning-functions)。

+   斯坦凡·鲍姆加特纳的博客文章[“TypeScript 中的 `never` 类型与错误处理”](https://oida.dev/typescript-never-and-error-handling/)。
