# 第十五章：函数

> 原文：[15. Functions](https://exploringjs.com/es5/ch15.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


函数是可以调用的值。定义函数的一种方式称为*函数声明*。例如，以下代码定义了具有单个参数`x`的函数`id`：

```js
function id(x) {
    return x;
}
```

`return`语句从`id`返回一个值。您可以通过提及其名称，后跟括号中的参数来调用函数：

```js
> id('hello')
'hello'
```

如果您从函数中不返回任何内容，则返回`undefined`（隐式）：

```js
> function f() { }
> f()
undefined
```

本节仅展示了定义函数的一种方式和调用函数的一种方式。其他方式将在后面描述。

## JavaScript 中函数的三种角色

一旦您像刚才所示那样定义了一个函数，它可以扮演多种角色：

非方法函数（“普通函数”）

您可以直接调用函数。然后它将作为普通函数工作。以下是一个示例调用：

```js
id('hello')
```

按照惯例，普通函数的名称以小写字母开头。

构造函数

您可以通过`new`运算符调用函数。然后它变成一个构造函数，一个对象的工厂。以下是一个示例调用：

```js
new Date()
```

按照惯例，构造函数的名称以大写字母开头。

方法

您可以将函数存储在对象的属性中，这将使其成为一个*方法*，您可以通过该对象调用它。以下是一个示例调用：

```js
obj.method()
```

按照惯例，方法的名称以小写字母开头。

非方法函数在本章中有解释；构造函数和方法在第十七章中有解释。

## 术语：“参数”与“参数”

术语*参数*和*参数*通常可以互换使用，因为上下文通常可以清楚地表明所需的含义。以下是区分它们的一个经验法则。

+   *参数*用于定义函数。它们也被称为形式参数和形式参数。在下面的例子中，`param1`和`param2`是参数：

    ```js
    function foo(param1, param2) {
        ...
    }
    ```

+   *参数*用于调用函数。它们也被称为实际参数和实际参数。在下面的例子中，`3`和`7`是参数：

    ```js
    foo(3, 7);
    ```

## 定义函数

本节描述了创建函数的三种方法：

+   通过函数表达式

+   通过函数声明

+   通过构造函数`Function()`

所有函数都是对象，是`Function`的实例：

```js
function id(x) {
    return x;
}
console.log(id instanceof Function); // true
```

因此，函数从`Function.prototype`获取它们的方法。

### 函数表达式

函数表达式产生一个值 - 一个函数对象。例如：

```js
var add = function (x, y) { return x + y };
console.log(add(2, 3)); // 5
```

前面的代码将函数表达式的结果分配给变量`add`，并通过该变量调用它。函数表达式产生的值可以分配给一个变量（如最后一个例子中所示），作为另一个函数的参数传递，等等。因为普通函数表达式没有名称，它们也被称为*匿名函数表达式*。

#### 命名函数表达式

您可以给函数表达式一个名称。*命名函数表达式*允许函数表达式引用自身，这对于自我递归很有用：

```js
var fac = function me(n) {
    if (n > 0) {
        return n * me(n-1);
    } else {
        return 1;
    }
};
console.log(fac(3)); // 6
```

### 注意

命名函数表达式的名称只能在函数表达式内部访问：

```js
var repeat = function me(n, str) {
    return n > 0 ? str + me(n-1, str) : '';
};
console.log(repeat(3, 'Yeah')); // YeahYeahYeah
console.log(me); // ReferenceError: me is not defined
```

### 函数声明

以下是一个函数声明：

```js
function add(x, y) {
    return x + y;
}
```

前面的代码看起来像一个函数表达式，但它是一个语句（参见[表达式与语句](ch07.html#expr_vs_stmt "表达式与语句")）。它大致相当于以下代码：

```js
var add = function (x, y) {
    return x + y;
};
```

换句话说，函数声明声明一个新变量，创建一个函数对象，并将其分配给变量。

### 函数构造函数

构造函数`Function()`评估存储在字符串中的 JavaScript 代码。例如，以下代码等同于前面的例子：

```js
var add = new Function('x', 'y', 'return x + y');
```

然而，这种定义函数的方式很慢，并且将代码保留在字符串中（无法访问工具）。因此，最好尽可能使用函数表达式或函数声明。[使用 new Function()评估代码](ch23.html#function_constructor "使用 new Function()评估代码")更详细地解释了`Function()`；它的工作方式类似于`eval()`。

## 提升

*提升*意味着“移动到作用域的开头”。函数声明完全提升，变量声明只部分提升。

函数声明完全被提升。这允许您在声明之前调用函数：

```js
foo();
function foo() {  // this function is hoisted
    ...
}
```

前面的代码之所以有效是因为 JavaScript 引擎将`foo`的声明移动到作用域的开头。它们执行代码，就好像它看起来是这样的：

```js
function foo() {
    ...
}
foo();
```

`var`声明也会被提升，但只有声明，而不是使用它们进行的赋值。因此，类似于前面的例子使用`var`声明和函数表达式会导致错误：

```js
foo();  // TypeError: undefined is not a function
var foo = function foo() {
    ...
};
```

只有变量声明被提升。引擎执行前面的代码如下：

```js
var foo;
foo();  // TypeError: undefined is not a function
foo = function foo() {
    ...
};
```

## 函数的名称

大多数 JavaScript 引擎支持函数对象的非标准属性`name`。函数声明具有它：

```js
> function f1() {}
> f1.name
'f1'
```

匿名函数表达式的名称是空字符串：

```js
> var f2 = function () {};
> f2.name
''
```

然而，命名函数表达式确实有一个名称：

```js
> var f3 = function myName() {};
> f3.name
'myName'
```

函数的名称对于调试很有用。有些人总是给他们的函数表达式命名。

## 哪个更好：函数声明还是函数表达式？

您是否更喜欢以下的函数声明？

```js
function id(x) {
    return x;
}
```

或者等效的`var`声明加上函数表达式的组合？

```js
var id = function (x) {
    return x;
};
```

它们基本上是相同的，但是函数声明比函数表达式有两个优点：

+   它们被提升（参见[提升](ch15.html#function_hoisting "Hoisting")），因此您可以在它们出现在源代码中之前调用它们。

+   它们有一个名称（请参见[函数的名称](ch15.html#function_names "函数的名称"））。但是，JavaScript 引擎正在更好地推断匿名函数表达式的名称。

## 对函数调用的更多控制：call()，apply()和 bind()

`call()`，`apply()`和`bind()`是所有函数都具有的方法（请记住函数是对象，因此具有方法）。它们可以在调用方法时提供`this`的值，因此主要在面向对象的上下文中很有趣（参见[调用函数时设置 this：call()，apply()和 bind()](ch17_split_000.html#oop_call_apply_bind "调用函数时设置 this：call()，apply()和 bind()"））。本节解释了非方法的两种用法。

### func.apply(thisValue, argArray)

此方法在调用函数`func`时使用`argArray`的元素作为参数；也就是说，以下两个表达式是等价的：

```js
func(arg1, arg2, arg3)
func.apply(null, [arg1, arg2, arg3])
```

`thisValue`是在执行`func`时`this`的值。在非面向对象的设置中不需要它，因此在这里是`null`。

`apply()`在函数以类似数组的方式接受多个参数时很有用，但不是一个数组。

由于`apply()`，我们可以使用`Math.max()`（参见[其他函数](ch21.html#Math_max "其他函数"））来确定数组的最大元素：

```js
> Math.max(17, 33, 2)
33
> Math.max.apply(null, [17, 33, 2])
33
```

### func.bind(thisValue, arg1, ..., argN)

这执行*部分函数应用* - 创建一个新函数，该函数使用`thisValue`调用`func`，并使用以下参数：从`arg1`到`argN`，然后是新函数的实际参数。在以下非面向对象的设置中，不需要`thisValue`，这就是为什么它在这里是`null`。

在这里，我们使用`bind()`创建一个新函数`plus1()`，它类似于`add()`，但只需要参数`y`，因为`x`始终为 1：

```js
function add(x, y) {
    return x + y;
}
var plus1 = add.bind(null, 1);
console.log(plus1(5));  // 6
```

换句话说，我们已经创建了一个等效于以下代码的新函数：

```js
function plus1(y) {
    return add(1, y);
}
```

## 处理缺失或额外的参数

JavaScript 不强制函数的 arity：您可以使用任意数量的实际参数调用它，而不受已定义的形式参数的限制。因此，实际参数和形式参数的数量可以以两种方式不同：

实际参数比形式参数多

额外的参数将被忽略，但可以通过特殊的类数组变量`arguments`检索（稍后讨论）。

实际参数比形式参数少

所有缺失的形式参数都具有值`undefined`。

### 按索引获取所有参数：特殊变量 arguments

特殊变量`arguments`仅存在于函数内（包括方法）。它是一个类似数组的对象，保存当前函数调用的所有实际参数。以下代码使用它：

```js
function logArgs() {
    for (var i=0; i<arguments.length; i++) {
        console.log(i+'. '+arguments[i]);
    }
}
```

以下是交互：

```js
> logArgs('hello', 'world')
0\. hello
1\. world
```

`arguments`具有以下特点：

+   它类似于数组，但不是数组。一方面，它有一个`length`属性，可以通过索引读取和写入单个参数。

另一方面，`arguments`不是一个数组，它只是类似于数组。它没有任何数组方法（`slice()`，`forEach()`等）。幸运的是，您可以借用数组方法或将`arguments`转换为数组，如[类数组对象和通用方法](ch17_split_001.html#array-like_objects "类数组对象和通用方法")中所述。

+   它是一个对象，因此所有对象方法和运算符都是可用的。例如，你可以使用`in`运算符（[迭代和属性检测](ch17_split_000.html#iterate_and_detect_properties "Iteration and Detection of Properties")）来检查`arguments`是否“有”给定的索引：

    ```js
    > function f() { return 1 in arguments }
    > f('a')
    false
    > f('a', 'b')
    true
    ```

你可以以类似的方式使用`hasOwnProperty()`（[迭代和属性检测](ch17_split_000.html#iterate_and_detect_properties "Iteration and Detection of Properties")）：

```js
> function g() { return arguments.hasOwnProperty(1) }
> g('a', 'b')
true
```

#### 已弃用的`arguments`特性

严格模式下会取消`arguments`的一些更不寻常的特性：

+   `arguments.callee`指的是当前函数。它主要用于在匿名函数中进行自递归，并且在严格模式下是不允许的。作为一种解决方法，可以使用命名函数表达式（参见[命名函数表达式](ch15.html#named_function_expression "Named function expressions")），它可以通过其名称引用自身。

+   在非严格模式下，如果更改参数，`arguments`会保持最新：

    ```js
    function sloppyFunc(param) {
        param = 'changed';
        return arguments[0];
    }
    console.log(sloppyFunc('value'));  // changed
    ```

但是在严格模式下不会进行这种更新：

```js
function strictFunc(param) {
    'use strict';
    param = 'changed';
    return arguments[0];
}
console.log(strictFunc('value'));  // value
```

+   严格模式禁止对变量`arguments`进行赋值（例如通过`arguments++`）。仍然允许对元素和属性进行赋值。

### 强制参数，强制最小数量

有三种方法可以找出参数是否缺失。首先，你可以检查它是否为`undefined`：

```js
function foo(mandatory, optional) {
    if (mandatory === undefined) {
        throw new Error('Missing parameter: mandatory');
    }
}
```

其次，你可以将参数解释为布尔值。然后`undefined`被视为`false`。但是，有一个警告：其他几个值也被视为`false`（参见[真值和假值](ch10.html#truthy_falsy "Truthy and Falsy Values")），因此检查无法区分，比如`0`和缺少的参数：

```js
if (!mandatory) {
    throw new Error('Missing parameter: mandatory');
}
```

第三，你也可以检查`arguments`的长度以强制最小 arity：

```js
if (arguments.length < 1) {
    throw new Error('You need to provide at least 1 argument');
}
```

最后一种方法与其他方法不同：

+   前两种方法不区分`foo()`和`foo(undefined)`。在这两种情况下，都会抛出异常。

+   第三种方法对`foo()`抛出异常，并对`foo(undefined)`将`optional`设置为`undefined`。

### 可选参数

如果参数是可选的，这意味着如果缺少参数，则给它一个默认值。与强制参数类似，有四种替代方案。

首先，检查`undefined`：

```js
function bar(arg1, arg2, optional) {
    if (optional === undefined) {
        optional = 'default value';
    }
}
```

其次，将`optional`解释为布尔值：

```js
if (!optional) {
    optional = 'default value';
}
```

第三，你可以使用或运算符`||`（参见[逻辑或(||)](ch10.html#logical_or "Logical Or (||)")），如果左操作数不是假值，则返回左操作数。否则，返回右操作数：

```js
// Or operator: use left operand if it isn't falsy
optional = optional || 'default value';
```

第四，你可以通过`arguments.length`检查函数的 arity：

```js
if (arguments.length < 3) {
    optional = 'default value';
}
```

再次，最后一种方法与其他方法不同：

+   前三种方法不区分`bar(1, 2)`和`bar(1, 2, undefined)`。在这两种情况下，`optional`都是`'default value'`。

+   第四种方法为`bar(1, 2)`设置`optional`为`'default value'`，并且对于`bar(1, 2, undefined)`保持`undefined`（即不变）。

另一种可能性是将可选参数作为*命名参数*传递，作为对象字面量的属性（参见[命名参数](ch15.html#named_parameters "Named Parameters")）。

### 模拟通过引用传递参数

在 JavaScript 中，你不能通过引用传递参数；也就是说，如果你将一个变量传递给一个函数，它的值会被复制并传递给函数（按值传递）。因此，函数无法更改变量。如果需要这样做，必须将变量的值封装在数组中。

这个例子演示了一个增加变量的函数：

```js
function incRef(numberRef) {
    numberRef[0]++;
}
var n = [7];
incRef(n);
console.log(n[0]);  // 8
```

### 陷阱：意外的可选参数

如果将函数`c`作为参数传递给另一个函数`f`，则必须了解两个签名：

+   `f`期望其参数具有的签名。`f`可能提供多个参数，而`c`可以决定使用其中的多少（如果有的话）。

+   `c`的实际签名。例如，它可能支持可选参数。

如果两者不一致，那么您可能会得到意想不到的结果：`c`可能具有您不知道的可选参数，并且会错误地解释`f`提供的附加参数。

例如，考虑数组方法`map()`（参见[转换方法](ch18.html#Array.prototype.map "转换方法"）），其参数通常是一个带有单个参数的函数：

```js
> [ 1, 2, 3 ].map(function (x) { return x * x })
[ 1, 4, 9 ]
```

您可以将`parseInt()`作为参数传递给一个函数（参见[通过 parseInt()获取整数](ch11.html#parseInt "通过 parseInt()获取整数"））：

```js
> parseInt('1024')
1024
```

您可能（错误地）认为`map()`只提供了一个参数，而`parseInt()`只接受了一个参数。然后您会对以下结果感到惊讶：

```js
> [ '1', '2', '3' ].map(parseInt)
[ 1, NaN, NaN ]
```

`map()`期望具有以下签名的函数：

```js
function (element, index, array)
```

但是`parseInt()`具有以下签名：

```js
parseInt(string, radix?)
```

因此，`map()`不仅填充了`string`（通过`element`），还填充了`radix`（通过`index`）。这意味着前面数组的值是这样产生的：

```js
> parseInt('1', 0)
1
> parseInt('2', 1)
NaN
> parseInt('3', 2)
NaN
```

总之，对于您不确定其签名的函数和方法要小心。如果使用它们，明确指定接收了哪些参数并传递了哪些参数通常是有意义的。这是通过回调函数实现的：

```js
> ['1', '2', '3'].map(function (x) { return parseInt(x, 10) })
[ 1, 2, 3 ]
```

## 命名参数

在调用编程语言中的函数（或方法）时，您必须将实际参数（由调用者指定）映射到函数定义的形式参数。有两种常见的方法来实现这一点：

+   *位置参数*按位置进行映射。第一个实际参数映射到第一个形式参数，第二个实际参数映射到第二个形式参数，依此类推。

+   *命名参数*使用*名称*（标签）执行映射。名称与函数定义中的形式参数相关联，并标记函数调用中的实际参数。命名参数出现的顺序并不重要，只要它们被正确标记。

命名参数有两个主要好处：它们为函数调用中的参数提供描述，并且对于可选参数也很有效。我将首先解释这些好处，然后向您展示如何通过对象字面量在 JavaScript 中模拟命名参数。

### 命名参数作为描述

一旦函数有多个参数，您可能会对每个参数的用途感到困惑。例如，假设您有一个名为`selectEntries()`的函数，它从数据库中返回条目。给定以下函数调用：

```js
selectEntries(3, 20, 2);
```

这两个数字代表什么？Python 支持命名参数，这使得很容易弄清楚发生了什么：

```js
selectEntries(start=3, end=20, step=2)  # Python syntax
```

### 可选命名参数

可选位置参数仅在末尾省略时才有效。在其他任何地方，您必须插入占位符，例如`null`，以便剩余参数具有正确的位置。对于可选命名参数，这不是问题。您可以轻松地省略其中任何一个。以下是一些示例：

```js
# Python syntax
selectEntries(step=2)
selectEntries(end=20, start=3)
selectEntries()
```

### 在 JavaScript 中模拟命名参数

JavaScript 不像 Python 和许多其他语言那样原生支持命名参数。但是有一个相当优雅的模拟方法：通过对象字面量命名参数，作为单个实际参数传递。当您使用这种技术时，`selectEntries()`的调用看起来像：

```js
selectEntries({ start: 3, end: 20, step: 2 });
```

该函数接收一个具有属性`start`、`end`和`step`的对象。您可以省略其中任何一个：

```js
selectEntries({ step: 2 });
selectEntries({ end: 20, start: 3 });
selectEntries();
```

您可以将`selectEntries()`实现如下：

```js
function selectEntries(options) {
    options = options || {};
    var start = options.start || 0;
    var end = options.end || getDbLength();
    var step = options.step || 1;
    ...
}
```

您还可以将位置参数与命名参数结合使用。后者通常出现在最后：

```js
selectEntries(posArg1, posArg2, { namedArg1: 7, namedArg2: true });
```

### 注意

在 JavaScript 中，这里显示的命名参数模式有时被称为*选项*或*选项对象*（例如，由 jQuery 文档）。

