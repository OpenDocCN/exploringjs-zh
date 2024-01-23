# 二十三、类型计算概述

> 原文：[`exploringjs.com/tackling-ts/ch_computing-with-types-overview.html`](https://exploringjs.com/tackling-ts/ch_computing-with-types-overview.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   23.1 类型作为元值

+   23.2 泛型类型：类型的工厂

+   23.3 联合类型和交集类型

    +   23.3.1 联合类型（`|`)

    +   23.3.2 交集类型（`&`)

+   23.4 控制流

    +   23.4.1 条件类型

    +   23.4.2 映射类型

+   23.5 其他各种运算符

    +   23.5.1 索引类型查询运算符 `keyof`

    +   23.5.2 [索引访问运算符 `T[K]`](ch_computing-with-types-overview.html#the-indexed-access-operator-tk)

    +   23.5.3 类型查询运算符 `typeof`

* * *

在本章中，我们将探讨如何在 TypeScript 的编译时计算类型。

请注意，本章的重点是学习如何计算类型。因此，我们会经常使用文字类型，示例的实际相关性较小。

### 23.1 类型作为元值

考虑以下两个级别的 TypeScript 代码：

+   程序级别：在运行时，我们可以使用值和函数。

+   类型级别：在编译时，我们可以使用特定类型和泛型类型。

类型级别是程序级别的元级别。

| 级别 | 可用于 | 操作数 | 操作 |
| --- | --- | --- | --- |
| 程序级别 | 运行时 | 值 | 函数 |
| 类型级别 | 编译时 | 特定类型 | 泛型类型 |

我们可以计算类型是什么意思？以下代码是一个例子：

```ts
type ObjectLiteralType = {
 first: 1,
 second: 2,
};

// %inferred-type: "first" | "second"
type Result = keyof ObjectLiteralType; // (A)
```

在 A 行，我们正在采取以下步骤：

+   我们计算的输入是类型 `ObjectLiteralType`，即对象字面类型。

+   我们对输入应用操作 `keyof`。它列出对象类型的属性键。

+   我们将 `keyof` 的输出命名为 `Result`。

在类型级别上，我们可以计算以下“值”：

```ts
type ObjectLiteralType = {
 prop1: string,
 prop2: number,
};

interface InterfaceType {
 prop1: string;
 prop2: number;
}

type TupleType = [boolean, bigint];

//::::: Nullish types and literal types :::::
// Same syntax as values, but they are all types!

type UndefinedType = undefined;
type NullType = null;

type BooleanLiteralType = true;
type NumberLiteralType = 12.34;
type BigIntLiteralType = 1234n;
type StringLiteralType = 'abc';
```

### 23.2 泛型类型：类型的工厂

泛型类型是元级别的函数 - 例如：

```ts
type Wrap<T> = [T];
```

泛型类型 `Wrap<>` 具有参数 `T`。其结果是 `T`，包装在一个元组类型中。这是我们如何使用这个元函数的方式：

```ts
// %inferred-type: [string]
type Wrapped = Wrap<string>;
```

我们将参数 `string` 传递给 `Wrap<>` 并将结果命名为 `Wrapped`。结果是一个具有单个组件的元组类型 - 类型 `string`。

### 23.3 联合类型和交集类型

#### 23.3.1 联合类型（`|`）

类型运算符 `|` 用于创建联合类型：

```ts
type A = 'a' | 'b' | 'c';
type B = 'b' | 'c' | 'd';

// %inferred-type: "a" | "b" | "c" | "d"
type Union = A | B;
```

如果我们将类型 `A` 和类型 `B` 视为集合，那么 `A | B` 就是这些集合的集合论并集。换句话说：结果的成员是至少是操作数之一的成员。

从语法上讲，我们也可以在联合类型的第一个组件前面加上 `|`。当类型定义跨越多行时，这是很方便的：

```ts
type A =
 | 'a'
 | 'b'
 | 'c'
;
```

##### 23.3.1.1 联合作为元值的集合

TypeScript 将元值的集合表示为文字类型的联合。我们已经看到了一个例子：

```ts
type Obj = {
 first: 1,
 second: 2,
};

// %inferred-type: "first" | "second"
type Result = keyof Obj;
```

我们很快将看到类型级别的操作，用于循环遍历这样的集合。

##### 23.3.1.2 对象类型的联合

由于联合类型的每个成员都是组件类型中至少一个成员，我们只能安全地访问所有组件类型共享的属性（行 A）。要访问任何其他属性，我们需要一个类型保护（行 B）：

```ts
type ObjectTypeA = {
 propA: bigint,
 sharedProp: string,
}
type ObjectTypeB = {
 propB: boolean,
 sharedProp: string,
}

type Union = ObjectTypeA | ObjectTypeB;

function func(arg: Union) {
 // string
 arg.sharedProp; // (A) OK
 // @ts-expect-error: Property 'propB' does not exist on type 'Union'.
 arg.propB; // error

 if ('propB' in arg) { // (B) type guard
 // ObjectTypeB
 arg;

 // boolean
 arg.propB;
 }
}
```

#### 23.3.2 交集类型（`&`）

类型运算符 `&` 用于创建交集类型：

```ts
type A = 'a' | 'b' | 'c';
type B = 'b' | 'c' | 'd';

// %inferred-type: "b" | "c"
type Intersection = A & B;
```

如果我们将类型`A`和类型`B`视为集合，则`A & B`是这些集合的集合论交集。换句话说：结果的成员是两个操作数的成员。

##### 23.3.2.1 对象类型的交集

两个对象类型的交集具有两种类型的属性：

```ts
type Obj1 = { prop1: boolean };
type Obj2 = { prop2: number };
type Both = {
 prop1: boolean,
 prop2: number,
};

// Type Obj1 & Obj2 is assignable to type Both
// %inferred-type: true
type IntersectionHasBothProperties = IsAssignableTo<Obj1 & Obj2, Both>;
```

（泛型类型 `IsAssignableTo<>` 将在后面解释。）

##### 23.3.2.2 使用交集类型进行混合

如果我们将对象类型 `Named` 混入另一个类型 `Obj` 中，那么我们需要一个交集类型（行 A）：

```ts
interface Named {
 name: string;
}
function addName<Obj extends object>(obj: Obj, name: string)
 : Obj & Named // (A)
{
 const namedObj = obj as (Obj & Named);
 namedObj.name = name;
 return namedObj;
}

const obj = {
 last: 'Doe',
};

// %inferred-type: { last: string; } & Named
const namedObj = addName(obj, 'Jane');
```

### 23.4 控制流

#### 23.4.1 条件类型

*条件类型* 有以下语法：

```ts
«Type2» extends «Type1» ? «ThenType» : «ElseType»
```

如果 `Type2` 可分配给 `Type1`，则此类型表达式的结果是 `ThenType`。否则，它是 `ElseType`。

##### 23.4.1.1 例子：只包装具有属性`.length`的类型

在以下示例中，`Wrap<>`仅在具有属性`.length`且其值为数字的一元元组中包装类型：

```ts
type Wrap<T> = T extends { length: number } ? [T] : T;

// %inferred-type: [string]
type A = Wrap<string>;

// %inferred-type: RegExp
type B = Wrap<RegExp>;
```

##### 23.4.1.2 例子：检查可分配性

我们可以使用条件类型来实现可分配性检查：

```ts
type IsAssignableTo<A, B> = A extends B ? true : false;

// Type `123` is assignable to type `number`
// %inferred-type: true
type Result1 = IsAssignableTo<123, number>;

// Type `number` is not assignable to type `123`
// %inferred-type: false
type Result2 = IsAssignableTo<number, 123>;
```

有关类型关系*可分配性*的更多信息，请参见[[content not included]](ch_missing-chapters-online.html)。

##### 23.4.1.3 条件类型是可分配的

条件类型是[*可分配的*](https://en.wikipedia.org/wiki/Distributive_property)：将条件类型 `C` 应用于联合类型 `U` 等同于将 `C` 应用于 `U` 的每个组件的并集。这是一个例子：

```ts
type Wrap<T> = T extends { length: number } ? [T] : T;

// %inferred-type: boolean | [string] | [number[]]
type C1 = Wrap<boolean | string | number[]>;

// Equivalent:
type C2 = Wrap<boolean> | Wrap<string> | Wrap<number[]>;
```

换句话说，可分配性使我们能够“循环”遍历联合类型的组件。

这是可分配性的另一个例子：

```ts
type AlwaysWrap<T> = T extends any ? [T] : [T];

// %inferred-type: ["a"] | ["d"] | [{ a: 1; } & { b: 2; }]
type Result = AlwaysWrap<'a' | ({ a: 1 } & { b: 2 }) | 'd'>;
```

##### 23.4.1.4 使用可分配条件类型，我们使用类型`never`来忽略事物

将`never`类型解释为集合时，它是空的。因此，如果它出现在联合类型中，它将被忽略：

```ts
// %inferred-type: "a" | "b"
type Result = 'a' | 'b' | never;
```

这意味着我们可以使用 `never` 来忽略联合类型的组件：

```ts
type DropNumbers<T> = T extends number ? never : T;

// %inferred-type: "a" | "b"
type Result1 = DropNumbers<1 | 'a' | 2 | 'b'>;
```

如果我们交换了 then-branch 和 else-branch 的类型表达式，会发生什么：

```ts
type KeepNumbers<T> = T extends number ? T : never;

// %inferred-type: 1 | 2
type Result2 = KeepNumbers<1 | 'a' | 2 | 'b'>;
```

##### 23.4.1.5 内置实用类型：`Exclude<T, U>`

从联合类型中排除类型是一种常见的操作，TypeScript 提供了内置实用类型`Exclude<T, U>`：

```ts
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;

// %inferred-type: "a" | "b"
type Result1 = Exclude<1 | 'a' | 2 | 'b', number>;

// %inferred-type: "a" | 2
type Result2 = Exclude<1 | 'a' | 2 | 'b', 1 | 'b' | 'c'>;
```

##### 23.4.1.6 内置实用类型：`Extract<T, U>`

`Exclude<T, U>` 的反操作是 `Extract<T, U>`：

```ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;

// %inferred-type: 1 | 2
type Result1 = Extract<1 | 'a' | 2 | 'b', number>;

// %inferred-type: 1 | "b"
type Result2 = Extract<1 | 'a' | 2 | 'b', 1 | 'b' | 'c'>;
```

##### 23.4.1.7 链接条件类型

类似于 JavaScript 的三元运算符，我们也可以链式使用 TypeScript 的条件类型运算符：

```ts
type LiteralTypeName<T> =
 T extends undefined ? "undefined" :
 T extends null ? "null" :
 T extends boolean ? "boolean" :
 T extends number ? "number" :
 T extends bigint ? "bigint" :
 T extends string ? "string" :
 never;

// %inferred-type: "bigint"
type Result1 = LiteralTypeName<123n>;

// %inferred-type: "string" | "number" | "boolean"
type Result2 = LiteralTypeName<true | 1 | 'a'>;
```

##### 23.4.1.8 `infer` 和条件类型

https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-inference-in-conditional-types

**例子：**

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

**例子：**

```ts
type Syncify<Interf> = {
 [K in keyof Interf]:
 Interf[K] extends (...args: any[]) => Promise<infer Result>
 ? (...args: Parameters<Interf[K]>) => Result
 : Interf[K];
};

// Example:

interface AsyncInterface {
 compute(arg: number): Promise<boolean>;
 createString(): Promise<String>;
}

type SyncInterface = Syncify<AsyncInterface>;
 // type SyncInterface = {
 //     compute: (arg: number) => boolean;
 //     createString: () => String;
 // }
```

#### 23.4.2 映射类型

*映射类型*通过循环遍历键的集合来生成对象-例如：

```ts
// %inferred-type: { a: number; b: number; c: number; }
type Result = {
 [K in 'a' | 'b' | 'c']: number
};
```

运算符 `in` 是映射类型的重要部分：它指定新对象文字类型的键来自哪里。

##### 23.4.2.1 内置实用类型：`Pick<T, K>`

以下内置实用类型允许我们通过指定要保留的现有对象类型的属性来创建新对象：

```ts
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
 [P in K]: T[P];
};
```

它的使用方式如下：

```ts
type ObjectLiteralType = {
 eeny: 1,
 meeny: 2,
 miny: 3,
 moe: 4,
};

// %inferred-type: { eeny: 1; miny: 3; }
type Result = Pick<ObjectLiteralType, 'eeny' | 'miny'>;
```

##### 23.4.2.2 内置实用类型：`Omit<T, K>`

以下内置实用类型允许我们通过指定要省略的现有对象类型的属性来创建新的对象类型：

```ts
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

解释：

+   `K extends keyof any` 意味着 `K` 必须是所有属性键类型的子类型：

    ```ts
    // %inferred-type: string | number | symbol
    type Result = keyof any;
    ```

+   `Exclude<keyof T, K>>` 的意思是：取出 `T` 的键并移除所有在 `K` 中提到的“值”。

`Omit<>`的使用方式如下：

```ts
type ObjectLiteralType = {
 eeny: 1,
 meeny: 2,
 miny: 3,
 moe: 4,
};

// %inferred-type: { meeny: 2; moe: 4; }
type Result = Omit<ObjectLiteralType, 'eeny' | 'miny'>;
```

### 23.5 其他各种运算符

#### 23.5.1 索引类型查询运算符 `keyof`

我们已经遇到了类型运算符`keyof`。它列出了对象类型的属性键：

```ts
type Obj = {
 0: 'a',
 1: 'b',
 prop0: 'c',
 prop1: 'd',
};

// %inferred-type: 0 | 1 | "prop0" | "prop1"
type Result = keyof Obj;
```

将`keyof`应用于元组类型会产生一个可能有些意外的结果：

```ts
// number | "0" | "1" | "2" | "length" | "pop" | "push" | ···
type Result = keyof ['a', 'b', 'c'];
```

结果包括：

+   元组元素的索引，作为字符串："0" | "1" | "2"

+   索引属性键的类型`number`

+   特殊实例属性`.length`的名称

+   所有`Array`方法的名称："pop" | "push" | ···

空对象文字类型的属性键是空集`never`：

```ts
// %inferred-type: never
type Result = keyof {};
```

这是`keyof`处理交集类型和联合类型的方式：

```ts
type A = { a: number, shared: string };
type B = { b: number, shared: string };

// %inferred-type: "a" | "b" | "shared"
type Result1 = keyof (A & B);

// %inferred-type: "shared"
type Result2 = keyof (A | B);
```

如果我们记得`A & B`具有类型`A`和类型`B`的*共同*属性，这就有意义了。`A`和`B`只有`.shared`属性是共同的，这解释了`Result2`。

#### 23.5.2 索引访问操作符`T[K]`

索引访问操作符`T[K]`返回`T`的所有属性的类型，这些属性的键可以赋值给类型`K`。`T[K]`也被称为*查找类型*。

这些是使用该操作符的例子：

```ts
type Obj = {
 0: 'a',
 1: 'b',
 prop0: 'c',
 prop1: 'd',
};

// %inferred-type: "a" | "b"
type Result1 = Obj[0 | 1];

// %inferred-type: "c" | "d"
type Result2 = Obj['prop0' | 'prop1'];

// %inferred-type: "a" | "b" | "c" | "d"
type Result3 = Obj[keyof Obj];
```

括号中的类型必须可以赋值给所有属性键的类型（由`keyof`计算得出）。这就是为什么`Obj[number]`和`Obj[string]`是不允许的。然而，如果索引类型有一个索引签名（行 A），我们可以使用`number`和`string`作为索引类型：

```ts
type Obj = {
 [key: string]: RegExp, // (A)
};

// %inferred-type: string | number
type KeysOfObj = keyof Obj;

// %inferred-type: RegExp
type ValuesOfObj = Obj[string];
```

`KeysOfObj`包括类型`number`，因为数字键是 JavaScript（因此也是 TypeScript）中字符串键的子集。

元组类型也支持索引访问：

```ts
type Tuple = ['a', 'b', 'c', 'd'];

// %inferred-type:  "a" | "b"
type Elements = Tuple[0 | 1];
```

括号操作符也是分布式的：

```ts
type MyType = { prop: 1 } | { prop: 2 } | { prop: 3 };

// %inferred-type: 1 | 2 | 3
type Result1 = MyType['prop'];

// Equivalent:
type Result2 =
 | { prop: 1 }['prop']
 | { prop: 2 }['prop']
 | { prop: 3 }['prop']
;
```

#### 23.5.3 类型查询操作符`typeof`

类型操作符`typeof`将（JavaScript）值转换为其（TypeScript）类型。它的操作数必须是标识符或一系列用点分隔的标识符：

```ts
const str = 'abc';

// %inferred-type: "abc"
type Result = typeof str;
```

第一个`'abc'`是一个值，而第二个`"abc"`是它的类型，一个字符串字面类型。

这是使用`typeof`的另一个例子：

```ts
const func = (x: number) => x + x;
// %inferred-type: (x: number) => number
type Result = typeof func;
```

§14.1.2 “向类型添加符号”描述了`typeof`的一个有趣用例。

[评论](https://github.com/rauschma/tackling-ts/issues/25)
