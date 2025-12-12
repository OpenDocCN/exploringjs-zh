# 24 数组类型

> 原文：[`exploringjs.com/ts/book/ch_typing-arrays.html`](https://exploringjs.com/ts/book/ch_typing-arrays.html)

(广告，请勿屏蔽。)

1.  24.1 数组类型化方法

    1.  [24.1.1 数组类型：数组类型字面量 `T[]` 与泛型类型 `Array<T>`](#array-types-array-type-literal-t-vs-generic-type-array-t)

    1.  24.1.2 数组元组类型：元组类型字面量

    1.  24.1.3 同时也是数组类型的对象：具有索引签名的接口

1.  24.2 陷阱：类型推断并不总是正确推断数组类型

    1.  24.2.1 数组类型推断困难

    1.  24.2.2 非空数组字面量的类型推断

    1.  24.2.3 空数组字面量的类型推断

1.  24.3 数组的常量断言及其对类型推断的影响

    1.  24.3.1 `as const` 陷阱：常量元组不能赋值给可变类型

    1.  24.3.2 `as const` 陷阱：推断的类型非常狭窄

    1.  24.3.3 关于只读数组和元组的更多信息

1.  24.4 数组陷阱：检查元素是否存在

1.  [24.5 为什么有两种表示法？TypeScript 为什么更喜欢 `T[]`？](#why-are-there-two-notations-why-does-typescript-prefer-t)

    1.  24.5.1 为什么我更喜欢 `Array<T>` 的表示法

    1.  [24.5.2 支持 `T[]` 的一个观点](#one-point-in-favor-of-t)

    1.  [24.5.3 语法注意事项：`T[]` 中的 `[]` 强烈绑定](#syntactic-caveat-the-in-t-binds-strongly)

    1.  24.5.4 检查数组表示法

在本章中，我们探讨如何在 TypeScript 中对数组进行类型化。

### 24.1 数组类型化方法

TypeScript 有两种数组类型：

+   数组类型：所有元素具有相同的类型。数组的长度可变。

+   数组元组类型：数组的长度是固定的。元素不一定具有相同的类型。

#### [24.1.1 数组类型：数组类型字面量 `T[]` 与泛型类型 `Array<T>`](#array-types-array-type-literal-t-vs-generic-type-array-t)

数组类型字面量由元素类型后跟 `[]` 组成。在以下代码中，数组类型字面量是 `string[]`：

```ts
// Each Array element has the type `string`:
const myStringArray: string[] = ['fee', 'fi', 'fo', 'fum'];

```

数组类型字面量相当于使用泛型类型 `Array`：

```ts
const myStringArray: Array<string> = ['fee', 'fi', 'fo', 'fum'];

```

如果元素类型更复杂，我们需要为数组类型字面量使用括号：

```ts
(number|string)[]
(() => boolean)[]

```

在这种情况下，泛型类型 `Array` 工作得更好：

```ts
Array<number|string>
Array<() => boolean>

```

#### 24.1.2 数组元组类型：元组类型字面量

如果数组有固定长度，并且每个元素都有不同的、固定的类型，该类型取决于其位置，那么我们可以使用元组类型字面量，如`[string, string, boolean]`：

```ts
const yes: [string, string, boolean] = ['oui', 'sí', true];

```

#### 24.1.3 既是数组又是类似数组的对象：具有索引签名的接口

如果一个接口只有一个索引签名，我们可以将其用于数组：

```ts
interface StringArray {
 [index: number]: string;
}
const strArr: StringArray = ['Huey', 'Dewey', 'Louie'];

```

具有索引签名和属性签名的接口仅适用于对象（因为索引元素和属性需要同时定义）：

```ts
interface FirstNamesAndLastName {
 [index: number]: string;
 lastName: string;
}

const ducks: FirstNamesAndLastName = {
 0: 'Huey',
 1: 'Dewey',
 2: 'Louie',
 lastName: 'Duck',
};

```

### 24.2 陷阱：类型推断并不总是正确推断数组类型

#### 24.2.1 推断数组类型困难

由于有两种数组类型，TypeScript 不可能总是猜对正确的类型。例如，考虑以下分配给变量`fields`的数组字面量：

```ts
const fields: Fields = [
 ['first', 'string', true],
 ['last', 'string', true],
 ['age', 'number', false],
];

```

`fields`的最佳类型是什么？以下都是合理的选择：

```ts
type Fields = Array<[string, string, boolean]>;

```

```ts
type Fields = Array<[string, ('string'|'number'), boolean]>;

```

```ts
type Fields = Array<Array<string|boolean>>;

```

```ts
type Fields = [
 [string, string, boolean],
 [string, string, boolean],
 [string, string, boolean],
];

```

```ts
type Fields = [
 [string, 'string', boolean],
 [string, 'string', boolean],
 [string, 'number', boolean],
];

```

```ts
type Fields = [
 Array<string|boolean>,
 Array<string|boolean>,
 Array<string|boolean>,
];

```

#### 24.2.2 非空数组字面量的类型推断

当我们使用非空数组字面量时，TypeScript 的默认做法是推断数组类型（而不是元组类型）：

```ts
const arr = [123, 'abc'];
assertType<(string | number)[]>(arr);

```

然而，这并不总是我们想要的：

```ts
function func(p: readonly [number, number]) { // (A)
 return p;
}
const pair1 = [1, 2];
assertType<number[]>(pair1);

// @ts-expect-error: Argument of type 'number[]' is not assignable to
// parameter of type 'readonly [number, number]'. [...]
func(pair1); // (B)

```

在这一行我们需要使用关键字`readonly`，这样`pair3`（见下文）才能被接受。

我们可以通过类型注解来修复行 B 中的错误：

```ts
const pair2: [number, number] = [1, 2];
func(pair2); // OK

```

另一个选项是 const 断言（`as const`）：

```ts
const pair3 = [1, 2] as const;
func(pair3); // OK

```

关于 const 断言的更多内容即将介绍。

#### 24.2.3 空数组字面量的类型推断

如果我们用一个空数组字面量初始化变量，那么 TypeScript 最初推断出类型`any[]`，并在我们进行更改时逐步更新该类型：

```ts
// @ts-expect-error: Variable 'arr' implicitly has type 'any[]' in some
// locations where its type cannot be determined.
const arr = []; // (A)
// @ts-expect-error: Variable 'arr' implicitly has an 'any[]' type.
assertType<any[]>(arr); // (B)

arr.push(123);
assertType<number[]>(arr);

arr.push('abc');
assertType<(string | number)[]>(arr);

```

有趣的是，如果我们删除行 B，行 A 中的错误就会消失：TypeScript 只有在以某种方式观察到该类型时才会对`any[]`类型提出抱怨。

如果我们使用赋值而不是`.push()`，效果相同：

```ts
// @ts-expect-error: Variable 'arr' implicitly has type 'any[]' in some
// locations where its type cannot be determined.
const arr = [];
// @ts-expect-error: Variable 'arr' implicitly has an 'any[]' type.
assertType<any[]>(arr);

arr[0] = 123;
assertType<number[]>(arr);

arr[1] = 'abc';
assertType<(string | number)[]>(arr);

```

相反，如果数组字面量至少有一个元素，那么元素类型是固定的，以后不会改变：

```ts
const arr = [123];
assertType<number[]>(arr);

// @ts-expect-error: Argument of type 'string' is not assignable to
// parameter of type 'number'.
arr.push('abc');

```

### 24.3 数组的 const 断言及其对类型推断的影响

我们可以在数组字面量后添加一个 const 断言（const 断言）：

```ts
const rockCategories =
 ['igneous', 'metamorphic', 'sedimentary'] as const;
assertType<
 readonly ['igneous', 'metamorphic', 'sedimentary']
>(rockCategories);

```

我们声明`rockCategories`不会改变。这有以下影响：

+   数组变为`readonly` – 我们不能使用改变它的操作：

    ```ts
    // @ts-expect-error: Property 'push' does not exist on type
    // 'readonly ["igneous", "metamorphic", "sedimentary"]'.
    rockCategories.push('sand');

    ```

+   TypeScript 推断出元组。比较：

    ```ts
    const rockCategories2 = ['igneous', 'metamorphic', 'sedimentary'];
    assertType<string[]>(rockCategories2);

    ```

+   TypeScript 推断出狭窄的字面量类型（如`'igneous'`等），而不是更通用的类型。也就是说，推断出的元组类型不是`[string, string, string]`。

这里有一些带有和不带有 const 断言的数组字面量的示例：

```ts
const numbers1 = [1, 2, 3, 4] as const;
assertType<readonly [1, 2, 3, 4]>(numbers1);
const numbers2 = [1, 2, 3, 4];
assertType<number[]>(numbers2);

const booleanAndString1 = [true, 'abc'] as const;
assertType<readonly [true, 'abc']>(booleanAndString1);
const booleanAndString2 = [true, 'abc'];
assertType<(string | boolean)[]>(booleanAndString2);

```

#### 24.3.1 `as const`陷阱：const 元组不可赋值给可变类型

```ts
function argIsArray(arg: Array<string>) {}
function argIsReadonlyArray(arg: ReadonlyArray<string>) {}
function argIsTuple(arg: [string, string]) {}
function argIsReadonlyTuple(arg: readonly [string, string]) {}

const constTuple = ['a', 'b'] as const;
// @ts-expect-error: Argument of type 'readonly ["a", "b"]' is not
// assignable to parameter of type '[string, string]'. The type 'readonly
// ["a", "b"]' is 'readonly' and cannot be assigned to the mutable type
// '[string, string]'.
argIsTuple(constTuple);
argIsReadonlyTuple(constTuple);
// @ts-expect-error: Argument of type 'readonly ["a", "b"]' is not
// assignable to parameter of type 'string[]'. The type 'readonly
// ["a", "b"]' is 'readonly' and cannot be assigned to the mutable type
// 'string[]'.
argIsArray(constTuple);
argIsReadonlyArray(constTuple);

```

可变元组没有这个问题：

```ts
const mutableTuple: [string, string] = ['a', 'b'];
argIsTuple(mutableTuple);
argIsReadonlyTuple(mutableTuple);
argIsArray(mutableTuple);
argIsReadonlyArray(mutableTuple);

```

我们的教训：如果可能的话，我们应该为函数和方法的元组参数使用 `readonly`。

#### 24.3.2 `as const` 陷阱：推断的类型非常狭窄

常量断言的数组字面量的推断类型尽可能狭窄。这导致 `let` 声明的变量出现问题：我们不能分配任何除了我们用于初始化的元组之外的元组：

```ts
let arr = [1, 2] as const;

arr = [1, 2]; // OK

// @ts-expect-error: Type '3' is not assignable to type '2'.
arr = [1, 3];

```

#### 24.3.3 关于只读数组和元组的更多信息

见 “只读访问权限 (`readonly` 等)`。

### 24.4 数组陷阱：检查元素是否存在

使用 编译器选项 `noUncheckedIndexedAccess`，TypeScript 总是推断 `Array<T>` 元素的类型为 `undefined | T`：

```ts
function f(arr: Array<string>): void {
 if (arr.length > 0) { // (A)
 const elem = arr[0];
 assertType<string | undefined>(elem);
 }
}

```

即使在行 A 中检查 `.length` 也不会改变这一点。然而，`in` 操作符表现更好：

```ts
function f(arr: Array<string>): void {
 if (0 in arr) {
 const elem = arr[0];
 assertType<string>(elem);
 }
}

```

或者，我们可以在访问数组元素后检查 `undefined`：

```ts
function f(arr: Array<string>): void {
 const elem = arr[0];
 if (elem !== undefined) {
 assertType<string>(elem);
 }
}

```

### [24.5 为什么有两种表示法？为什么 TypeScript 倾向于 `T[]`？](#why-are-there-two-notations-why-does-typescript-prefer-t)

对于字符串数组，以下两种表示法完全等价：

+   `string[]`

+   `Array<string>`

为什么有两种表示法？

+   [TypeScript 0.9](https://devblogs.microsoft.com/typescript/announcing-typescript-0-9/) 将泛型引入语言，并启用了 `Array<T>` 的表示法。

+   在此之前，`T[]` 是数组的唯一表示法。

因此，看起来 TypeScript 在 0.9 版本发布时只是坚持了现状：它仍然只独家使用 `T[]` – 例如，在推断类型或甚至显示被写成 `Array<T>` 的类型时。

即使我已经读过一两次说 JSX 是 TypeScript 倾向的原因，但我并不认为这是事实：

+   `Array<T>` 表示法与 JSX 兼容：它仅存在于类型级别，可以在 `.jsx` 文件中无任何问题使用。

+   然而，JSX 确实使一种语法变得无法使用 – [通过尖括号进行类型断言](https://www.typescriptlang.org/docs/handbook/jsx.html#the-as-operator)：

    ```ts
    // Not possible in .tsx files:
    const value1 = <DesiredType>valueWithWrongType;

    // Can be used everywhere:
    const value2 = valueWithWrongType as DesiredType;

    ```

#### 24.5.1 为什么我喜欢 `Array<T>` 的表示法

我发现它通常看起来更好 – 尤其是当与元素类型相关的结构变得更加复杂时：

```ts
// Array of tuples
type AT1 = Array<[string, number]>;
type AT2 = [string, number][];

// Array of union elements
type AU1 = Array<number | string>;
type AU2 = (number | string)[];

// Inferring type variables
type ExtractElem1<A> = A extends Array<infer Elem> ? Elem : never;
type ExtractElem2<A> = A extends (infer Elem)[] ? Elem : never;

// Readonly types
type RO1 = ReadonlyArray<unknown>;
type RO2 = readonly unknown[];
 // `readonly` applies to `[]` not to `unknown`!

```

更多原因：

+   `Array<T>` 看起来与 `Set<T>` 和 `Map<K,V>` 类似。

+   `T[]` 可能会与 `[T]`（具有单个组件且类型为 `T` 的元组）混淆 – 尤其是新接触 TypeScript 的人。

+   如果我们想要创建一个没有类型注解的空数组，那么这种语法与尖括号类型表示法更一致：

    ```ts
    const strArr = new Array<string>();

    ```

#### [24.5.2 `T[]` 的一个优点](#one-point-in-favor-of-t)

因为 TypeScript 总是使用 `T[]`，使用该表示法的代码与语言工具更一致。

#### [24.5.3 语法注意事项：`T[]` 中的 `[]` 强烈绑定](#syntactic-caveat-the-in-t-binds-strongly)

`T[]` 中的方括号具有强绑定性。因此，我们需要为任何由多个标记组成的类型 `T` 添加括号 – 例如（行 A，行 B）:

```ts
type ArrayOfStringOrNumber = (string | number)[]; // (A)

const Activation = {
 Active: 'Active',
 Inactive: 'Inactive',
} as const;
type ActivationKeys = (keyof typeof Activation)[]; // (B)
 // ("Active" | "Inactive")[]

```

#### 24.5.4 数组符号的代码检查

typescript-eslint 有一个名为 `array-type` 的规则，用于强制执行一致的数组符号风格。选项包括：

+   总是使用 `T[]`

+   总是使用 `Array<T>`

+   `T[]` 用于单个标记的 `T`；否则使用 `Array<T>`
