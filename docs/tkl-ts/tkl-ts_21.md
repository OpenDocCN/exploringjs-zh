# 十七、与类相关的类型

> 原文：[`exploringjs.com/tackling-ts/ch_class-related-types.html`](https://exploringjs.com/tackling-ts/ch_class-related-types.html)

* * *

+   17.1 类的两个原型链

+   17.2 类的实例接口

+   17.3 类的接口

    +   17.3.1 示例：从 JSON 转换和转换为 JSON

    +   17.3.2 示例：TypeScript 内置接口用于类`Object`及其实例

+   17.4 类作为类型

    +   17.4.1 陷阱：类的结构工作，而不是名义上的

+   17.5 进一步阅读

* * *

在这一章关于 TypeScript 的内容中，我们研究与类及其实例相关的类型。

### 17.1 类的两个原型链

考虑这个类：

```ts
class Counter extends Object {
 static createZero() {
 return new Counter(0);
 }
 value: number;
 constructor(value: number) {
 super();
 this.value = value;
 }
 increment() {
 this.value++;
 }
}
// Static method
const myCounter = Counter.createZero();
assert.ok(myCounter instanceof Counter);
assert.equal(myCounter.value, 0);

// Instance method
myCounter.increment();
assert.equal(myCounter.value, 1);
```

![图 2：类 Counter 创建的对象。左侧：类及其超类 Object。右侧：实例 myCounter，Counter 的原型属性和超类 Object 的原型方法。](img/40fe7734ba7c30ae1823aa567ec39cc3.png)

图 2：类`Counter`创建的对象。左侧：类及其超类`Object`。右侧：实例`myCounter`，`Counter`的原型属性和超类`Object`的原型方法。

图 2 中的图表显示了类`Counter`的运行时结构。在这个图表中有两个对象的原型链：

+   类（左侧）：静态原型链由组成类`Counter`的对象组成。类`Counter`的原型对象是它的超类`Object`。

+   实例（右侧）：实例原型链由组成实例`myCounter`的对象组成。链以实例`myCounter`开始，然后是`Counter.prototype`（其中包含类`Counter`的原型方法）和`Object.prototype`（其中包含类`Object`的原型方法）。

在本章中，我们首先探讨实例对象，然后是作为对象的类。

### 17.2 类的实例接口

接口指定对象提供的服务。例如：

```ts
interface CountingService {
 value: number;
 increment(): void;
}
```

TypeScript 的接口是结构化的：为了使一个对象实现一个接口，它只需要具有正确类型的正确属性。我们可以在下面的例子中看到这一点：

```ts
const myCounter2: CountingService = new Counter(3);
```

结构接口很方便，因为我们甚至可以为已经存在的对象创建接口（即，在事后引入它们）。

如果我们提前知道一个对象必须实现一个给定的接口，通常最好提前检查它是否实现了，以避免后来的意外。我们可以通过`implements`来对类的实例进行这样的检查：

```ts
class Counter implements CountingService {
 // ···
};
```

注：

+   TypeScript 不区分继承的属性（如`.increment`）和自有属性（如`.value`）。

+   另外，接口忽略私有属性，并且不能通过接口指定私有属性。这是可以预料的，因为私有数据仅供内部使用。

### 17.3 类的接口

类本身也是对象（函数）。因此，我们可以使用接口来指定它们的属性。这里的主要用例是描述对象的工厂。下一节给出了一个例子。

#### 17.3.1 示例：从 JSON 转换和转换为 JSON

以下两个接口可用于支持其实例从 JSON 转换和转换为 JSON 的类：

```ts
// Converting JSON to instances
interface JsonStatic {
 fromJson(json: any): JsonInstance;
}

// Converting instances to JSON
interface JsonInstance {
 toJson(): any;
}
```

我们在下面的代码中使用这些接口：

```ts
class Person implements JsonInstance {
 static fromJson(json: any): Person {
 if (typeof json !== 'string') {
 throw new TypeError(json);
 }
 return new Person(json);
 }
 name: string;
 constructor(name: string) {
 this.name = name;
 }
 toJson(): any {
 return this.name;
 }
}
```

这是我们可以立即检查类`Person`（作为对象）是否实现了接口`JsonStatic`的方法：

```ts
// Assign the class to a type-annotated variable
const personImplementsJsonStatic: JsonStatic = Person;
```

以下方式进行此检查可能看起来是一个好主意：

```ts
const Person: JsonStatic = class implements JsonInstance {
 // ···
};
```

然而，这并不真正起作用：

+   我们不能`new`-call `Person`，因为`JsonStatic`没有构造签名。

+   如果`Person`具有超出`.fromJson()`的静态属性，TypeScript 不会让我们访问它们。

#### 17.3.2 示例：TypeScript 的内置接口用于类`Object`及其实例

看一下 TypeScript 内置类型是很有启发性的：

一方面，接口`ObjectConstructor`是为了类`Object`本身：

```ts
/**
 * Provides functionality common to all JavaScript objects.
 */
declare var Object: ObjectConstructor;

interface ObjectConstructor {
 new(value?: any): Object;
 (): any;
 (value: any): any;

 /** A reference to the prototype for a class of objects. */
 readonly prototype: Object;

 /**
 * Returns the prototype of an object.
 * @param  o The object that references the prototype.
 */
 getPrototypeOf(o: any): any;

}
```

另一方面，接口`Object`是为了`Object`的实例：

```ts
interface Object {
 /** The initial value of Object.prototype.constructor is the standard built-in Object constructor. */
 constructor: Function;

 /** Returns a string representation of an object. */
 toString(): string;
}
```

名称`Object`在两个不同的语言级别上都被使用了：

+   在动态级别，对于一个全局变量。

+   在静态级别，对于一个类型。

### 17.4 类作为类型

考虑以下类：

```ts
class Color {
 name: string;
 constructor(name: string) {
 this.name = name;
 }
}
```

这个类定义创建了两个东西。

首先，一个名为`Color`的构造函数（可以通过`new`调用）：

```ts
assert.equal(
 typeof Color, 'function')
```

其次，一个名为`Color`的接口，匹配`Color`的实例：

```ts
const green: Color = new Color('green');
```

这里有证据表明`Color`确实是一个接口：

```ts
interface RgbColor extends Color {
 rgbValue: [number, number, number];
}
```

#### 17.4.1 陷阱：类在结构上工作，而不是名义上。

不过有一个陷阱：使用`Color`作为静态类型并不是一个非常严格的检查：

```ts
class Color {
 name: string;
 constructor(name: string) {
 this.name = name;
 }
}
class Person {
 name: string;
 constructor(name: string) {
 this.name = name;
 }
}

const person: Person = new Person('Jane');
const color: Color = person; // (A)
```

为什么 TypeScript 在 A 行没有抱怨呢？这是由于结构类型：`Person`和`Color`的实例具有相同的结构，因此在静态上是兼容的。

##### 17.4.1.1 关闭结构类型

我们可以通过添加私有属性使这两组对象不兼容：

```ts
class Color {
 name: string;
 private branded = true;
 constructor(name: string) {
 this.name = name;
 }
}
class Person {
 name: string;
 private branded = true;
 constructor(name: string) {
 this.name = name;
 }
}

const person: Person = new Person('Jane');

// @ts-expect-error: Type 'Person' is not assignable to type 'Color'.
//   Types have separate declarations of a private property
//   'branded'. (2322)
const color: Color = person;
```

这种情况下，私有属性关闭了结构类型。

### 17.5 进一步阅读

+   [章节“原型链和类”](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html) 在“JavaScript for impatient programmers”

[评论](https://github.com/rauschma/tackling-ts/issues/17)
