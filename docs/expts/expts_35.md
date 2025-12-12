# 28 类型断言（与转换相关）

> 原文：[`exploringjs.com/ts/book/ch_type-assertions.html`](https://exploringjs.com/ts/book/ch_type-assertions.html)

（广告，请勿拦截。）

1.  28.1 类型断言

    1.  28.1.1 类型断言的已弃用替代语法

    1.  28.1.2 示例：断言索引签名

    1.  28.1.3 示例：`as any`

1.  28.2 与类型断言相关的构造

    1.  28.2.1 非空断言操作符（后缀 `!`)

    1.  28.2.2 确定赋值断言

    1.  28.2.3 常量断言 (`as const`)

    1.  28.2.4 `satisfies` 操作符

本章介绍 TypeScript 中的 *类型断言*，这与其他语言中的类型转换相关，并且通过 `as` 操作符执行。

### 28.1 类型断言

类型断言让我们可以覆盖 TypeScript 为一个值计算出的静态类型。这对于解决类型系统的局限性是有用的。

类型断言与其他语言的类型转换相关，但它们不会抛出异常，也不会在运行时执行任何操作（它们在静态时执行一些最小的检查）。

```ts
const data: object = ['a', 'b', 'c']; // (A)

// @ts-expect-error: Property 'length' does not exist on type 'object'.
data.length; // (B)

assert.equal(
 (data as Array<string>).length, 3 // (C)
);

```

注释：

+   在行 A 中，我们将 Array 的类型扩展为 `object`。

+   在行 B 中，我们看到这个类型不允许我们访问任何属性（详情）。

+   在行 C 中，我们使用类型断言（操作符 `as`）告诉 TypeScript `data` 是一个 Array。现在我们可以访问属性 `.length`。

类型断言是最后的手段，应尽可能避免。它们移除了静态类型系统通常为我们提供的保障。

注意，在行 A 中，我们同样覆盖了 TypeScript 的类型。但我们是通过类型注解来实现的。这种覆盖方式比类型断言更安全，因为我们受到更多的约束：TypeScript 的类型必须可以赋值给注解的类型。

#### 28.1.1 类型断言的已弃用替代语法

TypeScript 有一个用于类型断言的替代“尖括号”语法：

```ts
<Array<string>>data

```

我建议避免这种语法：

+   这种方式已经过时了。

+   它与 React JSX 代码（在 `.tsx` 文件中）不兼容。

+   如果编译器选项 erasableSyntaxOnly 已激活，则不允许使用。

#### 28.1.2 示例：断言索引签名

在以下代码（行 A）中，我们使用类型断言 `as Dict`，这样我们就可以访问推断类型为 `object` 的值的属性。也就是说，我们正在用 `Dict` 静态类型覆盖 `object` 静态类型。

```ts
type Dict = {[k:string]: unknown};

function getPropertyValue(dict: unknown, key: string): unknown {
 if (typeof dict === 'object' && dict !== null && key in dict) {
 assertType<object>(dict);

 // @ts-expect-error: Element implicitly has an 'any' type because
 // expression of type 'string' can't be used to index type '{}'. No
 // index signature with a parameter of type 'string' was found on
 // type '{}'.
 dict[key];

 return (dict as Dict)[key]; // (A)
 } else {
 throw new Error();
 }
}

```

#### 28.1.3 示例：`as any`

在以下示例中，行 A 中的计算返回类型与行 B 中我们返回的值不匹配：

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

为了消除错误，我们在行 A 使用 `as any`：

```ts
function prependDollarSign<
 Obj extends object
>(obj: Obj): PrependDollarSign<Obj> {
 return Object.fromEntries(
 Object.entries(obj)
 .map(
 ([key, value]) => ['$'+key, value]
 )
 ) as any; // (A)
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

这是一个极端的措施。遗憾的是在这种情况下是不可避免的。更多信息，请参阅“函数的返回类型通常与返回值不匹配”（§33.14）。

### 28.2 与类型断言相关的构造

#### 28.2.1 非空断言操作符（后缀 `!`）

如果一个值的类型是包含类型 `undefined` 或 `null` 的联合类型，则 *非空断言操作符*（或 *非空断言操作符*）会从联合中移除这些类型。我们告诉 TypeScript：“这个值不能是 `undefined` 或 `null`。”因此，我们可以执行被这两个值的类型所阻止的操作——例如：

```ts
const theName = 'Jane' as (null | string);

// @ts-expect-error: 'theName' is possibly 'null'.
theName.length;

assert.equal(
 theName!.length, 4); // OK

```

##### 28.2.1.1 示例 - Maps：`.has()` 后的 `.get()`

在我们使用 Map 方法 `.has()` 之后，我们知道 Map 有一个特定的键。遗憾的是，`.get()` 的结果并没有反映这一知识，这就是为什么我们必须使用非空断言操作符的原因：

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

当 Map 的值不能为 `undefined` 时，我们可以避免使用非空断言操作符。然后可以通过检查 `.get()` 的结果是否为 `undefined` 来检测缺失条目：

```ts
function getLength(strMap: Map<string, string>, key: string): number {
 const value = strMap.get(key);
 assertType<string | undefined>(value);

 if (value === undefined) { // (A)
 return -1;
 }
 assertType<string>(value);

 return value.length;
}

```

#### 28.2.2 确定赋值断言

如果启用了*严格属性初始化*，我们有时需要告诉 TypeScript 我们确实初始化了某些属性——即使它认为我们没有。

这是一个 TypeScript 虽然不应该但仍然抱怨的例子：

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

如果我们在行 A 和行 B 使用 *确定赋值断言*（感叹号），错误就会消失：

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

#### 28.2.3 常量断言（`as const`）

常量断言使值只读，并导致更具体的推断类型——例如：

```ts
const obj = { prop: 123 };
type _1 = Assert<Equal<
 typeof obj, { prop: number }
>>;
const constObj = { prop: 123 } as const;
type _2 = Assert<Equal<
 typeof constObj, { readonly prop: 123 }
>>;

const arr = ['a', 'b'];
type _3 = Assert<Equal<
 typeof arr, string[]
>>;
const constTuple = ['a', 'b'] as const;
type _4 = Assert<Equal<
 typeof constTuple, readonly ["a", "b"]
>>;

```

更多信息：“常量断言（`as const`）”（§25.7）。

#### 28.2.4 `satisfies` 操作符

`satisfies` 操作符强制一个值具有给定的类型，但（主要）不会影响该值的类型——例如：

```ts
const TextStyle  = {
 Bold: {
 html: 'b',
 latex: 'textbf',
 },
 // @ts-expect-error: Property 'latex' is missing in type
 // '{ html: string; }' but required in type 'TTextStyle'.
 Italics: { // (A)
 html: 'i',
 },
} satisfies Record<string, TTextStyle>; // (B)
type TTextStyle = {
 html: string,
 latex: string,
};

type TextStyleKeys = keyof typeof TextStyle; // (C)
type _ = Assert<Equal<
 TextStyleKeys, "Bold" | "Italics"
>>;

```

行 B 中的 `satisfies` 操作符捕获了行 A 中的错误，但不会阻止 `TextStyle` 具有对象字面量类型。因此，我们可以在行 C 中提取属性键。

更多信息：“`satisfies` 操作符”（§29）。
