# 第二十四章： Unicode 和 JavaScript

> 原文：[24. Unicode and JavaScript](https://exploringjs.com/es5/ch24.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章是对 Unicode 及其在 JavaScript 中的处理的简要介绍。

## Unicode 历史

Unicode 始于 1987 年，由 Joe Becker（施乐），Lee Collins（苹果）和 Mark Davis（苹果）发起。其想法是创建一个通用字符集，因为当时对于编码纯文本存在许多不兼容的标准：许多变体的 8 位 ASCII，Big Five（繁体中文），GB 2312（简体中文）等。在 Unicode 之前，没有多语言纯文本的标准，但有丰富的文本系统（例如苹果的 WorldScript），允许您组合多个编码。

第一份 Unicode 草案提案于 1988 年发布。此后继续工作并扩大工作组。[*Unicode 联盟*](http://www.unicode.org/consortium/consort.html)于 1991 年 1 月 3 日成立：

> Unicode 联盟是一家致力于开发、维护和推广软件国际化标准和数据的非营利性公司，特别是 Unicode 标准[...]

Unicode 1.0 标准的第一卷于 1991 年 10 月出版，第二卷于 1992 年 6 月出版。

## 重要的 Unicode 概念

字符的概念可能看起来很简单，但它有许多方面。这就是为什么 Unicode 是一个如此复杂的标准。以下是重要的基本概念：

字符和字形

这两个术语的意思相似。字符是数字实体，而字形是书面语言的原子单位（字母、印刷连字、中文字符、标点符号等）。程序员以字符为思考单位，而用户以字形为思考单位。有时需要使用多个字符来表示单个字形。例如，我们可以通过组合字符*o*和字符^（抑扬符）来产生单个字形ô。

字形

这是一种显示字形的具体方式。有时，相同的字形在不同的上下文或其他因素下显示方式不同。例如，字形*f*和*i*可以呈现为字形*f*和字形*i*，通过连字字形连接，或者没有连字。

代码点

Unicode 通过称为*代码点*的数字来表示它支持的字符。代码点的十六进制范围是 0x0 到 0x10FFFF（17 倍 16 位）。

代码单元

为了存储或传输代码点，我们将它们编码为*代码单元*，这是具有固定长度的数据片段。长度以位为单位，并由编码方案确定，Unicode 有几种编码方案，例如 UTF-8 和 UTF-16。名称中的数字表示代码单元的长度，以位为单位。如果一个代码点太大而无法适应单个代码单元，它必须被分解为多个单元；也就是说，表示单个代码点所需的代码单元数量可能会有所不同。

BOM（字节顺序标记）

如果一个代码单元大于一个字节，字节顺序很重要。BOM 是文本开头的一个伪字符（可能被编码为多个代码单元），指示代码单元是*大端*（最重要的字节在前）还是*小端*（最不重要的字节在前）。没有 BOM 的文本的默认值是大端。BOM 还指示所使用的编码；对于 UTF-8、UTF-16 等编码是不同的。此外，如果 Web 浏览器没有关于文本编码的其他信息，它还可以作为 Unicode 的标记。然而，由于几个原因，BOM 并不经常使用：

+   UTF-8 是迄今为止最流行的 Unicode 编码，不需要 BOM，因为只有一种字节排序方式。

+   几种字符编码规定了固定的字节顺序。那么就不应该使用 BOM。例如 UTF-16BE（UTF-16 大端）、UTF-16LE、UTF-32BE 和 UTF-32LE。这是处理字节顺序的更安全的方式，因为元数据和数据保持分开，不会混淆。

规范化

有时相同的字形可以用几种方式表示。例如，字形ö可以表示为单个代码点，也可以表示为一个*o*后跟一个组合字符¨（分音符，双点）。规范化是将文本转换为规范表示的过程；等效的代码点和代码点序列都被转换为相同的代码点（或代码点序列）。这对于文本处理（例如搜索文本）很有用。Unicode 规定了几种规范化。

字符属性

规范指定了规范的几个属性，其中一些列在这里：

+   *名称*。一个由大写字母 A-Z，数字 0-9，连字符(-)和<空格>组成的英文名称。两个例子：

+   “λ”的名称是“希腊小写字母λ”。

+   “!”的名称是“感叹号”。

+   [*一般类别*](http://bit.ly/1fwsjL9)。将字符分成字母、大写字母、数字和标点等类别。

+   *年龄*。该字符是在哪个版本的 Unicode 中引入的（1.0、1.1、2.0 等）？

+   *已弃用*。是否不鼓励使用该字符？

+   *以及更多*。

## 代码点

代码点的范围最初是 16 位。随着 Unicode 版本 2.0（1996 年 7 月）的扩展，它现在被分成了 17 个*平面*，编号从 0 到 16。每个平面包括 16 位（十六进制表示法：0x0000–0xFFFF）。因此，在接下来的十六进制范围中，四个底部以外的数字包含了平面的编号。

+   第 0 平面，基本多文种平面（BMP）：0x0000–0xFFFF

+   第 1 平面，补充多语种平面（SMP）：0x10000–0x1FFFF

+   第 2 平面，补充表意文字平面（SIP）：0x20000–0x2FFFF

+   第 3–13 平面，未分配

+   第 14 平面，补充特殊用途平面（SSP）：0xE0000–0xEFFFF

+   第 15–16 平面，补充专用区域（S PUA A/B）：0x0F0000–0x10FFFF

第 1–16 平面称为*补充平面*或*星际平面*。

## Unicode 编码

*UTF-32*（Unicode 转换格式 32）是一种具有 32 位代码单元的格式。任何代码点都可以由单个代码单元编码，使得这是唯一的固定长度编码；对于其他编码，编码一个点所需的单元数量是变化的。

*UTF-16*是一种具有 16 位代码单元的格式，需要一个到两个单元来表示一个代码点。BMP 代码点可以由单个代码单元表示。高代码点是 20 位（16 乘以 16 位），在减去 0x10000（BMP 的范围）后。这些位被编码为两个代码单元（所谓的*代理对*）：

领先代理

最重要的 10 位：存储在范围 0xD800–0xDBFF 中。也称为*高代理代码单元*。

尾随代理

最不重要的 10 位：存储在范围 0xDC00–0xDFFF 中。也称为*低代理代码单元*。

以下表格（改编自 Unicode 标准 6.2.0，表 3-5）可视化了位的分布：

| 代码点 | UTF-16 代码单元 |
| --- | --- |
| xxxxxxxxxxxxxxxx（16 位）| xxxxxxxxxxxxxxxx |
| pppppxxxxxxyyyyyyyyyy（21 位=5+6+10 位）| 110110qqqqxxxxxx 110111yyyyyyyyyy（qqqq = ppppp − 1）|

为了启用这种编码方案，BMP 有一个未使用的代码点范围为 0xD800–0xDFFF 的空隙。因此，领先代理、尾随代理和 BMP 代码点的范围是不相交的，使得在面对错误时解码更加健壮。以下函数将代码点编码为 UTF-16（稍后我们将看到一个使用它的示例）：

```js
function toUTF16(codePoint) {
    var TEN_BITS = parseInt('1111111111', 2);
    function u(codeUnit) {
        return '\\u'+codeUnit.toString(16).toUpperCase();
    }

    if (codePoint <= 0xFFFF) {
        return u(codePoint);
    }
    codePoint -= 0x10000;

    // Shift right to get to most significant 10 bits
    var leadingSurrogate = 0xD800 | (codePoint >> 10);

    // Mask to get least significant 10 bits
    var trailingSurrogate = 0xDC00 | (codePoint & TEN_BITS);

    return u(leadingSurrogate) + u(trailingSurrogate);
}
```

*UCS-2*，一种已弃用的格式，使用 16 位代码单元来表示（仅！）BMP 的代码点。当 Unicode 代码点的范围扩展到 16 位之外时，UTF-16 取代了 UCS-2。

*UTF-8*具有 8 位代码单元。它在传统 ASCII 编码和 Unicode 之间架起了一座桥梁。ASCII 只有 128 个字符，其编号与前 128 个 Unicode 代码点相同。UTF-8 是向后兼容的，因为所有 ASCII 代码都是有效的代码单元。换句话说，在范围 0–127 的单个代码单元中编码了相同范围内的单个代码点。这些代码单元的最高位为零。另一方面，如果最高位为 1，则会跟随更多的单元，以为更高的代码点提供额外的位。这导致了以下编码方案：

+   0000–007F：0xxxxxxx（7 位，存储在 1 字节中）

+   0080–07FF：110xxxxx，10xxxxxx（5+6 位=11 位，存储在 2 字节中）

+   0800–FFFF：1110xxxx，10xxxxxx，10xxxxxx（4+6+6 位=16 位，存储在 3 字节中）

+   10000–1FFFFF：11110xxx，10xxxxxx，10xxxxxx，10xxxxxx（3+6+6+6 位=21 位，存储在 4 字节中）。最高代码点是 10FFFF，因此 UTF-8 有一些额外的空间。

如果最高位不为 0，则零之前的 1 的数量表示序列中有多少个代码单元。初始单元之后的所有单元都具有位前缀 10。因此，初始代码单元和后续代码单元的范围是不相交的，这有助于从编码错误中恢复。

UTF-8 已成为最流行的 Unicode 格式。最初，它之所以受欢迎，是因为它与 ASCII 的向后兼容性。后来，它因其在操作系统、编程环境和应用程序中的广泛和一致的支持而受到青睐。

## JavaScript 源代码和 Unicode

JavaScript 处理 Unicode 源代码有两种方式：内部（在解析期间）和外部（在加载文件时）。

### 内部源代码

在内部，JavaScript 源代码被视为一系列 UTF-16 代码单元。根据[第 6 节](http://ecma-international.org/ecma-262/5.1/#sec-6)的 EMCAScript 规范：

> ECMAScript 源文本以 Unicode 字符编码的形式表示，版本为 3.0 或更高。[...] ECMAScript 源文本被假定为本规范的目的是一系列 16 位代码单元。[...] 如果实际源文本以除 16 位代码单元以外的形式编码，必须处理为首先转换为 UTF-16。

在标识符、字符串文字和正则表达式文字中，任何代码单元也可以通过 Unicode 转义序列`\uHHHH`来表示，其中`HHHH`是四个十六进制数字。例如：

```js
> var f\u006F\u006F = 'abc';
> foo
'abc'

> var λ = 123;
> \u03BB
123
```

这意味着您可以在源代码中使用 Unicode 字符的文字和变量名，而不会离开 ASCII 范围。

在字符串文字中，还有一种额外的转义可用：用两位十六进制数字表示的*十六进制转义序列*，表示范围在 0x00-0xFF 的代码单元。例如：

```js
> '\xF6' === 'ö'
true
> '\xF6' === '\u00F6'
true
```

### 外部源代码

虽然内部使用 UTF-16，但 JavaScript 源代码通常不以该格式存储。当 Web 浏览器通过`<script>`标签加载源文件时，它会确定编码[如下](http://bit.ly/1fwstC9)：

+   如果文件以 BOM 开头，则编码是 UTF 变体，取决于使用的 BOM。

+   否则，如果文件是通过 HTTP(S)加载的，那么`Content-Type`头可以通过`charset`参数指定编码。例如：

    ```js
    Content-Type: application/javascript; charset=utf-8
    ```

### 提示

JavaScript 文件的正确*媒体类型*（以前称为*MIME 类型*）是`application/javascript`。但是，较旧的浏览器（例如 Internet Explorer 8 及更早版本）最可靠地使用`text/javascript`。不幸的是，`<script>`标签的`type`属性的[默认值](http://bit.ly/1fwsvKe)是`text/javascript`。至少对于 JavaScript，您可以省略该属性；包含它没有好处。

+   否则，如果`<script>`标签具有`charset`属性，则将使用该编码。即使属性`type`包含有效的媒体类型，该类型也不得具有参数`charset`（就像前述的`Content-Type`头）。这确保了`charset`和`type`的值不会冲突。

+   否则，使用包含`<script>`标签的文档的编码。例如，这是 HTML5 文档的开头，其中`<meta>`标签声明文档编码为 UTF-8：

    ```js
    <!doctype html>
    <html>
    <head>
        <meta charset="UTF-8">
    ...
    ```

强烈建议您始终指定编码。如果不指定，将使用特定于区域设置的[默认编码](http://bit.ly/1oODGWp)。换句话说，在不同国家，人们将以不同方式看待文件。只有最低的 7 位在各个区域设置中相对稳定。

我的建议可以总结如下：

+   对于您自己的应用程序，您可以使用 Unicode。但必须将应用程序的 HTML 页面的编码指定为 UTF-8。

+   对于库，最安全的做法是发布 ASCII（7 位）代码。

一些缩小工具可以将具有超出 7 位的 Unicode 代码点的源代码转换为“7 位干净”的源代码。它们通过用 Unicode 转义替换非 ASCII 字符来实现。例如，以下调用[UglifyJS](https://github.com/mishoo/UglifyJS2)将文件*test.js*翻译为：

```js
uglifyjs -b beautify=false,ascii-only=true test.js
```

文件*test.js*如下所示：

```js
var σ = 'Köln';
```

UglifyJS 的输出如下：

```js
var \u03c3="K\xf6ln";
```

考虑以下负面示例。有一段时间，库 D3.js 以 UTF-8 发布。这导致了一个[错误](https://github.com/mbostock/d3/issues/1195)，因为当它从编码不是 UTF-8 的页面加载时，代码包含了诸如以下语句：

```js
var π = Math.PI, ε = 1e-6;
```

标识符π和ε没有被正确解码，也没有被识别为有效的变量名。此外，一些超出 7 位的代码点的字符串文字也没有被正确解码。作为一种解决方法，您可以通过向`<script>`标签添加适当的`charset`属性来加载代码：

```js
<script charset="utf-8" src="d3.js"></script>
```

## JavaScript 字符串和 Unicode

JavaScript 字符串是一系列 UTF-16 代码单元。根据 ECMAScript 规范，[第 8.4 节](http://ecma-international.org/ecma-262/5.1/#sec-8.4)：

> 当一个字符串包含实际文本数据时，每个元素被认为是单个 UTF-16 代码单元。

### 转义序列

如前所述，您可以在字符串文字中使用 Unicode 转义序列和十六进制转义序列。例如，您可以通过将*o*与重音符（代码点 0x0308）组合来产生字符ö：

```js
> console.log('o\u0308')
ö
```

这适用于 JavaScript 命令行，例如 Web 浏览器控制台和 Node.js REPL。您还可以将这种类型的字符串插入到 Web 页面的 DOM 中。

### 通过转义引用星际飞机字符

网络上有许多不错的 Unicode 符号表。看看 Tim Whitlock 的[“Emoji Unicode Tables”](http://apps.timwhitlock.info/emoji/tables/unicode)，并对现代 Unicode 字体中有多少符号感到惊讶。表中的符号都不是图像；它们都是字体字形。假设您想通过 JavaScript 显示一个星际飞机中的 Unicode 字符（显然，这样做存在风险：并非所有字体都支持所有这些字符）。例如，考虑一头奶牛，代码点为 0x1F404：！[](img/spjs_24in01.png.jpg)。

您可以复制字符并直接粘贴到您的 Unicode 编码 JavaScript 源代码中：

！[](img/spjs_24in02.png)

JavaScript 引擎将解码源代码（通常为 UTF-8）并创建一个具有两个 UTF-16 代码单元的字符串。或者，您可以自己计算两个代码单元并使用 Unicode 转义序列。有一些网络应用程序可以执行这种计算，例如：

+   [UTF 转换器](http://macchiato.com/unicode/convert.html)

+   [Mathias Bynens 的“JavaScript 转义”](http://mothereff.in/js-escapes)

先前定义的函数`toUTF16`也执行了它：

```js
> toUTF16(0x1F404)
'\\uD83D\\uDC04'
```

UTF-16 代理对（0xD83D，0xDC04）确实编码了奶牛：

！[](img/spjs_24in03.png)

### 计数字符

如果字符串包含代理对（两个编码单元编码一个代码点），那么`length`属性不再计算图形元素。它计算编码单元：

！[](img/spjs_24in04.png)

这可以通过库来修复，例如 Mathias Bynens 的[Punycode.js](https://github.com/bestiejs/punycode.js)，它与 Node.js 捆绑在一起：

```js
> var puny = require('punycode');
> puny.ucs2.decode(str).length
1
```

### Unicode 规范化

如果您想在字符串中搜索或比较它们，那么您需要进行规范化，例如通过库[unorm](https://github.com/walling/unorm)（由 Bjarke Walling）。

## JavaScript 正则表达式和 Unicode

JavaScript 正则表达式中的 Unicode 支持（请参阅第十九章）非常有限。例如，没有办法匹配“大写字母”等 Unicode 类别。

行终止符影响匹配。行终止符是下表中指定的四个字符之一：

| 代码单元 | 名称 | 字符转义序列 |
| --- | --- | --- |
| \u000A | 换行符 | `\n` |
| \u000D | 回车 | `\r` |
| \u2028 | 行分隔符 |  |
| \u2029 | 段落分隔符 |  |

以下正则表达式构造基于 Unicode：

+   `\s \S`（空白，非空白）具有基于 Unicode 的定义：

    ```js
    > /^\s$/.test('\uFEFF')
    true
    ```

+   `.`（点）匹配所有代码单元（不是代码点！）除了行终止符。请参阅下一节，了解如何匹配任何代码点。

+   多行模式`/m`：在多行模式下，断言`^`匹配输入的开头和行终止符之后。断言`$`匹配行终止符之前和输入的结尾。在非多行模式下，它们只在输入的开头或结尾匹配。

其他重要的字符类是基于 ASCII 而不是 Unicode 定义的：

+   `\d \D`（数字，非数字）：数字等同于`[0-9]`。

+   `\w \W`（单词字符，非单词字符）：单词字符等同于`[A-Za-z0-9_]`。

+   `\b \B`（在单词边界，单词内）：单词是由单词字符（`[A-Za-z0-9_]`）组成的序列。例如，在字符串`'über'`中，字符类转义`\b`将字符*b*视为单词的开始：

    ```js
    > /\bb/.test('über')
    true
    ```

### 匹配任何代码单元和任何代码点

要匹配任何代码单元，您可以使用`[\s\S]`；请参见[原子：通用](ch19.html#regex_atoms_general "原子：通用")。

要匹配任何代码点，您需要使用：¹⁸

```js
([\0-\uD7FF\uE000-\uFFFF]|[\uD800-\uDBFF][\uDC00-\uDFFF])
```

前面的模式工作原理如下：

```js
([BMP code point]|[leading surrogate][trailing surrogate])
```

由于所有这些范围都是不相交的，该模式将正确匹配 UTF-16 字符串中的代码点。

### 库

一些库可帮助处理 JavaScript 中的 Unicode：

+   [Regenerate](https://github.com/mathiasbynens/regenerate)有助于生成像前面那样的范围，以匹配任何代码单元。它旨在用作构建工具的一部分，但也可以动态工作，用于尝试各种功能。

+   [XRegExp](http://xregexp.com)是一个正则表达式库，它有一个[官方附加组件](http://xregexp.com/plugins/#unicode)，可以通过以下三种构造之一匹配 Unicode 类别、脚本、块和属性：

    ```js
    \p{...} \p{^...} \P{...}
    ```

例如，`\p{Letter}`匹配各种字母表中的字母，而`\p{^Letter}`和`\P{Letter}`都匹配所有其他代码点。第三十章包含了对 XRegExp 的简要概述。

+   ECMAScript 国际化 API（请参见[ECMAScript 国际化 API](ch30.html#i18n_api "ECMAScript 国际化 API")）提供了对 Unicode 的排序和搜索等功能。

### 推荐阅读和章节来源

有关 Unicode 的更多信息，请参见以下内容：

+   维基百科上有几篇关于[Unicode](http://en.wikipedia.org/wiki/Unicode)及其术语的好文章。

+   [Unicode.org](http://www.unicode.org/)，Unicode 联盟的官方网站，以及其[FAQ](http://www.unicode.org/faq/)也是很好的资源。

+   Joel Spolsky 的介绍性文章[“每个软件开发人员绝对必须了解的有关 Unicode 和字符集的绝对最低限度（没有借口！）”](http://www.joelonsoftware.com/articles/Unicode.html)很有帮助。

有关 JavaScript 中的 Unicode 支持的信息，请参见：

+   [Mathias Bynens 的文章](http://mathiasbynens.be/notes/javascript-encoding)“JavaScript 的内部字符编码：UCS-2 还是 UTF-16？”

+   《JavaScript，正则表达式和 Unicode》由 Steven Levithan

### 致谢

以下人员为本章做出了贡献：Mathias Bynens（@mathias），Anne van Kesteren（@annevk）和 Calvin Metcalf（@CWMma）。

* * *

¹⁸ 严格来说，任何[Unicode 标量值](http://www.unicode.org/glossary/#unicode_scalar_value)。

