# 十二、TypeScript 枚举：它们是如何工作的？可以用于什么？

> 原文：[`exploringjs.com/tackling-ts/ch_enums.html`](https://exploringjs.com/tackling-ts/ch_enums.html)

* * *

+   12.1 基础知识

    +   12.1.1 数字枚举

    +   12.1.2 基于字符串的枚举

    +   12.1.3 异构枚举

    +   12.1.4 省略初始化器

    +   12.1.5 枚举成员名称的大小写

    +   12.1.6 枚举成员名称加引号

+   12.2 指定枚举成员的值（高级）

    +   12.2.1 字面量枚举成员

    +   12.2.2 常量枚举成员

    +   12.2.3 计算的枚举成员

+   12.3 数字枚举的缺点

    +   12.3.1 缺点：日志记录

    +   12.3.2 缺点：松散的类型检查

    +   12.3.3 建议：优先使用基于字符串的枚举

+   12.4 枚举的用例

    +   12.4.1 用例：位模式

    +   12.4.2 用例：多个常量

    +   12.4.3 用例：比布尔值更自描述

    +   12.4.4 用例：更好的字符串常量

+   12.5 运行时的枚举

    +   12.5.1 反向映射

    +   12.5.2 运行时的基于字符串的枚举

+   12.6 `const`枚举

    +   12.6.1 编译非常量枚举

    +   12.6.2 编译常量枚举

+   12.7 编译时的枚举

    +   12.7.1 枚举是对象

    +   12.7.2 字面量枚举的安全检查

    +   12.7.3 `keyof`和枚举

+   12.8 致谢

* * *

本章回答以下两个问题：

+   TypeScript 的枚举是如何工作的？

+   它们可以用于什么？

在下一章中，我们将看看枚举的替代方案。

### 12.1 基础知识

`boolean`是一个具有有限值的类型：`false`和`true`。使用枚举，TypeScript 允许我们自己定义类似的类型。

#### 12.1.1 数字枚举

这是一个数字枚举：

```ts
enum NoYes {
 No = 0,
 Yes = 1, // trailing comma
}

assert.equal(NoYes.No, 0);
assert.equal(NoYes.Yes, 1);
```

解释：

+   `No`和`Yes`被称为枚举`NoYes`的*成员*。

+   每个枚举成员都有一个*名称*和一个*值*。例如，第一个成员的名称是`No`，值是`0`。

+   以等号开头并指定值的成员定义部分称为*初始化器*。

+   与对象文字一样，允许并忽略尾随逗号。

我们可以将成员用作字面量，例如`true`，`123`或`'abc'`。例如：

```ts
function toGerman(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 }
}
assert.equal(toGerman(NoYes.No), 'Nein');
assert.equal(toGerman(NoYes.Yes), 'Ja');
```

#### 12.1.2 基于字符串的枚举

我们也可以使用字符串作为枚举成员的值：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}

