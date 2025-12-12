# 27 函数类型化

> 原文：[`exploringjs.com/ts/book/ch_typing-functions.html`](https://exploringjs.com/ts/book/ch_typing-functions.html)

(广告，请勿拦截。)

1.  27.1 定义静态类型函数

    1.  27.1.1 函数声明

    1.  27.1.2 箭头函数

1.  27.2 函数类型

    1.  27.2.1 函数类型签名

    1.  27.2.2 具有调用签名的接口

    1.  27.2.3 检查可调用值是否匹配函数类型

1.  27.3 参数

    1.  27.3.1 何时需要为参数添加类型注解？

    1.  27.3.2 可选参数

    1.  27.3.3 剩余参数

    1.  27.3.4 命名参数

    1.  27.3.5 `this`作为参数（高级）

1.  27.4 返回类型`never`：不返回的函数

    1.  27.4.1 反对返回类型`never | T`的原因

    1.  27.4.2 `@types/node`中的返回类型`never`

1.  27.5 重载（高级）

    1.  27.5.1 重载函数声明

    1.  27.5.2 通过元组类型的联合重载函数

    1.  27.5.3 通过接口重载函数

    1.  27.5.4 基于字符串参数的重载（事件处理等）

    1.  27.5.5 重载方法

1.  27.6 可赋值性（高级）

    1.  27.6.1 可赋值性的规则

    1.  27.6.2 函数赋值规则的影响

1.  27.7 本章的进一步阅读和资源

本章探讨了 TypeScript 中函数的静态类型。

![图标“阅读”](img/00b0d6029a045810b908b88d1a6733d2.png) **在本章中，“函数”指的是“函数、方法或构造函数”**

在本章中，关于函数的大部分内容（尤其是关于参数处理的内容），也适用于方法和构造函数。

### 27.1 定义静态类型函数

#### 27.1.1 函数声明

这是一个 TypeScript 中函数声明的示例：

```ts
function repeat1(str: string, times: number): string { // (A)
 return str.repeat(times);
}
assert.equal(
 repeat1('*', 5), '*****'
);

```

+   参数：如果编译器选项`--noImplicitAny`开启（如果`--strict`开启，则默认开启），则每个参数的类型必须是可推断的或显式指定的。（我们稍后会详细探讨推断。）对于参数，无法进行推断，这就是为什么`str`和`times`有类型注解。

+   返回类型：函数的默认返回类型是推断的。这通常足够好。我们选择显式指定 `repeat1()` 的返回类型为 `string`（行 A 中的最后一个类型注解）。

#### 27.1.2 箭头函数

`repeat1()` 的箭头函数版本如下所示：

```ts
const repeat2 = (str: string, times: number): string => {
 return str.repeat(times);
};

```

箭头函数也可以有表达式体：

```ts
const repeat3 = (str: string, times: number): string =>
 str.repeat(times);

```

### 27.2 函数类型

#### 27.2.1 函数类型签名

我们可以通过函数类型签名来定义函数类型：

```ts
type Repeat = (str: string, times: number) => string;

```

这个函数类型的名称是 `Repeat`。它有：

+   两个类型为 `string` 和 `number` 的参数。我们需要在函数类型签名中命名参数，但在检查两个函数类型是否兼容时，这些名称会被忽略。

+   返回类型 `string`。注意，这次类型是由箭头分隔的，不能省略。

哪些函数可以分配给这个类型？至少是那些具有相同参数类型和返回类型的函数。但还有一些其他的函数也可以分配。我们将在本章后面探索可分配性规则时看到哪些。

#### 27.2.2 具有调用签名的接口

我们也可以使用接口来定义函数类型：

```ts
interface Repeat {
 (str: string, times: number): string; // (A)
}

```

注意：

+   行 A 中的接口成员是一个 *调用签名*。它看起来类似于方法签名，但没有名称。

+   结果的类型由冒号（而不是箭头）分隔，不能省略。

一方面，接口更冗长。另一方面，它们允许我们指定函数的属性（虽然很少见，但确实会发生）：

```ts
interface Incrementor1 {
 (x: number): number;
 increment: number;
}

```

我们也可以通过函数签名类型和对象字面量类型的交集类型（`&`）来指定属性：

```ts
type Incrementor2 =
 & ((x: number) => number)
 & { increment: number }
;

```

#### 27.2.3 检查可调用值是否匹配函数类型

例如，考虑以下场景：一个库导出以下函数类型。

```ts
type StringPredicate = (str: string) => boolean;

```

我们想定义一个与 `StringPredicate` 兼容类型的函数，并且我们想立即检查这确实是这样（而不是在我们第一次使用它时才发现）。

##### 27.2.3.1 检查箭头函数

如果我们通过 `const` 声明一个变量，我们可以通过类型注解来执行检查：

```ts
const pred1: StringPredicate = (str) => str.length > 0;

```

注意，我们不需要指定参数 `str` 的类型，因为 TypeScript 可以使用 `StringPredicate` 来推断它。

##### 27.2.3.2 检查函数声明（简单）

检查函数声明更复杂。考虑以下函数声明：

```ts
function pred2(str: string): boolean {
 return str.length > 0;
}

```

这是我们检查可分配性的两种内置方式：

```ts
const pred2ImplementsStringPredicate: StringPredicate = pred2;
pred2 satisfies StringPredicate;

```

`satisfies` 在 “`satisfies` 操作符”（§29） 中解释。

以下两个检查需要库 `asserttt`：

```ts
assertType<StringPredicate>(pred2);
type _ = Assert<Assignable<
 StringPredicate,
 typeof pred2
>>;

```

注意，除了最后一个检查之外，所有检查都会产生不必要的 JavaScript 代码。

##### 27.2.3.3 检查函数声明（奢侈）

以下解决方案是你通常不会做的事情——所以如果你不完全理解它，请不要担心。但它很好地展示了几个高级功能：

```ts
function pred3(
 ...[str]: Parameters<StringPredicate>
): ReturnType<StringPredicate>
{
 return str.length > 0;
}

```

+   参数：我们使用实用类型 `Parameters<>` 来提取一个包含参数类型的元组。三个点声明了一个剩余参数，它收集所有参数到一个元组/数组中。`[str]` 解构这个元组。（关于剩余参数的更多内容将在本章后面介绍。）

+   返回值：我们使用实用类型 `ReturnType<>` 来提取返回类型。

内置实用类型 `Parameters<>` 和 `ReturnType<>` 在 “通过 `infer` 提取函数类型的一部分” (§35.3.1) 中进行了解释。

### 27.3 参数

#### 27.3.1 参数何时需要类型注解？

回顾：如果 `--noImplicitAny` 被打开（`--strict` 会打开它），每个参数的类型必须是可以推断的或必须显式指定。

在以下示例中，TypeScript 无法推断 `str` 的类型，我们必须指定它：

```ts
function twice(str: string) {
 return str + str;
}

```

在行 A 中，TypeScript 可以使用类型 `StringMapFunction` 来推断 `str` 的类型，我们不需要添加类型注解：

```ts
type StringMapFunction = (str: string) => string;
const twice: StringMapFunction = (str) => str + str; // (A)

```

在这里，TypeScript 可以使用 `.map()` 的类型来推断 `str` 的类型：

```ts
assert.deepEqual(
 ['a', 'b', 'c'].map((str) => str + str),
 ['aa', 'bb', 'cc']
);

```

这是一种 `.map()` 类型：

```ts
interface Array<T> {
 map<U>(
 callbackfn: (value: T, index: number, array: T[]) => U,
 thisArg?: any
 ): U[];
 // ···
}

```

#### 27.3.2 可选参数

在本节中，我们探讨了几种允许省略参数的方法。

##### 27.3.2.1 可选参数：`str?: string`

如果我们在参数名称后加上问号，该参数变为可选的，在调用函数时可以省略：

```ts
function trim1(str?: string): string {
 // Internal type of str:
 assertType<string | undefined>(str);

 if (str === undefined) {
 return '';
 }
 return str.trim();
}

// External type of trim1:
type _ = Assert<Equal<
 typeof trim1,
 (str?: string | undefined) => string
>>;

```

这是调用 `trim1()` 的方法：

```ts
assert.equal(
 trim1('\n  abc \t'), 'abc'
);

assert.equal(
 trim1(), ''
);

// `undefined` is equivalent to omitting the parameter
assert.equal(
 trim1(undefined), ''
);

```

作为旁注，以下两种类型是相等的（`Equal<>` 是严格检查）因为可选修饰符（`?`）意味着类型 `undefined`：

```ts
type _ = Assert<Equal<
 (str?: string | undefined) => string,
 (str?: string) => string
>>;

```

##### 27.3.2.2 联合类型：`str: string | undefined`

外部，`trim1()` 的参数 `str` 类型为 `string | undefined`。因此，`trim1()` 主要等同于以下函数。

```ts
function trim2(str: string | undefined): string {
 // Internal type of str:
 assertType<string | undefined>(str);

 if (str === undefined) {
 return '';
 }
 return str.trim();
}

// External type of trim2:
type _ = Assert<Equal<
 typeof trim2,
 (str: string | undefined) => string
>>;

```

`trim2()` 与 `trim1()` 的唯一区别是参数在函数调用中不能省略（行 A）。换句话说：当我们省略类型为 `T|undefined` 的参数时，我们必须明确。

```ts
assert.equal(
 trim2('\n  abc \t'), 'abc'
);

// @ts-expect-error: Expected 1 arguments, but got 0.
trim2(); // (A)

assert.equal(
 trim2(undefined), '' // OK!
);

```

##### 27.3.2.3 参数默认值：`str = ''`

如果我们为 `str` 指定默认值，我们不需要提供类型注解，因为 TypeScript 可以推断类型：

```ts
function trim3(str = ''): string {
 // Internal type of str:
 assertType<string>(str);

 return str.trim();
}

// External type of trim3:
type _ = Assert<Equal<
 typeof trim3,
 (str?: string | undefined) => string
>>;

```

`str` 的内部类型是 `string`，因为默认值确保它永远不会是 `undefined`。

让我们调用 `trim3()`:

```ts
assert.equal(
 trim3('\n  abc \t'), 'abc');

// Omitting the parameter triggers the parameter default value:
assert.equal(
 trim3(), '');

// `undefined` is allowed and triggers the parameter default value:
assert.equal(
 trim3(undefined), '');

```

##### 27.3.2.4 参数默认值加上类型注解

我们也可以同时指定类型和默认值：

```ts
function trim4(str: string = ''): string {
 return str.trim();
}

```

#### 27.3.3 剩余参数

剩余参数收集所有剩余的参数到一个数组中。因此，它的静态类型是数组或元组。

##### 27.3.3.1 带有数组类型的剩余参数

在以下示例中，剩余参数 `parts` 具有数组类型：

```ts
function join(separator: string, ...parts: Array<string>) {
 return parts.join(separator);
}
assert.equal(
 join('-', 'state', 'of', 'the', 'art'),
 'state-of-the-art');

```

##### 27.3.3.2 带有元组类型的剩余参数

下一个示例演示了两个功能：

+   我们使用元组类型 `[string, number]` 作为剩余参数。

+   我们解构剩余参数 – 这是一个 JavaScript 功能。

```ts
function repeat1(...[str, times]: [string, number]): string {
 return str.repeat(times);
}

```

`repeat1()` 等同于以下函数：

```ts
function repeat2(str: string, times: number): string {
 return str.repeat(times);
}

```

#### 27.3.4 命名参数

[*命名参数*](https://exploringjs.com/js/book/ch_callables.html#named-parameters) 是 JavaScript 中的一种流行模式，其中使用对象字面量为每个参数命名。其外观如下：

```ts
assert.equal(
 padStart({str: '7', len: 3, fillStr: '0'}),
 '007'
);

```

在纯 JavaScript 中，函数可以使用解构来访问命名参数值。然而，在 TypeScript 中，我们还需要为对象字面量指定一个类型，这会导致冗余：

```ts
function padStart(
 { str, len, fillStr = ' ' } // (A)
 : { str: string, len: number, fillStr: string } // (B)
): string {
 return str.padStart(len, fillStr);
}

```

注意，解构（包括 `fillStr` 的默认值）都在行 A 中完成，而行 B 则完全关于 TypeScript。

我们也可以定义一个单独的类型，而不是在行 B 中使用的内联对象字面量类型：

```ts
function padStart(
 { str, len, fillStr = ' ' }: PadStartArgs
): string {
 return str.padStart(len, fillStr);
}
type PadStartArgs = {
 str: string,
 len: number,
 fillStr: string,
};

```

一个优点是函数声明更加简洁。一个缺点是我们必须在其他地方查看类型。

#### 27.3.5 `this` 作为参数（高级）

每个普通函数始终都有一个隐式参数 `this` – 这使得它可以在对象中用作方法。有时我们需要为 `this` 指定一个类型。对于这种情况，TypeScript 提供了专门的语法：普通函数的一个参数可以命名为 `this`。这样的参数仅在编译时存在，在运行时消失。

例如，考虑以下 DOM 事件源的接口（在略微简化的版本中）：

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

回调 `listener` 的 `this` 总是 `EventSource` 的一个实例。

下一个示例演示 TypeScript 如何使用 `this` 参数提供的类型信息来检查 `.call()` 的第一个参数（行 A 和行 B）：

```ts
function toIsoString(this: Date): string {
 return this.toISOString();
}

// @ts-expect-error: Argument of type 'string' is not assignable to
// parameter of type 'Date'.
assert.throws(() => toIsoString.call('abc')); // (A) error

toIsoString.call(new Date()); // (B) OK

```

此外，我们无法将 `toIsoString()` 作为对象 `obj` 的方法调用，因为这样它的接收者不是 `Date` 的实例：

```ts
const obj = { toIsoString };
// @ts-expect-error: The 'this' context of type
// '{ toIsoString: (this: Date) => string; }' is not assignable to
// method's 'this' of type 'Date'.
assert.throws(() => obj.toIsoString()); // error
obj.toIsoString.call(new Date()); // OK

```

### 27.4 返回类型 `never`：不返回的函数

`never` 还可以作为永远不会返回的函数的标记 – 例如：

```ts
function throwError(message: string): never {
 throw new Error(message);
}
function infiniteLoop(): never {
 while (true) {}
}

```

TypeScript 的类型推断会考虑这样的函数。例如，`returnStringIfTrue()` 推断的返回类型是 `string`，因为我们调用了行 A 中的 `throwError()`。

```ts
function returnStringIfTrue(flag: boolean) {
 if (flag) {
 return 'abc';
 }
 throwError('Flag must be true'); // (A)
}
type _ = Assert<Equal<
 ReturnType<typeof returnStringIfTrue>,
 string
>>;

```

如果我们省略行 A，则会得到一个错误，并且推断的返回类型是 `'abc' | undefined`：

```ts
// @ts-expect-error: Not all code paths return a value.
function returnStringIfTrue(flag: boolean) {
 if (flag) {
 return 'abc';
 }
}
type _ = Assert<Equal<
 ReturnType<typeof returnStringIfTrue>,
 'abc' | undefined
>>;

```

#### 27.4.1 反对返回类型 `never | T` 的理由

原则上，我们可以使用类型 `never | T` 来表示一个函数，在某些情况下会抛出异常而不是正常返回。然而，有两大理由反对这样做：

+   抛出异常通常不会改变函数的返回类型。这就是为什么它被称为*异常*。

+   `never | T` 与 `T` 相同（如我们在这章前面所见）。

#### 27.4.2 `@types/node`中的返回类型`never`

在 Node.js 中，以下函数的返回类型为`never`：

+   `process.exit()`

+   `process.abort()`

+   `assert.fail()`

### 27.5 重载（高级）

有时候，单个类型签名不足以充分描述函数的工作方式。

#### 27.5.1 函数声明重载

考虑以下例子中我们调用的函数`getFullName()`（行 A 和行 B）：

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
 getFullName(idToCustomer, '1234'), 'Jane Bond' // (A)
);
assert.equal(
 getFullName(lars), 'Lars Croft' // (B)
);

```

我们将如何实现`getFullName()`？以下实现适用于前面例子中的两个函数调用：

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

然而，使用这个类型签名，编译时合法的函数调用在运行时会产生错误：

```ts
assert.throws(() => getFullName(idToCustomer)); // missing ID
assert.throws(() => getFullName(lars, '5678')); // ID not allowed

```

以下代码解决了这些问题：

```ts
function getFullName(customer: Customer): string; // (A)
function getFullName( // (B)
 map: Map<string, Customer>, id: string
): string;
function getFullName( // (C)
 customerOrMap: Customer | Map<string, Customer>,
 id?: string
): string {
 // ···
}

// @ts-expect-error: Argument of type 'Map<string, Customer>' is not
// assignable to parameter of type 'Customer'.
getFullName(idToCustomer); // missing ID

// @ts-expect-error: Argument of type '{ id: string; fullName: string; }'
// is not assignable to parameter of type 'Map<string, Customer>'.
// [...]
getFullName(lars, '5678'); // ID not allowed

```

这里发生了什么？`getFullName()` 的类型签名被重载了：

+   函数有两个外部*重载签名*：没有实现的类型签名（行 A 和行 B）。

+   它还有一个*实现签名*：实际实现的类型签名，它是内部的，并且必须与外部签名兼容。顺便提一下，它与前面的例子相同。

我的建议是只有在无法避免的情况下才使用重载。一个替代方案是将重载的函数拆分成多个具有不同名称的函数——例如：

+   `getFullName()`

+   `getFullNameViaMap()`

#### 27.5.2 通过元组类型的联合重载函数

我们也可以通过元组类型的联合来重载函数（行 A）：

```ts
interface Customer {
 id: string;
 fullName: string;
}
function getFullName(
 ...args: // (A)
 | [customer: Customer]
 | [map: Map<string, Customer>, id: string]
): string {
 if (args.length === 2) {
 // Type is narrowed:
 assertType<
 [map: Map<string, Customer>, id: string]
 >(args);
 const [map, id] = args;
 const customer = map.get(id);
 if (customer === undefined) {
 throw new Error('Unknown ID: ' + id);
 }
 return customer.fullName;
 } else {
 const [customer] = args;
 return customer.fullName;
 }
}

```

请记住，这种重载只有在所有情况下的返回类型都相同的情况下才有效。

注意元组类型元素的标签：

```ts
[ customer: Customer ]
[ map: Map<string, Customer>, id: string ]

```

这些标签是可选的，通常会被忽略，但可能会出现在自动完成和类型提示中。换句话说：它们更像注释。前两种类型等同于：

```ts
[ Customer ]
[ Map<string, Customer>, string ]

```

更多信息，请参阅“标记元组元素”（§37.1.3）。

#### 27.5.3 通过接口重载函数

在接口中，我们可以有多个不同的调用签名。这使得我们能够在以下例子中使用接口`GetFullName`进行重载：

```ts
interface Customer {
 id: string;
 fullName: string;
}

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

#### 27.5.4 基于字符串参数的重载（事件处理等）

在下一个例子中，我们重载并使用了字符串字面量类型（如`'click'`）。这允许我们根据`type`参数的值改变`listener`参数的类型：

```ts
function addEventListener(
 elem: HTMLElement, type: 'click',
 listener: (event: MouseEvent) => void
): void;
function addEventListener(
 elem: HTMLElement, type: 'keypress',
 listener: (event: KeyboardEvent) => void
): void;
function addEventListener( // (A)
 elem: HTMLElement, type: string,
 listener: (event: any) => void
): void {
 elem.addEventListener(type, listener); // (B)
}

```

在这种情况下，正确获取实现（从行 A 开始）的类型相对困难，以便使体中的语句（行 B）生效。作为最后的手段，我们在 `listener` 的参数 `event` 上使用了类型 `any`。

#### 27.5.5 重载方法

##### 27.5.5.1 重载具体方法

下一个示例演示了方法的重载：方法 `.add()` 被重载。

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
 sb.toString(), 'I can see 3 monkeys!'
)

