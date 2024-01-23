# 二十一、类型断言（与转换相关）

> 原文：[`exploringjs.com/tackling-ts/ch_type-assertions.html`](https://exploringjs.com/tackling-ts/ch_type-assertions.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   21.1 类型断言

    +   21.1.1 类型断言的替代语法

    +   21.1.2 示例：断言一个接口

    +   21.1.3 示例：断言索引签名

+   21.2 与类型断言相关的构造

    +   21.2.1 非空断言操作符（后缀`!`）

    +   21.2.2 明确赋值断言

* * *

本章讨论了 TypeScript 中的*类型断言*，它与其他语言中的类型转换相关，并通过`as`操作符执行。

### 21.1 类型断言

类型断言允许我们覆盖 TypeScript 为值计算的静态类型。这对于解决类型系统的限制非常有用。

类型断言与其他语言中的类型转换相关，但它们不会抛出异常，也不会在运行时执行任何操作（它们在静态上执行了一些最小的检查）。

```ts
const data: object = ['a', 'b', 'c']; // (A)

// @ts-expect-error: Property 'length' does not exist on type 'object'.
data.length; // (B)

assert.equal(
 (data as Array<string>).length, 3); // (C)
```

注释：

+   在 A 行，我们将数组的类型扩展为 `object`。

+   在 B 行，我们看到这种类型不允许我们访问任何属性（[详情](https://2ality.com/2020/01/typing-objects-typescript.html#object-%28lowercase-“o”%29-in-typescript%3A-non-primitive-values)）。

+   在 C 行，我们使用类型断言（操作符`as`）告诉 TypeScript `data` 是一个数组。现在我们可以访问属性`.length`。

类型断言是最后的手段，应尽量避免使用。它们（暂时地）移除了静态类型系统通常给我们的安全网。

请注意，在 A 行，我们还覆盖了 TypeScript 的静态类型。但我们是通过类型注释来实现的。这种覆盖方式比类型断言要安全得多，因为我们受到了更严格的约束：TypeScript 的类型必须可以赋值给注释的类型。

#### 21.1.1 类型断言的替代语法

TypeScript 有一种替代的“尖括号”语法用于类型断言：

```ts
<Array<string>>data
```

我建议避免使用这种语法。它已经过时，并且与 React JSX 代码（在`.tsx`文件中）不兼容。

#### 21.1.2 示例：断言一个接口

为了访问任意对象`obj`的属性`.name`，我们暂时将`obj`的静态类型更改为`Named`（A 行和 B 行）。

```ts
interface Named {
 name: string;
}
function getName(obj: object): string {
 if (typeof (obj as Named).name === 'string') { // (A)
 return (obj as Named).name; // (B)
 }
 return '(Unnamed)';
}
```

#### 21.1.3 示例：断言索引签名

在以下代码（A 行）中，我们使用类型断言 `as Dict`，这样我们就可以访问一个值的推断类型为 `object` 的属性。也就是说，我们用静态类型 `Dict` 覆盖了静态类型 `object`。

```ts
type Dict = {[k:string]: any};

function getPropertyValue(dict: unknown, key: string): any {
 if (typeof dict === 'object' && dict !== null && key in dict) {
 // %inferred-type: object
 dict;

 // @ts-expect-error: Element implicitly has an 'any' type because
 // expression of type 'string' can't be used to index type '{}'.
 // [...]
 dict[key];

 return (dict as Dict)[key]; // (A)
 } else {
 throw new Error();
 }
}
```

### 21.2 与类型断言相关的构造

#### 21.2.1 非空断言操作符（后缀`!`）

如果值的类型是包括 `undefined` 或 `null` 类型的联合类型，*非空断言操作符*（或*非空断言操作符*）会从联合类型中移除这些类型。我们告诉 TypeScript：“这个值不能是 `undefined` 或 `null`。” 因此，我们可以执行被这两个值类型阻止的操作 - 例如：

```ts
const theName = 'Jane' as (null | string);

// @ts-expect-error: Object is possibly 'null'.
theName.length;

assert.equal(
 theName!.length, 4); // OK
```

##### 21.2.1.1 示例 - Map：`.has()`后的`.get()`

在使用 Map 方法`.has()`之后，我们知道 Map 具有给定的键。然而，`.get()`的结果并不反映这一知识，这就是为什么我们必须使用非空断言操作符的原因：

```ts
function getLength(strMap: Map<string, string>, key: string): number {
 if (strMap.has(key)) {
 // We are sure x is not undefined:
 const value = strMap.get(key)!; // (A)
 return value.length;
 }
 return -1;
}
```

我们可以在 Map 的值不能为 `undefined` 时避免使用 nullish 断言操作符。然后，可以通过检查 `.get()` 的结果是否为 `undefined` 来检测缺失的条目：

```ts
function getLength(strMap: Map<string, string>, key: string): number {
 // %inferred-type: string | undefined
 const value = strMap.get(key);
 if (value === undefined) { // (A)
 return -1;
 }

 // %inferred-type: string
 value;

 return value.length;
}
```

#### 21.2.2 明确赋值断言

如果打开了*strict property initialization*，我们偶尔需要告诉 TypeScript 我们确实初始化了某些属性 - 尽管它认为我们没有。

这是一个例子，即使不应该，TypeScript 也会抱怨：

```ts
class Point1 {
 // @ts-expect-error: Property 'x' has no initializer and is not definitely
 // assigned in the constructor.
 x: number;

 // @ts-expect-error: Property 'y' has no initializer and is not definitely
 // assigned in the constructor.
 y: number;

 constructor() {
 this.initProperties();
 }
 initProperties() {
 this.x = 0;
 this.y = 0;
 }
}
```

如果我们在 A 行和 B 行使用*definite assignment assertions*（感叹号），错误就会消失：

```ts
class Point2 {
 x!: number; // (A)
 y!: number; // (B)
 constructor() {
 this.initProperties();
 }
 initProperties() {
 this.x = 0;
 this.y = 0;
 }
}
```

[评论](https://github.com/rauschma/tackling-ts/issues/22)
