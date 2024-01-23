# 七、TypeScript 的基本要点

> 原文：[`exploringjs.com/tackling-ts/ch_typescript-essentials.html`](https://exploringjs.com/tackling-ts/ch_typescript-essentials.html)

* * *

+   7.1 你将学到什么

+   7.2 指定类型检查的全面性

+   7.3 TypeScript 中的类型

+   7.4 类型注解

+   7.5 类型推断

+   7.6 通过类型表达式指定类型

+   7.7 两种语言级别：动态 vs. 静态

+   7.8 类型别名

+   7.9 数组类型标注

    +   7.9.1 数组作为列表

    +   7.9.2 数组作为元组

+   7.10 函数类型

    +   7.10.1 更复杂的示例

    +   7.10.2 函数声明的返回类型

    +   7.10.3 可选参数

    +   7.10.4 剩余参数

+   7.11 联合类型

    +   7.11.1 默认情况下，`undefined`和`null`不包括在类型中

    +   7.11.2 明确省略

+   7.12 可选 vs. 默认值 vs. `undefined|T`

+   7.13 对象类型标注

    +   7.13.1 通过接口将对象作为记录进行类型标注

    +   7.13.2 TypeScript 的结构类型 vs. 名义类型

    +   7.13.3 对象字面量类型

    +   7.13.4 可选属性

    +   7.13.5 方法

+   7.14 类型变量和泛型类型

    +   7.14.1 示例：值的容器

+   7.15 示例：泛型类

    +   7.15.1 示例：映射

    +   7.15.2 函数和方法的类型变量

    +   7.15.3 更复杂的函数示例

+   7.16 结论：理解初始示例

* * *

本章介绍了 TypeScript 的基本要点。

### 7.1 你将学到什么

阅读完本章后，你应该能够理解以下 TypeScript 代码：

```ts
interface Array<T> {
 concat(...items: Array<T[] | T>): T[];
 reduce<U>(
 callback: (state: U, element: T, index: number, array: T[]) => U,
 firstState?: U
 ): U;
 // ···
}
```

你可能会认为这很神秘。我同意你的看法！但是（正如我希望证明的那样），这种语法相对容易学习。一旦你理解了它，它可以立即、准确和全面地总结代码的行为方式，而无需阅读冗长的英文描述。

### 7.2 指定类型检查的全面性

TypeScript 编译器可以配置的方式有很多。一个重要的选项组控制编译器对 TypeScript 代码的检查程度。通过`--strict`激活最大设置，我建议始终使用它。这使得程序稍微难以编写，但我们也获得了静态类型检查的全部好处。

![](img/65d35c0a2478236e12cc4321e1b02db6.png) **这就是你现在需要了解的关于`--strict`的一切**

如果想了解更多细节，请继续阅读。

将`--strict`设置为`true`，会将以下所有选项设置为`true`：

+   `--noImplicitAny`：如果 TypeScript 无法推断类型，我们必须指定类型。这主要适用于函数和方法的参数：使用这个设置，我们必须注释它们。

+   `--noImplicitThis`：如果`this`的类型不清晰，则会报错。

+   `--alwaysStrict`：尽可能使用 JavaScript 的严格模式。

+   `--strictNullChecks`：`null`不是任何类型的一部分（除了它自己的类型`null`），如果它是可接受的值，必须明确提及。

+   `--strictFunctionTypes`：启用对函数类型的更严格检查。

+   `--strictPropertyInitialization`：类定义中的属性必须初始化，除非它们可以有值`undefined`。

我们将在本书的后面看到更多的编译器选项，当我们开始使用 TypeScript 创建 npm 包和 web 应用时。TypeScript 手册中有关于它们的[全面文档](https://www.typescriptlang.org/docs/handbook/compiler-options.html)。

### 7.3 TypeScript 中的类型

在本章中，类型只是一组值。JavaScript 语言（不是 TypeScript！）只有八种类型：

1.  未定义：只有一个元素`undefined`的集合

1.  Null：只有一个元素`null`的集合

1.  布尔：只有两个元素`false`和`true`的集合

1.  数字：所有数字的集合

1.  BigInt：所有任意精度整数的集合

1.  字符串：所有字符串的集合

1.  符号：所有符号的集合

1.  对象：所有对象的集合（包括函数和数组）

所有这些类型都是*动态*的：我们可以在运行时使用它们。

TypeScript 为 JavaScript 带来了一个额外的层次：*静态类型*。这些只在编译或类型检查源代码时存在。每个存储位置（变量、属性等）都有一个静态类型，用于预测其动态值。类型检查确保这些预测成真。

还有很多可以*静态*检查的内容（不运行代码）。例如，如果函数`toString(num)`的参数`num`的静态类型是`number`，那么函数调用`toString('abc')`是非法的，因为参数`'abc'`的静态类型错误。

### 7.4 类型注释

```ts
function toString(num: number): string {
 return String(num);
}
```

在上一个函数声明中有两种类型注释：

+   参数`num`：冒号后跟`number`

+   `toString()`的结果：冒号后跟`string`

`number`和`string`都是*类型表达式*，用于指定存储位置的类型。

### 7.5 类型推断

通常，如果没有类型注释，TypeScript 可以*推断*出静态类型。例如，如果我们省略`toString()`的返回类型，TypeScript 会推断它是`string`：

```ts
// %inferred-type: (num: number) => string
function toString(num: number) {
 return String(num);
}
```

类型推断不是猜测：它遵循清晰的规则（类似于算术）来推导未明确指定类型的地方的类型。在这种情况下，返回语句应用了一个将任意值映射到字符串的函数`String()`，将类型为`number`的值`num`映射到字符串并返回结果。这就是为什么推断的返回类型是`string`。

如果位置的类型既没有明确指定也无法推断，TypeScript 会使用类型`any`。这是所有值的类型和通配符，如果一个值具有该类型，我们可以做任何事情。

在`--strict`中，只有在显式使用`any`时才允许使用它。换句话说：每个位置必须有一个显式或推断的静态类型。在下面的例子中，参数`num`都没有，我们会得到一个编译时错误：

```ts
// @ts-expect-error: Parameter 'num' implicitly has an 'any' type. (7006)
function toString(num) {
 return String(num);
}
```

### 7.6 通过类型表达式指定类型

类型注解的冒号后面的类型表达式从简单到复杂不等，创建方式如下。

基本类型是有效的类型表达式：

+   JavaScript 的动态类型的静态类型：

    +   `undefined`，`null`

    +   `boolean`，`number`，`bigint`，`string`

    +   `symbol`

    +   `object`。

+   特定于 TypeScript 的类型：

    +   `Array`（在 JavaScript 中不是严格的类型）

    +   `any`（所有值的类型）

    +   等等。

有许多种方式将基本类型组合成新的*复合类型*。例如，通过*类型运算符*，它们类似于集合运算符*并集*（`∪`）和*交集*（`∩`）组合集合的方式。我们很快会看到如何做到这一点。

### 7.7 两种语言级别：动态 vs. 静态

TypeScript 有两种语言级别：

+   *动态级别*由 JavaScript 管理，并且包括运行时的代码和值。

+   *静态级别*由 TypeScript（不包括 JavaScript）管理，并且在编译时包括静态类型。

我们可以在语法中看到这两个级别：

```ts
const undef: undefined = undefined;
```

+   在动态级别上，我们使用 JavaScript 声明一个变量`undef`并用值`undefined`初始化它。

+   在静态级别上，我们使用 TypeScript 指定变量`undef`的静态类型为`undefined`。

请注意，相同的语法`undefined`，根据它是在动态级别还是静态级别使用，意思不同。

![](img/6a9318e9b4540abb73475976e01d55f9.png) 尝试培养对两种语言级别的认识

这在很大程度上有助于理解 TypeScript。

### 7.8 类型别名

使用`type`我们可以为现有类型创建一个新名称（别名）：

```ts
type Age = number;
const age: Age = 82;
```

### 7.9 数组的类型

数组在 JavaScript 中扮演两种角色（可能是一种或两种）：

+   列表：所有元素具有相同的类型。数组的长度不同。

+   元组：数组的长度是固定的。元素通常不具有相同的类型。

#### 7.9.1 数组作为列表

有两种方式来表达数组`arr`被用作一个所有元素都是数字的列表的事实：

```ts
let arr1: number[] = [];
let arr2: Array<number> = [];
```

通常情况下，如果有赋值，TypeScript 可以推断变量的类型。在这种情况下，我们实际上必须帮助它，因为对于空数组，它无法确定元素的类型。

稍后我们会回到尖括号表示法（`Array<number>`）。

#### 7.9.2 数组作为元组

如果我们将二维点存储在一个数组中，那么我们使用该数组作为元组。看起来是这样的：

```ts
let point: [number, number] = [7, 5];
```

对于数组字面量，类型注解是必需的，因为 TypeScript 会推断列表类型，而不是元组类型：

```ts
// %inferred-type: number[]
let point = [7, 5];
```

元组的另一个例子是`Object.entries(obj)`的结果：一个数组，每个`obj`的属性都有一个[key, value]对。

```ts
// %inferred-type: [string, number][]
const entries = Object.entries({ a: 1, b: 2 });

assert.deepEqual(
 entries,
 [[ 'a', 1 ], [ 'b', 2 ]]);
```

推断的类型是元组的数组。

### 7.10 函数类型

这是一个函数类型的例子：

```ts
(num: number) => string
```

这种类型包括每个接受一个`number`类型的参数并返回一个`string`的函数。让我们在类型注解中使用这种类型：

```ts
const toString: (num: number) => string = // (A)
 (num: number) => String(num); // (B)
```

通常，我们必须为函数指定参数类型。但在这种情况下，`num`在 B 行的类型可以从 A 行的函数类型中推断出来，我们可以省略它：

```ts
const toString: (num: number) => string =
 (num) => String(num);
```

如果我们省略`toString`的类型注解，TypeScript 会从箭头函数中推断出类型：

```ts
// %inferred-type: (num: number) => string
const toString = (num: number) => String(num);
```

这次，`num`必须有一个类型注解。

#### 7.10.1 更复杂的例子

下面的例子更加复杂：

```ts
function stringify123(callback: (num: number) => string) {
 return callback(123);
}
```

我们使用函数类型来描述`stringify123()`的参数`callback`。由于这个类型注解，TypeScript 拒绝了以下函数调用。

```ts
// @ts-expect-error: Argument of type 'NumberConstructor' is not
// assignable to parameter of type '(num: number) => string'.
//   Type 'number' is not assignable to type 'string'.(2345)
stringify123(Number);
```

但它接受这个函数调用：

```ts
assert.equal(
 stringify123(String), '123');
```

#### 7.10.2 函数声明的返回类型

TypeScript 通常可以推断函数的返回类型，但允许显式指定它们并且偶尔是有用的（至少，它不会有害）。

对于`stringify123()`，指定返回类型是可选的，看起来像这样：

```ts
function stringify123(callback: (num: number) => string): string {
 return callback(123);
}
```

##### 7.10.2.1 特殊的返回类型`void`

`void`是函数的一个特殊返回类型：它告诉 TypeScript 函数总是返回`undefined`。

它可能会显式地这样做：

```ts
function f1(): void {
 return undefined;
}
```

或者它可能会隐式地这样做：

```ts
function f2(): void {}
```

然而，这样的函数不能明确返回除`undefined`之外的值：

```ts
function f3(): void {
 // @ts-expect-error: Type '"abc"' is not assignable to type 'void'. (2322)
 return 'abc';
}
```

#### 7.10.3 可选参数

标识符后面的问号表示参数是可选的。例如：

```ts
function stringify123(callback?: (num: number) => string) {
 if (callback === undefined) {
 callback = String;
 }
 return callback(123); // (A)
}
```

TypeScript 只有在确保`callback`不是`undefined`时才允许我们在 A 行进行函数调用（如果参数被省略，则它是`undefined`）。

##### 7.10.3.1 参数默认值

TypeScript 支持[参数默认值](https://exploringjs.com/impatient-js/ch_callables.html#parameter-default-values)：

```ts
function createPoint(x=0, y=0): [number, number] {
 return [x, y];
}

assert.deepEqual(
 createPoint(),
 [0, 0]);
assert.deepEqual(
 createPoint(1, 2),
 [1, 2]);
```

默认值使参数变为可选。通常我们可以省略类型注释，因为 TypeScript 可以推断类型。例如，它可以推断`x`和`y`都是`number`类型。

如果我们想添加类型注释，那么会是这样的。

```ts
function createPoint(x:number = 0, y:number = 0): [number, number] {
 return [x, y];
}
```

#### 7.10.4 剩余参数

我们还可以在 TypeScript 参数定义中使用[剩余参数](https://exploringjs.com/impatient-js/ch_callables.html#rest-parameters)。它们的静态类型必须是数组（列表或元组）：

```ts
function joinNumbers(...nums: number[]): string {
 return nums.join('-');
}
assert.equal(
 joinNumbers(1, 2, 3),
 '1-2-3');
```

### 7.11 联合类型

变量持有的值（一次一个值）可能是不同类型的成员。在这种情况下，我们需要*联合类型*。例如，在以下代码中，`stringOrNumber`的类型是`string`或`number`：

```ts
function getScore(stringOrNumber: string|number): number {
 if (typeof stringOrNumber === 'string'
 && /^\*{1,5}$/.test(stringOrNumber)) {
 return stringOrNumber.length;
 } else if (typeof stringOrNumber === 'number'
 && stringOrNumber >= 1 && stringOrNumber <= 5) {
 return stringOrNumber
 } else {
 throw new Error('Illegal value: ' + JSON.stringify(stringOrNumber));
 }
}

assert.equal(getScore('*****'), 5);
assert.equal(getScore(3), 3);
```

`stringOrNumber`的类型是`string|number`。类型表达式`s|t`的结果是类型`s`和`t`的集合论并集（解释为集合）。

#### 7.11.1 默认情况下，`undefined`和`null`不包括在类型中

在许多编程语言中，`null`是所有对象类型的一部分。例如，在 Java 中，当变量的类型是`String`时，我们可以将其设置为`null`，Java 不会抱怨。

相反，在 TypeScript 中，`undefined`和`null`由单独的不相交类型处理。如果我们想允许它们，我们需要联合类型，如`undefined|string`和`null|string`：

```ts
let maybeNumber: null|number = null;
maybeNumber = 123;
```

否则，我们会得到一个错误：

```ts
// @ts-expect-error: Type 'null' is not assignable to type 'number'. (2322)
let maybeNumber: number = null;
maybeNumber = 123;
```

请注意，TypeScript 不强制我们立即初始化（只要我们在初始化之前不从变量中读取）：

```ts
let myNumber: number; // OK
myNumber = 123;
```

#### 7.11.2 明确省略

回想一下之前的这个函数：

```ts
function stringify123(callback?: (num: number) => string) {
 if (callback === undefined) {
 callback = String;
 }
 return callback(123); // (A)
}
```

让我们重写`stringify123()`，使参数`callback`不再是可选的：如果调用者不想提供一个函数，他们必须明确传递`null`。结果如下。

```ts
function stringify123(
 callback: null | ((num: number) => string)) {
 const num = 123;
 if (callback === null) { // (A)
 callback = String;
 }
 return callback(num); // (B)
}

assert.equal(
 stringify123(null),
 '123');

// @ts-expect-error: Expected 1 arguments, but got 0\. (2554)
assert.throws(() => stringify123());
```

再次，我们必须在进行函数调用之前处理`callback`不是函数的情况（A 行），否则我们会得到一个错误。

### 7.12 可选 vs. 默认值 vs. `undefined|T`

以下三个参数声明非常相似：

+   参数是可选的：`x?: number`

+   参数有默认值：`x = 456`

+   参数有联合类型：`x: undefined | number`

如果参数是可选的，可以省略。在这种情况下，它的值是`undefined`：

```ts
function f1(x?: number) { return x }

assert.equal(f1(123), 123); // OK
assert.equal(f1(undefined), undefined); // OK
assert.equal(f1(), undefined); // can omit
```

如果参数有默认值，则在参数被省略或设置为`undefined`时使用该值：

```ts
function f2(x = 456) { return x }

assert.equal(f2(123), 123); // OK
assert.equal(f2(undefined), 456); // OK
assert.equal(f2(), 456); // can omit
```

如果参数具有联合类型，则不能省略，但是我们可以将其设置为`undefined`：

```ts
function f3(x: undefined | number) { return x }

assert.equal(f3(123), 123); // OK
assert.equal(f3(undefined), undefined); // OK

// @ts-expect-error: Expected 1 arguments, but got 0\. (2554)
f3(); // can’t omit
```

### 7.13 对象类型

与数组类似，对象在 JavaScript 中扮演两种角色（有时混合在一起）：

+   记录：在开发时已知的固定数量的属性。每个属性可以具有不同的类型。

+   字典：在开发时不知道名称的任意数量的属性。所有属性都具有相同的类型。

在本章中，我们忽略了对象作为字典的部分-它们在§15.4.5“索引签名：对象作为字典”中有涵盖。顺便说一句，Map 通常是字典的更好选择。

#### 7.13.1 通过接口对对象作为记录进行类型标注

接口描述对象作为记录。例如：

```ts
interface Point {
 x: number;
 y: number;
}
```

我们也可以用逗号分隔成员：

```ts
interface Point {
 x: number,
 y: number,
}
```

#### 7.13.2 TypeScript 的结构类型与名义类型

TypeScript 类型系统的一个重要优势是它是*结构化*的，而不是*名义化*的。也就是说，接口`Point`匹配所有具有适当结构的对象：

```ts
interface Point {
 x: number;
 y: number;
}
function pointToString(pt: Point) {
 return `(${pt.x}, ${pt.y})`;
}

assert.equal(
 pointToString({x: 5, y: 7}), // compatible structure
 '(5, 7)');
```

相反，在 Java 的名义类型系统中，我们必须在每个类中明确声明它实现的接口。因此，一个类只能实现在其创建时存在的接口。

#### 7.13.3 对象文字类型

*对象文字类型*是匿名接口：

```ts
type Point = {
 x: number;
 y: number;
};
```

对象文字类型的一个好处是它们可以内联使用：

```ts
function pointToString(pt: {x: number, y: number}) {
 return `(${pt.x}, ${pt.y})`;
}
```

#### 7.13.4 可选属性

如果属性可以省略，我们在其名称后面加上一个问号：

```ts
interface Person {
 name: string;
 company?: string;
}
```

在下面的示例中，`john`和`jane`都符合接口`Person`：

```ts
const john: Person = {
 name: 'John',
};
const jane: Person = {
 name: 'Jane',
 company: 'Massive Dynamic',
};
```

#### 7.13.5 方法

接口也可以包含方法：

```ts
interface Point {
 x: number;
 y: number;
 distance(other: Point): number;
}
```

就 TypeScript 的类型系统而言，方法定义和其值为函数的属性是等价的：

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

我的建议是使用最能表达属性设置方式的语法。

### 7.14 类型变量和通用类型

回顾 TypeScript 的两个语言级别：

+   值存在于*动态级别*。

+   类型存在于*静态级别*。

同样：

+   普通函数存在于动态级别，是值的工厂，并且具有表示值的参数。参数在括号之间声明：

    ```ts
    const valueFactory = (x: number) => x; // definition
    const myValue = valueFactory(123); // use
    ```

+   *通用类型*存在于静态级别，是类型的工厂，并且具有表示类型的参数。参数在尖括号之间声明：

    ```ts
    type TypeFactory<X> = X; // definition
    type MyType = TypeFactory<string>; // use
    ```

![](img/6a9318e9b4540abb73475976e01d55f9.png) **命名类型参数**

在 TypeScript 中，通常使用单个大写字符（如`T`，`I`和`O`）作为类型参数。但是，任何合法的 JavaScript 标识符都是允许的，而且更长的名称通常使代码更容易理解。

#### 7.14.1 示例：值的容器

```ts
// Factory for types
interface ValueContainer<Value> {
 value: Value;
}

// Creating one type
type StringContainer = ValueContainer<string>;
```

`Value`是一个*类型变量*。可以在尖括号之间引入一个或多个类型变量。

### 7.15 示例：一个通用类

类也可以有类型参数：

```ts
class SimpleStack<Elem> {
 #data: Array<Elem> = [];
 push(x: Elem): void {
 this.#data.push(x);
 }
 pop(): Elem {
 const result = this.#data.pop();
 if (result === undefined) {
 throw new Error();
 }
 return result;
 }
 get length() {
 return this.#data.length;
 }
}
```

类`SimpleStack`具有类型参数`Elem`。当我们实例化类时，我们还为类型参数提供一个值：

```ts
const stringStack = new SimpleStack<string>();
stringStack.push('first');
stringStack.push('second');
assert.equal(stringStack.length, 2);
assert.equal(stringStack.pop(), 'second');
```

#### 7.15.1 示例：Maps

在 TypeScript 中，Map 是带有泛型的。例如：

```ts
const myMap: Map<boolean,string> = new Map([
 [false, 'no'],
 [true, 'yes'],
]);
```

由于类型推断（基于`new Map()`的参数），我们可以省略类型参数：

```ts
// %inferred-type: Map<boolean, string>
const myMap = new Map([
 [false, 'no'],
 [true, 'yes'],
]);
```

#### 7.15.2 函数和方法的类型变量

函数定义可以像这样引入类型变量：

```ts
function identity<Arg>(arg: Arg): Arg {
 return arg;
}
```

我们使用函数如下。

```ts
// %inferred-type: number
const num1 = identity<number>(123);
```

由于类型推断，我们可以再次省略类型参数：

```ts
// %inferred-type: 123
const num2 = identity(123);
```

请注意，TypeScript 推断出了类型`123`，这是一个具有一个数字的集合，比类型`number`更具体。

##### 7.15.2.1 箭头函数和方法

箭头函数也可以有类型参数：

```ts
const identity = <Arg>(arg: Arg): Arg => arg;
```

这是方法的类型参数语法：

```ts
const obj = {
 identity<Arg>(arg: Arg): Arg {
 return arg;
 },
};
```

#### 7.15.3 更复杂的函数示例

```ts
function fillArray<T>(len: number, elem: T): T[] {
 return new Array<T>(len).fill(elem);
}
```

类型变量`T`在此代码中出现了四次：

+   它是通过`fillArray<T>`引入的。因此，它的范围是函数。

+   它首次用于参数`elem`的类型注释。

+   它第二次用于指定`fillArray()`的返回类型。

+   它也被用作构造函数`Array()`的类型参数。

我们可以在调用`fillArray()`（行 A）时省略类型参数，因为 TypeScript 可以从参数`elem`中推断出`T`：

```ts
// %inferred-type: string[]
const arr1 = fillArray<string>(3, '*');
assert.deepEqual(
 arr1, ['*', '*', '*']);

// %inferred-type: string[]
const arr2 = fillArray(3, '*'); // (A)
```

### 7.16 结论：理解初始示例

让我们使用我们之前学到的知识来理解我们之前看到的代码片段：

```ts
interface Array<T> {
 concat(...items: Array<T[] | T>): T[];
 reduce<U>(
 callback: (state: U, element: T, index: number, array: T[]) => U,
 firstState?: U
 ): U;
 // ···
}
```

这是一个数组的接口，其元素的类型为`T`：

+   方法`.concat()`有零个或多个参数（通过 rest 参数定义）。每个参数的类型为`T[]|T`。也就是说，它要么是`T`值的数组，要么是单个`T`值。

+   方法`.reduce()`引入了自己的类型变量`U`。`U`用于表示以下实体都具有相同类型的事实：

    +   `callback()`的`state`参数

    +   `callback()`的结果

    +   `.reduce()`的可选参数`firstState`

    +   `.reduce()`的结果

    除了`state`，`callback()`还有以下参数：

    +   `element`，其类型与数组元素的类型`T`相同

    +   `index`；一个数字

    +   具有类型`T`的元素的`array`

[评论](https://github.com/rauschma/tackling-ts/issues/7)