assert.equal(NoYes.No, 'No');
assert.equal(NoYes.Yes, 'Yes');
```

#### 12.1.3 异构枚举

最后一种枚举称为*异构*。异构枚举的成员值是数字和字符串的混合：

```ts
enum Enum {
 One = 'One',
 Two = 'Two',
 Three = 3,
 Four = 4,
}
assert.deepEqual(
 [Enum.One, Enum.Two, Enum.Three, Enum.Four],
 ['One', 'Two', 3, 4]
);
```

异构枚举很少使用，因为它们的应用很少。

遗憾的是，TypeScript 只支持数字和字符串作为枚举成员的值。不允许其他值，比如符号。

#### 12.1.4 省略初始化器

我们可以在两种情况下省略初始化器：

+   我们可以省略第一个成员的初始化器。然后该成员的值为 0（零）。

+   如果前一个成员有数字值，则可以省略成员的初始化器。然后当前成员的值为前一个成员的值加一。

这是一个没有任何初始化程序的数字枚举：

```ts
enum NoYes {
 No,
 Yes,
}
assert.equal(NoYes.No, 0);
assert.equal(NoYes.Yes, 1);
```

这是一个异构枚举，其中省略了一些初始化程序：

```ts
enum Enum {
 A,
 B,
 C = 'C',
 D = 'D',
 E = 8, // (A)
 F,
}
assert.deepEqual(
 [Enum.A, Enum.B, Enum.C, Enum.D, Enum.E, Enum.F],
 [0, 1, 'C', 'D', 8, 9]
);
```

请注意，我们不能省略行 A 中的初始化程序，因为前一个成员的值不是数字。

#### 12.1.5 枚举成员名称的大小写

有几个命名常量的先例（在枚举或其他地方）：

+   传统上，JavaScript 使用全大写的名称，这是它从 Java 和 C 继承的约定：

    +   `Number.MAX_VALUE`

    +   `Math.SQRT2`

+   众所周知的符号是以小写字母开头的驼峰命名，因为它们与属性名称相关：

    +   `Symbol.asyncIterator`

+   TypeScript 手册使用以大写字母开头的驼峰命名。这是标准的 TypeScript 风格，我们在 `NoYes` 枚举中使用了它。

#### 12.1.6 引用枚举成员名称

与 JavaScript 对象类似，我们可以引用枚举成员的名称：

```ts
enum HttpRequestField {
 'Accept',
 'Accept-Charset',
 'Accept-Datetime',
 'Accept-Encoding',
 'Accept-Language',
}
assert.equal(HttpRequestField['Accept-Charset'], 1);
```

没有办法计算枚举成员的名称。对象文字支持通过方括号进行计算属性键。

### 12.2 指定枚举成员值（高级）

TypeScript 通过初始化方式区分三种枚举成员：

+   *文字枚举成员*：

    +   要么没有初始化程序

    +   或者通过数字文字或字符串文字初始化。

+   *常量枚举成员* 是通过在编译时可以计算结果的表达式初始化的。

+   *计算的枚举成员* 是通过任意表达式初始化的。

到目前为止，我们只使用了文字成员。

在前面的列表中，提到的成员不太灵活，但支持更多功能。继续阅读以获取更多信息。

#### 12.2.1 文字枚举成员

如果枚举成员的值已指定，则该枚举成员是*文字*：

+   隐式地

+   或通过数字文字（包括否定的数字文字）

+   或通过字符串文字。

如果枚举只有文字成员，我们可以将这些成员用作类型（类似于如何使用数字文字作为类型）：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}
function func(x: NoYes.No) { // (A)
 return x;
}

func(NoYes.No); // OK

// @ts-expect-error: Argument of type '"No"' is not assignable to
// parameter of type 'NoYes.No'.
func('No');

// @ts-expect-error: Argument of type 'NoYes.Yes' is not assignable to
// parameter of type 'NoYes.No'.
func(NoYes.Yes);
```

行 A 中的 `NoYes.No` 是*枚举成员类型*。

此外，文字枚举支持完整性检查（稍后我们将详细介绍）。

#### 12.2.2 常量枚举成员

如果枚举成员的值可以在编译时计算，则该枚举成员是常量。因此，我们可以隐式地指定其值（也就是说，我们让 TypeScript 为我们指定它）。或者我们可以明确指定它，并且只允许使用以下语法：

+   数字文字或字符串文字

+   对先前定义的常量枚举成员的引用（在当前枚举或以前的枚举中）

+   括号

+   一元运算符 `+`、`-`、`~`

+   二进制运算符 `+`、`-`、`*`、`/`、`%`、`<<`、`>>`、`>>>`、`&`、`|`、`^`

这是一个枚举的例子，其成员都是常量（稍后我们将看到该枚举的用法）：

```ts
enum Perm {
 UserRead     = 1 << 8, // bit 8
 UserWrite    = 1 << 7,
 UserExecute  = 1 << 6,
 GroupRead    = 1 << 5,
 GroupWrite   = 1 << 4,
 GroupExecute = 1 << 3,
 AllRead      = 1 << 2,
 AllWrite     = 1 << 1,
 AllExecute   = 1 << 0,
}
```

通常，常量成员不能用作类型。但是，仍然执行完整性检查。

#### 12.2.3 计算的枚举成员

*计算的枚举成员* 的值可以通过任意表达式指定。例如：

