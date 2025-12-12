# 21 TypeScript 中的类定义

> 原文：[`exploringjs.com/ts/book/ch_class-definitions.html`](https://exploringjs.com/ts/book/ch_class-definitions.html)

(Ad, please don’t block.)

1.  21.1 纯 JavaScript 中的类速查表

    1.  21.1.1 类的基本成员

    1.  21.1.2 修饰符：`static`

    1.  21.1.3 类似修饰符的名称前缀：`#` (私有)

    1.  21.1.4 访问器修饰符：`get` (getter) 和 `set` (setter)

    1.  21.1.5 方法修饰符：`*` (生成器)

    1.  21.1.6 方法修饰符：`async`

    1.  21.1.7 计算类成员名称

    1.  21.1.8 修饰符的组合

    1.  21.1.9 内部机制

    1.  21.1.10 关于纯 JavaScript 中类定义的更多信息

1.  21.2 TypeScript 中的非公共数据槽

    1.  21.2.1 私有属性

    1.  21.2.2 私有字段

    1.  21.2.3 私有属性与私有字段

    1.  21.2.4 受保护的属性

1.  21.3 私有构造函数

1.  21.4 初始化实例属性

    1.  21.4.1 严格的属性初始化

1.  21.5 我们应该避免的便利特性

    1.  21.5.1 推断成员类型

    1.  21.5.2 将构造函数参数设置为 `public`、`private` 或 `protected`

1.  21.6 抽象类

1.  21.7 方法关键字 `override`

1.  21.8 类与对象类型

    1.  21.8.1 类 `Counter`

    1.  21.8.2 对象类型 `Counter`

    1.  21.8.3 选择类或对象类型？

在本章中，我们探讨 TypeScript 中类定义的工作方式：

+   首先，我们快速浏览一下纯 JavaScript 中类定义的特性。

+   然后我们探讨 TypeScript 带来了哪些新增特性。

### 21.1 纯 JavaScript 中的类速查表

本节是关于纯 JavaScript 中类定义的速查表。

#### 21.1.1 类的基本成员

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

![阅读图标](img/00b0d6029a045810b908b88d1a6733d2.png) **下一节将介绍修饰符**  

最后，有一个表格显示了修饰符如何组合。

#### 21.1.2 修饰符：`static`

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

#### 21.1.3 类似修饰符的名称前缀：`#` (私有)

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

```

#### 21.1.4 访问器的修饰符：`get` (获取器) 和 `set` (设置器)

大概来说，访问器是继承自实例并由属性访问调用的原型方法。有两种类型的访问器：获取器和设置器。

```ts
class MyClass4 {
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
const inst5 = new MyClass4();
assert.equal(inst5.name, 'Rumpelstiltskin'); // getter
inst5.name = 'Queen'; // setter
assert.equal(inst5.name, 'Queen'); // getter

```

#### 21.1.5 方法的修饰符：`*` (生成器)

```ts
class MyClass5 {
 * publicPrototypeGeneratorMethod() {
 yield 'hello';
 yield 'world';
 }
}

const inst6 = new MyClass5();
assert.deepEqual(
 Array.from(inst6.publicPrototypeGeneratorMethod()),
 ['hello', 'world']
);

```

#### 21.1.6 方法的修饰符：`async`

```ts
class MyClass6 {
 async publicPrototypeAsyncMethod() {
 const result = await Promise.resolve('abc');
 return result + result;
 }
}

const inst7 = new MyClass6();
assert.equal(
 await inst7.publicPrototypeAsyncMethod(),
 'abcabc'
);

```

#### 21.1.7 计算类成员名称

```ts
const publicInstanceFieldKey = Symbol('publicInstanceFieldKey');
const publicPrototypeMethodKey = Symbol('publicPrototypeMethodKey');

class MyClass7 {
 [publicInstanceFieldKey] = 1;
 [publicPrototypeMethodKey]() {
 return 2;
 }
}

const inst8 = new MyClass7();
assert.equal(inst8[publicInstanceFieldKey], 1);
assert.equal(inst8[publicPrototypeMethodKey](), 2);

```

注释：

+   此功能的用例主要是像 `Symbol.iterator` 这样的符号。但任何表达式都可以在方括号内使用。

+   我们可以计算字段、方法和访问器的名称。

+   我们无法计算私有成员（它们始终是固定的）的名称。

#### 21.1.8 修饰符的组合

字段：

| 级别 | 私有 | 代码 |
| --- | --- | --- |
| (实例) |  | `field` |
| (实例) | `#` | `#field` |
| `static` |  | `static field` |
| `static` | `#` | `static #field` |

方法（列：级别、访问器、异步、生成器、私有、代码 - 无主体）：

| 级别 | 访问器 | 异步 | 生成器 | 私有 | 代码 |
| --- | --- | --- | --- | --- | --- |
| (原型) |  |  |  |  | `m()` |
| (原型) | `get` |  |  |  | `get p()` |
| (原型) | `set` |  |  |  | `set p(x)` |
| (原型) |  | `async` |  |  | `async m()` |
| (原型) |  |  | `*` |  | `* m()` |
| (原型) |  | `async` | `*` |  | `async * m()` |
| (原型类似) |  |  |  | `#` | `#m()` |
| (原型类似) | `get` |  |  | `#` | `get #p()` |
| (原型类似) | `set` |  |  | `#` | `set #p(x)` |
| (原型类似) |  | `async` |  | `#` | `async #m()` |
| (原型类似) |  |  | `*` | `#` | `* #m()` |
| (原型类似) |  | `async` | `*` | `#` | `async * #m()` |
| `static` |  |  |  |  | `static m()` |
| `static` | `get` |  |  |  | `static get p()` |
| `static` | `set` |  |  |  | `static set p(x)` |
| `static` |  | `async` |  |  | `static async m()` |
| `static` |  |  | `*` |  | `static * m()` |
| `static` |  | `async` | `*` |  | `static async * m()` |
| `static` |  |  |  | `#` | `static #m()` |
| `static` | `get` |  |  | `#` | `static get #p()` |
| `static` | `set` |  |  | `#` | `static set #p(x)` |
| `static` |  | `async` |  | `#` | `static async #m()` |
| `static` |  |  | `*` | `#` | `static * #m()` |
| `static` |  | `async` | `*` | `#` | `static async * #m()` |

#### 21.1.9 内部机制

需要注意的是，在类中，有两个原型对象链：

+   实例链从实例开始。

+   静态链从该实例的类开始。

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

图 21.1 展示了由 `ClassA` 和 `ClassB` 创建的原型链的样子。

![](img/820493c8f187cebea74380b7ced1d6f2.png)

图 21.1：类 `ClassA` 和 `ClassB` 创建了两个原型链：一个用于类（左侧），另一个用于实例（右侧）。

#### 21.1.10 关于纯 JavaScript 中类定义的更多信息

+   “Exploring JavaScript”中的“类”章节 [Chapter “Classes”](https://exploringjs.com/js/book/ch_classes.html)

### 21.2 TypeScript 中的非公共数据槽位

默认情况下，TypeScript 中的所有数据槽位都是公共属性。有两种方法可以使数据保持私有：

+   私有属性

+   私有字段

我们将在下面查看这两个特性。

注意，TypeScript 目前不支持私有方法。

#### 21.2.1 私有属性

私有属性是 TypeScript 独有的（静态）特性。任何属性都可以通过在它前面加上关键字 `private` 来使其变为私有（行 A）：

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

如果我们在错误的范围内访问该属性，现在我们将得到编译时错误（行 A）：

```ts
const john = new PersonPrivateProperty('John');

assert.equal(
 john.sayHello(), 'Hello John!'
);

// @ts-expect-error: Property 'name' is private and only accessible
// within class 'PersonPrivateProperty'.
john.name; // (A)

```

然而，`private` 在运行时并不会改变任何事情。在那里，属性 `.name` 与公共属性没有区别：

```ts
assert.deepEqual(
 Object.keys(john),
 ['name']
);

```

通过查看类编译成的 JavaScript 代码，我们可以看到私有属性在运行时并没有受到保护：

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

#### 21.2.2 私有字段

私有字段是 JavaScript 中的一个新特性，TypeScript 从 3.8 版本开始支持：

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

这个版本的 `Person` 主要以与私有属性版本相同的方式使用：

```ts
const john = new PersonPrivateField('John');

assert.equal(
 john.sayHello(), 'Hello John!'
);

```

然而，这次，数据是完全封装的。在类外使用私有字段语法甚至会导致 JavaScript 语法错误。这就是为什么我们不得不在行 A 中使用 `eval()` 来执行此代码的原因：

```ts
assert.throws(
 () => eval('john.#name'), // (A)
 {
 name: 'SyntaxError',
 message: "Private field '#name' must be declared in "
 + "an enclosing class",
 }
);

assert.deepEqual(
 Object.keys(john),
 []
);

```

编译成 JavaScript 后，`PersonPrivateField` 看起来几乎相同：

```ts
class PersonPrivateField {
 #name;
 constructor(name) {
 this.#name = name;
 }
 sayHello() {
 return `Hello ${this.#name}!`;
 }
}

```

#### 21.2.3 私有属性与私有字段

+   私有属性的不利之处：

    +   我们不能在子类中重用私有属性的名字（因为属性在运行时不是私有的）。

    +   运行时没有封装。

+   私有属性的优势：

    +   客户端可以绕过封装并访问私有属性。如果有人需要绕过错误，这可能是有用的。换句话说：数据完全封装既有优点也有缺点。

    +   一些 JavaScript 辅助函数，例如用于克隆或序列化为 JSON 的函数，不与私有字段一起工作。

#### 21.2.4 受保护的属性

私有字段和私有属性在子类中无法访问（行 B）：

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
 override sayHello() { // (A)
 // @ts-expect-error: Property 'name' is private and only
 // accessible within class 'PrivatePerson'.
 return `Hello ${this.name} from ${this.company}!`; // (B)
 } 
}

```

关键字 `override` 将在 稍后 解释——它是用于覆盖超类方法的。

我们可以通过将行 A 中的 `private` 改为 `protected` 来修复前面的例子（为了保持一致性，我们也在行 B 中进行了更改）：

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
 override sayHello() {
 return `Hello ${this.name} from ${this.company}!`; // OK
 } 
}

```

### 21.3 私有构造函数

目前，JavaScript 不支持哈希私有构造函数。但是 TypeScript 支持 `private`。这在我们有静态工厂方法并且希望客户端始终使用这些方法而不是直接使用构造函数时很有用。静态方法可以访问私有类成员，这就是为什么工厂方法仍然可以使用构造函数。

在以下代码中，有一个静态工厂方法 `DataContainer.create()`。它通过异步加载数据设置实例。将异步代码保留在工厂方法中使得实际类可以完全同步：

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
const dataContainer = await DataContainer.create();
assert.equal(
 dataContainer.getData(),
 'DATA: downloaded'
);

```

在实际代码中，我们会在行 A 使用 `fetch()` 或类似的基于 Promise 的 API 来异步加载数据。

私有构造函数阻止 `DataContainer` 被继承。如果我们想允许子类，我们必须将其设置为 `protected`。

### 21.4 初始化实例属性

#### 21.4.1 严格的属性初始化

如果编译器设置 `--strictPropertyInitialization` 被打开（如果我们使用 `--strict`，则情况如此），那么 TypeScript 会检查所有声明的实例属性是否已正确初始化：

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

+   或者通过属性声明的初始化器：

    ```ts
    class Point {
     x = 0;
     y = 0;

     // No constructor needed
    }

    ```

然而，有时我们以 TypeScript 不认可的方式初始化属性。然后我们可以使用感叹号（*确定赋值断言*）来关闭 TypeScript 的警告（行 A 和行 B）：

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

##### 21.4.1.1 示例：通过对象设置实例属性

在以下示例中，我们还需要确定赋值断言。在这里，我们通过构造函数参数 `props` 设置实例属性：

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

注意事项：

+   在行 B 中，我们初始化所有属性：我们使用 `Object.assign()` 将参数 `props` 的属性复制到 `this`。

+   在行 A 中，`implements` 确保类声明了接口 `CompilerErrorProps` 的所有属性。

### 21.5 我们应该避免的便利功能

#### 21.5.1 推断成员类型

`tsc` 可以推断成员 `.str` 的类型，因为我们已经在行 A 中对其进行了赋值。然而，这与编译器选项 `isolatedDeclarations`（它启用外部工具生成声明而不进行推断）不兼容：

```ts
class C {
 str;
 constructor(str: string) {
 this.str = str; // (A)
 }
}

```

#### 21.5.2 使构造函数参数为 `public`、`private` 或 `protected`

JavaScript 目前没有与该子节中描述的 TypeScript 功能等效的功能——这就是为什么如果编译器选项 `erasableSyntaxOnly`（在 ch_tsconfig-json.html#erasableSyntaxOnly 中描述）处于活动状态，这是非法的。

如果我们为一个构造函数参数 `prop` 使用 `public` 修饰符，那么 TypeScript 会为我们做两件事：

+   它声明了一个公共实例属性 `.prop`。

+   它将参数 `prop` 赋值给该实例属性。

这是一个例子：

```ts
class Point {
 constructor(public x: number, public y: number) {
 }
}

```

如果我们使用 `private` 或 `protected` 而不是 `public`，则相应的实例属性将是私有的或受保护的。

TypeScript 类 `Point` 编译成以下 JavaScript 代码：

```ts
class Point {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
}

```

### 21.6 抽象类

TypeScript 中有两种结构可以是抽象的：

+   抽象类不能被实例化。只有它的子类可以 – 如果它们自身不是抽象的。

+   抽象方法没有实现，只有类型签名。每个具体的子类都必须有一个具有相同名称和兼容类型签名的具体方法。

    +   如果一个类有任何抽象方法，它也必须是抽象的。

以下代码演示了抽象类和方法。

一方面，有抽象超类 `Printable` 和其辅助类 `StringBuilder`：

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

另一方面，有具体的子类 `Entries` 和 `Entry`：

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

最后，这是我们使用 `Entries` 和 `Entry` 的例子：

```ts
const entries = new Entries([
 new Entry('accept-ranges', 'bytes'),
 new Entry('content-length', '6518'),
]);
assert.equal(
 entries.toString(),
 'accept-ranges: bytes\ncontent-length: 6518\n'
);

```

关于抽象类的注意事项：

+   抽象类可以被视为一个接口，其中一些成员已经有了实现。

+   虽然 class 可以实现多个接口，但它只能扩展最多一个抽象类。

+   “抽象性”仅在编译时存在。在运行时，抽象类是普通类，抽象方法不存在（因为它们只提供编译时信息）。

+   抽象类可以被视为模板，其中每个抽象方法都是一个需要由子类填充（实现）的空白。

### 21.7 方法关键字 `override`

关键字 `override` 用于覆盖超类中的方法 – 例如：

```ts
class A {
 m(): void {}
}
class B extends A {
 // `override` is required
 override m(): void {} // (A)
}

```

如果编译器选项 `noImplicitOverride`（在 ch_tsconfig-json.html#noImplicitOverride 中）处于活动状态，那么 TypeScript 如果在行 A 中没有 `override`，则会报错。

在实现抽象方法时，我们也可以使用 `override`。这不是必需的，但我发现这是一个有用的信息：

```ts
abstract class A {
 abstract m(): void;
}
class B extends A {
 // `override` is optional
 override m(): void {}
}

```

### 21.8 类与对象类型

在 JavaScript 中，我们不必使用类，我们也可以直接使用对象。TypeScript 支持这两种方法。

#### 21.8.1 类 `Counter`

这是一个实现计数器的类：

```ts
class Counter {
 count = 0;
 inc(): void {
 this.count++;
 }
}

// Trying out the functionality
const counter = new Counter();
counter.inc();
assert.equal(
 counter.count, 1
);

```

#### 21.8.2 对象类型 `Counter`

在 TypeScript 中，一个类定义了类型和实例工厂。在以下代码中，它们是分开的：我们有对象类型 `Counter` 和工厂 `createCounter()`。

```ts
type Counter = {
 count: number,
};
function createCounter(): Counter {
 return {
 count: 0,
 };
}
function inc(counter: Counter): void {
 counter.count++;
}

// Trying out the functionality
const counter = createCounter();
inc(counter);
assert.equal(
 counter.count, 1
);

```

#### 21.8.3 选择哪个：类或对象类型？

类的益处：

+   所有内容都在一个地方紧凑地指定：

    +   类型

    +   实例工厂

    +   `inc` 等操作

    +   属性默认值靠近其定义的地方指定是我想找到的有用之处 – 例如，`.count` 的默认值是 0。

+   我们可以通过 `instanceof` 检查一个值的类型 – 例如，用于缩小类型。

+   我们可以使用私有字段。

对象类型的益处：

+   如果对象被克隆，它们的工作效果会更好：用于克隆的库函数无法处理私有字段，而[`structuredClone()`](https://2ality.com/2022/01/structured-clone.html)不会保留实例的类。

+   如果对象在领域之间移动，它们的工作效果会更好：每个领域都有自己的给定类的版本，这使得移动类实例变得有问题。

序列化和反序列化（到/从 JSON 等）是一个有趣的用例：

+   使用对象类型，反序列化会更简单，因为我们可以直接使用`JSON.parse()`的结果（可能是在通过 Zod 验证类型之后）。

+   如果不是所有数据都可以轻松序列化和反序列化，事情会变得更加复杂——例如，如果属性包含一个`Map`。那么，类有一个好处：我们可以通过实现方法`.toJSON()`来自定义序列化。

除了这些标准之外，选择哪一个取决于你是否更喜欢更面向对象的代码或更函数式的代码。

我们还没有涵盖继承——在那里，你也有选择面向对象编码风格（类）和函数式编码风格（区分联合）之间的选择。更多信息，请参阅“类层次结构 vs. 区分联合”（§19.3）。
