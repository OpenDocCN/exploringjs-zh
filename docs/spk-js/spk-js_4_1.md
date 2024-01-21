# 第二十六章：元代码风格指南

> 原文：[26. A Meta Code Style Guide](https://exploringjs.com/es5/ch26.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 有许多优秀的风格指南。因此，没有必要再写一个。相反，本章描述了元风格规则，并调查了现有的风格指南和已建立的最佳实践。它还提到了我喜欢的一些更有争议的做法。这个想法是为了补充现有的风格指南，而不是取代它们。

## 现有的风格指南

这些是我喜欢的风格指南：

+   [Idiomatic.js：编写一致的、惯用的 JavaScript 的原则](https://github.com/rwaldron/idiomatic.js/)

+   [Google JavaScript 风格指南](http://bit.ly/1oOEfQ7)

+   [jQuery JavaScript 风格指南](http://contribute.jquery.org/style-guide/js/)

+   [Airbnb JavaScript 风格指南](https://github.com/airbnb/javascript)

此外，还有两个元风格指南：

+   [GitHub 上的流行约定](http://sideeffect.kr/popularconvention/)分析 GitHub 代码，找出最常用的编码约定。

+   [JavaScript，获胜的风格](http://seravo.fi/2013/javascript-the-winning-style)检查了几种流行风格指南的大多数推荐。

## 一般提示

本节将涵盖一些一般的代码编写技巧。

### 代码应该一致

编写一致代码的两个重要规则。第一条规则是，如果你开始一个新项目，你应该想出一个风格，记录下来，并在任何地方都遵循它。团队越大，检查对风格的自动遵循就越重要，可以通过诸如 JSHint 之类的工具实现。在风格方面，有许多决定要做。其中大多数都有普遍认可的答案。其他必须根据项目定义。例如：

+   有多少空格（括号后，语句之间等）

+   缩进（例如，每级缩进多少空格）

+   如何在哪里编写`var`语句

第二条规则是，如果你加入一个现有项目，你应该严格遵循它的规则（即使你不同意它们）。

### 代码应该易于理解

> 每个人都知道调试比一开始编写程序要困难两倍。因此，如果你在编写时越聪明，那么你将如何调试呢？ ——Brian Kernighan

对于大多数代码，用于阅读的时间远远大于用于编写的时间。因此，使前者尽可能简单非常重要。以下是一些指导方针：

更短并不总是更好

有时写*更多*意味着事情实际上更容易阅读。让我们考虑两个例子。首先，熟悉的事物更容易理解。这意味着使用熟悉的、稍微更冗长的结构可能更可取。其次，人类读取标记，而不是字符。因此，`redBalloon`比`rdBlln`更容易阅读。

好的代码就像教科书

大多数代码库都充满了新的想法和概念。这意味着如果你想要使用一个代码库，你需要学习这些想法和概念。与教科书相比，代码的额外挑战在于人们不会线性阅读它。他们会随时跳进来，应该能够大致理解发生了什么。代码库的三个部分有所帮助：

+   *代码*应该解释*发生了什么；它应该是不言自明的。为了编写这样的代码，使用描述性标识符，并将长函数（或方法）分解为更小的子函数。如果这些函数足够小并且有意义的名称，你通常可以避免注释。

+   *注释*应该解释*为什么*事情发生。如果你需要了解一个概念才能理解代码，你可以在标识符中包含该概念的名称，或者在注释中提到它。阅读代码的人可以查阅文档，了解更多关于该概念的信息。

+   *文档*应填补代码和注释留下的空白。它应该告诉你如何开始使用代码库，并为你提供大局观。它还应包含所有重要概念的词汇表。

不要聪明；不要让我思考

有很多巧妙的代码利用对语言的深入了解来实现令人印象深刻的简洁性。这样的代码通常像一个谜题，很难理解。你会遇到这样的观点，如果人们不理解这样的代码，也许他们真的应该先学习 JavaScript。但这不是这篇文章要讨论的。无论你有多聪明，进入其他人的思维世界总是具有挑战性的。所以简单的代码并不愚蠢，它是大部分努力都花在让一切易于理解的代码。

避免为速度或代码大小进行优化

许多巧妙的技巧都是针对这些优化的。然而，你通常不需要它们。一方面，JavaScript 引擎变得越来越智能，自动优化遵循已建立模式的代码的速度。另一方面，缩小工具（第三十二章）重写你的代码，使其尽可能小。在这两种情况下，工具都是为你聪明的，这样你就不必自己聪明了。

有时你别无选择，只能优化代码的性能。如果你这样做，请确保测量和优化正确的部分。在浏览器中，问题通常与 DOM 和 HTML 相关，而不是语言本身。

## 常见的最佳实践

大多数 JavaScript 程序员都同意以下最佳实践：

+   使用严格模式。它使 JavaScript 成为一种更清洁的语言（参见[严格模式](ch07.html#strict_mode "严格模式")）。

+   始终使用分号。避免自动分号插入的陷阱（参见[自动分号插入](ch07.html#automatic_semicolon_insertion "自动分号插入")）。

+   始终使用严格相等（`===`）和严格不等（`!==`）。我建议永远不要偏离这个规则。即使它们是等价的，我甚至更喜欢以下两个条件中的第一个：

    ```js
    if (x !== undefined && x !== null) ...  // my choice
    if (x != null) ...  // equivalent
    ```

+   要么只使用空格，要么只使用制表符进行缩进，但不要混合使用它们。

+   引用字符串：在 JavaScript 中，你可以用单引号或双引号写字符串文字。单引号更常见。它们使得处理 HTML 代码更容易（通常 HTML 代码中的属性值是双引号）。其他考虑因素在[字符串文字](ch12.html#quoting_strings "字符串文字")中提到。

+   避免全局变量（[最佳实践：避免创建全局变量](ch16.html#avoid_global_variables "最佳实践：避免创建全局变量")）。

### 括号样式

在大括号界定代码块的语言中，括号样式决定你放置这些括号的位置。在类 C 语言（如 Java 和 JavaScript）中，有两种最常见的括号样式：Allman 样式和 1TBS。

#### Allman 样式

如果一个语句包含一个块，那么该块被认为与语句的头部有些分离：它的左大括号在自己的一行上，与头部的缩进级别相同。例如：

```js
// Allman brace style
function foo(x, y, z)
{
    if (x)
    {
        a();
    }
    else
    {
        b();
        c();
    }
}
```

#### 1TBS（真正的括号样式）

在这里，一个块与其语句的标题更紧密地关联在一起；它在同一行之后开始。控制流语句的主体总是放在大括号中，即使只有一个语句。例如：

```js
// One True Brace Style
function foo(x, y, z) {
    if (x) {
        a();
    } else {
        b();
        c();
    }
}
```

1TBS 是(Kernighan 和 Ritchie)样式的一个变体。在 K&R 样式中，函数以 Allman 样式编写，并且在不必要的情况下省略大括号，例如，在单语句`then`情况下：

```js
// K&R brace style
function foo(x, y, z)
{
    if (x)
        a();
    else {
        b();
        c();
    }
}
```

#### JavaScript

JavaScript 世界中的事实标准是 1TBS。它是从 Java 继承而来，大多数风格指南都推荐使用它。其中一个原因是客观的。如果你返回一个对象字面量，你必须将开括号放在与关键字`return`相同的行上，就像这样（否则，自动分号插入会在`return`后插入一个分号，意味着什么也没有返回；参见[Pitfall: ASI can unexpectedly break up statements](ch07.html#asi_after_return "Pitfall: ASI can unexpectedly break up statements")）：

```js
return {
    name: 'Jane'
};
```

显然，对象字面量不是一个代码块，但如果两者格式化方式相同，看起来更一致，你犯错的可能性就更小。

我的个人风格和偏好是：

+   1TBS（这意味着你应该尽可能使用大括号）。

+   作为例外，如果一个语句可以写在一行上，我会省略大括号。例如：

    ```js
    if (x) return x;
    ```

### 更喜欢字面量而不是构造函数

几个字面量产生的对象也可以通过构造函数创建。然而，后者通常是更好的选择：

```js
var obj = new Object(); // no
var obj = {}; // yes

var arr = new Array(); // no
var arr = []; // yes

var regex = new RegExp('abc'); // avoid if possible
var regex = /abc/; // yes
```

永远不要使用构造函数`Array`来创建具有给定元素的数组。[初始化具有元素的数组（避免！）](ch18.html#avoid_array_constructor "初始化具有元素的数组（避免！）")解释了原因：

```js
var arr = new Array('a', 'b', 'c'); // never ever
var arr = [ 'a', 'b', 'c' ]; // yes
```

### 不要聪明

本节收集了一些不推荐的聪明用法。

#### 条件运算符

不要嵌套条件运算符：

```js
// Don’t:
return x === 0 ? 'red' : x === 1 ? 'green' : 'blue';

// Better:
if (x === 0) {
    return 'red';
} else if (x === 1) {
    return 'green';
} else {
    return 'blue';
}

// Best:
switch (x) {
    case 0:
        return 'red';
    case 1:
        return 'green';
    default:
        return 'blue';
}
```

#### 缩写 if 语句

不要通过逻辑运算符缩写`if`语句：

```js
foo && bar(); // no
if (foo) bar(); // yes

foo || bar(); // no
if (!foo) bar(); // yes
```

#### 增量运算符

如果可能的话，使用增量运算符（`++`）和减量运算符（`--`）作为语句；不要将它们用作表达式。在后一种情况下，它们会返回一个值，虽然有一个助记符，但你仍然需要思考来弄清楚发生了什么：

```js
// Unsure: what is happening?
return ++foo;

// Easy to understand
++foo;
return foo;
```

#### 检查未定义

```js
if (x === void 0) x = 0; // not necessary in ES5
if (x === undefined) x = 0; // preferable
```

从 ECMAScript 5 开始，第二种检查方式更好。[更改未定义](ch08.html#changing_undefined "更改未定义")解释了为什么。

#### 将数字转换为整数

```js
return x >> 0; // no
return Math.round(x); // yes
```

移位运算符可以用来将数字转换为整数。然而，通常最好使用更明确的替代方法，比如`Math.round()`。[转换为整数](ch11.html#converting_to_integer "转换为整数")概述了所有转换为整数的方法。

### 可接受的聪明用法

有时候你可以在 JavaScript 中很聪明——如果这种聪明已经成为一种已经建立的模式。

#### 默认值

使用或（`||`）运算符提供默认值是一种常见的模式——例如，对于参数：

```js
function f(x) {
    x = x || 0;
    ...
}
```

有关详细信息和更多示例，请参阅[模式：提供默认值](ch10.html#default_via_or "模式：提供默认值")。

#### 通用方法

如果你通用地使用方法，你可以将`Object.prototype`缩写为`{}`。以下两个表达式是等价的：

```js
Object.prototype.hasOwnProperty.call(obj, propKey)
{}.hasOwnProperty.call(obj, propKey)
```

`Array.prototype`可以缩写为`[]`：

```js
Array.prototype.slice.call(arguments)
[].slice.call(arguments)
```

我对这个持观望态度。这是一个技巧（你正在通过一个实例访问原型属性）。但它减少了混乱，我期望引擎最终会优化这种模式。

#### ECMAScript 5：尾随逗号

在 ECMAScript 5 中，对象字面量中的尾随逗号是合法的：

```js
var obj = {
    first: 'Jane',
    last: 'Doe', // legal: trailing comma
};
```

#### ECMAScript 5：保留字

ECMAScript 5 还允许你使用保留字（如`new`）作为属性键：

```js
> var obj = { new: 'abc' };
> obj.new
'abc'
```

## 有争议的规则

让我们看看一些我喜欢的、有点具有争议的惯例。

### 语法

我们将从语法惯例开始：

紧凑的空格

我喜欢*相对*紧凑的空格。这个模型是用英语写的：在开括号后和闭括号前没有空格。逗号后有空格：

```js
var result = foo('a', 'b');
var arr = [ 1, 2, 3 ];
if (flag) {
    ...
}
```

对于匿名函数，我遵循道格拉斯·克罗克福德的规则，在关键字`function`后面加一个空格。理由是，如果去掉名字，这就是一个命名函数表达式的样子：

```js
function foo(arg) { ... }  // named function expression
function (arg) { ... }     // anonymous function expression
```

每个缩进级别四个空格

我看到的大多数代码都使用空格缩进，因为制表符在应用程序和操作系统之间显示的方式有很大不同。我更喜欢每级缩进四个空格，因为这样缩进更加明显。

将条件操作符放在括号中

这有助于阅读，因为更容易确定操作符的范围：

```js
return result ? result : theDefault;  // no
return (result ? result : theDefault);  // yes
```

### 变量

接下来，我将介绍变量的约定：

每行只声明一个变量

不要用单个声明声明多个变量：

```js
// no
var foo = 3,
    bar = 2,
    baz;

// yes
var foo = 3;
var bar = 2;
var baz;
```

这种方法的优势在于删除、插入和重新排列行更简单，行也会自动正确缩进。

保持变量声明局部

如果你的函数不太长（无论如何都不应该太长），那么你可以在提升方面放松一些，假装`var`声明是块作用域的。换句话说，你可以在使用变量的上下文中声明变量（在循环内，在`then`块或`else`块内等）。这种局部封装使得代码片段更容易理解。也更容易删除代码片段或将其移动到其他地方。

如果你在一个块内，就待在那个块内

作为前一条规则的补充：不要在两个不同的块中声明相同的变量。例如：

```js
// Don’t do this
if (v) {
    var x = v;
} else {
    var x = 10;
}
doSomethingWith(x);
```

前面的代码和下面的代码有相同的效果和意图，所以应该这样写：

```js
var x;
if (v) {
    x = v;
} else {
    x = 10;
}
doSomethingWith(x);
```

### 面向对象

现在我们将讨论与面向对象有关的约定。

优先使用构造函数而不是其他实例创建模式

我建议你：

+   总是使用构造函数。

+   创建实例时总是使用`new`。

这样做的主要优势是：

+   你的代码更适合 JavaScript 主流，更有可能在不同框架之间移植。

+   在现代引擎中，使用构造函数的实例非常快（例如，通过[hidden classes](http://bit.ly/1oOEAlZ)）。

+   在即将到来的 ECMAScript 6 中，类将是默认的继承构造。

对于构造函数，使用严格模式很重要，因为它可以防止你忘记实例化时使用`new`操作符。你应该知道你可以在构造函数中返回任何对象。有关使用构造函数的更多提示，请参阅[实现构造函数的提示](ch17_split_001.html#constructor_tips "Tips for Implementing Constructors")。

避免使用闭包来处理私有数据

如果你希望对象的私有数据完全安全，你必须使用闭包。否则，你可以使用普通属性。一个常见的做法是在私有属性的名称前加下划线。闭包的问题在于代码变得更加复杂（除非你将所有方法都放在实例中，这是不符合惯例且慢的），而且速度更慢（访问闭包中的数据目前比访问属性更慢）。[保持数据私有](ch17_split_001.html#private_data_for_objects "Keeping Data Private")更详细地介绍了这个主题。

如果构造函数没有参数，写括号

我发现这样的构造函数调用用括号看起来更清晰：

```js
var foo = new Foo;  // no
var foo = new Foo();  // yes
```

小心操作符优先级

使用括号，这样两个操作符就不会相互竞争——结果并不总是你所期望的：

```js
> false && true || true
true
> false && (true || true)
false
> (false && true) || true
true
```

`instanceof`特别棘手：

```js
> ! {} instanceof Array
false
> (!{}) instanceof Array
false
> !({} instanceof Array)
true
```

然而，我发现构造函数后的方法调用并不成问题：

```js
new Foo().bar().baz();  // ok
(new Foo()).bar().baz();  // not necessary
```

### 杂项

这一部分收集了各种提示：

强制转换

通过`Boolean`、`Number`、`String()`、`Object()`（作为函数使用——永远不要将这些函数用作构造函数）将值强制转换为类型。理由是这种约定更具描述性：

```js
> +'123'  // no
123
> Number('123')  // yes
123

> ''+true  // no
'true'
> String(true)  // yes
'true'
```

避免使用`this`作为隐式参数

`this`应该只指当前方法调用的接收者；不应滥用作为隐式参数。理由是这样的函数更容易调用和理解。我也喜欢保持面向对象和函数机制分开：

```js
// Avoid:
function handler() {
    this.logError(...);
}

// Prefer:
function handler(context) {
    context.logError(...);
}
```

通过`in`和`hasOwnProperty`检查属性的存在（参见[Iteration and Detection of Properties](ch17_split_000.html#iterate_and_detect_properties "Iteration and Detection of Properties")）

这比与`undefined`进行比较或检查真实性更加自明和安全：

```js
// All properties:
if (obj.foo)  // no
if (obj.foo !== undefined)  // no
if ('foo' in obj) ... // yes

// Own properties:
if (obj.hasOwnProperty('foo')) ... // risky for arbitrary objects
if (Object.prototype.hasOwnProperty.call(obj, 'foo')) ... // safe
```

快速失败

如果可以的话，最好是快速失败，而不是悄悄失败。JavaScript 只是如此宽容（例如，除以零），因为 ECMAScript 的第一个版本没有异常。例如，不要强制转换值；抛出异常。但是，当您的代码处于生产状态时，您必须找到从失败中恢复的方法。

## 结论

每当您考虑样式问题时，请问自己：什么使我的代码更容易理解？抵制诱惑，不要聪明，把大部分机械聪明留给 JavaScript 引擎和缩小器（参见第三十二章）。

* * *

¹⁹ 甚至有人说它们是同义词，1TBS 是一种戏谑地指代 K&R 的方式。

