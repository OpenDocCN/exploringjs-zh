# 19 数组的类型化

> 原文：[`exploringjs.com/tackling-ts/ch_typing-arrays.html`](https://exploringjs.com/tackling-ts/ch_typing-arrays.html)

* * *

+   19.1 数组的角色

+   19.2 数组的类型化方式

    +   19.2.1 数组角色“列表”：数组类型字面量 vs. 接口类型`Array`

    +   19.2.2 数组角色“元组”：元组类型字面量

    +   19.2.3 也是类似数组的对象：带有索引签名的接口

+   19.3 陷阱：类型推断并不总是正确获取数组类型

    +   19.3.1 推断数组类型很困难

    +   19.3.2 非空数组字面量的类型推断

    +   19.3.3 空数组字面量的类型推断

    +   19.3.4 对数组和类型推断进行`const`断言

+   19.4 陷阱：TypeScript 假设索引永远不会越界

* * *

在本章中，我们将讨论如何在 TypeScript 中为数组添加类型。

### 19.1 数组的角色

数组在 JavaScript 中可以扮演以下角色（单一或混合）：

+   列表：所有元素具有相同的类型。数组的长度不同。

+   元组：数组的长度是固定的。元素不一定具有相同的类型。

TypeScript 通过提供各种数组类型化的方式来适应这两种角色。我们将在下面看看这些方式。

### 19.2 数组的类型化方式

#### 19.2.1 数组角色“列表”：数组类型字面量 vs. 接口类型`Array`

数组类型字面量由元素类型后跟`[]`组成。在下面的代码中，数组类型字面量是`string[]`：

```ts
// Each Array element has the type `string`:
const myStringArray: string[] = ['fee', 'fi', 'fo', 'fum'];
```

数组类型字面量是使用全局通用接口类型`Array`的简写：

```ts
const myStringArray: Array<string> = ['fee', 'fi', 'fo', 'fum'];
```

如果元素类型更复杂，我们需要使用数组类型字面量的括号：

```ts
(number|string)[]
(() => boolean)[]
```

在这种情况下，通用类型`Array`更适用：

```ts
Array<number|string>
Array<() => boolean>
```

#### 19.2.2 数组角色“元组”：元组类型字面量

如果数组的长度固定，并且每个元素具有不同的固定类型，取决于其位置，则我们可以使用元组类型字面量，例如`[string, string, boolean]`：

```ts
const yes: [string, string, boolean] = ['oui', 'sí', true];
```

#### 19.2.3 也是类似数组的对象：带有索引签名的接口

如果一个接口只有一个索引签名，我们可以用它来表示数组：

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

### 19.3 陷阱：类型推断并不总是正确获取数组类型

#### 19.3.1 推断数组类型很困难

由于数组的两种角色，TypeScript 不可能总是猜对类型。例如，考虑以下分配给变量`fields`的数组字面量：

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

#### 19.3.2 非空数组字面量的类型推断

当我们使用非空数组字面量时，TypeScript 的默认值是推断列表类型（而不是元组类型）：

```ts
// %inferred-type: (string | number)[]
const arr = [123, 'abc'];
```

然而，这并不总是我们想要的：

```ts
function func(p: [number, number]) {
 return p;
}
// %inferred-type: number[]
const pair1 = [1, 2];

// @ts-expect-error: Argument of type 'number[]' is not assignable to
// parameter of type '[number, number]'. [...]
func(pair1);
```

我们可以通过在`const`声明中添加类型注释来解决这个问题，从而避免类型推断：

```ts
const pair2: [number, number] = [1, 2];
func(pair2); // OK
```

#### 19.3.3 空数组字面量的类型推断

如果我们用空数组字面量初始化一个变量，那么 TypeScript 最初会推断类型为`any[]`，并在我们进行更改时逐渐更新该类型：

```ts
// %inferred-type: any[]
const arr1 = [];

arr1.push(123);
// %inferred-type: number[]
arr1;

arr1.push('abc');
// %inferred-type: (string | number)[]
arr1;
```

请注意，初始推断类型不受后续发生的影响。

如果我们使用赋值而不是`.push()`，事情会保持不变：

```ts
// %inferred-type: any[]
const arr1 = [];

arr1[0] = 123;
// %inferred-type: number[]
arr1;

arr1[1] = 'abc';
// %inferred-type: (string | number)[]
arr1;
```

相反，如果数组文字至少有一个元素，则元素类型是固定的，以后不会改变：

```ts
// %inferred-type: number[]
const arr = [123];

// @ts-expect-error: Argument of type '"abc"' is not assignable to
// parameter of type 'number'. (2345)
arr.push('abc');
```

#### 19.3.4 数组和类型推断的`const`断言

我们可以在数组文字后缀中使用[a `const` assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)：

```ts
// %inferred-type: readonly ["igneous", "metamorphic", "sedimentary"]
const rockCategories =
 ['igneous', 'metamorphic', 'sedimentary'] as const;
```

我们声明`rockCategories`不会改变。这有以下影响：

+   数组变为`readonly` - 我们不能使用改变它的操作：

    ```ts
    // @ts-expect-error: Property 'push' does not exist on type
    // 'readonly ["igneous", "metamorphic", "sedimentary"]'. (2339)
    rockCategories.push('sand');
    ```

+   TypeScript 推断出一个元组。比较：

    ```ts
    // %inferred-type: string[]
    const rockCategories2 = ['igneous', 'metamorphic', 'sedimentary'];
    ```

+   TypeScript 推断出文字类型（例如`"igneous"`等）而不是更一般的类型。也就是说，推断的元组类型不是`[string, string, string]`。

以下是使用和不使用`const`断言的更多数组文字的示例：

```ts
// %inferred-type: readonly [1, 2, 3, 4]
const numbers1 = [1, 2, 3, 4] as const;
// %inferred-type: number[]
const numbers2 = [1, 2, 3, 4];

// %inferred-type: readonly [true, "abc"]
const booleanAndString1 = [true, 'abc'] as const;
// %inferred-type: (string | boolean)[]
const booleanAndString2 = [true, 'abc'];
```

##### 19.3.4.1 `const`断言的潜在陷阱

`const`断言有两个潜在的陷阱。

首先，推断类型尽可能狭窄。这对于使用`let`声明的变量会造成问题：我们不能分配除了初始化时使用的元组之外的任何其他元组：

```ts
let arr = [1, 2] as const;

arr = [1, 2]; // OK

// @ts-expect-error: Type '3' is not assignable to type '2'. (2322)
arr = [1, 3];
```

其次，通过`as const`声明的元组不能被改变：

```ts
let arr = [1, 2] as const;

// @ts-expect-error: Cannot assign to '1' because it is a read-only
// property. (2540)
arr[1] = 3;
```

这既不是优势也不是劣势，但我们需要意识到这一点。

### 19.4 陷阱：TypeScript 假设索引永远不会超出范围

每当我们通过索引访问数组元素时，TypeScript 总是假设索引在范围内（A 行）：

```ts
const messages: string[] = ['Hello'];

// %inferred-type: string
const message = messages[3]; // (A)
```

由于这个假设，`message`的类型是`string`。而不是`undefined`或`undefined|string`，正如我们可能期望的那样。

如果我们使用元组类型，我们会得到一个错误：

```ts
const messages: [string] = ['Hello'];

// @ts-expect-error: Tuple type '[string]' of length '1' has no element
// at index '1'. (2493)
const message = messages[1];
```

`as const`会产生相同的效果，因为它会导致推断出一个元组类型。

[评论](https://github.com/rauschma/tackling-ts/issues/19)
