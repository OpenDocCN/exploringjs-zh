# 七、语法

> 原文：[`exploringjs.com/impatient-js/ch_syntax.html`](https://exploringjs.com/impatient-js/ch_syntax.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   7.1 JavaScript 语法概述

    +   7.1.1 基本结构

    +   7.1.2 模块

    +   7.1.3 类

    +   7.1.4 异常处理

    +   7.1.5 合法的变量和属性名称

    +   7.1.6 大小写风格

    +   7.1.7 名称的大写形式

    +   7.1.8 更多命名约定

    +   7.1.9 分号放在哪里？

+   7.2 （高级）

+   7.3 标识符

    +   7.3.1 有效的标识符（变量名等）

    +   7.3.2 保留字

+   7.4 语句 vs 表达式

    +   7.4.1 语句

    +   7.4.2 表达式

    +   7.4.3 允许的位置

+   7.5 模糊的语法

    +   7.5.1 相同的语法：函数声明和函数表达式

    +   7.5.2 相同的语法：对象字面量和块

    +   7.5.3 消歧

+   7.6 分号

    +   7.6.1 分号的经验法则

    +   7.6.2 分号：控制语句

+   7.7 自动分号插入（ASI）

    +   7.7.1 意外触发的 ASI

    +   7.7.2 意外未触发的 ASI

+   7.8 分号：最佳实践

+   7.9 严格模式 vs 松散模式

    +   7.9.1 切换到严格模式

    +   7.9.2 严格模式的改进

* * *

### 7.1 JavaScript 语法概述

这是对 JavaScript 语法的第一次简要介绍。如果有些东西还不明白，不要担心。它们将在本书的后面更详细地解释。

这个概述也不是详尽无遗的。它侧重于基本要点。

#### 7.1.1 基本结构

##### 7.1.1.1 注释

```js
// single-line comment

/*
Comment with
multiple lines
*/
```

##### 7.1.1.2 *原始*（原子）值

**布尔值：**

```js
true
false
```

**数字：**

```js
1.141
-123
```

基本数字类型用于浮点数（双精度）和整数。

**大整数：**

```js
17n
-49n
```

基本数字类型只能在 53 位加符号的范围内正确表示整数。大整数可以任意增长。

**字符串：**

```js
'abc'
"abc"
`String with interpolated values: ${256} and ${true}`
```

JavaScript 没有额外的字符类型。它使用字符串来表示它们。

##### 7.1.1.3 断言

*断言*描述了计算结果预期的样子，并在这些期望不正确时抛出异常。例如，以下断言说明了计算结果 7 加 1 必须是 8：

```js
assert.equal(7 + 1, 8);
```

`assert.equal()`是一个方法调用（对象是`assert`，方法是`.equal()`），有两个参数：实际结果和期望结果。它是 Node.js 断言 API 的一部分，将在本书的后面解释。

还有`assert.deepEqual()`用于深度比较对象。

##### 7.1.1.4 记录到控制台

记录到控制台的浏览器或 Node.js：

```js
// Printing a value to standard out (another method call)
console.log('Hello!');

// Printing error information to standard error
console.error('Something went wrong!');
```

##### 7.1.1.5 运算符

```js
// Operators for booleans
assert.equal(true && false, false); // And
assert.equal(true || false, true); // Or

// Operators for numbers
assert.equal(3 + 4, 7);
assert.equal(5 - 1, 4);
assert.equal(3 * 4, 12);
assert.equal(10 / 4, 2.5);

// Operators for bigints
assert.equal(3n + 4n, 7n);
assert.equal(5n - 1n, 4n);
assert.equal(3n * 4n, 12n);
assert.equal(10n / 4n, 2n);

// Operators for strings
assert.equal('a' + 'b', 'ab');
assert.equal('I see ' + 3 + ' monkeys', 'I see 3 monkeys');

// Comparison operators
assert.equal(3 < 4, true);
assert.equal(3 <= 4, true);
assert.equal('abc' === 'abc', true);
assert.equal('abc' !== 'def', true);
```

JavaScript 还有一个`==`比较运算符。我建议避免使用它-原因在§13.4.3“建议：始终使用严格相等”中有解释。

##### 7.1.1.6 声明变量

`const`创建*不可变变量绑定*：每个变量必须立即初始化，我们不能以后分配不同的值。但是，值本身可能是可变的，我们可能能够更改其内容。换句话说：`const`不会使值不可变。

```js
// Declaring and initializing x (immutable binding):
const x = 8;

// Would cause a TypeError:
// x = 9;
```

`let`创建*可变变量绑定*：

```js
// Declaring y (mutable binding):
let y;

// We can assign a different value to y:
y = 3 * 5;

// Declaring and initializing z:
let z = 3 * 5;
```

##### 7.1.1.7 普通函数声明

```js
// add1() has the parameters a and b
function add1(a, b) {
 return a + b;
}
// Calling function add1()
assert.equal(add1(5, 2), 7);
```

##### 7.1.1.8 箭头函数表达式

箭头函数表达式通常用作函数调用和方法调用的参数：

```js
const add2 = (a, b) => { return a + b };
// Calling function add2()
assert.equal(add2(5, 2), 7);

// Equivalent to add2:
const add3 = (a, b) => a + b;
```

前面的代码包含以下两个箭头函数（*表达式*和*语句*的术语在本章的后面有解释）：

```js
// An arrow function whose body is a code block
(a, b) => { return a + b }

// An arrow function whose body is an expression
(a, b) => a + b
```

##### 7.1.1.9 普通对象

```js
// Creating a plain object via an object literal
const obj = {
 first: 'Jane', // property
 last: 'Doe', // property
 getFullName() { // property (method)
 return this.first + ' ' + this.last;
 },
};

// Getting a property value
assert.equal(obj.first, 'Jane');
// Setting a property value
obj.first = 'Janey';

// Calling the method
assert.equal(obj.getFullName(), 'Janey Doe');
```

##### 7.1.1.10 数组

```js
// Creating an Array via an Array literal
const arr = ['a', 'b', 'c'];
assert.equal(arr.length, 3);

// Getting an Array element
assert.equal(arr[1], 'b');
// Setting an Array element
arr[1] = 'β';

// Adding an element to an Array:
arr.push('d');

assert.deepEqual(
 arr, ['a', 'β', 'c', 'd']);
```

##### 7.1.1.11 控制流语句

条件语句：

```js
if (x < 0) {
 x = -x;
}
```

`for-of`循环：

```js
const arr = ['a', 'b'];
for (const element of arr) {
 console.log(element);
}
// Output:
// 'a'
// 'b'
```

#### 7.1.2 模块

每个模块都是单个文件。例如，考虑以下两个包含模块的文件：

```js
file-tools.mjs
main.mjs
```

`file-tools.mjs`中的模块导出其函数`isTextFilePath()`：

```js
export function isTextFilePath(filePath) {
 return filePath.endsWith('.txt');
}
```

`main.mjs`中的模块导入整个模块`path`和函数`isTextFilePath()`：

```js
// Import whole module as namespace object `path`
import * as path from 'path';
// Import a single export of module file-tools.mjs
import {isTextFilePath} from './file-tools.mjs';
```

#### 7.1.3 类

```js
class Person {
 constructor(name) {
 this.name = name;
 }
 describe() {
 return `Person named ${this.name}`;
 }
 static logNames(persons) {
 for (const person of persons) {
 console.log(person.name);
 }
 }
}

class Employee extends Person {
 constructor(name, title) {
 super(name);
 this.title = title;
 }
 describe() {
 return super.describe() +
 ` (${this.title})`;
 }
}

const jane = new Employee('Jane', 'CTO');
assert.equal(
 jane.describe(),
 'Person named Jane (CTO)');
```

#### 7.1.4 异常处理

```js
function throwsException() {
 throw new Error('Problem!');
}

function catchesException() {
 try {
 throwsException();
 } catch (err) {
 assert.ok(err instanceof Error);
 assert.equal(err.message, 'Problem!');
 }
}
```

注意：

+   `try-finally`和`try-catch-finally`也受支持。

+   我们可以抛出任何值，但是堆栈跟踪等功能仅由`Error`及其子类支持。

#### 7.1.5 合法的变量和属性名

变量名和属性名的语法类别称为*标识符*。

标识符允许具有以下字符：

+   Unicode 字母：`A`–`Z`，`a`–`z`（等等）

+   `$`，`_`

+   Unicode 数字：`0`–`9`（等等）

    +   变量名不能以数字开头

一些单词在 JavaScript 中具有特殊含义，被称为*保留字*。例如：`if`，`true`，`const`。

保留字不能用作变量名：

```js
const if = 123;
 // SyntaxError: Unexpected token if
```

但它们允许作为属性的名称：

```js
> const obj = { if: 123 };
> obj.if
123
```

#### 7.1.6 大小写风格

连接单词的常见大小写风格有：

+   驼峰命名法：`threeConcatenatedWords`

+   下划线命名法（也称为*蛇形命名法*）：`three_concatenated_words`

+   破折号命名法（也称为*烤肉串命名法*）：`three-concatenated-words`

#### 7.1.7 名称的大写形式

一般来说，JavaScript 使用驼峰命名法，除了常量。

小写：

+   函数，变量：`myFunction`

+   方法：`obj.myMethod`

+   CSS：

    +   CSS 实体：`special-class`

    +   对应的 JavaScript 变量：`specialClass`

大写：

+   类：`MyClass`

+   常量：`MY_CONSTANT`

    +   常量通常也以驼峰命名法编写：`myConstant`

#### 7.1.8 更多命名约定

以下命名约定在 JavaScript 中很受欢迎。

如果参数名以下划线开头（或者是下划线），则表示该参数未被使用——例如：

```js
arr.map((_x, i) => i)
```

如果对象的属性名以下划线开头，则该属性被视为私有：

```js
class ValueWrapper {
 constructor(value) {
 this._value = value;
 }
}
```

#### 7.1.9 分号放在哪里？

在语句结束时：

```js
const x = 123;
func();
```

但如果该语句以大括号结束，则不是条件语句：

```js
while (false) {
 // ···
} // no semicolon

function func() {
 // ···
} // no semicolon
```

但在这样的语句后添加分号不是语法错误——它被解释为空语句：

```js
// Function declaration followed by empty statement:
function func() {
 // ···
};
```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验：基础**

参见测验应用程序。

### 7.2（高级）

本章的所有其余部分都是高级内容。

### 7.3 标识符

#### 7.3.1 有效标识符（变量名等）

首字符：

+   Unicode 字母（包括重音字符，如`é`和`ü`，以及非拉丁字母的字符，如`α`）

+   `$`

+   `_`

后续字符：

+   合法的首字符

+   Unicode 数字（包括东阿拉伯数字）

+   一些其他 Unicode 标记和标点符号

例子：

```js
const ε = 0.0001;
const строка = '';
let _tmp = 0;
const $foo2 = true;
```

#### 7.3.2 保留字

保留字不能作为变量名，但可以作为属性名。

所有 JavaScript *关键字*都是保留字：

> `await` `break` `case` `catch` `class` `const` `continue` `debugger` `default` `delete` `do` `else` `export` `extends` `finally` `for` `function` `if` `import` `in` `instanceof` `let` `new` `return` `static` `super` `switch` `this` `throw` `try` `typeof` `var` `void` `while` `with` `yield`

以下标记也是关键字，但目前在语言中没有使用：

> `enum` `implements` `package` `protected` `interface` `private` `public`

以下文字是保留字：

> `true` `false` `null`

从技术上讲，这些词并不是保留字，但你也应该避免使用它们，因为它们实际上是关键字：

> `Infinity` `NaN` `undefined` `async`

你也不应该为你自己的变量和参数使用全局变量的名称（`String`，`Math`等）。

### 7.4 语句 vs. 表达式

在本节中，我们探讨了 JavaScript 如何区分两种语法结构：*语句*和*表达式*。之后，我们会看到这可能会引起问题，因为相同的语法在不同的上下文中可能意味着不同的东西。

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **我们假装只有语句和表达式**

为了简单起见，我们假设在 JavaScript 中只有语句和表达式。

#### 7.4.1 语句

*语句*是一段可以执行并执行某种操作的代码。例如，`if`是一个语句：

```js
let myStr;
if (myBool) {
 myStr = 'Yes';
} else {
 myStr = 'No';
}
```

另一个语句的例子：函数声明。

```js
function twice(x) {
 return x + x;
}
```

#### 7.4.2 表达式

*表达式*是一段可以*评估*产生一个值的代码。例如，括号之间的代码就是一个表达式：

```js
let myStr = (myBool ? 'Yes' : 'No');
```

括号之间使用的运算符`_?_:_`称为*三元运算符*。它是`if`语句的表达式版本。

让我们看更多表达式的例子。我们输入表达式，REPL 为我们评估它们：

```js
> 'ab' + 'cd'
'abcd'
> Number('123')
123
> true || false
true
```

#### 7.4.3 允许在哪里？

JavaScript 源代码中的当前位置决定了你可以使用哪种语法结构：

+   一个函数的主体必须是一系列语句：

    ```js
    function max(x, y) {
     if (x > y) {
     return x;
     } else {
     return y;
     }
    }
    ```

+   函数调用或方法调用的参数必须是表达式：

    ```js
    console.log('ab' + 'cd', Number('123'));
    ```

然而，表达式可以用作语句。然后它们被称为*表达式语句*。相反的情况并不成立：当上下文要求一个表达式时，你不能使用一个语句。

以下代码演示了任何表达式`bar()`都可以是表达式或语句 - 这取决于上下文：

```js
function f() {
 console.log(bar()); // bar() is expression
 bar(); // bar(); is (expression) statement 
}
```

### 7.5 歧义语法

JavaScript 有几种编程结构在语法上是模棱两可的：相同的语法根据是在语句上下文还是表达式上下文中使用而有不同的解释。本节探讨了这种现象和它引起的陷阱。

#### 7.5.1 相同的语法：函数声明和函数表达式

函数声明是一个语句：

```js
function id(x) {
 return x;
}
```

*函数表达式*是一个表达式（`=`的右侧）：

```js
const id = function me(x) {
 return x;
};
```

#### 7.5.2 相同的语法：对象字面量和代码块

在下面的代码中，`{}`是一个*对象字面量*：一个创建空对象的表达式。

```js
const obj = {};
```

这是一个空代码块（一个语句）：

```js
{
}
```

#### 7.5.3 消除歧义

歧义只在语句上下文中是一个问题：如果 JavaScript 解析器遇到歧义的语法，它不知道它是一个普通语句还是一个表达式语句。例如：

+   如果一个语句以`function`开头：它是函数声明还是函数表达式？

+   如果一个语句以`{`开头：它是对象字面量还是代码块？

为了消除歧义，以`function`或`{`开头的语句永远不会被解释为表达式。如果你想要一个表达式语句以这些标记之一开头，你必须将其包裹在括号中：

```js
(function (x) { console.log(x) })('abc');

// Output:
// 'abc'
```

在这段代码中：

1.  我们首先通过函数表达式创建一个函数：

    ```js
    function (x) { console.log(x) }
    ```

1.  然后我们调用那个函数：`('abc')`

在（1）中显示的代码片段之所以被解释为表达式，是因为我们将其包裹在括号中。如果我们不这样做，我们将得到一个语法错误，因为此时 JavaScript 期望一个函数声明，并抱怨缺少函数名。此外，你不能在函数声明后立即放置一个函数调用。

在本书的后面，我们将看到由语法歧义引起的更多陷阱的例子：

+   通过对象解构赋值

+   从箭头函数返回对象文字

### 7.6 分号

#### 7.6.1 分号的经验法则

每个语句都以分号结束：

```js
const x = 3;
someFunction('abc');
i++;
```

除了以块结束的语句：

```js
function foo() {
 // ···
}
if (y > 0) {
 // ···
}
```

以下情况稍微棘手：

```js
const func = () => {}; // semicolon!
```

整个`const`声明（一个语句）以分号结束，但在其中有一个箭头函数表达式。也就是说，结束语句并不是语句本身以花括号结束；而是嵌入的箭头函数表达式。这就是为什么最后有一个分号。

#### 7.6.2 分号：控制语句

控制语句的主体本身就是一个语句。例如，这是`while`循环的语法：

```js
while (condition)
 statement
```

主体可以是一个单独的语句：

```js
while (a > 0) a--;
```

但是块也是语句，因此是控制语句的合法主体：

```js
while (a > 0) {
 a--;
}
```

如果你想让一个循环有一个空主体，你的第一个选择是一个空语句（只是一个分号）：

```js
while (processNextItem() > 0);
```

你的第二个选择是一个空块：

```js
while (processNextItem() > 0) {}
```

### 7.7 自动分号插入（ASI）

虽然我建议总是写分号，但在 JavaScript 中大多数分号是可选的。使这种可能的机制称为*自动分号插入*（ASI）。在某种程度上，它纠正了语法错误。

ASI 的工作方式如下。语句的解析会一直持续，直到出现以下情况之一：

+   一个分号

+   行终止符后跟一个非法标记

换句话说，ASI 可以被看作是在换行符处插入分号。接下来的小节将介绍 ASI 的陷阱。

#### 7.7.1 意外触发的 ASI

关于 ASI 的好消息是 - 如果你不依赖它并总是写分号 - 那么你需要注意的只有一个陷阱。那就是 JavaScript 禁止在某些标记后换行。如果你插入一个换行符，分号也会被插入。

这在实际上最相关的标记是`return`。例如，考虑以下代码：

```js
return
{
 first: 'jane'
};
```

这段代码被解析为：

```js
return;
{
 first: 'jane';
}
;
```

也就是说：

+   没有操作数的返回语句：`return;`

+   代码块的开始：`{`

+   表达式语句`'jane';`与标签`first:`

+   代码块的结束：`}`

+   空语句：`;`

为什么 JavaScript 会这样做？它防止在`return`后的行中意外返回一个值。

#### 7.7.2 意外未触发的 ASI

在某些情况下，当你认为应该触发 ASI 时，ASI *没有*被触发。这使得不喜欢分号的人的生活变得更加复杂，因为他们需要注意这些情况。以下是三个例子。还有更多。

**例 1：**意外的函数调用。

```js
a = b + c
(d + e).print()
```

解析为：

```js
a = b + c(d + e).print();
```

**例 2：**意外的除法。

```js
a = b
/hi/g.exec(c).map(d)
```

解析为：

```js
a = b / hi / g.exec(c).map(d);
```

**例 3：**意外的属性访问。

```js
someFunction()
['ul', 'ol'].map(x => x + x)
```

执行为：

```js
const propKey = ('ul','ol'); // comma operator
assert.equal(propKey, 'ol');

someFunction()[propKey].map(x => x + x);
```

### 7.8 分号：最佳实践

我建议你总是写分号：

+   我喜欢它给代码带来的视觉结构 - 你清楚地看到语句在哪里结束。

+   要记住的规则较少。

+   大多数 JavaScript 程序员使用分号。

然而，也有很多人不喜欢分号增加的视觉混乱。如果你是其中之一：不用它们的代码是合法的。我建议你使用工具来帮助你避免错误。以下是两个例子：

+   自动代码格式化程序[Prettier](https://prettier.io)可以配置为不使用分号。它然后会自动修复问题。例如，如果它遇到以方括号开头的行，它会在该行前面加上一个分号。

+   静态检查器[ESLint](https://eslint.org)有[一个规则](https://eslint.org/docs/rules/semi)，告诉你首选的风格（总是分号或尽可能少的分号），并警告你关于关键问题。

### 7.9 严格模式与松散模式

从 ECMAScript 5 开始，JavaScript 有两种可以执行的*模式*：

+   正常的“松散”模式是脚本的默认模式（作为模块的前导片段，并受浏览器支持）。

+   严格模式是模块和类的默认模式，并且可以在脚本中打开（稍后会解释）。在此模式下，消除了普通模式的几个陷阱，并且会抛出更多的异常。

在现代 JavaScript 代码中，您很少会遇到懈怠模式，它几乎总是位于模块中。在本书中，我假设严格模式总是打开的。

#### 7.9.1 打开严格模式

在脚本文件和 CommonJS 模块中，您可以通过在第一行放置以下代码来为整个文件切换到严格模式：

```js
'use strict';
```

这个“指令”的好处是，ECMAScript 5 之前的版本简单地忽略它：它是一个什么都不做的表达式语句。

您还可以仅为单个函数切换到严格模式：

```js
function functionInStrictMode() {
 'use strict';
}
```

#### 7.9.2 严格模式的改进

让我们看看严格模式比懈怠模式做得更好的三件事。在这一部分中，所有代码片段都在懈怠模式下执行。

##### 7.9.2.1 懈怠模式陷阱：更改未声明的变量会创建全局变量

在非严格模式下，更改未声明的变量会创建一个全局变量。

```js
function sloppyFunc() {
 undeclaredVar1 = 123;
}
sloppyFunc();
// Created global variable `undeclaredVar1`:
assert.equal(undeclaredVar1, 123);
```

严格模式做得更好，并抛出`ReferenceError`。这样更容易检测拼写错误。

```js
function strictFunc() {
 'use strict';
 undeclaredVar2 = 123;
}
assert.throws(
 () => strictFunc(),
 {
 name: 'ReferenceError',
 message: 'undeclaredVar2 is not defined',
 });
```

`assert.throws()`声明其第一个参数，一个函数，在调用时会抛出`ReferenceError`。

##### 7.9.2.2 在严格模式下，函数声明是块作用域的，在懈怠模式下是函数作用域的

在严格模式下，通过函数声明创建的变量仅存在于最内层的封闭块中：

```js
function strictFunc() {
 'use strict';
 {
 function foo() { return 123 }
 }
 return foo(); // ReferenceError
}
assert.throws(
 () => strictFunc(),
 {
 name: 'ReferenceError',
 message: 'foo is not defined',
 });
```

在懈怠模式下，函数声明是函数作用域的：

```js
function sloppyFunc() {
 {
 function foo() { return 123 }
 }
 return foo(); // works
}
assert.equal(sloppyFunc(), 123);
```

##### 7.9.2.3 懈怠模式在更改不可变数据时不会抛出异常

在严格模式下，如果尝试更改不可变数据，会得到一个异常：

```js
function strictFunc() {
 'use strict';
 true.prop = 1; // TypeError
}
assert.throws(
 () => strictFunc(),
 {
 name: 'TypeError',
 message: "Cannot create property 'prop' on boolean 'true'",
 });
```

在懈怠模式下，赋值会悄无声息地失败：

```js
function sloppyFunc() {
 true.prop = 1; // fails silently
 return true.prop;
}
assert.equal(sloppyFunc(), undefined);
```

![](img/aec4653f22c8cf0e517ff5024759dfe1.png) **进一步阅读：懈怠模式**

有关懈怠模式与严格模式之间的区别的更多信息，请参见[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验：高级**

查看测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/5)