```ts
enum NoYesNum {
 No = 123,
 Yes = Math.random(), // OK
}
```

这是一个数字枚举。基于字符串的枚举和异构枚举更受限制。例如，我们不能使用方法调用来指定成员值：

```ts
enum NoYesStr {
 No = 'No',
 // @ts-expect-error: Computed values are not permitted in
 // an enum with string valued members.
 Yes = ['Y', 'e', 's'].join(''),
}
```

TypeScript 不会对计算的枚举成员执行完整性检查。

### 12.3 数字枚举的缺点

#### 12.3.1 缺点：记录

在记录数字枚举的成员时，我们只看到数字：

```ts
enum NoYes { No, Yes }

console.log(NoYes.No);
console.log(NoYes.Yes);

// Output:
// 0
// 1
```

#### 12.3.2 缺点：松散的类型检查

在将枚举用作类型时，静态允许的值不仅仅是枚举成员的值-任何数字都被接受：

```ts
enum NoYes { No, Yes }
function func(noYes: NoYes) {}
func(33); // no error!
```

为什么没有更严格的静态检查？[Daniel Rosenwasser 解释](https://github.com/microsoft/TypeScript/issues/26362#issuecomment-412198938)：

> 这种行为是由位操作驱动的。有时候 `SomeFlag.Foo | SomeFlag.Bar` 旨在产生另一个 `SomeFlag`。而不是得到 `number`，你不想要强制转换回 `SomeFlag`。
> 
> 我认为如果我们重新使用 TypeScript 并且仍然有枚举，我们会为位标志制定一个单独的构造。

很快我们将更详细地演示枚举如何用于位模式。

#### 12.3.3 建议：更喜欢基于字符串的枚举

我的建议是更喜欢基于字符串的枚举（为了简洁起见，本章并不总是遵循此建议）：

```ts
enum NoYes { No='No', Yes='Yes' }
```

一方面，日志输出对人类更有用：

```ts
console.log(NoYes.No);
console.log(NoYes.Yes);

// Output:
// 'No'
// 'Yes'
```

另一方面，我们获得了更严格的类型检查：

```ts
function func(noYes: NoYes) {}

// @ts-expect-error: Argument of type '"abc"' is not assignable
// to parameter of type 'NoYes'.
func('abc');

// @ts-expect-error: Argument of type '"Yes"' is not assignable
// to parameter of type 'NoYes'.
func('Yes'); // (A)
```

甚至不允许等于成员值的字符串（行 A）。

### 12.4 枚举的用例

#### 12.4.1 用例：位模式

在[Node.js 文件系统模块](https://nodejs.org/api/fs.html)中，有几个函数具有参数`mode`。它通过数字编码指定文件权限，这是 Unix 的遗留物：

+   权限指定了三类用户的权限：

    +   用户：文件的所有者

    +   组：与文件关联的组的成员

    +   所有人：所有人

+   按类别，可以授予以下权限：

    +   r（读取）：允许类别中的用户读取文件

    +   w（写入）：允许类别中的用户更改文件

    +   x（执行）：允许类别中的用户运行文件

这意味着权限可以由 9 位表示（每个类别有 3 个权限）：

|  | 用户 | 组 | 所有人 |
| --- | --- | --- | --- |
| 权限 | r，w，x | r，w，x | r，w，x |
| 位 | 8, 7, 6 | 5, 4, 3 | 2, 1, 0 |

Node.js 不这样做，但我们可以使用枚举来处理这些标志：

```ts
enum Perm {
 UserRead     = 1 << 8, // bit 8
 UserWrite    = 1 << 7,
 UserExecute  = 1 << 6,
 GroupRead    = 1 << 5,
 GroupWrite   = 1 << 4,
 GroupExecute = 1 << 3,
 AllRead      = 1 << 2,
 AllWrite     = 1 << 1,
 AllExecute   = 1 << 0,
}
```

位模式通过[按位或](https://exploringjs.com/impatient-js/ch_numbers.html#binary-bitwise-operators)进行组合：

```ts
// User can change, read and execute.
// Everyone else can only read and execute.
assert.equal(
 Perm.UserRead | Perm.UserWrite | Perm.UserExecute |
 Perm.GroupRead | Perm.GroupExecute |
 Perm.AllRead | Perm.AllExecute,
 0o755);

// User can read and write.
// Group members can read.
// Everyone can’t access at all.
assert.equal(
 Perm.UserRead | Perm.UserWrite | Perm.GroupRead,
 0o640);
```

##### 12.4.1.1 位模式的替代方案

位模式的主要思想是有一组标志，可以选择这些标志的任何子集。

因此，使用真实集合来选择子集是执行相同任务的更直接的方式：

```ts
enum Perm {
 UserRead = 'UserRead',
 UserWrite = 'UserWrite',
 UserExecute = 'UserExecute',
 GroupRead = 'GroupRead',
 GroupWrite = 'GroupWrite',
 GroupExecute = 'GroupExecute',
 AllRead = 'AllRead',
 AllWrite = 'AllWrite',
 AllExecute = 'AllExecute',
}
function writeFileSync(
 thePath: string, permissions: Set<Perm>, content: string) {
 // ···
}
writeFileSync(
 '/tmp/hello.txt',
 new Set([Perm.UserRead, Perm.UserWrite, Perm.GroupRead]),
 'Hello!');
```

#### 12.4.2 用例：多个常量

有时，我们有一组属于一起的常量：

```ts
const off = Symbol('off');
const info = Symbol('info');
const warn = Symbol('warn');
const error = Symbol('error');
```

这是枚举的一个很好的用例：

```ts
enum LogLevel {
 off = 'off',
 info = 'info',
 warn = 'warn',
 error = 'error',
}
```

枚举的一个好处是常量名称被分组并嵌套在命名空间`LogLevel`中。

另一个是我们自动获得了类型`LogLevel`。如果我们想要这样的类型用于常量，我们需要更多的工作：

```ts
type LogLevel =
 | typeof off
 | typeof info
 | typeof warn
 | typeof error
;
```

有关此方法的更多信息，请参见§13.1.3“符号单例类型的联合”。

#### 12.4.3 用例：比布尔值更具自描述性

当布尔值用于表示替代方案时，枚举通常更具自描述性。

##### 12.4.3.1 布尔示例：有序 vs. 无序列表

例如，要表示列表是否有序，我们可以使用布尔值：

```ts
class List1 {
 isOrdered: boolean;
 // ···
}
```

然而，枚举更具自描述性，并且具有额外的好处，即如果需要，我们可以随后添加更多的替代方案。

```ts
enum ListKind { ordered, unordered }
class List2 {
 listKind: ListKind;
 // ···
}
```

##### 12.4.3.2 布尔示例：错误处理模式

同样，我们可以通过布尔值指定如何处理错误：

```ts
function convertToHtml1(markdown: string, throwOnError: boolean) {
 // ···
}
```

或者我们可以通过枚举值来实现：

```ts
enum ErrorHandling {
 throwOnError = 'throwOnError',
 showErrorsInContent = 'showErrorsInContent',
}
function convertToHtml2(markdown: string, errorHandling: ErrorHandling) {
 // ···
}
```

#### 12.4.4 用例：更好的字符串常量

考虑以下创建正则表达式的函数。

```ts
const GLOBAL = 'g';
const NOT_GLOBAL = '';
type Globalness = typeof GLOBAL | typeof NOT_GLOBAL;

function createRegExp(source: string,
 globalness: Globalness = NOT_GLOBAL) {
 return new RegExp(source, 'u' + globalness);
 }

assert.deepEqual(
 createRegExp('abc', GLOBAL),
 /abc/ug);

assert.deepEqual(
 createRegExp('abc', 'g'), // OK
 /abc/ug);
```

我们可以使用枚举而不是字符串常量：

```ts
enum Globalness {
 Global = 'g',
 notGlobal = '',
}

function createRegExp(source: string, globalness = Globalness.notGlobal) {
 return new RegExp(source, 'u' + globalness);
}

assert.deepEqual(
 createRegExp('abc', Globalness.Global),
 /abc/ug);

assert.deepEqual(
 // @ts-expect-error: Argument of type '"g"' is not assignable to parameter of type 'Globalness | undefined'. (2345)
 createRegExp('abc', 'g'), // error
 /abc/ug);
```

这种方法的好处是什么？

+   它更简洁。

+   它稍微更安全：类型`Globalness`只接受成员名称，而不是字符串。

### 12.5 运行时的枚举

TypeScript 将枚举编译为 JavaScript 对象。例如，考虑以下枚举：

```ts
enum NoYes {
 No,
 Yes,
}
```

TypeScript 将此枚举编译为：

```ts
var NoYes;
(function (NoYes) {
 NoYes[NoYes["No"] = 0] = "No";
 NoYes[NoYes["Yes"] = 1] = "Yes";
})(NoYes || (NoYes = {}));
```

在此代码中，进行了以下赋值：

```ts
NoYes["No"] = 0;
NoYes["Yes"] = 1;

NoYes[0] = "No";
NoYes[1] = "Yes";
```

有两组赋值：

+   前两个赋值将枚举成员名称映射到值。

+   接下来的两个赋值将值映射到名称。这使得*反向映射*成为可能，接下来我们将看一下。

#### 12.5.1 反向映射

给定一个数字枚举：

```ts
enum NoYes {
 No,
 Yes,
}
```

正常映射是从成员名称到成员值：

```ts
// Static (= fixed) lookup:
assert.equal(NoYes.Yes, 1);

// Dynamic lookup:
assert.equal(NoYes['Yes'], 1);
```

数字枚举还支持从成员值到成员名称的*反向映射*：

```ts
assert.equal(NoYes[1], 'Yes');
```

反向映射的一个用例是打印枚举成员的名称：

```ts
function getQualifiedName(value: NoYes) {
 return 'NoYes.' + NoYes[value];
}
assert.equal(
 getQualifiedName(NoYes.Yes), 'NoYes.Yes');
```

#### 12.5.2 运行时的基于字符串的枚举

基于字符串的枚举在运行时具有更简单的表示。

考虑以下枚举。

```ts
enum NoYes {
 No = 'NO!',
 Yes = 'YES!',
}
```

它编译为以下 JavaScript 代码：

```ts
var NoYes;
(function (NoYes) {
 NoYes["No"] = "NO!";
 NoYes["Yes"] = "YES!";
})(NoYes || (NoYes = {}));
```

TypeScript 不支持基于字符串的枚举的反向映射。

### 12.6 `const`枚举

如果枚举以关键字`const`为前缀，则在运行时没有表示。相反，直接使用其成员的值。

#### 12.6.1 编译非 const 枚举

要观察这种效果，让我们首先检查以下非 const 枚举：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}

function toGerman(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 }
}
```

TypeScript 将此代码编译为：

```ts
"use strict";
var NoYes;
(function (NoYes) {
 NoYes["No"] = "No";
 NoYes["Yes"] = "Yes";
})(NoYes || (NoYes = {}));

function toGerman(value) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 }
}
```

#### 12.6.2 编译 const 枚举

这与以前的代码相同，但现在枚举是 const：

```ts
const enum NoYes {
 No,
 Yes,
}
function toGerman(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 }
}
```

现在，枚举的表示作为构造体消失了，只剩下其成员的值：

```ts
function toGerman(value) {
 switch (value) {
 case "No" /* No */:
 return 'Nein';
 case "Yes" /* Yes */:
 return 'Ja';
 }
}
```

### 12.7 编译时的枚举

#### 12.7.1 枚举是对象

TypeScript 将（非 const）枚举视为对象：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}
function func(obj: { No: string }) {
 return obj.No;
}
assert.equal(
 func(NoYes), // allowed statically!
 'No');
```

