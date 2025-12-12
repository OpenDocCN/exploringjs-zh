# 31 断言函数

> 原文：[`exploringjs.com/ts/book/ch_assertion-functions.html`](https://exploringjs.com/ts/book/ch_assertion-functions.html)

（广告，请勿拦截。）

1.  31.1 Node.js 中的断言函数

    1.  31.1.1 用于断言条件的`assert(cond)`

    1.  31.1.2 用于断言相等的`assert.equal(value1, value2)`

1.  31.2 TypeScript 对断言函数的支持

1.  31.3 断言布尔条件：`asserts «cond»`

    1.  31.3.1 断言类型守卫

1.  31.4 断言参数类型：`asserts «arg» is «type»`

    1.  31.4.0.1 示例断言函数：`assert.equal()`

1.  31.5 断言函数的替代方案

    1.  31.5.1 技巧：强制转换

    1.  31.5.2 技巧：抛出异常

1.  31.6 快速参考：断言函数

    1.  31.6.1 断言签名：`asserts «cond»`

    1.  31.6.2 断言签名：`asserts «arg» is «type»`

在编程中，断言函数是一个在程序中的特定点检查条件是否为真的函数，如果不为真则抛出异常。在本章中，我们将探讨：

+   断言函数在 JavaScript 中的使用方式。

+   TypeScript 在类型级别上如何支持它们。

### 31.1 Node.js 中的断言函数

Node.js 有一个完整的模块，`'node:assert/strict'`，其中包含断言函数。您可以在网上查看[其文档](https://nodejs.org/api/assert.html)。让我们看看两个重要的例子。

#### 31.1.1 用于断言条件的`assert(cond)`

函数`assert()`让我们可以断言条件。我认为它就像是在代码中写下的注释，因此是可验证的——例如：

```ts
import * as path from 'node:path';
import assert from 'node:assert/strict';
export function replaceExt(filePath: string, ext: string): string {
 assert(ext.startsWith('.')); // (A)
 const parsed = path.parse(filePath);
 return path.join(parsed.dir, parsed.name + ext);
}

```

`ext`必须以点开头可能是一个注释。相反，我们选择将其编写为断言（行 A）。如果条件为假，`assert()`会抛出异常。换句话说：`assert(cond)`意味着“`cond`必须为真”。

#### 31.1.2 用于断言相等的`assert.equal(value1, value2)`

断言函数`assert.equal()`主要用于测试：

```ts
import assert from 'node:assert/strict';
const add = (x: number, y: number) => x + y;
assert.equal(
 add(3, 4), 7
);

```

如果`assert.equal()`的两个参数不相等，它会抛出一个异常。

### 31.2 TypeScript 对断言函数的支持

TypeScript 为断言函数提供了特殊支持。我们可以指定两种类型的*断言签名*作为`void`函数的返回类型。

首先，我们可以断言一个布尔条件（想想`assert()`）：

```ts
function assertCondition(condition: boolean): asserts condition {
 // ...
}

```

其次，我们可以断言参数的类型：

```ts
function assertArgIsMyType(arg: unknown): asserts arg is MyType {
 // ...
}

```

接下来，我们将探讨调用断言函数如何影响类型级别。

### 31.3 断言布尔条件：`asserts «cond»`

这是一个 Node 的 `assert()` 函数的重新实现，这样我们也可以在浏览器中使用它。它断言一个布尔 `condition`：

```ts
function assertTrue(
 condition: boolean, message?: string | Error
): asserts condition { // (A)
 if (!condition) {
 if (message instanceof Error) {
 throw message;
 }
 message ??= 'Assertion failed';
 throw new Error(message); // (B)
 }
}

```

行 A 中的断言签名使 `assertTrue` 成为一个断言函数。我们将在下一小节中探讨其类型级别的后果。以下设计决策被采纳：

+   `condition` 的类型是 `boolean` 而不是 `unknown`，因为这迫使我们避免进行真值检查。

+   在行 B 中，我们也可以使用自定义的 `Error` 子类，例如名为 `AssertionError` 的一个。

#### 31.3.1 断言类型守卫

在条件中使用 *类型守卫*（探索值类型的布尔表达式）可以缩小类型：

```ts
if («typeGuard») {
 // One or more types are narrowed
}

```

断言类型守卫有类似的效果：

```ts
assertTrue(«typeGuard»);
// One or more types are narrowed

```

让我们看看一个例子：

```ts
function func(value: unknown) {
 assertTrue(value instanceof Set); // (A)
 assertType<Set<any>>(value);
}

```

`value instanceof Set` 是一个类型守卫，因为我们已在行 A 中断言了它，所以将 `value` 的类型缩小到 `Set<any>`。

### 31.4 断言参数类型：`asserts «arg» is «type»`

以下断言函数缩小其参数 `value` 的类型：

```ts
function assertIsNumber(value: unknown): asserts value is number {
 if (typeof value !== 'number') {
 throw new TypeError();
 }
}

```

让我们使用 `assertIsNumber()`：

```ts
function func(value: unknown) {
 assertIsNumber(value); // (A)
 assertType<number>(value);
}

```

行 A 中的断言将 `value` 的类型从 `unknown` 缩小到 `number`。

注意，`assertIsNumber()` 确实缩小了：

```ts
function func(str: string) {
 assertIsNumber(str);
 // string & number is `never`
 assertType<string & number>(str);
}

```

我们将 `str` 的原始类型 `string` 缩小到更窄的类型 `string & number` - 这与 底部类型 `never` 相同。

##### 31.4.0.1 示例断言函数：`assert.equal()`

这是一个简化版的 Node 的 `assert.equal()` 函数，具有相同的类型：

```ts
function assertEqual<T>(
 actual: unknown, expected: T, message?: string | Error
): asserts actual is T {
 if (actual !== expected) {
 if (message instanceof Error) {
 throw message;
 }
 message ??= `Assertion failed: ${actual} === ${expected}`;
 throw new Error(message);
 }
}

```

如果它终止，则 `actual` 的类型缩小到类型 `T`。这很少有用，但我发现这个函数这样做很有趣。

##### 31.4.0.2 示例断言函数：`assertNonNullable()`

以下断言函数断言其参数 `value` 不是可空的：

```ts
export function assertNonNullable<T>(
 value: T, message?: string
): asserts value is NonNullable<T> {
 if (value === undefined || value === null) {
 message ??= 'Value must not be undefined or null';
 throw new TypeError(message);
 }
}

```

我们使用 实用类型 `NonNullable<T>` 从 `T` 中移除 `undefined` 和 `null` - 例如：

```ts
function getValue(map: Map<string, number>, key: string): number {
 const value = map.get(key);
 assertType<undefined | number>(value);
 assertNonNullable(value);
 assertType<number>(value);
 return value;
}

```

##### 31.4.0.3 示例断言函数：向对象添加属性

以下断言函数向对象添加一个属性并相应地更新其类型：

```ts
function addName<T extends object>(obj: T, name: string)
: asserts obj is (T & HasName) { // (A)
 (obj as HasName).name = name;
}
type HasName = {
 name: string,
};

const point = { x: 0, y: 0 };
addName(point, 'zero');
assertType<{ x: number, y: number, name: string }>(point);

```

交集类型 `S & T`（如行 A 中的类型）具有类型 `S` 和类型 `T` 的属性。

### 31.5 断言函数的替代方案

#### 31.5.1 技巧：强制转换

断言函数缩小了现有值的类型。强制转换函数返回具有新类型的现有值 - 例如：

```ts
function forceNumber(value: unknown): number {
 if (typeof value !== 'number') {
 throw new TypeError();
 }
 return value;
}

const value1a: unknown = 123;
const value1b = forceNumber(value1a);
assertType<number>(value1b);

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
assertType<number>(value1);

const value2: unknown = 'abc';
assert.throws(() => assertIsNumber(value2));

```

强制转换是一种多才多艺的技术，其用途不仅限于断言函数。例如，我们可以转换：

+   从易于书写的输入格式（例如 JSON 架构）。

+   转换为易于在代码中处理的输出格式。

更多信息，请参阅“数据的外部表示与内部表示” (§32.3.4)。

#### 31.5.2 技巧：抛出异常

考虑以下代码：

```ts
function getLengthOfValue(strMap: Map<string, string>, key: string)
: number {
 if (strMap.has(key)) {
 const value = strMap.get(key);

 assertType<string | undefined>(value); // before type check
 // We know that value can’t be `undefined`
 if (value === undefined) { // (A)
 throw new Error();
 }
 assertType<string>(value); // after type check

 return value.length;
 }
 return -1;
}

```

除了从行 A 开始的`if`语句外，我们还可以使用断言函数：

```ts
assertNonNullable(value);

```

如果我们不希望编写这样的函数，抛出异常是一个快速的选择。类似于调用断言函数，这种技术也会更新静态类型。

### 31.6 快速参考：断言函数

#### 31.6.1 断言签名：`asserts «cond»`

```ts
function assertTrue(condition: boolean): asserts condition {
 if (!condition) {
 throw new Error(); // assertion error
 }
}

```

+   断言签名：`asserts condition`

+   结果：`void` 或异常

#### 31.6.2 断言签名：`asserts «arg» is «type»`

```ts
function assertIsString(value: unknown): asserts value is string {
 if (typeof value !== 'string') {
 throw new Error(); // assertion error
 }
}

```

+   断言签名：`asserts value is string`

+   结果：`void` 或异常
