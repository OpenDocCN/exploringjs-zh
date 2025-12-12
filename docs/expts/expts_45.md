# 37 使用元组类型进行计算

> 原文：[`exploringjs.com/ts/book/ch_computing-with-tuple-types.html`](https://exploringjs.com/ts/book/ch_computing-with-tuple-types.html)

(Ad, 请勿拦截。)

1.  37.1 元组类型的语法

    1.  37.1.1 基本语法

    1.  37.1.2 可变元组元素

    1.  37.1.3 带标签的元组元素

1.  37.2 元组类型

    1.  37.2.1 元组与 `--noUncheckedIndexedAccess`

    1.  37.2.2 强制数组字面量被推断为元组

    1.  37.2.3 使用 `readonly` 接受 const 元组

    1.  37.2.4 强制固定数组长度

1.  37.3 元组类型的键

    1.  37.3.1 提取元组的索引键（字符串）

    1.  37.3.2 提取元组的索引（数字）

1.  37.4 通过映射类型映射元组

    1.  37.4.1 映射类型如何处理元组类型的键

    1.  37.4.2 映射保留元组元素的标签

    1.  37.4.3 元组和具有键重映射的映射类型 (`as`)

    1.  37.4.4 示例：为 `Promise.all()` 进行类型定义

1.  37.5 从元组中提取联合类型

    1.  [37.5.1 将索引访问运算符 `T[K]` 应用到元组上](#from-tuple-to-union)

    1.  37.5.2 从元组元组中提取联合类型

    1.  37.5.3 从对象元组中提取联合类型

1.  37.6 使用元组类型进行计算

    1.  37.6.1 提取元组的部分

    1.  37.6.2 使用元组对作为查找表

    1.  37.6.3 连接元组

    1.  37.6.4 对元组进行递归

1.  37.7 实际世界示例

    1.  37.7.1 保持参数名称的偏应用

    1.  37.7.2 为 `zip()` 函数进行类型定义

    1.  37.7.3 为 `zipObj()` 函数进行类型定义

    1.  37.7.4 `util.promisify()`：将基于回调的函数转换为基于 Promise 的函数

1.  37.8 使用元组进行计算的限制

1.  37.9 本章来源

JavaScript 的数组非常灵活，TypeScript 提供了两种不同的类型来处理它们：

+   用于任意长度值序列的数组类型，其中所有值都具有相同的类型 – 例如：`Array<string>`

+   用于值类型固定长度序列的元组类型，其中每个值可能具有不同的类型 – 例如：`[number, string, boolean]`

在本章中，我们探讨后者——特别是如何在类型级别上使用元组进行计算。

### 37.1 元组类型的语法

#### 37.1.1 基本语法

元组类型具有以下语法：

```ts
[ Required, Optional?, ...RestElement[] ]

```

+   首先，零个或多个必需元素。

+   然后，零个或多个可选元素。

+   最后，可选地，一个单独的剩余元素。

例子：

```ts
type T = [string, boolean?, ...number[]];
const v1: T = ['a', true, 1, 2, 3];
const v2: T = ['a', true];
const v3: T = ['a'];
// @ts-expect-error: Type '[]' is not assignable to type 'T'.
const v4: T = [];

```

**还有一条额外的规则：必需元素可以出现在剩余元素之后——但前提是它们之前没有可选元素**：

```ts
type T1 = [number, ...boolean[], string]; // OK
type T2 = [...boolean[], string]; // OK

// @ts-expect-error: A required element cannot follow
// an optional element.
type T3 = [number?, ...boolean[], string];

```

##### 37.1.1.1 可选元素只能省略在末尾

```ts
type T = [string, boolean?, ...number[]];

const v1: T = ['a', false, 1, 2, 3]; // OK
const v2: T = ['a']; // OK

// @ts-expect-error: Type 'number' is not assignable to
// type 'boolean'.
const v3: T = ['a', 1, 2, 3];

```

如果编译器选项 `exactOptionalPropertyTypes`（ch_tsconfig-json.html#exactOptionalPropertyTypes）处于活动状态，我们甚至不能做以下操作：

```ts
// @ts-expect-error: Type '[string, undefined, number, number, number]' is
// not assignable to type 'T'. Type at position 1 in source is not
// compatible with type at position 1 in target. Type 'undefined' is not
// assignable to type 'boolean'.
const v4: T = ['a', undefined, 1, 2, 3];

```

注意，这与 JavaScript 处理参数和解构的方式相似——例如：

```ts
function f(x, y=3, ...z) {
 return {x,y,z};
}

```

如果我们想启用中间省略元素，我们可以使用一个联合：

```ts
// The `boolean` element can be omitted:
type T =
 | [string, boolean, ...number[]]
 | [string, ...number[]]
;
const v1: T = ['a', false, 1, 2, 3]; // OK
const v2: T = ['a', 1, 2, 3]; // OK

```

如果有第二个参数，它将被分配给 `y` 并不会成为 `z` 的一个元素。

#### 37.1.2 可变元组元素

*可变*意味着“具有可变（不是固定）的元数”。元组的元数是其长度。

*可变元素*（或*扩展元素*）在类型级别上允许扩展到元组：

```ts
type Tuple1 = ['a', 'b'];
type Tuple2 = [1, 2];
type _ = Assert<Equal<
 [true, ...Tuple1, ...Tuple2, false], // type expression
 [ true, 'a', 'b', 1, 2, false ] // result
>>;

```

将其与 JavaScript 中的扩展进行比较：

```ts
const tuple1 = ['a', 'b'];
const tuple2 = [1, 2];
assert.deepEqual(
 [true, ...tuple1, ...tuple2, false], // expression
 [ true, 'a', 'b', 1, 2, false ] // result
);

```

扩展的类型通常是类型变量，并且必须可分配给 `readonly any[]` ——即，它必须是一个数组或一个元组。它可以具有任何长度——因此术语“可变”。它通常是一个类型变量，必须可分配给 `readonly any[]` ——即，它必须是一个数组或一个元组。它可以具有任何长度——因此术语“可变”。它通常是一个类型变量，必须可分配给 `readonly any[]` ——即，它必须是一个数组或一个元组。它可以具有任何长度——因此术语“可变”：

> 直观地讲，可变元素 `...T` 是一个占位符，它通过泛型类型实例化被替换为一个或多个元素。

##### 37.1.2.1 实例化泛型元组的规范化

扩展的结果被调整，以便始终符合本节开头描述的形状。为了探索它是如何工作的，我们将使用实用类型 `Spread1` 和 `Spread2`：

```ts
type Spread1<T extends unknown[]> = [...T];
type Spread2<T1 extends unknown[], T2 extends unknown[]> =
 [...T1, ...T2]
;

type _ = [
 // A tuple with only a spread Array becomes an Array:
 Assert<Equal<
 Spread1<Array<string>>,
 string[]
 >>,

 // If an Array is spread at the end, it becomes a rest element:
 Assert<Equal<
 Spread2<['a', 'b'], Array<number>>,
 ['a', 'b', ...number[]]
 >>,

 // If two Arrays are spread, they are merged so that there
 // is at most one rest element:
 Assert<Equal<
 Spread2<Array<string>, Array<number>>,
 [...(string | number)[]]
 >>,

 // Optional elements after an Array are merged into it:
 Assert<Equal<
 Spread2<Array<string>, [number?, boolean?]>,
 (string | number | boolean | undefined)[]
 >>,

 // Optional elements `T` before required ones become `undefined|T`:
 Assert<Equal<
 Spread2<[string?], [number]>,
 [string | undefined, number]
 >>,

 // Required elements between Arrays are also merged:
 Assert<Equal<
 Spread2<[boolean, ...number[]], [string, ...bigint[]]>,
 [boolean, ...(string | number | bigint)[]]
 >>,
];

```

注意，我们只能扩展一个类型 `T`，如果它通过 `extends` 限制为一个数组类型：

```ts
type Spread1a<T extends unknown[]> = [...T]; // OK
// @ts-expect-error: A rest element type must be an array type.
type Spread1b<T> = [...T];

```

#### 37.1.3 带标签的元组元素

我们也可以为元组元素指定标签：

```ts
type Interval = [start: number, end: number];

```

如果有一个元素被标记，则所有元素都必须被标记。对于可选元素，语法会随着标签而改变——问号（`?`）添加到标签中，而不是类型（TypeScript 在编辑时会告诉你是否出错）：

```ts
type Tuple1 = [string, boolean?, ...number[]];
type Tuple2 = [requ: string, opt?: boolean, ...rest: number[]];

```

标签有什么作用？不多：它们有助于自动完成，并且在某些类型操作中被保留，但在类型系统中没有其他作用：

+   我们无法从它们中推导出任何东西（例如，从普通参数推导出选项对象）。

+   它们不影响类型兼容性等。

因此：如果名称很重要，你应该使用对象类型。

##### 37.1.3.1 提取的函数参数是带标签的

如果我们提取函数参数，我们得到带标签的元组元素：

```ts
type _1 = Assert<Equal<
 Parameters<(sym: symbol, bool: boolean) => void>,
 [sym: symbol, bool: boolean]
>>;

```

注意，没有方法可以检查实际的元组元素标签——这些检查也会成功：

```ts
// Different labels
type _2 = Assert<Equal<
 Parameters<(sym: symbol, bool: boolean) => void>,
 [HELLO: symbol, EVERYONE: boolean]
>>;

// No labels
type _3 = Assert<Equal<
 Parameters<(sym: symbol, bool: boolean) => void>,
 [symbol, boolean]
>>;

```

##### 37.1.3.2 使用场景：重载

如果一个剩余参数具有元组类型，TypeScript 会使用标签作为函数参数：

```ts
function f1(...args: [str: string, num: number]) {}
 // function f1(str: string, num: number): void
function f2(...args: [string, number]) {}
 // function f2(args_0: string, args_1: number): void

```

多亏了标签，元组成为了一个更好的重载替代方案，因为自动完成可以显示参数名称：

```ts
// Overloading with tuples
function f(
 ...args:
 | [str: string, num: number]
 | [num: number]
 | [bool: boolean]
): void {
 // ···
}

```

```ts
// Traditional overloading
function f(str: string, num: number): void;
function f(num: number): void;
function f(bool: boolean): void;
function f(arg0: string | number | boolean, num?: number): void {
 // ···
}

```

但有一个限制，即元组不能影响返回类型。

##### 37.1.3.3 使用场景：在转换函数时保留参数名称

当我们处理部分应用时（later in this chapter），将展示它是如何工作的。

### 37.2 数组类型的类型

#### 37.2.1 元组和 `--noUncheckedIndexedAccess`

如果我们切换到 `tsconfig.json` 选项 `noUncheckedIndexedAccess`（[ch_tsconfig-json.html#noUncheckedIndexedAccess]），那么 TypeScript 会更诚实地表达它对一个可索引类型的了解。

使用数组时，TypeScript 在编译时永远不知道哪些索引位置有元素——这就是为什么索引读取总是可能返回 `undefined` 的原因：

```ts
const arr: Array<string> = ['a', 'b', 'c'];
const arrayElement = arr[1];
assertType<string | undefined>(arrayElement);

```

使用元组时，TypeScript 知道整个形状，并且可以为索引读取提供更好的类型：

```ts
const tuple: [string, string, string] = ['a', 'b', 'c'];
const tupleElement = tuple[1];
assertType<string>(tupleElement);

```

#### 37.2.2 强制将数组字面量推断为元组

默认情况下，JavaScript 数组字面量具有数组类型：

```ts
// Array
const value1 = ['a', 1];
assertType<
 (string | number)[]
>(value1);

```

改变这一点最常见的方式是通过 `as const` 注解：

```ts
// Tuple
const value2 = ['a', 1] as const;
assertType<
 readonly ['a', 1]
>(value2);

```

但我们也可以使用 `satisfies`：

```ts
// Non-empty tuple
const value3 = ['a', 1] satisfies [unknown, ...unknown[]];
assertType<
 [string, number]
>(value3);

// Tuple (possibly empty)
const value4 = ['a', 1] satisfies [unknown?, ...unknown[]];
assertType<
 [string, number]
>(value4);

```

注意，`as const` 还将元素类型缩小到 `'a'` 和 `1`。使用 `satisfies` 时，它们是 `string` 和 `number` – 除非我们为元素使用 `as const`：

```ts
// Tuple
const value5 = [
 'a' as const, 1 as const
] satisfies [unknown?, ...unknown[]];
assertType<
 ['a', 1]
>(value5);

```

如果我们在剩余元素之前省略元组元素（在末尾），我们就会回到数组类型：

```ts
// Array
const value6 = ['a', 1] satisfies [...unknown[]];
assertType<
 (string | number)[]
>(value6);

```

我们还可以为元组使用另一种类型：

```ts
// Tuple
const value7 = ['a', 1] satisfies unknown[] | [];
assertType<
 [string, number]
>(value7);

```

#### 37.2.3 使用 `readonly` 接受 const 元组

如果类型 `T` 被约束为普通数组类型，那么它不匹配 `as const` 字面量的类型：

```ts
type Tuple<T extends Array<unknown>> = T;
const arr = ['a', 'b'] as const;
// @ts-expect-error: Type 'readonly ["a", "b"]' does not satisfy
// the constraint 'unknown[]'.
type _ = Tuple<typeof arr>;

```

我们可以通过切换到 `ReadonlyArray` 来改变这一点：

```ts
type Tuple<T extends ReadonlyArray<unknown>> = T;
const arr = ['a', 'b'] as const;
type Result = Tuple<typeof arr>;
type _ = Assert<Equal<
 Result, readonly ['a', 'b']
>>;

```

以下两种表示法是等价的：

```ts
ReadonlyArray<unknown>
readonly unknown[]

```

在本章中，我并不总是将数组类型设置为只读，因为这会增加视觉上的杂乱。

#### 37.2.4 强制固定数组长度

我们可以使用以下技巧来强制数组字面量的固定长度：

```ts
function join3<T extends string[] & {length: 3}>(...strs: T) {
 return strs.join('');
}
join3('a', 'b', 'c'); // OK

// @ts-expect-error: Argument of type '["a", "b"]' is not assignable
// to parameter of type 'string[] & { length: 3; }'.
join3('a', 'b');

```

但有一个限制，即如果 `strs` 来自一个类型为数组的变量，则此技术不起作用：

```ts
const arr = ['a', 'b', 'c'];
// @ts-expect-error: Argument of type 'string[]' is not assignable
// to parameter of type 'string[] & { length: 3; }'.
join3(...arr);

```

相反，一个元组可以这样工作：

```ts
const tuple = ['a', 'b', 'c'] as const;
join3(...tuple);

```

### 37.3 元组类型的键

数组的键看起来是这样的（注意第一行中的 `Includes`）：

```ts
type _ = Assert<Includes<
 keyof Array<string>,
 number | 'length' | 'push' | 'join' // ...
>>;

```

我们可以看到数组索引的键（`number`）、`.length` 和数组方法。

元组的键类似，但除了索引的广泛类型 `number` 之外，它们还为每个索引都有一个字符串化的数字：

```ts
type _ = Assert<Includes<
 keyof ['a', 'b'],
 number | '0' | '1' | 'length' | 'push' | 'join'  // ...
>>;

```

为什么使用字符串字面量类型而不是数字字面量类型？后者在与 `number` 的联合中会消失：

```ts
type _ = Assert<Includes<
 number | 0 | 1,
 number
>>;

```

注意，ECMAScript 规范也使用字符串键来表示数组元素（[更多信息](https://exploringjs.com/js/book/ch_arrays.html#arrays-are-actually-dictionaries)）：

```ts
> Object.keys(['a', 'b'])
[ '0', '1' ]

```

#### 37.3.1 提取元组的索引键（字符串）

此实用类型返回元组 `T` 的所有字符串键，这些键是索引：

```ts
type TupleIndexKeys<T extends ReadonlyArray<unknown>> =
 (keyof T) & `${number}`
;

```

```ts
type _ = Assert<Equal<
 TupleIndexKeys<['a', 'b']>,
 '0' | '1'
>>;

```

我们使用 `&` 来创建 `T` 的键和模板字面量类型 `` `${number}` `` 之间的交集类型——这是所有被转换为字符串的数字字符串的类型（参见“将原始类型插入模板字面量”（§38.2.5））。

#### 37.3.2 提取元组的索引（数字）

获取元组的数字索引（数字，而不是字符串化的数字）需要更多的工作：

```ts
type TupleIndices<T extends ReadonlyArray<unknown>> =
 StrToNum<keyof T>
;

```

```ts
type _ = Assert<Equal<
 TupleIndices<['a', 'b']>,
 0 | 1
>>;

```

`TupleIndices` 使用以下辅助类型，它提取带有数字的字符串字面量类型并将它们转换为数字。

```ts
type StrToNum<T> =
 T extends `${infer N extends number}` ? N : never // (A)
;

```

```ts
type _ = Assert<Equal<
 StrToNum<number | '0' | '1' | 'length' | 'push' | 'join'>,
 0 | 1
>>;

```

在行 A 中，`StrToNum` 使用模板字面量类型加上 `infer` 来解析字符串字面量类型内的数字。如果没有数字，它返回 `never`。由于行 A 中的条件类型是分配的，我们可以用它来过滤联合类型（如末尾所示）。

### 37.4 通过映射类型映射元组

映射类型具有以下语法：

```ts
type MapOverType<Type> = {
 [Key in keyof Type]: Promise<Type[Key]>
};

```

#### 37.4.1 映射类型如何处理元组类型的键

回想一下，将 `keyof` 应用到元组上会产生各种值：方法名称、字符串化的索引等。

在其基本形式中，映射类型以两种方式帮助我们处理元组：

+   首先，它过滤键，并且只遍历字符串索引键（`'0'`、`'1'`等）。

+   其次，它返回一个元组（而不是对象字面量类型）。

以下示例演示了这两种现象。`KeyToKey<T>` 返回一个元组，其元素是元组 `T` 的字符串索引键：

```ts
type KeyToKey<T> = {
 [K in keyof T]: K
};
type _ = Assert<Equal<
 KeyToKey<['a', 'b']>,
 // Result is a tuple
 ['0', '1']
>>;

```

#### 37.4.2 映射保留元组元素的标签

映射保留元组元素的标签：

```ts
type WrapValues<T> = {
 [Key in keyof T]: Promise<T[Key]>
};
type _ = Assert<Equal<
 WrapValues<[a: number, b: number]>,
 [a: Promise<number>, b: Promise<number>]
>>;

```

#### 37.4.3 元组和带有键重映射（`as`）的映射类型

如果我们在映射类型上使用键重映射（`as`），则结果将不再是元组，并且将考虑元组的所有键（而不是仅其索引）：

```ts
type KeyAsKeyToKey<T> = {
 [K in keyof T as K]: K
};
type _ = Assert<Equal<
 // Use Pick<> because result of KeyAsKeyToKey<> is large
 Pick<
 KeyAsKeyToKey<['a', 'b']>,
 '0' | '1' | 'length' | 'push' | 'join'
 >,
 // Result is an object, not a tuple
 {
 length: 'length';
 push: 'push';
 join: 'join';
 0: '0';
 1: '1';
 }
>>;

```

如果我们想坚持使用元组索引，我们必须过滤 `keyof` 的结果。为此，我们可以使用之前定义的实用类型 `TupleIndexKeys`：

```ts
type StringTupleToObject<T extends ReadonlyArray<string>> = {
 [K in TupleIndexKeys<T> as T[K]]: K
};
type _ = Assert<Equal<
 StringTupleToObject<['a', 'b']>,
 {
 a: '0',
 b: '1',
 }
>>;

```

注意，`TupleIndices` 返回字符串字面量类型——这解释了属性值。如果我们更喜欢数字字面量类型，我们可以使用之前定义的实用类型 `TupleIndices`：

```ts
type StringTupleToObject<T extends ReadonlyArray<string>> = {
 [K in TupleIndices<T> as T[K]]: K
};
type _ = Assert<Equal<
 StringTupleToObject<['a', 'b']>,
 {
 a: 0,
 b: 1,
 }
>>;

```

#### 37.4.4 示例：类型化 `Promise.all()`

这就是 `Promise.all()` 的类型看起来像什么（我稍微编辑了[实际代码](https://github.com/microsoft/TypeScript/blob/main/src/lib/es2015.promise.d.ts)）：

我们将使用以下辅助类型，它解包元组中的 Promise：

```ts
type AwaitedTuple<T extends ReadonlyArray<unknown>> = {
 -readonly [K in keyof T]: Awaited<T[K]> // (A)
}
type _ = Assert<Equal<
 AwaitedTuple<readonly [Promise<number>, Promise<string>]>,
 [number, string]
>>;

```

注意事项：

+   线 A 中的`-readonly`移除了每个元组元素以及整个元组的该修饰符。

+   线 A 中使用的内置实用类型`Awaited`（用于行 A）像`await`一样工作，并且（大致上）解包 Promise。

使用这个辅助类型，我们的`Promise.all()`版本很容易进行类型化：

```ts
function promiseAll<
 T extends ReadonlyArray<unknown> | [] // (A)
>(values: T): Promise<AwaitedTuple<T>> {
 // ···
}
const result = promiseAll(
 [Promise.resolve(123), Promise.resolve('abc')]
);
assertType<Promise<[number, string]>>(result);

```

线 A 中`extends`后面的约束实现了两个目的：

+   `readonly`表示除了可变`values`之外，也接受只读`values`。

+   `unknown[] | []` 表示数组字面量被解释为元组。

### 37.5 从元组中提取联合类型

#### [37.5.1 将索引访问运算符`T[K]`应用于元组](#from-tuple-to-union)

如果我们将[索引访问运算符`T[K]`](ch_computing-with-types-overview.html#indexed-access-types)应用于元组，我们将得到元组元素作为联合类型：

```ts
type UnionOf<Tup extends ReadonlyArray<unknown>> = Tup[number];

const flowers = ['rose', 'sunflower', 'lavender'] as const;
type _ = Assert<Equal<
 UnionOf<typeof flowers>,
 'rose' | 'sunflower' | 'lavender'
>>;

```

#### 37.5.2 从元组元组中提取联合类型

有时，将数据编码为元组的集合是有意义的——例如，当我们想要通过其任何元素查找元组，而性能不是那么重要时。相比之下，Map 只支持通过键进行查找。

对于 Map，计算键和值很容易——我们可以使用这些值来约束查找数据时的值。我们能否对元组元组做同样的事情？我们可以，如果我们使用索引访问运算符`T[K]`两次：

```ts
const englishSpanishGerman = [
 ['yes', 'sí', 'ja'],
 ['no', 'no', 'nein'],
 ['maybe', 'tal vez', 'vielleicht'],
] as const;

type English = (typeof englishSpanishGerman)[number][0];
type _1 = Assert<Equal<
 English, 'yes' | 'no' | 'maybe'
>>;

type Spanish = (typeof englishSpanishGerman)[number][1];
type _2 = Assert<Equal<
 Spanish, 'sí' | 'no' | 'tal vez'
>>;

```

#### 37.5.3 从对象元组中提取联合类型

同样的方法也适用于对象元组：

```ts
const listCounterStyles = [
 { name: 'upperRoman', regExp: /^[IVXLCDM]+$/ },
 { name: 'lowerRoman', regExp: /^[ivxlcdm]+$/ },
 { name: 'upperLatin', regExp: /^[A-Z]$/ },
 { name: 'lowerLatin', regExp: /^[a-z]$/ },
 { name: 'decimal',    regExp: /^[0-9]+$/ },
] as const satisfies Array<{regExp: RegExp, name: string}>;

type CounterNames = (typeof listCounterStyles)[number]['name'];
type _ = Assert<Equal<
 CounterNames,
 | 'upperRoman' | 'lowerRoman'
 | 'upperLatin' | 'lowerLatin'
 | 'decimal'
>>;

```

### 37.6 计算元组类型

+   在本节中，我们将通过一些小例子来探索使用元组类型的计算。

+   在下一节中，我们将探讨这类计算的实际情况。

#### 37.6.1 提取元组的部分

要提取元组的部分，我们使用`infer`。

##### 37.6.1.1 提取元组的第一个元素

我们通过使用通配符类型`unknown`作为匹配任何内容的占位符来推断第一个元素，并忽略所有其他元素。

```ts
type First<T extends Array<unknown>> =
T extends [infer F, ...unknown[]]
 ? F
 : never
;
type _ = Assert<Equal<
 First<['a', 'b', 'c']>,
 'a'
>>;

```

##### 37.6.1.2 提取元组的最后一个元素

我们用来提取第一个元素（在先前的例子中）的方法也适用于提取最后一个元素：

```ts
type Last<T extends Array<unknown>> =
T extends [...unknown[], infer L]
 ? L
 : never
;
type _ = Assert<Equal<
 Last<['a', 'b', 'c']>,
 'c'
>>;

```

##### 37.6.1.3 提取元组的*其余部分*（第一个元素之后的元素）

要提取元组的*其余部分*（第一个元素之后的元素），我们使用通配符类型`unknown`来表示第一个元素，并推断出它之后的内容：

```ts
type Rest<T extends Array<unknown>> =
T extends [unknown, ...infer R]
 ? R
 : never
;
type _ = Assert<Equal<
 Rest<['a', 'b', 'c']>,
 ['b', 'c']
>>;

```

#### 37.6.2 使用成对元组作为查找表

对于许多用途，对象字面量类型作为查找表非常方便：在类型级别，查找仅在键是字符串、数字或符号时才有效。此外，TypeScript 不区分字符串和数字。这反映了 JavaScript 的工作方式，并防止我们区分数字 `1` 和字符串 `'1'`：

```ts
type LookupTable = {
 [1]: 'a',
};
type _ = [
 Assert<Equal<
 LookupTable[1], 'a'
 >>,
 Assert<Equal<
 LookupTable['1'], 'a'
 >>,
];

```

作为替代方案，我们可以使用对（包含两个元素的元组）作为查找表：

```ts
type LookupTable = [
 [undefined, 'undefined'],
 [null, 'null'],
 [boolean, 'boolean'],
 [number, 'number'],
 [bigint, 'bigint'],
 [string, 'string'],
];
type R = Assert<Equal<
 Lookup<LookupTable, string>, 'string'
>>;

```

这些类型实现了查找功能：

```ts
type LookupOne<Pair extends readonly [unknown, unknown], Key> =
 Pair extends [Key, infer Value] ? Value : never;
type Lookup<Table extends ReadonlyArray<readonly [unknown, unknown]>, Key> =
 LookupOne<Table[number], Key>;

```

这是如何工作的？步骤 1：将元组的对转换为对（通过[索引访问类型 (`T[K]`)](ch_computing-with-types-overview.html#indexed-access-types)）。

```ts
type _1 = Assert<Equal<
 LookupTable[number],
 | [undefined, 'undefined']
 | [null, 'null']
 | [boolean, 'boolean']
 | [number, 'number']
 | [bigint, 'bigint']
 | [string, 'string']
>>;

```

第 2 步：将 `LookupOne` 应用到每一对上。如果我们将这个泛型类型应用到联合上，这会自动发生，因为它的条件类型是分配的：

```ts
type _2 = [
 Assert<Equal<
 LookupOne<[undefined, 'undefined'], string>,
 never
 >>,
 Assert<Equal<
 LookupOne<[string, 'string'], string>,
 'string'
 >>,
];

```

第 3 步：由于 `never` 是空集，所以在评估 `LookupOne` 分配应用的中间结果后，我们得到最终结果 `'string'`：

```ts
type _3 = Assert<Equal<
 never | never | never | never | never | 'string',
 'string' // final result
>>;

```

#### 37.6.3 元组的连接

要连接两个元组 `T1` 和 `T2`，我们需要将它们都展开：

```ts
type Concat<T1 extends Array<unknown>, T2 extends Array<unknown>> =
 [...T1, ...T2]
;
type _ = Assert<Equal<
 Concat<['a', 'b'], ['c', 'd']>,
 ['a', 'b', 'c', 'd']
>>;

```

#### 37.6.4 元组的递归

为了探索元组的递归，让我们通过递归包装元组元素来实现（我们之前使用的是映射类型）：

TypeScript 中的元组递归

```ts
type WrapValues<Tup> =
 Tup extends [infer First, ...infer Rest] // (A)
 ? [Promise<First>, ...WrapValues<Rest>] // (B)
 : [] // (C)
;
type _ = Assert<Equal<
 WrapValues<['a', 'b', 'c']>,
 [Promise<'a'>, Promise<'b'>, Promise<'c'>]
>>;

```

我们使用一种受函数式编程语言如何递归列表启发的技术：

+   行 A：我们检查是否可以将 `Tup` 分割为第一个元素 `First` 和剩余元素 `Rest`。

+   行 B：如果是的话，`Tup` 至少有一个元素。我们返回一个元组，其第一个元素是包装的 `First`，其余元素通过自我递归调用计算得出。

+   行 C：如果没有，则 `Tup` 为空。我们返回一个空元组。

在函数式编程中，`First` 通常被称为 `Head`，而 `Rest` 通常被称为 `Tail`。

##### 37.6.4.1 展平元组的元组

让我们使用递归将元组的元组展平：

```ts
type Flatten<Tups extends Array<Array<unknown>>> =
 Tups extends [
 infer Tup extends Array<unknown>, // (A)
 ...infer Rest extends Array<Array<unknown>> // (B)
 ]
 ? [...Tup, ...Flatten<Rest>]
 : []
;
type _ = Assert<Equal<
 Flatten<[['a', 'b'], ['c', 'd'], ['e']]>,
 ['a', 'b', 'c', 'd', 'e']
>>;

```

在这种情况下，推断出的类型 `Tup` 和 `Rest` 更复杂——这就是为什么 TypeScript 如果我们不使用 `extends`（行 A，行 B）来约束它们会报错。

##### 37.6.4.2 过滤元组

以下代码使用递归从元组中过滤掉空字符串：

```ts
type RemoveEmptyStrings<T extends Array<string>> =
 T extends [
 infer First extends string,
 ...infer Rest extends Array<string>
 ]
 ? First extends ''
 ? RemoveEmptyStrings<Rest>
 : [First, ...RemoveEmptyStrings<Rest>]
 : []
;
type _ = Assert<Equal<
 RemoveEmptyStrings<['', 'a', '', 'b', '']>,
 ['a', 'b']
>>;

```

注意，我们必须使用递归进行过滤。使用映射类型和通过 `as` 进行键重映射不会工作有两个原因：

+   由于 `as`，这种类型构建的是一个对象类型，而不是元组类型。

+   映射保留索引，因此删除属性会留下空隙。

```ts
type RemoveEmptyStrings<T extends Array<string>> = {
 [K in keyof T as (T[K] extends '' ? never : K)]: T[K]
};
type Filtered = RemoveEmptyStrings<['', 'a', '', 'b', '']>
 // type Filtered = {
 //   [x: number]: "" | "a" | "b";
 //   1: "a";
 //   3: "b";
 //   length: 5;
 //   toString: () => string;
 //   ...
 // }

```

##### 37.6.4.3 创建具有给定长度的元组

如果我们想要创建一个具有给定长度 `Len` 的元组，我们会面临一个挑战：我们如何知道何时停止？我们不能递减 `Len`，我们只能检查它是否等于一个给定的值（行 A）：

```ts
type Repeat<
 Len extends number, Value,
 Acc extends Array<unknown> = []
> = 
 Acc['length'] extends Len // (A)
 ? Acc // (B)
 : Repeat<Len, Value, [...Acc, Value]> // (C)
;

type _ = [
 Assert<Equal<
 Repeat<3, '*'>,
 ['*', '*', '*']
 >>,
 Assert<Equal<
 Repeat<3, string>,
 [string, string, string]
 >>,
 Assert<Equal<
 Repeat<3, unknown>,
 [unknown, unknown, unknown]
 >>,
];

```

这段代码是如何工作的？我们使用另一种函数式编程技术，并引入一个内部累加器参数 `Acc`：

+   当递归仍在进行时，我们在 `Acc`（行 C）中组装最终结果。

+   当 `Acc` 的长度等于 `Len`（行 A）时，我们完成并可以返回 `Acc`（行 B）。

##### 37.6.4.4 计算数字范围

我们可以使用相同的技巧来计算一系列数字。但这次，我们将累加器的当前长度追加到累加器中：

```ts
type NumRange<Upper extends number, Acc extends number[] = []> =
 Upper extends Acc['length']
 ? Acc
 : NumRange<Upper, [...Acc, Acc['length']]>
;
type _ = Assert<Equal<
 NumRange<3>,
 [0, 1, 2]
>>;

```

##### 37.6.4.5 丢弃初始元素

这是实现一个实用类型的一种方法，该类型可以移除 `Tuple` 的前 `Num` 个元素：

```ts
type Drop<
 Tuple extends Array<unknown>,
 Num extends number,
 Counter extends Array<boolean> = []
> =
 Counter['length'] extends Num
 ? Tuple
 : Tuple extends [unknown, ...infer Rest extends Array<unknown>]
 ? Drop<Rest, Num, [true, ...Counter]>
 : Tuple
;
type _ = Assert<Equal<
 Drop<['a', 'b', 'c'], 2>,
 ['c']
>>;

```

这次，我们使用累加器变量 `Counter` 来计数，直到 `Counter['length']` 等于 `Num`。

我们还可以使用推理（[由 Heribert Schütz 提出](https://mastodon.social/@hcschuetz/113906317654567023)）：

```ts
type Drop<
 Tuple extends Array<unknown>,
 Num extends number
> =
 Tuple extends [...Repeat<Num, unknown>, ...infer Rest]
 ? Rest
 : never
;

```

我们使用 实用类型 `Repeat` 来计算一个元组，其中每个元素都是匹配任何类型的通配符类型 `unknown`。然后我们将 `Tuple` 与以这些元素开始的元组模式进行匹配。剩余的元素是我们正在寻找的结果，我们通过 `infer` 提取它。

### 37.7 现实世界的例子

#### 37.7.1 保留参数名称的偏应用

让我们实现函数 `applyPartial(func, args)` 以部分应用函数 `func`。它的工作方式与函数方法 `.bind()` 类似：

```ts
function applyPartial<
 Func extends (...args: any[]) => any,
 InitialArgs extends unknown[],
>(func: Func, ...initialArgs: InitialArgs) {
 return (...remainingArgs: RemainingArgs<Func, InitialArgs>)
 : ReturnType<Func> => {
 return func(...initialArgs, ...remainingArgs);
 };
}

//----- Test -----

function add(x: number, y: number): number {
 return x + y;
}
const add3 = applyPartial(add, 3);
type _1 = Assert<Equal<
 typeof add3,
 // The parameter name is preserved!
 (y: number) => number
>>;

```

我们返回一个部分应用的 `func`。为了计算参数 `remainingArgs` 的类型，我们从 `Func` 的参数中移除 `InitialArgs` – 通过以下实用类型：

```ts
type RemainingArgs<
 Func extends (...args: any[]) => any,
 InitialArgs extends unknown[],
> =
 Func extends (
 ...args: [...InitialArgs,
 ...infer TrailingArgs]
 ) => unknown
 ? TrailingArgs
 : never
;

//----- Test -----

type _2 = Assert<Equal<
 RemainingArgs<typeof add, [number]>,
 [y: number]
>>;

```

#### 37.7.2 输入函数 `zip()`

考虑一个将可迭代元组转换为元组可迭代对象的 `zip()` 函数（[实现源代码](https://github.com/rauschma/iterable/blob/main/ts/src/sync.ts)）：

```ts
> zip([[1, 2, 3], ['a', 'b', 'c']])
[ [1, 'a'], [2, 'b'], [3, 'c'] ]

```

以下实用类型 `Zip` 计算其返回类型：

```ts
type Zip<Tuple extends Array<Iterable<unknown>>> =
 Iterable<
 { [Key in keyof Tuple]: UnwrapIterable<Tuple[Key]> }
 >
;
type UnwrapIterable<Iter> =
 Iter extends Iterable<infer T>
 ? T
 : never
;

type _ = Assert<Equal<
 Zip<[Iterable<string>, Iterable<number>]>,
 Iterable<[string, number]>
>>;

```

#### 37.7.3 输入函数 `zipObj()`

函数 `zipObj()` 与 `zip()` 类似：它将可迭代对象转换为对象可迭代对象（[实现源代码](https://github.com/rauschma/iterable/blob/main/ts/src/sync.ts)）：

```ts
> zipObj({num: [1, 2, 3], str: ['a', 'b', 'c']})
[ {num: 1, str: 'a'}, {num: 2, str: 'b'}, {num: 3, str: 'c'} ]

```

以下实用类型 `ZipObj` 计算其返回类型：

```ts
type ZipObj<Obj extends Record<string, Iterable<unknown>>> =
 Iterable<
 { [Key in keyof Obj]: UnwrapIterable<Obj[Key]> }
 >
;
type UnwrapIterable<Iter> =
 Iter extends Iterable<infer T>
 ? T
 : never
;

type _ = Assert<Equal<
 ZipObj<{a: Iterable<string>, b: Iterable<number>}>,
 Iterable<{a: string; b: number}>
>>;

```

#### 37.7.4 `util.promisify()`：将基于回调的函数转换为基于 Promise 的函数

Node.js 函数 `util.promisify(cb)`（[官方文档](https://nodejs.org/api/util.html#utilpromisifyoriginal)）将返回其结果通过回调的函数转换为返回其结果通过 Promise 的函数。其官方类型定义很长：

```ts
// 0 arguments
export function promisify<TResult>(
 fn: (callback: (err: any, result: TResult) => void) => void,
): () => Promise<TResult>;
export function promisify(
 fn: (callback: (err?: any) => void) => void
): () => Promise<void>;

// 1 argument
export function promisify<T1, TResult>(
 fn: (arg1: T1, callback: (err: any, result: TResult) => void) => void,
): (arg1: T1) => Promise<TResult>;
export function promisify<T1>(
 fn: (arg1: T1, callback: (err?: any) => void) => void
): (arg1: T1) => Promise<void>;

// 2 arguments
export function promisify<T1, T2, TResult>(
 fn: (arg1: T1, arg2: T2, callback: (err: any, result: TResult) => void) => void,
): (arg1: T1, arg2: T2) => Promise<TResult>;
export function promisify<T1, T2>(
 fn: (arg1: T1, arg2: T2, callback: (err?: any) => void) => void,
): (arg1: T1, arg2: T2) => Promise<void>;

// Etc.: up to 5 arguments

```

让我们尝试简化它：

```ts
function promisify<Args extends any[], CB extends NodeCallback>(
 fn: (...args: [...Args, CB]) => void,
): (...args: Args) => Promise<ExtractResultType<CB>> {
 // ···
}
type NodeCallback =
 | ((err: any, result: any) => void)
 | ((err: any) => void)
;

//----- Test -----

function nodeFunc(
 arr: Array<string>,
 cb: (err: Error, str: string) => void
) {}
const asyncFunc = promisify(nodeFunc);
assertType<
 (arr: string[]) => Promise<string>
>(asyncFunc);

```

之前的代码使用了以下实用类型：

```ts
type ExtractResultType<F extends NodeCallback> =
 F extends (err: any) => void
 ? void
 : F extends (err: any, result: infer TResult) => void
 ? TResult
 : never
;

//----- Test -----

type _ = [
 Assert<Equal<
 ExtractResultType<(err: Error, result: string) => void>,
 string
 >>,
 Assert<Equal<
 ExtractResultType<(err: Error) => void>,
 void
 >>,
];

```

### 37.8 使用元组进行计算的局限性

有一些约束我们无法通过 TypeScript 的类型系统来表示。以下代码是一个例子：

```ts
type Same<T> = {a: T, b: T};

function one<T>(obj: Same<T>) {}
// @ts-expect-error: Type 'string' is not assignable to type 'boolean'.
one({a: false, b: 'abc'}); // 👍 error

function many<A, B, C, D, E>(
 objs: [Same<A>, Same<B>]
 | [Same<A>, Same<B>, Same<C>]
 | [Same<A>, Same<B>, Same<C>, Same<D>]
 | [Same<A>, Same<B>, Same<C>, Same<D>, Same<E>,
 ...Array<Same<unknown>>]
) {}

many([
 {a: true, b: true},
 {a: 'abc', b: 'abc'},
 // @ts-expect-error: Type 'boolean' is not assignable to type 'number'.
 {a: 7, b: false} // 👍 error
]);

```

我们想表达：

+   函数 `many()` 接收一个对象数组。

+   两个属性的类型应该是相同的。

我们不能循环并每次循环迭代引入一个变量。因此，我们手动列出最常见的案例。

### 37.9 源于本章的内容

+   由安德斯·海尔斯伯格提出的拉取请求[“可变元组类型”](https://github.com/microsoft/TypeScript/pull/39094)

+   TypeScript 4.0 博客文章中的[“可变元组类型”](https://devblogs.microsoft.com/typescript/announcing-typescript-4-0/#variadic-tuple-types)部分

+   TypeScript 4.0 博客文章中的[“标签化元组元素”](https://devblogs.microsoft.com/typescript/announcing-typescript-4-0/#labeled-tuple-elements)部分