#### 12.7.2 对文字枚举的安全检查

当我们接受枚举成员的值时，通常希望确保：

+   我们不会收到非法值。

+   我们不会忘记考虑任何枚举成员的值。如果我们稍后添加成员，这一点尤其重要。

继续阅读以获取更多信息。我们将使用以下枚举进行工作：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}
```

##### 12.7.2.1 防止非法值

在以下代码中，我们采取了两项措施防止非法值：

```ts
function toGerman1(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 default:
 throw new TypeError('Unsupported value: ' + JSON.stringify(value));
 }
}

assert.throws(
 // @ts-expect-error: Argument of type '"Maybe"' is not assignable to
 // parameter of type 'NoYes'.
 () => toGerman1('Maybe'),
 /^TypeError: Unsupported value: "Maybe"$/);
```

措施是：

+   在编译时，类型`NoYes`防止非法值传递给参数`value`。

+   在运行时，如果出现意外值，将使用`default`情况抛出异常。

##### 12.7.2.2 通过完整性检查防止遗漏情况

我们可以采取更多措施。以下代码执行*完整性检查*：如果我们忘记考虑所有枚举成员，TypeScript 将警告我们。

```ts
class UnsupportedValueError extends Error {
 constructor(value: never) {
 super('Unsupported value: ' + value);
 }
}

