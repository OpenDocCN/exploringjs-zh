# 29   满足操作符

> 原文：[`exploringjs.com/ts/book/ch_satisfies.html`](https://exploringjs.com/ts/book/ch_satisfies.html)

(广告，请勿拦截。)

1.  29.1   什么是 `satisfies` 操作符？

    1.  29.1.1   语法：`satisfies` 前不允许有行终止符

    1.  29.1.2   一个示例

    1.  29.1.3   示例：可选对象属性

1.  29.2   检查对象属性值类型

    1.  29.2.1   改进：属性值的类型

    1.  29.2.2   改进：通过 `satisfies` 检查属性值

1.  29.3   检查对象属性键类型

1.  29.4   限制字面量值

    1.  29.4.1   示例：通过 `fetch()` 发送 JSON

    1.  29.4.2   示例：`export default`

1.  29.5   `satisfies` 不总是必需的

    1.  29.5.1   示例：`string` 和 `number` 的联合

    1.  29.5.2   示例：区分联合

1.  29.6   `satisfies` 可以改变推断类型

    1.  29.6.1   从类型到字面量类型

    1.  29.6.2   从数组到元组

    1.  29.6.3   `satisfies` 不会改变显式类型

1.  29.7   类型级别满足检查

1.  29.8   进一步阅读

`satisfies` 操作符让我们可以在不影响它的情况下（主要是）检查值的类型。在本章中，我们将探讨它如何精确工作以及它在哪里有用。

### 29.1   什么是 `satisfies` 操作符？

`satisfies` 操作符在编译时强制给定 `value` 可分配给给定的 `Type`：

```ts
value satisfies Type

```

+   结果仍然是 `value`。此操作符在运行时没有效果。

+   `value` 的类型通常不会改变。尽管如此，也有一些例外，我们将在稍后讨论。

#### 29.1.1   语法：`satisfies` 前不允许有行终止符

这是不允许的：

```ts
const sayHello = (name) => `Hello ${name}!`
 // @ts-expect-error: Cannot find name 'satisfies'.
 satisfies (str: string) => string;

```

括号可以有所帮助：

```ts
const sayHello = (
 (name) => `Hello ${name}!`
) satisfies (str: string) => string;

```

#### 29.1.2   一个示例

让我们检查各种注解如何影响对象字面量的类型。我们从没有任何注解的情况开始：

```ts
const point1 = { x: 2, y: 5 };
assertType<
 { x: number, y: number }
>(point1);

```

TypeScript 将 `.x` 和 `.y` 的类型泛化为 `number`。如果我们使用 `as const` 注解，这会发生变化：

```ts
const point2 = { x: 2, y: 5 } as const;
assertType<
 { readonly x: 2, readonly y: 5 }
>(point2);

```

现在两个属性都是只读的，并且具有狭窄的类型。如果我们想通过一个新的类型 `Point` 检查我们点的形状是否正确，会发生什么？

```ts
type Point = { x: number, y: number };
const point3: Point = { x: 2, y: 5 } as const;
assertType<
 Point
>(point3);

```

`point3` 再次具有更广泛的类型。TypeScript 推断类型名称 `Point`，它是 `point1` 所具有类型的别名。我们声明该变量时没有添加任何类型级别的注解。

`satisfies` 允许我们检查我们的点具有正确的形状，同时不丢弃 `as const` 给我们的狭窄类型：

```ts
const point4 = { x: 2, y: 5 } as const satisfies Point;
assertType<
 { readonly x: 2, readonly y: 5 }
>(point4);

```

`satisfies` 与 `as` 有何不同？一方面，`as` 通常会改变其左侧的类型。另一方面，它不像 `satisfies` 那样彻底地进行类型检查：

```ts
// Should warn about missing property but doesn’t!
const point5 = { x: 2 } as const as Point; // OK
assertType<
 Point
>(point5);

```

相比之下，如果省略属性 `.y`，`satisfies` 会警告我们：

```ts
// @ts-expect-error: Type '{ readonly x: 2; }' does not satisfy
// the expected type 'Point'.
const point6 = { x: 2 } as const satisfies Point;

```

为什么我们想要点的类型是狭窄的？实际上，我们通常对更广泛的类型感到非常满意。但是，有一些用例需要它们是狭窄的。我们将在下一节中查看这些用例。

#### 29.1.3 示例：可选对象属性

以下代码演示了 `partialPoint1` 具有更广泛的类型 `PartialPoint` 使得它不太容易使用：

```ts
type PartialPoint = { x?: number, y?: number };

const partialPoint1: PartialPoint = { y: 7 };

// Should be an error
const x1 = partialPoint1.x;
assertType<number | undefined>(x1);

// Type should be `number`
const y1 = partialPoint1.y;
assertType<number | undefined>(y1);

```

这是我们使用 `satisfies` 时会发生的情况：

```ts
const partialPoint2 = { y: 7 } satisfies PartialPoint;

// @ts-expect-error: Property 'x' does not exist on type
// '{ y: number; }'.
const x2 = partialPoint2.x;

const y2 = partialPoint2.y;
assertType<number>(y2);

```

### 29.2 类型检查对象属性值

以下对象实现了一个用于文本样式的枚举：

```ts
const TextStyle = {
 Bold: {
 html: 'b',
 latex: 'textbf',
 },
 Italics: {
 html: 'i',
 // Missing: latex
 },
};

type TextStyleKeys = keyof typeof TextStyle;
type _ = Assert<Equal<
 TextStyleKeys, "Bold" | "Italics"
>>;

```

+   一方面，我们可以从属性键推导出类型 `TextStyleKeys`，这是很整洁的。

+   另一方面，我们忘记了属性 `TextStyle.Italics.latex`，TypeScript 没有警告我们。

#### 29.2.1 改进：属性值的类型

让我们使用以下类型来检查属性值：

```ts
type TTextStyle = {
 html: string,
 latex: string,
};

```

我们第一次尝试看起来是这样的：

```ts
const TextStyle: Record<string, TTextStyle>  = {
 Bold: {
 html: 'b',
 latex: 'textbf',
 },
 // @ts-expect-error: Property 'latex' is missing in type
 // '{ html: string; }' but required in type 'TTextStyle'.
 Italics: {
 html: 'i',
 },
};

type TextStyleKeys = keyof typeof TextStyle;
type _ = Assert<Equal<
 TextStyleKeys, string
>>;

```

+   优点：我们得到一个警告，表明属性缺失。

+   缺点：我们不能再提取属性键了——`TextStyleKeys` 现在是 `string` 类型。

#### 29.2.2 改进：通过 `satisfies` 检查属性值

再次，`satisfies` 可以帮助我们：

```ts
const TextStyle  = {
 Bold: {
 html: 'b',
 latex: 'textbf',
 },
 // @ts-expect-error: Property 'latex' is missing in type
 // '{ html: string; }' but required in type 'TTextStyle'.
 Italics: {
 html: 'i',
 },
} satisfies Record<string, TTextStyle>;

type TextStyleKeys = keyof typeof TextStyle;
type _ = Assert<Equal<
 TextStyleKeys, "Bold" | "Italics"
>>;

```

现在，我们得到一个关于缺失属性的警告和一个有用的类型 `TextStyleKeys`。

### 29.3 类型检查对象属性键

在前面的示例中，我们检查了属性值的形状。有时我们还要处理有限的一组属性键，并希望检查这些键——以避免拼写错误等。

以下示例受 [对 `satisfies` 的 pull request](https://github.com/microsoft/TypeScript/issues/47920) 的启发，并使用类型 `ColorName` 来检查属性键，使用类型 `Color` 来检查属性值。我们得到一个错误，因为缺少键 `'blue'`：

```ts
type ColorName = 'red' | 'green' | 'blue';
type Color =
 | string // hex
 | [number, number, number] // RGB
;
const fullColorTable = {
 red: [255, 0, 0],
 green: '#00FF00',
// @ts-expect-error:
// Type '{ red: [number, number, number]; green: string; }'
// does not satisfy the expected type 'Record<ColorName, Color>'.
} satisfies Record<ColorName, Color>;

```

如果我们不想使用所有键，但仍想检查拼写错误，可以通过使对象的记录部分化（行 A）来实现：

```ts
const partialColorTable = {
 red: [255, 0, 0],
 // @ts-expect-error: Object literal may only specify known
 // properties, but 'greenn' does not exist in type
 // 'Partial<Record<ColorName, Color>>'.
 // Did you mean to write 'green'?
 greenn: '#00FF00',
} satisfies Partial<Record<ColorName, Color>>; // (A)

```

我们还可以提取属性键：

```ts
const partialColorTable2 = {
 red: [255, 0, 0],
 green: '#00FF00',
} satisfies Partial<Record<ColorName, Color>>;

type PropKeys = keyof typeof partialColorTable2;
type _ = Assert<Equal<
 PropKeys, "red" | "green"
>>;

```

### 29.4 限制字面量值

考虑以下函数调用：

```ts
JSON.stringify({ /*···*/ });

```

`JSON.stringify()` 函数的参数类型为 `any`。我们如何确保传递给它的对象具有正确的形状（属性键没有拼写错误等）？一个选项是创建一个对象变量：

```ts
const obj: SomeType = { /*···*/ };
JSON.stringify(obj);

```

另一个选项是使用 `satisfies`：

```ts
JSON.stringify({ /*···*/ } satisfies SomeType);

```

以下小节展示了这种技术在几个用例中的应用。

#### 29.4.1 示例：通过 `fetch()` 发送 JSON

以下代码受到了 [Matt Pocock 的文章](https://www.totaltypescript.com/how-to-use-satisfies-operator#strongly-typed-post-request-with-satisfies) 的启发：我们正在使用 `fetch()` 向 API 发送数据。

```ts
type Product = {
 name: string,
 quantity: number,
};

const response = await fetch('/api/products', {
 method: 'POST',
 body: JSON.stringify(
 {
 name: 'Toothbrush',
 quantity: 3,
 } satisfies Product
 ),
 headers: {
 'Content-Type': 'application/json',
 },
});

```

#### 29.4.2 示例：`export default`

内联命名导出可以有类型注解：

```ts
import type { StringCallback } from './core.js';

export const toUpperCase: StringCallback =
 (str) => str.toUpperCase();

```

使用默认导出时，没有方法可以添加一个，但我们可以使用 `satisfies`：

```ts
import type { StringCallback } from './core.js';

export default (
 (str) => str.toUpperCase()
) satisfies StringCallback;

```

### 29.5 `satisfies` 不总是必需的

在本节中，我们查看那些似乎需要 `satisfies` 的代码，但 TypeScript 已经正确缩小了代码。

#### 29.5.1 示例：`string` 和 `number` 的联合

在以下代码中，我们不需要对 `str1` 和 `str2` 使用 `satisfies` – 它们已经被缩小到 `string`：

```ts
type StrOrNum = string | number;

const str1 = 'abc' as StrOrNum;
// @ts-expect-error: Property 'toUpperCase' does not exist on
// type 'StrOrNum'.
str1.toUpperCase();

const str2: StrOrNum = 'abc';
str2.toUpperCase(); // OK

let str3: StrOrNum = 'abc';
str3.toUpperCase(); // OK

```

#### 29.5.2 示例：区分联合

在下一个示例中，`linkToIntro` 也被缩小到区分联合 `LinkHref` 的一个元素：

```ts
type LinkHref =
 | {
 kind: 'LinkHrefUrl',
 url: string,
 }
 | {
 kind: 'LinkHrefId',
 id: string,
 }
;
const linkToIntro: LinkHref = {
 kind: 'LinkHrefId',
 id: '#intro',
};
// Type was narrowed:
assertType<
 {
 kind: 'LinkHrefId',
 id: string,
 }
>(linkToIntro);

```

### 29.6 `satisfies` 可以改变推断类型

虽然 `satisfies` 通常不会改变值的推断类型，但也有一些例外。

#### 29.6.1 从类型到字面量类型

```ts
type Robin = {
 name: 'Robin',
};

const robin1 = { name: 'Robin' };
assertType<string>(robin1.name);

const robin2 = { name: 'Robin' } satisfies Robin;
assertType<'Robin'>(robin2.name);

```

#### 29.6.2 从数组到元组

```ts
// No `satisfies`
const tuple1 = ['a', 1];
assertType<
 (string | number)[]
>(tuple1);

// Non-empty tuple
const tuple2 = ['a', 1] satisfies [unknown, ...unknown[]];
assertType<
 [string, number]
>(tuple2);

// Any tuple
const tuple3 = [] satisfies [unknown?, ...unknown[]];
assertType<
 []
>(tuple3);

// Any tuple
const tuple4 = ['a', 1] satisfies [] | unknown[];
assertType<
 [string, number]
>(tuple4);

```

#### 29.6.3 `satisfies` 不会改变显式类型

```ts
const tuple1: Array<string | number> = ['a', 1];

// @ts-expect-error: Type '(string | number)[]' does not satisfy
// the expected type '[unknown, ...unknown[]]'.
const tuple2 = tuple1 satisfies [unknown, ...unknown[]];

```

### 29.7 类型级别的满足性检查

TypeScript 确实有一个在类型级别上类似于 `satisfies` 的操作，但我们可以自己实现它：

```ts
type Satisfies<Type extends Constraint, Constraint> = Type;

type T1 = Satisfies<123, number>; // 123
type _ = Assert<Equal<
 T1, 123
>>;

// @ts-expect-error: Type 'number' does not satisfy
// the constraint 'string'.
type T2 = Satisfies<123, string>;

```

### 29.8 进一步阅读

+   提交请求 [“`satisfies` 操作符以确保表达式匹配某些类型”](https://github.com/microsoft/TypeScript/issues/47920)
