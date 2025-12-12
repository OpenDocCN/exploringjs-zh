# 34 条件类型 (C ? T : F)

> 原文：[`exploringjs.com/ts/book/ch_conditional-types.html`](https://exploringjs.com/ts/book/ch_conditional-types.html)

(广告，请勿屏蔽。)

1.  34.1 语法和第一个例子

    1.  34.1.1 条件类型的链式调用

    1.  34.1.2 条件类型的嵌套

    1.  34.1.3 示例：仅包装具有 `.length` 属性的类型

1.  34.2 条件类型在联合类型上的分配律

    1.  34.2.1 只有 `extends` 的左侧被分配

    1.  34.2.2 只有类型变量触发分配

    1.  34.2.3 防止分配律

    1.  34.2.4 技术：始终应用

1.  34.3 通过条件返回 `never` 过滤联合类型

    1.  34.3.1 内置实用类型 `Exclude<T, U>`

    1.  34.3.2 内置实用类型 `Extract<T, U>`

1.  34.4 通过条件类型中的 `infer` 提取复合类型的部分

1.  34.5 为条件类型编写条件

    1.  34.5.1 检查一个类型是否可以赋值给另一个类型？

    1.  34.5.2 检查泛型类型是否返回特定值

    1.  34.5.3 检查两个类型是否相等

    1.  34.5.4 逻辑或

    1.  34.5.5 逻辑与

1.  34.6 延迟条件类型

1.  34.7 本章来源

TypeScript 中的条件类型是一个 if-then-else 表达式：它的结果要么是两个分支之一——具体是哪一个取决于条件。这在泛型类型中特别有用。条件类型也是处理联合类型的一个基本工具，因为它们允许我们“遍历”它们。如果你想知道这一切是如何工作的，请继续阅读。

### 34.1 语法和第一个例子

一个 *条件类型* 有以下语法：

```ts
«Sub» extends «Super» ? «TrueBranch» : «FalseBranch»

```

条件类型有三个部分：

+   如果 `Sub` 可以赋值给 `Super`…（条件）

+   …然后这个类型表达式的结果是 `TrueBranch`。

+   否则，结果是 `FalseBranch`。

我喜欢这样格式化较长的条件类型：

```ts
«Sub» extends «Super»
 ? «TrueBranch»
 : «FalseBranch»

```

这是使用条件类型的第一个例子：

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

#### 34.1.1 条件类型的链式调用

类似于 JavaScript 的三元运算符，我们也可以链式调用 TypeScript 的条件类型运算符：

```ts
type PrimitiveTypeName<T> =
 T extends undefined ? 'undefined' :
 T extends null ? 'null' :
 T extends boolean ? 'boolean' :
 T extends number ? 'number' :
 T extends bigint ? 'bigint' :
 T extends string ? 'string' :
 never;

type _ = [
 Assert<Equal<
 PrimitiveTypeName<123n>,
 'bigint'
 >>,
 Assert<Equal<
 PrimitiveTypeName<bigint>,
 'bigint'
 >>,
];

```

#### 34.1.2 条件类型的嵌套

在上一个示例中，真分支总是很短，而假分支包含下一个（嵌套）条件类型。这就是为什么每个条件类型都有相同的缩进。

然而，如果嵌套条件类型出现在真分支中，那么缩进有助于人类阅读代码 – 例如：

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
 ["a", "b"]
>>;

```

关于此代码的更多信息，请参阅“过滤元组”（§37.6.4.2） – 此示例由此而来。

#### 34.1.3 示例：仅包装具有属性 `.length` 的类型

在以下示例中，`Wrap<>` 仅在具有 `.length` 属性且其值为数字的 Promises 中包装类型：

```ts
type WrapLen<T> = T extends { length: number } ? Promise<T> : T;
type _ = [
 Assert<Equal<
 WrapLen<string>,
 Promise<string>
 >>,
 Assert<Equal<
 WrapLen<boolean>,
 boolean
 >>,
];

```

### 34.2 条件类型在联合类型上是分配的

条件类型在联合类型上是[*分配的*](https://en.wikipedia.org/wiki/Distributive_property)：将条件类型 `C` 应用到联合类型 `U` 与将 `C` 应用到 `U` 的每个元素的联合相同。这是一个例子：

```ts
type WrapLen<T> = T extends { length: number } ? Promise<T> : T;
type _1 = Assert<Equal<
 WrapLen<'hello' | boolean | Array<number>>, // (A)
 Promise<'hello'> | boolean | Promise<Array<number>>
>>;

```

分配律使我们能够在行 A 中“循环”遍历联合类型的元素：`WrapLen<T>` 被应用于每个元素，并且仅包装具有 `.length` 属性且其值为数字的值。

为了比较，这是非联合类型发生的情况：

```ts
type _2 = [
 Assert<Equal<
 WrapLen<string>,
 Promise<string>
 >>,
 Assert<Equal<
 WrapLen<boolean>,
 boolean
 >>,
];

```

#### 34.2.1 只有 `extends` 的左侧是分配的

我们已经看到条件类型是分配在 `extends` 的左侧的：

```ts
type Left<T> = T extends any ? Promise<T> : never;
type _ = Assert<Equal<
 Left<'a'|'b'>,
 Promise<'a'> | Promise<'b'>
>>;

```

那么右侧呢？在那里没有发生分配：

```ts
type Right<T> = any extends T ? Promise<T> : never;
type _ = Assert<Equal<
 Right<'a'|'b'>,
 Promise<'a' | 'b'>
>>;

```

#### 34.2.2 只有类型变量触发分配

如果我们在条件类型的条件中直接提及联合类型，则不会发生分配：

```ts
type IsTrue = false | true extends true ? 'yes' : 'no';
type _ = Assert<Equal<
 IsTrue,
 'no'
>>;

```

将其与使用类型变量 `T` 进行比较：

```ts
type IsTrue<T> = T extends true ? 'yes' : 'no';
type _ = Assert<Equal<
 IsTrue<false | true>,
 'no' | 'yes'
>>;

```

#### 34.2.3 防止分配

考虑以下泛型类型：

```ts
type IsString<T> = T extends string ? 'yes' : 'no';
type _ = [
 Assert<Equal<
 IsString<string>, 'yes'
 >>,
 Assert<Equal<
 IsString<number>, 'no'
 >>,
 Assert<Equal<
 IsString<string | number>, 'yes' | 'no' // (A)
 >>,
];

```

在行 A 中，我们可以看到 `IsString` 是分配的 – 这是有意义的，因为我们已经使用条件类型来定义它。但这种情况并不是我们想要的：我们希望 `IsString` 告诉我们完整的类型 `string|number` *不是* 可分配给 `string`。这就是我们如何防止分配的方法：

```ts
type IsString<T> = [T] extends [string] ? 'yes' : 'no';
type _ = [
 Assert<Equal<
 IsString<string>, 'yes'
 >>,
 Assert<Equal<
 IsString<number>, 'no'
 >>,
 Assert<Equal<
 IsString<string | number>, 'no'
 >>,
];

```

条件类型仅在 `extends` 的左侧是一个裸类型变量时才是分配的。通过包装 `extends` 的左右两侧，预期的检查仍然发生，但没有分配。

#### 34.2.4 技巧：始终应用

条件类型是处理联合类型的重要工具，因为它们使我们能够遍历它们。有时，我们只想无条件地将每个联合元素映射到新类型。然后我们可以使用以下技巧：

```ts
type AlwaysWrap<T> = T extends any ? [T] : never;
type _ = Assert<Equal<
 AlwaysWrap<boolean>,
 [false] | [true]
>>;

```

注意类型 `boolean` 实际上只是联合 `false | true`。

以下（看似更简单）的方法**不**适用——`T`需要成为条件的一部分。否则，条件类型不是分配的。

```ts
type AlwaysWrap<T> = true extends true ? [T] : never;
type _ = Assert<Equal<
 AlwaysWrap<boolean>,
 [boolean]
>>;

```

### 34.3   通过条件返回`never`来过滤联合类型

将类型解释为集合，类型`never`是空的。因此，如果它出现在联合类型中，它将被忽略：

```ts
type _ = Assert<Equal<
 'a' | 'b' | never,
 'a' | 'b'
>>;

```

这意味着我们可以使用`never`来忽略联合类型的组件：

```ts
type DropNumber<T> = T extends number ? never : T;
type _ = Assert<Equal<
 DropNumber<1 | 'a' | 2 | 'b'>,
 'a' | 'b'
>>;

```

如果我们交换真分支和假分支的类型表达式，就会发生这种情况：

```ts
type KeepNumber<T> = T extends number ? T : never;

type _ = Assert<Equal<
 KeepNumber<1 | 'a' | 2 | 'b'>,
 1 | 2
>>;

```

#### 34.3.1   内置实用类型`Exclude<T, U>`

从联合类型中排除类型是一个如此常见的操作，以至于 TypeScript 提供了内置实用类型`Exclude<T, U>`：

```ts
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;

type Union = 1 | 'a' | 2 | 'b';
type _ = [
 Assert<Equal<
 Exclude<Union, number>,
 'a' | 'b'
 >>,
 Assert<Equal<
 Exclude<Union, 1 | 'a' | 'x'>,
 2 | 'b'
 >>,
];

```

将`Exclude<T, U>`解释为集合操作，它是`T − U`。

要查看`Exclude`的有趣用例，请参阅“提取区分联合的子类型”（§19.2.3）。

#### 34.3.2   内置实用类型`Extract<T, U>`

`Exclude<T, U>`的逆是`Extract<T, U>`（这也是 TypeScript 内置的）：

```ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;

type Union = 1 | 'a' | 2 | 'b';
type _ = [
 Assert<Equal<
 Extract<Union, number>,
 1 | 2
 >>,
 Assert<Equal<
 Extract<Union, 1 | 'a' | 'x'>,
 1 | 'a'
 >>,
];

```

将`Extract<T, U>`解释为集合操作，它是`T ∩ U`。

### 34.4   在条件类型中通过`infer`提取复合类型的部分

`infer`让我们提取复合类型的部分，并且只能在条件类型的`extends`子句中使用：

```ts
type ElemType<Arr> = Arr extends Array<infer Elem> ? Elem : never;
type _ = Assert<Equal<
 ElemType<Array<string>>, string
>>;

```

更多信息，请参阅“通过`infer`提取复合类型的部分”（§35）。

### 34.5   为条件类型编写条件

#### 34.5.1   一个类型是否可赋值给另一个类型？

我们可以使用条件类型来实现一个可赋值性检查：

```ts
type IsAssignableFrom<A, B> = B extends A ? true : false;
type _ = [
 Assert<Equal<
 // Type `123` is assignable to type `number`
 IsAssignableFrom<number, 123>,
 true
 >>,
 Assert<Equal<
 // Type `'abc'` is not assignable to type `number`
 IsAssignableFrom<number, 'abc'>,
 false
 >>,
];

```

为什么这是正确的？回想一下，在条件类型的条件中，`A extends B`检查`A`是否可赋值给`B`。

#### 34.5.2   检查泛型类型是否返回特定值

在以下代码中，我们检查`Str`是否等于`Uppercase<Str>`：

```ts
type IsUppercase<Str extends string> = Str extends Uppercase<Str>
 ? true
 : false;

type _ = [
 Assert<Equal<
 IsUppercase<'SUNSHINE'>, true
 >>,
 Assert<Equal<
 IsUppercase<'SUNSHINe'>, false
 >>,
];

```

我们实际上并没有检查相等性，我们只是检查`Str`是否可赋值给`Uppercase<Str>`——通过`extends`。

有一个需要注意的事项——`never`可赋值给所有类型：

```ts
type _ = [
 Assert<Equal<
 never extends true ? 'yes' : 'no',
 'yes'
 >>,
 Assert<Equal<
 never extends false ? 'yes' : 'no',
 'yes'
 >>,
];

```

更多关于`Uppercase`的信息，请参阅“字符串操作的实用类型”（§38.3）。

#### 34.5.3   检查两个类型是否相等

在下一个示例中，我们使用来自`asserttt`的泛型实用类型`Equal`来检查两个类型是否相等（行 A）：

```ts
type SimplifyTuple<Tup> =
 Tup extends [infer A, infer B]
 ? (Equal<A, B> extends true ? [A] : [A, B]) // (A)
 : never
;
type _ = [
 Assert<Equal<
 SimplifyTuple<['a', 'b']>,
 ['a', 'b']
 >>,
 Assert<Equal<
 SimplifyTuple<['a', 'a']>,
 ['a']
 >>,
];

```

注意，我们检查`Equal<A, B>`的结果是否可赋值给`true`。

#### 34.5.4   逻辑或

要检查`X || Y`，我们检查：

```ts
X ? true
: Y ? true
: false

```

示例：

```ts
type ContainsNumber<Tup> =
 Tup extends [infer A, infer B]
 ? A extends number ? true
 : B extends number ? true
 : false
 : never
;
type _ = [
 Assert<Equal<
 ContainsNumber<['a', 'b']>,
 false
 >>,
 Assert<Equal<
 ContainsNumber<[1, 'a']>,
 true
 >>,
 Assert<Equal<
 ContainsNumber<[number, 'a']>,
 true
 >>,
];

```

#### 34.5.5   逻辑与

要检查`X && Y`，我们可以使用一个技巧，并通过`extends`检查`[X, Y]`——例如：

```ts
export type TEqual<X, Y> =
 [IsAny<X>, IsAny<Y>] extends [true, true] ? true
 : [IsAny<X>, IsAny<Y>] extends [false, false] ? MutuallyAssignable<X, Y>
 : false

```

关于这段代码的更多信息，请参阅“确保`any`只等于自身”（§39.3.3）。

### 34.6 延迟的条件类型

总结一下，让我们看看一个有趣的现象：通常，条件类型的结果是它的真分支或假分支。然而，如果它的条件包含一个或多个尚未有值的类型变量，那么它就是*延迟的*，并不会转换成一个更简单的值——例如：

```ts
type StringOrNumber<Kind extends 'string' | 'number'> =
 Kind extends 'string' ? string : number
;

function randomValue<K extends 'string' | 'number'>(kind: K) {
 type Result = StringOrNumber<K>;
 type _ = Assert<Equal<
 Result,
 K extends 'string' ? string : number // (A)
 >>;
 // ···
}

```

在行 A 中，我们可以看到`Result`既不是`string`也不是`number`，而是一个延迟的条件类型。

### 34.7 本章节的来源

+   TypeScript 手册中的[“条件类型”](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)章节

+   TypeScript 手册（版本 1）中的[“条件类型”](https://www.typescriptlang.org/docs/handbook/advanced-types.html#conditional-types)章节

    +   描述了延迟的条件类型。
