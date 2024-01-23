# 二十二、[类型守卫和断言函数]

> 原文：[`exploringjs.com/tackling-ts/ch_type-guards-assertion-functions.html`](https://exploringjs.com/tackling-ts/ch_type-guards-assertion-functions.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   22.1 静态类型何时过于泛化？

    +   22.1.1 通过`if`和类型守卫缩小范围

    +   22.1.2 通过`switch`和类型守卫缩小范围

    +   22.1.3 类型过于泛化的更多情况

    +   22.1.4 类型`unknown`

+   22.2 通过内置类型守卫缩小范围

    +   22.2.1 严格相等（`===`）

    +   22.2.2 `typeof`、`instanceof`、`Array.isArray`

    +   22.2.3 通过`in`运算符检查不同的属性

    +   22.2.4 检查共享属性的值（辨别联合）

    +   22.2.5 缩小点名

    +   22.2.6 缩小数组元素类型

+   22.3 用户定义的类型守卫

    +   22.3.1 用户定义类型守卫的示例：`isArrayWithInstancesOf()`

    +   22.3.2 用户定义类型守卫的示例：`isTypeof()`

+   22.4 断言函数

    +   22.4.1 TypeScript 对断言函数的支持

    +   22.4.2 断言布尔类型的参数：`asserts «cond»`

    +   22.4.3 断言参数的类型：`asserts «arg» is «type»`

+   22.5 快速参考：用户定义的类型守卫和断言函数

    +   22.5.1 用户定义类型守卫

    +   22.5.2 断言函数

+   22.6 断言函数的替代方法

    +   22.6.1 技巧：强制转换

    +   22.6.2 技巧：抛出异常

+   22.7 `@hqoss/guards`：带有类型守卫的库

* * *

在 TypeScript 中，一个值可能对于某些操作来说类型过于泛化，例如，联合类型。本章回答以下问题：

+   类型的*缩小*是什么？

    +   剧透：*缩小*意味着将存储位置（例如变量或属性）的静态类型`T`更改为`T`的子集。例如，将类型`null|string`缩小为类型`string`通常很有用。

+   *类型守卫*和*断言函数*是什么，我们如何使用它们来缩小类型？

    +   剧透：`typeof`和`instanceof`是类型守卫。

### 22.1 当静态类型过于一般化时？

要看看静态类型如何过于一般化，请考虑以下函数 `getScore()`：

```ts
assert.equal(
 getScore('*****'), 5);
assert.equal(
 getScore(3), 3);
```

`getScore()` 的骨架如下所示：

```ts
function getScore(value: number|string): number {
 // ···
}
```

在 `getScore()` 的主体中，我们不知道 `value` 的类型是 `number` 还是 `string`。在我们知道之前，我们无法真正处理 `value`。

#### 22.1.1 通过 `if` 和类型守卫缩小

解决方案是通过 `typeof`（A 行和 B 行）在运行时检查 `value` 的类型：

```ts
function getScore(value: number|string): number {
 if (typeof value === 'number') { // (A)
 // %inferred-type: number
 value;
 return value;
 }
 if (typeof value === 'string') { // (B)
 // %inferred-type: string
 value;
 return value.length;
 }
 throw new Error('Unsupported value: ' + value);
}
```

在本章中，我们将类型解释为值的集合。（有关此解释和另一种解释的更多信息，请参见[[content not included]](ch_missing-chapters-online.html)。）

在从 A 行和 B 行开始的 then-blocks 中，由于我们执行的检查，`value` 的静态类型发生了变化。我们现在正在处理原始类型 `number|string` 的子集。这种减小类型大小的方式称为*缩小*。检查 `typeof` 的结果和类似的运行时操作称为*类型守卫*。

请注意，缩小不会改变 `value` 的原始类型，它只会在我们通过更多检查时变得更具体。

#### 22.1.2 通过 `switch` 和类型守卫缩小

如果我们使用 `switch` 而不是 `if`，缩小也会起作用：

```ts
function getScore(value: number|string): number {
 switch (typeof value) {
 case 'number':
 // %inferred-type: number
 value;
 return value;
 case 'string':
 // %inferred-type: string
 value;
 return value.length;
 default:
 throw new Error('Unsupported value: ' + value);
 }
}
```

#### 22.1.3 类型过于一般化的更多情况

这些是类型过于一般化的更多例子：

+   可空类型：

    ```ts
    function func1(arg: null|string) {}
    function func2(arg: undefined|string) {}
    ```

+   辨别联合：

    ```ts
    type Teacher = { kind: 'Teacher', teacherId: string };
    type Student = { kind: 'Student', studentId: string };
    type Attendee = Teacher | Student;

    function func3(attendee: Attendee) {}
    ```

+   可选参数的类型：

    ```ts
    function func4(arg?: string) {
     // %inferred-type: string | undefined
     arg;
    }
    ```

请注意，这些类型都是联合类型！

#### 22.1.4 类型 `unknown`

如果一个值具有类型 `unknown`，我们几乎无法对其进行任何操作，必须首先缩小其类型（A 行）：

```ts
function parseStringLiteral(stringLiteral: string): string {
 const result: unknown = JSON.parse(stringLiteral);
 if (typeof result === 'string') { // (A)
 return result;
 }
 throw new Error('Not a string literal: ' + stringLiteral);
}
```

换句话说：类型 `unknown` 太一般化了，我们必须缩小它。在某种程度上，`unknown` 也是一个联合类型（所有类型的联合）。

### 22.2 通过内置类型守卫缩小

正如我们所见，*类型守卫* 是一种操作，根据其运行时是否满足某些条件，返回 `true` 或 `false`。 TypeScript 的类型推断通过在结果为 `true` 时缩小操作数的静态类型来支持类型守卫。 

#### 22.2.1 严格相等 (`===`)

严格相等作为一种类型守卫：

```ts
function func(value: unknown) {
 if (value === 'abc') {
 // %inferred-type: "abc"
 value;
 }
}
```

对于一些联合类型，我们可以使用 `===` 来区分它们的组件：

```ts
interface Book {
 title: null | string;
 isbn: string;
}

function getTitle(book: Book) {
 if (book.title === null) {
 // %inferred-type: null
 book.title;
 return '(Untitled)';
 } else {
 // %inferred-type: string
 book.title;
 return book.title;
 }
}
```

使用 `===` 包括和 `!===` 排除联合类型组件只有在该组件是*单例类型*（一个成员的集合）时才有效。类型 `null` 是一个单例类型。它唯一的成员是值 `null`。

#### 22.2.2 `typeof`, `instanceof`, `Array.isArray`

这些是三种常见的内置类型守卫：

```ts
function func(value: Function|Date|number[]) {
 if (typeof value === 'function') {
 // %inferred-type: Function
 value;
 }

 if (value instanceof Date) {
 // %inferred-type: Date
 value;
 }

 if (Array.isArray(value)) {
 // %inferred-type: number[]
 value;
 }
}
```

注意在 then-blocks 中 `value` 的静态类型是如何缩小的。

#### 22.2.3 通过操作符 `in` 检查不同的属性

如果用于检查不同的属性，操作符 `in` 就是一种类型守卫：

```ts
type FirstOrSecond =
 | {first: string}
 | {second: string};

function func(firstOrSecond: FirstOrSecond) {
 if ('second' in firstOrSecond) {
 // %inferred-type: { second: string; }
 firstOrSecond;
 }
}
```

请注意以下检查将不起作用：

```ts
function func(firstOrSecond: FirstOrSecond) {
 // @ts-expect-error: Property 'second' does not exist on
 // type 'FirstOrSecond'. [...]
 if (firstOrSecond.second !== undefined) {
 // ···
 }
}
```

在这种情况下的问题是，如果不缩小，我们无法访问类型为 `FirstOrSecond` 的值的属性 `.second`。

##### 22.2.3.1 操作符 `in` 不会缩小非联合类型

遗憾的是，`in` 只能帮助我们处理联合类型：

```ts
function func(obj: object) {
 if ('name' in obj) {
 // %inferred-type: object
 obj;

 // @ts-expect-error: Property 'name' does not exist on type 'object'.
 obj.name;
 }
}
```

#### 22.2.4 检查共享属性的值（辨别联合）

在辨别联合中，联合类型的组件具有一个或多个共同的属性，其值对于每个组件都是不同的。这些属性称为*辨别者*。

检查辨别者的值是一种类型守卫：

```ts
type Teacher = { kind: 'Teacher', teacherId: string };
type Student = { kind: 'Student', studentId: string };
type Attendee = Teacher | Student;

function getId(attendee: Attendee) {
 switch (attendee.kind) {
 case 'Teacher':
 // %inferred-type: { kind: "Teacher"; teacherId: string; }
 attendee;
 return attendee.teacherId;
 case 'Student':
 // %inferred-type: { kind: "Student"; studentId: string; }
 attendee;
 return attendee.studentId;
 default:
 throw new Error();
 }
}
```

在前面的例子中，`.kind` 是一个辨别者：联合类型 `Attendee` 的每个组件都有这个属性，并且具有唯一的值。

`if` 语句和相等检查与 `switch` 语句类似：

```ts
function getId(attendee: Attendee) {
 if (attendee.kind === 'Teacher') {
 // %inferred-type: { kind: "Teacher"; teacherId: string; }
 attendee;
 return attendee.teacherId;
 } else if (attendee.kind === 'Student') {
 // %inferred-type: { kind: "Student"; studentId: string; }
 attendee;
 return attendee.studentId;
 } else {
 throw new Error();
 }
}
```

#### 22.2.5 缩小点名

我们还可以缩小属性的类型（甚至是通过属性名称链访问的嵌套属性的类型）：

```ts
type MyType = {
 prop?: number | string,
};
function func(arg: MyType) {
 if (typeof arg.prop === 'string') {
 // %inferred-type: string
 arg.prop; // (A)

 [].forEach((x) => {
 // %inferred-type: string | number | undefined
 arg.prop; // (B)
 });

 // %inferred-type: string
 arg.prop;

 arg = {};

 // %inferred-type: string | number | undefined
 arg.prop; // (C)
 }
}
```

让我们看看前面代码中的几个位置：

+   A 行：我们通过类型守卫缩小了 `arg.prop` 的类型。

+   B 行：回调可能会在很久以后执行（考虑异步代码），这就是为什么 TypeScript 在回调内部取消缩小。

+   C 行：前面的赋值也取消了缩小。

#### 22.2.6 缩小数组元素类型

##### 22.2.6.1 数组方法`.every()`不会缩小

如果我们使用`.every()`来检查所有数组元素是否非空，TypeScript 不会缩小`mixedValues`的类型（A 行）：

```ts
const mixedValues: ReadonlyArray<undefined|null|number> =
 [1, undefined, 2, null];

if (mixedValues.every(isNotNullish)) {
 // %inferred-type: readonly (number | null | undefined)[]
 mixedValues; // (A)
}
```

请注意`mixedValues`必须是只读的。如果不是，那么在`if`语句中，对它的另一个引用将静态地允许我们将`null`推入`mixedValues`中。但这会使`mixedValues`的缩小类型不正确。

前面的代码使用了以下*用户定义的类型守卫*（稍后会详细介绍）：

```ts
function isNotNullish<T>(value: T): value is NonNullable<T> { // (A)
 return value !== undefined && value !== null;
}
```

`NonNullable<Union>`（A 行）是[一个实用类型](https://www.typescriptlang.org/docs/handbook/utility-types.html)，它从联合类型`Union`中移除了`undefined`和`null`类型。

##### 22.2.6.2 数组方法`.filter()`产生具有更窄类型的数组

`.filter()`产生具有更窄类型的数组（即，它实际上并没有缩小现有类型）：

```ts
// %inferred-type: (number | null | undefined)[]
const mixedValues = [1, undefined, 2, null];

// %inferred-type: number[]
const numbers = mixedValues.filter(isNotNullish);

function isNotNullish<T>(value: T): value is NonNullable<T> { // (A)
 return value !== undefined && value !== null;
}
```

遗憾的是，我们必须直接使用类型守卫函数-箭头函数与类型守卫是不够的：

```ts
// %inferred-type: (number | null | undefined)[]
const stillMixed1 = mixedValues.filter(
 x => x !== undefined && x !== null);

// %inferred-type: (number | null | undefined)[]
const stillMixed2 = mixedValues.filter(
 x => typeof x === 'number');
```

### 22.3 用户定义类型守卫

TypeScript 允许我们定义自己的类型守卫-例如：

```ts
function isFunction(value: unknown): value is Function {
 return typeof value === 'function';
}
```

返回类型`value is Function`是*类型断言*的一部分。它是`isFunction()`的类型签名的一部分：

```ts
// %inferred-type: (value: unknown) => value is Function
isFunction;
```

用户定义的类型守卫必须始终返回布尔值。如果`isFunction(x)`返回`true`，TypeScript 会将实际参数`x`的类型缩小为`Function`：

```ts
function func(arg: unknown) {
 if (isFunction(arg)) {
 // %inferred-type: Function
 arg; // type is narrowed
 }
}
```

请注意，TypeScript 不关心我们如何计算用户定义类型守卫的结果。这给了我们很大的自由度，关于我们使用的检查。例如，我们可以将`isFunction()`实现为以下方式：

```ts
function isFunction(value: any): value is Function {
 try {
 value(); // (A)
 return true;
 } catch {
 return false;
 }
}
```

遗憾的是，我们必须对参数`value`使用类型`any`，因为类型`unknown`不允许我们在 A 行进行函数调用。

#### 22.3.1 用户定义类型守卫的示例：`isArrayWithInstancesOf()`

```ts
/**
 * This type guard for Arrays works similarly to `Array.isArray()`,
 * but also checks if all Array elements are instances of `T`.
 * As a consequence, the type of `arr` is narrowed to `Array<T>`
 * if this function returns `true`.
 * 
 * Warning: This type guard can make code unsafe – for example:
 * We could use another reference to `arr` to add an element whose
 * type is not `T`. Then `arr` doesn’t have the type `Array<T>`
 * anymore.
 */
function isArrayWithInstancesOf<T>(
 arr: any, Class: new (...args: any[])=>T)
 : arr is Array<T>
{
 if (!Array.isArray(arr)) {
 return false;
 }
 if (!arr.every(elem => elem instanceof Class)) {
 return false;
 }

 // %inferred-type: any[]
 arr; // (A)

 return true;
}
```

在 A 行，我们可以看到`arr`的推断类型*不是*`Array<T>`，但我们的检查确保它目前是。这就是为什么我们可以返回`true`。TypeScript 相信我们，并在我们使用`isArrayWithInstancesOf()`时将其缩小为`Array<T>`：

```ts
const value: unknown = {};
if (isArrayWithInstancesOf(value, RegExp)) {
 // %inferred-type: RegExp[]
 value;
}
```

#### 22.3.2 用户定义类型守卫的示例：`isTypeof()`

##### 22.3.2.1 第一次尝试

这是在 TypeScript 中实现`typeof`的第一次尝试：

```ts
/**
 * An implementation of the `typeof` operator.
 */
function isTypeof<T>(value: unknown, prim: T): value is T {
 if (prim === null) {
 return value === null;
 }
 return value !== null && (typeof prim) === (typeof value);
}
```

理想情况下，我们可以通过字符串指定`value`的预期类型（即`typeof`的结果之一）。但是，然后我们将不得不从该字符串中派生类型`T`，并且如何做到这一点并不是立即明显的（正如我们很快将看到的那样）。作为一种解决方法，我们通过`T`的成员`prim`来指定`T`：

```ts
const value: unknown = {};
if (isTypeof(value, 123)) {
 // %inferred-type: number
 value;
}
```

##### 22.3.2.2 使用重载

更好的解决方案是使用重载（有几种情况被省略）：

```ts
/**
 * A partial implementation of the `typeof` operator.
 */
function isTypeof(value: any, typeString: 'boolean'): value is boolean;
function isTypeof(value: any, typeString: 'number'): value is number;
function isTypeof(value: any, typeString: 'string'): value is string;
function isTypeof(value: any, typeString: string): boolean {
 return typeof value === typeString;
}

const value: unknown = {};
if (isTypeof(value, 'boolean')) {
 // %inferred-type: boolean
 value;
}
```

（这个方法是由[Nick Fisher](https://twitter.com/spadgos/status/1266839605883666432)提出的。）

##### 22.3.2.3 使用接口作为类型映射

另一种方法是使用接口作为从字符串到类型的映射（有几种情况被省略）：

```ts
interface TypeMap {
 boolean: boolean;
 number: number;
 string: string;
}

/**
 * A partial implementation of the `typeof` operator.
 */
function isTypeof<T extends keyof TypeMap>(value: any, typeString: T)
: value is TypeMap[T] {
 return typeof value === typeString;
}

const value: unknown = {};
if (isTypeof(value, 'string')) {
 // %inferred-type: string
 value;
}
```

（这个方法是由[Ran Lottem](https://dev.to/krumpet/generic-type-guard-in-typescript-258l)提出的。）

### 22.4 断言函数

断言函数检查其参数是否满足某些条件，如果不满足则抛出异常。例如，许多语言支持的一个断言函数是`assert()`。`assert(cond)`如果布尔条件`cond`为`false`，则抛出异常。

在 Node.js 上，`assert()`通过[内置模块`assert`](https://nodejs.org/api/assert.html)支持。以下代码在 A 行中使用了它：

```ts
import assert from 'assert';
function removeFilenameExtension(filename: string) {
 const dotIndex = filename.lastIndexOf('.');
 assert(dotIndex >= 0); // (A)
 return filename.slice(0, dotIndex);
}
```

#### 22.4.1 TypeScript 对断言函数的支持

如果我们使用*断言签名*作为返回类型标记这样的函数，TypeScript 的类型推断将特别支持断言函数。就函数的返回方式和返回内容而言，断言签名等同于`void`。但它还会触发缩小。

有两种断言签名：

+   断言布尔参数：`asserts «cond»`

+   断言参数的类型：`asserts «arg» is «type»`

#### 22.4.2 断言布尔参数：`asserts «cond»`

在下面的例子中，断言签名`asserts condition`表示参数`condition`必须为`true`。否则，将抛出异常。

```ts
function assertTrue(condition: boolean): asserts condition {
 if (!condition) {
 throw new Error();
 }
}
```

这就是`assertTrue()`导致缩小的方式：

```ts
function func(value: unknown) {
 assertTrue(value instanceof Set);

 // %inferred-type: Set<any>
 value;
}
```

我们使用参数`value instanceof Set`类似于类型守卫，但是`false`触发异常，而不是跳过条件语句的一部分。

#### 22.4.3 断言参数的类型：`asserts «arg» is «type»`

在下面的例子中，断言签名`asserts value is number`表示参数`value`必须具有类型`number`。否则，将抛出异常。

```ts
function assertIsNumber(value: any): asserts value is number {
 if (typeof value !== 'number') {
 throw new TypeError();
 }
}
```

这次，调用断言函数会缩小其参数的类型：

```ts
function func(value: unknown) {
 assertIsNumber(value);

 // %inferred-type: number
 value;
}
```

##### 22.4.3.1 示例断言函数：向对象添加属性

函数`addXY()`会向现有对象添加属性，并相应地更新它们的类型：

```ts
function addXY<T>(obj: T, x: number, y: number)
: asserts obj is (T & { x: number, y: number }) {
 // Adding properties via = would be more complicated...
 Object.assign(obj, {x, y});
}

const obj = { color: 'green' };
addXY(obj, 9, 4);

// %inferred-type: { color: string; } & { x: number; y: number; }
obj;
```

交集类型`S & T`具有类型`S`和类型`T`的属性。

### 22.5 快速参考：用户定义的类型守卫和断言函数

#### 22.5.1 用户定义的类型守卫

```ts
function isString(value: unknown): value is string {
 return typeof value === 'string';
}
```

+   类型谓词：`value is string`

+   结果：`boolean`

#### 22.5.2 断言函数

##### 22.5.2.1 断言签名：`asserts «cond»`

```ts
function assertTrue(condition: boolean): asserts condition {
 if (!condition) {
 throw new Error(); // assertion error
 }
}
```

+   断言签名：`asserts condition`

+   结果：`void`，异常

##### 22.5.2.2 断言签名：`asserts «arg» is «type»`

```ts
function assertIsString(value: unknown): asserts value is string {
 if (typeof value !== 'string') {
 throw new Error(); // assertion error
 }
}
```

+   断言签名：`asserts value is string`

+   结果：`void`，异常

### 22.6 断言函数的替代方法

#### 22.6.1 技术：强制转换

断言函数会缩小现有值的类型。强制转换函数会返回具有新类型的现有值 - 例如：

```ts
function forceNumber(value: unknown): number {
 if (typeof value !== 'number') {
 throw new TypeError();
 }
 return value;
}

const value1a: unknown = 123;
// %inferred-type: number
const value1b = forceNumber(value1a);

const value2: unknown = 'abc';
assert.throws(() => forceNumber(value2));
```

相应的断言函数如下所示：

```ts
function assertIsNumber(value: unknown): asserts value is number {
 if (typeof value !== 'number') {
 throw new TypeError();
 }
}

const value1: unknown = 123;
assertIsNumber(value1);
// %inferred-type: number
value1;

const value2: unknown = 'abc';
assert.throws(() => assertIsNumber(value2));
```

强制转换是一种多用途的技术，除了断言函数的用途之外还有其他用途。例如，我们可以转换：

+   从易于编写的输入格式（比如 JSON 模式）

+   转换为易于在代码中使用的输出格式。

有关更多信息，请参见[[content not included]](ch_missing-chapters-online.html)。

#### 22.6.2 技术：抛出异常

考虑以下代码：

```ts
function getLengthOfValue(strMap: Map<string, string>, key: string)
: number {
 if (strMap.has(key)) {
 const value = strMap.get(key);

 // %inferred-type: string | undefined
 value; // before type check

 // We know that value can’t be `undefined`
 if (value === undefined) { // (A)
 throw new Error();
 }

 // %inferred-type: string
 value; // after type check

 return value.length;
 }
 return -1;
}
```

我们也可以使用断言函数来代替从 A 行开始的`if`语句：

```ts
assertNotUndefined(value);
```

如果我们不想编写这样的函数，抛出异常是一个快速的替代方法。与调用断言函数类似，这种技术也会更新静态类型。

### 22.7 `@hqoss/guards`：带有类型守卫的库

[库`@hqoss/guards`](https://github.com/hqoss/guards)提供了一组用于 TypeScript 的类型守卫 - 例如：

+   基本类型：`isBoolean()`，`isNumber()`，等等。

+   特定类型：`isObject()`，`isNull()`，`isFunction()`，等等。

+   各种检查：`isNonEmptyArray()`，`isInteger()`，等等。

[评论](https://github.com/rauschma/tackling-ts/issues/23)
