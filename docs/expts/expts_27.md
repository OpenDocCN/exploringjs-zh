# 22   与类相关的类型

> 原文：[`exploringjs.com/ts/book/ch_class-related-types.html`](https://exploringjs.com/ts/book/ch_class-related-types.html)

（广告，请勿拦截。）

1.  22.1   类的两个原型链

1.  22.2   类实例的接口

1.  22.3   类的接口

    1.  22.3.1   示例：从 JSON 转换到和从 JSON 转换

    1.  22.3.2   示例：TypeScript 为类 `Object` 及其实例提供的内置接口

1.  22.4   类作为类型

    1.  22.4.1   陷阱：类按结构工作，而不是按名义

1.  22.5   进一步阅读

在本章中，我们探讨与类及其实例相关的类型。

### 22.1   类的两个原型链

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

![](img/f0091b5ecd0cda2b8e4df5b426d81e52.png)

图 22.1：由类 `Counter` 创建的对象。左侧：类及其超类 `Object`。右侧：实例 `myCounter`、`Counter` 的原型属性和超类 `Object` 的原型方法。

图 22.1 显示了类 `Counter` 的运行时结构。图中包含两个对象的原型链：

+   类（左侧）：类的静态原型链由构成类 `Counter` 的对象组成。类 `Counter` 的原型对象是其超类 `Object`。

+   实例（右侧）：实例原型链由构成实例 `myCounter` 的对象组成。链从实例 `myCounter` 开始，接着是 `Counter.prototype`（包含类 `Counter` 的原型方法）和 `Object.prototype`（包含类 `Object` 的原型方法）。

在本章中，我们将首先探讨实例对象，然后是作为对象的类。

### 22.2   类实例的接口

接口指定对象提供的服务。例如：

```ts
interface CountingService {
 value: number;
 increment(): void;
}

```

TypeScript 的接口按结构工作结构化：为了使一个对象实现接口，它只需要具有正确的属性和正确的类型。我们可以在以下示例中看到这一点：

```ts
const myCounter2: CountingService = new Counter(3);

```

结构化接口很方便，因为我们甚至可以为已经存在的对象创建接口（即，我们可以在事后引入它们）。

如果我们事先知道一个对象必须实现给定的接口，那么在后期避免意外通常是有意义的，我们可以通过 `implements` 为类的实例这样做：

```ts
class Counter implements CountingService {
 // ···
};

```

注释：

+   我们可以实现任何对象类型（不仅仅是接口）。

+   TypeScript 不区分继承属性（如`.increment`）和自身属性（如`.value`）。

+   作为旁注，接口会忽略私有属性，并且不能通过它们来指定。考虑到私有数据仅用于内部目的，这是预期的。

### 22.3 类接口

类本身也是对象（函数）。因此，我们可以使用接口来指定它们的属性。这里的主要用例是描述对象的工厂。下一节将给出一个示例。

#### 22.3.1 示例：从 JSON 转换为以及转换为 JSON

以下两个接口可以用于支持其实例从 JSON 转换为以及转换为 JSON 的类：

```ts
// Converting JSON to instances
interface JsonStatic {
 fromJson(json: unknown): JsonInstance;
}

// Converting instances to JSON
interface JsonInstance {
 toJson(): unknown;
}

```

我们在以下代码中使用这些接口：

```ts
class Person implements JsonInstance {
 static fromJson(json: unknown): Person {
 if (typeof json !== 'string') {
 throw new TypeError();
 }
 return new Person(json);
 }
 name: string;
 constructor(name: string) {
 this.name = name;
 }
 toJson(): unknown {
 return this.name;
 }
}

```

这就是我们如何立即检查类`Person`（作为一个对象）是否实现了接口`JsonStatic`：

```ts
type _ = Assert<Assignable<JsonStatic, typeof Person>>;

```

如果你不想为此目的使用库（带有`Assert`和`Assignable`实用类型），你可以使用以下模式：

```ts
// Assign the class to a type-annotated variable
const personImplementsJsonStatic: JsonStatic = Person;

```

这种模式的缺点是它会产生额外的 JavaScript 代码。

##### 22.3.1.1 我们能做得更好吗？

避免外部检查会更好——例如，像这样：

```ts
const Person = class implements JsonInstance {
 static fromJson(json: unknown): Person { // (A)
 // ···
 }
 // ···
} satisfies JsonStatic; // (B)
type Person = typeof Person.prototype; // (C)

```

在行 B 中，我们使用了`satisfies`运算符，它强制值`Person`可以赋值给`JsonStatic`，同时保留该值的类型。这很重要，因为`Person`不应该仅限于`JsonStatic`中定义的内容。

可惜，这种替代方法甚至更加冗长，并且无法编译。编译器错误之一在行 C：

> 类型别名'Person'循环引用自身。

为什么？在行 A 中提到了`Person`类型。即使我们将类型`Person`重命名为`TPerson`，这个错误也不会消失。

#### 22.3.2 示例：TypeScript 为`Object`类及其实例提供的内置接口

看一下 TypeScript 的内置类型是有教育意义的：

一方面，接口`ObjectConstructor`是用于由全局变量`Object`指向的类：

```ts
declare var Object: ObjectConstructor;
interface ObjectConstructor {
 /** Invocation via `new` */
 new(value?: any): Object; // (A)
 /** Invocation via function calls */
 (value?: any): any;

 readonly prototype: Object; // (B)

 getPrototypeOf(o: any): any;
 // ···
}

```

另一方面，接口`Object`（在行 A 和行 B 中提到）是用于`Object`的实例：

```ts
interface Object {
 constructor: Function;
 toString(): string;
 toLocaleString(): string;
 valueOf(): Object;
 hasOwnProperty(v: PropertyKey): boolean;
 isPrototypeOf(v: Object): boolean;
 propertyIsEnumerable(v: PropertyKey): boolean;
}

```

换句话说——`Object`这个名字在两个不同的语言级别被使用：

+   在动态级别，对于全局变量。

+   在静态级别，对于类型。

### 22.4 类作为类型

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

首先，一个名为`Color`的构造函数（可以通过`new`来调用）：

```ts
assert.equal(
 typeof Color, 'function'
);

```

其次，一个名为`Color`的接口，它与`Color`的实例相匹配：

```ts
const green: Color = new Color('green');

```

这里是`Color`确实是一个接口的证明：

```ts
interface RgbColor extends Color {
 rgbValue: [number, number, number];
}

```

#### 22.4.1 陷阱：类在结构上工作，而不是名义上

尽管如此，有一个陷阱：将`Color`用作静态类型不是一个非常严格的检查：

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

为什么 TypeScript 在行 A 中没有抱怨？这是因为结构化类型：`Person`和`Color`的实例具有相同的结构，因此它们在静态上是兼容的。

##### 22.4.1.1 关闭结构化类型

我们可以通过添加一个私有字段（或一个`private`属性）将`Color`转换为一个命名类型：

```ts
class Color {
 name: string;
 #isBranded = true;
 constructor(name: string) {
 this.name = name;
 }
}
class Person {
 name: string;
 #isBranded = true;
 constructor(name: string) {
 this.name = name;
 }
}

const robin: Person = new Person('Robin');
// @ts-expect-error: Type 'Person' is not assignable to type 'Color'.
// Property '#isBranded' in type 'Person' refers to a different member that
// cannot be accessed from within type 'Color'.
const color: Color = robin;

```

这种关闭结构化类型的方式被称为*品牌化*。请注意，尽管`Color`和`Person`的私有字段具有相同的名称和类型，但它们是不兼容的。这反映了 JavaScript 的工作方式：我们不能从`Person`访问`Color`的私有字段，反之亦然。

##### 22.4.1.2 使用品牌化案例：从对象类型迁移到类

假设我们想要将以下代码从行 A 中的对象类型迁移到一个类中：

```ts
type Person = { // (A)
 name: string,
};

function storePerson(person: Person): void {
 // ...
}

storePerson({
 name: 'Robin',
});

```

在我们的第一次尝试中，使用对象字面量调用`storePerson()`仍然有效：

```ts
class Person {
 name: string;
 constructor(name: string) {
 this.name = name;
 }
}

function storePerson(person: Person): void {
 // ...
}

storePerson({
 name: 'Robin',
});

```

一旦我们对`Person`进行品牌化，就会得到一个编译器错误：

```ts
class Person {
 name: string;
 #isBranded = true;
 constructor(name: string) {
 this.name = name;
 }
}

function storePerson(person: Person): void {
 // ...
}

// @ts-expect-error: Argument of type '{ name: string; }' is not assignable
// to parameter of type 'Person'. Property '#isBranded' is missing in type
// '{ name: string; }' but required in type 'Person'.
storePerson({
 name: 'Robin',
});

```

这就是我们修复错误的方法：

```ts
storePerson(
 new Person('Robin')
);

```

### 22.5 进一步阅读

+   “探索 JavaScript”中的[“类”章节](https://exploringjs.com/js/book/ch_classes.html)
