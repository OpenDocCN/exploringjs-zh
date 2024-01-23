# 16 TypeScript 中的类定义

> 原文：[`exploringjs.com/tackling-ts/ch_class-definitions.html`](https://exploringjs.com/tackling-ts/ch_class-definitions.html)

* * *

+   16.1 速查表：纯 JavaScript 中的类

    +   16.1.1 类的基本成员

    +   16.1.2 修饰符：`static`

    +   16.1.3 类似修饰符的名称前缀：`#`（私有）

    +   16.1.4 访问器的修饰符：`get`（getter）和`set`（setter）

    +   16.1.5 方法的修饰符：`*`（生成器）

    +   16.1.6 方法的修饰符：`async`

    +   16.1.7 计算类成员名称

    +   16.1.8 修饰符的组合

    +   16.1.9 底层原理

    +   16.1.10 纯 JavaScript 中类定义的更多信息

+   16.2 TypeScript 中的非公共数据槽

    +   16.2.1 私有属性

    +   16.2.2 私有字段

    +   16.2.3 私有属性 vs. 私有字段

    +   16.2.4 受保护的属性

+   16.3 私有构造函数 (ch_class-definitions.html#private-constructors)

+   16.4 初始化实例属性

    +   16.4.1 严格的属性初始化

    +   16.4.2 使构造函数参数`public`、`private`或`protected`

+   16.5 抽象类

* * *

在本章中，我们将研究 TypeScript 中类定义的工作方式：

+   首先，我们快速查看纯 JavaScript 中类定义的特性。

+   然后我们探讨 TypeScript 为此带来了哪些新增内容。

### 16.1 速查表：纯 JavaScript 中的类

本节是关于纯 JavaScript 中类定义的速查表。

#### 16.1.1 类的基本成员

```ts
class OtherClass {}

class MyClass1 extends OtherClass {

 publicInstanceField = 1;

 constructor() {
 super();
 }

 publicPrototypeMethod() {
 return 2;
 }
}

const inst1 = new MyClass1();
assert.equal(inst1.publicInstanceField, 1);
assert.equal(inst1.publicPrototypeMethod(), 2);
```

![](img/65d35c0a2478236e12cc4321e1b02db6.png)  **接下来的部分是关于修饰符的**

最后，有一张表显示了修饰符如何组合。

#### 16.1.2 修饰符：`static`

```ts
class MyClass2 {

 static staticPublicField = 1;

 static staticPublicMethod() {
 return 2;
 }
}

assert.equal(MyClass2.staticPublicField, 1);
assert.equal(MyClass2.staticPublicMethod(), 2);
```

#### 16.1.3 类似修饰符的名称前缀：`#`（私有）

```ts
class MyClass3 {
 #privateField = 1;

 #privateMethod() {
 return 2;
 }

 static accessPrivateMembers() {
 // Private members can only be accessed from inside class definitions
 const inst3 = new MyClass3();
 assert.equal(inst3.#privateField, 1);
 assert.equal(inst3.#privateMethod(), 2);
 }
}
MyClass3.accessPrivateMembers();
```

JavaScript 警告：

+   [目前对私有方法的支持非常有限。](https://github.com/tc39/proposal-private-methods#implementations)

+   [私有字段有更广泛的支持，但也有限制。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Class_fields#Private_class_fields)

TypeScript 自 3.8 版本以来一直支持私有字段，但目前不支持私有方法。

#### 16.1.4 访问器的修饰符：`get`（getter）和`set`（setter）

大致上，访问器是通过访问属性调用的方法。有两种类型的访问器：getter 和 setter。

```ts
class MyClass5 {
 #name = 'Rumpelstiltskin';

 /** Prototype getter */
 get name() {
 return this.#name;
 }

 /** Prototype setter */
 set name(value) {
 this.#name = value;
 }
}
const inst5 = new MyClass5();
assert.equal(inst5.name, 'Rumpelstiltskin'); // getter
inst5.name = 'Queen'; // setter
assert.equal(inst5.name, 'Queen'); // getter
```

#### 16.1.5 方法的修饰符：`*`（生成器）

```ts
class MyClass6 {
 * publicPrototypeGeneratorMethod() {
 yield 'hello';
 yield 'world';
 }
}

const inst6 = new MyClass6();
assert.deepEqual(
 [...inst6.publicPrototypeGeneratorMethod()],
 ['hello', 'world']);
```

#### 16.1.6 方法的修饰符：`async`

```ts
class MyClass7 {
 async publicPrototypeAsyncMethod() {
 const result = await Promise.resolve('abc');
 return result + result;
 }
}

const inst7 = new MyClass7();
inst7.publicPrototypeAsyncMethod()
 .then(result => assert.equal(result, 'abcabc'));
```

#### 16.1.7 计算类成员名称

```ts
const publicInstanceFieldKey = Symbol('publicInstanceFieldKey');
const publicPrototypeMethodKey = Symbol('publicPrototypeMethodKey');

class MyClass8 {

 [publicInstanceFieldKey] = 1;

 [publicPrototypeMethodKey]() {
 return 2;
 }
}

const inst8 = new MyClass8();
assert.equal(inst8[publicInstanceFieldKey], 1);
assert.equal(inst8[publicPrototypeMethodKey](), 2);
```

评论：

+   此功能的主要用例是诸如`Symbol.iterator`之类的符号。但是任何表达式都可以在方括号内使用。

+   我们可以计算字段、方法和访问器的名称。

+   我们无法计算私有成员的名称（这些名称始终是固定的）。

#### 16.1.8 修饰符的组合

字段（没有级别意味着构造存在于实例级别）：

| 级别 | 可见性 |
| --- | --- |
| (实例) |  |
| (实例) | `#` |
| `static` |  |
| `static` | `#` |

方法（没有级别表示构造存在于原型级别）：

| 级别 | 访问器 | 异步 | 生成器 | 可见性 |
| --- | --- | --- | --- | --- |
| (原型) |  |  |  |  |
| (原型) | `get` |  |  |  |
| (原型) | `set` |  |  |  |
| (原型) |  | `async` |  |  |
| (原型) |  |  | `*` |  |
| (原型) |  | `async` | `*` |  |
| (与原型相关) |  |  |  | `#` |
| (与原型相关) | `get` |  |  | `#` |
| (与原型相关) | `set` |  |  | `#` |
| (与原型相关) |  | `async` |  | `#` |
| (与原型相关) |  |  | `*` | `#` |
| (与原型相关) |  | `async` | `*` | `#` |
| `static` |  |  |  |  |
| `static` | `get` |  |  |  |
| `static` | `set` |  |  |  |
| `static` |  | `async` |  |  |
| `static` |  |  | `*` |  |
| `static` |  | `async` | `*` |  |
| `static` |  |  |  | `#` |
| `static` | `get` |  |  | `#` |
| `static` | `set` |  |  | `#` |
| `static` |  | `async` |  | `#` |
| `static` |  |  | `*` | `#` |
| `static` |  | `async` | `*` | `#` |

方法的限制：

+   访问器不能是异步的或生成器。

#### 16.1.9 底层

重要的是要记住，对于类，有两条原型对象链：

+   以一个实例开始的实例链。

+   从该实例的类开始的静态链。

考虑以下纯 JavaScript 示例：

```ts
class ClassA {
 static staticMthdA() {}
 constructor(instPropA) {
 this.instPropA = instPropA;
 }
 prototypeMthdA() {}
}
class ClassB extends ClassA {
 static staticMthdB() {}
 constructor(instPropA, instPropB) {
 super(instPropA);
 this.instPropB = instPropB;
 }
 prototypeMthdB() {}
}
const instB = new ClassB(0, 1);
```

图 1 显示了由 `ClassA` 和 `ClassB` 创建的原型链的样子。

![图 1：`ClassA` 和 `ClassB` 创建了两条原型链：一条是类的（左侧），一条是实例的（右侧）。](img/0665e9f0d4e7a5093d20ad851c81a01c.png)

图 1：`ClassA` 和 `ClassB` 创建了两条原型链：一条是类的（左侧），一条是实例的（右侧）。

#### 16.1.10 纯 JavaScript 中类定义的更多信息

+   [公共字段、私有字段、私有方法/获取器/设置器](https://2ality.com/2019/07/public-class-fields.html)（博客文章）

+   [所有剩余的 JavaScript 类特性](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html)（“JavaScript for impatient programming”中的章节）

### 16.2 TypeScript 中的非公共数据槽

在 TypeScript 中，默认情况下，所有数据槽都是公共属性。有两种方法可以保持数据私有：

+   私有属性

+   私有字段

我们接下来会看两者。

请注意，TypeScript 目前不支持私有方法。

#### 16.2.1 私有属性

私有属性是 TypeScript 专有的（静态）特性。通过在关键字 `private`（A 行）前面加上前缀，任何属性都可以被设置为私有的：

```ts
class PersonPrivateProperty {
 private name: string; // (A)
 constructor(name: string) {
 this.name = name;
 }
 sayHello() {
 return `Hello ${this.name}!`;
 }
}
```

现在，如果我们在错误的范围内访问该属性，我们会得到编译时错误（A 行）：

```ts
const john = new PersonPrivateProperty('John');

assert.equal(
 john.sayHello(), 'Hello John!');

// @ts-expect-error: Property 'name' is private and only accessible
// within class 'PersonPrivateProperty'. (2341)
john.name; // (A)
```

然而，`private` 在运行时不会改变任何东西。在那里，属性`.name` 与公共属性无法区分：

```ts
assert.deepEqual(
 Object.keys(john),
 ['name']);
```

当我们查看类编译成的 JavaScript 代码时，我们还可以看到私有属性在运行时不受保护：

```ts
class PersonPrivateProperty {
 constructor(name) {
 this.name = name;
 }
 sayHello() {
 return `Hello ${this.name}!`;
 }
}
```

#### 16.2.2 私有字段

私有字段是 TypeScript 自 3.8 版本以来支持的新 JavaScript 特性：

```ts
class PersonPrivateField {
 #name: string;
 constructor(name: string) {
 this.#name = name;
 }
 sayHello() {
 return `Hello ${this.#name}!`;
 }
}
```

这个版本的 `Person` 大部分用法与私有属性版本相同：

```ts
const john = new PersonPrivateField('John');

assert.equal(
 john.sayHello(), 'Hello John!');
```

然而，这次，数据完全封装起来了。在类外部使用私有字段语法甚至是 JavaScript 语法错误。这就是为什么我们必须在 A 行使用 `eval()`，以便我们可以执行这段代码：

```ts
assert.throws(
 () => eval('john.#name'), // (A)
 {
 name: 'SyntaxError',
 message: "Private field '#name' must be declared in "
 + "an enclosing class",
 });

assert.deepEqual(
 Object.keys(john),
 []);
```

编译结果现在更加复杂（稍微简化）：

```ts
var __classPrivateFieldSet = function (receiver, privateMap, value) {
 if (!privateMap.has(receiver)) {
 throw new TypeError(
 'attempted to set private field on non-instance');
 }
 privateMap.set(receiver, value);
 return value;
};

// Omitted: __classPrivateFieldGet

var _name = new WeakMap();
class Person {
 constructor(name) {
 // Add an entry for this instance to _name
 _name.set(this, void 0);

 // Now we can use the helper function:
 __classPrivateFieldSet(this, _name, name);
 }
 // ···
}
```

这段代码使用了一个保持实例数据私有的常见技术：

+   每个 WeakMap 实现一个私有字段。

+   它将每个实例与一个私有数据关联起来。

关于这个主题的更多信息：请参阅[“JavaScript for impatient programmers”](https://exploringjs.com/impatient-js/ch_weakmaps.html#private-data-in-weakmaps)。

#### 16.2.3 私有属性 vs. 私有字段

+   私有属性的缺点：

    +   我们不能在子类中重用私有属性的名称（因为属性在运行时不是私有的）。

    +   在运行时没有封装。

+   私有属性的优点：

    +   客户端可以规避封装并访问私有属性。如果有人需要解决 bug，这可能是有用的。换句话说：数据完全封装有利有弊。

#### 16.2.4 受保护的属性

私有字段和私有属性不能在子类中访问（A 行）：

```ts
class PrivatePerson {
 private name: string;
 constructor(name: string) {
 this.name = name;
 }
 sayHello() {
 return `Hello ${this.name}!`;
 }
}
class PrivateEmployee extends PrivatePerson {
 private company: string;
 constructor(name: string, company: string) {
 super(name);
 this.company = company;
 }
 sayHello() {
 // @ts-expect-error: Property 'name' is private and only
 // accessible within class 'PrivatePerson'. (2341)
 return `Hello ${this.name} from ${this.company}!`; // (A)
 } 
}
```

我们可以通过在 A 行将`private`改为`protected`来修复上一个示例（出于一致性的考虑，我们也在 B 行进行了切换）：

```ts
class ProtectedPerson {
 protected name: string; // (A)
 constructor(name: string) {
 this.name = name;
 }
 sayHello() {
 return `Hello ${this.name}!`;
 }
}
class ProtectedEmployee extends ProtectedPerson {
 protected company: string; // (B)
 constructor(name: string, company: string) {
 super(name);
 this.company = company;
 }
 sayHello() {
 return `Hello ${this.name} from ${this.company}!`; // OK
 } 
}
```

### 16.3 私有构造函数

构造函数也可以是私有的。当我们有静态工厂方法并且希望客户端始终使用这些方法而不是直接使用构造函数时，这是很有用的。静态方法可以访问私有类成员，这就是为什么工厂方法仍然可以使用构造函数的原因。

在以下代码中，有一个静态工厂方法`DataContainer.create()`。它通过异步加载的数据设置实例。将异步代码放在工厂方法中使得实际类完全同步：

```ts
class DataContainer {
 #data: string;
 static async create() {
 const data = await Promise.resolve('downloaded'); // (A)
 return new this(data);
 }
 private constructor(data: string) {
 this.#data = data;
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}
DataContainer.create()
 .then(dc => assert.equal(
 dc.getData(), 'DATA: downloaded'));
```

在实际代码中，我们会使用`fetch()`或类似的基于 Promise 的 API 来在 A 行异步加载数据。

私有构造函数防止`DataContainer`被子类化。如果我们想允许子类，我们必须将其设置为`protected`。

### 16.4 初始化实例属性

#### 16.4.1 严格的属性初始化

如果编译器设置`--strictPropertyInitialization`被打开（如果我们使用`--strict`，则是这种情况），那么 TypeScript 会检查所有声明的实例属性是否被正确初始化：

+   要么通过构造函数中的赋值：

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

+   或通过属性声明的初始化程序：

    ```ts
    class Point {
     x = 0;
     y = 0;

     // No constructor needed
    }
    ```

然而，有时我们以 TypeScript 无法识别的方式初始化属性。然后我们可以使用感叹号（*确定赋值断言*）来关闭 TypeScript 的警告（A 行和 B 行）：

```ts
class Point {
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

##### 16.4.1.1 例子：通过对象设置实例属性

在下面的示例中，我们还需要确定赋值断言。在这里，我们通过构造函数参数`props`设置实例属性。

```ts
class CompilerError implements CompilerErrorProps { // (A)
 line!: number;
 description!: string;
 constructor(props: CompilerErrorProps) {
 Object.assign(this, props); // (B)
 }
}

// Helper interface for the parameter properties
interface CompilerErrorProps {
 line: number,
 description: string,
}

// Using the class:
const err = new CompilerError({
 line: 123,
 description: 'Unexpected token',
});
```

注：

+   在 B 行，我们初始化了所有属性：我们使用`Object.assign()`将参数`props`的属性复制到`this`中。

+   在 A 行，`implements`确保类声明了接口`CompilerErrorProps`中的所有属性。

#### 16.4.2 使构造函数参数`public`，`private`或`protected`

如果我们对构造函数参数使用关键字`public`，那么 TypeScript 会为我们做两件事：

+   它声明了一个具有相同名称的公共实例属性。

+   它将参数分配给该实例属性。

因此，以下两个类是等价的：

```ts
class Point1 {
 constructor(public x: number, public y: number) {
 }
}

class Point2 {
 x: number;
 y: number;
 constructor(x: number, y: number) {
 this.x = x;
 this.y = y;
 }
}
```

如果我们使用`private`或`protected`而不是`public`，那么相应的实例属性就是私有的或受保护的（而不是公共的）。

### 16.5 抽象类

在 TypeScript 中，两个构造可以是抽象的：

+   抽象类不能被实例化。只有它的子类可以——如果它们自己不是抽象的。

+   抽象方法没有实现，只有类型签名。每个具体的子类必须具有相同名称和兼容类型签名的具体方法。

    +   如果一个类有任何抽象方法，它也必须是抽象的。

以下代码演示了抽象类和方法。

一方面，有一个抽象的超类`Printable`及其辅助类`StringBuilder`：

```ts
class StringBuilder {
 string = '';
 add(str: string) {
 this.string += str;
 }
}
abstract class Printable {
 toString() {
 const out = new StringBuilder();
 this.print(out);
 return out.string;
 }
 abstract print(out: StringBuilder): void;
}
```

另一方面，有具体的子类`Entries`和`Entry`：

```ts
class Entries extends Printable {
 entries: Entry[];
 constructor(entries: Entry[]) {
 super();
 this.entries = entries;
 }
 print(out: StringBuilder): void {
 for (const entry of this.entries) {
 entry.print(out);
 }
 }
}
class Entry extends Printable {
 key: string;
 value: string;
 constructor(key: string, value: string) {
 super();
 this.key = key;
 this.value = value;
 }
 print(out: StringBuilder): void {
 out.add(this.key);
 out.add(': ');
 out.add(this.value);
 out.add('\n');
 }
}
```

最后，这是我们使用`Entries`和`Entry`：

```ts
const entries = new Entries([
 new Entry('accept-ranges', 'bytes'),
 new Entry('content-length', '6518'),
]);
assert.equal(
 entries.toString(),
 'accept-ranges: bytes\ncontent-length: 6518\n');
```

关于抽象类的注释：

+   抽象类可以被视为具有一些成员已经有实现的接口。

+   虽然一个类可以实现多个接口，但它最多只能扩展一个抽象类。

+   “抽象性”只存在于编译时。在运行时，抽象类是普通类，抽象方法不存在（因为它们只提供编译时信息）。

+   抽象类可以被视为模板，其中每个抽象方法都是必须由子类填写（实现）的空白。

[评论](https://github.com/rauschma/tackling-ts/issues/16)
