# 16 正则表达式：通过示例了解先行断言

> 原文：[`exploringjs.com/deep-js/ch_regexp-lookaround-assertions.html`](https://exploringjs.com/deep-js/ch_regexp-lookaround-assertions.html)

* * *

+   16.1 速查表：先行断言

+   16.2 本章警告

+   16.3 示例：指定匹配项之前或之后的内容（正向先行断言）

+   16.4 示例：指定匹配项之前或之后的内容（负向先行断言）

    +   16.4.1 没有简单的替代方案来使用负向先行断言

+   16.5 插曲：将先行断言指向内部

+   16.6 示例：匹配不以'abc'开头的字符串

+   16.7 示例：匹配不包含'.mjs'的子字符串

+   16.8 示例：跳过带有注释的行

+   16.9 示例：智能引号

    +   16.9.1 通过反斜杠支持转义

+   16.10 致谢

+   16.11 进一步阅读

* * *

在本章中，我们使用示例来探讨正则表达式中的先行断言。先行断言是非捕获的，必须匹配（或不匹配）输入字符串中当前位置之前（或之后）的内容。

### 16.1 速查表：先行断言

表 4：可用先行断言的概述。

| 模式 | 名称 |  |
| --- | --- | --- |
| `(?=«pattern»)` | 正向先行断言 | ES3 |
| `(?!«pattern»)` | 负向先行断言 | ES3 |
| `(?<=«pattern»)` | 正向后行断言 | ES2018 |
| `(?<!«pattern»)` | 负向后行断言 | ES2018 |

有四个先行断言（表格 4）

+   先行断言（ECMAScript 3）：

    +   正向先行断言：`(?=«pattern»)` 如果`pattern`匹配输入字符串中当前位置之后的内容，则匹配成功。

    +   负向先行断言：`(?!«pattern»)` 如果`pattern`不匹配输入字符串中当前位置之后的内容，则匹配成功。

+   后行断言（ECMAScript 2018）：

    +   正向后行断言：`(?<=«pattern»)` 如果`pattern`匹配输入字符串中当前位置之前的内容，则匹配成功。

    +   负向后行断言：`(?<!«pattern»)` 如果`pattern`不匹配输入字符串中当前位置之前的内容，则匹配成功。

### 16.2 本章警告

+   这些示例展示了通过先行断言可以实现的内容。然而，正则表达式并不总是最佳解决方案。另一种技术，比如适当的解析，可能是更好的选择。

+   后行断言是一个相对较新的功能，可能不被您所针对的所有 JavaScript 引擎支持。

+   先行断言可能会对性能产生负面影响，特别是如果它们的模式匹配长字符串。

### 16.3 示例：指定匹配项之前或之后的内容（正向先行断言）

在以下交互中，我们提取带引号的单词：

```js
> 'how "are" "you" doing'.match(/(?<=")[a-z]+(?=")/g)
[ 'are', 'you' ]
```

两个先行断言在这里帮助了我们：

+   `(?<=")` “必须由一个引号前导”

+   `(?=")` “必须跟随一个引号”

环视断言在`.match()`中特别方便，因为它在`/g`模式下返回整个匹配（捕获组 0）。环视断言的模式匹配的内容不会被捕获。没有环视断言，引号会出现在结果中：

```js
> 'how "are" "you" doing'.match(/"([a-z]+)"/g)
[ '"are"', '"you"' ]
```

### 16.4 示例：指定匹配之前或之后不出现的内容（负环视）

我们如何实现与前一节相反的操作，并从字符串中提取所有未引用的单词？

+   输入：`'how "are" "you" doing'`

+   输出：`['how', 'doing']`

我们的第一个尝试是将正环视断言简单地转换为负环视断言。然而，这种方法失败了：

```js
> 'how "are" "you" doing'.match(/(?<!")[a-z]+(?!")/g)
[ 'how', 'r', 'o', 'doing' ]
```

问题在于我们提取了不被引号括起来的字符序列。这意味着在字符串`'"are"'`中，“are”中间的“r”被认为是未引用的，因为它前面是“a”，后面是“e”。

我们可以通过声明前缀和后缀不能是引号或字母来解决这个问题：

```js
> 'how "are" "you" doing'.match(/(?<!["a-z])[a-z]+(?!["a-z])/g)
[ 'how', 'doing' ]
```

另一个解决方案是通过`\b`要求字符序列`[a-z]+`在单词边界开始和结束：

```js
> 'how "are" "you" doing'.match(/(?<!")\b[a-z]+\b(?!")/g)
[ 'how', 'doing' ]
```

负回顾断言和负前瞻断言的一个好处是它们也可以分别在字符串的开头或结尾工作，正如示例中所演示的那样。

#### 16.4.1 没有负环视断言的简单替代方案

负环视断言是一个强大的工具，通常无法通过其他正则表达式手段来模拟。

如果我们不想使用它们，通常必须采取完全不同的方法。例如，在这种情况下，我们可以将字符串拆分为（带引号和不带引号的）单词，然后过滤这些单词：

```js
const str = 'how "are" "you" doing';

const allWords = str.match(/"?[a-z]+"?/g);
const unquotedWords = allWords.filter(
 w => !w.startsWith('"') || !w.endsWith('"'));
assert.deepEqual(unquotedWords, ['how', 'doing']);
```

这种方法的好处：

+   它适用于旧版引擎。

+   这很容易理解。

### 16.5 插曲：指向内部的环视断言

到目前为止，我们所看到的所有示例都有一个共同点，即环视断言规定了匹配之前或之后必须出现的内容，但不包括这些字符在匹配中。

本章其余部分显示的正则表达式是不同的：它们的环视断言指向内部并限制了匹配中的内容。

### 16.6 示例：匹配不以`'abc'`开头的字符串

假设我们想匹配所有不以`'abc'`开头的字符串。我们的第一次尝试可能是正则表达式`/^(?!abc)/`。

这对`.test()`来说效果很好：

```js
> /^(?!abc)/.test('xyz')
true
```

然而，`.exec()`给了我们一个空字符串：

```js
> /^(?!abc)/.exec('xyz')
{ 0: '', index: 0, input: 'xyz', groups: undefined }
```

问题在于像环视断言这样的断言不会扩展匹配的文本。也就是说，它们不会捕获输入字符，它们只是对输入中的当前位置提出要求。

因此，解决方案是添加一个能够捕获输入字符的模式：

```js
> /^(?!abc).*$/.exec('xyz')
{ 0: 'xyz', index: 0, input: 'xyz', groups: undefined }
```

如期望的那样，这个新的正则表达式拒绝了以`'abc'`为前缀的字符串：

```js
> /^(?!abc).*$/.exec('abc')
null
> /^(?!abc).*$/.exec('abcd')
null
```

它接受了不具有完整前缀的字符串：

```js
> /^(?!abc).*$/.exec('ab')
{ 0: 'ab', index: 0, input: 'ab', groups: undefined }
```

### 16.7 示例：匹配不包含`'.mjs'`的子字符串

在下面的示例中，我们想要找到

```js
import ··· from '«module-specifier»';
```

其中`module-specifier`不以`'.mjs'`结尾。

```js
const code = `
import {transform} from './util';
import {Person} from './person.mjs';
import {zip} from 'lodash';
`.trim();
assert.deepEqual(
 code.match(/^import .*? from '[^']+(?<!\.mjs)';$/umg),
 [
 "import {transform} from './util';",
 "import {zip} from 'lodash';",
 ]);
```

在这里，回顾断言`(?<!\.mjs)`充当了一个*保护*，防止正则表达式匹配包含`'.mjs`’的字符串。

### 16.8 示例：跳过带有注释的行

场景：我们想解析带有设置的行，同时跳过注释。例如：

```js
const RE_SETTING = /^(?!#)([^:]*):(.*)$/

const lines = [
 'indent: 2', // setting
 '# Trim trailing whitespace:', // comment
 'whitespace: trim', // setting
];
for (const line of lines) {
 const match = RE_SETTING.exec(line);
 if (match) {
 const key = JSON.stringify(match[1]);
 const value = JSON.stringify(match[2]);
 console.log(`KEY: ${key} VALUE: ${value}`);
 }
}

// Output:
// 'KEY: "indent" VALUE: " 2"'
// 'KEY: "whitespace" VALUE: " trim"'
```

我们是如何得到正则表达式`RE_SETTING`的？

我们从以下正则表达式开始处理设置：

```js
/^([^:]*):(.*)$/
```

直观地，它是以下部分的序列：

+   行的开头

+   非冒号（零个或多个）

+   一个冒号

+   任何字符（零个或多个）

+   行的末尾

这个正则表达式确实拒绝了*一些*注释：

```js
> /^([^:]*):(.*)$/.test('# Comment')
false
```

但它接受其他的（其中有冒号）：

```js
> /^([^:]*):(.*)$/.test('# Comment:')
true
```

我们可以通过在前面加上`(?!#)`来修复这个问题。直观地，它的意思是：“输入字符串中的当前位置不能紧跟着字符`#`。”

新的正则表达式按预期工作：

```js
> /^(?!#)([^:]*):(.*)$/.test('# Comment:')
false
```

### 16.9 示例：智能引号

假设我们想将成对的直引号转换为弯引号：

+   输入：``"yes" and "no"``

+   输出：``“yes” and “no”``

这是我们的第一次尝试：

```js
> `The words "must" and "should".`.replace(/"(.*)"/g, '“$1”')
'The words “must" and "should”.'
```

只有第一个引号和最后一个引号是卷曲的。问题在于`*`量词会[*贪婪地*](https://exploringjs.com/impatient-js/ch_regexps.html#quantifiers)匹配（尽可能多地匹配）。

如果我们在`*`后面加上一个问号，它会*勉强地*匹配：

```js
> `The words "must" and "should".`.replace(/"(.*?)"/g, '“$1”')
'The words “must” and “should”.'
```

#### 16.9.1 通过反斜杠支持转义

如果我们想要通过反斜杠允许引号的转义怎么办？我们可以在引号之前使用保护符`(?<!\\)`来做到这一点：

```js
> const regExp = /(?<!\\)"(.*?)(?<!\\)"/g;
> String.raw`\"straight\" and "curly"`.replace(regExp, '“$1”')
'\\"straight\\" and “curly”'
```

作为后处理步骤，我们仍然需要做：

```js
.replace(/\\"/g, `"`)
```

然而，当有一个反斜杠转义的反斜杠时，这个正则表达式可能会失败：

```js
> String.raw`Backslash: "\\"`.replace(/(?<!\\)"(.*?)(?<!\\)"/g, '“$1”')
'Backslash: "\\\\"'
```

第二个反斜杠阻止了引号变成卷曲的形状。

如果我们让我们的保护符更复杂一些，我们可以解决这个问题（`?:`使得该组不被捕获）：

```js
(?<=^\\*)
```

新的保护符允许在引号之前有一对反斜杠：

```js
> const regExp = /(?<=^\\*)"(.*?)(?<=^\\*)"/g;
> String.raw`Backslash: "\\"`.replace(regExp, '“$1”')
'Backslash: “\\\\”'
```

还有一个问题。这个保护符会阻止第一个引号在字符串开头时被匹配到：

```js
> const regExp = /(?<=^\\*)"(.*?)(?<=^\\*)"/g;
> `"abc"`.replace(regExp, '“$1”')
'"abc"'
```

我们可以通过将第一个保护符改为：`(?<=^\\*|^)`来解决这个问题

```js
> const regExp = /(?<=^\\*|^)"(.*?)(?<=^\\*)"/g;
> `"abc"`.replace(regExp, '“$1”')
'“abc”'
```

### 16.10 致谢

+   第一个处理引号前转义反斜杠的正则表达式[是由`@jonasraoni`在 Twitter 上提出的](https://twitter.com/jonasraoni/status/992506010454683650)。

### 16.11 进一步阅读

+   [章节“正则表达式（`RegExp`）”](https://exploringjs.com/impatient-js/ch_regexps.html) 在“JavaScript 程序员的急切指南”中

[评论](https://github.com/rauschma/deep-js/issues/16)
