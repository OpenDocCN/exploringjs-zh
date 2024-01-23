# 二十、字符串

> 原文：[`exploringjs.com/impatient-js/ch_strings.html`](https://exploringjs.com/impatient-js/ch_strings.html)

* * *

+   20.1 字符串速查表

    +   20.1.1 处理字符串

    +   20.1.2 JavaScript 字符 vs. 代码点 vs. 图形簇

    +   20.1.3 字符串方法

+   20.2 普通字符串字面量

    +   20.2.1 转义

+   20.3 访问 JavaScript 字符

+   20.4 通过 `+` 进行字符串连接

+   20.5 转换为字符串

    +   20.5.1 对象的字符串化

    +   20.5.2 自定义对象的字符串化

    +   20.5.3 值的另一种字符串化方式

+   20.6 比较字符串

+   20.7 文本的原子：代码点、JavaScript 字符、图形簇

    +   20.7.1 使用代码点

    +   20.7.2 使用代码单元（字符代码）

    +   20.7.3 ASCII 转义

    +   20.7.4 注意事项：图形簇

+   20.8 字符串快速参考

    +   20.8.1 转换为字符串

    +   20.8.2 文本原子的数值

    +   20.8.3 `String.prototype`: 查找和匹配

    +   20.8.4 `String.prototype`: 提取

    +   20.8.5 `String.prototype`: 组合

    +   20.8.6 `String.prototype`: 转换

    +   20.8.7 来源

* * *

### 20.1 字符串速查表

字符串是 JavaScript 中的原始值，是不可变的。也就是说，与字符串相关的操作总是产生新的字符串，而不会改变现有的字符串。

#### 20.1.1 处理字符串

字符串字面量：

```js
const str1 = 'Don\'t say "goodbye"'; // string literal
const str2 = "Don't say \"goodbye\""; // string literals
assert.equal(
 `As easy as ${123}!`, // template literal
 'As easy as 123!',
);
```

反斜杠用于：

+   转义文字定界符（前一个示例的前两行）

+   表示特殊字符：

    +   `\\` 表示反斜杠

    +   `\n` 表示换行

    +   `\r` 表示回车

    +   `\t` 表示制表符

在 `String.raw` 标记模板中（A 行），反斜杠被视为普通字符：

```js
assert.equal(
 String.raw`\ \n\t`, // (A)
 '\\  \\n\\t',
);
```

将值转换为字符串：

```js
> String(undefined)
'undefined'
> String(null)
'null'
> String(123.45)
'123.45'
> String(true)
'true'
```

复制字符串的部分

```js
// There is no type for characters;
// reading characters produces strings:
const str3 = 'abc';
assert.equal(
 str3[2], 'c' // no negative indices allowed
);
assert.equal(
 str3.at(-1), 'c' // negative indices allowed
);

// Copying more than one character:
assert.equal(
 'abc'.slice(0, 2), 'ab'
);
```

连接字符串：

```js
assert.equal(
 'I bought ' + 3 + ' apples',
 'I bought 3 apples',
);

let str = '';
str += 'I bought ';
str += 3;
str += ' apples';
assert.equal(
 str, 'I bought 3 apples',
);
```

#### 20.1.2 JavaScript 字符 vs. 代码点 vs. 图形簇

**JavaScript 字符** 大小为 16 位。它们是字符串中的索引和 `.length` 计数的内容。

**代码点** 是 Unicode 文本的原子部分。大多数代码点适合一个 JavaScript 字符，其中一些占据两个（特别是表情符号）：

```js
assert.equal(
 'A'.length, 1
);
assert.equal(
 '🙂'.length, 2
);
```

**图形簇**（*用户感知的字符*）代表书面符号。每个图形簇由一个或多个代码点组成。

因此，我们不应该将文本拆分为 JavaScript 字符，而应该将其拆分为图形簇。有关如何处理文本的更多信息，请参见§20.7 “文本的原子：代码点、JavaScript 字符、图形簇”。

#### 20.1.3 字符串方法

本小节简要概述了字符串 API。本章末尾有更全面的快速参考。

查找子字符串：

```js
> 'abca'.includes('a')
true
> 'abca'.startsWith('ab')
true
> 'abca'.endsWith('ca')
true

> 'abca'.indexOf('a')
0
> 'abca'.lastIndexOf('a')
3
```

拆分和连接：

```js
assert.deepEqual(
 'a, b,c'.split(/, ?/),
 ['a', 'b', 'c']
);
assert.equal(
 ['a', 'b', 'c'].join(', '),
 'a, b, c'
);
```

填充和修剪：

```js
> '7'.padStart(3, '0')
'007'
> 'yes'.padEnd(6, '!')
'yes!!!'

> '\t abc\n '.trim()
'abc'
> '\t abc\n '.trimStart()
'abc\n '
> '\t abc\n '.trimEnd()
'\t abc'
```

重复和更改大小写：

```js
> '*'.repeat(5)
'*****'
> '= b2b ='.toUpperCase()
'= B2B ='
> 'ΑΒΓ'.toLowerCase()
'αβγ'
```

### 20.2 普通字符串字面量

普通字符串字面量由单引号或双引号括起来：

```js
const str1 = 'abc';
const str2 = "abc";
assert.equal(str1, str2);
```

单引号更常用，因为它更容易提及 HTML，而 HTML 更喜欢双引号。

下一章涵盖了*模板文字*，它给我们提供了：

+   字符串插值

+   多行

+   原始字符串文字（反斜杠没有特殊含义）

#### 20.2.1 转义

反斜杠让我们创建特殊字符：

+   Unix 换行符：`'\n'`

+   Windows 换行符：`'\r\n'`

+   制表符：`'\t'`

+   反斜杠：`'\\'`

反斜杠还允许我们在字符串文字内部使用字符串文字的分隔符：

```js
assert.equal(
 'She said: "Let\'s go!"',
 "She said: \"Let's go!\"");
```

### 20.3 访问 JavaScript 字符

JavaScript 没有额外的字符数据类型-字符始终表示为字符串。

```js
const str = 'abc';

// Reading a JavaScript character at a given index
assert.equal(str[1], 'b');

// Counting the JavaScript characters in a string:
assert.equal(str.length, 3);
```

我们在屏幕上看到的字符称为*图形簇*。它们中的大多数由单个 JavaScript 字符表示。但是，还有一些图形簇（特别是表情符号）由多个 JavaScript 字符表示：

```js
> '🙂'.length
2
```

这是在§20.7 “文本的原子：代码点、JavaScript 字符、图形簇”中解释的。

### 20.4 通过`+`进行字符串连接

如果至少有一个操作数是字符串，则加号运算符（`+`）将任何非字符串转换为字符串并连接结果：

```js
assert.equal(3 + ' times ' + 4, '3 times 4');
```

赋值运算符`+=`在我们想要逐步组装字符串时非常有用：

```js
let str = ''; // must be `let`!
str += 'Say it';
str += ' one more';
str += ' time';

assert.equal(str, 'Say it one more time');
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **通过`+`进行连接是有效的**

使用`+`来组装字符串非常有效，因为大多数 JavaScript 引擎在内部对其进行了优化。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：连接字符串**

`exercises/strings/concat_string_array_test.mjs`

### 20.5 转换为字符串

这是将值`x`转换为字符串的三种方法：

+   `String(x)`

+   `''+x`

+   `x.toString()`（对于`undefined`和`null`无效）

建议：使用描述性和安全的`String()`。

示例：

```js
assert.equal(String(undefined), 'undefined');
assert.equal(String(null), 'null');

assert.equal(String(false), 'false');
assert.equal(String(true), 'true');

assert.equal(String(123.45), '123.45');
```

布尔值的陷阱：如果我们通过`String()`将布尔值转换为字符串，通常无法通过`Boolean()`将其转换回来：

```js
> String(false)
'false'
> Boolean('false')
true
```

唯一返回`false`的字符串是空字符串。

#### 20.5.1 字符串化对象

普通对象具有默认的字符串表示，这并不是非常有用：

```js
> String({a: 1})
'[object Object]'
```

数组具有更好的字符串表示，但仍然隐藏了许多信息：

```js
> String(['a', 'b'])
'a,b'
> String(['a', ['b']])
'a,b'

> String([1, 2])
'1,2'
> String(['1', '2'])
'1,2'

> String([true])
'true'
> String(['true'])
'true'
> String(true)
'true'
```

字符串化函数，返回它们的源代码：

```js
> String(function f() {return 4})
'function f() {return 4}'
```

#### 20.5.2 自定义对象的字符串化

我们可以通过实现`toString()`方法来覆盖对象的内置字符串化方式：

```js
const obj = {
 toString() {
 return 'hello';
 }
};

assert.equal(String(obj), 'hello');
```

#### 20.5.3 字符串化值的另一种方法

JSON 数据格式是 JavaScript 值的文本表示。因此，`JSON.stringify()`也可以用于将值转换为字符串：

```js
> JSON.stringify({a: 1})
'{"a":1}'
> JSON.stringify(['a', ['b']])
'["a",["b"]]'
```

警告是 JSON 仅支持`null`、布尔值、数字、字符串、数组和对象（它总是将其视为通过对象文字创建的）。

提示：第三个参数让我们可以切换到多行输出并指定缩进的数量-例如：

```js
console.log(JSON.stringify({first: 'Jane', last: 'Doe'}, null, 2));
```

此语句产生以下输出：

```js
{
  "first": "Jane",
  "last": "Doe"
}
```

### 20.6 比较字符串

字符串可以通过以下运算符进行比较：

```js
< <= > >=
```

有一个重要的警告需要考虑：这些运算符基于 JavaScript 字符的数值进行比较。这意味着 JavaScript 用于字符串的顺序与字典和电话簿中使用的顺序不同：

```js
> 'A' < 'B' // ok
true
> 'a' < 'B' // not ok
false
> 'ä' < 'b' // not ok
false
```

正确比较文本超出了本书的范围。它通过[ECMAScript 国际化 API（`Intl`）](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Intl)支持。

### 20.7 文本的原子：代码点、JavaScript 字符、图形簇

§19 “Unicode – a brief introduction”的快速回顾：

+   *代码点*是 Unicode 文本的原子部分。每个代码点的大小为 21 位。

+   JavaScript 字符串通过 UTF-16 编码格式实现 Unicode。它使用一个或两个 16 位*代码单元*来编码单个代码点。

    +   每个 JavaScript 字符（在字符串中索引）都是一个代码单元。在 JavaScript 标准库中，代码单元也被称为*char codes*。

+   *字形簇*（*用户感知字符*）表示屏幕或纸张上显示的书写符号。需要一个或多个代码点来编码单个字形簇。

以下代码演示了一个代码点包含一个或两个 JavaScript 字符。我们通过`.length`来计算后者：

```js
// 3 code points, 3 JavaScript characters:
assert.equal('abc'.length, 3);

// 1 code point, 2 JavaScript characters:
assert.equal('🙂'.length, 2);
```

以下表格总结了我们刚刚探讨的概念：

| 实体 | 大小 | 通过编码 |
| --- | --- | --- |
| JavaScript 字符（UTF-16 代码单元） | 16 位 | - |
| Unicode 代码点 | 21 位 | 1-2 个代码单元 |
| Unicode 字形簇 |  | 1 个或多个代码点 |

#### 20.7.1 处理代码点

让我们探索一下 JavaScript 处理代码点的工具。

*Unicode 代码点转义*允许我们以十六进制（1-5 位数字）指定代码点。它产生一个或两个 JavaScript 字符。

```js
> '\u{1F642}'
'🙂'
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **Unicode 转义序列**

在 ECMAScript 语言规范中，*Unicode 代码点转义*和*Unicode 代码单元转义*（我们稍后会遇到）被称为*Unicode 转义序列*。

`String.fromCodePoint()`将一个代码点转换为 1-2 个 JavaScript 字符：

```js
> String.fromCodePoint(0x1F642)
'🙂'
```

`.codePointAt()`将 1-2 个 JavaScript 字符转换为一个代码点：

```js
> '🙂'.codePointAt(0).toString(16)
'1f642'
```

我们可以*迭代*字符串，这将访问代码点（而不是 JavaScript 字符）。迭代在本书的后面有描述。一种迭代的方式是通过`for-of`循环：

```js
const str = '🙂a';
assert.equal(str.length, 3);

for (const codePointChar of str) {
 console.log(codePointChar);
}

// Output:
// '🙂'
// 'a'
```

`Array.from()`也是基于迭代和访问代码点的：

```js
> Array.from('🙂a')
[ '🙂', 'a' ]
```

这使得它成为一个很好的计算代码点的工具：

```js
> Array.from('🙂a').length
2
> '🙂a'.length
3
```

#### 20.7.2 处理代码单元（char codes）

字符串的索引和长度是基于 JavaScript 字符的（由 UTF-16 代码单元表示）。

要以十六进制指定代码单元，我们可以使用一个*Unicode 代码单元转义*，其中恰好有四个十六进制数字：

```js
> '\uD83D\uDE42'
'🙂'
```

我们可以使用`String.fromCharCode()`。*Char code*是标准库对*代码单元*的名称：

```js
> String.fromCharCode(0xD83D) + String.fromCharCode(0xDE42)
'🙂'
```

要获取字符的 char code，请使用`.charCodeAt()`：

```js
> '🙂'.charCodeAt(0).toString(16)
'd83d'
```

#### 20.7.3 ASCII 转义

如果一个字符的代码点低于 256，我们可以通过*ASCII 转义*来引用它，其中恰好有两个十六进制数字：

```js
> 'He\x6C\x6Co'
'Hello'
```

（ASCII 转义的官方名称是*十六进制转义序列* - 这是第一个使用十六进制数字的转义。）

#### 20.7.4 注意事项：字形簇

在处理可能用任何人类语言书写的文本时，最好在字形簇的边界处拆分，而不是在代码点的边界处。

TC39 正在研究[`Intl.Segmenter`](https://github.com/tc39/proposal-intl-segmenter)，这是 ECMAScript 国际化 API 的一个提案，支持 Unicode 分段（沿着字形簇边界，单词边界，句子边界等）。

在该提案成为标准之前，我们可以使用几个可用的库之一（在网上搜索“JavaScript grapheme”）。

### 20.8 快速参考：字符串

#### 20.8.1 转换为字符串

Tbl. 14 描述了各种值如何转换为字符串。

表 14：将值转换为字符串。

| `x` | `String(x)` |
| --- | --- |
| `undefined` | `'undefined'` |
| `null` | `'null'` |
| boolean | `false` `→` `'false'`, `true` `→` `'true'` |
| number | 例子：`123` `→` `'123'` |
| bigint | 例子：`123n` `→` `'123'` |
| string | `x`（输入，不变） |
| symbol | 例子：`Symbol('abc')` `→` `'Symbol(abc)'` |
| object | 可以通过，例如，`toString()` 进行配置 |

#### 20.8.2 文本原子的数值

+   **Char code**：表示 JavaScript 字符的数字。JavaScript 对*Unicode 代码单元*的名称。

    +   大小：16 位，无符号

    +   将数字转换为字符串：`String.fromCharCode()` ^([ES1])

    +   将字符串转换为数字：字符串方法`.charCodeAt()` ^([ES1])

+   **代码点**：表示 Unicode 文本的一个原子部分的数字。

    +   大小：21 位，无符号（17 个平面，每个 16 位）

    +   将数字转换为字符串：`String.fromCodePoint()` ^([ES6])

    +   将字符串转换为数字：字符串方法`.codePointAt()` ^([ES6])

#### 20.8.3 `String.prototype`: 查找和匹配

（`String.prototype`是存储字符串方法的地方。）

+   `.endsWith(searchString: string, endPos=this.length): boolean` ^([ES6])

    如果字符串的长度为`endPos`，则返回`true`，如果不是则返回`false`。

    ```js
    > 'foo.txt'.endsWith('.txt')
    true
    > 'abcde'.endsWith('cd', 4)
    true
    ```

+   `.includes(searchString: string, startPos=0): boolean` ^([ES6])

    如果字符串包含`searchString`则返回`true`，否则返回`false`。搜索从`startPos`开始。

    ```js
    > 'abc'.includes('b')
    true
    > 'abc'.includes('b', 2)
    false
    ```

+   `.indexOf(searchString: string, minIndex=0): number` ^([ES1])

    返回字符串中`searchString`出现的最低索引，否则返回`-1`。任何返回的索引都将是`minIndex`或更高。

    ```js
    > 'abab'.indexOf('a')
    0
    > 'abab'.indexOf('a', 1)
    2
    > 'abab'.indexOf('c')
    -1
    ```

+   `.lastIndexOf(searchString: string, maxIndex=Infinity): number` ^([ES1])

    返回字符串中`searchString`出现的最高索引，否则返回`-1`。任何返回的索引都将是`maxIndex`或更低。

    ```js
    > 'abab'.lastIndexOf('ab', 2)
    2
    > 'abab'.lastIndexOf('ab', 1)
    0
    > 'abab'.lastIndexOf('ab')
    2
    ```

+   [1 of 2] `.match(regExp: string | RegExp): RegExpMatchArray | null` ^([ES3])

    如果`regExp`是一个带有标志`/g`的正则表达式，则`.match()`返回字符串中`regExp`的第一个匹配项。如果没有匹配项则返回`null`。如果`regExp`是一个字符串，则在执行先前提到的步骤之前，用它创建一个正则表达式（类似于`new RegExp()`的参数）。

    结果具有以下类型：

    ```js
    interface RegExpMatchArray extends Array<string> {
     index: number;
     input: string;
     groups: undefined | {
     [key: string]: string
     };
    }
    ```

    编号的捕获组成为数组索引（这就是为什么这种类型扩展了`Array`）。命名捕获组（ES2018）成为`.groups`的属性。在此模式下，`.match()`的工作方式类似于`RegExp.prototype.exec()`。

    例子：

    ```js
    > 'ababb'.match(/a(b+)/)
    { 0: 'ab', 1: 'b', index: 0, input: 'ababb', groups: undefined }
    > 'ababb'.match(/a(?<foo>b+)/)
    { 0: 'ab', 1: 'b', index: 0, input: 'ababb', groups: { foo: 'b' } }
    > 'abab'.match(/x/)
    null
    ```

+   [2 of 2] `.match(regExp: RegExp): string[] | null` ^([ES3])

    如果`regExp`的标志`/g`被设置，`.match()`返回所有匹配项的数组，如果没有匹配项则返回`null`。

    ```js
    > 'ababb'.match(/a(b+)/g)
    [ 'ab', 'abb' ]
    > 'ababb'.match(/a(?<foo>b+)/g)
    [ 'ab', 'abb' ]
    > 'abab'.match(/x/g)
    null
    ```

+   `.search(regExp: string | RegExp): number` ^([ES3])

    返回`regExp`在字符串中出现的索引。如果`regExp`是一个字符串，则用它创建一个正则表达式（类似于`new RegExp()`的参数）。

    ```js
    > 'a2b'.search(/[0-9]/)
    1
    > 'a2b'.search('[0-9]')
    1
    ```

+   `.startsWith(searchString: string, startPos=0): boolean` ^([ES6])

    如果`searchString`在索引`startPos`处出现，则返回`true`。否则返回`false`。

    ```js
    > '.gitignore'.startsWith('.')
    true
    > 'abcde'.startsWith('bc', 1)
    true
    ```

#### 20.8.4 `String.prototype`: 提取

+   `.slice(start=0, end=this.length): string` ^([ES3])

    返回字符串的子字符串，从索引`start`（包括）开始，到索引`end`（不包括）结束。如果索引为负数，则在使用之前将其添加到`.length`（`-1`变为`this.length-1`等）。

    ```js
    > 'abc'.slice(1, 3)
    'bc'
    > 'abc'.slice(1)
    'bc'
    > 'abc'.slice(-2)
    'bc'
    ```

+   `.at(index: number): string | undefined` ^([ES2022])

    返回索引处的 JavaScript 字符作为字符串。如果`index`为负数，则在使用之前将其添加到`.length`（`-1`变为`this.length-1`等）。

    ```js
    > 'abc'.at(0)
    'a'
    > 'abc'.at(-1)
    'c'
    ```

+   `.split(separator: string | RegExp, limit?: number): string[]` ^([ES3])

    将字符串拆分为子字符串数组 - 出现在分隔符之间的字符串。分隔符可以是一个字符串：

    ```js
    > 'a | b | c'.split('|')
    [ 'a ', ' b ', ' c' ]
    ```

    它也可以是正则表达式：

    ```js
    > 'a : b : c'.split(/ *: */)
    [ 'a', 'b', 'c' ]
    > 'a : b : c'.split(/( *):( *)/)
    [ 'a', ' ', ' ', 'b', ' ', ' ', 'c' ]
    ```

    最后一个调用演示了正则表达式中的组捕获成为返回的数组的元素。

    **警告：`.split('')`将字符串拆分为 JavaScript 字符。**在处理星号代码点（编码为两个 JavaScript 字符）时效果不佳。例如，表情符号是星号的：

    ```js
    > '🙂X🙂'.split('')
    [ '\uD83D', '\uDE42', 'X', '\uD83D', '\uDE42' ]
    ```

    相反，最好使用`Array.from()`（或扩展）：

    ```js
    > Array.from('🙂X🙂')
    [ '🙂', 'X', '🙂' ]
    ```

+   `.substring(start: number, end=this.length): string` ^([ES1])

    使用`.slice()`代替这个方法。`.substring()`在旧引擎中实现不一致，并且不支持负索引。

#### 20.8.5 `String.prototype`: 组合

+   `.concat(...strings: string[]): string` ^([ES3])

    返回字符串和`strings`的连接。`'a'.concat('b')`等同于`'a'+'b'`。后者更受欢迎。

    ```js
    > 'ab'.concat('cd', 'ef', 'gh')
    'abcdefgh'
    ```

+   `.padEnd(len: number, fillString=' '): string` ^([ES2017])

    将（片段）`fillString`附加到字符串，直到达到所需的长度`len`。如果它已经达到或超过`len`，则返回时不会有任何更改。

    ```js
    > '#'.padEnd(2)
    '# '
    > 'abc'.padEnd(2)
    'abc'
    > '#'.padEnd(5, 'abc')
    '#abca'
    ```

+   `.padStart(len: number, fillString=' '): string` ^([ES2017])

    将（片段）`fillString`前置到字符串，直到达到所需的长度`len`。如果它已经达到或超过`len`，则返回时不会有任何更改。

    ```js
    > '#'.padStart(2)
    ' #'
    > 'abc'.padStart(2)
    'abc'
    > '#'.padStart(5, 'abc')
    'abca#'
    ```

+   `.repeat(count=0): string` ^([ES6])

    返回字符串，连接`count`次。

    ```js
    > '*'.repeat()
    ''
    > '*'.repeat(3)
    '***'
    ```

#### 20.8.6 `String.prototype`: 转换

+   `.normalize(form: 'NFC'|'NFD'|'NFKC'|'NFKD' = 'NFC'): string` ^([ES6])

    根据[Unicode 规范化形式](https://unicode.org/reports/tr15/)对字符串进行规范化。

+   [1 of 2] `.replaceAll(searchValue: string | RegExp, replaceValue: string): string` ^([ES2021])

    ![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png)  **如果无法使用`.replaceAll()`该怎么办**

    如果在目标平台上不支持`.replaceAll()`，则可以使用`.replace()`。如何使用在[§43.6.8.1 “`str.replace(searchValue, replacementValue)` ^([ES3])”](ch_regexps.html#String.prototype.replace)中有解释。

    用`replaceValue`替换所有匹配的`searchValue`。如果`searchValue`是一个没有标志`/g`的正则表达式，则会抛出`TypeError`。

    ```js
    > 'x.x.'.replaceAll('.', '#')
    'x#x#'
    > 'x.x.'.replaceAll(/./g, '#')
    '####'
    > 'x.x.'.replaceAll(/./, '#')
    TypeError: String.prototype.replaceAll called with
    a non-global RegExp argument
    ```

    `replaceValue`中的特殊字符是：

    +   `$$`: 变成`$`

    +   `$n`: 变成编号为`n`的捕获组（遗憾的是，`$0`代表字符串`'$0'`，它不是指完全匹配）

    +   `$&`: 变成完全匹配

    +   `$``: 变成匹配前的所有内容

    +   `$'`: 变成匹配后的所有内容

    示例：

    ```js
    > 'a 1995-12 b'.replaceAll(/([0-9]{4})-([0-9]{2})/g, '|$2|')
    'a |12| b'
    > 'a 1995-12 b'.replaceAll(/([0-9]{4})-([0-9]{2})/g, '|$&|')
    'a |1995-12| b'
    > 'a 1995-12 b'.replaceAll(/([0-9]{4})-([0-9]{2})/g, '|$`|')
    'a |a | b'
    ```

    命名捕获组（ES2018）也受支持：

    +   `$<name>` 变成命名组`name`的捕获

    示例：

    ```js
    assert.equal(
     'a 1995-12 b'.replaceAll(
     /(?<year>[0-9]{4})-(?<month>[0-9]{2})/g, '|$<month>|'),
     'a |12| b');
    ```

+   [2 of 2] `.replaceAll(searchValue: string | RegExp, replacer: (...args: any[]) => string): string` ^([ES2021])

    如果第二个参数是一个函数，则出现的地方将被它返回的字符串替换。它的参数`args`是：

    +   `matched: string`。完全匹配

    +   `g1: string|undefined`。编号为 1 的捕获组

    +   `g2: string|undefined`。编号为 2 的捕获组

    +   （等等）

    +   `offset: number`。匹配在输入字符串中的位置？

    +   `input: string`。整个输入字符串

    ```js
    const regexp = /([0-9]{4})-([0-9]{2})/g;
    const replacer = (all, year, month) => '|' + all + '|';
    assert.equal(
     'a 1995-12 b'.replaceAll(regexp, replacer),
     'a |1995-12| b');
    ```

    命名捕获组（ES2018）也受支持。如果有任何，最后会添加一个带有包含捕获的属性的对象的参数：

    ```js
    const regexp = /(?<year>[0-9]{4})-(?<month>[0-9]{2})/g;
    const replacer = (...args) => {
     const groups=args.pop();
     return '|' + groups.month + '|';
    };
    assert.equal(
     'a 1995-12 b'.replaceAll(regexp, replacer),
     'a |12| b');
    ```

+   `.replace(searchValue: string | RegExp, replaceValue: string): string` ^([ES3])

+   `.replace(searchValue: string | RegExp, replacer: (...args: any[]) => string): string` ^([ES3])

    `.replace()`的工作方式类似于`.replaceAll()`，但如果`searchValue`是字符串或没有`/g`的正则表达式，则只替换第一次出现：

    ```js
    > 'x.x.'.replace('.', '#')
    'x#x.'
    > 'x.x.'.replace(/./, '#')
    '#.x.'
    ```

    有关此方法的更多信息，请参见[§43.6.8.1 “`str.replace(searchValue, replacementValue)` ^([ES3])”](ch_regexps.html#String.prototype.replace)。

+   `.toUpperCase(): string` ^([ES1])

    返回字符串的副本，其中所有小写字母都转换为大写。这对各种字母表的工作效果取决于 JavaScript 引擎。

    ```js
    > '-a2b-'.toUpperCase()
    '-A2B-'
    > 'αβγ'.toUpperCase()
    'ΑΒΓ'
    ```

+   `.toLowerCase(): string` ^([ES1])

    返回字符串的副本，其中所有大写字母都转换为小写。这对各种字母表的工作效果取决于 JavaScript 引擎。

    ```js
    > '-A2B-'.toLowerCase()
    '-a2b-'
    > 'ΑΒΓ'.toLowerCase()
    'αβγ'
    ```

+   `.trim(): string` ^([ES5])

    返回字符串的副本，其中所有前导和尾随空白（空格、制表符、行终止符等）都消失了。

    ```js
    > '\r\n#\t  '.trim()
    '#'
    > '  abc  '.trim()
    'abc'
    ```

+   `.trimEnd(): string` ^([ES2019])

    类似于`.trim()`，但只修剪字符串的末尾：

    ```js
    > '  abc  '.trimEnd()
    '  abc'
    ```

+   `.trimStart(): string` ^([ES2019])

    类似于`.trim()`，但只修剪字符串的开头：

    ```js
    > '  abc  '.trimStart()
    'abc  '
    ```

#### 20.8.7 来源

+   [TypeScript 的内置类型](https://github.com/Microsoft/TypeScript/blob/master/lib/)

+   [JavaScript 的 MDN Web 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

+   [ECMAScript 语言规范](https://tc39.github.io/ecma262/)

**练习：使用字符串方法**

`exercises/strings/remove_extension_test.mjs`

**测验**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/13)
