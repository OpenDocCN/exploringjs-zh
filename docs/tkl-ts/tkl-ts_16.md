# 十三、[TypeScript 中枚举的替代方案]

> 原文：[`exploringjs.com/tackling-ts/ch_enum-alternatives.html`](https://exploringjs.com/tackling-ts/ch_enum-alternatives.html)

* * *

+   13.1 单例值的联合

    +   13.1.1 原始文字类型

    +   13.1.2 字符串文字类型的联合

    +   13.1.3 符号单例类型的联合

    +   13.1.4 本节的结论：联合类型 vs. 枚举

+   13.2 鉴别联合

    +   13.2.1 步骤 1：将语法树作为类层次结构

    +   13.2.2 步骤 2：将语法树作为类的联合类型

    +   13.2.3 步骤 3：将语法树作为鉴别联合

    +   13.2.4 鉴别联合 vs. 普通联合类型

+   13.3 对象字面量作为枚举

    +   13.3.1 具有字符串值属性的对象字面量

    +   13.3.2 使用对象字面量作为枚举的优缺点

+   13.4 枚举模式

+   13.5 枚举和枚举替代方案的总结

+   13.6 致谢

* * *

上一章探讨了 TypeScript 枚举的工作原理。在本章中，我们将看看枚举的替代方案。

### 13.1 单例值的联合

枚举将成员名称映射到成员值。如果我们不需要或不想要间接引用，我们可以使用所谓的*原始文字类型*的联合 - 每个值一个。在我们能够深入了解细节之前，我们需要了解原始文字类型。

#### 13.1.1 原始文字类型

快速回顾：我们可以将类型视为值的集合。

*单例类型*是具有一个元素的类型。原始文字类型是单例类型：

```ts
type UndefinedLiteralType = undefined;
type NullLiteralType = null;

type BooleanLiteralType = true;
type NumericLiteralType = 123;
type BigIntLiteralType = 123n; // --target must be ES2020+
type StringLiteralType = 'abc';
```

`UndefinedLiteralType`是具有单个元素`undefined`的类型，等等。

在这里需要注意两个语言级别（我们在本书的早些时候已经遇到了这些级别）。考虑以下变量声明：

```ts
const abc: 'abc' = 'abc';
```

+   第一个'abc'表示一种类型（字符串文字类型）。

+   第二个'abc'表示一个值。

原始文字类型的两个用例是：

+   字符串参数的重载使得以下方法调用的第一个参数决定第二个参数的类型：

    ```ts
    elem.addEventListener('click', myEventHandler);
    ```

+   我们可以使用原始文字类型的联合来定义类型，通过列举其成员：

    ```ts
    type IceCreamFlavor = 'vanilla' | 'chocolate' | 'strawberry';
    ```

继续阅读有关第二个用例的更多信息。

#### 13.1.2 字符串文字类型的联合

我们将从枚举开始，然后将其转换为字符串文字类型的联合。

```ts
enum NoYesEnum {
 No = 'No',
 Yes = 'Yes',
}
function toGerman1(value: NoYesEnum): string {
 switch (value) {
 case NoYesEnum.No:
 return 'Nein';
 case NoYesEnum.Yes:
 return 'Ja';
 }
}
assert.equal(toGerman1(NoYesEnum.No), 'Nein');
assert.equal(toGerman1(NoYesEnum.Yes), 'Ja');
```

`NoYesStrings`是`NoYesEnum`的联合类型版本：

```ts
type NoYesStrings = 'No' | 'Yes';

function toGerman2(value: NoYesStrings): string {
 switch (value) {
 case 'No':
 return 'Nein';
 case 'Yes':
 return 'Ja';
 }
}
assert.equal(toGerman2('No'), 'Nein');
assert.equal(toGerman2('Yes'), 'Ja');
```

类型`NoYesStrings`是字符串文字类型`'No'`和`'Yes'`的联合。联合类型运算符`|`与集合论的联合运算符`∪`相关。

##### 13.1.2.1 字符串文字类型的联合可以进行穷尽性检查

以下代码演示了对字符串文字类型的联合进行穷尽性检查：

```ts
// @ts-expect-error: Function lacks ending return statement and
// return type does not include 'undefined'. (2366)
function toGerman3(value: NoYesStrings): string {
 switch (value) {
 case 'Yes':
 return 'Ja';
 }
}
```

我们忘记了'No'的情况，TypeScript 警告我们该函数可能返回不是字符串的值。

我们也可以更明确地检查穷尽性：

```ts
class UnsupportedValueError extends Error {
 constructor(value: never) {
 super('Unsupported value: ' + value);
 }
}

function toGerman4(value: NoYesStrings): string {
 switch (value) {
 case 'Yes':
 return 'Ja';
 default:
 // @ts-expect-error: Argument of type '"No"' is not
 // assignable to parameter of type 'never'. (2345)
 throw new UnsupportedValueError(value);
 }
}
```

现在 TypeScript 警告我们，如果 `value` 是 `'No'`，则会到达 `default` 情况。

![](img/65d35c0a2478236e12cc4321e1b02db6.png)  **有关穷尽性检查的更多信息**

有关此主题的更多信息，请参见§12.7.2.2 “通过穷尽性检查防止遗漏情况”。

##### 13.1.2.2 缺点：字符串文字的联合类型在类型安全性上不如其他类型

字符串文字联合的一个缺点是非成员值可能被误认为是成员：

```ts
type Spanish = 'no' | 'sí';
type English = 'no' | 'yes';

const spanishWord: Spanish = 'no';
const englishWord: English = spanishWord;
```

这是合理的，因为西班牙语的 `'no'` 和英语的 `'no'` 是相同的值。实际问题在于没有办法给它们不同的标识。

#### 13.1.3 符号单例类型的联合

##### 13.1.3.1 示例：`LogLevel`

我们也可以使用符号单例类型的联合，而不是字符串文字类型的联合。这次让我们从一个不同的枚举开始：

```ts
enum LogLevel {
 off = 'off',
 info = 'info',
 warn = 'warn',
 error = 'error',
}
```

转换为符号单例类型的联合，如下所示：

```ts
const off = Symbol('off');
const info = Symbol('info');
const warn = Symbol('warn');
const error = Symbol('error');

// %inferred-type: unique symbol | unique symbol |
// unique symbol | unique symbol
type LogLevel =
 | typeof off
 | typeof info
 | typeof warn
 | typeof error
;
```

为什么我们在这里需要 `typeof`？`off` 等是值，不能出现在类型方程中。类型运算符 `typeof` 通过将值转换为类型来解决此问题。

让我们考虑前面示例的两种变体。

##### 13.1.3.2 变体 #1：内联符号

我们可以内联符号（而不是引用单独的 `const` 声明）吗？遗憾的是，类型运算符 `typeof` 的操作数必须是标识符或由点分隔的标识符“路径”。因此，这种语法是非法的：

```ts
type LogLevel = typeof Symbol('off') | ···
```

##### 13.1.3.3 变体 #2：`let` 而不是 `const`

我们可以使用 `let` 而不是 `const` 来声明变量吗？（这不一定是一种改进，但仍然是一个有趣的问题。）

我们不能这样做，因为我们需要 TypeScript 为 `const` 声明的变量推断出更窄的类型：

```ts
// %inferred-type: unique symbol
const constSymbol = Symbol('constSymbol');

// %inferred-type: symbol
let letSymbol1 = Symbol('letSymbol1');
```

使用 `let`，`LogLevel` 只是 `symbol` 的别名。

`const` 断言通常解决这种问题。但在这种情况下不起作用：

```ts
// @ts-expect-error: A 'const' assertions can only be applied to references to enum
// members, or string, number, boolean, array, or object literals. (1355)
let letSymbol2 = Symbol('letSymbol2') as const;
```

##### 13.1.3.4 在函数中使用 `LogLevel`

以下函数将 `LogLevel` 的成员转换为字符串：

```ts
function getName(logLevel: LogLevel): string {
 switch (logLevel) {
 case off:
 return 'off';
 case info:
 return 'info';
 case warn:
 return 'warn';
 case error:
 return 'error';
 }
}

assert.equal(
 getName(warn), 'warn');
```

##### 13.1.3.5 符号单例类型的联合 vs. 字符串文字类型的联合

这两种方法如何比较？

+   穷尽性检查对两者都适用。

+   使用符号更加冗长。

+   每个符号“文字”都创建一个独特的符号，不会与任何其他符号混淆。对于字符串文字来说并非如此。详情请继续阅读。

回想一下西班牙语的 `'no'` 被误认为是英语的 `'no'` 的例子：

```ts
type Spanish = 'no' | 'sí';
type English = 'no' | 'yes';

const spanishWord: Spanish = 'no';
const englishWord: English = spanishWord;
```

如果我们使用符号，我们就不会有这个问题：

```ts
const spanishNo = Symbol('no');
const spanishSí = Symbol('sí');
type Spanish = typeof spanishNo | typeof spanishSí;

const englishNo = Symbol('no');
const englishYes = Symbol('yes');
type English = typeof englishNo | typeof englishYes;

const spanishWord: Spanish = spanishNo;
// @ts-expect-error: Type 'unique symbol' is not assignable to type 'English'. (2322)
const englishWord: English = spanishNo;
```

#### 13.1.4 本节的结论：联合类型 vs. 枚举

联合类型和枚举有一些共同点：

+   我们可以自动完成成员值。但我们做法不同：

    +   使用枚举后，我们在枚举名称和点之后获得自动完成。

    +   使用联合类型，我们必须显式触发自动完成。

+   穷尽性检查对两者也适用。

但它们也有不同之处。联合符号单例类型的缺点是：

+   它们稍微冗长。

+   它们没有成员的命名空间。

+   从它们迁移到不同的结构（如果有必要的话）稍微困难一些：更容易找到枚举成员值被提及的地方。

联合符号单例类型的优势是：

+   它们不是自定义的 TypeScript 语言构造，因此更接近纯 JavaScript。

+   字符串枚举只在编译时是类型安全的。符号单例类型的联合在运行时也是类型安全的。

    +   这一点尤其重要，如果我们编译后的 TypeScript 代码与纯 JavaScript 代码交互。

### 13.2 辨别联合

[辨别联合](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions)与函数式编程语言中的[代数数据类型](https://wiki.haskell.org/Algebraic_data_type)相关。

要理解它们的工作原理，请考虑表示表达式的数据结构 *语法树*：

```ts
1 + 2 + 3
```

语法树要么是：

+   一个数字

+   两个语法树的相加

下一步：

1.  我们将首先为语法树创建一个面向对象的类层次结构。

1.  然后我们将把它转换为稍微更加功能化的东西。

1.  最后，我们将得到一个歧视联合。

#### 13.2.1 第 1 步：将语法树作为类层次结构

这是一个典型的面向对象的语法树实现：

```ts
// Abstract = can’t be instantiated via `new`
abstract class SyntaxTree1 {}
class NumberValue1 extends SyntaxTree1 {
 constructor(public numberValue: number) {
 super();
 }
}
class Addition1 extends SyntaxTree1 {
 constructor(public operand1: SyntaxTree1, public operand2: SyntaxTree1) {
 super();
 }
}
```

`SyntaxTree1`是`NumberValue1`和`Addition1`的超类。关键字`public`在语法上是为了方便：

+   声明实例属性`.numberValue`

+   通过参数`numberValue`初始化该属性

这是使用`SyntaxTree1`的示例：

```ts
const tree = new Addition1(
 new NumberValue1(1),
 new Addition1(
 new NumberValue1(2),
 new NumberValue1(3), // trailing comma
 ), // trailing comma
);
```

注意：[JavaScript 中允许在参数列表中使用尾随逗号](https://exploringjs.com/es2016-es2017/ch_trailing-comma-parameters.html) 自 ECMAScript 2016 以来。

#### 13.2.2 第 2 步：将语法树作为类的联合类型

如果我们通过联合类型定义语法树（行 A），我们就不需要面向对象的继承：

```ts
class NumberValue2 {
 constructor(public numberValue: number) {}
}
class Addition2 {
 constructor(public operand1: SyntaxTree2, public operand2: SyntaxTree2) {}
}
type SyntaxTree2 = NumberValue2 | Addition2; // (A)
```

由于`NumberValue2`和`Addition2`没有超类，它们不需要在它们的构造函数中调用`super()`。

有趣的是，我们以与以前相同的方式创建树：

```ts
const tree = new Addition2(
 new NumberValue2(1),
 new Addition2(
 new NumberValue2(2),
 new NumberValue2(3),
 ),
);
```

#### 13.2.3 第 3 步：将语法树作为歧视联合

最后，我们转向了歧视联合。这些是`SyntaxTree3`的类型定义：

```ts
interface NumberValue3 {
 kind: 'number-value';
 numberValue: number;
}
interface Addition3 {
 kind: 'addition';
 operand1: SyntaxTree3;
 operand2: SyntaxTree3;
}
type SyntaxTree3 = NumberValue3 | Addition3;
```

我们已经从类切换到了接口，因此从类的实例切换到了普通对象。

歧视联合的接口必须至少有一个共同的属性，并且该属性必须对每个属性具有不同的值。该属性称为*歧视器*或*标签*。`SyntaxTree3`的歧视器是`.kind`。它的类型是字符串字面类型。

比较：

+   实例的直接类由其原型确定。

+   歧视联合的成员类型由其歧视器确定。

这是一个与`SyntaxTree3`匹配的对象：

```ts
const tree: SyntaxTree3 = { // (A)
 kind: 'addition',
 operand1: {
 kind: 'number-value',
 numberValue: 1,
 },
 operand2: {
 kind: 'addition',
 operand1: {
 kind: 'number-value',
 numberValue: 2,
 },
 operand2: {
 kind: 'number-value',
 numberValue: 3,
 },
 }
};
```

我们在 A 行不需要类型注释，但它有助于确保数据具有正确的结构。如果我们不在这里这样做，我们以后会发现问题。

在下一个示例中，`tree`的类型是歧视联合。每次检查其歧视器（行 C）时，TypeScript 都会相应地更新其静态类型：

```ts
function getNumberValue(tree: SyntaxTree3) {
 // %inferred-type: SyntaxTree3
 tree; // (A)

 // @ts-expect-error: Property 'numberValue' does not exist on type 'SyntaxTree3'.
 // Property 'numberValue' does not exist on type 'Addition3'.(2339)
 tree.numberValue; // (B)

 if (tree.kind === 'number-value') { // (C)
 // %inferred-type: NumberValue3
 tree; // (D)
 return tree.numberValue; // OK!
 }
 return null;
}
```

在 A 行，我们还没有检查过歧视器`.kind`。因此，`tree`的当前类型仍然是`SyntaxTree3`，我们无法在 B 行访问属性`.numberValue`（因为联合类型的类型只有一个具有此属性）。

在 D 行，TypeScript 知道`.kind`是`'number-value'`，因此可以推断出`tree`的类型为`NumberValue3`。这就是为什么在下一行访问`.numberValue`是可以的，这次。

##### 13.2.3.1 实现歧视联合的函数

我们用一个实现歧视联合的函数示例来结束这一步。

如果有一个操作可以应用于所有子类型的成员，则类和歧视联合的方法不同：

+   面向对象的方法：使用类时，通常使用多态方法，其中每个类都有不同的实现。

+   功能方法：使用歧视联合时，通常使用一个处理所有可能情况并通过检查其参数的歧视器来决定要执行什么操作的单个函数。

以下示例演示了功能方法。歧视器在 A 行进行检查，并确定执行哪个`switch`情况。

```ts
function syntaxTreeToString(tree: SyntaxTree3): string {
 switch (tree.kind) { // (A)
 case 'addition':
 return syntaxTreeToString(tree.operand1)
 + ' + ' + syntaxTreeToString(tree.operand2);
 case 'number-value':
 return String(tree.numberValue);
 }
}

assert.equal(syntaxTreeToString(tree), '1 + 2 + 3');
```

请注意，TypeScript 对歧视联合执行穷尽性检查：如果我们忘记了某种情况，TypeScript 会警告我们。

这是先前代码的面向对象版本：

```ts
abstract class SyntaxTree1 {
 // Abstract = enforce that all subclasses implement this method:
 abstract toString(): string;
}
class NumberValue1 extends SyntaxTree1 {
 constructor(public numberValue: number) {
 super();
 }
 toString(): string {
 return String(this.numberValue);
 }
}
class Addition1 extends SyntaxTree1 {
 constructor(public operand1: SyntaxTree1, public operand2: SyntaxTree1) {
 super();
 }
 toString(): string {
 return this.operand1.toString() + ' + ' + this.operand2.toString();
 }
}

const tree = new Addition1(
 new NumberValue1(1),
 new Addition1(
 new NumberValue1(2),
 new NumberValue1(3),
 ),
);

assert.equal(tree.toString(), '1 + 2 + 3');
```

##### 13.2.3.2 可扩展性：面向对象的方法 vs. 功能方法

每种方法都很好地实现了一种可扩展性：

+   使用面向对象的方法，如果我们想要添加新操作，就必须修改每个类。但是，添加新类型不需要对现有代码进行任何更改。

+   使用功能方法时，如果我们想要添加新类型，就必须修改每个函数。相反，添加新操作很简单。

#### 13.2.4 歧视联合 vs. 普通联合类型

辨别联合和普通联合类型有两个共同点：

+   没有成员值的命名空间。

+   TypeScript 执行穷举检查。

接下来的两个小节探讨了辨别联合相对于普通联合的两个优势：

##### 13.2.4.1 好处：描述性属性名称

通过辨别联合，值得到了描述性的属性名称。让我们比较一下：

普通联合：

```ts
type FileGenerator = (webPath: string) => string;
type FileSource1 = string|FileGenerator;
```

辨别联合：

```ts
interface FileSourceFile {
 type: 'FileSourceFile',
 nativePath: string,
}
interface FileSourceGenerator {
 type: 'FileSourceGenerator',
 fileGenerator: FileGenerator,
}
type FileSource2 = FileSourceFile | FileSourceGenerator;
```

现在阅读源代码的人立即知道字符串是什么：一个本地路径名。

##### 13.2.4.2 好处：当部分是无法区分时，我们也可以使用它

以下辨别联合不能作为普通联合实现，因为我们无法在 TypeScript 中区分联合的类型。

```ts
interface TemperatureCelsius {
 type: 'TemperatureCelsius',
 value: number,
}
interface TemperatureFahrenheit {
 type: 'TemperatureFahrenheit',
 value: number,
}
type Temperature = TemperatureCelsius | TemperatureFahrenheit;
```

### 13.3 对象文字作为枚举

在 JavaScript 中，实现枚举的常见模式如下：

```ts
const Color = {
 red: Symbol('red'),
 green: Symbol('green'),
 blue: Symbol('blue'),
};
```

我们可以尝试在 TypeScript 中使用它如下：

```ts
// %inferred-type: symbol
Color.red; // (A)

// %inferred-type: symbol
type TColor2 = // (B)
 | typeof Color.red
 | typeof Color.green
 | typeof Color.blue
;

function toGerman(color: TColor): string {
 switch (color) {
 case Color.red:
 return 'rot';
 case Color.green:
 return 'grün';
 case Color.blue:
 return 'blau';
 default:
 // No exhaustiveness check (inferred type is not `never`):
 // %inferred-type: symbol
 color;

 // Prevent static error for return type:
 throw new Error();
 }
}
```

遗憾的是，`Color`的每个属性的类型都是`symbol`（A 行），而`TColor`（B 行）是`symbol`的别名。因此，我们可以将任何符号传递给`toGerman()`，TypeScript 在编译时不会抱怨：

```ts
assert.equal(
 toGerman(Color.green), 'grün');
assert.throws(
 () => toGerman(Symbol())); // no static error!
```

`const`断言通常在这种情况下有所帮助，但这次不行：

```ts
const ConstColor = {
 red: Symbol('red'),
 green: Symbol('green'),
 blue: Symbol('blue'),
} as const;

// %inferred-type: symbol
ConstColor.red;
```

唯一修复这个问题的方法是通过常量：

```ts
const red = Symbol('red');
const green = Symbol('green');
const blue = Symbol('blue');

// %inferred-type: unique symbol
red;

// %inferred-type: unique symbol | unique symbol | unique symbol
type TColor2 = typeof red | typeof green | typeof blue;
```

#### 13.3.1 具有字符串值属性的对象文字

```ts
const Color = {
 red: 'red',
 green: 'green',
 blue: 'blue',
} as const; // (A)

// %inferred-type: "red"
Color.red;

// %inferred-type: "red" | "green" | "blue"
type TColor =
 | typeof Color.red
 | typeof Color.green
 | typeof Color.blue
;
```

我们需要在 A 行使用[`as const`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)，这样`Color`的属性就不会有更一般的`string`类型。然后`TColor`也有一个比`string`更具体的类型。

与使用具有符号值属性的对象作为枚举相比，具有字符串值属性的对象为：

+   在开发时更好，因为我们得到穷举检查，并且可以为值派生一个狭窄的类型（不使用外部常量）。

+   在运行时更糟糕，因为字符串可能被误认为是枚举值。

#### 13.3.2 使用对象文字作为枚举的优势和劣势

优势：

+   我们有一个值的命名空间。

+   我们不使用自定义构造，更接近纯 JavaScript。

+   我们可以为枚举值派生一个狭窄的类型（如果我们使用字符串值属性）。

+   对这种类型执行穷举检查。

劣势：

+   没有动态成员检查（没有额外工作）。

+   非枚举值可以在静态或运行时被误认为是枚举值（如果我们使用字符串值属性）。

### 13.4 枚举模式

以下示例演示了[受 Java 启发的枚举模式](https://2ality.com/2020/01/enum-pattern.html)，它适用于纯 JavaScript 和 TypeScript：

```ts
class Color {
 static red = new Color();
 static green = new Color();
 static blue = new Color();
}

// @ts-expect-error: Function lacks ending return statement and return type
// does not include 'undefined'. (2366)
function toGerman(color: Color): string { // (A)
 switch (color) {
 case Color.red:
 return 'rot';
 case Color.green:
 return 'grün';
 case Color.blue:
 return 'blau';
 }
}

assert.equal(toGerman(Color.blue), 'blau');
```

遗憾的是，TypeScript 不执行穷举检查，这就是为什么我们在 A 行得到一个错误的原因。

### 13.5 枚举和枚举替代方案的总结

以下表格总结了 TypeScript 中枚举及其替代方案的特点：

|  | 唯一 | 命名空间 | 迭代 | 内存 CT | 内存 RT | 穷举 |
| --- | --- | --- | --- | --- | --- | --- |
| 数字枚举 | `-` | `✔` | `✔` | `✔` | `-` | `✔` |
| 字符串枚举 | `✔` | `✔` | `✔` | `✔` | `-` | `✔` |
| 字符串联合 | `-` | `-` | `-` | `✔` | `-` | `✔` |
| 符号联合 | `✔` | `-` | `-` | `✔` | `-` | `✔` |
| 辨别联合 | `-`（1） | `-` | `-` | `✔` | `-`（2） | `✔` |
| 符号属性 | `✔` | `✔` | `✔` | `-` | `-` | `-` |
| 字符串属性 | `-` | `✔` | `✔` | `✔` | `-` | `✔` |
| 枚举模式 | `✔` | `✔` | `✔` | `✔` | `✔` | `-` |

表格列的标题：

+   唯一值：没有非枚举值可以被误认为是枚举值。

+   枚举键的命名空间

+   是否可以遍历枚举值？

+   编译时值的成员检查：是否有枚举值的狭窄类型？

+   运行时值的成员检查：

    +   对于枚举模式，运行时成员检查是`instanceof`。

    +   请注意，如果可以遍历枚举值，成员检查可以相对容易地实现。

+   穷举检查（TypeScript 静态检查）

表格单元格中的脚注：

1.  辨别联合并不是真正独特的，但是将值误认为联合成员的可能性相对较小（特别是如果我们为辨别属性使用唯一的名称）。

1.  如果辨别属性有一个足够独特的名称，它可以用来检查成员资格。

### 13.6 致谢

+   感谢[Kirill Sukhomlin](https://twitter.com/kirilloid_ru)提出了如何为对象文字定义`TColor`的建议。

[评论](https://github.com/rauschma/tackling-ts/issues/13)