function toGerman2(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 default:
 throw new UnsupportedValueError(value);
 }
}
```

完整性检查是如何工作的？对于每种情况，TypeScript 推断`value`的类型：

```ts
function toGerman2b(value: NoYes) {
 switch (value) {
 case NoYes.No:
 // %inferred-type: NoYes.No
 value;
 return 'Nein';
 case NoYes.Yes:
 // %inferred-type: NoYes.Yes
 value;
 return 'Ja';
 default:
 // %inferred-type: never
 value;
 throw new UnsupportedValueError(value);
 }
}
```

在默认情况下，TypeScript 推断`value`的类型为`never`，因为我们永远不会到达那里。但是，如果我们向`NoYes`添加一个成员`.Maybe`，那么`value`的推断类型将是`NoYes.Maybe`。而该类型在编译时与`new UnsupportedValueError()`的参数的类型`never`静态不兼容。这就是为什么我们在编译时会得到以下错误消息：

```ts
Argument of type 'NoYes.Maybe' is not assignable to parameter of type 'never'.
```

方便的是，这种完整性检查也适用于`if`语句：

```ts
function toGerman3(value: NoYes) {
 if (value === NoYes.No) {
 return 'Nein';
 } else if (value === NoYes.Yes) {
 return 'Ja';
 } else {
 throw new UnsupportedValueError(value);
 }
}
```

##### 12.7.2.3 检查完整性的另一种方法

或者，如果我们指定返回类型，还可以获得完整性检查：

```ts
function toGerman4(value: NoYes): string {
 switch (value) {
 case NoYes.No:
 const x: NoYes.No = value;
 return 'Nein';
 case NoYes.Yes:
 const y: NoYes.Yes = value;
 return 'Ja';
 }
}
```

如果我们向`NoYes`添加一个成员，那么 TypeScript 会抱怨`toGerman4()`可能会返回`undefined`。

**这种方法的缺点：**

+   这种方法不适用于`if`语句（[更多信息](https://github.com/microsoft/TypeScript/issues/17358)）。

+   不会在运行时执行检查。

#### 12.7.3 `keyof`和枚举

我们可以使用`keyof`类型运算符来创建元素为枚举成员键的类型。当我们这样做时，我们需要将`keyof`与`typeof`结合使用：

```ts
enum HttpRequestKeyEnum {
 'Accept',
 'Accept-Charset',
 'Accept-Datetime',
 'Accept-Encoding',
 'Accept-Language',
}
// %inferred-type: "Accept" | "Accept-Charset" | "Accept-Datetime" |
// "Accept-Encoding" | "Accept-Language"
type HttpRequestKey = keyof typeof HttpRequestKeyEnum;

function getRequestHeaderValue(request: Request, key: HttpRequestKey) {
 // ···
}
```

##### 12.7.3.1 在没有`typeof`的情况下使用`keyof`

如果我们在没有`typeof`的情况下使用`keyof`，则会得到不同且不太有用的类型：

```ts
// %inferred-type: "toString" | "toFixed" | "toExponential" |
// "toPrecision" | "valueOf" | "toLocaleString"
type Keys = keyof HttpRequestKeyEnum;
```

`keyof HttpRequestKeyEnum`与`keyof number`相同。

### 12.8 致谢

+   感谢 Disqus 用户`@spira_mirabilis`对本章的反馈。

[评论](https://github.com/rauschma/tackling-ts/issues/12)
