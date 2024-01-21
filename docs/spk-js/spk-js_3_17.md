# 第二十三章：标准全局变量

> 原文：[23. Standard Global Variables](https://exploringjs.com/es5/ch23.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章是 ECMAScript 规范标准化的全局变量的参考。Web 浏览器有更多全局变量，这些变量在 [MDN 上列出](https://developer.mozilla.org/en-US/docs/Web/API/Window)。所有全局变量都是全局对象的（自有或继承的）属性（在浏览器中是 `window`；参见 [全局对象](ch16.html#global_object "全局对象")）。

## 构造函数

有关以下构造函数的详细信息，请参见括号中指示的部分：

+   `Array`（[数组构造函数](ch18.html#array_constructor "数组构造函数"））

+   `Boolean`（[原始值的包装对象](ch08.html#wrapper_objects "原始值的包装对象"））

+   `Date`（[日期构造函数](ch20.html#date_constructors "日期构造函数"））

+   `Function`（[使用 new Function() 评估代码](ch23.html#function_constructor "使用 new Function() 评估代码"））

+   `Number`（[原始值的包装对象](ch08.html#wrapper_objects "原始值的包装对象"））

+   `对象`（[将任何值转换为对象](ch17_split_000.html#toobject "将任何值转换为对象"））

+   `RegExp`（[创建正则表达式](ch19.html#creating_regexps "创建正则表达式"））

+   `String`（[原始值的包装对象](ch08.html#wrapper_objects "原始值的包装对象"））

## 错误构造函数

有关这些构造函数的详细信息，请参见 [错误构造函数](ch14.html#error_constructors "错误构造函数"）：

+   `Error`

+   `EvalError`

+   `RangeError`

+   `ReferenceError`

+   `SyntaxError`

+   `TypeError`

+   `URIError`

## 非构造函数

有几个全局函数不是构造函数。它们在本节中列出。

### 编码和解码文本

以下函数处理 URI 编码和解码的几种方式：

`encodeURI(uri)`

在 `uri` 中对特殊字符进行百分比编码。特殊字符是除以下字符外的所有 Unicode 字符：

| URI 字符： | `; , / ? : @ & = + $ #` |
| --- | --- |
| 未编码： | `a-z A-Z 0-9 - _ . ! ~ * ' ( )` |

例如：

```js
> encodeURI('http://example.com/Für Elise/')
'http://example.com/F%C3%BCr%20Elise/'
```

`encodeURIComponent(uriComponent)`

在 `uriComponent` 中对所有字符进行百分比编码，除了：

| 未编码： | `a-z A-Z 0-9 - _ . ! ~ * ' ( )` |

与 `encodeURI` 相反，URL 和文件名中有意义的字符也被编码了。因此，您可以使用此函数将任何文本转换为合法的文件名或 URL 路径段。例如：

```js
> encodeURIComponent('http://example.com/Für Elise/')
'http%3A%2F%2Fexample.com%2FF%C3%BCr%20Elise%2F'
```

`decodeURI(encodedURI)`

解码由 `encodeURI` 生成的百分比编码的 URI：

```js
> decodeURI('http://example.com/F%C3%BCr%20Elise/')
'http://example.com/Für Elise/'
```

`encodeURI` 不会对 URI 字符进行编码，`decodeURI` 也不会对其进行解码，即使它们已经被正确编码：

```js
> decodeURI('%2F')
'%2F'
> decodeURIComponent('%2F')
'/'
```

`decodeURIComponent(encodedURIComponent)`

解码由 `encodeURIComponent` 生成的百分比编码的 URI 组件。与 `decodeURI` 相反，所有百分比编码的字符都被解码：

```js
> decodeURIComponent('http%3A%2F%2Fexample.com%2FF%C3%BCr%20Elise%2F')
'http://example.com/Für Elise/'
```

以下内容已被弃用：

+   `escape(str)`对`str`进行百分比编码。它已被弃用，因为它不能正确处理非 ASCII 字符。请改用`encodeURIComponent()`。

+   `unescape(str)`对`str`进行百分比解码。它已被弃用，因为它不能正确处理非 ASCII 字符。请改用`decodeURIComponent()`。

### 对数字进行分类和解析

以下方法有助于对数字进行分类和解析：

+   `isFinite(number)` ([检查是否为无穷大](ch11.html#isFinite "检查是否为无穷大"))

+   `isNaN(value)` ([陷阱：检查值是否为 NaN](ch11.html#isNaN "陷阱：检查值是否为 NaN"))

+   `parseFloat(string)` ([parseFloat()](ch11.html#parseFloat "parseFloat()"))

+   `parseInt(string, radix)` ([通过 parseInt()获取整数](ch11.html#parseInt "通过 parseInt()获取整数"))

## 通过 eval()和 new Function()动态评估 JavaScript 代码

本节将介绍如何在 JavaScript 中动态评估代码。

### 使用 eval()评估代码

函数调用：

```js
eval(str)
```

评估`str`中的 JavaScript 代码。例如：

```js
> var a = 12;
> eval('a + 5')
17
```

请注意，`eval()`在语句上下文中解析（参见[表达式与语句](ch07.html#expr_vs_stmt "表达式与语句")）：

```js
> eval('{ foo: 123 }')  // code block
123
> eval('({ foo: 123 })')  // object literal
{ foo: 123 }
```

#### 在严格模式下使用 eval()

对于`eval()`，您确实应该使用严格模式（参见[严格模式](ch07.html#strict_mode "严格模式")）。在松散模式下，评估的代码可以在周围范围内创建局部变量：

```js
function sloppyFunc() {
    eval('var foo = 123');  // added to the scope of sloppyFunc
    console.log(foo);  // 123
}
```

在严格模式下无法发生：

```js
function strictFunc() {
    'use strict';
    eval('var foo = 123');
    console.log(foo);  // ReferenceError: foo is not defined
}
```

然而，即使在严格模式下，评估的代码仍然可以读取和写入周围范围内的变量。要防止这种访问，您需要间接调用`eval()`。

#### 间接 eval()在全局范围内进行评估

有两种调用`eval()`的方法：

+   [直接](http://ecma-international.org/ecma-262/5.1/#sec-15.1.2.1.1)。通过直接调用名称为`eval`的函数。

+   间接调用。以其他方式（通过`call()`，作为`window`的方法，通过在不同名称下存储它并在那里调用等）。

正如我们已经看到的，直接`eval()`在当前范围内执行代码：

```js
var x = 'global';

function directEval() {
    'use strict';
    var x = 'local';

    console.log(eval('x')); // local
}
```

相反，间接`eval()`在全局范围内执行它：

```js
var x = 'global';

function indirectEval() {
    'use strict';
    var x = 'local';

    // Don’t call eval directly
    console.log(eval.call(null, 'x')); // global
    console.log(window.eval('x')); // global
    console.log((1, eval)('x')); // global (1)

    // Change the name of eval
    var xeval = eval;
    console.log(xeval('x')); // global

    // Turn eval into a method
    var obj = { eval: eval };
    console.log(obj.eval('x')); // global
}
```

（1）的解释：当您通过名称引用变量时，初始结果是所谓的[*引用*](http://ecma-international.org/ecma-262/5.1/#sec-8.7)，一个具有两个主要字段的数据结构：

+   `base`指向*环境*，即变量值存储的数据结构。

+   `referencedName`是变量的名称。

在`eval()`函数调用期间，函数调用运算符（括号）遇到对`eval`的引用，并且可以确定要调用的函数的名称。因此，这样的函数调用触发了直接的`eval()`。但是，您可以通过不给出调用运算符的引用来强制间接`eval()`。这是通过在应用运算符之前检索引用的值来实现的。逗号运算符在第（1）行为我们执行此操作。此运算符评估第一个操作数并返回评估第二个操作数的结果。评估始终产生值，这意味着引用被解析并丢失了函数名称。

间接评估的代码总是松散的。这是代码独立于其当前环境进行评估的结果：

```js
function strictFunc() {
    'use strict';

    var code = '(function () { return this }())';
    var result = eval.call(null, code);
    console.log(result !== undefined); // true, sloppy mode
}
```

### 使用 new Function()评估代码

构造函数`Function()`的签名为：

```js
new Function(param1, ..., paramN, funcBody)
```

它创建一个函数，其零个或多个参数的名称为`param1`，`parem2`等，其主体为`funcBody`；也就是说，创建的函数如下所示：

```js
function («param1», ..., «paramN») {
    «funcBody»
}
```

让我们使用`new Function()`创建一个函数`f`，它返回其参数的总和：

```js
> var f = new Function('x', 'y', 'return x+y');
> f(3, 4)
7
```

类似于间接`eval()`，`new Function()`创建其作用域为全局的函数:¹⁶

```js
var x = 'global';

function strictFunc() {
    'use strict';
    var x = 'local';

    var f = new Function('return x');
    console.log(f()); // global
}
```

这样的函数默认情况下也是松散的：

```js
function strictFunc() {
    'use strict';

    var sl = new Function('return this');
    console.log(sl() !== undefined); // true, sloppy mode

    var st = new Function('"use strict"; return this');
    console.log(st() === undefined); // true, strict mode
}
```

### eval()与 new Function()

通常，最好使用`new Function()`而不是`eval()`来评估代码：函数参数为评估的代码提供了清晰的接口，而且你不需要间接`eval()`的略显笨拙的语法来确保评估的代码只能访问全局变量（除了它自己的变量）。

### 最佳实践

你不应该使用`eval()`和`new Function()`。动态评估代码很慢，而且存在潜在的安全风险。它还会阻止大多数使用静态分析的工具（如 IDE）考虑代码。

通常有更好的替代方案。例如，Brendan Eich 最近在推特上[发推文](http://bit.ly/1fwpWrB)指出了程序员们使用的反模式，他们想要访问存储在变量`propName`中的属性：

```js
var value = eval('obj.'+propName);
```

这个想法是有道理的：点运算符只支持固定的，静态提供的属性键。在这种情况下，属性键只在运行时知道，这就是为什么需要`eval()`来使用该运算符。幸运的是，JavaScript 还有方括号运算符，它接受动态属性键。因此，以下是前面代码的更好版本：

```js
var value = obj[propName];
```

你也不应该使用`eval()`或`new Function()`来解析 JSON 数据。这是不安全的。要么依赖 ECMAScript 5 对 JSON 的内置支持（参见第二十二章），要么使用一个库。

#### 合法的用例

`eval()`和`new Function()`有一些合法的，尽管是高级的用例：带有函数的配置数据（JSON 不允许），模板库，解释器，命令行和模块系统。

### 结论

这是 JavaScript 动态评估代码的一个相对高级的概述。如果你想深入了解，可以查看 kangax 的文章[“全局 eval。有哪些选项？”](http://perfectionkills.com/global-eval-what-are-the-options/)。

## 控制台 API

在大多数 JavaScript 引擎中，有一个全局对象`console`，其中包含用于记录和调试的方法。该对象不是语言本身的一部分，但已成为事实上的标准。由于它们的主要目的是调试，`console`方法在开发过程中最常用，而在部署的代码中很少使用。

本节概述了控制台 API。它记录了 Chrome 32、Firebug 1.12、Firefox 25、Internet Explorer 11、Node.js 0.10.22 和 Safari 7.0 的现状。

### 控制台 API 在各种引擎之间的标准化程度如何？

控制台 API 的实现差异很大，而且不断变化。如果你想要权威的文档，你有两个选择。首先，你可以查看 API 的标准概述：

+   Firebug 首先实现了控制台 API，其在其维基中的[文档](http://bit.ly/1fwq1vk)是目前最接近标准的东西。

+   此外，Brian Kardell 和 Paul Irish 正在制定[API 规范](http://bit.ly/1fwq7mX)，这应该会导致更一致的行为。

其次，你可以查看各种引擎的文档：

+   [Chrome](https://developers.google.com/chrome-developer-tools/docs/console-api/)

+   [Firebug](https://getfirebug.com/wiki/index.php/Console_API)

+   [Firefox](https://developer.mozilla.org/en-US/docs/Web/API/console)

+   [Internet Explorer](http://msdn.microsoft.com/en-us/library/ie/hh772183.aspx)

+   [Node.js](http://nodejs.org/api/stdio.html)

+   [Safari](http://bit.ly/1fwq9er)

### 警告

在 Internet Explorer 9 中存在一个错误。在该浏览器中，只有开发者工具至少打开过一次，`console`对象才存在。这意味着如果在工具打开之前引用`console`，你会得到一个`ReferenceError`。作为一种解决方法，你可以检查`console`是否存在，如果不存在则创建一个虚拟实现。

### 简单的日志记录

控制台 API 包括以下记录方法：

`console.clear()`

清除控制台。

`console.debug(object1, object2?, ...)`

最好使用`console.log()`，它与此方法相同。

`console.error(object1, object2?, ...)`

将参数记录到控制台。在浏览器中，记录的内容可能会被“错误”图标标记，和/或包括堆栈跟踪或代码链接。

`console.exception(errorObject, object1?, ...])` [仅限 Firebug]

记录`object1`等，并显示交互式堆栈跟踪。

`console.info(object1?, object2?, ...)`

将参数记录到控制台。在浏览器中，记录的内容可能会被“信息”图标标记，和/或包括堆栈跟踪或代码链接。

`console.log(object1?, object2?, ...)`

将参数记录到控制台。如果第一个参数是`printf`风格的格式字符串，则使用它来打印其余的参数。例如（Node.js REPL）：

```js
> console.log('%s', { foo: 'bar' })
[object Object]
> console.log('%j', { foo: 'bar' })
{"foo":"bar"}
```

唯一可靠的跨平台格式化指令是`%s`。Node.js 支持`%j`以将数据格式化为 JSON；浏览器倾向于支持记录交互内容的指令。

`console.trace()`

记录堆栈跟踪（在许多浏览器中是交互式的）。

`console.warn(object1?, object2?, ...)`

将参数记录到控制台。在浏览器中，记录的内容可能会被“警告”图标标记，和/或包括堆栈跟踪或代码链接。

在以下表中指出了各种平台的支持：

|  | Chrome | Firebug | Firefox | IE | Node.js | Safari |
| --- | --- | --- | --- | --- | --- | --- |
| `clear` | ✓ | ✓ |  | ✓ |  | ✓ |
| `debug` | ✓ | ✓ | ✓ | ✓ |  | ✓ |
| `error` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| *`exception`* |  | ✓ |  |  |  |  |
| `info` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `log` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `trace` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `warn` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

`exception`以斜体排版，因为它只在单个平台上受支持。

### 检查和计数

控制台 API 包括以下检查和计数方法：

`console.assert(expr, obj?)`

如果`expr`为`false`，则将`obj`记录到控制台并抛出异常。如果为`true`，则什么也不做。

`console.count(label?)`

计算带有此语句的行被执行的次数。

在以下表中指出了各种平台的支持：

|  | Chrome | Firebug | Firefox | IE | Node.js | Safari |
| --- | --- | --- | --- | --- | --- | --- |
| `assert` | ✓ | ✓ |  | ✓ | ✓ | ✓ |
| `count` | ✓ | ✓ |  | ✓ |  | ✓ |

### 格式化日志

控制台 API 包括以下格式化日志的方法：

`console.dir(object)`

将对象的表示打印到控制台。在浏览器中，该表示可以交互地进行探索。

`console.dirxml(object)`

打印 HTML 或 XML 元素的 XML 源树。

`console.group(object1?, object2?, ...)`

将对象记录到控制台并打开一个包含所有未来记录内容的嵌套块。通过调用`console.groupEnd()`来关闭该块。该块最初是展开的，但可以折叠。

`console.groupCollapsed(object1?, object2?, ...)`

类似于`console.group()`，但是该块最初是折叠的。

`console.groupEnd()`

关闭由`console.group()`或`console.groupCollapsed()`打开的组。

`console.table(data, columns?)`

将数组打印为表格，每行一个元素。可选参数`columns`指定在列中显示哪些属性/数组索引。如果缺少该参数，则所有属性键都将用作表格列。缺少的属性和数组元素显示为列中的`undefined`：

```js
var persons = [
    { firstName: 'Jane', lastName: 'Bond' },
    { firstName: 'Lars', lastName: 'Croft', age: 72 }
];
// Equivalent:
console.table(persons);
console.table(persons, ['firstName', 'lastName', 'age']);
```

结果表如下：

| (索引) | 名字 | 姓氏 | 年龄 |
| --- | --- | --- | --- |
| 0 | `Jane` | `Bond` | undefined |
| 1 | `Lars` | `Croft` | 72 |

在以下表中指出了各种平台的支持：

|  | Chrome | Firebug | Firefox | IE | Node.js | Safari |
| --- | --- | --- | --- | --- | --- | --- |
| `dir` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `dirxml` | ✓ | ✓ |  | ✓ |  | ✓ |
| `group` | ✓ | ✓ | ✓ | ✓ |  | ✓ |
| `groupCollapsed` | ✓ | ✓ | ✓ | ✓ |  | ✓ |
| `groupEnd` | ✓ | ✓ | ✓ | ✓ |  | ✓ |
| `table` | ✓ | ✓ |  |  |  |  |

### 分析和计时

控制台 API 包括以下用于分析和计时的方法：

`控制台.标记时间线（标签）` [仅限 Safari]

与`console.timeStamp`相同。

控制台.性能（标题？）

打开分析。可选的`title`用于分析报告。

`控制台.分析结束()`

停止分析并打印分析报告。

`控制台.时间（标签）`

启动标签为`label`的计时器。

控制台.时间结束（标签）

停止标签为`label`的计时器并打印自启动以来经过的时间。

控制台.时间戳（标签？）

记录具有给定`label`的时间戳。可以记录到控制台或时间轴。

以下表格显示了各种平台上的支持：

|  | Chrome | Firebug | Firefox | IE | Node.js | Safari |
| --- | --- | --- | --- | --- | --- | --- |
| *`markTimeline`* |  |  |  |  |  | ✓ |
| `profile` | ✓ | ✓ | (devtools) | ✓ |  | ✓ |
| `profileEnd` | ✓ | ✓ | (devtools) | ✓ |  | ✓ |
| `time` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `timeEnd` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `timeStamp` | ✓ | ✓ |  |  |  |  |

`markTimeline`以斜体排版，因为它仅在单个平台上受支持。 （devtools）表示必须打开开发人员工具才能使该方法起作用。¹⁷

## 命名空间和特殊值

以下全局变量用作函数的命名空间。有关详细信息，请参阅括号中指示的材料：

`JSON`

JSON API 功能（[第二十二章](ch22.html "第二十二章.JSON"））

`数学`

数学 API 功能（[第二十一章](ch21.html "第二十一章.数学"））

`对象`

元编程功能（[对象操作小抄：使用对象](ch17_split_001.html#oop_cheat_sheet "对象操作小抄：使用对象"））

以下全局变量包含特殊值。有关更多信息，请查看括号中指示的材料：

`未定义`

表示某物不存在的值（[未定义和 null](ch08.html#undefined_null "未定义和 null"）：

```js
> ({}.foo) === undefined
true
```

`NaN`

一个表示某物是“非数字”（[NaN](ch11.html#nan "NaN"）的值：

```js
> 1 / 'abc'
NaN
```

`无穷大`

表示数值无穷大∞的值（[无穷大](ch11.html#infinity "无穷大"）：

```js
> 1 / 0
Infinity
```

* * *

¹⁶ Mariusz Nowak（@medikoo）告诉我，由`Function`评估的代码默认情况下在任何地方都是松散的。

¹⁷ 感谢 Matthias Reuter（@gweax）和 Philipp Kyeck（@pkyeck）对本节的贡献。