```

##### 27.5.5.2 重载接口方法

`Array.from()` 的类型定义是重载接口方法的例子：

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

+   在第一个签名中，返回的 Array 的元素类型与参数相同。

+   在第二个签名中，返回的 Array 的元素类型与 `mapfn` 的结果相同。这个版本的 `Array.from()` 与 `Array.prototype.map()` 类似。

### 27.6 赋值（高级）

在本节中，我们探讨赋值规则的类型兼容性：类型 `Src` 的函数能否转移到类型 `Trg` 的存储位置（变量、对象属性、参数等）？

理解赋值规则有助于我们回答诸如以下问题：

+   给定一个形式参数的函数类型签名，哪些函数可以作为实际参数在函数调用中传递？

+   给定一个属性的函数类型签名，哪些函数可以分配给它？

#### 27.6.1 赋值规则

在本小节中，我们检查赋值规则的一般规则（包括函数的规则）。在下一个小节中，我们将探讨这些规则对函数的意义。

如果以下条件之一成立，则类型 `Src` 可以赋值给类型 `Trg`：

+   `Src` 和 `Trg` 是相同类型。

+   `Src` 或 `Trg` 是 `any` 类型。

+   `Src` 是一个字符串字面量类型，而 `Trg` 是原始类型 String。

+   `Src` 是一个联合类型，并且 `Src` 的每个组成部分类型都可以赋值给 `Trg`。

+   `Src` 和 `Trg` 是函数类型，并且：

    +   `Trg` 有剩余参数或 `Src` 所需的参数数量小于或等于 `Trg` 的总参数数量。

    +   对于在两个签名中都存在的参数，`Trg` 中的每个参数类型都可以赋值给 `Src` 中相应的参数类型。

    +   `Trg` 的返回类型是 `void` 或 `Src` 的返回类型可以赋值给 `Trg` 的返回类型。

+   （省略剩余条件。）

#### 27.6.2 函数赋值规则的影响

在本小节中，我们探讨赋值规则对以下两个函数 `targetFunc` 和 `sourceFunc` 的意义：

```ts
const targetFunc: Trg = sourceFunc;

