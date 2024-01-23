# 21 使用模板文字和标记模板

> 原文：[`exploringjs.com/impatient-js/ch_template-literals.html`](https://exploringjs.com/impatient-js/ch_template-literals.html)

* * *

+   21.1 消歧：“模板”

+   21.2 模板文字

+   21.3 标记模板

    +   21.3.1 熟练与原始模板字符串（高级）(ch_template-literals.html#template-strings-cooked-vs-raw)

+   21.4 标记模板的示例（通过库提供）

    +   21.4.1 标记函数库：lit-html

    +   21.4.2 标记函数库：re-template-tag

    +   21.4.3 标记函数库：graphql-tag

+   21.5 原始字符串文字

+   21.6 （高级）(ch_template-literals.html#advanced-3)

+   21.7 多行模板文字和缩进(ch_template-literals.html#multiline-template-literals-and-indentation)

    +   21.7.1 修复：用于去除缩进的模板标记(ch_template-literals.html#fix-template-tag-for-dedenting)

    +   21.7.2 修复：`.trim()`(ch_template-literals.html#fix-.trim)

+   21.8 通过模板文字进行简单模板化

    +   21.8.1 更复杂的例子

    +   21.8.2 简单的 HTML 转义

* * *

在我们深入研究*模板文字*和*标记模板*这两个功能之前，让我们先来看看术语*模板*的多重含义。

### 21.1 消歧：“模板”

尽管所有这些名称中都有*模板*，并且它们看起来都很相似，但以下三个事物在很大程度上是不同的：

+   *文本模板*是从数据到文本的函数。它经常用于 Web 开发，并经常通过文本文件定义。例如，以下文本定义了库[Handlebars](https://handlebarsjs.com)的模板：

    ```js
    <div class="entry">
     <h1>{{title}}</h1>
     <div class="body">
     {{body}}
     </div>
    </div>
    ```

    这个模板有两个空白需要填写：`title`和`body`。它的使用方式如下：

    ```js
    // First step: retrieve the template text, e.g. from a text file.
    const tmplFunc = Handlebars.compile(TMPL_TEXT); // compile string
    const data = {title: 'My page', body: 'Welcome to my page!'};
    const html = tmplFunc(data);
    ```

+   *模板文字*类似于字符串文字，但具有附加功能-例如插值。它由反引号界定：

    ```js
    const num = 5;
    assert.equal(`Count: ${num}!`, 'Count: 5!');
    ```

+   在语法上，*标记模板*是一个遵循函数（或者更确切地说，求值为函数的表达式）的模板文字。这导致函数被调用。它的参数来自模板文字的内容。

    ```js
    const getArgs = (...args) => args;
    assert.deepEqual(
     getArgs`Count: ${5}!`,
     [['Count: ', '!'], 5] );
    ```

    请注意，`getArgs()`接收文字模板的文本和通过`${}`插值的数据。

### 21.2 模板文字

模板文字与普通字符串文字相比具有两个新功能。

首先，它支持*字符串插值*：如果我们将动态计算的值放在`${}`中，它将被转换为字符串并插入到文字模板返回的字符串中。

```js
const MAX = 100;
function doSomeWork(x) {
 if (x > MAX) {
 throw new Error(`At most ${MAX} allowed: ${x}!`);
 }
 // ···
}
assert.throws(
 () => doSomeWork(101),
 {message: 'At most 100 allowed: 101!'});
```

其次，模板文字可以跨越多行：

```js
const str = `this is
a text with
multiple lines`;
```

模板文字始终产生字符串。

### 21.3 标记模板

A 行中的表达式是*标记模板*。它相当于使用 B 行中数组中列出的参数调用`tagFunc()`。

```js
function tagFunc(...args) {
 return args;
}

const setting = 'dark mode';
const value = true;

assert.deepEqual(
 tagFunc`Setting ${setting} is ${value}!`, // (A)
 [['Setting ', ' is ', '!'], 'dark mode', true] // (B)
);
```

第一个反引号之前的函数`tagFunc`称为*标记函数*。它的参数是：

+   *模板字符串*（第一个参数）：一个包含围绕插值`${}`的文本片段的数组。

    +   在示例中：`['Setting ', ' is ', '!']`

+   *替换*（剩余参数）：插值的值。

    +   在示例中：`'dark mode'`和`true`

文字模板的静态（固定）部分（模板字符串）与动态部分（替换）保持分开。

标记函数可以返回任意值。

#### 21.3.1 熟练与原始模板字符串（高级）

到目前为止，我们只看到了模板字符串的*cooked 解释*。但是标签函数实际上有两种解释：

+   *cooked 解释*，其中反斜杠具有特殊含义。例如，`\t`会产生一个制表符。模板字符串的这种解释存储为第一个参数中的数组。

+   *原始解释*，其中反斜杠没有特殊含义。例如，`\t`会产生两个字符 - 一个反斜杠和一个`t`。模板字符串的这种解释存储在第一个参数（一个数组）的`.raw`属性中。

原始解释通过`String.raw`启用原始字符串文字（稍后描述）和类似的应用。

以下标签函数`cookedRaw`使用了两种解释：

```js
function cookedRaw(templateStrings, ...substitutions) {
 return {
 cooked: Array.from(templateStrings), // copy only Array elements
 raw: templateStrings.raw,
 substitutions,
 };
}
assert.deepEqual(
 cookedRaw`\tab${'subst'}\newline\\`,
 {
 cooked: ['\tab', '\newline\\'],
 raw:    ['\\tab', '\\newline\\\\'],
 substitutions: ['subst'],
 });
```

我们还可以在标记模板中使用 Unicode 代码点转义（`\u{1F642}`），Unicode 代码单元转义（`\u03A9`）和 ASCII 转义（`\x52`）：

```js
assert.deepEqual(
 cookedRaw`\u{54}\u0065\x78t`,
 {
 cooked: ['Text'],
 raw:    ['\\u{54}\\u0065\\x78t'],
 substitutions: [],
 });
```

如果其中一个转义的语法不正确，相应的 cooked 模板字符串是`undefined`，而原始版本仍然是逐字的：

```js
assert.deepEqual(
 cookedRaw`\uu\xx ${1} after`,
 {
 cooked: [undefined, ' after'],
 raw:    ['\\uu\\xx ', ' after'],
 substitutions: [1],
 });
```

在模板文字和字符串文字中，不正确的转义会产生语法错误。在 ES2018 之前，它们甚至在标记模板中产生错误。为什么要改变呢？现在我们可以使用标记模板来处理以前非法的文本 - 例如：

```js
windowsPath`C:\uuu\xxx\111`
latex`\unicode`
```

### 21.4 标记模板的示例（通过库提供）

标记模板非常适合支持小型嵌入式语言（所谓的*领域特定语言*）。我们将继续举几个例子。

#### 21.4.1 标签函数库：lit-html

[lit-html](https://github.com/Polymer/lit-html)是一个基于标记模板的模板库，被[前端框架 Polymer](https://www.polymer-project.org/)使用：

```js
import {html, render} from 'lit-html';

const template = (items) => html`
 <ul>
 ${
 repeat(items,
 (item) => item.id,
 (item, index) => html`<li>${index}. ${item.name}</li>`
 )
 }
 </ul>
`;
```

`repeat()`是一个用于循环的自定义函数。它的第二个参数为第三个参数返回的值生成唯一的键。请注意该参数使用的嵌套标记模板。

#### 21.4.2 标签函数库：re-template-tag

[re-template-tag](https://github.com/rauschma/re-template-tag)是一个简单的库，用于组合正则表达式。使用`re`标记的模板会产生正则表达式。主要好处是我们可以通过`${}`（A 行）插入正则表达式和纯文本：

```js
const RE_YEAR = re`(?<year>[0-9]{4})`;
const RE_MONTH = re`(?<month>[0-9]{2})`;
const RE_DAY = re`(?<day>[0-9]{2})`;
const RE_DATE = re`/${RE_YEAR}-${RE_MONTH}-${RE_DAY}/u`; // (A)

const match = RE_DATE.exec('2017-01-27');
assert.equal(match.groups.year, '2017');
```

#### 21.4.3 标签函数库：graphql-tag

[graphql-tag 库](https://github.com/apollographql/graphql-tag)让我们可以通过标记模板创建 GraphQL 查询：

```js
import gql from 'graphql-tag';

const query = gql`
 {
 user(id: 5) {
 firstName
 lastName
 }
 }
 `;
```

此外，还有用于在 Babel、TypeScript 等中预编译此类查询的插件。

### 21.5 原始字符串文字

原始字符串文字是通过标签函数`String.raw`实现的。它们是字符串文字，其中反斜杠不会做任何特殊处理（例如转义字符等）：

```js
assert.equal(String.raw`\back`, '\\back');
```

这有助于在数据包含反斜杠时使用 - 例如，包含正则表达式的字符串：

```js
const regex1 = /^\./;
const regex2 = new RegExp('^\\.');
const regex3 = new RegExp(String.raw`^\.`);
```

所有三个正则表达式都是等效的。使用普通字符串文字时，我们必须将反斜杠写两次，以便为该文字转义。使用原始字符串文字时，我们不必这样做。

原始字符串文字也可用于指定 Windows 文件名路径：

```js
const WIN_PATH = String.raw`C:\foo\bar`;
assert.equal(WIN_PATH, 'C:\\foo\\bar');
```

### 21.6 （高级）

所有剩余的部分都是高级的

### 21.7 多行模板文字和缩进

如果我们在模板文字中放入多行文本，存在两个目标的冲突：一方面，模板文字应该缩进以适应源代码。另一方面，其内容的行应该从最左边的列开始。

例如：

```js
function div(text) {
 return `
 <div>
 ${text}
 </div>
 `;
}
console.log('Output:');
console.log(
 div('Hello!')
 // Replace spaces with mid-dots:
 .replace(/ /g, '·')
 // Replace \n with #\n:
 .replace(/\n/g, '#\n')
);
```

由于缩进，模板文字很好地适应了源代码。遗憾的是，输出也被缩进了。我们不希望在开头有换行，结尾有换行加两个空格。

```js
Output:
#
····<div>#
······Hello!#
····</div>#
··
```

有两种方法可以解决这个问题：通过标记模板或修剪模板文字的结果。

#### 21.7.1 修复：用于去除缩进的模板标签

第一个修复是使用自定义模板标签来移除不需要的空白。它使用初始换行后的第一行来确定文本从哪一列开始，并在所有地方缩短缩进。它还移除了开头的换行和结尾的缩进。一个这样的模板标签是[Desmond Brand 的`dedent`](https://github.com/dmnd/dedent)：

```js
import dedent from 'dedent';
function divDedented(text) {
 return dedent`
 <div>
 ${text}
 </div>
 `.replace(/\n/g, '#\n');
}
console.log('Output:');
console.log(divDedented('Hello!'));
```

这次，输出没有缩进：

```js
Output:
<div>#
 Hello!#
</div>
```

#### 21.7.2 修复：`.trim()`

第二个修复更快，但也更脏：

```js
function divDedented(text) {
 return `
<div>
 ${text}
</div>
 `.trim().replace(/\n/g, '#\n');
}
console.log('Output:');
console.log(divDedented('Hello!'));
```

字符串方法`.trim()`会移除开头和结尾的多余空白，但内容本身必须从最左边的列开始。这种解决方案的优点是我们不需要自定义标签函数。缺点是它看起来很丑。

输出与`dedent`相同：

```js
Output:
<div>#
 Hello!#
</div>
```

### 21.8 使用模板文字进行简单模板化

虽然模板文字看起来像文本模板，但如何使用它们进行（文本）模板化并不是立即明显的：文本模板从对象中获取数据，而模板文字从变量中获取数据。解决方案是在接收模板数据的函数的主体中使用模板文字，例如：

```js
const tmpl = (data) => `Hello ${data.name}!`;
assert.equal(tmpl({name: 'Jane'}), 'Hello Jane!');
```

#### 21.8.1 一个更复杂的例子

作为一个更复杂的例子，我们想要取一个地址数组并生成一个 HTML 表格。这是数组：

```js
const addresses = [
 { first: '<Jane>', last: 'Bond' },
 { first: 'Lars', last: '<Croft>' },
];
```

生成 HTML 表格的函数`tmpl()`如下所示：

```js
const tmpl = (addrs) => `
<table>
 ${addrs.map(
 (addr) => `
 <tr>
 <td>${escapeHtml(addr.first)}</td>
 <td>${escapeHtml(addr.last)}</td>
 </tr>
 `.trim()
 ).join('')}
</table>
`.trim();
```

此代码包含两个模板函数：

+   第一个函数（第 1 行）接受一个包含地址的数组`addrs`，并返回一个带有表格的字符串。

+   第二个函数（第 4 行）接受一个包含地址的对象`addr`，并返回一个带有表格行的字符串。注意末尾的`.trim()`，它会移除不必要的空白。

第一个模板函数通过将一个包含在 HTML 中的数组包装成一个字符串来生成其结果（第 10 行）。该数组是通过将第二个模板函数映射到`addrs`的每个元素（第 3 行）而生成的。因此，它包含带有表格行的字符串。

辅助函数`escapeHtml()`用于转义特殊的 HTML 字符（第 6 行和第 7 行）。其实现在下一小节中显示。

让我们用地址调用`tmpl()`并记录结果：

```js
console.log(tmpl(addresses));
```

输出是：

```js
<table>
 <tr>
 <td>&lt;Jane&gt;</td>
 <td>Bond</td>
 </tr><tr>
 <td>Lars</td>
 <td>&lt;Croft&gt;</td>
 </tr>
</table>
```

#### 21.8.2 简单的 HTML 转义

以下函数会转义纯文本，以便在 HTML 中原样显示：

```js
function escapeHtml(str) {
 return str
 .replace(/&/g, '&amp;') // first!
 .replace(/>/g, '&gt;')
 .replace(/</g, '&lt;')
 .replace(/"/g, '&quot;')
 .replace(/'/g, '&#39;')
 .replace(/`/g, '&#96;')
 ;
}
assert.equal(
 escapeHtml('Rock & Roll'), 'Rock &amp; Roll');
assert.equal(
 escapeHtml('<blank>'), '&lt;blank&gt;');
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：HTML 模板化**

练习和奖励挑战：`exercises/template-literals/templating_test.mjs`

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/14)
