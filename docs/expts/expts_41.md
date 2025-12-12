# 33 概述：使用类型进行计算

> 原文：[`exploringjs.com/ts/book/ch_computing-with-types-overview.html`](https://exploringjs.com/ts/book/ch_computing-with-types-overview.html)

(Ad, please don’t block.)

1.  33.1 TypeScript 中的计算：程序级与类型级

1.  33.2 在类型级别上我们可以计算的类型“值”

    1.  33.2.1 原始类型

    1.  33.2.2 字面量类型

    1.  33.2.3 非泛型对象类型

    1.  33.2.4 复合类型

    1.  33.2.5 字面量类型的联合作为值集合

1.  33.3 泛型类型是类型级函数

    1.  33.3.1 术语：泛型类型、参数化类型、具体类型

    1.  33.3.2 可选类型参数

    1.  33.3.3 约束类型参数

1.  33.4 `typeof` 类型运算符：从类型级别引用程序级

    1.  33.4.1 程序级 `typeof` 与类型级 `typeof`

    1.  33.4.2 `typeof` 的语法

1.  33.5 `keyof` 类型运算符

    1.  33.5.1 数字键：JavaScript 与 TypeScript

    1.  33.5.2 `keyof` 和索引签名

    1.  33.5.3 数组的 `keyof`

    1.  33.5.4 元组的 `keyof`

    1.  33.5.5 交集类型和联合类型的 `keyof`

1.  [33.6 索引访问类型 `T[K]`](#indexed-access-types)

    1.  [33.6.1 `T[K]`: `K` 必须是 `T` 的键的子集](#indexed-access-type-index)

    1.  33.6.2 元组的索引访问

    1.  33.6.3 示例：实现 `ValueOf`

    1.  33.6.4 示例：获取属性值

    1.  33.6.5 示例：查找表和索引访问

    1.  33.6.6 `lib.dom.d.ts` 中的索引访问和查找表

1.  33.7 条件类型 (`C ? T : F`)

    1.  33.7.1 通过 `infer` 提取复合类型的部分

1.  33.8 定义局部类型变量

    1.  33.8.1 示例

1.  [33.9 映射类型 `{[K in U]: X}`](#mapped-types-k-in-u-x)

1.  33.10 模板字面量类型：字符串处理

1.  33.11 使用联合类型进行计算

    1.  33.11.1 交集与联合

    1.  33.11.2 对联合类型的分配律

    1.  33.11.3 模板字面量类型的分配律

    1.  [33.11.4 索引访问类型 `T[K]` 是分配性的](#distributivity-of-indexed-access-types)

    1.  33.11.5 条件类型是分配性的

1.  33.12 使用对象类型进行计算

1.  33.13 使用元组类型进行计算

1.  33.14 函数的返回类型通常与返回值不匹配

1.  33.15 结论

在本章中，我们探讨在 TypeScript 中如何在编译时使用类型进行计算。

![图标“问题”](img/471cce0defd950c2994152f322a88405.png) **在实践中使用类型计算有用吗？**

我们首先必须学习基础知识，而且一些例子可能看起来有点抽象。但那些基础知识有助于解决实际问题——其中一些在 结论 中列出。

如果你正在 *使用* 库，你通常可以不使用类型计算而过得去。然而，如果你正在 *编写* 库，类型计算往往很有用。

### 33.1 TypeScript 中的计算：程序级别与类型级别

TypeScript 代码有两个计算级别：

+   程序级别：在运行时，我们可以通过值和函数进行计算。

+   类型级别：在编译时，我们可以通过具体类型和泛型类型进行计算。

|  | 程序级别 | 类型级别 |
| --- | --- | --- |
| 编程语言 | JavaScript | TypeScript（不包括 JS） |
| 操作数 | 值 | 具体类型 |
| 操作 | 函数 | 泛型类型 |
| 操作调用 | 调用一个函数 | 实例化一个泛型类型 |
| 计算发生 | 运行时 | 编译时 |

这是在类型级别上进行计算的一个例子：

```ts
type Result = Uppercase<'hello'>;
type _ = Assert<Equal<
 Result, 'HELLO'
>>;

```

`Uppercase` 是一个泛型类型。它的参数，用尖括号括起来，是字符串字面量类型 `'hello'`。泛型类型的实例化结果是字符串字面量类型 `'HELLO'`。

程序级别的类似计算看起来是这样的：

```ts
const result = 'hello'.toUpperCase();
assert.equal(
 result, 'HELLO'
);

```

在下一小节中，我们将检查在类型级别上可以使用的“值”。然后我们将定义我们自己的类型级别“函数”。

### 33.2 “在类型级别上我们可以计算”的“值”

在类型级别上，我们可以使用以下“值”进行计算。

#### 33.2.1 原始类型

这些是原始类型：

+   `undefined`

+   `null`

+   `boolean`

+   `number`

+   `bigint`

+   `string`

+   `symbol`

尽管其中两个看起来像 JavaScript 值，但我们仍然在类型级别上操作：

+   `undefined` 是一个只有 `undefined` 值的类型。

+   `null` 是一个只有 `null` 值的类型。

#### 33.2.2 字面量类型

这些是字面量类型的例子：

```ts
type BooleanLiteralType = true;
type NumberLiteralType = 12.34;
type BigIntLiteralType = 1234n;
type StringLiteralType = 'abc';

```

我们仍然在类型级别上操作：

+   `true` 是一个只有 `true` 值的类型。它是 `boolean` 的子集。

+   `12.34` 是一个只有 `12.34` 值的类型。它是 `number` 的子集。

+   等等。

#### 33.2.3 非泛型对象类型

这些是非泛型对象类型的示例：

+   `RegExp`

+   `Date`

+   `Uint8Array`

#### 33.2.4 复合类型

我们还可以组合类型来生成新的类型——例如：

```ts
type InstantiatedGenericType = Array<number>;

type TupleType = [boolean, bigint];

type ObjectLiteralType = {
 prop1: string,
 prop2: number,
};

```

#### 33.2.5 字面量类型的联合作为值集

在类型计算中，字面量类型的联合通常用于表示值集——例如：

```ts
type Person = {
 givenName: string,
 familyName: string,
};
type _ = Assert<Equal<
 keyof Person,
 'givenName' | 'familyName'
>>;

```

`keyof` 操作符 返回对象类型的键。它通过字符串字面量类型的联合来实现这一点。

### 33.3 泛型类型是类型级别的函数

以下示例是类型级别的代码（在编译时执行）：

```ts
type Pair<T> = [T, T]; // (A)
type Result = Pair<'abc'>; // (B)
type _ = Assert<Equal<
 Result, ['abc', 'abc']
>>;

```

+   在行 A 中，我们定义了具有一个名为 `T` 的 *类型参数* 的 *泛型类型* `Pair`。

+   在行 B 中，我们定义类型 `Result` 为 `Pair` 的实例化，其类型为字符串字面量类型 `'abc'`。

+   注意，`Pair` 的主体包含类型变量（`T`，两次）。当我们实例化它时，这些变量会被具体的类型所替换。

以下示例类似于程序级别的代码（在运行时执行）：

```ts
const pair = (x) => [x, x];
const result = pair('abc');
assert.deepEqual(
 result, ['abc', 'abc']
);

```

#### 33.3.1 术语：泛型类型、参数化类型、具体类型

我喜欢 [Angelika Langer 的定义](http://www.angelikalanger.com/GenericsFAQ/FAQSections/ParameterizedTypes.html#FAQ001)：

+   *泛型类型* 是具有形式类型参数的类型。

+   *参数化类型* 是泛型类型的实例化，它使用实际的类型参数。

例如：

```ts
type Pair<T> = [T, T];

```

`Pair` 是一个泛型类型。`Pair<3>` 是一个参数化类型——`Pair` 的一个实例。我们说 `Pair<3>` *构造*（“返回”）类型 `[3, 3]`。

*具体类型* 是一个特定的（可能是复合的）类型，可以在类型注解中使用：

```ts
let v1: number;
let v2: Pair<3>;

```

#### 33.3.2 可选类型参数

我们可以通过指定默认值来使类型参数可选，默认值通过等号（`=`）指定：

```ts
type Pair<T='hello'> = [T, T];
type _ = Assert<Equal<
 Pair,
 ['hello', 'hello']
>>;

```

#### 33.3.3 约束类型参数

如果类型参数定义仅仅是变量，它接受任何类型，但我们也可以通过关键字 `extends` 来约束它接受的类型：

```ts
type NumberPair<T extends number> = [T, T];

type P1 = NumberPair<123>; // OK
type P2 = NumberPair<number>; // OK

// @ts-expect-error: Type 'string' does not satisfy the constraint 'number'.
type P3 = NumberPair<'abc'>;

```

`T extends C` 的意思是：

+   `T` 必须可以赋值给 `C`。

+   `T` 必须是 `C` 的子集。

在泛型类型的参数定义中，`extends` 与函数参数定义中的冒号（`:`）类似：

```ts
type NumberPair<T extends number> = [T, T];
const numberPair = (x: number) => [x, x];

```

我们还可以将 `extends` 与参数默认值结合使用：

```ts
type NumberPair<T extends number = 0> = [T, T];
type _ = Assert<Equal<
 NumberPair,
 [0, 0]
>>;

```

### 33.4 `typeof` 类型操作符：从类型级别引用程序级别

（非类型）变量和类型表达式存在于两个不同的级别：

+   变量存在于程序级别。

+   类型表达式存在于类型级别。

因此，我们无法直接在类型表达式中提及变量。然而，类型级别的 `typeof` 操作符使我们能够在类型表达式中引用变量的类型：

```ts
let programLevelVariable = 'abc';

// The right-hand side of `=` is a type expression
type TypeLevelType = Array<typeof programLevelVariable>;
type _ = Assert<Equal<
 TypeLevelType,
 Array<string>
>>;

```

#### 33.4.1 程序级别的 `typeof` 与类型级别的 `typeof`

JavaScript 也有一个`typeof`运算符——它在程序级别上操作。对于给定的值，它返回其类型的名称作为字符串：

```ts
assert.equal(
 typeof 'abc',
 'string'
);
assert.equal(
 typeof 123,
 'number'
);

```

类型级别的`typeof`的结果通常比程序级别的`typeof`的结果更复杂：

```ts
const robin = {
 givenName: 'Robin',
 familyName: 'Doe',
};
type _ = Assert<Equal<
 typeof robin,
 {
 givenName: string;
 familyName: string;
 }
>>;

```

#### 33.4.2 `typeof`的语法

操作数必须是一个标识符，该标识符可以可选地后跟成员访问（点操作符或方括号操作符）。例如：

```ts
const article = {
 tags: ['dev', 'typescript'],
};
type _ = [
 Assert<Equal<
 typeof article,
 {
 tags: string[];
 }
 >>,
 Assert<Equal<
 typeof article.tags,
 string[]
 >>,
 Assert<Equal<
 typeof article.tags[0],
 string
 >>,
];

```

任何其他操作数都会产生语法错误：

```ts
type _ = typeof 'abc';
 // Error: Identifier expected.

```

### 33.5 `keyof`类型运算符

类型运算符`keyof`列出了对象类型的属性键：

```ts
type Obj = {
 prop1: 'a',
 prop2: 'b',
};

type _ = Assert<Equal<
 keyof Obj,
 'prop1' | 'prop2'
>>;

```

空对象类型的属性键是空集`never`：

```ts
type _ = Assert<Equal<
 keyof {},
 never
>>;

```

#### 33.5.1 数字键：JavaScript 与 TypeScript

##### 33.5.1.1 JavaScript 中的数字键

JavaScript 将所有数字键（无论是否加引号）都视为字符串：

```ts
> Object.keys({0: 'a', 1: 'b'})
[ '0', '1' ]
> Object.keys({'0': 'a', '1': 'b'})
[ '0', '1' ]

```

类似地，数组元素是属性，其键是字符串化的数字：

```ts
> Object.keys(['a', 'b'])
[ '0', '1' ]

```

关于 JavaScript 中数组元素的信息，请参阅[“探索 JavaScript”](https://exploringjs.com/js/book/ch_arrays.html#arrays-are-actually-dictionaries)。

##### 33.5.1.2 TypeScript 中的数字键

在对象字面量类型中，未加引号的数字键是数字字面量类型，加引号的数字键是字符串字面量类型：

```ts
type _ = Assert<Equal<
 keyof {0: 'a', '1': 'b'},
 0 | '1'
>>;

```

如果类型是从 JavaScript 值派生的，TypeScript 也会做出这种区分：

```ts
const obj = {0: 'a', '1': 'b'};
type _ = Assert<Equal<
 keyof typeof obj,
 0 | '1'
>>;

```

数组类型的索引是数字（注意第一行中的`Includes`）：

```ts
type _ = Assert<Includes<
 keyof Array<string>,
 number // type for all indices
>>;

```

#### 33.5.2 `keyof`和索引签名

数字索引签名键的类型是`number`：

```ts
type NumberIndexSignature = {
 [k: number]: unknown,
};
type _ = Assert<Equal<
 keyof NumberIndexSignature,
 number
>>;

```

字符串索引签名键的类型是`string | number`，因为在 JavaScript 中，数字键是字符串键的子集（如前一小节所述）：

```ts
type StringIndexSignature = {
 [k: string]: unknown,
};
type _ = Assert<Equal<
 keyof StringIndexSignature,
 string | number
>>;

```

#### 33.5.3 数组的`keyof`

数组类型的键包括多种类型（注意第一行中的`Includes`）：

```ts
type _ = Assert<Includes<
 keyof Array<string>,
 number | 'length' | 'push' | 'join'
>>;

```

键包括：

+   数组索引的类型为`number`

+   特殊实例属性`.length`的名称

+   所有`Array`方法的名称：`'push' | 'join' | ···`

#### 33.5.4 元组的`keyof`

由于元组主要是数组，它们的键看起来很相似（注意第一行中的`Includes`）：

```ts
type _ = Assert<Includes<
 keyof ['a', 'b'],
 number | '0' | '1' | 'length' | 'push' | 'join'
>>;

```

与数组类似，有`number`、`'length'`和方法名称。此外，每个元素都有一个字符串化的索引。

有关此主题的更多信息，包括如何提取元组索引，请参阅“元组类型的键”（§37.3）。

#### 33.5.5 交集类型和联合类型的`keyof`

这就是`keyof`如何处理交集类型和联合类型：

```ts
type A = { a: number, shared: string };
type B = { b: number, shared: string };

type _1 = Assert<Equal<
 keyof (A & B),
 'a' | 'b' | 'shared'
>>;

type _2 = Assert<Equal<
 keyof (A | B),
 'shared'
>>;

```

如果我们记住这一点，这就有意义了：

+   类型`A & B`的对象具有类型`A`和类型`B`的属性。

+   类型`A | B`的对象要么具有类型`A`的属性，要么具有类型`B`的属性。也就是说，只有两种类型都有的属性始终存在。

### [33.6 索引访问类型 `T[K]`](#indexed-access-types)

索引访问操作符 `T[K]` 返回所有键可赋值给类型 `K` 的 `T` 的属性类型。`T[K]` 也被称为 *查找类型*。

这些是使用该操作符的例子：

```ts
type Obj = {
 0: 'a',
 1: 'b',
 prop0: 'c',
 prop1: 'd',
 [Symbol.iterator]: 'e',
};

type _ = [
 Assert<Equal<
 Obj[0 | 1],
 'a' | 'b'
 >>,
 // The stringified versions of number keys work the same
 Assert<Equal<
 Obj['0' | '1'],
 'a' | 'b'
 >>,
 Assert<Equal<
 Obj['prop0' | 'prop1'],
 'c' | 'd'
 >>,
 Assert<Equal<
 Obj[keyof Obj],
 'a' | 'b' | 'c' | 'd' | 'e'
 >>,
 // - Symbol.iterator is a value (program level).
 // - typeof Symbol.iterator is a type (type level).
 Assert<Equal<
 Obj[typeof Symbol.iterator],
 'e'
 >>,
];

```

#### [33.6.1 `T[K]`：`K` 必须是 `T` 的键的子集](#indexed-access-type-index)

括号中的类型必须可赋值给所有属性键的类型（由 `keyof` 计算得出）。这就是为什么不允许在这里使用 `Obj[string]` 和 `Obj[number]`：

```ts
type Obj = {prop: 'yes'};
type _ = [
 // @ts-expect-error: Type 'Obj' has no matching index signature for type
 // 'string'.
 Obj[string],
 // @ts-expect-error: Type 'Obj' has no matching index signature for type
 // 'number'.
 Obj[number],
];

```

然而，如果索引类型有一个索引签名（行 A），我们可以使用 `string` 和 `number` 作为索引类型：

```ts
type Obj = {
 [key: string]: RegExp, // (A)
};
type _ = [
 Assert<Equal<
 keyof Obj, // (B)
 string | number
 >>,
 Assert<Equal<
 Obj[string],
 RegExp
 >>,
 Assert<Equal<
 Obj[number],
 RegExp
 >>,
];

```

`keyof Obj`（行 B）包括类型 `number`，因为数字键是字符串键的子集（在 JavaScript 中，因此也在 TypeScript 中）。

#### 33.6.2 元组的索引访问

元组类型也支持索引访问：

```ts
type Tuple = ['a', 'b', 'c'];
type _ = [
 Assert<Equal<
 Tuple[0 | 1],
 'a' | 'b'
 >>,
 Assert<Equal<
 Tuple['0' | '1'],
 'a' | 'b'
 >>,
 Assert<Equal<
 Tuple[number],
 'a' | 'b' | 'c'
 >>,
];

```

我们可以用 `number` 作为索引，因为元组的 `keyof` 包含了类型 `number` (更多信息)。

#### 33.6.3 示例：实现 `ValueOf`

TypeScript 有一个 `keyof` 操作符但没有 `valueof` 操作符。然而，我们可以自己实现这个操作符：

```ts
type ValueOf<T> = T[keyof T];

type Obj = { a: string, b: number };
type _ = Assert<Equal<
 ValueOf<Obj>,
 string | number
>>;

```

#### 33.6.4 示例：获取属性值

以下函数检索 `obj` 的属性值，其键为 `key`：

```ts
function get<O, K extends keyof O>(obj: O, key: K): O[K] {
 return obj[key];
}
const obj = {
 a: 1,
 b: 2,
};
const result = get(obj, 'a');
assert.equal(result, 1);
assertType<number>(result);

```

很有趣的是，除了正确计算 `result` 的类型外，TypeScript 还会警告我们如果键错误：

```ts
// @ts-expect-error: Argument of type '"aaa"' is not assignable to
// parameter of type '"a" | "b"'.
get(obj, 'aaa');

```

#### 33.6.5 示例：查找表和索引访问

多亏了索引访问操作符，我们可以轻松地将一种类型映射到另一种类型：

```ts
type TypeofLookupTable = {
 'undefined': undefined,
 'boolean': boolean,
 'number': number,
 'bigint': bigint,
 'string': string,
 'symbol': symbol,
 'object': null | object,
 'function': Function,
};
type TypeofResult = keyof TypeofLookupTable;
type TypeofStringToType<S extends TypeofResult> = TypeofLookupTable[S];
type _ = [
 Assert<Equal<
 TypeofStringToType<'undefined'>,
 undefined
 >>,
 Assert<Equal<
 TypeofStringToType<'bigint'>,
 bigint
 >>,
];

```

殊可惜，如果键类型是 `string`、`number` 或 `symbol` 的子集，对象字面量类型仅作为查找表工作。对于其他类型，我们需要使用元组来工作。

#### 33.6.6 `lib.dom.d.ts` 中的索引访问和查找表

DOM 的内置类型定义（[`lib.dom.d.ts`](https://github.com/microsoft/TypeScript/blob/main/src/lib/dom.generated.d.ts)）使用索引访问和查找表 `GlobalEventHandlersEventMap`：

```ts
interface GlobalEventHandlersEventMap {
 "abort": UIEvent;
 "animationcancel": AnimationEvent;
 "animationend": AnimationEvent;
 "animationiteration": AnimationEvent;
 "animationstart": AnimationEvent;
 "auxclick": MouseEvent;
 "beforeinput": InputEvent;
 "beforetoggle": Event;
 "blur": FocusEvent;
 "cancel": Event;
 "canplay": Event;
 "canplaythrough": Event;
 "change": Event;
 "click": MouseEvent;
 // ···
}

/** One of the interfaces extended by interface `Window` */
interface GlobalEventHandlers {
 // ···
 addEventListener<K extends keyof GlobalEventHandlersEventMap>(
 type: K, // a string
 listener: (
 this: GlobalEventHandlers,
 ev: GlobalEventHandlersEventMap[K]
 ) => any,
 options?: boolean | AddEventListenerOptions
 ): void;
}

```

### 33.7 条件类型 (`C ? T : F`)

*条件类型* 有以下语法：

```ts
«Sub» extends «Super» ? «TrueBranch» : «FalseBranch»

```

如果 `Sub` 可以赋值给 `Super`，则条件类型的结果是 `TrueBranch`。否则，它是 `FalseBranch`。这是一个例子：

```ts
type IsNumber<T> = T extends number ? true : false;
type _ = [
 Assert<Equal<
 IsNumber<123>, true
 >>,
 Assert<Equal<
 IsNumber<number>, true
 >>,
 Assert<Equal<
 IsNumber<'abc'>, false
 >>,
];

```

更多信息请参阅“条件类型 (`C ? T : F`)”（§34）“Conditional types (`C ? T : F`)” (§34)。

#### 33.7.1 通过 `infer` 提取复合类型的部分

`infer` 关键字只能用于条件类型的条件中，并将复合类型的部分提取到类型变量中 - 例如，以下泛型类型提取了 `Array<>` 角括号内的内容：

```ts
type ElemType<Arr> = Arr extends Array<infer Elem> ? Elem : never;
type _ = Assert<Equal<
 ElemType<Array<string>>, string
>>;

```

`infer` 与 JavaScript 中的 [解构](https://exploringjs.com/js/book/ch_destructuring.html#ch_destructuring) 有很多共同之处。

更多信息，请参阅“通过 `infer` 提取复合类型的部分”（§35）“Extracting parts of compound types via `infer`” (§35)。

### 33.8 定义局部类型变量

正常的编程语言允许我们定义局部变量来帮助管理各种数据。然而，TypeScript 的类型级别没有这个功能。如果有，它看起来会是这样：

```ts
type Result = let Var = «Value» in «Body»;

```

然而，我们可以通过 `infer` 来模拟它：

```ts
type Result = «Value» extends infer Var ? «Body» : never;

```

我们还可以同时定义多个变量：

```ts
type Result = [«Value1», «Value2», «Value3»] extends
 infer [Var1, Var2, Var3]
 ? «Body»
 : never
;

```

#### 33.8.1 示例

这是一个这种技术有用的例子：

```ts
type WrapTriple<T> = Promise<T> extends infer W
 ? [W, W, W]
 : never
;
type _ = Assert<Equal<
 WrapTriple<number>,
 [Promise<number>, Promise<number>, Promise<number>]
>>;

```

在泛型类型的“主体”中，我们还可以使用不同的技术——一个具有默认值的辅助参数（以下代码中的 `W`）：

```ts
type WrapTriple2<T, W=Promise<T>> = [W, W, W];

```

### [33.9 映射类型 `{[K in U]: X}`](#mapped-types-k-in-u-x)

大致来说，映射类型通过遍历其键来创建输入类型 `T`（通常是对象类型或元组类型）的新版本：

```ts
{
 [K in keyof T]: «PropValue»
}

```

`«PropValue»` 是一个类型表达式，它通常以某种方式使用 `K`。这是一个例子：

```ts
type InputObj = {
 str: string,
 num: number,
};
type Arrayify<Obj> = {
 [K in keyof Obj]: Array<Obj[K]>
};
type _ = Assert<Equal<
 Arrayify<InputObj>,
 {
 str: Array<string>,
 num: Array<number>,
 }
>>;

```

### 33.10 模板字面量类型：字符串处理

模板字面量类型与 JavaScript 模板字面量的语法相同。它们有两个重要的用例：

首先，连接字符串字面量类型（模板字面量位于行 A，由反引号分隔）：

```ts
type MethodName = 'compute';
type AsyncMethodName = `async${Capitalize<MethodName>}`; // (A)
type _ = Assert<Equal<
 AsyncMethodName, 'asyncCompute'
>>;

```

第二，提取字符串字面量类型的一部分：

```ts
type AsyncMethodName = 'asyncCompute';
type MethodName = Uncapitalize<
 AsyncMethodName extends `async${infer MN}` ? MN : never // (A)
>;
type _ = Assert<Equal<
 MethodName, 'compute'
>>;

```

在行 A 中，我们通过 `infer` 操作符将 `AsyncMethodName` 的一部分提取到类型变量 `MN` 中。该操作符在 JavaScript 中的解构操作中工作方式类似。它必须在一个条件类型（`Cond ? True : False`）内部使用。

连接和提取字符串字面量类型在许多情况下都很有用，例如，它们使我们能够转换对象属性的名称。

### 33.11 使用联合类型进行计算

在本节中，我们探讨如何使用联合类型进行计算。

#### 33.11.1 交集和联合

我们可以通过 `&` 来交集联合类型：

```ts
type Union1 = 'a' | 'b' | 0 | 1;
type Union2 = 'b' | 'c' | 1 | 2;
type Intersection = Union1 & Union2;
type _ = Assert<Equal<
 Intersection,
 1 | 'b'
>>;

```

并且，正如预期的那样，我们也可以通过 `|` 计算联合：

```ts
type Union1 = 'a' | 'b';
type Union2 = 'b' | 'c';
type UnionResult = Union1 | Union2;
type _ = Assert<Equal<
 UnionResult,
 'a' | 'b' | 'c'
>>;

```

#### 33.11.2 联合类型的分配律

联合类型的一个有趣现象是，某些操作对它们是*分配律的*：

+   将操作应用于非联合类型会产生一个单一的输出类型。

+   将操作应用于联合类型会产生输出类型的联合——每个输入联合的元素对应一个。

#### 33.11.3 模板字面量类型是分配律的

下一个示例演示了将行 A 中的模板字面量类型应用于联合会产生字符串字面量类型的联合。

```ts
type Union = 'l' | 'f' | 'r';
type _ = Assert<Equal<
 `${Union}ight`, // (A)
 'light' | 'fight' | 'right'
>>;

```

#### [33.11.4 索引访问类型 `T[K]` 是分配律的](#distributivity-of-indexed-access-types)

下一个例子将索引访问类型应用于对象字面量类型的联合：

```ts
type Union = { prop: 1 } | { prop: 2 } | { prop: 3 };
type _ = Assert<Equal<
 Union['prop'],
 1 | 2 | 3
>>;

```

#### 33.11.5 条件类型是分配律的

因为它们是分配的，条件类型是处理联合类型最重要的工具。在本节中，我们将探讨一些示例。更多信息请见“条件类型在联合类型上是分配的”（§34.2）。

##### 33.11.5.1 映射联合类型

```ts
type WrapStrings<T> = T extends string ? Promise<T> : T;
type _ = [
 Assert<Equal<
 WrapStrings<'abc'>, // normal instantiation
 Promise<'abc'>
 >>,
 Assert<Equal<
 WrapStrings<123>, // normal instantiation
 123
 >>,
 Assert<Equal<
 WrapStrings<'a' | 'b' | 0 | 1>, // distributed instantiation
 Promise<'a'> | Promise<'b'> | 0 | 1
 >>,
];

```

##### 33.11.5.2 过滤联合类型

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

### 33.12 使用对象类型进行计算

如何使用对象类型进行计算在其他地方有解释：

+   我们可以通过映射类型来转换和创建对象类型。

+   模板字面量类型帮助我们更改属性名。

### 33.13 使用元组类型进行计算

见“使用元组类型进行计算”（§37）：

+   我们可以通过映射类型来映射元组。

+   对于其他操作，例如过滤，我们通常需要递归。

### 33.14 函数的计算返回类型通常不匹配返回值

函数计算返回类型的一个缺点是 TypeScript 通常认为返回值的类型与计算类型不匹配。这是一个例子：

```ts
type PrependDollarSign<Obj> = {
 [Key in (keyof Obj & string) as `$${Key}`]: Obj[Key]
};
function prependDollarSign<
 Obj extends object
>(obj: Obj): PrependDollarSign<Obj> { // (A)
 // @ts-expect-error: Type '{ [k: string]: any; }' is not assignable to
 // type 'PrependDollarSign<Obj>'.
 return Object.fromEntries( // (B)
 Object.entries(obj)
 .map(
 ([key, value]) => ['$'+key, value]
 )
 );
}

```

很遗憾，行 B 中返回的值不能赋值给行 A 中指定的返回类型。解决这个问题有几种方法——所有这些方法都涉及到类型断言（`as`）。这是一个解决方案——在行 B 中使用`as any`：

```ts
function prependDollarSign<
 Obj extends object
>(obj: Obj): PrependDollarSign<Obj> { // (A)
 return Object.fromEntries(
 Object.entries(obj)
 .map(
 ([key, value]) => ['$'+key, value]
 )
 ) as any; // (B)
}

const dollarObject = prependDollarSign({
 prop: 123,
});
assert.deepEqual(
 dollarObject,
 {
 $prop: 123,
 }
);
assertType<
 {
 $prop: number,
 }
>(dollarObject);

```

获取正确返回类型的选项：

+   明确定义返回类型（如我们在行 A 中所做）加上行 B 中的类型断言：

    +   `as any`（我们的当前解决方案）

    +   `as unknown as PrependDollarSign<Obj>`（我们已采取的替代方案）。

    +   不起作用：`as unknown`

+   推断返回类型（在行 A 中省略）加上行 B 中的类型断言：

    +   `as unknown as PrependDollarSign<Obj>`

我更喜欢上面使用的解决方案，因为推断的返回类型可以防止生成`.d.ts`文件的一些方式：“`isolatedDeclarations`：更有效地生成`.d.ts`文件”（§8.8.5）

### 33.15 结论

使用类型进行计算非常迷人：

+   我们正在类型级别编写程序。

+   它使我们能够为相对复杂的功能提供类型，例如：

    +   `Promise.all()`

    +   `zip()`

    +   属性路径

    +   在类型级别将短横线命名转换为驼峰命名

+   我们实际上在两个级别上实现了解决方案（参见上一节）：一次在程序级别，一次在类型级别。

计算类型确实会使代码更加复杂。我的总体建议是：尽可能保持类型简单，并且只有在绝对必要时才进行类型级别的计算。在某些情况下，通过重构代码和/或数据，可能可以使用更简单的类型。

另一方面，有些项目中编写类型需要智慧，但使用它们却很有趣。一个小例子是我编写的一个简单的 SQL API 原型[链接](https://github.com/rauschma/simple-sql)。
