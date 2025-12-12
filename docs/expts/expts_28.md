# 23 类作为值的类型

> 原文：[`exploringjs.com/ts/book/ch_classes-as-values.html`](https://exploringjs.com/ts/book/ch_classes-as-values.html)

（广告，请勿拦截。）

1.  23.1 问题：类作为值应该使用哪种类型？

1.  23.2 回答：类作为值的类型

    1.  23.2.1 类型运算符 `typeof`

    1.  23.2.2 构造函数类型字面量

    1.  23.2.3 带有构造签名的对象类型字面量

1.  23.3 构造函数的泛型类型：`Class<T>`

    1.  23.3.1 示例：创建实例

    1.  23.3.2 示例：通过 `instanceof` 进行类型缩小

    1.  23.3.3 示例：带有运行时检查的类型转换

    1.  23.3.4 示例：断言函数

    1.  23.3.5 示例：运行时类型安全的映射

    1.  23.3.6 陷阱：`Class<T>` 不匹配抽象类

在本章中，我们探讨类作为值：

+   对于这样的值，我们应该使用什么类型？

+   这些类型的用例是什么？

### 23.1 问题：类作为值应该使用哪种类型？

考虑以下类：

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

这个函数接受一个类并创建其实例：

```ts
function createPoint(PointClass: C, x: number, y: number): Point {
 return new PointClass(x, y);
}

```

如果我们想让函数返回 `Point` 的实例，我们应该使用什么类型 `C`？

### 23.2 回答：类作为值的类型

#### 23.2.1 类型运算符 `typeof`

在 “TypeScript 的两种语言级别”（§4.4） 中，我们探讨了 TypeScript 的两种语言级别：

+   动态级别：JavaScript（代码和值）

+   静态级别：TypeScript（静态类型）

类 `Point` 创建了两件事：

+   构造函数 `Point`

+   `Point` 实例的接口

根据我们在哪里提到 `Point`，它意味着不同的事情。这就是为什么我们不能为 `PointClass` 使用类型 `Point`：它匹配类 `Point` 的*实例*，而不是类 `Point` 本身。

相反，我们需要使用类型运算符 `typeof`（它与 JavaScript 运算符同名）。`typeof v` 表示值 `v` 的类型。

让我们省略 `createPoint()` 的返回类型，看看 TypeScript 如何推断：

```ts
function createPoint(PointClass: typeof Point, x: number, y: number) {
 return new PointClass(x, y);
}

const point = createPoint(Point, 3, 6);
assertType<Point>(point); // (A)
assert.ok(point instanceof Point);

```

如预期，`createPoint()` 创建了类型为 `Point` 的值（行 A）。

#### 23.2.2 构造函数类型字面量

*构造函数类型字面量* 是构造函数类型的字面量：`new` 后跟一个函数类型字面量（行 A）：

```ts
function createPoint(
 PointClass: new (x: number, y: number) => Point, // (A)
 x: number, y: number
) {
 return new PointClass(x, y);
}

```

其类型的 `new` 前缀表示 `PointClass` 是一个必须通过 `new` 调用的函数。

构造函数类型字面量非常灵活——例如，我们可以要求构造函数（如类）：

+   有特定的参数。

+   返回具有特定接口的实例（见下面的代码）

```ts
function f(
 ClassThatImplementsInterf: new () => Interf
) {}

```

#### 23.2.3 带有构造签名的对象类型字面量

回想一下，接口和对象字面量类型（OLTs）的成员包括方法签名和调用签名。调用签名使接口和 OLTs 能够描述函数。

类似地，*构造签名*使接口和 OLTs 能够描述构造函数。它们看起来像调用签名，但添加了前缀 `new`。在下一个示例中，`PointClass` 有一个具有构造签名的对象字面量类型：

```ts
function createPoint(
 PointClass: {new (x: number, y: number): Point},
 x: number, y: number
) {
 return new PointClass(x, y);
}

```

### 23.3 一种用于构造函数的泛型类型：`Class<T>`

通过我们所获得的知识，我们现在可以为类创建一个通用的类型作为值——通过引入一个类型参数 `T`：

```ts
type Class<T> = new (...args: any[]) => T;

```

除了类型别名，我们还可以使用接口：

```ts
interface Class<T> {
 new(...args: any[]): T;
}

```

`Class<T>` 是一个类型，其实例与类型 `T` 匹配。

#### 23.3.1 示例：创建实例

`Class<T>` 使我们能够编写 `createPoint()` 的泛型版本：

```ts
function createInstance<T>(TheClass: Class<T>, ...args: unknown[]): T {
 return new TheClass(...args);
}

```

`createInstance()` 的使用方法如下：

```ts
class Person {
 constructor(public name: string) {}
}

const jane = createInstance(Person, 'Jane');
assertType<Person>(jane);

```

`createInstance()` 是 `new` 操作符，通过一个函数实现。

#### 23.3.2 示例：通过 `instanceof` 进行类型缩小

在行 A 中，`instanceof` 缩小了 `arg` 的类型：之前它是 `unknown`，之后它是 `T`。

```ts
function isInstance<T>(TheClass: Class<T>, arg: unknown): boolean {
 type _ = Assert<Equal<
 typeof arg, unknown
 >>;
 if (arg instanceof TheClass) { // (A)
 type _ = Assert<Equal<
 typeof arg, T
 >>;
 return true;
 }
 return false;
}

```

#### 23.3.3 示例：带有运行时检查的类型转换

我们可以使用 `Class<T>` 来实现类型转换：

```ts
function cast<T>(TheClass: Class<T>, value: unknown): T {
 if (!(value instanceof TheClass)) {
 throw new Error(`Not an instance of ${TheClass.name}: ${value}`)
 }
 return value;
}

```

使用 `cast()`，我们可以将一个值的类型更改为更具体的东西。这在运行时也是安全的，因为我们既静态地改变了类型，又执行了动态检查。以下代码提供了一个示例：

```ts
function parseObject(jsonObjectStr: string): Object {
 const parsed = JSON.parse(jsonObjectStr);
 type _ = Assert<Equal<
 typeof parsed, any
 >>;
 return cast(Object, parsed);
}

```

#### 23.3.4 示例：断言函数

我们可以将前一小节中的函数 `cast()` 转换为一个 断言函数：

```ts
/**
 * After invoking this function, the inferred type of `value` is `T`.
 */
export function throwIfNotInstance<T>(
 TheClass: Class<T>, value: unknown
): asserts value is T { // (A)
 if (!(value instanceof TheClass)) {
 throw new Error(`Not an instance of ${TheClass}: ${value}`);
 }
}

```

返回类型（行 A）使 `throwIfNotInstance()` 成为一个断言函数，它缩小了类型：

```ts
const parsed = JSON.parse('[1, 2]');
type _1 = Assert<Equal<
 typeof parsed, any
>>;
throwIfNotInstance(Array, parsed);
type _2 = Assert<Equal<
 typeof parsed, Array<unknown>
>>;

```

#### 23.3.5 示例：运行时类型安全的映射

`Class<T>` 和 `cast()` 的一种用例是类型安全的映射：

```ts
class TypeSafeMap {
 #data = new Map<unknown, unknown>();
 get<T>(key: Class<T>) {
 const value = this.#data.get(key);
 return cast(key, value);
 }
 set<T>(key: Class<T>, value: T): this {
 cast(key, value); // runtime check
 this.#data.set(key, value);
 return this;
 }
 has(key: unknown) {
 return this.#data.has(key);
 }
}

```

`TypeSafeMap` 中的每个条目的键是一个类。该类决定了条目值的静态类型，并在运行时进行检查。

这是 `TypeSafeMap` 在起作用：

```ts
const map = new TypeSafeMap();

map.set(RegExp, /abc/);

const re = map.get(RegExp);
assertType<RegExp>(re);

// Static and dynamic error!
assert.throws(
 // @ts-expect-error: Argument of type 'string' is not assignable
 // to parameter of type 'Date'.
 () => map.set(Date, 'abc')
);

```

#### 23.3.6 陷阱：`Class<T>` 不匹配抽象类

考虑以下类：

```ts
abstract class Shape {
}
class Circle extends Shape {
 // ···
}

```

`Class<T>` 不匹配抽象类 `Shape`（最后一行）：

```ts
type Class<T> = new (...args: any[]) => T;

// @ts-expect-error: Type 'typeof Shape' is not assignable to
// type 'Class<Shape>'. Cannot assign an abstract constructor type
// to a non-abstract constructor type.
const shapeClasses1: Array<Class<Shape>> = [Circle, Shape];

```

为什么呢？理由是构造函数类型字面量和构造签名应该只用于实际上可以被 `new` 调用的值。

如果我们想让 `Class<T>` 匹配抽象类和具体类，我们可以使用一个 *抽象构造签名*：

```ts
type Class<T> = abstract new (...args: any[]) => T;
const shapeClasses: Array<Class<Shape>> = [Circle, Shape];

```

有一个注意事项——这个类型不能被 `new` 调用：

```ts
function createInstance<T>(TheClass: Class<T>, ...args: unknown[]): T {
 // @ts-expect-error: Cannot create an instance of an abstract class.
 return new TheClass(...args);
}

```

然而，新的 `Class<T>` 对于所有其他用例都工作得很好，包括 `instanceof`：

```ts
function isInstance<T>(TheClass: Class<T>, arg: unknown): boolean {
 type _ = Assert<Equal<
 typeof arg, unknown
 >>;
 if (arg instanceof TheClass) {
 type _ = Assert<Equal<
 typeof arg, T
 >>;
 return true;
 }
 return false;
}

```

因此，我们可以将旧类型名称更改为 `NewableClass<T>` – 以便在需要类可被 `new` 调用的情况下使用：

```ts
type NewableClass<T> = new (...args: any[]) => T;
function createInstance<T>(TheClass: NewableClass<T>, ...args: unknown[]): T {
 return new TheClass(...args);
}

```
