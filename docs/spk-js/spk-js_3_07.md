# 第十三章：语句

> 原文：[13. Statements](https://exploringjs.com/es5/ch13.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章涵盖了 JavaScript 的语句：变量声明、循环、条件语句等。

## 声明和赋值变量

`var`用于*声明*一个变量，它创建变量并使您能够使用它。等号（`=`）用于给它赋值：

```js
var foo;
foo = 'abc';
```

`var`还允许您将前面的两个语句合并为一个：

```js
var foo = 'abc';
```

最后，您还可以将多个`var`语句合并为一个：

```js
var x, y=123, z;
```

了解有关变量如何工作的更多信息，请阅读第十六章。

## 循环和条件语句的主体

复合语句，如循环和条件语句，嵌入了一个或多个“主体”——例如，`while`循环：

```js
while («condition»)
    «statement»
```

对于`«statement»`主体，您有选择。您可以使用单个语句：

```js
while (x >= 0) x--;
```

或者您可以使用一个块（它算作一个单独的语句）：

```js
while (x > 0) {
    x--;
}
```

如果要使主体包含多个语句，您需要使用一个块。除非完整的复合语句可以写在一行中，否则我建议使用一个块。

## 循环

本节探讨了 JavaScript 的循环语句。

### 与循环一起使用的机制

以下机制可以与所有循环一起使用：

`break ⟦«label»⟧`

退出循环。

`continue ⟦«label»⟧`

停止当前循环迭代，并立即继续下一个。

标签

标签是一个标识符，后面跟着一个冒号。在循环前，标签允许您即使从嵌套在其中的循环中也可以中断或继续该循环。在块的前面，您可以跳出该块。在这两种情况下，标签的名称成为`break`或`continue`的参数。这是一个打破块的例子：

```js
function findEvenNumber(arr) {
    loop: { // label
        for (var i=0; i<arr.length; i++) {
            var elem = arr[i];
            if ((elem % 2) === 0) {
                console.log('Found: ' + elem);
                break loop;
            }
        }
        console.log('No even number found.');
    }
    console.log('DONE');
}
```

### 当

一个`while`循环：

```js
while («condition»)
    «statement»
```

只要`condition`成立，就执行`statement`。如果`condition`始终为`true`，则会得到一个无限循环：

```js
while (true) { ... }
```

在以下示例中，我们删除数组的所有元素并将它们记录到控制台：

```js
var arr = [ 'a', 'b', 'c' ];
while (arr.length > 0) {
    console.log(arr.shift());
}
```

这是输出：

```js
a
b
c
```

### do-while

一个`do-while`循环：

```js
do «statement»
while («condition»);
```

至少执行`statement`一次，然后只要`condition`成立。例如：

```js
var line;
do {
    line = prompt('Enter a number:');
} while (!/^[0-9]+$/.test(line));
```

### 为了

在`for`循环中：

```js
for (⟦«init»⟧; ⟦«condition»⟧; ⟦«post_iteration»⟧)
    «statement»
```

`init`在循环之前执行一次，只要`condition`为`true`，循环就会继续。您可以在`init`中使用`var`声明变量，但是这些变量的作用域始终是完整的周围函数。`post_iteration`在循环的每次迭代之后执行。考虑到所有这些，前面的循环等同于以下`while`循环：

```js
«init»;
while («condition») {
    «statement»
    «post_iteration»;
}
```

以下示例是迭代数组的传统方法（其他可能性在[最佳实践：迭代数组](ch18.html#array_iteration "最佳实践：迭代数组")中描述）：

```js
var arr = [ 'a', 'b', 'c' ];
for (var i=0; i<arr.length; i++) {
    console.log(arr[i]);
}
```

如果您省略头部的所有部分，`for`循环将变得无限：

```js
for (;;) {
    ...
}
```

### 对于

一个`for-in`循环：

```js
for («variable» in «object»)
    «statement»
```

遍历`object`的所有属性键，包括继承的属性。但是，标记为不可枚举的属性将被忽略（参见[属性属性和属性描述符](ch17_split_000.html#property_attributes "属性属性和属性描述符")）。以下规则适用于`for-in`循环：

+   您可以使用`var`声明变量，但是这些变量的作用域始终是完整的周围函数。

+   在迭代期间可以删除属性。

#### 最佳实践：不要对数组使用 for-in

不要使用`for-in`来遍历数组。首先，它遍历索引，而不是值：

```js
> var arr = [ 'a', 'b', 'c' ];
> for (var key in arr) { console.log(key); }
0
1
2
```

其次，它还遍历所有（非索引）属性键。以下示例说明了当您向数组添加属性`foo`时会发生什么：

```js
> var arr = [ 'a', 'b', 'c' ];
> arr.foo = true;
> for (var key in arr) { console.log(key); }
0
1
2
foo
```

因此，最好使用普通的`for`循环或数组方法`forEach()`（参见[最佳实践：迭代数组](ch18.html#array_iteration "最佳实践：迭代数组")）。

#### 最佳实践：小心使用对象的 for-in

`for-in`循环遍历*所有*（可枚举）属性，包括继承的属性。这可能不是您想要的。让我们使用以下构造函数来说明问题：

```js
function Person(name) {
    this.name = name;
}
Person.prototype.describe = function () {
    return 'Name: '+this.name;
};
```

`Person`的实例从`Person.prototype`继承了属性`describe`，这是由`for-in`看到的：

```js
var person = new Person('Jane');
for (var key in person) {
    console.log(key);
}
```

这是输出：

```js
name
describe
```

通常，使用`for-in`的最佳方法是通过`hasOwnProperty()`跳过继承的属性：

```js
for (var key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(key);
    }
}
```

这是输出：

```js
name
```

还有一个最后的警告：`person`可能有一个`hasOwnProperty`属性，这将阻止检查起作用。为了安全起见，您必须直接引用通用方法（参见[通用方法：从原型中借用方法](ch17_split_001.html#generic_method "通用方法：从原型中借用方法")）`Object.prototype.hasOwnProperty`：

```js
for (var key in person) {
    if (Object.prototype.hasOwnProperty.call(person, key)) {
        console.log(key);
    }
}
```

还有其他更舒适的方法可以遍历属性键，这些方法在[最佳实践：遍历自有属性](ch17_split_000.html#object_iteration "最佳实践：遍历自有属性")中有描述。

### 对于每个-在

这个循环只存在于 Firefox 上。不要使用它。

## 条件

本节涵盖了 JavaScript 的条件语句。

### if-then-else

在`if-then-else`语句中：

```js
if («condition»)
    «then_branch»
⟦else
    «else_branch»⟧
```

`then_branch`和`else_branch`可以是单个语句或语句块（参见[循环和条件的主体](ch13.html#loops_conditionals_bodies "循环和条件的主体")）。

#### 链接 if 语句

您可以链接几个`if`语句：

```js
if (s1 > s2) {
    return 1;
} else if (s1 < s2) {
    return -1;
} else {
    return 0;
}
```

请注意，在前面的例子中，所有的`else`分支都是单个语句（`if`语句）。只允许`else`分支为块的编程语言需要一些类似`else-if`分支的东西来进行链接。

#### 陷阱：悬空的 else

以下示例的`else`分支被称为“悬空”，因为不清楚它属于两个`if`语句中的哪一个：

```js
if («cond1») if («cond2») «stmt1» else «stmt2»
```

这是一个简单的规则：使用大括号。前面的片段等同于以下代码（在这里很明显`else`属于谁）：

```js
if («cond1») {
    if («cond2») {
        «stmt1»
    } else {
        «stmt2»
    }
}
```

### switch

一个`switch`语句：

```js
switch («expression») {
    case «label1_1»:
    case «label1_2»:
        ...
        «statements1»
        ⟦break;⟧
    case «label2_1»:
    case «label2_2»:
        ...
        «statements2»
        ⟦break;⟧
    ...
    ⟦default:
        «statements_default»
        ⟦break;⟧⟧
}
```

评估`expression`，然后跳转到与结果匹配的`case`子句。如果没有匹配的标签，`switch`会跳转到`default`子句（如果存在）或者不执行任何操作。

`case`后的“操作数”可以是任何表达式；它通过`===`与`switch`的参数进行比较。

如果不使用终止语句结束子句，执行将继续到下一个子句。最常用的终止语句是`break`。但是`return`和`throw`也可以工作，尽管它们通常不仅仅离开`switch`语句。

以下示例说明了如果使用`throw`或`return`，则不需要`break`：

```js
function divide(dividend, divisor) {
    switch (divisor) {
        case 0:
            throw 'Division by zero';
        default:
            return dividend / divisor;
    }
}
```

在这个例子中，没有`default`子句。因此，如果`fruit`不匹配任何`case`标签，则什么也不会发生：

```js
function useFruit(fruit) {
    switch (fruit) {
        case 'apple':
            makeCider();
            break;
        case 'grape':
            makeWine();
            break;
        // neither apple nor grape: do nothing
    }
}
```

在这里，有多个连续的`case`标签：

```js
function categorizeColor(color) {
    var result;
    switch (color) {
        case 'red':
        case 'yellow':
        case 'blue':
            result = 'Primary color: '+color;
            break;
        case 'or':
        case 'green':
        case 'violet':
            result = 'Secondary color: '+color;
            break;
        case 'black':
        case 'white':
            result = 'Not a color';
            break;
        default:
            throw 'Illegal argument: '+color;
    }
    console.log(result);
}
```

这个例子演示了`case`后面的值可以是任意表达式：

```js
function compare(x, y) {
    switch (true) {
        case x < y:
            return -1;
        case x === y:
            return 0;
        default:
            return 1;
    }
}
```

前面的`switch`语句通过遍历`case`子句来寻找其参数`true`的匹配项。如果其中一个`case`表达式求值为`true`，则执行相应的`case`主体。因此，前面的代码等同于以下`if`语句：

```js
function compare(x, y) {
    if (x < y) {
        return -1;
    } else if (x === y) {
        return 0;
    } else {
        return 1;
    }
}
```

通常应该更喜欢后一种解决方案；它更加自解释。

## `with`语句

本节解释了`with`语句在 JavaScript 中的工作原理以及为什么不鼓励使用它。

### 语法和语义

`with`语句的语法如下：

```js
with («object»)
    «statement»
```

它将`object`的属性转换为`statement`的局部变量。例如：

```js
var obj = { first: 'John' };
with (obj) {
    console.log('Hello '+first); // Hello John
}
```

它的预期用途是在多次访问对象时避免冗余。以下是一个带有冗余的代码示例：

```js
foo.bar.baz.bla   = 123;
foo.bar.baz.yadda = 'abc';
```

`with`使这更短：

```js
with (foo.bar.baz) {
    bla   = 123;
    yadda = 'abc';
}
```

### `with`语句已被弃用

通常不鼓励使用`with`语句（下一节解释了原因）。例如，在严格模式下是禁止的：

```js
> function foo() { 'use strict'; with ({}); }
SyntaxError: strict mode code may not contain 'with' statements
```

#### 避免使用`with`语句的技巧

避免这样的代码：

```js
// Don't do this:
with (foo.bar.baz) {
    console.log('Hello '+first+' '+last);
}
```

而是使用一个短名称的临时变量：

```js
var b = foo.bar.baz;
console.log('Hello '+b.first+' '+b.last);
```

如果您不想将临时变量`b`暴露给当前作用域，可以使用 IIFE（参见[通过 IIFE 引入新作用域](ch16.html#iife "通过 IIFE 引入新作用域")）：

```js
(function () {
    var b = foo.bar.baz;
    console.log('Hello '+b.first+' '+b.last);
}());
```

您还可以选择将要访问的对象作为 IIFE 的参数：

```js
(function (b) {
    console.log('Hello '+b.first+' '+b.last);
}(foo.bar.baz));
```

### 弃用的原因

要理解为什么`with`被弃用，请看下面的例子，并注意函数的参数如何完全改变了它的工作方式：

```js
function logit(msg, opts) {
    with (opts) {
        console.log('msg: '+msg); // (1)
    }
}
```

如果`opts`有一个`msg`属性，那么第（1）行的语句不再访问参数`msg`。它访问属性：

```js
> logit('hello', {})  // parameter msg
msg: hello
> logit('hello', { msg: 'world' })  // property opts.msg
msg: world
```

`with`语句引起了三个问题：

性能下降

变量查找变慢，因为对象被临时插入到作用域链中。

代码变得不太可预测

您无法通过查看其语法环境（其词法上下文）来确定标识符指的是什么。根据[Brendan Eich](http://bit.ly/1jCrTKj)的说法，这才是`with`被弃用的实际原因，而不是性能考虑：

> `with`违反了词法作用域，使程序分析（例如安全性）变得困难或不可行。

缩小器（在第三十二章中描述）无法缩短变量名

在`with`语句内部，无法静态确定名称是指变量还是属性。缩小器只能重命名变量。

以下是`with`使代码变得脆弱的示例：

```js
function foo(someArray) {
    var values = ...;  // (1)
    with (someArray) {
        values.someMethod(...);  // (2)
        ...
    }
}
foo(myData);  // (3)
```

即使您无法访问数组`myData`，也可以阻止行（3）中的函数调用起作用。

如何？通过向`Array.prototype`添加一个属性`values`。例如：

```js
Array.prototype.values = function () {
    ...
};
```

现在，行（2）中的代码调用`someArray.values.someMethod()`而不是`values.someMethod()`。原因是，在`with`语句内，`values`现在指的是`someArray.values`，而不再是行（1）中的局部变量。

这不仅仅是一个思想实验：数组方法`values()`已添加到 Firefox 并破坏了 TYPO3 内容管理系统。[Brandon Benvie 找出了问题所在](http://mzl.la/1jCrXti)。

## 调试器语句

`debugger`语句的语法如下：

```js
debugger;
```

如果调试器处于活动状态，此语句将作为断点；如果没有，它没有可观察的效果。

