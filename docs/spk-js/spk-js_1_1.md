# 第一章：基本 JavaScript

> 原文：[1. Basic JavaScript](https://exploringjs.com/es5/ch01.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章是关于“基本 JavaScript”，这是我为 JavaScript 的一个子集选择的名称，尽可能简洁，同时仍然能让你高效地工作。当你开始学习 JavaScript 时，我建议你在学习其他语言之前先在其中编程一段时间。这样，你就不必一次学习所有内容，这可能会让人困惑。

## 背景

本节简要介绍了 JavaScript 的背景，以帮助你理解它为什么是这样的。

### JavaScript 与 ECMAScript

*ECMAScript*是 JavaScript 的官方名称。之所以需要一个新名称，是因为*Java*有商标（最初由 Sun 持有，现在由 Oracle 持有）。目前，Mozilla 是少数几家被允许正式使用*JavaScript*名称的公司之一，因为它很久以前就获得了许可证。对于常见用法，以下规则适用：

+   *JavaScript*意味着编程语言。

+   *ECMAScript*是语言规范的官方名称。因此，每当提到语言的版本时，人们都说*ECMAScript*。JavaScript 的当前版本是 ECMAScript 5；ECMAScript 6 目前正在开发中。

### 影响和语言的性质

JavaScript 的创造者 Brendan Eich 别无选择，只能很快地创建这种语言（否则，Netscape 可能会采用其他更糟糕的技术）。他从几种编程语言中借鉴了一些东西：Java（语法，原始值与对象），Scheme 和 AWK（一级函数），Self（原型继承），以及 Perl 和 Python（字符串，数组和正则表达式）。

JavaScript 在 ECMAScript 3 之前没有异常处理，这就解释了为什么语言经常自动转换值并经常悄悄失败：最初它无法抛出异常。

一方面，JavaScript 有一些怪癖，缺少相当多的功能（块作用域变量，模块，支持子类等）。另一方面，它有几个强大的功能，可以让你解决这些问题。在其他语言中，你学习语言特性。在 JavaScript 中，你经常学习模式而不是语言特性。

鉴于它的影响，毫不奇怪 JavaScript 可以实现一种混合了函数式编程（高阶函数；内置的`map`，`reduce`等）和面向对象编程（对象，继承）的编程风格。

## 语法

本节解释了 JavaScript 的基本语法原则。

### 语法概述

一些语法的例子：

```js
// Two slashes start single-line comments

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

+   单个等号（`=`）用于将值赋给变量。

+   三个等号（`===`）用于比较两个值（参见[相等运算符](ch01.html#basic_equality_operators "相等运算符")）。

### 语句与表达式

要理解 JavaScript 的语法，你应该知道它有两个主要的语法类别：语句和表达式：

+   语句“做事情”。程序是一系列语句。这是一个语句的例子，它声明（创建）一个变量`foo`：

    ```js
    var foo;
    ```

+   表达式产生值。它们是函数参数，赋值的右侧等等。这是一个表达式的例子：

    ```js
    3 * 7
    ```

语句和表达式之间的区别最好通过 JavaScript 有两种不同的`if-then-else`的方式来说明——作为语句：

```js
var x;
if (y >= 0) {
    x = y;
} else {
    x = -y;
}
```

或作为一个表达式：

```js
var x = y >= 0 ? y : -y;
```

你可以将后者用作函数参数（但不能使用前者）：

```js
myFunction(y >= 0 ? y : -y)
```

最后，无论 JavaScript 在哪里期望一个语句，你也可以使用一个表达式；例如：

```js
foo(7, 1);
```

整行是一个语句（所谓的*表达式语句*），但函数调用`foo(7, 1)`是一个表达式。

### 分号

在 JavaScript 中，分号是可选的。但是，我建议始终包括它们，因为否则 JavaScript 可能会错误猜测语句的结束。详细信息请参见[自动分号插入](ch07.html#automatic_semicolon_insertion "自动分号插入")。

分号终止语句，但不终止块。有一种情况下，您会在块后看到一个分号：函数表达式是以块结尾的表达式。如果这样的表达式出现在语句的最后，它后面会跟着一个分号：

```js
// Pattern: var _ = ___;
var x = 3 * 7;
var f = function () { };  // function expr. inside var decl.
```

### 注释

JavaScript 有两种注释：单行注释和多行注释。单行注释以`//`开头，并在行尾终止：

```js
x++; // single-line comment
```

多行注释由`/*`和`*/`界定：

```js
/* This is
 a multiline
 comment.
 */
```

## 变量和赋值

在 JavaScript 中，变量在使用之前被声明：

```js
var foo;  // declare variable `foo`
```

### 赋值

您可以声明一个变量并同时赋值：

```js
var foo = 6;
```

您也可以给现有变量赋值：

```js
foo = 4;  // change variable `foo`
```

### 复合赋值运算符

有复合赋值运算符，比如`+=`。以下两个赋值是等价的：

```js
x += 1;
x = x + 1;
```

### 标识符和变量名

*标识符*是在 JavaScript 中扮演各种语法角色的名称。例如，变量的名称是标识符。标识符区分大小写。

大致而言，标识符的第一个字符可以是任何 Unicode 字母、美元符号（`$`）或下划线（`_`）。随后的字符还可以是任何 Unicode 数字。因此，以下都是合法的标识符：

```js
arg0
_tmp
$elem
π
```

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

最后，您还应该避免使用标准全局变量的名称（参见第二十三章）。您可以将它们用于局部变量而不会破坏任何东西，但您的代码仍然会变得混乱。

## 值

JavaScript 有许多我们从编程语言中期望的值：布尔值、数字、字符串、数组等等。JavaScript 中的所有值都有*属性*。每个属性都有一个*键*（或*名称*）和一个*值*。您可以将属性视为记录的字段。您可以使用点（`.`）运算符来读取属性：

```js
value.propKey
```

例如，字符串`'abc'`具有属性`length`：

```js
> var str = 'abc';
> str.length
3
```

前面的也可以写成：

```js
> 'abc'.length
3
```

点运算符也用于给属性赋值：

```js
> var obj = {};  // empty object
> obj.foo = 123; // create property `foo`, set it to 123
123
> obj.foo
123
```

您也可以用它来调用方法：

```js
> 'hello'.toUpperCase()
'HELLO'
```

在上面的例子中，我们已经在值`'hello'`上调用了方法`toUpperCase()`。

### 原始值与对象

JavaScript 在值之间做了一个相当武断的区分：

+   *原始值*是布尔值、数字、字符串、`null`和`undefined`。

+   所有其他值都是*对象*。

两者之间的一个主要区别是它们的比较方式；每个对象都有唯一的标识，并且只有（严格）等于自身：

```js
> var obj1 = {};  // an empty object
> var obj2 = {};  // another empty object
> obj1 === obj2
false
> obj1 === obj1
true
```

相反，所有编码相同值的原始值都被视为相同：

```js
> var prim1 = 123;
> var prim2 = 123;
> prim1 === prim2
true
```

接下来的两节将更详细地解释原始值和对象。

### 原始值

以下是所有原始值（或简称*原始值*）：

+   布尔值：`true`，`false`（参见[布尔值](ch01.html#basic_booleans "布尔值")）

+   数字：`1736`，`1.351`（参见[数字](ch01.html#basic_numbers "数字")）

+   字符串：`'abc'`，`"abc"`（参见[字符串](ch01.html#basic_strings "字符串")）

+   两个“非值”：`undefined`，`null`（参见[undefined 和 null](ch01.html#basic_undefined_null "undefined 和 null")）

原始值具有以下特征：

按值比较

“内容”进行比较：

```js
> 3 === 3
true
> 'abc' === 'abc'
true
```

始终不可变

属性无法更改，添加或删除：

```js
> var str = 'abc';

> str.length = 1; // try to change property `length`
> str.length      // ⇒ no effect
3

> str.foo = 3; // try to create property `foo`
> str.foo      // ⇒ no effect, unknown property
undefined
```

（读取未知属性始终返回`undefined`。）

### 对象

所有非原始值都是*对象*。最常见的对象类型是：

+   *普通对象*，可以通过*对象字面量*创建（参见[单个对象](ch01.html#basic_single_objects "单个对象")）：

    ```js
    {
        firstName: 'Jane',
        lastName: 'Doe'
    }
    ```

前面的对象有两个属性：属性`firstName`的值为`'Jane'`，属性`lastName`的值为`'Doe'`。

+   *数组*，可以通过*数组字面量*创建（参见[数组](ch01.html#basic_arrays "数组")）：

    ```js
    [ 'apple', 'banana', 'cherry' ]
    ```

前面的数组有三个元素，可以通过数字索引访问。例如，'apple'的索引是 0。

+   *正则表达式*，可以通过*正则表达式字面量*创建（参见[正则表达式](ch01.html#basic_regexps "正则表达式")）：

    ```js
    /^a+b+$/
    ```

对象具有以下特征：

按引用比较

进行身份比较；每个值都有自己的身份：

```js
> {} === {}  // two different empty objects
false

> var obj1 = {};
> var obj2 = obj1;
> obj1 === obj2
true
```

默认可变

通常可以自由更改，添加和删除属性（参见[单个对象](ch01.html#basic_single_objects "单个对象")）：

```js
> var obj = {};
> obj.foo = 123; // add property `foo`
> obj.foo
123
```

### undefined 和 null

大多数编程语言都有表示缺少信息的值。JavaScript 有两个这样的“非值”，`undefined`和`null`：

+   `undefined`表示“没有值”。未初始化的变量是`undefined`：

    ```js
    > var foo;
    > foo
    undefined
    ```

缺少参数是`undefined`：

```js
> function f(x) { return x }
> f()
undefined
```

如果读取不存在的属性，将得到`undefined`：

```js
> var obj = {}; // empty object
> obj.foo
undefined
```

+   `null`表示“没有对象”。每当期望对象时（参数，对象链中的最后一个等），它被用作非值。

### 警告

`undefined`和`null`没有属性，甚至没有标准方法，如`toString()`。

#### 检查 undefined 或 null

函数通常允许您通过`undefined`或`null`指示缺少值。您可以通过显式检查来做相同的事情：

```js
if (x === undefined || x === null) {
    ...
}
```

您还可以利用`undefined`和`null`都被视为`false`的事实：

```js
if (!x) {
    ...
}
```

### 警告

`false`，`0`，`NaN`和`''`也被视为`false`（参见[真值和假值](ch01.html#basic_truthy_falsy "真值和假值")）。

### 使用 typeof 和 instanceof 对值进行分类

有两个用于对值进行分类的运算符：`typeof`主要用于原始值，而`instanceof`用于对象。

`typeof`看起来像这样：

```js
typeof value
```

它返回描述`value`“类型”的字符串。以下是一些示例：

```js
> typeof true
'boolean'
> typeof 'abc'
'string'
> typeof {} // empty object literal
'object'
> typeof [] // empty array literal
'object'
```

以下表列出了`typeof`的所有结果：

| 操作数 | 结果 |
| --- | --- |
| `undefined` | `'undefined'` |
| `null` | `'object'` |
| 布尔值 | `'boolean'` |
| 数字值 | `'number'` |
| 字符串值 | `'string'` |
| 函数 | `'function'` |
| 所有其他正常值 | `'object'` |
|（引擎创建的值）| JavaScript 引擎允许创建值，其`typeof`返回任意字符串（与此表中列出的所有结果都不同）。 |

`typeof null`返回`'object'`是一个无法修复的错误，因为这会破坏现有的代码。这并不意味着`null`是一个对象。

`instanceof`看起来像这样：

```js
value instanceof Constr
```

如果`value`是由构造函数`Constr`创建的对象，则返回`true`（参见[构造函数：对象的工厂](ch01.html#basic_constructors "构造函数：对象的工厂")）。以下是一些示例：

```js
> var b = new Bar();  // object created by constructor Bar
> b instanceof Bar
true

> {} instanceof Object
true
> [] instanceof Array
true
> [] instanceof Object  // Array is a subconstructor of Object
true

> undefined instanceof Object
false
> null instanceof Object
false
```

## 布尔值

原始布尔类型包括值`true`和`false`。以下运算符产生布尔值：

+   二进制逻辑运算符：`&&`（与），`||`（或）

+   前缀逻辑运算符：`!`（非）

+   比较运算符：

+   相等运算符：`===`，`!==`，`==`，`!=`

+   排序运算符（用于字符串和数字）：`>`, `>=`, `<`, `<=`

### 真值和假值

每当 JavaScript 期望布尔值（例如`if`语句的条件）时，可以使用任何值。它将被解释为`true`或`false`。以下值被解释为`false`：

+   `undefined`，`null`

+   布尔值：`false`

+   数字：`-0`，`NaN`

+   字符串：`''`

所有其他值（包括所有对象！）都被认为是`true`。被解释为`false`的值称为*假值*，被解释为`true`的值称为*真值*。`Boolean()`作为函数调用，将其参数转换为布尔值。您可以使用它来测试值的解释方式：

```js
> Boolean(undefined)
false
> Boolean(0)
false
> Boolean(3)
true
> Boolean({}) // empty object
true
> Boolean([]) // empty array
true
```

### 二进制逻辑运算符

JavaScript 中的二进制逻辑运算符是*短路*的。也就是说，如果第一个操作数足以确定结果，第二个操作数将不会被评估。例如，在以下表达式中，函数`foo()`永远不会被调用：

```js
false && foo()
true  || foo()
```

此外，二进制逻辑运算符返回它们的操作数之一，这些操作数可能是布尔值也可能不是。使用真值检查来确定哪一个：

和(`&&`)

如果第一个操作数为假值，则返回它。否则，返回第二个操作数：

```js
> NaN && 'abc'
NaN
> 123 && 'abc'
'abc'
```

或(`||`)

如果第一个操作数为真值，则返回它。否则，返回第二个操作数：

```js
> 'abc' || 123
'abc'
> '' || 123
123
```

### 相等运算符

JavaScript 有两种相等性：

+   普通，或“宽松”，（不）相等：`==`和`!=`

+   严格（不）相等：`===`和`!==`

普通相等性认为太多的值是相等的（详细内容在[普通（宽松）相等性（==，！=）](ch09.html#normal_equality "Normal (Lenient) Equality (==, !=)")中有解释），这可能会隐藏错误。因此，建议始终使用严格相等性。

## 数字

JavaScript 中的所有数字都是浮点数：

```js
> 1 === 1.0
true
```

特殊数字包括以下内容：

`NaN`（“不是一个数字”）

一个错误值：

```js
> Number('xyz')  // 'xyz' can’t be converted to a number
NaN
```

`Infinity`

也是大多数错误值：

```js
> 3 / 0
Infinity
> Math.pow(2, 1024)  // number too large
Infinity
```

`Infinity`大于任何其他数字（除了`NaN`）。同样，`-Infinity`小于任何其他数字（除了`NaN`）。这使得这些数字在作为默认值时非常有用（例如，当你正在寻找最小值或最大值时）。

## 运算符

JavaScript 有以下算术运算符（参见[算术运算符](ch11.html#arithmetic_operators "Arithmetic Operators")）：

+   加法：`number1 + number2`

+   减法：`number1 - number2`

+   乘法：`number1 * number2`

+   除法：`number1 / number2`

+   余数：`number1 % number2`

+   增量：`++variable`, `variable++`

+   递减：`--variable`, `variable--`

+   否定：`-value`

+   转换为数字：`+value`

全局对象`Math`（参见[Math](ch01.html#basic_math "Math")）通过函数提供更多的算术运算。

JavaScript 还有位操作的运算符（例如，位与；参见[位运算符](ch11.html#bitwise_operators "Bitwise Operators")）。

## 字符串

字符串可以直接通过字符串字面量创建。这些字面量由单引号或双引号括起来。反斜杠(`\`)转义字符并产生一些控制字符。以下是一些例子：

```js
'abc'
"abc"

'Did she say "Hello"?'
"Did she say \"Hello\"?"

'That\'s nice!'
"That's nice!"

'Line 1\nLine 2'  // newline
'Backlash: \\'
```

单个字符通过方括号访问：

```js
> var str = 'abc';
> str[1]
'b'
```

属性`length`计算字符串中的字符数：

```js
> 'abc'.length
3
```

与所有原始值一样，字符串是不可变的；如果要更改现有字符串，需要创建一个新字符串。

### 字符串运算符

字符串通过加号(`+`)操作符进行连接，如果其中一个操作数是字符串，则将另一个操作数转换为字符串：

```js
> var messageCount = 3;
> 'You have ' + messageCount + ' messages'
'You have 3 messages'
```

要在多个步骤中连接字符串，使用`+=`操作符：

```js
> var str = '';
> str += 'Multiple ';
> str += 'pieces ';
> str += 'are concatenated.';
> str
'Multiple pieces are concatenated.'
```

### 字符串方法

字符串有许多有用的方法（参见[字符串原型方法](ch12.html#string_prototype_methods "String Prototype Methods")）。以下是一些例子：

```js
> 'abc'.slice(1)  // copy a substring
'bc'
> 'abc'.slice(1, 2)
'b'

> '\t xyz  '.trim()  // trim whitespace
'xyz'

> 'mjölnir'.toUpperCase()
'MJÖLNIR'

> 'abc'.indexOf('b')  // find a string
1
> 'abc'.indexOf('x')
-1
```

## 语句

JavaScript 中的条件和循环在以下部分介绍。

### 条件

`if`语句有一个`then`子句和一个可选的`else`子句，根据布尔条件执行：

```js
if (myvar === 0) {
    // then
}

if (myvar === 0) {
    // then
} else {
    // else
}

if (myvar === 0) {
    // then
} else if (myvar === 1) {
    // else-if
} else if (myvar === 2) {
    // else-if
} else {
    // else
}
```

我建议始终使用大括号（它们表示零个或多个语句的块）。但如果一个子句只是一个语句，你不必这样做（对于控制流语句`for`和`while`也是如此）：

```js
if (x < 0) return -x;
```

以下是一个`switch`语句。`fruit`的值决定执行哪个`case`：

```js
switch (fruit) {
    case 'banana':
        // ...
        break;
    case 'apple':
        // ...
        break;
    default:  // all other cases
        // ...
}
```

`case`后的“操作数”可以是任何表达式；它通过`===`与`switch`的参数进行比较。

### 循环

`for`循环的格式如下：

```js
for (⟦«init»⟧; ⟦«condition»⟧; ⟦«post_iteration»⟧)
    «statement»
```

`init`在循环开始时执行。在每次循环迭代之前检查`condition`；如果变为`false`，则终止循环。`post_iteration`在每次循环迭代后执行。

此示例在控制台上打印数组`arr`的所有元素：

```js
for (var i=0; i < arr.length; i++) {
    console.log(arr[i]);
}
```

`while`循环在其条件成立时继续循环其主体：

```js
// Same as for loop above:
var i = 0;
while (i < arr.length) {
    console.log(arr[i]);
    i++;
}
```

`do-while`循环在其条件成立时继续循环其主体。由于条件跟随主体，因此主体始终至少执行一次：

```js
do {
    // ...
} while (condition);
```

在所有循环中：

+   `break`离开循环。

+   `continue`开始新的循环迭代。

## 函数

定义函数的一种方式是通过*函数声明*：

```js
function add(param1, param2) {
    return param1 + param2;
}
```

前面的代码定义了一个函数`add`，它有两个参数`param1`和`param2`，并返回这两个参数的总和。这是如何调用该函数的：

```js
> add(6, 1)
7
> add('a', 'b')
'ab'
```

定义`add()`的另一种方式是通过将*函数表达式*分配给变量`add`：

```js
var add = function (param1, param2) {
    return param1 + param2;
};
```

函数表达式产生一个值，因此可以直接用于将函数作为参数传递给其他函数：

```js
someOtherFunction(function (p1, p2) { ... });
```

### 函数声明被提升

函数声明是*提升*的-完整地移动到当前范围的开头。这允许您引用稍后声明的函数：

```js
function foo() {
    bar();  // OK, bar is hoisted
    function bar() {
        ...
    }
}
```

请注意，虽然`var`声明也被提升（参见[变量被提升](ch01.html#basic_var_hoisting "变量被提升")），但是它们执行的赋值不会：

```js
function foo() {
    bar();  // Not OK, bar is still undefined
    var bar = function () {
        // ...
    };
}
```

### 特殊变量参数

您可以使用任意数量的参数调用 JavaScript 中的任何函数；语言永远不会抱怨。但是，它将使所有参数通过特殊变量`arguments`可用。`arguments`看起来像一个数组，但没有数组方法：

```js
> function f() { return arguments }
> var args = f('a', 'b', 'c');
> args.length
3
> args[0]  // read element at index 0
'a'
```

### 参数太多或太少

让我们使用以下函数来探索 JavaScript 中如何处理太多或太少的参数（函数`toArray()`显示在[将参数转换为数组](ch01.html#basic_toarray "将参数转换为数组")中）：

```js
function f(x, y) {
    console.log(x, y);
    return toArray(arguments);
}
```

将忽略额外的参数（除了`arguments`）：

```js
> f('a', 'b', 'c')
a b
[ 'a', 'b', 'c' ]
```

缺少参数将获得值`undefined`：

```js
> f('a')
a undefined
[ 'a' ]
> f()
undefined undefined
[]
```

### 可选参数

以下是为参数分配默认值的常见模式：

```js
function pair(x, y) {
    x = x || 0;  // (1)
    y = y || 0;
    return [ x, y ];
}
```

在第（1）行，`||`运算符返回`x`，如果它是真值（不是`null`，`undefined`等）。否则，它将返回第二个操作数：

```js
> pair()
[ 0, 0 ]
> pair(3)
[ 3, 0 ]
> pair(3, 5)
[ 3, 5 ]
```

### 强制参数个数

如果要强制执行*arity*（特定数量的参数），可以检查`arguments.length`：

```js
function pair(x, y) {
    if (arguments.length !== 2) {
        throw new Error('Need exactly 2 arguments');
    }
    ...
}
```

### 将参数转换为数组

`arguments`不是数组，它只是*类似数组*（参见[类似数组对象和通用方法](ch17_split_001.html#array-like_objects "类似数组对象和通用方法")）。它有一个`length`属性，您可以通过方括号中的索引访问其元素。但是，您无法删除元素或调用其中任何数组方法。因此，有时需要将`arguments`转换为数组，这就是以下函数所做的事情（它在[类似数组对象和通用方法](ch17_split_001.html#array-like_objects "类似数组对象和通用方法")中有解释）：

```js
function toArray(arrayLikeObject) {
    return Array.prototype.slice.call(arrayLikeObject);
}
```

## 异常处理

处理异常的最常见方法（参见第十四章）如下：

```js
function getPerson(id) {
    if (id < 0) {
        throw new Error('ID must not be negative: '+id);
    }
    return { id: id }; // normally: retrieved from database
}

function getPersons(ids) {
    var result = [];
    ids.forEach(function (id) {
        try {
            var person = getPerson(id);
            result.push(person);
        } catch (exception) {
            console.log(exception);
        }
    });
    return result;
}
```

`try`子句包围关键代码，如果在`try`子句内抛出异常，则执行`catch`子句。使用前面的代码：

```js
> getPersons([2, -5, 137])
[Error: ID must not be negative: -5]
[ { id: 2 }, { id: 137 } ]
```

## 严格模式

严格模式（参见[严格模式](ch07.html#strict_mode "严格模式")）启用更多警告，并使 JavaScript 成为一种更干净的语言（非严格模式有时被称为“松散模式”）。要打开它，请首先在 JavaScript 文件或`<script>`标记中键入以下行：

```js
'use strict';
```

您还可以为每个函数启用严格模式：

```js
function functionInStrictMode() {
    'use strict';
}
```

## 变量作用域和闭包

在 JavaScript 中，你在使用变量之前通过`var`声明变量：

```js
> var x;
> x = 3;
> y = 4;
ReferenceError: y is not defined
```

你可以使用单个`var`语句声明和初始化多个变量：

```js
var x = 1, y = 2, z = 3;
```

但我建议每个变量使用一个语句（原因在[Syntax](ch26.html#style_one_var_per_line "Syntax")中有解释）。因此，我会重写上一个语句为：

```js
var x = 1;
var y = 2;
var z = 3;
```

由于提升（参见[变量被提升](ch01.html#basic_var_hoisting "变量被提升")），通常最好在函数的开头声明变量。

### 变量是函数作用域的

变量的作用域总是整个函数（而不是当前的块）。例如：

```js
function foo() {
    var x = -512;
    if (x < 0) {  // (1)
        var tmp = -x;
        ...
    }
    console.log(tmp);  // 512
}
```

我们可以看到变量`tmp`不仅限于从第（1）行开始的块；它存在直到函数的结束。

### 变量被提升

每个变量声明都是*提升*的：声明被移动到函数的开头，但它所做的赋值保持不变。例如，考虑下面函数中第（1）行的变量声明：

```js
function foo() {
    console.log(tmp); // undefined
    if (false) {
        var tmp = 3;  // (1)
    }
}
```

在内部，前面的函数是这样执行的：

```js
function foo() {
    var tmp;  // hoisted declaration
    console.log(tmp);
    if (false) {
        tmp = 3;  // assignment stays put
    }
}
```

### 闭包

每个函数都与包围它的函数的变量保持连接，即使它离开了被创建的作用域。例如：

```js
function createIncrementor(start) {
    return function () {  // (1)
        start++;
        return start;
    }
}
```

从第（1）行开始的函数离开了它被创建的上下文，但仍然连接到`start`的一个活动版本：

```js
> var inc = createIncrementor(5);
> inc()
6
> inc()
7
> inc()
8
```

*闭包*是一个函数加上与其周围作用域的变量的连接。因此，`createIncrementor()`返回的是一个闭包。

### IIFE 模式：引入新的作用域

有时你想引入一个新的变量作用域——例如，防止一个变量成为全局的。在 JavaScript 中，你不能使用块来做到这一点；你必须使用一个函数。但是有一种使用函数的块状方式的模式。它被称为*IIFE*（[立即调用函数表达式](http://bit.ly/i-ife)，发音为“iffy”）：

```js
(function () {  // open IIFE
    var tmp = ...;  // not a global variable
}());  // close IIFE
```

确保按照前面的示例精确地输入（除了注释）。IIFE 是一个在定义后立即调用的函数表达式。在函数内部，存在一个新的作用域，防止`tmp`成为全局的。请参阅[IIFE 引入新的作用域](ch16.html#iife "IIFE 引入新的作用域")了解 IIFE 的详细信息。

#### IIFE 的用例：通过闭包无意中共享

闭包保持与外部变量的连接，有时这并不是你想要的：

```js
var result = [];
for (var i=0; i < 5; i++) {
    result.push(function () { return i });  // (1)
}
console.log(result1; // 5 (not 1)
console.log(result3; // 5 (not 3)
```

第（1）行返回的值始终是`i`的当前值，而不是函数创建时的值。循环结束后，`i`的值为 5，这就是为什么数组中的所有函数都返回该值。如果你想让第（1）行的函数接收当前`i`值的快照，你可以使用 IIFE：

```js
for (var i=0; i < 5; i++) {
    (function () {
        var i2 = i; // copy current i
        result.push(function () { return i2 });
    }());
}
```

## 对象和构造函数

本节涵盖了 JavaScript 的两种基本面向对象的机制：单个对象和*构造函数*（它们是对象的工厂，类似于其他语言中的类）。

### 单个对象

像所有的值一样，对象都有属性。实际上，你可以把对象看作是一组属性，其中每个属性都是一个（键，值）对。键是一个字符串，值是任何 JavaScript 值。

在 JavaScript 中，你可以直接通过*对象字面量*创建普通对象：

```js
'use strict';
var jane = {
    name: 'Jane',

    describe: function () {
        return 'Person named '+this.name;
    }
};
```

前面的对象有属性`name`和`describe`。你可以读取（“获取”）和写入（“设置”）属性：

```js
> jane.name  // get
'Jane'
> jane.name = 'John';  // set
> jane.newProperty = 'abc';  // property created automatically
```

像`describe`这样的函数值属性被称为*方法*。它们使用`this`来引用调用它们的对象：

```js
> jane.describe()  // call method
'Person named John'
> jane.name = 'Jane';
> jane.describe()
'Person named Jane'
```

`in`运算符检查一个属性是否存在：

```js
> 'newProperty' in jane
true
> 'foo' in jane
false
```

如果读取一个不存在的属性，你会得到值`undefined`。因此，前面的两个检查也可以这样执行：

```js
> jane.newProperty !== undefined
true
> jane.foo !== undefined
false
```

`delete`运算符移除一个属性：

```js
> delete jane.newProperty
true
> 'newProperty' in jane
false
```

### 任意属性键

属性键可以是任何字符串。到目前为止，我们已经在对象文字和点运算符之后看到了属性键。但是，只有在它们是标识符时，才能以这种方式使用它们（参见[Identifiers and Variable Names](ch01.html#basic_identifiers_variable_names "Identifiers and Variable Names")）。如果要使用其他字符串作为键，必须在对象文字中对其进行引用，并使用方括号来获取和设置属性：

```js
> var obj = { 'not an identifier': 123 };
> obj['not an identifier']
123
> obj['not an identifier'] = 456;
```

方括号还允许您计算属性的键：

```js
> var obj = { hello: 'world' };
> var x = 'hello';

> obj[x]
'world'
> obj['hel'+'lo']
'world'
```

### 提取方法

如果提取一个方法，它将失去与对象的连接。单独使用时，该函数不再是一个方法，`this` 的值为 `undefined`（在严格模式下）。

例如，让我们回到之前的对象 `jane`：

```js
'use strict';
var jane = {
    name: 'Jane',

    describe: function () {
        return 'Person named '+this.name;
    }
};
```

我们想从 `jane` 中提取方法 `describe`，将其放入变量 `func` 中，并调用它。但是，这样做不起作用：

```js
> var func = jane.describe;
> func()
TypeError: Cannot read property 'name' of undefined
```

解决方案是使用所有函数都具有的 `bind()` 方法。它创建一个新函数，其 `this` 始终具有给定值：

```js
> var func2 = jane.describe.bind(jane);
> func2()
'Person named Jane'
```

### 方法内的函数

每个函数都有自己的特殊变量 `this`。如果在方法内部嵌套函数，这是不方便的，因为您无法从函数中访问方法的 `this`。以下是一个示例，我们调用 `forEach` 以使用函数遍历数组：

```js
var jane = {
    name: 'Jane',
    friends: [ 'Tarzan', 'Cheeta' ],
    logHiToFriends: function () {
        'use strict';
        this.friends.forEach(function (friend) {
            // `this` is undefined here
            console.log(this.name+' says hi to '+friend);
        });
    }
}
```

调用 `logHiToFriends` 会产生一个错误：

```js
> jane.logHiToFriends()
TypeError: Cannot read property 'name' of undefined
```

让我们看看修复这个问题的两种方法。首先，我们可以将 `this` 存储在不同的变量中：

```js
logHiToFriends: function () {
    'use strict';
    var that = this;
    this.friends.forEach(function (friend) {
        console.log(that.name+' says hi to '+friend);
    });
}
```

或者，`forEach` 有一个第二个参数，允许您为 `this` 提供一个值：

```js
logHiToFriends: function () {
    'use strict';
    this.friends.forEach(function (friend) {
        console.log(this.name+' says hi to '+friend);
    }, this);
}
```

在 JavaScript 中，函数表达式经常用作函数调用中的参数。当您从这些函数表达式之一引用 `this` 时，一定要小心。

### 构造函数：对象的工厂

到目前为止，您可能认为 JavaScript 对象 *只* 是从字符串到值的映射，这是 JavaScript 对象文字所暗示的概念，它看起来像其他语言的映射/字典文字。但是，JavaScript 对象还支持一项真正面向对象的功能：继承。本节并未完全解释 JavaScript 继承的工作原理，但它向您展示了一个简单的模式，以便您开始。如果您想了解更多，请参阅第十七章。

除了作为“真正的”函数和方法外，函数在 JavaScript 中还扮演另一个角色：如果通过 `new` 运算符调用，它们将成为 *构造函数*——对象的工厂。因此，构造函数在其他语言中是类的粗略类比。按照惯例，构造函数的名称以大写字母开头。例如：

```js
// Set up instance data
function Point(x, y) {
    this.x = x;
    this.y = y;
}
// Methods
Point.prototype.dist = function () {
    return Math.sqrt(this.x*this.x + this.y*this.y);
};
```

我们可以看到构造函数有两个部分。首先，函数 `Point` 设置实例数据。其次，属性 `Point.prototype` 包含一个具有方法的对象。前者数据对每个实例都是特定的，而后者数据在所有实例之间共享。

要使用 `Point`，我们通过 `new` 运算符调用它：

```js
> var p = new Point(3, 5);
> p.x
3
> p.dist()
5.830951894845301
```

`p` 是 `Point` 的一个实例：

```js
> p instanceof Point
true
```

## 数组

数组是可以通过从零开始的整数索引访问的元素序列。

### 数组文字

数组文字对于创建数组很方便：

```js
> var arr = [ 'a', 'b', 'c' ];
```

前面的数组有三个元素：字符串 `'a'`、`'b'` 和 `'c'`。您可以通过整数索引访问它们：

```js
> arr[0]
'a'
> arr[0] = 'x';
> arr
[ 'x', 'b', 'c' ]
```

`length` 属性指示数组有多少个元素。您可以使用它来追加元素和删除元素：

```js
> var arr = ['a', 'b'];
> arr.length
2

> arr[arr.length] = 'c';
> arr
[ 'a', 'b', 'c' ]
> arr.length
3

> arr.length = 1;
> arr
[ 'a' ]
```

`in` 运算符也适用于数组：

```js
> var arr = [ 'a', 'b', 'c' ];
> 1 in arr // is there an element at index 1?
true
> 5 in arr // is there an element at index 5?
false
```

请注意，数组是对象，因此可以具有对象属性：

```js
> var arr = [];
> arr.foo = 123;
> arr.foo
123
```

### 数组方法

数组有许多方法（参见[Array Prototype Methods](ch18.html#array_prototype_methods "Array Prototype Methods")）。以下是一些示例：

```js
> var arr = [ 'a', 'b', 'c' ];

> arr.slice(1, 2)  // copy elements
[ 'b' ]
> arr.slice(1)
[ 'b', 'c' ]

> arr.push('x')  // append an element
4
> arr
[ 'a', 'b', 'c', 'x' ]

> arr.pop()  // remove last element
'x'
> arr
[ 'a', 'b', 'c' ]

> arr.shift()  // remove first element
'a'
> arr
[ 'b', 'c' ]

> arr.unshift('x')  // prepend an element
3
> arr
[ 'x', 'b', 'c' ]

> arr.indexOf('b')  // find the index of an element
1
> arr.indexOf('y')
-1

> arr.join('-')  // all elements in a single string
'x-b-c'
> arr.join('')
'xbc'
> arr.join()
'x,b,c'
```

### 遍历数组

有几种用于遍历元素的数组方法（参见[Iteration (Nondestructive)](ch18.html#array_iteration_methods "Iteration (Nondestructive)")）。最重要的两个是 `forEach` 和 `map`。

`forEach` 遍历数组并将当前元素及其索引传递给函数：

```js
[ 'a', 'b', 'c' ].forEach(
    function (elem, index) {  // (1)
        console.log(index + '. ' + elem);
    });
```

前面的代码产生以下输出：

```js
0\. a
1\. b
2\. c
```

请注意，第（1）行中的函数可以忽略参数。例如，它可能只有参数`elem`。

`map`通过将函数应用于现有数组的每个元素来创建一个新数组：

```js
> [1,2,3].map(function (x) { return x*x })
[ 1, 4, 9 ]
```

## 正则表达式

JavaScript 内置支持正则表达式（第十九章是教程，更详细地解释了它们的工作原理）。它们由斜杠分隔：

```js
/^abc$/
/[A-Za-z0-9]+/
```

### 方法 test(): 是否有匹配项？

```js
> /^a+b+$/.test('aaab')
true
> /^a+b+$/.test('aaa')
false
```

### 方法 exec(): 匹配和捕获组

```js
> /a(b+)a/.exec('_abbba_aba_')
[ 'abbba', 'bbb' ]
```

返回的数组包含索引 0 处的完整匹配项，索引 1 处的第一个组的捕获，依此类推。还有一种方法（在[RegExp.prototype.exec: Capture Groups](ch19.html#RegExp.prototype.exec "RegExp.prototype.exec: Capture Groups")中讨论）可以重复调用此方法以获取所有匹配项。

### 方法 replace(): 搜索和替换

```js
> '<a> <bbb>'.replace(/<(.*?)>/g, '[$1]')
'[a] [bbb]'
```

`replace`的第一个参数必须是带有`/g`标志的正则表达式；否则，只会替换第一个匹配项。还有一种方法（如在[String.prototype.replace: Search and Replace](ch19.html#String.prototype.replace "String.prototype.replace: Search and Replace")中讨论的）可以使用函数来计算替换。

## 数学

`Math`（参见第二十一章）是一个具有算术函数的对象。以下是一些示例：

```js
> Math.abs(-2)
2

> Math.pow(3, 2)  // 3 to the power of 2
9

> Math.max(2, -1, 5)
5

> Math.round(1.9)
2

> Math.PI  // pre-defined constant for π
3.141592653589793

> Math.cos(Math.PI)  // compute the cosine for 180°
-1
```

## 标准库的其他功能

JavaScript 的标准库相对简陋，但还有更多可以使用的东西：

`Date`（第二十章）

一个日期的构造函数，其主要功能是解析和创建日期字符串以及访问日期的组件（年、小时等）。

`JSON`（第二十二章）

一个具有解析和生成 JSON 数据功能的对象。

`console.*` 方法（参见[控制台 API](ch23.html#console_api "控制台 API")）

这些特定于浏览器的方法不是语言本身的一部分，但其中一些也适用于 Node.js。
