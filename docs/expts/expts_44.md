# 36   映射类型 {[K in U]: X}

> 原文：[`exploringjs.com/ts/book/ch_mapped-types.html`](https://exploringjs.com/ts/book/ch_mapped-types.html)

（广告，请勿拦截。）

1.  36.1 基本映射类型

    1.  36.1.1 示例：有限键集

    1.  36.1.2 索引签名与映射类型

    1.  36.1.3 通过映射类型转换对象类型

    1.  36.1.4 映射保留类型的种类（元组、数组、对象等）

    1.  36.1.5 示例：使接口异步

    1.  36.1.6 示例：向枚举对象添加键

1.  36.2 通过键重映射（`as`）更改属性键

    1.  36.2.1 如果我们使用键重映射，结果总是一个对象字面量类型

1.  36.3 过滤属性

    1.  36.3.1 通过键重映射（`as`）过滤属性

    1.  36.3.2 通过过滤键联合过滤属性

    1.  36.3.3 用于保留属性的内置实用类型：`Pick<T, KeysToKeep>`

    1.  36.3.4 用于过滤属性的内建实用类型：`Omit<T, KeysToFilterOut>`

1.  36.4 通过映射类型添加和移除修饰符

    1.  36.4.1 示例：添加可选修饰符（`?`）

    1.  36.4.2 示例：移除可选修饰符（`?`）

    1.  36.4.3 示例：添加 `readonly` 修饰符

    1.  36.4.4 示例：移除 `readonly` 修饰符

1.  36.5 检测属性修饰符 `readonly` 和 `?`（可选）

    1.  36.5.1 检测属性是否为只读的

    1.  36.5.2 检测属性是否为可选的

1.  36.6 `Record` 是一个映射类型

1.  36.7 本章来源

使用映射类型最常见的方式是通过遍历其键来生成输入类型（通常是对象类型或元组类型）的新版本。

### 36.1 基本映射类型

一个基本的映射类型看起来像这样（名称 `Key` 只是一个示例；我们可以使用任何标识符）：

```ts
{
 [Key in «KeySet»]: «PropValue»
}

```

映射类型创建一个对象类型。它遍历 `«KeySet»` 的元素，并为每次迭代创建一个属性：

+   `Key` 是 `«KeySet»` 的当前元素。它决定了属性键。（有方法可以使用不同的键或跳过属性。我们将在稍后探讨这些方法。）

+   冒号后面的表达式确定属性值，并可以使用 `Key`。

#### 36.1.1 示例：有限键集合

以下映射类型遍历一组有限的键：

```ts
type Obj = {
 [K in 'a' | 'b' | 'c']: number
};
type _ = Assert<Equal<
 Obj,
 {
 a: number,
 b: number,
 c: number,
 }
>>;

```

#### 36.1.2 索引签名与映射类型

要了解索引签名与映射类型的不同，让我们首先回顾一下索引签名的定义。

##### 36.1.2.1 索引签名表示一个可能无限多的属性集合

这是一个索引签名的例子：

```ts
type StrToNum = {
 [key: string]: number, // index signature
};

```

索引签名表示一个可能无限多的属性集合。在这种情况下：

+   *绑定标识符* `key` 被忽略，可以是任何标识符。它区分了索引签名与 具有计算键的属性。

+   每个属性都有一个 `string` 类型的键。属性类型必须是无限的 – 例如：`string`、`number`、`symbol`、一个具有无限原始类型的模板字符串字面量（例如 `` `${bigint}` ``）

+   每个属性都有一个 `number` 类型的值。

换句话说：`StrToNum` 是一种对象类型，用作从字符串到数字的字典 – 例如：

```ts
const obj1: StrToNum = {};
const obj2: StrToNum = { a: 1 };
const obj3: StrToNum = { a: 1, b: 2, hello: 3 };

```

更多信息：“索引签名：对象作为字典”（§18.7）。

##### 36.1.2.2 映射类型是一个类型级别的函数

相比之下，映射类型是一个类型级别的函数。它将属性键映射到对象字面量类型：

```ts
type _ = [
 Assert<Equal<
 // Type-level function applied to a finite type
 {
 [K in 'a' | 'b' | 'c']: number
 },
 // Result: object literal type with normal properties
 {
 a: number,
 b: number,
 c: number,
 }
 >>,
 Assert<Equal<
 // Type-level function applied to an infinite type
 {
 [K in string]: number
 },
 // Result: object literal type with index signature
 {
 [x: string]: number
 }
 >>,
];

```

注意一个重要的区别：

+   `[K in string]`：`K` 是一个可以在冒号之后使用的类型变量。

+   `[x: string]`：`x` 不提供任何功能（它只作为语法标记），但它类似于一个正常变量。

#### 36.1.3 通过映射类型转换对象类型

映射类型最常见的用例是转换对象类型：

```ts
type Arrayify<Obj> = {
 [K in keyof Obj]: Array<Obj[K]> // (A)
};

type InputObj = {
 str: string,
 num: number,
};
type _ = Assert<Equal<
 Arrayify<InputObj>,
 {
 str: Array<string>,
 num: Array<number>,
 }
>>;

```

在行 A 中，我们使用了 `keyof Obj` 来计算 `Obj` 的键并遍历它们。我们使用了 索引访问类型 `Obj[K]` 和泛型类型 `Array` 来定义属性值。

#### 36.1.4 映射保留类型种类（元组、数组、对象等）

映射类型的输入（元组、数组、对象等）决定了输出看起来像什么：

```ts
type WrapValues<T> = {
 [Key in keyof T]: Promise<T[Key]>
};

type _ = [
 // Read-only tuple in, read-only tuple out
 Assert<Equal<
 WrapValues<readonly ['a', 'b']>,
 readonly [Promise<'a'>, Promise<'b'>]
 >>,

 // Tuple labels are preserved
 Assert<Equal<
 WrapValues<[labelA: 'a', labelB: 'b']>,
 [labelA: Promise<'a'>, labelB: Promise<'b'>]
 >>,

 // Array in, Array out
 Assert<Equal<
 WrapValues<Array<string>>,
 Array<Promise<string>>
 >>,

 // ReadonlyArray in, ReadonlyArray out
 Assert<Equal<
 WrapValues<ReadonlyArray<string>>,
 ReadonlyArray<Promise<string>>
 >>,

 // Object in, object out
 Assert<Equal<
 WrapValues<{ a: 1, b: 2 }>,
 { a: Promise<1>, b: Promise<2> }
 >>,

 // Read-only properties are preserved
 Assert<Equal<
 WrapValues<{ readonly a: 1, readonly b: 2 }>,
 { readonly a: Promise<1>, readonly b: Promise<2> }
 >>,
];

```

#### 36.1.5 示例：将接口转换为异步

泛型类型 `Asyncify<Intf>` 将同步接口 `Intf` 转换为异步接口：

```ts
interface SyncService {
 factorize(num: number): Array<number>;
 createDigest(text: string): string;
}
type AsyncService = Asyncify<SyncService>;
type _ = Assert<Equal<
 AsyncService,
 {
 factorize: (num: number) => Promise<Array<number>>,
 createDigest: (text: string) => Promise<string>,
 }
>>;

```

这是 `Asyncify` 的定义：

```ts
type Asyncify<Intf> = {
 [K in keyof Intf]: // (A)
 Intf[K] extends (...args: infer A) => infer R // (B)
 ? (...args: A) => Promise<R> // (C)
 : Intf[K] // (D)
};

```

+   我们使用映射类型来遍历 `Intf` 的属性（行 A）。

+   对于具有键 `K` 的每个属性，我们检查属性值 `Intf[K]` 是否是函数或方法（行 B）。

    +   如果是，我们使用 `infer` 关键字 从类型变量 `A` 中提取参数，并将返回类型提取到类型变量 `R` 中。我们使用这些变量创建一个新的属性值，其中返回类型 `R` 被包裹在一个 Promise 中（行 C）。

    +   如果没有，则属性值不会改变（行 D）。

#### 36.1.6 示例：向枚举对象添加键

考虑以下枚举对象：

```ts
const tokenDefs = {
 number: {
 key: 'number',
 re: /[0-9]+/,
 description: 'integer number',
 },
 identifier: {
 key: 'identifier',
 re: /[a-z]+/,
 },
} as const;

```

我们希望避免重复提及 `.key`。这是通过添加一个名为 `addKey()` 的函数来添加它们的示例：

```ts
const tokenDefs = addKeys({
 number: {
 re: /[0-9]+/,
 description: 'integer number',
 },
 identifier: {
 re: /[a-z]+/,
 },
} as const);

assert.deepEqual(
 tokenDefs,
 {
 number: {
 key: 'number',
 re: /[0-9]+/,
 description: 'integer number',
 },
 identifier: {
 key: 'identifier',
 re: /[a-z]+/,
 },
 }
);

assertType<
 {
 readonly number: {
 readonly re: RegExp,
 readonly description: 'integer number',
 key: string,
 },
 readonly identifier: {
 readonly re: RegExp,
 key: string,
 },
 }
>(tokenDefs);

```

`addKeys()` 不丢失类型信息非常有用：`tokenDefs` 的计算类型正确记录了属性 `.description` 存在的位置和不存在的位置：TypeScript 允许我们使用 `tokenDefs.number.description`（存在）但不能使用 `tokenDefs.identifier.description`（不存在）。

这是 `addKeys()` 的一个实现：

```ts
function addKeys<
 T extends Record<string, InputTokenDef>
>(tokenDefs: T)
: {[K in keyof T]: T[K] & {key: string}} // (A)
{
 const entries = Object.entries(tokenDefs);
 const pairs = entries.map(
 ([key, def]) => [key, {key, ...def}]
 );
 return Object.fromEntries(pairs);
}

// Information we have to provide
interface InputTokenDef {
 re: RegExp,
 description?: string,
}

// Information addKeys() adds for us
interface TokenDef extends InputTokenDef {
 key: string,
}

```

在行 A 中，我们使用 `&` 创建一个交集类型，它具有 `T[K]` 和 `{key: string}` 的属性。

### 36.2 通过键重映射更改属性键（`as`）

在映射类型的键部分，我们可以使用 `as` 来更改当前属性的属性键：

```ts
{ [P in K as N]: X }

```

在以下示例中，我们使用 `as` 在每个属性名前添加一个下划线：

```ts
type Point = {
 x: number,
 y: number,
};
type PrefixUnderscore<Obj> = {
 [K in keyof Obj & string as `_${K}`]: Obj[K] // (A)
};
type X = PrefixUnderscore<Point>;
type _ = Assert<Equal<
 PrefixUnderscore<Point>,
 {
 _x: number,
 _y: number,
 }
>>;

```

在行 A 中，如果 `K` 是一个符号，则模板字面量类型 `` `_${K}` `` 不会工作。这就是为什么我们将 `keyof Obj` 与 `string` 交集，并且只遍历 `Obj` 中是字符串的键。

#### 36.2.1 如果我们使用键重映射，结果始终是一个对象字面量类型

我们之前看到，将简单的映射类型应用于元组会产生一个元组。如果我们进行键重映射，这就会改变。然后结果始终是一个对象字面量类型 - 例如：

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

这个结果反映了元组的实际键。简单的映射类型隐式地过滤了这些键。更多信息，请参阅“通过映射类型映射元组”（§37.4）。

### 36.3 过滤属性

到目前为止，我们只改变了对象类型的属性键或值。在本节中，我们来看一下过滤属性。

#### 36.3.1 通过键重映射过滤属性（`as`）

过滤的最简单方法是使用 `as`：如果我们使用 `never` 作为属性键，则该属性将从结果中省略。

在以下示例中，我们移除了所有值不是字符串的属性：

```ts
type KeepStrProps<Obj> = {
 [
 Key in keyof Obj
 as Obj[Key] extends string ? Key : never
 ]: Obj[Key]
};

type Obj = {
 strPropA: 'A',
 strPropB: 'B',
 numProp1: 1,
 numProp2: 2,
};
type _ = Assert<Equal<
 KeepStrProps<Obj>,
 {
 strPropA: 'A',
 strPropB: 'B',
 }
>>;

```

#### 36.3.2 通过过滤键联合过滤属性

在 TypeScript 通过 `as` 进行键重映射之前，我们必须在迭代之前过滤具有属性键的联合。

让我们重新做之前的例子，但不使用 `as`：我们只想保留类型为 `Obj` 且值为字符串的属性。

```ts
type Obj = {
 strPropA: 'A',
 strPropB: 'B',
 numProp1: 1,
 numProp2: 2,
};

```

以下泛型辅助类型收集所有属性值的键，其值是字符串：

```ts
type KeysOfStrProps<T> = {
 [K in keyof T]: T[K] extends string ? K : never // (A)
}[keyof T]; // (B)

type _1 = Assert<Equal<
 KeysOfStrProps<Obj>,
 'strPropA' | 'strPropB'
>>;

```

我们分两步计算结果：

+   行 A：首先，我们创建一个对象，其中每个属性键 `K` 映射到：

    +   `K` – 如果属性值 `T[K]` 是一个字符串

    +   `never` – 否则

+   行 B：然后我们提取我们刚刚创建的对象的所有属性值。当我们这样做时，类型 `never` 消失了。

使用 `KeysOfStrProps`，现在很容易实现 `KeepStrProps` 而不需要 `as`：

```ts
type KeepStrProps<Obj> = {
 [Key in KeysOfStrProps<Obj>]: Obj[Key]
};
type _2 = Assert<Equal<
 KeepStrProps<Obj>,
 {
 strPropA: 'A',
 strPropB: 'B',
 }
>>;

```

#### 36.3.3 用于保留属性的内置实用类型：`Pick<T, KeysToKeep>`

以下 [内置实用类型](https://github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts) 允许我们通过指定要保留的现有对象类型的属性来创建一个新的对象：

```ts
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
 [P in K]: T[P];
};

```

我们通过迭代 `T` 的属性键的子集 `K` (`keyof T`) 来保留 `T` 的属性子集。

`Pick` 的用法如下：

```ts
type ObjectLiteralType = {
 eeny: 1,
 meeny: 2,
 miny: 3,
 moe: 4,
};

type _ = Assert<Equal<
 Pick<ObjectLiteralType, 'eeny' | 'miny'>,
 { eeny: 1, miny: 3 }
>>;

```

##### 36.3.3.1 通过 `Pick<>` 类型化函数

我们可以在 JavaScript 级别实现属性选择（如 [Underscore 库](https://underscorejs.org/#pick) 所提供的）。然后实用类型 `Pick<>` 帮助我们确定返回类型：

```ts
function pick<
 O extends Record<string, unknown>,
 K extends keyof O
>(
 object: O,
 ...keys: Array<K>
): Pick<O, K> {
 return Object.fromEntries(
 Object.entries(object)
 .filter(
 ([key, _value]) => keys.includes(key as K)
 )
 ) as any;
}

const address = {
 street: 'Evergreen Terrace',
 number: '742',
 city: 'Springfield',
 state: 'NT',
 zip: '49007',
};
const result = pick(address, 'street', 'number');

// Correct value?
assert.deepEqual(
 result,
 {
 street: 'Evergreen Terrace',
 number: '742',
 }
);

// Correct type?
assertType<
 {
 street: string,
 number: string,
 }
>(result);

```

##### 36.3.3.2 示例：移除 `Pick<T, K>` 参数 `K` 的约束

正如我们所见，`Pick<T, K>` 的参数 `K` 被限制为 `T` 的键。这阻止了一些有用的应用 – 例如：

```ts
type Obj = {
 '0': 'a',
 '1': 'b',
 length: 2,
};
// @ts-expect-error: Type '`${number}`' does not satisfy the constraint
// 'keyof Obj'.
type _1 = Pick<Obj, `${number}`>

```

`` `${number}` `` 是所有字符串化数字的类型（参见“将原始类型插入到模板字面量中”（§38.2.5））。我们希望提取所有键是该类型元素的属性。唉，我们无法使用 `Pick` 来实现这一点。这是一个参数 `K` 没有约束的 `Pick` 版本：

```ts
type PickFreely<T, K> = {
 [P in K & keyof T]: T[P];
};

```

注意，操作 `T[P]` 仅在 `P` 是 `T` 的键时才有效。因此，`in` 后的集合必须是 `keyof T` 的子集。这就是为什么我们使用 `K & keyof T` 而不是 `K` 的原因。

使用 `PickFreely`，我们可以提取属性：

```ts
type _2 = Assert<Equal<
 PickFreely<Obj, `${number}`>,
 {
 '0': 'a',
 '1': 'b',
 }
>>;

```

#### 36.3.4 用于过滤属性的内置实用类型：`Omit<T, KeysToFilterOut>`

以下 [内置实用类型](https://github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts) 允许我们通过指定要省略的现有对象类型的属性来创建一个新的对象类型：

```ts
/**
 * Construct a type with the properties of T except for those in
 * type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

```

说明：

+   `K extends keyof any` 表示 `K` 必须是所有可能的属性键的子集：

    ```ts
    type _ = Assert<Equal<
     keyof any,
     string | number | symbol
    >>;

    ```

+   `Exclude<keyof T, K>>` 表示：从 `T` 的键中移除 `K` 中提到的所有“值”。

`Omit<>` 的用法如下：

```ts
type ObjectLiteralType = {
 eeny: 1,
 meeny: 2,
 miny: 3,
 moe: 4,
};

type _ = Assert<Equal<
 Omit<ObjectLiteralType, 'eeny' | 'miny'>,
 { meeny: 2; moe: 4; }
>>;

```

### 36.4 通过映射类型添加和删除修饰符

在 TypeScript 中，属性可以有几种 *修饰符*：

+   属性可以是可选的：修饰符 `?`

+   属性可以是只读的：修饰符 `readonly`

我们可以通过映射类型添加或删除这些修饰符。

#### 36.4.1 示例：添加可选修饰符（`?`）

```ts
type AddOptional<T> = {
 [K in keyof T]+?: T[K]
};
type RequiredArticle = {
 title: string,
 tags: Array<string>,
 score: number,
};
type OptionalArticle = AddOptional<RequiredArticle>;
type _ = Assert<Equal<
 OptionalArticle,
 {
 title?: string | undefined;
 tags?: Array<string> | undefined;
 score?: number | undefined;
 }
>>;

```

符号 `+?` 表示：将当前属性设置为可选。我们可以省略 `+`，但我发现如果保留它，理解起来会更简单。

内置工具类型 `Partial<T>`（[`github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts`](https://github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts)）与我们上面的泛型类型 `AddOptional` 等价。

#### 36.4.2 示例：移除可选修饰符（`?`）

```ts
type RemoveOptional<T> = {
 [K in keyof T]-?: T[K]
};
type OptionalArticle = {
 title?: string,
 tags?: Array<string>,
 score: number,
};
type RequiredArticle = RemoveOptional<OptionalArticle>;
type _ = Assert<Equal<
 RequiredArticle,
 {
 title: string,
 tags: Array<string>,
 score: number,
 }
>>;

```

符号 `-?` 表示：将当前属性设置为必需（非可选）。

内置工具类型 `Required<T>`（[`github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts`](https://github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts)）与我们上面的泛型类型 `RemoveOptional` 等价。

#### 36.4.3 示例：添加 `readonly` 修饰符

```ts
type AddReadonly<Obj> = {
 +readonly [K in keyof Obj]: Obj[K]
};
type MutableArticle = {
 title: string,
 tags: Array<string>,
 score: number,
};
type ImmutableArticle = AddReadonly<MutableArticle>;
type _ = Assert<Equal<
 ImmutableArticle,
 {
 readonly title: string,
 readonly tags: Array<string>,
 readonly score: number,
 }
>>;

```

符号 `+readonly` 表示：将当前属性设置为只读。我们可以省略 `+`，但我发现如果保留它，理解起来会更简单。

内置工具类型 `Readonly<T>`（[`github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts`](https://github.com/Microsoft/TypeScript/blob/main/src/lib/es5.d.ts)）与我们上面的泛型类型 `AddReadonly` 等价。

#### 36.4.4 示例：移除 `readonly` 修饰符

```ts
type RemoveReadonly<Obj> = {
 -readonly [K in keyof Obj]: Obj[K]
};
type ImmutableArticle = {
 readonly title: string,
 readonly tags: Array<string>,
 score: number,
};
type MutableArticle = RemoveReadonly<ImmutableArticle>;
type _ = Assert<Equal<
 MutableArticle,
 {
 title: string,
 tags: Array<string>,
 score: number,
 }
>>;

```

符号 `-readonly` 表示：将当前属性设置为可变（非只读）。

没有内置的工具类型可以移除 `readonly` 修饰符。

### 36.5 检测属性修饰符 `readonly` 和 `?`（可选）

#### 36.5.1 检测属性是否为只读

这就是使用工具类型 `IsReadonly` 的样子：

```ts
interface Car {
 readonly year: number,
 get maker(): string, // technically `readonly`
 owner: string,
}

type _1 = [
 Assert<Equal<
 IsReadonly<Car, 'year'>, true
 >>,
 Assert<Equal<
 IsReadonly<Car, 'maker'>, true
 >>,
 Assert<Equal<
 IsReadonly<Car, 'owner'>, false
 >>,
];

```

悲剧的是，实现 `IsReadonly` 是复杂的：`readonly` 目前不会影响可赋性，并且不能通过 `extends` 来检测：

```ts
type SimpleEqual<T1, T2> =
 [T1] extends [T2]
 ? [T2] extends [T1] ? true : false
 : false
;
type _2 = Assert<Equal<
 SimpleEqual<
 {readonly year: number},
 {year: number}
 >,
 true
>>;

```

`T1` 和 `T2` 两边的括号是为了防止分配律。

这意味着我们需要进行更严格的相等性检查：

```ts
type StrictEqual<X, Y> =
 (<T>() => T extends X ? 1 : 2) extends
 (<T>() => T extends Y ? 1 : 2) ? true : false
;
type _3 = [
 Assert<Equal<
 StrictEqual<
 {readonly year: number},
 {year: number}
 >,
 false
 >>,
 Assert<Equal<
 StrictEqual<
 {year: number},
 {year: number}
 >,
 true
 >>,
 Assert<Equal<
 StrictEqual<
 {readonly year: number},
 {readonly year: number}
 >,
 true
 >>,
];

```

辅助类型 `StrictEqual` 是一种技巧，但目前是严格比较类型的最佳技术。其工作原理在“如何检查两个类型是否相等？”（§39.3）中解释。

现在我们可以实现 `IsReadonly`（基于 GitHub 用户 `inad9300` 的[代码](https://github.com/microsoft/TypeScript/issues/31581)）：

```ts
type IsReadonly<T, K extends keyof T> =
 StrictEqual<
 Pick<T, K>, // (A)
 Readonly<Pick<T, K>> // (B)
 >
;

```

我们比较两个对象：

+   只包含 `T` 的属性 `K` 的对象（行 A）

+   同一个对象，将所有属性设置为 `readonly`（行 B）

如果两个对象相等，那么将属性 `K` 设置为 `readonly` 并没有改变任何事情——这意味着它已经是 `readonly` 的。

相关 GitHub 问题：[“允许在映射类型中识别只读属性”](https://github.com/microsoft/TypeScript/issues/31581)

#### 36.5.2 检测属性是否为可选

这就是使用辅助类型 `IsOptional` 来检测属性是否可选的样子：

```ts
interface Person {
 name: undefined | string;
 age?: number;
}

type _1 = [
 Assert<Equal<
 IsOptional<Person, 'name'>, false
 >>,
 Assert<Equal<
 IsOptional<Person, 'age'>, true
 >>,
];

```

`IsOptional` 比起 `IsReadonly` 更容易实现，因为可选属性更容易检测：

```ts
type IsOptional<T extends Record<any, any>, K extends keyof T> =
 {} extends Pick<T, K> ? true : false
;

```

它是如何工作的？让我们看看 `Pick` 产生的结果：

```ts
type _2 = [
 Assert<Equal<
 Pick<Person, 'name'>,
 { name: undefined | string }
 >>,
 Assert<Equal<
 Pick<Person, 'age'>,
 { age?: number | undefined }
 >>,
];

```

只有后者对象可以赋值给空对象 `{}`。

### 36.6 `Record` 是一个映射类型

内置的实用类型 `Record` 简单地是一个映射类型的别名：

```ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
 [P in K]: T;
};

```

再次强调，`keyof any` 表示“有效的属性键”：

```ts
type _ = Assert<Equal<
 keyof any,
 string | number | symbol
>>;

```

这些是由 `Record` 产生的结果：

```ts
type _ = [
 Assert<Equal<
 // Finite key type
 Record<'a' | 'b', RegExp>,
 {
 a: RegExp,
 b: RegExp,
 }
 >>,
 Assert<Equal<
 // Infinite key type
 Record<string, boolean>,
 {
 [x: string]: boolean // index signature
 }
 >>,
];

```

### 36.7 本章来源

+   官方 TypeScript 手册中的 [“映射类型”](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) 章节内容。
