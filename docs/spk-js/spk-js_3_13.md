# 第十九章：正则表达式

> 原文：[19. Regular Expressions](https://exploringjs.com/es5/ch19.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章概述了正则表达式的 JavaScript API。它假定你对它们的工作原理有一定了解。如果你不了解，网上有很多好的教程。其中两个例子是：

+   [Regular-Expressions.info](http://www.regular-expressions.info/) by Jan Goyvaerts

+   [Cody Lindley 的 JavaScript 正则表达式启示](http://bit.ly/1fwoQMs)

## 正则表达式语法

这里使用的术语与 ECMAScript 规范中的语法非常接近。我有时会偏离以使事情更容易理解。

### 原子：一般

一般原子的语法如下：

特殊字符

以下所有字符都具有特殊含义：

```js
\ ^ $ . * + ? ( ) [ ] { } |
```

你可以通过在前面加上反斜杠来转义它们。例如：

```js
> /^(ab)$/.test('(ab)')
false
> /^\(ab\)$/.test('(ab)')
true
```

其他特殊字符包括：

+   在字符类`[...]`中：

    ```js
    -
    ```

+   在以问号`(?...)`开头的组内：

    ```js
    : = ! < >
    ```

尖括号仅由 XRegExp 库（参见第三十章）使用，用于命名组。

模式字符

除了前面提到的特殊字符之外，所有字符都与它们自己匹配。

`.`（点）

匹配任何 JavaScript 字符（UTF-16 代码单元），除了行终止符（换行符、回车符等）。要真正匹配任何字符，请使用`[\s\S]`。例如：

```js
> /./.test('\n')
false
> /[\s\S]/.test('\n')
true
```

字符转义（匹配单个字符）

+   特定的控制字符包括`\f`（换页符）、`\n`（换行符、新行）、`\r`（回车符）、`\t`（水平制表符）和`\v`（垂直制表符）。

+   `\0`匹配 NUL 字符（`\u0000`）。

+   任何控制字符：`\cA` – `\cZ`。

+   Unicode 字符转义：`\u0000` – `\xFFFF`（Unicode 代码单元；参见第二十四章）。

+   十六进制字符转义：`\x00` – `\xFF`。

字符类转义（匹配一组字符中的一个）

+   数字：`\d`匹配任何数字（与`[0-9]`相同）；`\D`匹配任何非数字（与`[^0-9]`相同）。

+   字母数字字符：`\w`匹配任何拉丁字母数字字符加下划线（与`[A-Za-z0-9_]`相同）；`\W`匹配所有未被`\w`匹配的字符。

+   空白：`\s`匹配空白字符（空格、制表符、换行符、回车符、换页符、所有 Unicode 空格等）；`\S`匹配所有非空白字符。

### 原子：字符类

字符类的语法如下：

+   `[«charSpecs»]`匹配至少一个`charSpecs`中的任何一个的单个字符。

+   `[^«charSpecs»]`匹配任何不匹配`charSpecs`中任何一个的单个字符。

以下构造都是字符规范：

+   源字符匹配它们自己。大多数字符都是源字符（甚至许多在其他地方是特殊的字符）。只有三个字符不是：

    ```js
        \ ] -
    ```

像往常一样，您可以通过反斜杠进行转义。如果要匹配破折号而不进行转义，它必须是方括号打开后的第一个字符，或者是范围的右侧，如下所述。

+   类转义：允许使用先前列出的任何字符转义和字符类转义。还有一个额外的转义：

+   退格（`\b`）：在字符类之外，`\b`匹配单词边界。在字符类内，它匹配控制字符*退格*。

+   范围包括源字符或类转义，后跟破折号（`-`），后跟源字符或类转义。

为了演示使用字符类，此示例解析了按照 ISO 8601 标准格式化的日期：

```js
function parseIsoDate(str) {
    var match = /^([0-9]{4})-([0-9]{2})-([0-9]{2})$/.exec(str);

    // Other ways of writing the regular expression:
    // /^([0-9][0-9][0-9][0-9])-([0-9][0-9])-([0-9][0-9])$/
    // /^(\d\d\d\d)-(\d\d)-(\d\d)$/

    if (!match) {
        throw new Error('Not an ISO date: '+str);
    }
    console.log('Year: '  + match[1]);
    console.log('Month: ' + match[2]);
    console.log('Day: '   + match[3]);
}
```

以下是交互：

```js
> parseIsoDate('2001-12-24')
Year: 2001
Month: 12
Day: 24
```

### 原子：组

组的语法如下：

+   `(«pattern»)`是一个捕获组。由`pattern`匹配的任何内容都可以通过反向引用或作为匹配操作的结果来访问。

+   `(?:«pattern»)`是一个非捕获组。`pattern`仍然与输入匹配，但不保存为捕获。因此，该组没有您可以引用的编号（例如，通过反向引用）。

`\1`、`\2`等被称为*反向引用*；它们指回先前匹配的组。反斜杠后面的数字可以是大于或等于 1 的任何整数，但第一个数字不能是 0。

在此示例中，反向引用保证了破��号之前和之后的 a 的数量相同：

```js
> /^(a+)-\1$/.test('a-a')
true
> /^(a+)-\1$/.test('aaa-aaa')
true
> /^(a+)-\1$/.test('aa-a')
false
```

此示例使用反向引用来匹配 HTML 标签（显然，通常应使用适当的解析器来处理 HTML）：

```js
> var tagName = /<([^>]+)>[^<]*<\/\1>/;
> tagName.exec('<b>bold</b>')[1]
'b'
> tagName.exec('<strong>text</strong>')[1]
'strong'
> tagName.exec('<strong>text</stron>')
null
```

### 量词

任何原子（包括字符类和组）都可以后跟一个量词：

+   `?`表示匹配零次或一次。

+   `*`表示匹配零次或多次。

+   `+`表示匹配一次或多次。

+   `{n}`表示精确匹配`n`次。

+   `{n,}`表示匹配`n`次或更多次。

+   `{n,m}`表示至少匹配`n`次，最多匹配`m`次。

默认情况下，量词是*贪婪*的；也就是说，它们尽可能多地匹配。您可以通过在任何前述量词（包括大括号中的范围）后加上问号（`?`）来获得*勉强*匹配（尽可能少）。例如：

```js
> '<a> <strong>'.match(/^<(.*)>/)[1]  // greedy
'a> <strong'
> '<a> <strong>'.match(/^<(.*?)>/)[1]  // reluctant
'a'
```

因此，`.*?`是一个用于匹配直到下一个原子出现的有用模式。例如，以下是刚刚显示的 HTML 标签的正则表达式的更紧凑版本（使用`[^<]*`代替`.*?`）：

```js
/<(.+?)>.*?<\/\1>/
```

### 断言

断言，如下列表所示，是关于输入中当前位置的检查：

| `^` | 仅在输入的开头匹配。 |
| --- | --- |
| `$` | 仅在输入的末尾匹配。 |
| `\b` | 仅在单词边界处匹配。不要与`[\b]`混淆，它匹配退格。 |
| `\B` | 仅当不在单词边界时匹配。 |
| `(?=«pattern»)` | 正向预查：仅当“模式”匹配接下来的内容时才匹配。“模式”仅用于向前查看，否则会被忽略。 |
| `(?!«pattern»)` | 负向预查：仅当“模式”不匹配接下来的内容时才匹配。“模式”仅用于向前查看，否则会被忽略。 |

此示例通过`\b`匹配单词边界：

```js
> /\bell\b/.test('hello')
false
> /\bell\b/.test('ello')
false
> /\bell\b/.test('ell')
true
```

此示例通过`\B`匹配单词内部：

```js
> /\Bell\B/.test('ell')
false
> /\Bell\B/.test('hell')
false
> /\Bell\B/.test('hello')
true
```

### 注意

不支持后行断言。[手动实现后行断言](ch19.html#regexp-look-behind "手动实现后行断言")解释了如何手动实现它。

### 分歧

分歧运算符（`|`）分隔两个选择；任一选择必须匹配分歧才能匹配。这些选择是原子（可选包括量词）。

该运算符的绑定非常弱，因此您必须小心，以确保选择不会延伸太远。例如，以下正则表达式匹配所有以`'aa'`开头或以`'bb'`结尾的字符串：

```js
> /^aa|bb$/.test('aaxx')
true
> /^aa|bb$/.test('xxbb')
true
```

换句话说，分歧比甚至`^`和`$`都要弱，两个选择是`^aa`和`bb$`。如果要匹配两个字符串`'aa'`和`'bb'`，则需要括号：

```js
/^(aa|bb)$/
```

同样，如果要匹配字符串`'aab'`和`'abb'`：

```js
/^a(a|b)b$/
```

## Unicode 和正则表达式

JavaScript 的正则表达式对 Unicode 的支持非常有限。特别是当涉及到星际飞船中的代码点时，您必须小心。第二十四章解释了详细信息。

## 创建正则表达式

您可以通过文字或构造函数创建正则表达式，并通过标志配置其工作方式。

### 文字与构造函数

有两种方法可以创建正则表达式：您可以使用文字或构造函数`RegExp`：

文字：`/xyz/i`：在加载时编译

构造函数（第二个参数是可选的）：`new RegExp('xyz'，'i')`：在运行时编译

文字和构造函数在编译时不同：

+   文字在加载时编译。在评估时，以下代码将引发异常：

    ```js
    function foo() {
        /[/;
    }
    ```

+   构造函数在调用时编译正则表达式。以下代码不会引发异常，但调用`foo()`会：

    ```js
    function foo() {
        new RegExp('[');
    }
    ```

因此，通常应使用文字，但如果要动态组装正则表达式，则需要构造函数。

### 标志

标志是正则表达式文字的后缀和正则表达式构造函数的参数；它们修改正则表达式的匹配行为。存在以下标志：

| 短名称 | 长名称 | 描述 |
| --- | --- | --- |
| `g` | 全局 | 给定的正则表达式多次匹配。影响几种方法，特别是`replace()`。 |
| `i` | 忽略大小写 | 在尝试匹配给定的正则表达式时忽略大小写。 |
| `m` | 多行模式 | 在多行模式下，开始运算符^和结束运算符$匹配每一行，而不是完整的输入字符串。|

短名称用于文字前缀和构造函数参数（请参见下一节中的示例）。长名称用于正则表达式的属性，指示在创建期间设置了哪些标志。

### 正则表达式的实例属性

正则表达式具有以下实例属性：

+   标志：表示设置了哪些标志的布尔值：

+   全局：标志`/g`设置了吗？

+   忽略大小写：标志`/i`设置了吗？

+   多行：标志`/m`设置了吗？

+   用于多次匹配的数据（设置了`/g`标志）：

+   `lastIndex`是下次继续搜索的索引。

以下是访问标志的实例属性的示例：

```js
> var regex = /abc/i;
> regex.ignoreCase
true
> regex.multiline
false
```

### 创建正则表达式的示例

在这个例子中，我们首先使用文字创建相同的正则表达式，然后使用构造函数，并使用`test()`方法来确定它是否匹配一个字符串：

```js
> /abc/.test('ABC')
false
> new RegExp('abc').test('ABC')
false
```

在这个例子中，我们创建一个忽略大小写的正则表达式（标志`/i`）：

```js
> /abc/i.test('ABC')
true
> new RegExp('abc', 'i').test('ABC')
true
```

## RegExp.prototype.test：是否有匹配？

`test()`方法检查正则表达式`regex`是否匹配字符串`str`：

```js
regex.test(str)
```

`test()`的操作方式取决于标志`/g`是否设置。

如果标志`/g`未设置，则该方法检查`str`中是否有匹配。例如：

```js
> var str = '_x_x';

> /x/.test(str)
true
> /a/.test(str)
false
```

如果设置了标志`/g`，则该方法返回`true`，直到`str`中`regex`的匹配次数。属性`regex.lastIndex`包含最后匹配后的索引：

```js
> var regex = /x/g;
> regex.lastIndex
0

> regex.test(str)
true
> regex.lastIndex
2

> regex.test(str)
true
> regex.lastIndex
4

> regex.test(str)
false
```

## String.prototype.search：有匹配的索引吗？

`search()`方法在`str`中查找与`regex`匹配的内容：

```js
str.search(regex)
```

如果有匹配，返回找到匹配的索引。否则，结果为`-1`。`regex`的`global`和`lastIndex`属性在执行搜索时被忽略（`lastIndex`不会改变）。

例如：

```js
> 'abba'.search(/b/)
1
> 'abba'.search(/x/)
-1
```

如果`search()`的参数不是正则表达式，则会转换为正则表达式：

```js
> 'aaab'.search('^a+b+$')
0
```

## RegExp.prototype.exec：捕获组

以下方法调用在匹配`regex`和`str`时捕获组：

```js
var matchData = regex.exec(str);
```

如果没有匹配，`matchData`为`null`。否则，`matchData`是一个*匹配结果*，一个带有两个额外属性的数组：

数组元素

+   元素 0 是完整正则表达式的匹配（如果愿意的话，是第 0 组）。

+   元素*n* > 1 是第*n*组的捕获。

属性

+   `input`是完整的输入字符串。

+   `index`是找到匹配的索引。

### 第一个匹配（标志/g 未设置）

如果标志`/g`未设置，则只返回第一个匹配：

```js
> var regex = /a(b+)/;
> regex.exec('_abbb_ab_')
[ 'abbb',
  'bbb',
  index: 1,
  input: '_abbb_ab_' ]
> regex.lastIndex
0
```

### 所有匹配（标志/g 设置）

如果设置了标志`/g`，则如果反复调用`exec()`，所有匹配都会被返回。返回值`null`表示没有更多的匹配。属性`lastIndex`指示下次匹配将继续的位置：

```js
> var regex = /a(b+)/g;
> var str = '_abbb_ab_';

> regex.exec(str)
[ 'abbb',
  'bbb',
  index: 1,
  input: '_abbb_ab_' ]
> regex.lastIndex
6

> regex.exec(str)
[ 'ab',
  'b',
  index: 7,
  input: '_abbb_ab_' ]
> regex.lastIndex
10

> regex.exec(str)
null
```

在这里我们循环匹配：

```js
var regex = /a(b+)/g;
var str = '_abbb_ab_';
var match;
while (match = regex.exec(str)) {
    console.log(match[1]);
}
```

我们得到以下输出：

```js
bbb
b
```

## String.prototype.match：捕获组或返回所有匹配的子字符串

以下方法调用匹配`regex`和`str`：

```js
var matchData = str.match(regex);
```

如果`regex`的标志`/g`未设置，此方法的工作方式类似于`RegExp.prototype.exec()`：

```js
> 'abba'.match(/a/)
[ 'a', index: 0, input: 'abba' ]
```

如果设置了标志，则该方法返回一个包含`str`中所有匹配子字符串的数组（即每次匹配的第 0 组），如果没有匹配则返回`null`：

```js
> 'abba'.match(/a/g)
[ 'a', 'a' ]
> 'abba'.match(/x/g)
null
```

## String.prototype.replace：搜索和替换

`replace()`方法搜索字符串`str`，找到与`search`匹配的内容，并用`replacement`替换它们：

```js
str.replace(search, replacement)
```

有几种方式可以指定这两个参数：

`search`

可以是字符串或正则表达式：

+   字符串：在输入字符串中直接查找。请注意，只有第一次出现的字符串会被替换。如果要替换多个出现，必须使用带有`/g`标志的正则表达式。这是一个意外和一个主要的陷阱。

+   正则表达式：与输入字符串匹配。警告：使用`global`标志，否则只会尝试一次匹配正则表达式。

`replacement`

可以是字符串或函数：

+   字符串：描述如何替换已找到的内容。

+   功能：通过参数提供匹配信息来计算替换。

### 替换是一个字符串

如果`replacement`是一个字符串，它的内容将被直接使用来替换匹配。唯一的例外是特殊字符美元符号（`$`），它启动所谓的*替换指令*：

+   组：`$n`插入匹配中的第 n 组。`n`必须至少为 1（`$0`没有特殊含义）。

+   匹配的子字符串：

+   `$``（反引号）插入匹配前的文本。

+   `$&`插入完整的匹配。

+   `$'`（撇号）插入匹配后的文本。

+   `$$`插入一个单独的`$`。

这个例子涉及匹配的子字符串及其前缀和后缀：

```js
> 'axb cxd'.replace(/x/g, "[$`,$&,$']")
'a[a,x,b cxd]b c[axb c,x,d]d'
```

这个例子涉及到一个组：

```js
> '"foo" and "bar"'.replace(/"(.*?)"/g, '#$1#')
'#foo# and #bar#'
```

### 替换是一个函数

如果`replacement`是一个函数，它会计算要替换匹配的字符串。此函数具有以下签名：

```js
function (completeMatch, group_1, ..., group_n, offset, inputStr)
```

`completeMatch`与以前的`$&`相同，`offset`指示找到匹配的位置，`inputStr`是正在匹配的内容。因此，您可以使用特殊变量`arguments`来访问组（通过`arguments[1]`访问第 1 组，依此类推）。例如：

```js
> function replaceFunc(match) { return 2 * match }
> '3 apples and 5 oranges'.replace(/[0-9]+/g, replaceFunc)
'6 apples and 10 oranges'
```

## 标志`/g`的问题

设置了`/g`标志的正则表达式如果必须多次调用才能返回所有结果，则存在问题。这适用于两种方法：

+   `RegExp.prototype.test()`

+   `RegExp.prototype.exec()`

然后 JavaScript 滥用正则表达式作为迭代器，作为结果序列的指针。这会导致问题：

问题 1：无法内联`/g`正则表达式

例如：

```js
// Don’t do that:
var count = 0;
while (/a/g.test('babaa')) count++;
```

前面的循环是无限的，因为每次循环迭代都会创建一个新的正则表达式，从而重新开始结果的迭代。因此，必须重写代码：

```js
var count = 0;
var regex = /a/g;
while (regex.test('babaa')) count++;
```

这是另一个例子：

```js
// Don’t do that:
function extractQuoted(str) {
    var match;
    var result = [];
    while ((match = /"(.*?)"/g.exec(str)) != null) {
        result.push(match[1]);
    }
    return result;
}
```

调用前面的函数将再次导致无限循环。正确的版本是（为什么`lastIndex`设置为 0 很快就会解释）：

```js
var QUOTE_REGEX = /"(.*?)"/g;
function extractQuoted(str) {
    QUOTE_REGEX.lastIndex = 0;
    var match;
    var result = [];
    while ((match = QUOTE_REGEX.exec(str)) != null) {
        result.push(match[1]);
    }
    return result;
}
```

使用该函数：

```js
> extractQuoted('"hello", "world"')
[ 'hello', 'world' ]
```

### 提示

最好的做法是不要内联（然后您可以给正则表达式起一个描述性的名称）。但是您必须意识到您不能这样做，即使是在快速的 hack 中也不行。

问题 2：`/g`正则表达式作为参数

调用`test()`和`exec()`多次的代码在作为参数传递给它的正则表达式时必须小心。它的标志`/g`必须激活，并且为了安全起见，它的`lastIndex`应该设置为零（下一个示例中提供了解释）。

问题 3：共享的`/g`正则表达式（例如，常量）

每当引用尚未新创建的正则表达式时，您应该在将其用作迭代器之前将其`lastIndex`属性设置为零（下一个示例中提供了解释）。由于迭代取决于`lastIndex`，因此这样的正则表达式不能同时在多个迭代中使用。

以下示例说明了问题 2。这是一个简单的实现函数，用于计算字符串`str`中正则表达式`regex`的匹配次数：

```js
// Naive implementation
function countOccurrences(regex, str) {
    var count = 0;
    while (regex.test(str)) count++;
    return count;
}
```

以下是使用此函数的示例：

```js
> countOccurrences(/x/g, '_x_x')
2
```

第一个问题是，如果正则表达式的`/g`标志未设置，此函数将进入无限循环。例如：

```js
countOccurrences(/x/, '_x_x') // never terminates
```

第二个问题是，如果`regex.lastIndex`不是 0，函数将无法正确工作，因为该属性指示从哪里开始搜索。例如：

```js
> var regex = /x/g;
> regex.lastIndex = 2;
> countOccurrences(regex, '_x_x')
1
```

以下实现解决了两个问题：

```js
function countOccurrences(regex, str) {
    if (! regex.global) {
        throw new Error('Please set flag /g of regex');
    }
    var origLastIndex = regex.lastIndex;  // store
    regex.lastIndex = 0;

    var count = 0;
    while (regex.test(str)) count++;

    regex.lastIndex = origLastIndex;  // restore
    return count;
}
```

一个更简单的替代方法是使用`match()`：

```js
function countOccurrences(regex, str) {
    if (! regex.global) {
        throw new Error('Please set flag /g of regex');
    }
    return (str.match(regex) || []).length;
}
```

有一个可能的陷阱：如果设置了`/g`标志并且没有匹配项，`str.match()`将返回`null`。在前面的代码中，我们通过使用`[]`来避免这种陷阱，如果`match()`的结果不是真值。

## 提示和技巧

本节提供了一些在 JavaScript 中使用正则表达式的技巧和窍门。

### 引用文本

有时，当手动组装正则表达式时，您希望逐字使用给定的字符串。这意味着不能解释任何特殊字符（例如，`*`，`[`）-所有这些字符都需要转义。JavaScript 没有内置的方法来进行这种引用，但是您可以编写自己的函数`quoteText`，它将按以下方式工作：

```js
> console.log(quoteText('*All* (most?) aspects.'))
\*All\* \(most\?\) aspects\.
```

如果您需要进行多次搜索和替换，则此函数特别方便。然后要搜索的值必须是设置了`global`标志的正则表达式。使用`quoteText()`，您可以使用任意字符串。该函数如下所示：

```js
function quoteText(text) {
    return text.replace(/[\\^$.*+?()[\]{}|=!<>:-]/g, '\\$&');
}
```

所有特殊字符都被转义，因为您可能希望在括号或方括号内引用多个字符。

### 陷阱：没有断言（例如，^，$），正则表达式可以在任何地方找到

如果您不使用`^`和`$`等断言，大多数正则表达式方法会在任何地方找到模式。例如：

```js
> /aa/.test('xaay')
true
> /^aa$/.test('xaay')
false
```

### 匹配一切或什么都不匹配

这是一个罕见的用例，但有时您需要一个正则表达式，它匹配一切或什么都不匹配。例如，一个函数可能有一个用于过滤的正则表达式参数。如果该参数缺失，您可以给它一个默认值，一个匹配一切的正则表达式。

#### 匹配一切

空正则表达式匹配一切。我们可以基于该正则表达式创建一个`RegExp`的实例，就像这样：

```js
> new RegExp('').test('dfadsfdsa')
true
> new RegExp('').test('')
true
```

但是，空正则表达式文字将是`//`，这在 JavaScript 中被解释为注释。因此，以下是通过文字获得的最接近的：`/(?:)/`（空的非捕获组）。该组匹配一切，同时不捕获任何内容，这个组不会影响`exec()`返回的结果。即使 JavaScript 本身在显示空正则表达式时也使用前面的表示：

```js
> new RegExp('')
/(?:)/
```

#### 匹配什么都不匹配

空正则表达式具有一个反义词——匹配什么都不匹配的正则表达式：

```js
> var never = /.^/;
> never.test('abc')
false
> never.test('')
false
```

### 手动实现后行断言

后行断言是一种断言。与先行断言类似，模式用于检查输入中当前位置的某些内容，但在其他情况下被忽略。与先行断言相反，模式的匹配必须*结束*在当前位置（而不是从当前位置开始）。

以下函数将字符串`'NAME'`的每个出现替换为参数`name`的值，但前提是该出现不是由引号引导的。我们通过“手动”检查当前匹配之前的字符来处理引号：

```js
function insertName(str, name) {
    return str.replace(
        /NAME/g,
        function (completeMatch, offset) {
            if (offset === 0 ||
                (offset > 0 && str[offset-1] !== '"')) {
                return name;
            } else {
                return completeMatch;
            }
        }
    );
}
```

```js
> insertName('NAME "NAME"', 'Jane')
'Jane "NAME"'
> insertName('"NAME" NAME', 'Jane')
'"NAME" Jane'
```

另一种方法是在正则表达式中包含可能转义的字符。然后，您必须临时向您正在搜索的字符串添加前缀；否则，您将错过该字符串开头的匹配：

```js
function insertName(str, name) {
    var tmpPrefix = ' ';
    str = tmpPrefix + str;
    str = str.replace(
        /([^"])NAME/g,
        function (completeMatch, prefix) {
            return prefix + name;
        }
    );
    return str.slice(tmpPrefix.length); // remove tmpPrefix
}
```

## 正则表达式速查表

原子（参见[原子：一般](ch19.html#regex_atoms_general "原子：一般")）：

+   `.`（点）匹配除行终止符（例如换行符）之外的所有内容。使用`[\s\S]`来真正匹配一切。

+   字符类转义：

+   `\d`匹配数字（`[0-9]`）；`\D`匹配非数字（`[^0-9]`）。

+   `\w`匹配拉丁字母数字字符加下划线（`[A-Za-z0-9_]`）；`\W`匹配所有其他字符。

+   `\s`匹配所有空白字符（空格、制表符、换行符等）；`\S`匹配所有非空白字符。

+   字符类（字符集）：`[...]`和`[^...]`

+   源字符：`[abc]`（除`\ ] -`之外的所有字符都与它们自身匹配）

+   字符类转义（参见前文）：`[\d\w]`

+   范围：`[A-Za-z0-9]`

+   组：

+   捕获组：`(...)`；反向引用：`\1`

+   非捕获组：`(?:...)`

量词（参见[量词](ch19.html#regexp_quantifiers "量词")）：

+   贪婪：

+   `? * +`

+   `{n} {n,} {n,m}`

+   勉强：在任何贪婪量词后面加上`?`。

断言（参见[断言](ch19.html#regexp_assertions "断言")）：

+   输入的开始，输入的结束：`^ $`

+   在词边界处，不在词边界处：`\b \B`

+   正向先行断言：`(?=...)`（模式必须紧跟其后，但在其他情况下被忽略）

+   负向先行断言：`(?!...)`（模式不能紧跟其后，但在其他情况下被忽略）

分支：`|`

创建正则表达式（参见[创建正则表达式](ch19.html#creating_regexps "创建正则表达式")）：

+   字面量：`/xyz/i`（在加载时编译）

+   构造函数：`new RegExp('xzy', 'i')`（在运行时编译）

标志（参见[标志](ch19.html#regexp_flags "标志")）：

+   全局：`/g`（影响几种正则表达式方法）

+   ignoreCase：`/i`

+   多行：`/m`（`^`和`$`按行匹配，而不是完整的输入）

方法：

+   `regex.test(str)`: 是否有匹配（参见[RegExp.prototype.test: 是否有匹配？](ch19.html#RegExp.prototype.test "RegExp.prototype.test: 是否有匹配？")）？

+   `/g`未设置：是否有匹配？

+   `/g`被设置：返回与匹配次数相同的`true`。

+   `str.search(regex)`: 有匹配项的索引是什么（参见[String.prototype.search: At What Index Is There a Match?](ch19.html#String.prototype.search "String.prototype.search: At What Index Is There a Match?")）？

+   `regex.exec(str)`: 捕获组（参见章节[RegExp.prototype.exec: Capture Groups](ch19.html#RegExp.prototype.exec "RegExp.prototype.exec: Capture Groups")）？

+   `/g`未设置：仅捕获第一个匹配项的组（仅调用一次）

+   `/g`已设置：捕获所有匹配项的组（重复调用；如果没有更多匹配项，则返回`null`）

+   `str.match(regex)`: 捕获组或返回所有匹配的子字符串（参见[String.prototype.match: Capture Groups or Return All Matching Substrings](ch19.html#String.prototype.match "String.prototype.match: Capture Groups or Return All Matching Substrings")）

+   `/g`未设置：捕获组

+   `/g`已设置：返回数组中所有匹配的子字符串

+   `str.replace(search, replacement)`: 搜索和替换（参见[String.prototype.replace: Search and Replace](ch19.html#String.prototype.replace "String.prototype.replace: Search and Replace")）

+   `search`：字符串或正则表达式（使用后者，设置`/g`！）

+   `replacement`：字符串（带有`$1`等）或函数（`arguments[1]`是第 1 组等），返回一个字符串

有关使用标志`/g`的提示，请参阅[Problems with the Flag /g](ch19.html#tips_flag_g "Problems with the Flag /g")。

### 致谢

Mathias Bynens（@mathias）和 Juan Ignacio Dopazo（@juandopazo）建议使用`match()`和`test()`来计算出现次数，Šime Vidas（@simevidas）警告我在没有匹配项时要小心使用`match()`。全局标志导致无限循环的陷阱来自[Andrea Giammarchi 的演讲](http://bit.ly/1fwpdXv)（@webreflection）。Claude Pache 告诉我在`quoteText()`中转义更多字符。

