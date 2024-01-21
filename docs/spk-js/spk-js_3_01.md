# 第七章：JavaScript 的语法

> 原文：[7. JavaScript’s Syntax](https://exploringjs.com/es5/ch07.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 的语法相当简单。本章描述了需要注意的事项。

## 语法概述

本节让你快速了解 JavaScript 的语法是什么样子的。

以下是五种基本类型的值：

+   布尔值：

    ```js
    true
    false
    ```

+   数字：

    ```js
    1023
    7.851
    ```

+   字符串：

    ```js
    'hello'
    "hello"
    ```

+   普通对象：

    ```js
    {
        firstName: 'Jane',
        lastName: 'Doe'
    }
    ```

+   数组：

    ```js
    [ 'apple', 'banana', 'cherry' ]
    ```

以下是一些基本语法的例子：

```js
// Two slashes start single-linecomments

var x;  // declaring a variable

x = 3 + y;  // assigning a value to the variable `x`

foo(x, y);  // calling function `foo` with parameters `x` and `y`
obj.bar(3);  // calling method `bar` of object `obj`

// A conditional statement
if (x === 0) {  // Is `x` equal to zero?
    x = 123;
}

// Defining function `baz` with parameters `a` and `b`
function baz(a, b) {
    return a + b;
}
```

注意等号的两种不同用法：

+   单个等号（`=`）用于将一个值赋给一个变量。

+   三个等号（`===`）用于比较两个值（参见[相等运算符](ch01.html#basic_equality_operators "相等运算符")）。

## 注释

有两种注释：

+   通过`//`进行单行注释，延伸到行的其余部分。这是一个例子：

    ```js
    var a = 0; // init
    ```

+   通过`/* */`进行多行注释，可以延伸到任意范围的文本。它们不能嵌套。以下是两个例子：

    ```js
    /* temporarily disabled
    processNext(queue);
    */

    function (a /* int */, b /* str */) {
    }
    ```

## 表达式与语句

本节讨论了 JavaScript 中一个重要的语法区别：表达式和语句之间的区别。

### 表达式

*表达式*产生一个值，并且可以在期望值的任何地方编写，例如，在函数调用的参数中或赋值的右侧。以下每一行都包含一个表达式：

```js
myvar
3 + x
myfunc('a', 'b')
```

### 语句

粗略地说，*语句*执行一个动作。循环和`if`语句是语句的例子。程序基本上是一系列语句。⁶

无论在哪里，JavaScript 都期望一个语句，你也可以写一个表达式。这样的语句称为*表达式语句*。反之则不成立：你不能在 JavaScript 期望表达式的地方写一个语句。例如，`if`语句不能成为函数的参数。

#### 条件语句与条件表达式

如果我们看一下两个语法类别的成员，即`if`语句和条件运算符（一个表达式），那么语句和表达式之间的区别就变得更加清晰了。

以下是一个`if`语句的例子：

```js
var salutation;
if (male) {
    salutation = 'Mr.';
} else {
    salutation = 'Mrs.';
}
```

还有一种类似的表达式，*条件运算符*。前面的语句等同于以下代码：

```js
var salutation = (male ? 'Mr.' : 'Mrs.');
```

等号和分号之间的代码是一个表达式。括号不是必需的，但我发现如果我把它放在括号中，条件运算符更容易阅读。

#### 使用模棱两可的表达式作为语句

两种表达式看起来像语句——它们在语法类别上是模棱两可的：

+   对象文字（表达式）看起来像块（语句）：

    ```js
    {
        foo: bar(3, 5)
    }
    ```

前面的结构要么是一个对象文字（详细信息：[对象文字](ch17_split_000.html#object_literals "对象文字")），要么是一个块，后面跟着标签`foo:`，再跟着函数调用`bar(3, 5)`。

+   命名函数表达式看起来像函数声明（语句）：

    ```js
    function foo() { }
    ```

前面的结构要么是一个命名的函数表达式，要么是一个函数声明。前者产生一个函数，后者创建一个变量并将一个函数赋给它（有关两种函数定义的详细信息：[定义函数](ch15.html#defining_functions "定义函数")）。

为了在解析过程中避免歧义，JavaScript 不允许你将对象文字和函数表达式用作语句。也就是说，表达式语句不能以以下内容开头：

+   花括号

+   关键字`function`

如果一个表达式以这两个标记中的任何一个开头，它只能出现在表达式上下文中。例如，你可以通过在表达式周围放置括号来满足这个要求。接下来，我们将看两个必要的例子。

#### 通过 eval()评估对象文字

`eval`在语句上下文中解析其参数。如果你想要`eval`返回一个对象，你必须在对象文字周围放括号：

```js
> eval('{ foo: 123 }')
123
> eval('({ foo: 123 })')
{ foo: 123 }
```

#### 立即调用函数表达式

以下代码是一个“立即调用的函数表达式”（IIFE），一个函数的主体会立即执行（您将在[通过 IIFE 引入新作用域](ch16.html#iife "通过 IIFE 引入新作用域")中了解到 IIFE 的用途）：

```js
> (function () { return 'abc' }())
'abc'
```

如果省略括号，您将得到语法错误，因为 JavaScript 看到一个函数声明，它不能是匿名的：

```js
> function () { return 'abc' }()
SyntaxError: function statement requires a name
```

如果添加名称，您也会得到语法错误，因为函数声明不能立即调用：

```js
> function foo() { return 'abc' }()
SyntaxError: Unexpected token )
```

函数声明后面必须是一个合法的语句，而`()`不是。

## 控制流语句和块

对于控制流语句，主体是一个单语句。以下是两个示例：

```js
if (obj !== null) obj.foo();

while (x > 0) x--;
```

然而，任何语句都可以被*块*替换，即包含零个或多个语句的花括号。因此，您也可以这样写：

```js
if (obj !== null) {
    obj.foo();
}

while (x > 0) {
    x--;
}
```

我更喜欢后一种控制流语句形式。对其进行标准化意味着单语句主体和多语句主体之间没有区别。因此，您的代码看起来更一致，并且在单语句和多于一条语句之间切换更容易。

## 使用分号的规则

在本节中，我们将讨论 JavaScript 中分号的使用。基本规则是：

+   通常，语句以分号终止。

+   例外是以块结束的语句。

JavaScript 中分号是可选的。缺少分号会通过所谓的“自动分号插入”（ASI）添加（请参阅[自动分号插入](ch07.html#automatic_semicolon_insertion "自动分号插入")）。然而，该功能并不总是按预期工作，这就是为什么您应该始终包括分号的原因。

### 语句结束于块之后没有分号

如果以块结束，以下语句不会以分号终止：

+   循环：`for`，`while`（但不包括`do-while`）

+   分支：`if`，`switch`，`try`

+   函数声明（但不是函数表达式）

以下是`while`与`do-while`的示例：

```js
while (a > 0) {
    a--;
} // no semicolon

do {
    a--;
} while (a > 0);
```

以下是函数声明与函数表达式的示例。后者后面跟着一个分号，因为它出现在`var`声明内（它*以*分号结束）：

```js
function foo() {
    // ...
} // no semicolon

var foo = function () {
    // ...
};
```

### 注意

如果在块后添加分号，您不会得到语法错误，因为它被视为一个空语句（请参阅下一节）。

### 提示

这就是您需要了解的关于分号的大部分内容。如果您始终添加分号，您可能可以不阅读本节其余部分。

### 空语句

分号本身是一个“空语句”，什么也不做。空语句可以出现在需要语句的任何地方。它们在需要语句但不需要语句的情况下很有用。在这种情况下，通常也允许块。例如，以下两个语句是等价的：

```js
while (processNextItem() > 0);
while (processNextItem() > 0) {}
```

函数`processNextItem`被假定返回剩余项目的数量。以下程序由三个空语句组成，也是语法上正确的：

```js
;;;
```

### 自动分号插入

自动分号插入（ASI）的目标是使分号在行末变为可选。术语“自动分号插入”所引发的图像是 JavaScript 解析器为您插入分号（在内部，通常处理方式不同）。

换句话说，ASI 帮助解析器确定语句何时结束。通常，它以分号结束。ASI 规定，如果：

+   行终止符（例如换行符）后面跟着一个非法标记。

+   遇到右括号。

+   已到达文件末尾。

#### 示例：通过非法标记进行 ASI

以下代码包含了一个行终止符后面跟着一个非法标记：

```js
if (a < 0) a = 0
console.log(a)
```

在`0`之后的`console`标记是非法的，并触发 ASI：

```js
if (a < 0) a = 0;
console.log(a);
```

#### 示例：通过右括号进行 ASI

在以下代码中，大括号内的语句未以分号终止：

```js
function add(a,b) { return a+b }
```

ASI 创建了前述代码的语法上正确的版本：

```js
function add(a,b) { return a+b; }
```

#### 陷阱：ASI 可能会意外地中断语句

如果在关键字`return`后有行终止符，ASI 也会被触发。例如：

```js
// Don't do this
return
{
    name: 'John'
};
```

ASI 将前面的转换为：

```js
return;
{
    name: 'John'
};
```

这是一个空的返回，紧接着是一个带有标签`name`的块，位于表达式语句`'John'`之前。在块之后，有一个空语句。

#### 陷阱：ASI 可能意外地不会被触发

有时，新行中的语句以允许作为前一语句的延续的标记开头。然后，尽管看起来应该被触发，但 ASI 不会被触发。例如：

```js
func()
[ 'ul', 'ol' ].foreach(function (t) { handleTag(t) })
```

第二行中的方括号被解释为对`func()`返回结果的索引。方括号内的逗号被解释为逗号运算符（在这种情况下返回`'ol'`；参见[逗号运算符](ch09.html#comma_operator "逗号运算符")）。因此，JavaScript 将前面的代码视为：

```js
func()['ol'].foreach(function (t) { handleTag(t) });
```

## 合法标识符

标识符用于命名事物，并在 JavaScript 中的各种句法角色中出现。例如，变量的名称和未引用的属性键的名称必须是有效的标识符。标识符区分大小写。

标识符的第一个字符是：

+   任何 Unicode 字母，包括拉丁字母如 D，希腊字母如λ，和西里尔字母如Д

+   美元符号（`$`）

+   下划线（`_`）

后续字符是：

+   任何合法的第一个字符

+   Unicode 类别“十进制数字（Nd）”中的任何 Unicode 数字；这包括欧洲数字如 7 和印度数字如٣

+   其他各种 Unicode 标记和标点符号

合法标识符的示例：

```js
var ε = 0.0001;
var строка = '';
var _tmp;
var $foo2;
```

尽管这使您可以在 JavaScript 代码中使用各种人类语言，但我建议使用英语，无论是标识符还是注释。这可以确保您的代码可以被尽可能多的人理解，这很重要，考虑到如今代码可以在国际间传播。

以下标识符是*保留字*——它们是语法的一部分，不能用作变量名（包括函数名和参数名）：

| `arguments` | `break` | `case` | `catch` |
| --- | --- | --- | --- |
| `class` | `const` | `continue` | `debugger` |
| `default` | `delete` | `do` | `else` |
| `enum` | `export` | `extends` | `false` |
| `finally` | `for` | `function` | `if` |
| `implements` | `import` | `in` | `instanceof` |
| `interface` | `let` | `new` | `null` |
| `package` | `private` | `protected` | `public` |
| `return` | `static` | `super` | `switch` |
| `this` | `throw` | `true` | `try` |
| `typeof` | `var` | `void` | `while` |

以下三个标识符不是保留字，但您应该将它们视为保留字：

| `Infinity` |
| `NaN` |
| `undefined` |

最后，您还应该避免使用标准全局变量的名称（参见第二十三章）。您可以将它们用作局部变量而不会破坏任何内容，但您的代码仍然会变得混乱。

请注意，您*可以*使用保留字作为未引用的属性键（自 ECMAScript 5 起）：

```js
> var obj = { function: 'abc' };
> obj.function
'abc'
```

您可以在 Mathias Bynens 的博客文章[“有效的 JavaScript 变量名”](http://mathiasbynens.be/notes/javascript-identifiers)中查找标识符的精确规则。

## 在数字文字上调用方法

在方法调用中，重要的是要区分浮点数点和方法调用点。因此，您不能写成`1.toString()`；您必须使用以下替代之一：

```js
1..toString()
1 .toString()  // space before dot
(1).toString()
1.0.toString()
```

## 严格模式

ECMAScript 5 有一个*严格模式*，可以使 JavaScript 更清晰，减少不安全的特性，增加警告，以及更合乎逻辑的行为。正常（非严格）模式有时被称为“松散模式”。

### 打开严格模式

您可以通过在 JavaScript 文件中或在`<script>`元素内首先输入以下行来打开严格模式：

```js
'use strict';
```

请注意，不支持 ECMAScript 5 的 JavaScript 引擎将简单地忽略前面的语句，因为以这种方式编写字符串（作为表达式语句；请参阅[语句](ch07.html#expression_statement "语句")）通常不会做任何事情。

您还可以按函数打开严格模式。要这样做，请像这样编写您的函数：

```js
function foo() {
    'use strict';
    ...
}
```

当您使用严格模式处理到处都可能破坏事物的旧代码库时，这很方便。

### 严格模式：推荐，但有注意事项

总的来说，严格模式启用的更改都是为了更好。因此，强烈建议您在编写新代码时使用它——只需在文件开头打开它。然而，有两个注意事项：

为现有代码启用严格模式可能会破坏它

代码可能依赖于不再可用的功能，或者可能依赖于在松散模式和严格模式中行为不同的行为。不要忘记您可以将单个严格模式函数添加到处于松散模式的文件中的选项。

小心处理包

当您连接和/或缩小文件时，您必须小心，严格模式在应该打开时没有关闭，或者反之亦然。两者都可能破坏代码。

以下部分详细解释了严格模式的特性。通常情况下，您不需要了解它们，因为您大多数情况下会因为您本不应该做的事情而得到更多的警告。

### 变量必须在严格模式下声明

在严格模式下，所有变量必须明确声明。这有助于防止拼写错误。在松散模式下，对未声明的变量进行赋值会创建一个全局变量：

```js
function sloppyFunc() {
    sloppyVar = 123;
}
sloppyFunc();  // creates global variable `sloppyVar`
console.log(sloppyVar);  // 123
```

在严格模式下，对未声明的变量进行赋值会引发异常：

```js
function strictFunc() {
    'use strict';
    strictVar = 123;
}
strictFunc();  // ReferenceError: strictVar is not defined
```

### 严格模式中的函数

严格模式限制了与函数相关的特性。

#### 函数必须在作用域的顶层声明

在严格模式下，所有函数必须在作用域的顶层声明（全局作用域或直接在函数内部）。这意味着您不能将函数声明放在块内。如果这样做，您将收到一个描述性的`SyntaxError`。例如，V8 会告诉您：“在严格模式代码中，函数只能在顶层或直接在另一个函数内部声明”：

```js
function strictFunc() {
    'use strict';
    {
        // SyntaxError:
        function nested() {
        }
    }
}
```

这是无用的，因为该函数是在周围函数的范围内创建的，而不是“在”块内部。

如果您想解决此限制，可以通过变量声明和函数表达式在块内创建一个函数：

```js
function strictFunc() {
    'use strict';
    {
        // OK:
        var nested = function () {
        };
    }
}
```

#### 函数参数的更严格规则

函数参数的规则不太宽容：禁止使用相同的参数名称两次，以及与参数同名的局部变量。

#### arguments 对象的属性更少

在严格模式下，`arguments`对象更简单：属性`arguments.callee`和`arguments.caller`已被删除，您不能对变量`arguments`进行赋值，`arguments`不会跟踪参数的更改（如果参数更改，相应的数组元素不会随之更改）。[arguments 的弃用特性](ch15.html#arguments_deprecated_features "arguments 的弃用特性")解释了详细信息。

#### 非方法函数中的`this`是未定义的

在松散模式下，非方法函数中`this`的值是全局对象（在浏览器中是`window`；请参阅[全局对象](ch16.html#global_object "全局对象")）：

```js
function sloppyFunc() {
    console.log(this === window);  // true
}
```

在严格模式下，它是`undefined`：

```js
function strictFunc() {
    'use strict';
    console.log(this === undefined);  // true
}
```

这对构造函数很有用。例如，以下构造函数`Point`是在严格模式下的：

```js
function Point(x, y) {
    'use strict';
    this.x = x;
    this.y = y;
}
```

由于严格模式，当您意外忘记`new`并将其作为函数调用时，您会收到警告：

```js
> var pt = Point(3, 1);
TypeError: Cannot set property 'x' of undefined
```

在松散模式下，您不会收到警告，并且会创建全局变量`x`和`y`。有关详细信息，请参阅[实现构造函数的提示](ch17_split_001.html#constructor_tips "实现构造函数的提示")。

### 在严格模式下，设置和删除不可变属性会引发异常

在严格模式下，非法的属性操作会抛出异常。例如，试图设置只读属性的值会抛出异常，试图删除不可配置属性也会抛出异常。以下是一个例子：

```js
var str = 'abc';
function sloppyFunc() {
    str.length = 7;  // no effect, silent failure
    console.log(str.length);  // 3
}
function strictFunc() {
    'use strict';
    str.length = 7; // TypeError: Cannot assign to
                    // read-only property 'length'
}
```

### 在严格模式下，不能删除未经限定的标识符

在松散模式下，你可以像这样删除全局变量`foo`：

```js
delete foo
```

在严格模式下，当你尝试删除未经限定的标识符时，你会得到一个语法错误。你仍然可以像这样删除全局变量：

```js
delete window.foo;  // browsers
delete global.foo;  // Node.js
delete this.foo;    // everywhere (in global scope)
```

### 在严格模式下，eval()更加干净

在严格模式下，`eval()`函数变得不那么古怪了：在评估的字符串中声明的变量不再添加到`eval()`周围的作用域中。详情请参阅[Evaluating Code Using eval()](ch23.html#eval "Evaluating Code Using eval()")。

### 在严格模式下被禁止的特性

在严格模式下，还有两个 JavaScript 特性是被禁止的：

+   不再允许使用`with`语句（参见[The with Statement](ch13.html#with_statement "The with Statement")）。在编译时（加载代码时）会得到语法错误。

+   不再有八进制数：在松散模式下，以零开头的整数被解释为八进制（基数 8）。例如：

    ```js
    > 010 === 8
    true
    ```

在严格模式下，如果你使用这种文字类型，你会得到一个语法错误：

```js
> function f() { 'use strict'; return 010 }
SyntaxError: Octal literals are not allowed in strict mode.
```

* * *

⁶ 为了简化问题，我假装声明是语句。