```

##### 27.6.2.1 参数和结果类型

+   目标参数类型必须可以赋值给相应的源参数类型。

    +   为什么？目标接受的一切也必须被源接受。

+   源返回类型必须可以赋值给目标返回类型。

    +   为什么？源返回的任何内容都必须与目标设定的期望相兼容。

示例：

```ts
const trg1: (x: RegExp) => Object = (x: Object) => /abc/;

```

以下示例演示了如果目标返回类型是 `void`，则源返回类型无关紧要。为什么是这样？TypeScript 中总是忽略 `void` 类型的结果。

```ts
const trg2: () => void = () => new Date();

```

##### 27.6.2.2 参数数量

源不能比目标有更多的参数：

```ts
// @ts-expect-error: Type '(x: string) => string' is not assignable to
// type '() => string'.
const trg3: () => string = (x: string) => 'abc';

```

源可以比目标有更少的参数：

```ts
const trg4: (x: string) => string = () => 'abc';

```

为什么是这样？目标指定了源期望：它必须接受参数 `x`。它确实做到了（但它忽略了它）。这种宽容性使得：

```ts
['a', 'b'].map(x => x + x)

```

`.map()` 的回调函数只包含 `.map()` 类型签名中提到的三个参数之一：

```ts
map<U>(
 callback: (value: T, index: number, array: T[]) => U,
 thisArg?: any
): U[];

```

### 27.7 进一步阅读和本章来源

+   [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/basic-types.html)

+   [“可调用值”章节](https://exploringjs.com/js/book/ch_callables.html) 在 “Exploring JavaScript”
