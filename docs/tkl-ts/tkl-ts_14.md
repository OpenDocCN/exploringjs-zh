# 十一、The top types any and unknown

> 原文：[`exploringjs.com/tackling-ts/ch_any-unknown.html`](https://exploringjs.com/tackling-ts/ch_any-unknown.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   11.1 TypeScript 的两个顶级类型

+   11.2 顶级类型`any`

    +   11.2.1 示例：`JSON.parse()`

    +   11.2.2 示例：`String()`

+   11.3 顶级类型`unknown`

* * *

在 TypeScript 中，`any`和`unknown`是包含所有值的类型。在本章中，我们将研究它们是什么以及它们可以用于什么。

### 11.1 TypeScript 的两个顶级类型

`any`和`unknown`是 TypeScript 中所谓的*顶级类型*。引用[Wikipedia](https://en.wikipedia.org/wiki/Top_type)：

> *顶级类型*是*通用*类型，有时被称为*通用超类型*，因为在任何给定类型系统中，所有其他类型都是子类型。在大多数情况下，它是包含感兴趣的类型系统中的每个可能值的类型。

也就是说，当将类型视为值的集合时（有关类型是什么的更多信息，请参见[[content not included]](ch_missing-chapters-online.html)），`any`和`unknown`是包含所有值的集合。顺便说一句，TypeScript 还有*底部类型*`never`，它是空集。

### 11.2 顶级类型`any`

如果一个值的类型是`any`，我们可以对它做任何事情：

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

每种类型都可以分配给类型`any`：

```ts
let storageLocation: any;

storageLocation = null;
storageLocation = true;
storageLocation = {};
```

类型`any`可以分配给每种类型：

```ts
function func(value: any) {
 const a: null = value;
 const b: boolean = value;
 const c: object = value;
}
```

使用`any`会失去 TypeScript 静态类型系统通常给我们的任何保护。因此，只有在无法使用更具体的类型或`unknown`时，才应该使用它作为最后的手段。

#### 11.2.1 示例：`JSON.parse()`

`JSON.parse()`的结果取决于动态输入，这就是为什么返回类型是`any`（我已经从签名中省略了参数`reviver`）：

```ts
JSON.parse(text: string): any;
```

在类型`unknown`存在之前，`JSON.parse()`被添加到 TypeScript 中。否则，它的返回类型可能是`unknown`。

#### 11.2.2 示例：`String()`

将任意值转换为字符串的函数`String()`具有以下类型签名：

```ts
interface StringConstructor {
 (value?: any): string; // call signature
 // ···
}
```

### 11.3 顶级类型`unknown`

类型`unknown`是类型`any`的类型安全版本。每当你考虑使用`any`时，先尝试使用`unknown`。

`any`允许我们做任何事情，而`unknown`则更加限制。

在对类型为`unknown`的值执行任何操作之前，我们必须通过以下方式先缩小它们的类型：

+   类型断言：

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: Object is of type 'unknown'.
     value.toFixed(2);

     // Type assertion:
     (value as number).toFixed(2); // OK
    }
    ```

+   相等性：

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: Object is of type 'unknown'.
     value * 5;

     if (value === 123) { // equality
     // %inferred-type: 123
     value;

     value * 5; // OK
     }
    }
    ```

+   类型守卫：

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: Object is of type 'unknown'.
     value.length;

     if (typeof value === 'string') { // type guard
     // %inferred-type: string
     value;

     value.length; // OK
     }
    }
    ```

+   断言函数：

    ```ts
    function func(value: unknown) {
     // @ts-expect-error: Object is of type 'unknown'.
     value.test('abc');

     assertIsRegExp(value);

     // %inferred-type: RegExp
     value;

     value.test('abc'); // OK
    }

    /** An assertion function */
    function assertIsRegExp(arg: unknown): asserts arg is RegExp {
     if (! (arg instanceof RegExp)) {
     throw new TypeError('Not a RegExp: ' + arg);
     }
    }
    ```

[评论](https://github.com/rauschma/tackling-ts/issues/21)
