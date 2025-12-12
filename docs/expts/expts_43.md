# 35 通过 `infer` 提取复合类型的部分

> 原文：[`exploringjs.com/ts/book/ch_infer-keyword.html`](https://exploringjs.com/ts/book/ch_infer-keyword.html)

（广告，请勿屏蔽。）

1.  35.1 `infer`：提取条件类型 `extends` 子句中的类型

    1.  35.1.1 示例：通过 `Record` 提取属性键和值

1.  35.2 通过 `extends` 约束 `infer`

1.  35.3 使用 `infer` 的内置实用类型

    1.  35.3.1 通过 `infer` 提取函数类型的部分

    1.  35.3.2 通过 `infer` 提取类类型的部分

1.  35.4 示例：异步接口的同步版本

1.  35.5 使用 `infer` 定义局部类型变量

1.  35.6 本章来源

在本章中，我们探讨如何通过 `infer` 关键字提取复合类型的部分。

如果你对 条件类型 有一定的了解会有帮助。

### 35.1 `infer`：提取条件类型 `extends` 子句中的类型

`infer` 在条件类型的约束部分中使用：

```ts
type _ = Type extends Constraint ? <then> : <else>;

```

约束可以是一个类型模式 - 例如：

```ts
T[]
Promise<T>
(arg: T) => R

```

在任何可以提及现有类型变量的位置（例如前例中的 `T` 或 `R`），我们也可以通过 `infer X` 引入一个新的类型变量 `X`。让我们看一个例子：以下泛型类型 `ElemType<Arr>` 提取了数组类型的元素类型：

```ts
type ElemType<Arr> = Arr extends Array<infer Elem> ? Elem : never;
type _ = Assert<Equal<
 ElemType<Array<string>>, string
>>;

```

`infer` 与 JavaScript 中的 [解构](https://exploringjs.com/js/book/ch_destructuring.html#ch_destructuring) 有很多相似之处。

#### 35.1.1 示例：通过 `Record` 提取属性键和值

实用类型 `Record` 允许我们自行实现 `keyof` 操作符：

```ts
const Color = {
 red: 0,
 green: 1,
 blue: 2,
} as const;

type KeyOf<T> = T extends Record<infer K, any> ? K : never;
type _1 = Assert<Equal<
 KeyOf<typeof Color>,
 "red" | "green" | "blue"
>>;

```

我们还可以实现一个实用类型 `PropValues` 来提取属性值：

```ts
type PropValues<T> = T extends Record<any, infer V> ? V : never;
type _2 = Assert<Equal<
 PropValues<typeof Color>,
 0 | 1 | 2
>>;

```

### 35.2 通过 `extends` 约束 `infer`

在某些情况下，我们需要通过使用 `extends` 来约束可以推断的内容以帮助 `infer` - 例如（基于 [Heribert Schütz 的想法](https://mastodon.social/@hcschuetz/114127961269403080)）：

```ts
type EnumFromTuple<T extends ReadonlyArray<string>> =
 T extends [
 infer First extends string, // (A)
 ...infer Rest extends ReadonlyArray<string> // (B)
 ]
 ? Record<First, First> & EnumFromTuple<Rest>
 : {}
 ;
type _ = [
 Assert<Equal< // (C)
 EnumFromTuple<['a', 'b']>,
 Record<'a', 'a'> & Record<'b', 'b'>
 >>,
 Assert<Equal< // (D)
 EnumFromTuple<['a', 'b']>,
 { a: 'a', b: 'b' }
 >>,
];

```

注意事项：

+   这段代码在没有行 A 和行 B 中的两个 `extends` 的情况下无法工作。

+   行 C 显示了 `EnumFromTuple` 生成的确切结果。

+   行 D 显示结果可以赋值给一个简单的对象字面量类型。

### 35.3 使用 `infer` 的内置实用类型

TypeScript 有几个内置的 [实用类型](https://www.typescriptlang.org/docs/handbook/utility-types.html)。在本节中，我们查看那些使用 `infer` 的类型。

#### 35.3.1 通过 `infer` 提取函数类型的一部分

```ts
/**
 * Obtain the parameters of a function type in a tuple
 */
type Parameters<T extends (...args: any) => any> =
 T extends (...args: infer P) => any ? P : never;

/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> =
 T extends (...args: any) => infer R ? R : any;

```

这两个实用类型都很简单：我们在想要提取类型的地方使用 `infer`。让我们使用这两种类型：

```ts
function add(x: number, y: number): number {
 return x + y;
}
type _1 = Assert<Equal<
 typeof add,
 (x: number, y: number) => number
>>;
type _2 = Assert<Equal<
 Parameters<typeof add>,
 [x: number, y: number]
>>;
type _3 = Assert<Equal<
 ReturnType<typeof add>,
 number
>>;

```

#### 35.3.2 通过 `infer` 提取类类型的一部分

以下非内置的类实用类型展示了 **构造签名** 的工作原理：

```ts
type Class<T> = abstract new (...args: Array<any>) => T;

```

此类型包括所有实例类型为 `T` 的类。关键字 `abstract` 表示它包括抽象类和具体类。如果没有这个关键字，它只会包括可实例化的类。

构造签名使我们能够提取类的一部分：

```ts
/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends abstract new (...args: any) => any> =
 T extends abstract new (...args: infer P) => any ? P : never;

/**
 * Obtain the return type of a constructor function type
 */
type InstanceType<T extends abstract new (...args: any) => any> =
 T extends abstract new (...args: any) => infer R ? R : any;

```

为了演示这些实用类型，让我们定义一个我们可以应用它们的类：

```ts
class Point {
 x: number;
 y: number;
 constructor(x: number, y: number) {
 this.x = x;
 this.y = y;
 }
}

```

注意，类 `Point` 定义了两件事：

+   一个类型：类的实例的 `Point` 类型。

+   一个值：一个用于创建类型为 `Point` 的对象的工厂。该值具有 `Class<Point>` 类型。

```ts
const pointAsValue: Class<Point> = Point;
type _ = [
 Assert<Equal<
 ConstructorParameters<typeof Point>,
 [x: number, y: number]
 >>,
 Assert<Equal<
 InstanceType<typeof Point>,
 Point
 >>,
];

```

### 35.4 示例：异步接口的同步版本

以下示例稍微复杂一些：它将对象类型中的所有异步方法转换为同步方法：

```ts
type Syncify<Intf> = {
 [K in keyof Intf]:
 Intf[K] extends (...args: infer A) => Promise<infer R> // (A)
 ? (...args: A) => R // (B)
 : Intf[K] // (C)
};

```

+   条件：当前的属性值 `Intf[K]` 是一个函数吗？（行 A）

+   如果是：新的属性值是一个具有相同参数 `A` 但未包装的返回类型 `R` 的函数。（行 B）

+   如果不是：新的属性值与旧的属性值相同。（行 C）

让我们将 `Syncify` 应用到一个基于 Promise 的方法接口 `AsyncService` 上：

```ts
interface AsyncService {
 factorize(num: number): Promise<Array<number>>;
 createDigest(text: string): Promise<string>;
}
type SyncService = Syncify<AsyncService>;
type _ = Assert<Equal<
 SyncService,
 {
 factorize: (num: number) => Array<number>,
 createDigest: (text: string) => string,
 }
>>;

```

### 35.5 使用 `infer` 定义局部类型变量

我们可以使用 `infer` 定义如 `W` 以下的局部类型变量：

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

更多信息，请参阅 “定义局部类型变量” (§33.8)。

### 35.6 本章来源

+   TypeScript 2.8 宣布博客中 [“在条件类型内进行推断”](https://devblogs.microsoft.com/typescript/announcing-typescript-2-8-2/#inferring-within-conditional-types) 的部分。

+   官方 TypeScript 手册中的 [“在条件类型内进行推断”](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types) 部分。
