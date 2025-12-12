# 14 顶级类型 any 和 unknown

> 原文：[`exploringjs.com/ts/book/ch_any-unknown.html`](https://exploringjs.com/ts/book/ch_any-unknown.html)

(广告，请勿屏蔽。)

1.  14.1 TypeScript 的两个顶级类型

1.  14.2 顶级类型 `any`

    1.  14.2.1 示例：`JSON.parse()`

    1.  14.2.2 示例：`String()`

    1.  14.2.3 编译器选项 `noImplicitAny`

1.  14.3 顶级类型 `unknown`

在 TypeScript 中，`any` 和 `unknown` 是包含所有值的类型。在本章中，我们将探讨它们是什么以及它们可以用于什么。

### 14.1 TypeScript 的两个顶级类型

`any` 和 `unknown` 是 TypeScript 中的所谓 *顶级类型*。引用 [Wikipedia](https://en.wikipedia.org/wiki/Top_type)：

> *顶级类型* [...] 是 *通用* 类型，有时也称为 *通用超类型*，因为任何给定类型系统中的所有其他类型都是其子类型 [...]. 在大多数情况下，它是包含感兴趣的类型系统中所有可能 [值] 的类型。

即，当我们将类型视为值的集合（有关类型的更多信息，请参阅“TypeScript 中的类型是什么？两种视角” (§13))时，`any` 和 `unknown` 是包含所有值的集合。

TypeScript 还有一个 *最低类型* `never`，它是空集，在“最低类型 `never`” (§15)中解释。

### 14.2 顶级类型 `any`

如果一个值具有 `any` 类型，我们可以对它做任何操作：

```ts
function func(value: any) {
 // Only allowed for numbers, but they are a subtype of `any`
 5 * value;

 // Normally the type signature of `value` must contain .propName
 value.propName;

 // Normally only allowed for Arrays and types with index signatures
 value[123];
}

```

每个类型都可以赋值给类型 `any`：

```ts
let storageLocation: any;

storageLocation = null;
storageLocation = true;
storageLocation = {};

```

类型 `any` 可赋值给任何类型：

```ts
function func(value: any) {
 const a: null = value;
 const b: boolean = value;
 const c: object = value;
}

```

使用 `any` 我们会失去 TypeScript 静态类型系统通常给予我们的任何保护。因此，它只能作为最后的手段使用，如果我们不能使用更具体的类型或 `unknown`。

#### 14.2.1 示例：`JSON.parse()`

`JSON.parse()` 的结果取决于动态输入，这就是为什么返回类型是 `any`（我已经从签名中省略了参数 `reviver`）：

```ts
JSON.parse(text: string): any;

```

在 `unknown` 类型存在之前，`JSON.parse()` 就被添加到 TypeScript 中。否则，它的返回类型可能就是 `unknown`。

#### 14.2.2 示例：`String()`

将任意值转换为字符串的函数 `String()` 有以下类型签名：

```ts
interface StringConstructor {
 (value?: any): string; // call signature
 // ···
}

```

#### 14.2.3 编译器选项 `noImplicitAny`

如果编译器选项 `noImplicitAny` 设置为 `true`，TypeScript 要求在它无法推断类型的位置显式添加类型注解。最重要的例子是参数定义。如果此选项设置为 `false`，它（隐式地）使用类型 `any` 在这些位置。

这是一个由 `noImplicitAny` 设置为 `true` 导致的编译器错误的示例：

```ts
// @ts-expect-error: Parameter 'name' implicitly has an 'any' type.
function hello(name) {
 return `Hello ${name}!`
}
assertType<
 (name: any) => string
>(hello);

```

TypeScript 不会对我们显式使用类型 `any` 提出异议：

```ts
function hello(name: any) {
 return `Hello ${name}!`
}

```

### 14.3 顶级类型 `unknown`

类型`unknown`是类型安全的`any`类型的版本。每当您考虑使用`any`时，请首先尝试使用`unknown`。`unknown`与`any`类似，我们可以将其分配给任何值：

```ts
let storageLocation: unknown;

storageLocation = null;
storageLocation = true;
storageLocation = {};

```

然而，`unknown`类型的值不能分配给任何类型：

```ts
function func(value: unknown) {
 // @ts-expect-error: Type 'unknown' is not assignable to type 'null'.
 const a: null = value;
 // @ts-expect-error: Type 'unknown' is not assignable to type 'boolean'.
 const b: boolean = value;
 // @ts-expect-error: Type 'unknown' is not assignable to type 'object'.
 const c: object = value;
}

```

因此，如果我们有一个`unknown`类型的值，我们必须在可以对该值进行任何操作之前缩小该类型——例如，通过：

+   类型断言:

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: 'value' is of type 'unknown'.
     value.toFixed(2);

     // Type assertion:
     (value as number).toFixed(2); // OK
    }

    ```

+   相等性：

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: 'value' is of type 'unknown'.
     value * 5;

     if (value === 123) { // equality
     assertType<123>(value);
     value * 5; // OK
     }
    }

    ```

+   类型守卫:

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: 'value' is of type 'unknown'.
     value.length;

     if (typeof value === 'string') { // type guard
     assertType<string>(value);
     value.length; // OK
     }
    }

    ```

+   断言函数:

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: 'value' is of type 'unknown'.
     value.test('abc');

     assertIsRegExp(value); // assertion function

     assertType<RegExp>(value);
     value.test('abc'); // OK
    }

    /** An assertion function */
    function assertIsRegExp(arg: unknown): asserts arg is RegExp {
     if (! (arg instanceof RegExp)) {
     throw new TypeError('Not a RegExp: ' + arg);
     }
    }

    ```
