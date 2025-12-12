# 17 向类型中添加特殊值

> 原文：[`exploringjs.com/ts/book/ch_special-values.html`](https://exploringjs.com/ts/book/ch_special-values.html)

(请勿阻塞，广告。)

1.  17.1 在波段中添加特殊值

    1.  17.1.1 向类型中添加`null`或`undefined`

    1.  17.1.2 向类型中添加符号

1.  17.2 在波段外添加特殊值

    1.  17.2.1 区分联合

    1.  17.2.2 其他类型的联合

理解类型的一种方式是将其视为值的集合。有时存在两个级别的值：

+   基础级别：正常值

+   元素级别：特殊值

在本章中，我们探讨如何向基础类型添加特殊值。

### 17.1 在波段中添加特殊值

添加特殊值的一种方法是通过创建一个新的类型，它是基础类型的超集，其中一些值是特殊的。这些特殊值被称为[*哨兵值*](https://en.wikipedia.org/wiki/Sentinel_value)。它们存在于[*波段内*](https://en.wikipedia.org/wiki/In-band_signaling)（想象在同一个通道内）并且是正常值的兄弟。

例如，考虑以下文本流接口：

```ts
interface LineStream {
 getNextLine(): string;
}

```

目前，`.getNextLine()`只处理文本行，但不处理文件结束（EOF）。我们如何添加对 EOF 的支持？

可能性包括：

+   需要在调用`.getNextLine()`之前调用的附加方法`.isEof()`。

+   当`.getNextLine()`遇到 EOF 时，它会抛出异常。

+   EOF 的哨兵值。

下两个小节描述了两种我们可以引入哨兵值的方法。

#### 17.1.1 向类型中添加`null`或`undefined`

当使用严格的 TypeScript 时，没有简单的对象类型（通过接口、对象模式、类等定义）包括`null`。这使得它成为一个很好的哨兵值，我们可以通过联合类型将其添加到基础类型`string`中：

```ts
type StreamValue = null | string;

interface LineStream {
 getNextLine(): StreamValue;
}

```

现在，无论何时我们使用`.getNextLine()`返回的值，TypeScript 都会强制我们考虑两种可能性：字符串和`null`——例如：

```ts
function countComments(ls: LineStream) {
 let commentCount = 0;
 while (true) {
 const line = ls.getNextLine();
 // @ts-expect-error: 'line' is possibly 'null'.
 if (line.startsWith('#')) { // (A)
 commentCount++;
 }
 if (line === null) break;
 }
 return commentCount;
}

```

在行 A 中，我们不能使用字符串方法`.startsWith()`，因为`line`可能是`null`。我们可以这样修复：

```ts
function countComments(ls: LineStream) {
 let commentCount = 0;
 while (true) {
 const line = ls.getNextLine();
 if (line === null) break;
 if (line.startsWith('#')) { // (A)
 commentCount++;
 }
 }
 return commentCount;
}

```

现在，当执行到达行 A 时，我们可以确信`line`不是`null`。

#### 17.1.2 向类型中添加符号

我们也可以使用除了`null`之外的其他值作为哨兵。符号最适合这项任务，因为每个符号都有一个唯一的身份，没有其他值可以与之混淆。

这就是如何使用符号来表示 EOF：

```ts
const EOF = Symbol('EOF');
type StreamValue = typeof EOF | string;

```

为什么我们需要`typeof`而不能直接使用`EOF`？那是因为`EOF`是一个值，不是一个类型。类型运算符`typeof`将`EOF`转换为类型。

##### 17.1.2.1 示例：符号作为错误值

以下函数`parseNumber()`使用符号`couldNotParseNumber`作为错误值：

```ts
const couldNotParseNumber = Symbol('couldNotParseNumber');

function parseNumber(str: string)
 : number | typeof couldNotParseNumber
{
 const result = Number(str);
 if (!Number.isNaN(result)) {
 return result;
 } else {
 return couldNotParseNumber;
 }
}

assert.equal(
 parseNumber('123'), 123
);
assert.equal(
 parseNumber('hello'), couldNotParseNumber
);

```

### 17.2 超出范围的特殊值添加

如果方法可能返回 *任何* 值，我们该怎么办？我们如何确保基本值和元值不会混淆？这是一个可能会发生这种情况的例子：

```ts
interface ValueStream<T> {
 getNextValue(): T;
}

```

无论我们为 `EOF` 选择什么值，都存在有人创建一个 `ValueStream<typeof EOF>` 并将此值添加到流中的风险。

解决方案是将普通值和特殊值分开，这样它们就不会混淆。特殊值独立存在被称为 [*超出范围*](https://en.wikipedia.org/wiki/Out-of-band_data)（想象不同的频道）。

#### 17.2.1 判别联合类型

*判别联合类型* 是几种具有至少一个共同属性（所谓的 *判别属性*）的对象类型的联合。判别属性必须对每种对象类型具有不同的值——我们可以将其视为对象类型的 ID。

##### 17.2.1.1 示例：`ValueStreamValue`

在以下示例中，`ValueStreamValue<T>` 是一个判别联合类型，其判别属性是 `.type`。

```ts
interface NormalValue<T> {
 type: 'normal'; // string literal type
 data: T;
}
interface Eof {
 type: 'eof'; // string literal type
}
type ValueStreamValue<T> = Eof | NormalValue<T>;

interface ValueStream<T> {
 getNextValue(): ValueStreamValue<T>;
}

```

```ts
function countValues<T>(vs: ValueStream<T>, data: T) {
 let valueCount = 0;
 while (true) {
 const value = vs.getNextValue(); // (A)
 assertType<Eof | NormalValue<T>>(value);

 if (value.type === 'eof') break;
 assertType<NormalValue<T>>(value); // (B)

 if (value.data === data) { // (C)
 valueCount++;
 }
 }
 return valueCount;
}

```

初始时，`value` 的类型是 `ValueStreamValue<T>`（行 A）。然后我们排除了用于 `.type` 的判别值 `'eof'`，其类型被缩小到 `NormalValue<T>`（行 B）。这就是为什么我们可以在行 C 访问属性 `.data` 的原因。

##### 17.2.1.2 示例：`IteratorResult`

当决定如何实现 [迭代器](https://exploringjs.com/js/book/ch_sync-iteration.html) 时，TC39 不想使用一个固定的哨兵值。否则，如果该值出现在可迭代对象中，代码就会出错。一个解决方案是在开始迭代时选择一个哨兵值。TC39 选择了具有公共属性 `.done` 的判别联合类型：

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

#### 17.2.2 其他类型的联合类型

只要我们有区分联合类型成员类型的手段，其他类型的联合类型可以像判别联合类型一样方便。

一种可能性是通过独特的属性来区分成员类型：

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

另一种可能性是通过 `typeof` 和/或实例检查来区分成员类型：

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
