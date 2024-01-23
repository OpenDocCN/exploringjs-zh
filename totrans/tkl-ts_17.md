# 14 向类型添加特殊值

> 原文：[`exploringjs.com/tackling-ts/ch_special-values.html`](https://exploringjs.com/tackling-ts/ch_special-values.html)

* * *

+   14.1 在带内添加特殊值

    +   14.1.1 向类型添加 `null` 或 `undefined`

    +   14.1.2 向类型添加符号

+   14.2 在带外添加特殊值

    +   14.2.1 辨别式联合

    +   14.2.2 其他类型的联合类型

* * *

理解类型的一种方式是将其视为值的集合。有时会有两个级别的值：

+   基本级别：普通值

+   元级别：特殊值

在本章中，我们将探讨如何向基本级别类型添加特殊值。

### 14.1 在带内添加特殊值

添加特殊值的一种方法是创建一个新类型，它是基本类型的超集，其中一些值是特殊的。这些特殊值被称为 [*哨兵*](https://en.wikipedia.org/wiki/Sentinel_value)。它们存在于 [*带内*](https://en.wikipedia.org/wiki/In-band_signaling)（想象在同一通道内），作为普通值的兄弟。

例如，考虑可读流的以下接口：

```ts
interface InputStream {
 getNextLine(): string;
}
```

目前，`.getNextLine()` 只处理文本行，而不处理文件结束（EOF）。我们如何为 EOF 添加支持？

可能的方法包括：

+   在调用 `.getNextLine()` 之前需要调用一个额外的方法 `.isEof()`。

+   当到达 EOF 时，`.getNextLine()` 会抛出异常。

+   EOF 的哨兵值。

接下来的两个小节描述了引入哨兵值的两种方法。

#### 14.1.1 向类型添加 `null` 或 `undefined`

在使用严格的 TypeScript 时，没有简单的对象类型（通过接口、对象模式、类等定义）包括 `null`。这使得它成为一个可以通过联合类型添加到基本类型 `string` 的良好哨兵值：

```ts
type StreamValue = null | string;

interface InputStream {
 getNextLine(): StreamValue;
}
```

现在，每当我们使用 `.getNextLine()` 返回的值时，TypeScript 强制我们考虑两种可能性：字符串和 `null`，例如：

```ts
function countComments(is: InputStream) {
 let commentCount = 0;
 while (true) {
 const line = is.getNextLine();
 // @ts-expect-error: Object is possibly 'null'.(2531)
 if (line.startsWith('#')) { // (A)
 commentCount++;
 }
 if (line === null) break;
 }
 return commentCount;
}
```

在 A 行，我们不能使用字符串方法 `.startsWith()`，因为 `line` 可能是 `null`。我们可以按照以下方式修复这个问题：

```ts
function countComments(is: InputStream) {
 let commentCount = 0;
 while (true) {
 const line = is.getNextLine();
 if (line === null) break;
 if (line.startsWith('#')) { // (A)
 commentCount++;
 }
 }
 return commentCount;
}
```

现在，当执行到 A 行时，我们可以确信 `line` 不是 `null`。

#### 14.1.2 向类型添加符号

我们还可以使用除 `null` 之外的值作为哨兵。符号和对象最适合这个任务，因为它们每个都有唯一的标识，没有其他值可以被误认为它。

这是如何使用符号来表示 EOF 的方法：

```ts
const EOF = Symbol('EOF');
type StreamValue = typeof EOF | string;
```

为什么我们需要 `typeof` 而不能直接使用 `EOF`？那是因为 `EOF` 是一个值，而不是一种类型。类型操作符 `typeof` 将 `EOF` 转换为一种类型。有关值和类型的不同语言级别的更多信息，请参见 §7.7 “两种语言级别：动态 vs. 静态”。

### 14.2 在带外添加特殊值

如果一个方法可能返回 *任何* 值，我们该怎么办？如何确保基本值和元值不会混淆？这是可能发生的一个例子：

```ts
interface InputStream<T> {
 getNextValue(): T;
}
```

无论我们选择什么值作为 `EOF`，都存在某人创建 `InputStream<typeof EOF>` 并将该值添加到流的风险。

解决方法是将普通值和特殊值分开，这样它们就不会混淆。特殊值单独存在被称为 [*带外*](https://en.wikipedia.org/wiki/Out-of-band_data)（想象不同的通道）。

#### 14.2.1 辨别式联合

*辨别式联合* 是几个对象类型的联合类型，它们都至少有一个共同的属性，即所谓的 *辨别器*。辨别器必须对每个对象类型具有不同的值 - 我们可以将其视为对象类型的 ID。

##### 14.2.1.1 示例：`InputStreamValue`

在下面的例子中，`InputStreamValue<T>` 是一个辨别联合，其辨别标志是`.type`。

```ts
interface NormalValue<T> {
 type: 'normal'; // string literal type
 data: T;
}
interface Eof {
 type: 'eof'; // string literal type
}
type InputStreamValue<T> = Eof | NormalValue<T>;

interface InputStream<T> {
 getNextValue(): InputStreamValue<T>;
}
```

```ts
function countValues<T>(is: InputStream<T>, data: T) {
 let valueCount = 0;
 while (true) {
 // %inferred-type: Eof | NormalValue<T>
 const value = is.getNextValue(); // (A)

 if (value.type === 'eof') break;

 // %inferred-type: NormalValue<T>
 value; // (B)

 if (value.data === data) { // (C)
 valueCount++;
 }
 }
 return valueCount;
}
```

最初，`value` 的类型是 `InputStreamValue<T>`（A 行）。然后我们排除了辨别标志`.type`的值`'eof'`，它的类型被缩小为`NormalValue<T>`（B 行）。这就是为什么我们可以在 C 行访问属性`.data`。

##### 14.2.1.2 示例：`IteratorResult`

在决定如何实现[迭代器](https://exploringjs.com/impatient-js/ch_sync-iteration.html)时，TC39 不想使用固定的哨兵值。否则，该值可能出现在可迭代对象中并破坏代码。一种解决方案是在开始迭代时选择一个哨兵值。相反，TC39 选择了一个带有共同属性`.done`的辨别联合：

```ts
interface IteratorYieldResult<TYield> {
 done?: false; // boolean literal type
 value: TYield;
}

interface IteratorReturnResult<TReturn> {
 done: true; // boolean literal type
 value: TReturn;
}

type IteratorResult<T, TReturn = any> =
 | IteratorYieldResult<T>
 | IteratorReturnResult<TReturn>;
```

#### 14.2.2 其他类型的联合类型

其他类型的联合类型可以像辨别联合一样方便，只要我们有手段来区分联合的成员类型。

一种可能性是通过唯一属性来区分成员类型：

```ts
interface A {
 one: number;
 two: number;
}
interface B {
 three: number;
 four: number;
}
type Union = A | B;

function func(x: Union) {
 // @ts-expect-error: Property 'two' does not exist on type 'Union'.
 // Property 'two' does not exist on type 'B'.(2339)
 console.log(x.two); // error

 if ('one' in x) { // discriminating check
 console.log(x.two); // OK
 }
}
```

另一种可能性是通过`typeof`和/或实例检查来区分成员类型：

```ts
type Union = [string] | number;

function logHexValue(x: Union) {
 if (Array.isArray(x)) { // discriminating check
 console.log(x[0]); // OK
 } else {
 console.log(x.toString(16)); // OK
 }
}
```

[评论](https://github.com/rauschma/tackling-ts/issues/14)
