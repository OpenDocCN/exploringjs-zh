# 第十二章：字符串

> 原文：[12. Strings](https://exploringjs.com/es5/ch12.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


字符串是 JavaScript 字符的不可变序列。每个字符都是一个 16 位的 UTF-16 代码单元。这意味着一个 Unicode 字符由一个或两个 JavaScript 字符表示。当您计算字符数或拆分字符串时，您主要需要考虑两个字符的情况（参见第二十四章）。

## 字符串文字

单引号和双引号都可以用来界定字符串文字：

```js
'He said: "Hello"'
"He said: \"Hello\""

'Everyone\'s a winner'
"Everyone's a winner"
```

因此，您可以自由地使用任何一种引号。不过，有几点需要考虑：

+   社区中最常见的风格是在 HTML 中使用双引号，在 JavaScript 中使用单引号。

+   另一方面，某些语言（例如 C 和 Java）中双引号专门用于字符串。因此，在多语言代码库中使用它们可能是有意义的。

+   对于 JSON（在第二十二章中讨论），您必须使用双引号。

如果您一贯使用引号，您的代码看起来会更整洁。但有时，不同的引号意味着您不必转义，这可以证明您不那么一致是合理的（例如，您可能通常使用单引号，但暂时切换到双引号来编写前面例子的最后一个）。

## 字符串文字中的转义

字符串文字中的大多数字符只是代表它们自己。反斜杠用于*转义*并启用了一些特殊功能：

行继续

您可以通过用反斜杠转义行尾（行终止字符，*行终止符*）来将字符串分布在多行上：

```js
var str = 'written \
over \
multiple \
lines';
console.log(str === 'written over multiple lines'); // true
```

另一种方法是使用加号运算符进行连接：

```js
var str = 'written ' +
          'over ' +
          'multiple ' +
          'lines';
```

字符转义序列

这些序列以反斜杠开头：

+   控制字符：`\b`是一个退格，`\f`是一个换页符，`\n`是一个换行符（新行），`\r`是一个回车，`\t`是一个水平制表符，`\v`是一个垂直制表符。

+   转义字符代表它们自己：`\'`是一个单引号，`\"`是一个双引号，`\\`是一个反斜杠。除了`b f n r t v x u`和十进制数字之外，所有字符也代表它们自己。以下是两个例子：

    ```js
    > '\"'
    '"'
    > '\q'
    'q'
    ```

NUL 字符（Unicode 代码点 0）

这个字符由`\0`表示。

十六进制转义序列

`\xHH`（`HH`是两个十六进制数字）指定了一个 ASCII 码的字符。例如：

```js
> '\x4D'
'M'
```

Unicode 转义序列

`\uHHHH`（`HHHH`是四个十六进制数字）指定了一个 UTF-16 代码单元（参见第二十四章）。以下是两个例子：

```js
> '\u004D'
'M'
> '\u03C0'
'π'
```

## 字符访问

有两个操作可以返回字符串的第*n*个字符。请注意，JavaScript 没有专门的字符数据类型；这些操作返回字符串：

```js
> 'abc'.charAt(1)
'b'
> 'abc'[1]
'b'
```

一些较旧的浏览器不支持通过方括号进行类似数组的字符访问。

## 转换为字符串

值将按以下方式转换为字符串：

| 值 | 结果 |
| --- | --- |
|  | `undefined` → `'undefined'` |
|  | `null` → `'null'` |
| 布尔值 | `false` → `'false'` |
|  | `true` → `'true'` |
| 数字 | 作为字符串的数字（例如，`3.141` → `'3.141'`） |
| 字符串 | 与输入相同（无需转换） |
| 对象 | 调用`ToPrimitive(value, String)`（请参阅[算法：ToPrimitive()——将值转换为原始值](ch08.html#toprimitive "算法：ToPrimitive()——将值转换为原始值")）并转换生成的原始值。 |

### 手动转换为字符串

将三种将任何值转换为字符串的最常见方法是：

| `String(value)` | （作为函数调用，而不是作为构造函数） |
| `''+value` |  |
| `value.toString()` | （对于`undefined`和`null`不起作用！） |

我更喜欢`String()`，因为它更具描述性。以下是一些示例：

```js
> String(false)
'false'
> String(7.35)
'7.35'
> String({ first: 'John', last: 'Doe' })
'[object Object]'
> String([ 'a', 'b', 'c' ])
'a,b,c'
```

请注意，对于显示数据，`JSON.stringify()`（[JSON.stringify(value, replacer?, space?)](ch22.html#JSON.stringify "JSON.stringify(value, replacer?, space?)")）通常比规范的字符串转换效果更好：

```js
> console.log(JSON.stringify({ first: 'John', last: 'Doe' }))
{"first":"John","last":"Doe"}
> console.log(JSON.stringify([ 'a', 'b', 'c' ]))
["a","b","c"]
```

当然，您必须意识到`JSON.stringify()`的局限性——它并不总是显示所有内容。例如，它隐藏了它无法处理的属性的值（函数等！）。另一方面，它的输出可以被`eval()`解析，并且可以将深度嵌套的数据显示为格式良好的树。

#### 陷阱：转换不可逆

考虑到 JavaScript 自动转换的频率，遗憾的是转换并不总是可逆的，特别是在布尔值方面：

```js
> String(false)
'false'
> Boolean('false')
true
```

对于`undefined`和`null`，我们面临类似的问题。

## 比较字符串

有两种比较字符串的方法。首先，您可以使用比较运算符：`<`，`>`，`===`，`<=`，`>=`。它们有以下缺点：

+   它们区分大小写：

    ```js
    > 'B' > 'A'  // ok
    true
    > 'B' > 'a'  // should be true
    false
    ```

+   它们不能很好地处理变音符和重音符号：

    ```js
    > 'ä' < 'b'  // should be true
    false
    > 'é' < 'f'  // should be true
    false
    ```

其次，您可以使用`String.prototype.localeCompare(other)`，这往往更好，但并不总是受支持（有关详细信息，请参阅[搜索和比较](ch12.html#String.prototype.localeCompare "搜索和比较")）。以下是 Firefox 控制台中的交互：

```js
> 'B'.localeCompare('A')
2
> 'B'.localeCompare('a')
2

> 'ä'.localeCompare('b')
-2
> 'é'.localeCompare('f')
-2
```

小于零的结果意味着接收器“小于”参数。大于零的结果意味着接收器“大于”参数。

## 连接字符串

有两种主要的字符串连接方法。

### 连接：加号（+）运算符

运算符`+`在其操作数之一是字符串时进行字符串连接。如果要在变量中收集字符串片段，则复合赋值运算符`+=`很有用：

```js
> var str = '';
> str += 'Say hello ';
> str += 7;
> str += ' times fast!';
> str
'Say hello 7 times fast!'
```

### 连接：连接字符串片段的数组

似乎以前的方法每次添加一个片段到`str`时都会创建一个新的字符串。旧的 JavaScript 引擎是这样做的，这意味着您可以通过首先将所有片段收集到一个数组中，然后作为最后一步连接它们来提高字符串连接的性能：

```js
> var arr = [];

> arr.push('Say hello ');
> arr.push(7);
> arr.push(' times fast');

> arr.join('')
'Say hello 7 times fast'
```

然而，较新的引擎通过`+`优化字符串连接，并在内部使用类似的方法。因此，在这些引擎上，加号运算符的速度更快。

## 字符串函数

函数`String`可以以两种方式调用：

`String(value)`

作为普通函数，它将`value`转换为原始字符串（请参阅[转换为字符串](ch12.html#tostring "转换为字符串")）：

```js
> String(123)
'123'
> typeof String('abc')  // no change
'string'
```

`new String(str)`

作为构造函数，它创建`String`的新实例（请参阅[原始值的包装对象](ch08.html#wrapper_objects "原始值的包装对象")），一个包装`str`的对象（非字符串被强制转换为字符串）。例如：

```js
> typeof new String('abc')
'object'
```

前一种调用是常见的。

## 字符串构造函数方法

`String.fromCharCode(codeUnit1, codeUnit2, ...)` 生成一个字符串，其字符由 16 位无符号整数`codeUnit1`，`codeUnit2`等指定的 UTF-16 代码单元组成。例如：

```js
> String.fromCharCode(97, 98, 99)
'abc'
```

如果要将数字数组转换为字符串，可以通过`apply()`（请参阅[func.apply(thisValue, argArray)](ch15.html#functional_apply "func.apply(thisValue, argArray")）来实现：

```js
> String.fromCharCode.apply(null, [97, 98, 99])
'abc'
```

`String.fromCharCode()`的反函数是`String.prototype.charCodeAt()`。

## 字符串实例属性长度

`length`属性指示字符串中的 JavaScript 字符数，并且是不可变的：

```js
> 'abc'.length
3
```

## 字符串原型方法

原始字符串的所有原始字符串方法都存储在`String.prototype`中（参见[原始通过包装器借用其方法](ch08.html#primitive_methods_via_wrappers "原始通过包装器借用其方法")）。接下来，我描述了它们如何用于原始字符串，而不是`String`的实例。

### 提取子字符串

以下方法从接收者中提取子字符串：

`String.prototype.charAt(pos)`

返回位置`pos`处的字符。例如：

```js
> 'abc'.charAt(1)
'b'
```

以下两个表达式返回相同的结果，但一些较旧的 JavaScript 引擎只支持使用`charAt()`来访问字符：

```js
str.charAt(n)
str[n]
```

`String.prototype.charCodeAt(pos)`

返回 JavaScript 字符（UTF-16 代码单元；参见第二十四章）在位置`pos`处的代码（一个 16 位无符号整数）。

这是如何创建字符代码数组的：

```js
> 'abc'.split('').map(function (x) { return x.charCodeAt(0) })
[ 97, 98, 99 ]
```

`charCodeAt()`的反函数是`String.fromCharCode()`。

`String.prototype.slice(start, end?)`

返回从位置`start`开始到位置`end`之前的子字符串。这两个参数都可以是负数，然后它们的长度将被添加到它们中：

```js
> 'abc'.slice(2)
'c'
> 'abc'.slice(1, 2)
'b'
> 'abc'.slice(-2)
'bc'
```

`String.prototype.substring(start, end?)`

应该避免使用`slice()`，它类似，但可以处理负位置，并且在各个浏览器中实现更一致。

`String.prototype.split(separator?, limit?)`

提取由`separator`分隔的接收者的子字符串，并将它们作为数组返回。该方法有两个参数：

+   `separator`：要么是一个字符串，要么是一个正则表达式。如果缺失，将返回完整的字符串，包裹在一个数组中。

+   `limit`：如果给定，返回的数组最多包含`limit`个元素。

以下是一些示例：

```js
> 'a,  b,c, d'.split(',')  // string
[ 'a', '  b', 'c', ' d' ]
> 'a,  b,c, d'.split(/,/)  // simple regular expression
[ 'a', '  b', 'c', ' d' ]
> 'a,  b,c, d'.split(/, */)   // more complex regular expression
[ 'a', 'b', 'c', 'd' ]
> 'a,  b,c, d'.split(/, */, 2)  // setting a limit
[ 'a', 'b' ]
> 'test'.split()  // no separator provided
[ 'test' ]
```

如果有一个组，那么匹配项也会作为数组元素返回：

```js
> 'a,  b  ,  '.split(/(,)/)
[ 'a', ',', '  b  ', ',', '  ' ]
> 'a,  b  ,  '.split(/ *(,) */)
[ 'a', ',', 'b', ',', '' ]
```

使用`''`（空字符串）作为分隔符，以产生一个包含字符串字符的数组：

```js
> 'abc'.split('')
[ 'a', 'b', 'c' ]
```

### 转换

前一节是关于提取子字符串，而这一节是关于将给定的字符串转换为新字符串。这些方法通常如下使用：

```js
var str = str.trim();
```

换句话说，原始字符串在（非破坏性地）转换后被丢弃：

`String.prototype.trim()`

从字符串的开头和结尾删除所有空格：

```js
> '\r\nabc \t'.trim()
'abc'
```

`String.prototype.concat(str1?, str2?, ...)`

返回接收者和`str1`、`str2`等的连接：

```js
> 'hello'.concat(' ', 'world', '!')
'hello world!'
```

`String.prototype.toLowerCase()`

创建一个新字符串，其中包含所有原始字符串的字符转换为小写：

```js
> 'MJÖLNIR'.toLowerCase()
'mjölnir'
```

`String.prototype.toLocaleLowerCase()`

与`toLowerCase()`相同，但遵守当前区域设置的规则。根据 ECMAScript 规范：“只有在少数情况下（如土耳其语）语言的规则与常规 Unicode 大小写映射冲突时才会有差异。”

`String.prototype.toUpperCase()`

创建一个新字符串，其中包含所有原始字符串的字符转换为大写：

```js
> 'mjölnir'.toUpperCase()
'MJÖLNIR'
```

`String.prototype.toLocaleUpperCase()`

与`toUpperCase()`相同，但遵守当前区域设置的规则。

### 搜索和比较

以下方法用于搜索和比较字符串：

`String.prototype.indexOf(searchString, position?)`

从`position`（默认为 0）开始搜索`searchString`。它返回`searchString`被找到的位置，或者-1（如果找不到）：

```js
> 'aXaX'.indexOf('X')
1
> 'aXaX'.indexOf('X', 2)
3
```

请注意，当涉及在字符串中查找文本时，正则表达式同样有效。例如，以下两个表达式是等价的：

```js
str.indexOf('abc') >= 0
/abc/.test(str)
```

`String.prototype.lastIndexOf(searchString, position?)`

从`position`（默认为末尾）开始向后搜索`searchString`。它返回`searchString`被找到的位置，或者-1（如果找不到）：

```js
> 'aXaX'.lastIndexOf('X')
3
> 'aXaX'.lastIndexOf('X', 2)
1
```

`String.prototype.localeCompare(other)`

对字符串与`other`进行区域敏感比较。它返回一个数字：

+   < 0 如果字符串在`other`之前

+   = 0 如果字符串等同于`other`

+   > 如果字符串在`other`之后

例如：

```js
> 'apple'.localeCompare('banana')
-2
> 'apple'.localeCompare('apple')
0
```

### 警告

并非所有 JavaScript 引擎都正确实现了这种方法。有些只是基于比较运算符。然而，ECMAScript 国际化 API（参见[ECMAScript 国际化 API](ch30.html#i18n_api "ECMAScript 国际化 API")）提供了一个基于 Unicode 的实现。也就是说，如果引擎中有这个 API，`localeCompare()`将起作用。

如果支持，`localeCompare()`比比较运算符更适合比较字符串。请参阅[比较字符串](ch12.html#comparing_strings "比较字符串")了解更多信息。

### 使用正则表达式进行测试、匹配和替换

以下方法适用于正则表达式：

`String.prototype.search(regexp)`（在[字符串原型搜索：有匹配的索引是什么？](ch19.html#String.prototype.search "字符串原型搜索：有匹配的索引是什么？")中更详细地解释）

返回`regexp`在接收者中匹配的第一个索引（如果没有匹配，则返回-1）：

```js
> '-yy-xxx-y-'.search(/x+/)
4
```

`String.prototype.match(regexp)`（在[字符串原型匹配：捕获组或返回所有匹配的子字符串](ch19.html#String.prototype.match "字符串原型匹配：捕获组或返回所有匹配的子字符串")中更详细地解释）

匹配给定的正则表达式与接收者。如果未设置`regexp`的标志`/g`，则返回第一个匹配的匹配对象：

```js
> '-abb--aaab-'.match(/(a+)b/)
[ 'ab',
  'a',
  index: 1,
  input: '-abb--aaab-' ]
```

如果标志`/g`被设置，那么所有完整的匹配（第 0 组）将以数组的形式返回：

```js
> '-abb--aaab-'.match(/(a+)b/g)
[ 'ab', 'aaab' ]
```

`String.prototype.replace(search, replacement)`（在[字符串原型替换：搜索和替换](ch19.html#String.prototype.replace "字符串原型替换：搜索和替换")中更详细地解释）

搜索`search`并用`replacement`替换它。`search`可以是一个字符串或一个正则表达式，`replacement`可以是一个字符串或一个函数。除非您使用一个设置了标志`/g`的正则表达式作为`search`，否则只会替换第一个出现的：

```js
> 'iixxxixx'.replace('i', 'o')
'oixxxixx'
> 'iixxxixx'.replace(/i/, 'o')
'oixxxixx'
> 'iixxxixx'.replace(/i/g, 'o')
'ooxxxoxx'
```

替换字符串中的美元符号（`$`）允许您引用完整的匹配或捕获的组：

```js
> 'iixxxixx'.replace(/i+/g, '($&)') // complete match
'(ii)xxx(i)xx'
> 'iixxxixx'.replace(/(i+)/g, '($1)') // group 1
'(ii)xxx(i)xx'
```

您还可以通过函数计算替换：

```js
> function repl(all) { return '('+all.toUpperCase()+')' }
> 'axbbyyxaa'.repl(/a+|b+/g, replacement)
'(A)x(BB)yyx(AA)'
```

* * *

¹⁴ 严格来说，JavaScript 字符串由一系列 UTF-16 代码单元组成。也就是说，JavaScript 字符是 Unicode 代码单元（参见第二十四章）。

