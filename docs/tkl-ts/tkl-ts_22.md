# 十八、类作为值的类型

> 原文：[`exploringjs.com/tackling-ts/ch_classes-as-values.html`](https://exploringjs.com/tackling-ts/ch_classes-as-values.html)

* * *

+   18.1 特定类的类型

+   18.2 类型操作符`typeof`

    +   18.2.1 构造函数类型文本

    +   18.2.2 带有构造签名的对象类型文本

+   18.3 类的通用类型：`Class<T>`

    +   18.3.1 示例：创建实例

    +   18.3.2 示例：带有运行时检查的类型转换

    +   18.3.3 示例：在运行时类型安全的映射

    +   18.3.4 陷阱：`Class<T>`不匹配抽象类

* * *

在本章中，我们探讨了类作为值：

+   我们应该为这些值使用什么类型？

+   这些类型的用例是什么？

### 18.1 特定类的类型

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

这个函数接受一个类并创建它的一个实例：

```ts
function createPoint(PointClass: ???, x: number, y: number) {
 return new PointClass(x, y);
}
```

如果我们希望参数`PointClass`的类型为`Point`或其子类，应该使用什么类型？

### 18.2 类型操作符`typeof`

在§7.7“两种语言级别：动态 vs. 静态”中，我们探讨了 TypeScript 的两种语言级别：

+   动态级别：JavaScript（代码和值）

+   静态级别：TypeScript（静态类型）

类`Point`创建了两个东西：

+   构造函数`Point`

+   接口`Point`用于`Point`的实例

根据我们提到`Point`的地方不同，它表示不同的东西。这就是为什么我们不能将类型`Point`用于`PointClass`：它匹配类`Point`的*实例*，而不是类`Point`本身。

相反，我们需要使用类型操作符`typeof`（TypeScript 语法的一部分，也存在于 JavaScript 中）。`typeof v`代表动态(!)值`v`的类型。

```ts
function createPoint(PointClass: typeof Point, x: number, y: number) { // (A)
 return new PointClass(x, y);
}

// %inferred-type: Point
const point = createPoint(Point, 3, 6);
assert.ok(point instanceof Point);
```

#### 18.2.1 构造函数类型文本

*构造函数类型文本*是一个带有前缀`new`的函数类型文本（A 行）。前缀表示`PointClass`是一个必须通过`new`调用的函数。

```ts
function createPoint(
 PointClass: new (x: number, y: number) => Point, // (A)
 x: number, y: number
) {
 return new PointClass(x, y);
}
```

#### 18.2.2 带有构造签名的对象类型文本

回想一下接口和对象文本类型（OLT）的成员包括方法签名和调用签名。调用签名使接口和 OLT 能够描述函数。

同样，*构造签名*使接口和 OLT 能够描述构造函数。它们看起来像带有前缀`new`的调用签名。在下一个示例中，`PointClass`具有带有构造签名的对象文本类型：

```ts
function createPoint(
 PointClass: {new (x: number, y: number): Point},
 x: number, y: number
) {
 return new PointClass(x, y);
}
```

### 18.3 类的通用类型：`Class<T>`

根据我们所学的知识，我们现在可以创建一个类的通用类型作为值 - 通过引入类型参数`T`：

```ts
type Class<T> = new (...args: any[]) => T;
```

除了类型别名，我们还可以使用接口：

```ts
interface Class<T> {
 new(...args: any[]): T;
}
```

`Class<T>`是一个类的类型，其实例匹配类型`T`。

#### 18.3.1 示例：创建实例

`Class<T>`使我们能够编写`createPoint()`的通用版本：

```ts
function createInstance<T>(AnyClass: Class<T>, ...args: any[]): T {
 return new AnyClass(...args);
}
```

`createInstance()`的使用方法如下：

```ts
class Person {
 constructor(public name: string) {}
}

// %inferred-type: Person
const jane = createInstance(Person, 'Jane');
```

`createInstance()`是`new`操作符，通过一个函数实现。

#### 18.3.2 示例：带有运行时检查的类型转换

我们可以使用`Class<T>`来实现类型转换：

```ts
function cast<T>(AnyClass: Class<T>, obj: any): T {
 if (! (obj instanceof AnyClass)) {
 throw new Error(`Not an instance of ${AnyClass.name}: ${obj}`)
 }
 return obj;
}
```

通过`cast()`，我们可以将值的类型更改为更具体的类型。这在运行时也是安全的，因为我们既静态地更改了类型，又执行了动态检查。以下代码提供了一个示例：

```ts
function parseObject(jsonObjectStr: string): Object {
 // %inferred-type: any
 const parsed = JSON.parse(jsonObjectStr);
 return cast(Object, parsed);
}
```

#### 18.3.3 示例：在运行时类型安全的映射

`Class<T>`和`cast()`的一个用例是类型安全的映射：

```ts
class TypeSafeMap {
 #data = new Map<any, any>();
 get<T>(key: Class<T>) {
 const value = this.#data.get(key);
 return cast(key, value);
 }
 set<T>(key: Class<T>, value: T): this {
 cast(key, value); // runtime check
 this.#data.set(key, value);
 return this;
 }
 has(key: any) {
 return this.#data.has(key);
 }
}
```

`TypeSafeMap`中每个条目的键都是一个类。该类确定条目值的静态类型，并且在运行时用于检查。

这是`TypeSafeMap`的实际应用：

```ts
const map = new TypeSafeMap();

map.set(RegExp, /abc/);

// %inferred-type: RegExp
const re = map.get(RegExp);

// Static and dynamic error!
assert.throws(
 // @ts-expect-error: Argument of type '"abc"' is not assignable
 // to parameter of type 'Date'.
 () => map.set(Date, 'abc'));
```

#### 18.3.4 陷阱：`Class<T>`与抽象类不匹配

当期望`Class<T>`时，我们不能使用抽象类：

```ts
abstract class Shape {
}
class Circle extends Shape {
 // ···
}

// @ts-expect-error: Type 'typeof Shape' is not assignable to type
// 'Class<Shape>'.
//   Cannot assign an abstract constructor type to a non-abstract
//   constructor type. (2322)
const shapeClasses1: Array<Class<Shape>> = [Circle, Shape];
```

为什么呢？原因是构造函数类型文字和构造签名应该只用于实际可以被`new`调用的值（[GitHub 上有更多信息的问题](https://github.com/microsoft/TypeScript/issues/5843)）。

这是一个变通方法：

```ts
type Class2<T> = Function & {prototype: T};

const shapeClasses2: Array<Class2<Shape>> = [Circle, Shape];
```

这种方法的缺点：

+   稍微令人困惑。

+   具有此类型的值不能用于`instanceof`检查（作为右操作数）。

[评论](https://github.com/rauschma/tackling-ts/issues/18)
