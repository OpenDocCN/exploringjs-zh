# 20 函数类型

> 原文：[`exploringjs.com/tackling-ts/ch_typing-functions.html`](https://exploringjs.com/tackling-ts/ch_typing-functions.html)

* * *

+   20.1 定义静态类型函数

    +   20.1.1 函数声明

    +   20.1.2 箭头函数

+   20.2 函数类型

    +   20.2.1 函数类型签名

    +   20.2.2 带有调用签名的接口

    +   20.2.3 检查可调用值是否匹配函数类型

+   20.3 参数

    +   20.3.1 何时必须对参数进行类型注释？

    +   20.3.2 可选参数

    +   20.3.3 剩余参数

    +   20.3.4 命名参数

    +   20.3.5 `this`作为参数（高级）

+   20.4 重载（高级）

    +   20.4.1 重载函数声明

    +   20.4.2 通过接口进行重载

    +   20.4.3 基于字符串参数的重载（事件处理等）

    +   20.4.4 重载方法

+   20.5 可赋值性（高级）

    +   20.5.1 可赋值性规则

    +   20.5.2 函数赋值规则的后果

+   20.6 进一步阅读和本章的来源

* * *

本章探讨了 TypeScript 中函数的静态类型。 

![](img/65d35c0a2478236e12cc4321e1b02db6.png)  **在本章中，“函数”指的是“函数或方法或构造函数”**

在本章中，关于函数的大部分内容（特别是参数处理方面）也适用于方法和构造函数。

### 20.1 定义静态类型函数

#### 20.1.1 函数声明

这是 TypeScript 中函数声明的一个例子：

```ts
function repeat1(str: string, times: number): string { // (A)
 return str.repeat(times);
}
assert.equal(
 repeat1('*', 5), '*****');
```

+   参数：如果编译器选项`--noImplicitAny`打开（如果`--strict`打开），则每个参数的类型必须是可推断的或明确指定的。（我们稍后会更仔细地看一下推断。）在这种情况下，无法进行推断，这就是为什么`str`和`times`有类型注释的原因。

+   返回值：默认情况下，函数的返回类型是推断的。通常这已经足够好了。在这种情况下，我们选择明确指定`repeat1()`的返回类型为`string`（A 行中的最后一个类型注释）。

#### 20.1.2 箭头函数

`repeat1()`的箭头函数版本如下所示：

```ts
const repeat2 = (str: string, times: number): string => {
 return str.repeat(times);
};
```

在这种情况下，我们也可以使用表达式体：

```ts
const repeat3 = (str: string, times: number): string =>
 str.repeat(times);
```

### 20.2 函数类型

#### 20.2.1 函数类型签名

我们可以通过函数类型签名为函数定义类型：

```ts
type Repeat = (str: string, times: number) => string;
```

这种类型的函数名为`Repeat`。它匹配所有具有以下特征的函数：

+   两个类型分别为`string`和`number`的参数。我们需要在函数类型签名中命名参数，但在检查两个函数类型是否兼容时，名称会被忽略。

+   返回类型为`string`。请注意，这次类型是由箭头分隔的，不能省略。 

这种类型匹配更多的函数。我们将在本章后面探讨*可赋值性*的规则时学习到更多信息。

#### 20.2.2 具有调用签名的接口

我们还可以使用接口来定义函数类型：

```ts
interface Repeat {
 (str: string, times: number): string; // (A)
}
```

注意：

+   A 行中的接口成员是*调用签名*。它看起来类似于方法签名，但没有名称。

+   结果的类型由冒号（而不是箭头）分隔，并且不能被省略。

一方面，接口更冗长。另一方面，它们让我们指定函数的属性（这很少见，但确实会发生）：

```ts
interface Incrementor1 {
 (x: number): number;
 increment: number;
}
```

我们还可以通过函数签名类型和对象字面类型的交集类型（`&`）来指定属性：

```ts
type Incrementor2 =
 (x: number) => number
 & { increment: number }
;
```

#### 20.2.3 检查可调用值是否与函数类型匹配

例如，考虑以下情景：一个库导出以下函数类型。

```ts
type StringPredicate = (str: string) => boolean;
```

我们想要定义一个类型与`StringPredicate`兼容的函数。并且我们希望立即检查是否确实如此（而不是在第一次使用时才发现）。

##### 20.2.3.1 检查箭头函数

如果我们通过`const`声明一个变量，我们可以通过类型注释进行检查：

```ts
const pred1: StringPredicate = (str) => str.length > 0;
```

注意，我们不需要指定参数`str`的类型，因为 TypeScript 可以使用`StringPredicate`来推断它。

##### 20.2.3.2 检查函数声明（简单）

检查函数声明更加复杂：

```ts
function pred2(str: string): boolean {
 return str.length > 0;
}

// Assign the function to a type-annotated variable
const pred2ImplementsStringPredicate: StringPredicate = pred2;
```

##### 20.2.3.3 检查函数声明（奢侈的）

以下解决方案有点过头（即，如果你不完全理解也不要担心），但它演示了几个高级特性：

```ts
function pred3(...[str]: Parameters<StringPredicate>)
 : ReturnType<StringPredicate> {
 return str.length > 0;
 }
```

+   参数：我们使用`Parameters<>`来提取具有参数类型的元组。三个点声明了一个剩余参数，它收集元组/数组中的所有参数。`[str]`对该元组进行解构。（本章后面将更多介绍剩余参数。）

+   返回值：我们使用`ReturnType<>`来提取返回类型。

### 20.3 参数

#### 20.3.1 何时必须对参数进行类型注释？

回顾：如果打开了`--noImplicitAny`（`--strict`会打开它），则每个参数的类型必须是可推断的或明确指定的。

在以下示例中，TypeScript 无法推断`str`的类型，我们必须指定它：

```ts
function twice(str: string) {
 return str + str;
}
```

在 A 行，TypeScript 可以使用类型`StringMapFunction`来推断`str`的类型，我们不需要添加类型注释：

```ts
type StringMapFunction = (str: string) => string;
const twice: StringMapFunction = (str) => str + str; // (A)
```

在这里，TypeScript 可以使用`.map()`的类型来推断`str`的类型：

```ts
assert.deepEqual(
 ['a', 'b', 'c'].map((str) => str + str),
 ['aa', 'bb', 'cc']);
```

这是`.map()`的类型：

```ts
interface Array<T> {
 map<U>(
 callbackfn: (value: T, index: number, array: T[]) => U,
 thisArg?: any
 ): U[];
 // ···
}
```

#### 20.3.2 可选参数

在本节中，我们将看几种允许参数被省略的方法。

##### 20.3.2.1 可选参数：`str?: string`

如果在参数名称后面加上问号，该参数就变成了可选的，在调用函数时可以省略：

```ts
function trim1(str?: string): string {
 // Internal type of str:
 // %inferred-type: string | undefined
 str;

 if (str === undefined) {
 return '';
 }
 return str.trim();
}

// External type of trim1:
// %inferred-type: (str?: string | undefined) => string
trim1;
```

这是`trim1()`的调用方式：

```ts
assert.equal(
 trim1('\n abc \t'), 'abc');

assert.equal(
 trim1(), '');

// `undefined` is equivalent to omitting the parameter
assert.equal(
 trim1(undefined), '');
```

##### 20.3.2.2 联合类型：`str: undefined|string`

在外部，`trim1()`的参数`str`的类型是`string|undefined`。因此，`trim1()`在大多数情况下等同于以下函数。

```ts
function trim2(str: undefined|string): string {
 // Internal type of str:
 // %inferred-type: string | undefined
 str;

 if (str === undefined) {
 return '';
 }
 return str.trim();
}

// External type of trim2:
// %inferred-type: (str: string | undefined) => string
trim2;
```

`trim2()`与`trim1()`唯一不同的地方是在函数调用时不能省略参数（A 行）。换句话说：当省略类型为`undefined|T`的参数时，我们必须明确指定。

```ts
assert.equal(
 trim2('\n abc \t'), 'abc');

// @ts-expect-error: Expected 1 arguments, but got 0\. (2554)
trim2(); // (A)

assert.equal(
 trim2(undefined), ''); // OK!
```

##### 20.3.2.3 参数默认值：`str = ''`

如果我们为`str`指定了参数默认值，我们不需要提供类型注释，因为 TypeScript 可以推断类型：

```ts
function trim3(str = ''): string {
 // Internal type of str:
 // %inferred-type: string
 str;

 return str.trim();
}

// External type of trim2:
// %inferred-type: (str?: string) => string
trim3;
```

注意，`str`的内部类型是`string`，因为默认值确保它永远不会是`undefined`。

让我们调用`trim3()`：

```ts
assert.equal(
 trim3('\n abc \t'), 'abc');

// Omitting the parameter triggers the parameter default value:
assert.equal(
 trim3(), '');

// `undefined` is allowed and triggers the parameter default value:
assert.equal(
 trim3(undefined), '');
```

##### 20.3.2.4 参数默认值加类型注释

我们还可以同时指定类型和默认值：

```ts
function trim4(str: string = ''): string {
 return str.trim();
}
```

#### 20.3.3 剩余参数

##### 20.3.3.1 具有数组类型的剩余参数

剩余参数将所有剩余参数收集到一个数组中。因此，它的静态类型通常是数组。在下面的例子中，`parts`是一个剩余参数：

```ts
function join(separator: string, ...parts: string[]) {
 return parts.join(separator);
}
assert.equal(
 join('-', 'state', 'of', 'the', 'art'),
 'state-of-the-art');
```

##### 20.3.3.2 具有元组类型的剩余参数

下一个示例演示了两个特性：

+   我们可以使用元组类型，如`[string, number]`，来作为剩余参数。

+   我们可以解构剩余参数（不仅仅是普通参数）。

```ts
function repeat1(...[str, times]: [string, number]): string {
 return str.repeat(times);
}
```

`repeat1()`等同于以下函数：

```ts
function repeat2(str: string, times: number): string {
 return str.repeat(times);
}
```

#### 20.3.4 命名参数

[*命名参数*](https://exploringjs.com/impatient-js/ch_callables.html#named-parameters)是 JavaScript 中的一种流行模式，其中使用对象文字为每个参数指定名称。看起来如下：

```ts
assert.equal(
 padStart({str: '7', len: 3, fillStr: '0'}),
 '007');
```

在纯 JavaScript 中，函数可以使用解构来访问命名参数值。遗憾的是，在 TypeScript 中，我们还必须为对象文字指定类型，这导致了冗余：

```ts
function padStart({ str, len, fillStr = ' ' } // (A)
 : { str: string, len: number, fillStr: string }) { // (B)
 return str.padStart(len, fillStr);
}
```

请注意，解构（包括`fillStr`的默认值）都发生在 A 行，而 B 行完全是关于 TypeScript 的。

可以定义一个单独的类型，而不是我们在 B 行中使用的内联对象文字类型。但是，在大多数情况下，我更喜欢不这样做，因为它略微违反了参数的本质，参数是每个函数的本地和唯一的。如果您更喜欢在函数头中有更少的内容，那也可以。

#### 20.3.5 `this`作为参数（高级）

每个普通函数始终具有隐式参数`this`-这使其可以在对象中用作方法。有时我们需要为`this`指定类型。对于这种用例，TypeScript 有专门的语法：普通函数的参数之一可以命名为`this`。这样的参数仅在编译时存在，并在运行时消失。

例如，考虑以下用于 DOM 事件源的接口（稍微简化版本）：

```ts
interface EventSource {
 addEventListener(
 type: string,
 listener: (this: EventSource, ev: Event) => any,
 options?: boolean | AddEventListenerOptions
 ): void;
 // ···
}
```

回调`listener`的`this`始终是`EventSource`的实例。

下一个示例演示了 TypeScript 如何使用`this`参数提供的类型信息来检查`.call()`的第一个参数（A 行和 B 行）：

```ts
function toIsoString(this: Date): string {
 return this.toISOString();
}

// @ts-expect-error: Argument of type '"abc"' is not assignable to
// parameter of type 'Date'. (2345)
assert.throws(() => toIsoString.call('abc')); // (A) error

toIsoString.call(new Date()); // (B) OK
```

此外，我们不能将`toIsoString()`作为对象`obj`的方法调用，因为它的接收器不是`Date`的实例：

```ts
const obj = { toIsoString };
// @ts-expect-error: The 'this' context of type
// '{ toIsoString: (this: Date) => string; }' is not assignable to
// method's 'this' of type 'Date'. [...]
assert.throws(() => obj.toIsoString()); // error
obj.toIsoString.call(new Date()); // OK
```

### 20.4 重载（高级）

有时单个类型签名无法充分描述函数的工作原理。

#### 20.4.1 重载函数声明

考虑我们在以下示例中调用的`getFullName()`函数（A 行和 B 行）：

```ts
interface Customer {
 id: string;
 fullName: string;
}
const jane = {id: '1234', fullName: 'Jane Bond'};
const lars = {id: '5678', fullName: 'Lars Croft'};
const idToCustomer = new Map<string, Customer>([
 ['1234', jane],
 ['5678', lars],
]);

assert.equal(
 getFullName(idToCustomer, '1234'), 'Jane Bond'); // (A)

assert.equal(
 getFullName(lars), 'Lars Croft'); // (B)
```

我们如何实现`getFullName()`？以下实现适用于前面示例中的两个函数调用：

```ts
function getFullName(
 customerOrMap: Customer | Map<string, Customer>,
 id?: string
): string {
 if (customerOrMap instanceof Map) {
 if (id === undefined) throw new Error();
 const customer = customerOrMap.get(id);
 if (customer === undefined) {
 throw new Error('Unknown ID: ' + id);
 }
 customerOrMap = customer;
 } else {
 if (id !== undefined) throw new Error();
 }
 return customerOrMap.fullName;
}
```

但是，使用这种类型签名，编译时可以产生运行时错误的函数调用是合法的：

```ts
assert.throws(() => getFullName(idToCustomer)); // missing ID
assert.throws(() => getFullName(lars, '5678')); // ID not allowed
```

以下代码修复了这些问题：

```ts
function getFullName(customerOrMap: Customer): string; // (A)
function getFullName( // (B)
 customerOrMap: Map<string, Customer>, id: string): string;
function getFullName( // (C)
 customerOrMap: Customer | Map<string, Customer>,
 id?: string
): string {
 // ···
}

// @ts-expect-error: Argument of type 'Map<string, Customer>' is not
// assignable to parameter of type 'Customer'. [...]
getFullName(idToCustomer); // missing ID

// @ts-expect-error: Argument of type '{ id: string; fullName: string; }'
// is not assignable to parameter of type 'Map<string, Customer>'.
// [...]
getFullName(lars, '5678'); // ID not allowed
```

这里发生了什么？`getFullName()`的类型签名被重载：

+   实际实现从 C 行开始。与前面的示例相同。

+   在 A 行和 B 行中，有两个类型签名（没有主体的函数头），可以用于`getFullName()`。实际实现的类型签名不能使用！

我的建议是只有在无法避免时才使用重载。一种替代方法是将重载的函数拆分为具有不同名称的多个函数-例如：

+   `getFullName()`

+   `getFullNameViaMap()`

#### 20.4.2 通过接口进行重载

在接口中，我们可以有多个不同的调用签名。这使我们能够在以下示例中使用接口`GetFullName`进行重载：

```ts
interface GetFullName {
 (customerOrMap: Customer): string;
 (customerOrMap: Map<string, Customer>, id: string): string;
}

const getFullName: GetFullName = (
 customerOrMap: Customer | Map<string, Customer>,
 id?: string
): string => {
 if (customerOrMap instanceof Map) {
 if (id === undefined) throw new Error();
 const customer = customerOrMap.get(id);
 if (customer === undefined) {
 throw new Error('Unknown ID: ' + id);
 }
 customerOrMap = customer;
 } else {
 if (id !== undefined) throw new Error();
 }
 return customerOrMap.fullName;
}
```

#### 20.4.3 基于字符串参数的重载（事件处理等）

在下一个示例中，我们通过接口进行重载并使用字符串文字类型（例如`'click'`）。这使我们能够根据参数`type`的值更改参数`listener`的类型：

```ts
function addEventListener(elem: HTMLElement, type: 'click',
 listener: (event: MouseEvent) => void): void;
function addEventListener(elem: HTMLElement, type: 'keypress',
 listener: (event: KeyboardEvent) => void): void;
function addEventListener(elem: HTMLElement, type: string,  // (A)
 listener: (event: any) => void): void {
 elem.addEventListener(type, listener); // (B)
 }
```

在这种情况下，相对难以正确获取实现的类型（从 A 行开始）以使主体中的语句（B 行）起作用。作为最后的手段，我们总是可以使用类型`any`。

#### 20.4.4 重载方法

##### 20.4.4.1 重载具体方法

下一个示例演示了方法的重载：方法`.add()`被重载。

```ts
class StringBuilder {
 #data = '';

 add(num: number): this;
 add(bool: boolean): this;
 add(str: string): this;
 add(value: any): this {
 this.#data += String(value);
 return this;
 }

 toString() {
 return this.#data;
 }
}

const sb = new StringBuilder();
sb
 .add('I can see ')
 .add(3)
 .add(' monkeys!')
;
assert.equal(
 sb.toString(), 'I can see 3 monkeys!')
```

##### 20.4.4.2 重载接口方法

`Array.from()`的类型定义是重载接口方法的一个示例：

```ts
interface ArrayConstructor {
 from<T>(arrayLike: ArrayLike<T>): T[];
 from<T, U>(
 arrayLike: ArrayLike<T>,
 mapfn: (v: T, k: number) => U,
 thisArg?: any
 ): U[];
}
```

+   在第一个签名中，返回的数组具有与参数相同的元素类型。

+   在第二个签名中，返回的数组元素与`mapfn`的结果具有相同的类型。这个版本的`Array.from()`类似于`Array.prototype.map()`。

### 20.5 可分配性（高级）

在本节中，我们将研究*可分配性*的类型兼容性规则：类型为`Src`的函数是否可以转移到类型为`Trg`的存储位置（变量、对象属性、参数等）？

理解可分配性有助于我们回答诸如：

+   在函数调用中，对于形式参数的函数类型签名，哪些函数可以作为实际参数传递？

+   对于属性的函数类型签名，可以分配给它的函数是哪些？

#### 20.5.1 可分配性的规则

在本小节中，我们将研究可分配性的一般规则（包括函数的规则）。在下一小节中，我们将探讨这些规则对函数的含义。

如果以下条件之一成立，则类型`Src`可以[分配](https://github.com/microsoft/TypeScript/blob/master/doc/spec-ARCHIVED.md#3114-assignment-compatibility)给类型`Trg`：

+   `Src`和`Trg`是相同的类型。

+   `Src`或`Trg`是`any`类型。

+   `Src`是一个字符串字面类型，`Trg`是原始类型 String。

+   `Src`是一个联合类型，`Src`的每个组成类型都可以分配给`Trg`。

+   `Src`和`Trg`是函数类型，并且：

    +   `Trg`具有剩余参数，或者`Src`的必需参数数量小于或等于`Trg`的总参数数量。

    +   对于两个签名中都存在的参数，`Trg`中的每个参数类型都可以分配给`Src`中对应的参数类型。

    +   `Trg`的返回类型是`void`，或者`Src`的返回类型可以分配给`Trg`的返回类型。

+   （其余条件被省略。）

#### 20.5.2 函数分配规则的后果

在本小节中，我们将研究分配规则对以下两个函数`targetFunc`和`sourceFunc`的含义：

```ts
const targetFunc: Trg = sourceFunc;
```

##### 20.5.2.1 参数和结果的类型

+   目标参数类型必须可以分配给相应的源参数类型。

    +   为什么？目标接受的任何内容也必须被源接受。

+   源返回类型必须可以分配给目标返回类型。

    +   为什么？源返回的任何内容都必须与目标设置的期望兼容。

示例：

```ts
const trg1: (x: RegExp) => Object = (x: Object) => /abc/;
```

以下示例演示了如果目标返回类型是`void`，那么源返回类型就不重要。为什么？在 TypeScript 中，`void`结果总是被忽略的。

```ts
const trg2: () => void = () => new Date();
```

##### 20.5.2.2 参数的数量

源不得比目标具有更多的参数：

```ts
// @ts-expect-error: Type '(x: string) => string' is not assignable to
// type '() => string'. (2322)
const trg3: () => string = (x: string) => 'abc';
```

源可以比目标具有更少的参数：

```ts
const trg4: (x: string) => string = () => 'abc';
```

为什么？目标指定了对源的期望：它必须接受参数`x`。它确实接受了（但它忽略了它）。这种宽松性使得：

```ts
['a', 'b'].map(x => x + x)
```

`.map()`的回调只有三个参数中的一个：

```ts
map<U>(
 callback: (value: T, index: number, array: T[]) => U,
 thisArg?: any
): U[];
```

### 20.6 本章的进一步阅读和来源

+   [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/basic-types.html)

+   [TypeScript 语言规范](https://github.com/microsoft/TypeScript/blob/master/doc/spec-ARCHIVED.md)

+   [章节“可调用值”](https://exploringjs.com/impatient-js/ch_callables.html) in “JavaScript for impatient programmers”

[评论](https://github.com/rauschma/tackling-ts/issues/20)
