# 15 对象类型

> 原文：[`exploringjs.com/tackling-ts/ch_typing-objects.html`](https://exploringjs.com/tackling-ts/ch_typing-objects.html)

* * *

+   15.1 对象扮演的角色

+   15.2 对象的类型

+   15.3 TypeScript 中的`Object` vs. `object`

    +   15.3.1 纯 JavaScript：对象 vs. `Object`的实例

    +   15.3.2 `Object`（大写“O”）在 TypeScript 中：类`Object`的实例

    +   15.3.3 TypeScript 中的`object`（小写“o”）：非原始值

    +   15.3.4 `Object` vs. `object`：原始值

    +   15.3.5 `Object` vs. `object`：不兼容的属性类型

+   15.4 对象类型文字和接口

    +   15.4.1 对象类型文字和接口之间的区别

    +   15.4.2 接口在 TypeScript 中的结构工作

    +   15.4.3 接口和对象类型文字的成员

    +   15.4.4 方法签名

    +   15.4.5 索引签名：对象作为字典

    +   15.4.6 接口描述`Object`的实例

    +   15.4.7 多余属性检查：何时允许额外属性？

+   15.5 类型推断

+   15.6 接口的其他特性

    +   15.6.1 可选属性

    +   15.6.2 只读属性

+   15.7 JavaScript 的原型链和 TypeScript 的类型

+   15.8 本章的来源

* * *

在本章中，我们将探讨在 TypeScript 中如何静态地为对象和属性进行类型化。

### 15.1 对象扮演的角色

在 JavaScript 中，对象可以扮演两种角色（总是至少其中一种，有时混合）：

+   *记录*具有在开发时已知的固定数量的属性。每个属性可以具有不同的类型。

+   *字典*具有任意数量的属性，其名称在开发时未知。所有属性键（字符串和/或符号）具有相同的类型，属性值也是如此。

首先，我们将探索对象作为记录。我们将在本章的后面简要地遇到对象作为字典。

### 15.2 对象类型

对象有两种不同的一般类型：

+   `Object`大写“O”是类`Object`的所有实例的类型：

    ```ts
    let obj1: Object;
    ```

+   小写“o”的`object`是所有非原始值的类型：

    ```ts
    let obj2: object;
    ```

对象也可以通过它们的属性进行类型化：

```ts
// Object type literal
let obj3: {prop: boolean};

// Interface
interface ObjectType {
 prop: boolean;
}
let obj4: ObjectType;
```

在接下来的章节中，我们将更详细地研究所有这些对象类型的方式。

### 15.3 TypeScript 中的`Object` vs. `object`

#### 15.3.1 纯 JavaScript：对象 vs. `Object`的实例

在纯 JavaScript 中，有一个重要的区别。

一方面，大多数对象都是`Object`的实例。

```ts
> const obj1 = {};
> obj1 instanceof Object
true
```

这意味着：

+   `Object.prototype`在它们的原型链中：

    ```ts
    > Object.prototype.isPrototypeOf(obj1)
    true
    ```

+   它们继承了它的属性。

    ```ts
    > obj1.toString === Object.prototype.toString
    true
    ```

另一方面，我们也可以创建没有`Object.prototype`的原型链的对象。例如，以下对象根本没有原型：

```ts
> const obj2 = Object.create(null);
> Object.getPrototypeOf(obj2)
null
```

`obj2`是一个不是类`Object`实例的对象：

```ts
> typeof obj2
'object'
> obj2 instanceof Object
false
```

#### 15.3.2 TypeScript 中的`Object`（大写“O”）：类`Object`的实例

请记住，每个类`C`都创建两个实体：

+   一个构造函数`C`。

+   描述构造函数实例的接口`C`。

同样，TypeScript 有两个内置接口：

+   接口`Object`指定了`Object`实例的属性，包括从`Object.prototype`继承的属性。

+   接口`ObjectConstructor`指定了类`Object`的属性。

这些是接口：

```ts
interface Object { // (A)
 constructor: Function;
 toString(): string;
 toLocaleString(): string;
 valueOf(): Object;
 hasOwnProperty(v: PropertyKey): boolean;
 isPrototypeOf(v: Object): boolean;
 propertyIsEnumerable(v: PropertyKey): boolean;
}

interface ObjectConstructor {
 /** Invocation via `new` */
 new(value?: any): Object;
 /** Invocation via function calls */
 (value?: any): any;

 readonly prototype: Object; // (B)

 getPrototypeOf(o: any): any;

 // ···
}
declare var Object: ObjectConstructor; // (C)
```

观察：

+   我们既有一个名为`Object`的变量（行 C），又有一个名为`Object`的类型（行 A）。

+   `Object`的直接实例没有自己的属性，因此`Object.prototype`也匹配`Object`（行 B）。

#### 15.3.3 TypeScript 中的`object`（小写“o”）：非原始值

在 TypeScript 中，`object`是所有非原始值的类型（原始值是`undefined`，`null`，布尔值，数字，大整数，字符串）。使用此类型，我们无法访问值的任何属性。

#### 15.3.4 TypeScript 中的`Object` vs. `object`：原始值

有趣的是，类型`Object`也匹配原始值：

```ts
function func1(x: Object) { }
func1('abc'); // OK
```

为什么？原始值具有`Object`所需的所有属性，因为它们继承了`Object.prototype`：

```ts
> 'abc'.hasOwnProperty === Object.prototype.hasOwnProperty
true
```

相反，`object`不匹配原始值：

```ts
function func2(x: object) { }
// @ts-expect-error: Argument of type '"abc"' is not assignable to
// parameter of type 'object'. (2345)
func2('abc');
```

#### 15.3.5 TypeScript 中的`Object` vs. `object`：不兼容的属性类型

使用类型`Object`时，如果对象具有与接口`Object`中相应属性冲突的类型，则 TypeScript 会发出警告：

```ts
// @ts-expect-error: Type '() => number' is not assignable to
// type '() => string'.
//   Type 'number' is not assignable to type 'string'. (2322)
const obj1: Object = { toString() { return 123 } };
```

使用类型`object`时，TypeScript 不会发出警告（因为`object`不指定任何属性，也不会有任何冲突）：

```ts
const obj2: object = { toString() { return 123 } };
```

### 15.4 对象类型文字和接口

TypeScript 有两种定义非常相似的对象类型的方式：

```ts
// Object type literal
type ObjType1 = {
 a: boolean,
 b: number;
 c: string,
};

// Interface
interface ObjType2 {
 a: boolean,
 b: number;
 c: string,
}
```

我们可以使用分号或逗号作为分隔符。允许并且是可选的尾随分隔符。

#### 15.4.1 对象类型文字和接口之间的区别

在本节中，我们将重点介绍对象类型文字和接口之间最重要的区别。

##### 15.4.1.1 内联

对象类型文字可以内联，而接口不能：

```ts
// Inlined object type literal:
function f1(x: {prop: number}) {}

// Referenced interface:
function f2(x: ObjectInterface) {} 
interface ObjectInterface {
 prop: number;
}
```

##### 15.4.1.2 重复名称

具有重复名称的类型别名是非法的：

```ts
// @ts-expect-error: Duplicate identifier 'PersonAlias'. (2300)
type PersonAlias = {first: string};
// @ts-expect-error: Duplicate identifier 'PersonAlias'. (2300)
type PersonAlias = {last: string};
```

相反，具有重复名称的接口会合并：

```ts
interface PersonInterface {
 first: string;
}
interface PersonInterface {
 last: string;
}
const jane: PersonInterface = {
 first: 'Jane',
 last: 'Doe',
};
```

##### 15.4.1.3 映射类型

对于映射类型（行 A），我们需要使用对象类型文字：

```ts
interface Point {
 x: number;
 y: number;
}

type PointCopy1 = {
 [Key in keyof Point]: Point[Key]; // (A)
};

// Syntax error:
// interface PointCopy2 {
//   [Key in keyof Point]: Point[Key];
// };
```

![](img/8c55f45a6e023f74c4403b0374043880.png) **有关映射类型的更多信息**

映射类型超出了本书的范围。有关更多信息，请参见[TypeScript 手册](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)。

##### 15.4.1.4 多态`this`类型

多态`this`类型只能在接口中使用：

```ts
interface AddsStrings {
 add(str: string): this;
};

class StringBuilder implements AddsStrings {
 result = '';
 add(str: string) {
 this.result += str;
 return this;
 }
}
```

![](img/8c55f45a6e023f74c4403b0374043880.png) **本节的来源**

+   [GitHub 问题“TypeScript：类型 vs. 接口”](https://github.com/peerigon/eslint-config-peerigon/issues/64) 由[Johannes Ewald](https://github.com/jhnns)

![](img/65d35c0a2478236e12cc4321e1b02db6.png) **从现在开始，“接口”意味着“接口或对象类型文字”（除非另有说明）。

#### 15.4.2 TypeScript 中的接口是结构化的

接口是结构化的 - 它们不必被实现才能匹配：

```ts
interface Point {
 x: number;
 y: number;
}
const point: Point = {x: 1, y: 2}; // OK
```

有关此主题的更多信息，请参见[[content not included]](ch_missing-chapters-online.html)。

#### 15.4.3 接口和对象类型文字的成员

接口和对象类型文字的主体内的构造被称为它们的*成员*。这些是最常见的成员：

```ts
interface ExampleInterface {
 // Property signature
 myProperty: boolean;

 // Method signature
 myMethod(str: string): number;

 // Index signature
 [key: string]: any;

 // Call signature
 (num: number): string;

 // Construct signature
 new(str: string): ExampleInstance; 
}
interface ExampleInstance {}
```

让我们更详细地看看这些成员：

+   属性签名定义属性：

    ```ts
    myProperty: boolean;
    ```

+   方法签名定义方法：

    ```ts
    myMethod(str: string): number;
    ```

    注意：参数的名称（在本例中为：`str`）有助于记录事物的工作原理，但没有其他目的。

+   需要索引签名来描述用作字典的数组或对象。

    ```ts
    [key: string]: any;
    ```

    注意：名称 `key` 仅用于文档目的。

+   调用签名使接口能够描述函数：

    ```ts
    (num: number): string;
    ```

+   构造签名使接口能够描述类和构造函数：

    ```ts
    new(str: string): ExampleInstance; 
    ```

属性签名应该是不言自明的。调用签名 和 构造签名 将在本书的后面进行描述。接下来我们将更仔细地看一下方法签名和索引签名。

#### 15.4.4 方法签名

就 TypeScript 的类型系统而言，方法定义和属性的值为函数的属性是等效的：

```ts
interface HasMethodDef {
 simpleMethod(flag: boolean): void;
}
interface HasFuncProp {
 simpleMethod: (flag: boolean) => void;
}

const objWithMethod: HasMethodDef = {
 simpleMethod(flag: boolean): void {},
};
const objWithMethod2: HasFuncProp = objWithMethod;

const objWithOrdinaryFunction: HasMethodDef = {
 simpleMethod: function (flag: boolean): void {},
};
const objWithOrdinaryFunction2: HasFuncProp = objWithOrdinaryFunction;

const objWithArrowFunction: HasMethodDef = {
 simpleMethod: (flag: boolean): void => {},
};
const objWithArrowFunction2: HasFuncProp = objWithArrowFunction;
```

我的建议是使用最能表达属性应如何设置的语法。

#### 15.4.5 索引签名：对象作为字典

到目前为止，我们只使用接口来表示具有固定键的对象记录。我们如何表达对象将用作字典的事实？例如：在以下代码片段中，`TranslationDict` 应该是什么？

```ts
function translate(dict: TranslationDict, english: string): string {
 return dict[english];
}
```

我们使用索引签名（行 A）来表示 `TranslationDict` 适用于将字符串键映射到字符串值的对象：

```ts
interface TranslationDict {
 [key:string]: string; // (A)
}
const dict = {
 'yes': 'sí',
 'no': 'no',
 'maybe': 'tal vez',
};
assert.equal(
 translate(dict, 'maybe'),
 'tal vez');
```

##### 15.4.5.1 为索引签名键添加类型

索引签名键必须是 `string` 或 `number`：

+   不允许使用符号。

+   `any` 是不允许的。

+   联合类型（例如 `string|number`）是不允许的。但是，每个接口可以使用多个索引签名。

##### 15.4.5.2 字符串键 vs. 数字键

与纯 JavaScript 一样，TypeScript 的数字属性键是字符串属性键的子集（[参见“JavaScript for impatient programmers”](https://exploringjs.com/impatient-js/ch_arrays.html#array-indices)）。因此，如果我们既有字符串索引签名又有数字索引签名，则前者的属性类型必须是后者的超类型。以下示例有效，因为 `Object` 是 `RegExp` 的超类型：

```ts
interface StringAndNumberKeys {
 [key: string]: Object;
 [key: number]: RegExp;
}

// %inferred-type: (x: StringAndNumberKeys) =>
// { str: Object; num: RegExp; }
function f(x: StringAndNumberKeys) {
 return { str: x['abc'], num: x[123] };
}
```

##### 15.4.5.3 索引签名 vs. 属性签名和方法签名

如果接口中既有索引签名又有属性和/或方法签名，那么索引属性值的类型也必须是属性值和/或方法的超类型。

```ts
interface I1 {
 [key: string]: boolean;

 // @ts-expect-error: Property 'myProp' of type 'number' is not assignable
 // to string index type 'boolean'. (2411)
 myProp: number;

 // @ts-expect-error: Property 'myMethod' of type '() => string' is not
 // assignable to string index type 'boolean'. (2411)
 myMethod(): string;
}
```

相比之下，以下两个接口不会产生错误：

```ts
interface I2 {
 [key: string]: number;
 myProp: number;
}

interface I3 {
 [key: string]: () => string;
 myMethod(): string;
}
```

#### 15.4.6 接口描述 `Object` 的实例

所有接口描述的对象都是 `Object` 的实例，并继承 `Object.prototype` 的属性。

在以下示例中，类型为 `{}` 的参数 `x` 与返回类型 `Object` 兼容：

```ts
function f1(x: {}): Object {
 return x;
}
```

同样，`{}` 有一个 `.toString()` 方法：

```ts
function f2(x: {}): { toString(): string } {
 return x;
}
```

#### 15.4.7 多余属性检查：何时允许额外属性？

例如，考虑以下接口：

```ts
interface Point {
 x: number;
 y: number;
}
```

有两种（等等）方式可以解释此接口：

+   封闭解释：它可以描述所有具有指定类型的属性 `.x` 和 `.y` 的对象。换句话说：这些对象不能有*多余的属性*（超出所需的属性）。

+   开放解释：它可以描述所有具有*至少*属性 `.x` 和 `.y` 的对象。换句话说：允许多余的属性。

TypeScript 使用两种解释。为了探索它是如何工作的，我们将使用以下函数：

```ts
function computeDistance(point: Point) { /*...*/ }
```

默认情况下，允许多余的属性 `.z`：

```ts
const obj = { x: 1, y: 2, z: 3 };
computeDistance(obj); // OK
```

但是，如果我们直接使用对象文字，则不允许多余的属性：

```ts
// @ts-expect-error: Argument of type '{ x: number; y: number; z: number; }'
// is not assignable to parameter of type 'Point'.
//   Object literal may only specify known properties, and 'z' does not
//   exist in type 'Point'. (2345)
computeDistance({ x: 1, y: 2, z: 3 }); // error

computeDistance({x: 1, y: 2}); // OK
```

##### 15.4.7.1 为什么对象文字中禁止多余的属性？

为什么对象文字有更严格的规则？它们可以防止属性键中的拼写错误。我们将使用以下接口来演示这意味着什么。

```ts
interface Person {
 first: string;
 middle?: string;
 last: string;
}
function computeFullName(person: Person) { /*...*/ }
```

属性 `.middle` 是可选的，可以省略（可选属性在本章后面有介绍）。对于 TypeScript 来说，错误地拼写它的名称看起来像是省略了它并提供了多余的属性。但是，它仍然捕获了拼写错误，因为在这种情况下不允许多余的属性：

```ts
// @ts-expect-error: Argument of type '{ first: string; mdidle: string;
// last: string; }' is not assignable to parameter of type 'Person'.
//   Object literal may only specify known properties, but 'mdidle'
//   does not exist in type 'Person'. Did you mean to write 'middle'?
computeFullName({first: 'Jane', mdidle: 'Cecily', last: 'Doe'});
```

##### 15.4.7.2 如果对象来自其他地方，为什么允许多余的属性？

这个想法是，如果一个对象来自其他地方，我们可以假设它已经经过审查，不会有任何拼写错误。然后我们可以不那么小心。

如果拼写错误不是问题，我们的目标应该是最大限度地提高灵活性。考虑以下函数：

```ts
interface HasYear {
 year: number;
}

function getAge(obj: HasYear) {
 const yearNow = new Date().getFullYear();
 return yearNow - obj.year;
}
```

如果不允许大多数传递给`getAge()`的值具有多余的属性，那么这个函数的用处将会非常有限。

##### 15.4.7.3 空接口允许多余的属性

如果接口为空（或者使用对象类型文字`{}`），则始终允许多余的属性：

```ts
interface Empty { }
interface OneProp {
 myProp: number;
}

// @ts-expect-error: Type '{ myProp: number; anotherProp: number; }' is not
// assignable to type 'OneProp'.
//   Object literal may only specify known properties, and
//   'anotherProp' does not exist in type 'OneProp'. (2322)
const a: OneProp = { myProp: 1, anotherProp: 2 };
const b: Empty = {myProp: 1, anotherProp: 2}; // OK
```

##### 15.4.7.4 仅匹配没有属性的对象

如果我们想要强制对象没有属性，我们可以使用以下技巧（来源：[Geoff Goodman](https://twitter.com/filearts/status/1222502898552180737)）：

```ts
interface WithoutProperties {
 [key: string]: never;
}

// @ts-expect-error: Type 'number' is not assignable to type 'never'. (2322)
const a: WithoutProperties = { prop: 1 };
const b: WithoutProperties = {}; // OK
```

##### 15.4.7.5 允许对象文字中的多余属性

如果我们想要允许对象文字中的多余属性怎么办？例如，考虑接口`Point`和函数`computeDistance1()`：

```ts
interface Point {
 x: number;
 y: number;
}

function computeDistance1(point: Point) { /*...*/ }

// @ts-expect-error: Argument of type '{ x: number; y: number; z: number; }'
// is not assignable to parameter of type 'Point'.
//   Object literal may only specify known properties, and 'z' does not
//   exist in type 'Point'. (2345)
computeDistance1({ x: 1, y: 2, z: 3 });
```

一种选择是将对象文字分配给一个中间变量：

```ts
const obj = { x: 1, y: 2, z: 3 };
computeDistance1(obj);
```

第二种选择是使用类型断言：

```ts
computeDistance1({ x: 1, y: 2, z: 3 } as Point); // OK
```

第三种选择是重写`computeDistance1()`，使其使用类型参数：

```ts
function computeDistance2<P extends Point>(point: P) { /*...*/ }
computeDistance2({ x: 1, y: 2, z: 3 }); // OK
```

第四种选择是扩展接口`Point`，以便允许多余的属性：

```ts
interface PointEtc extends Point {
 [key: string]: any;
}
function computeDistance3(point: PointEtc) { /*...*/ }

computeDistance3({ x: 1, y: 2, z: 3 }); // OK
```

我们将继续讨论两个示例，其中 TypeScript 不允许多余的属性是一个问题。

###### 15.4.7.5.1 允许多余的属性：示例`Incrementor`

在这个例子中，我们想要实现一个`Incrementor`，但是 TypeScript 不允许额外的属性`.counter`：

```ts
interface Incrementor {
 inc(): void
}
function createIncrementor(start = 0): Incrementor {
 return {
 // @ts-expect-error: Type '{ counter: number; inc(): void; }' is not
 // assignable to type 'Incrementor'.
 //   Object literal may only specify known properties, and
 //   'counter' does not exist in type 'Incrementor'. (2322)
 counter: start,
 inc() {
 // @ts-expect-error: Property 'counter' does not exist on type
 // 'Incrementor'. (2339)
 this.counter++;
 },
 };
}
```

然而，即使使用类型断言，仍然存在一个类型错误：

```ts
function createIncrementor2(start = 0): Incrementor {
 return {
 counter: start,
 inc() {
 // @ts-expect-error: Property 'counter' does not exist on type
 // 'Incrementor'. (2339)
 this.counter++;
 },
 } as Incrementor;
}
```

我们可以在接口`Incrementor`中添加索引签名。或者 - 尤其是如果不可能的话 - 我们可以引入一个中间变量：

```ts
function createIncrementor3(start = 0): Incrementor {
 const incrementor = {
 counter: start,
 inc() {
 this.counter++;
 },
 };
 return incrementor;
}
```

###### 15.4.7.5.2 允许多余的属性：示例`.dateStr`

以下比较函数可用于对具有属性`.dateStr`的对象进行排序：

```ts
function compareDateStrings(
 a: {dateStr: string}, b: {dateStr: string}) {
 if (a.dateStr < b.dateStr) {
 return +1;
 } else if (a.dateStr > b.dateStr) {
 return -1;
 } else {
 return 0;
 }
 }
```

例如，在单元测试中，我们可能希望直接使用对象文字调用此函数。TypeScript 不允许我们这样做，我们需要使用其中一种解决方法。

### 15.5 类型推断

这些是 TypeScript 通过各种方式创建的对象推断的类型：

```ts
// %inferred-type: Object
const obj1 = new Object();

// %inferred-type: any
const obj2 = Object.create(null);

// %inferred-type: {}
const obj3 = {};

// %inferred-type: { prop: number; }
const obj4 = {prop: 123};

// %inferred-type: object
const obj5 = Reflect.getPrototypeOf({});
```

原则上，`Object.create()`的返回类型可以是`object`。但是，`any`允许我们添加和更改结果的属性。

### 15.6 接口的其他特性

#### 15.6.1 可选属性

如果我们在属性名称后面加上问号（`?`），那么该属性就是可选的。相同的语法用于将函数、方法和构造函数的参数标记为可选。在下面的例子中，属性`.middle`是可选的：

```ts
interface Name {
 first: string;
 middle?: string;
 last: string;
}
```

因此，省略该属性是可以的（A 行）：

```ts
const john: Name = {first: 'Doe', last: 'Doe'}; // (A)
const jane: Name = {first: 'Jane', middle: 'Cecily', last: 'Doe'};
```

##### 15.6.1.1 可选 vs. `undefined|string`

`.prop1`和`.prop2`之间有什么区别？

```ts
interface Interf {
 prop1?: string;
 prop2: undefined | string; 
}
```

可选属性可以做`undefined|string`可以做的一切。我们甚至可以使用前者的值`undefined`：

```ts
const obj1: Interf = { prop1: undefined, prop2: undefined };
```

然而，只有`.prop1`可以省略：

```ts
const obj2: Interf = { prop2: undefined };

// @ts-expect-error: Property 'prop2' is missing in type '{}' but required
// in type 'Interf'. (2741)
const obj3: Interf = { };
```

诸如`undefined|string`和`null|string`之类的类型在我们想要明确省略时非常有用。当人们看到这样一个明确省略的属性时，他们知道它存在，但已被关闭。

#### 15.6.2 只读属性

在下面的例子中，属性`.prop`是只读的：

```ts
interface MyInterface {
 readonly prop: number;
}
```

因此，我们可以读取它，但我们不能更改它：

```ts
const obj: MyInterface = {
 prop: 1,
};

console.log(obj.prop); // OK

// @ts-expect-error: Cannot assign to 'prop' because it is a read-only
// property. (2540)
obj.prop = 2;
```

### 15.7 JavaScript 的原型链和 TypeScript 的类型

TypeScript 不区分自有属性和继承属性。它们都被简单地视为属性。

```ts
interface MyInterface {
 toString(): string; // inherited property
 prop: number; // own property
}
const obj: MyInterface = { // OK
 prop: 123,
};
```

`obj`从`Object.prototype`继承`.toString()`。

这种方法的缺点是，JavaScript 中的一些现象无法通过 TypeScript 的类型系统来描述。好处是类型系统更简单。

### 15.8 本章的来源

+   [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/basic-types.html)

+   [TypeScript 语言规范](https://github.com/microsoft/TypeScript/blob/master/doc/spec-ARCHIVED.md)

[评论](https://github.com/rauschma/tackling-ts/issues/15)
