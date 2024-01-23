# 十六、使用 util.parseArgs()解析命令行参数

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_node-util-parseargs.html`](https://exploringjs.com/nodejs-shell-scripting/ch_node-util-parseargs.html)

* * *

+   16.1 本章中隐含的导入

+   16.2 处理命令行参数的步骤

+   16.3 解析命令行参数

    +   16.3.1 基础知识

    +   16.3.2 多次使用选项

    +   16.3.3 更多使用长选项和短选项的方式

    +   16.3.4 引用值

    +   16.3.5 选项终结符

    +   16.3.6 严格的`parseArgs()`

+   16.4 `parseArgs` tokens

    +   16.4.1 令牌示例

    +   16.4.2 使用令牌实现子命令

* * *

在这一章中，我们将探讨如何使用 Node.js 模块`node:util`中的`parseArgs()`函数来解析命令行参数。

### 16.1 本章中隐含的导入

在本章的每个示例中都隐含了以下两个导入：

```js
import * as assert from 'node:assert/strict';
import {parseArgs} from 'node:util';
```

第一个导入是用于测试断言，用于检查值。第二个导入是用于本章主题`parseArgs()`函数。

### 16.2 处理命令行参数的步骤

处理命令行参数涉及以下步骤：

1.  用户输入一个文本字符串。

1.  shell 将字符串解析为一系列单词和操作符。

1.  如果调用一个命令，它会得到零个或多个单词作为参数。

1.  我们的 Node.js 代码通过存储在`process.argv`中的数组接收这些单词。[`process`](https://nodejs.org/api/globals.html#process)是 Node.js 上的全局变量。

1.  我们使用`parseArgs()`将数组转换为更方便处理的形式。

让我们使用以下的 shell 脚本`args.mjs`和 Node.js 代码来看看`process.argv`是什么样子的：

```js
#!/usr/bin/env node
console.log(process.argv);
```

我们从一个简单的命令开始：

```js
% ./args.mjs one two
[ '/usr/bin/node', '/home/john/args.mjs', 'one', 'two' ]
```

如果我们在 Windows 上通过 npm 安装命令，同样的命令在 Windows 命令 shell 上会产生以下结果：

```js
[
 'C:\\Program Files\\nodejs\\node.exe',
 'C:\\Users\\jane\\args.mjs',
 'one',
 'two'
]
```

无论我们如何调用 shell 脚本，`process.argv`始终以用于运行我们的代码的 Node.js 二进制文件的路径开始。接下来是我们脚本的路径。数组以传递给脚本的实际参数结束。换句话说：脚本的参数始终从索引 2 开始。

因此，我们改变我们的脚本，使其看起来像这样：

```js
#!/usr/bin/env node
console.log(process.argv.slice(2));
```

让我们尝试更复杂的参数：

```js
% ./args.mjs --str abc --bool home.html main.js
[ '--str', 'abc', '--bool', 'home.html', 'main.js' ]
```

这些参数包括：

+   选项`--str`，其值是文本`abc`。这样的选项称为*字符串选项*。

+   选项`--bool`，它没有关联的值 - 它是一个标志，要么存在要么不存在。这样的选项称为*布尔选项*。

+   两个所谓的*位置参数*，没有名称：`home.html`和`main.js`。

常见的使用参数的两种方式：

+   主要参数是位置参数，选项提供额外的 - 通常是可选的 - 信息。

+   只使用选项。

作为 JavaScript 函数调用，前面的示例看起来像这样（在 JavaScript 中，选项通常放在最后）：

```js
argsMjs('home.html', 'main.js', {str: 'abc', bool: false});
```

### 16.3 解析命令行参数

#### 16.3.1 基础知识

如果我们希望`parseArgs()`解析带有参数的数组，我们首先需要告诉它我们的选项是如何工作的。假设我们的脚本有：

+   一个布尔选项`--verbose`

+   一个接收非负整数的选项`--times`。`parseArgs()`对数字没有特殊支持，所以我们必须将其作为字符串选项。

+   一个字符串选项`--color`

我们将这些选项描述为 `parseArgs()` 如下：

```js
const options = {
 'verbose': {
 type: 'boolean',
 short: 'v',
 },
 'color': {
 type: 'string',
 short: 'c',
 },
 'times': {
 type: 'string',
 short: 't',
 },
};
```

只要 `options` 的属性键是有效的 JavaScript 标识符，你可以选择是否引用它。两者都有利弊。在本章中，它们总是被引用。这样，具有非标识符名称的选项（如 `my-new-option`）看起来与具有标识符名称的选项相同。

`options` 中的每个条目都可以具有以下属性（通过 TypeScript 类型定义）：

```js
type Options = {
 type: 'boolean' | 'string', // required
 short?: string, // optional
 multiple?: boolean, // optional, default `false`
};
```

+   `.type` 指定选项是布尔值还是字符串。

+   `.short` 定义选项的短版本。必须是单个字符。我们很快将看到如何使用短版本。

+   `.multiple` 指示选项是否最多可以使用一次或零次或多次。稍后我们将看到这意味着什么。

以下代码使用 `parseArgs()` 和 `options` 解析带有参数的数组：

```js
assert.deepEqual(
 parseArgs({options, args: [
 '--verbose', '--color', 'green', '--times', '5'
 ]}),
 {
 values: {__proto__:null,
 verbose: true,
 color: 'green',
 times: '5'
 },
 positionals: []
 }
);
```

存储在 `.values` 中的对象的原型是 `null`。这意味着我们可以使用 `in` 运算符来检查属性是否存在，而不必担心继承的属性，如 `.toString`。

如前所述，`--times` 的值为 5，被处理为字符串。

我们传递给 `parseArgs()` 的对象具有以下 TypeScript 类型：

```js
type ParseArgsProps = {
 options?: {[key: string], Options}, // optional, default: {}
 args?: Array<string>, // optional
 // default: process.argv.slice(2)
 strict?: boolean, // optional, default `true`
 allowPositionals?: boolean, // optional, default `false`
};
```

+   `.args`：要解析的参数。如果省略此属性，`parseArgs()` 将使用 `process.argv`，从索引 2 开始。

+   `.strict`：如果为 `true`，则如果 `args` 不正确，将抛出异常。稍后详细介绍。

+   `.allowPositionals`：`args` 是否包含位置参数？

这是 `parseArgs()` 的结果类型：

```js
type ParseArgsResult = {
 values: {[key: string]: ValuesValue}, // an object
 positionals: Array<string>, // always an Array
};
type ValuesValue = boolean | string | Array<boolean|string>;
```

+   `.values` 包含可选参数。我们已经看到字符串和布尔值作为属性值。当我们探索具有 `.multiple` 为 `true` 的选项定义时，我们将看到数组值属性。

+   `.positionals` 包含位置参数。

两个连字符用于引用选项的长版本。一个连字符用于引用短版本：

```js
assert.deepEqual(
 parseArgs({options, args: ['-v', '-c', 'green']}),
 {
 values: {__proto__:null,
 verbose: true,
 color: 'green',
 },
 positionals: []
 }
);
```

请注意，`.values` 包含选项的长名称。

我们通过解析混合了可选参数的位置参数来结束本小节：

```js
assert.deepEqual(
 parseArgs({
 options,
 allowPositionals: true,
 args: [
 'home.html', '--verbose', 'main.js', '--color', 'red', 'post.md'
 ]
 }),
 {
 values: {__proto__:null,
 verbose: true,
 color: 'red',
 },
 positionals: [
 'home.html', 'main.js', 'post.md'
 ]
 }
);
```

#### 16.3.2 多次使用选项

如果我们多次使用一个选项，默认情况下只有最后一次计数。它会覆盖所有先前的出现：

```js
const options = {
 'bool': {
 type: 'boolean',
 },
 'str': {
 type: 'string',
 },
};

assert.deepEqual(
 parseArgs({
 options, args: [
 '--bool', '--bool', '--str', 'yes', '--str', 'no'
 ]
 }),
 {
 values: {__proto__:null,
 bool: true,
 str: 'no'
 },
 positionals: []
 }
);
```

然而，如果我们在选项的定义中将 `.multiple` 设置为 `true`，`parseArgs()` 将以数组形式给出所有选项值：

```js
const options = {
 'bool': {
 type: 'boolean',
 multiple: true,
 },
 'str': {
 type: 'string',
 multiple: true,
 },
};

assert.deepEqual(
 parseArgs({
 options, args: [
 '--bool', '--bool', '--str', 'yes', '--str', 'no'
 ]
 }),
 {
 values: {__proto__:null,
 bool: [ true, true ],
 str: [ 'yes', 'no' ]
 },
 positionals: []
 }
);
```

#### 16.3.3 更多使用长和短选项的方式

考虑以下选项：

```js
const options = {
 'verbose': {
 type: 'boolean',
 short: 'v',
 },
 'silent': {
 type: 'boolean',
 short: 's',
 },
 'color': {
 type: 'string',
 short: 'c',
 },
};
```

以下是使用多个布尔选项的简洁方式：

```js
assert.deepEqual(
 parseArgs({options, args: ['-vs']}),
 {
 values: {__proto__:null,
 verbose: true,
 silent: true,
 },
 positionals: []
 }
);
```

我们可以通过等号直接附加长字符串选项的值。这称为*内联值*。

```js
assert.deepEqual(
 parseArgs({options, args: ['--color=green']}),
 {
 values: {__proto__:null,
 color: 'green'
 },
 positionals: []
 }
);
```

短选项不能有内联值。

#### 16.3.4 引用值

到目前为止，所有选项值和位置值都是单词。如果我们想使用包含空格的值，我们需要用双引号或单引号引起来。然而，并非所有 shell 都支持后者。

##### 16.3.4.1 Shell 如何解析带引号的值

为了检查 shell 如何解析带引号的值，我们再次使用脚本 `args.mjs`：

```js
#!/usr/bin/env node
console.log(process.argv.slice(2));
```

在 Unix 上，双引号和单引号之间的区别如下：

+   双引号：我们可以用反斜杠转义引号（否则原样传递），并且可以插入变量：

    ```js
    % ./args.mjs "say \"hi\"" "\t\n" "$USER"
    [ 'say "hi"', '\\t\\n', 'rauschma' ]
    ```

+   单引号：所有内容都原样传递，我们无法转义引号：

    ```js
    % ./args.mjs 'back slash\' '\t\n' '$USER' 
    [ 'back slash\\', '\\t\\n', '$USER' ]
    ```

以下交互演示了双引号和单引号的选项值：

```js
% ./args.mjs --str "two words" --str 'two words'
[ '--str', 'two words', '--str', 'two words' ]

% ./args.mjs --str="two words" --str='two words'
[ '--str=two words', '--str=two words' ]

% ./args.mjs -s "two words" -s 'two words'
[ '-s', 'two words', '-s', 'two words' ]
```

在 Windows 命令 shell 中，单引号没有任何特殊含义：

```js
>node args.mjs "say \"hi\"" "\t\n" "%USERNAME%"
[ 'say "hi"', '\\t\\n', 'jane' ]

>node args.mjs 'back slash\' '\t\n' '%USERNAME%'
[ "'back", "slash\\'", "'\\t\\n'", "'jane'" ]
```

在 Windows 命令 shell 中引用的选项值：

```js
>node args.mjs --str 'two words' --str "two words"
[ '--str', "'two", "words'", '--str', 'two words' ]

>node args.mjs --str='two words' --str="two words"
[ "--str='two", "words'", '--str=two words' ]

>>node args.mjs -s "two words" -s 'two words'
[ '-s', 'two words', '-s', "'two", "words'" ]
```

在 Windows PowerShell 中，我们可以用单引号引用，变量名不会在引号内插值，而且单引号无法转义：

```js
> node args.mjs "say `"hi`"" "\t\n" "%USERNAME%"
[ 'say hi', '\\t\\n', '%USERNAME%' ]
> node args.mjs 'backtick`' '\t\n' '%USERNAME%'
[ 'backtick`', '\\t\\n', '%USERNAME%' ]
```

##### 16.3.4.2 `parseArgs()` 如何处理带引号的值

这是 `parseArgs()` 如何处理带引号的值：

```js
const options = {
 'times': {
 type: 'string',
 short: 't',
 },
 'color': {
 type: 'string',
 short: 'c',
 },
};

// Quoted external option values
assert.deepEqual(
 parseArgs({
 options,
 args: ['-t', '5 times', '--color', 'light green']
 }),
 {
 values: {__proto__:null,
 times: '5 times',
 color: 'light green',
 },
 positionals: []
 }
);

// Quoted inline option values
assert.deepEqual(
 parseArgs({
 options,
 args: ['--color=light green']
 }),
 {
 values: {__proto__:null,
 color: 'light green',
 },
 positionals: []
 }
);

// Quoted positional values
assert.deepEqual(
 parseArgs({
 options, allowPositionals: true,
 args: ['two words', 'more words']
 }),
 {
 values: {__proto__:null,
 },
 positionals: [ 'two words', 'more words' ]
 }
);
```

#### 16.3.5 选项终结符

`parseArgs()` 支持所谓的*选项终结符*：如果 `args` 的一个元素是双连字符（`--`），那么剩余的参数都被视为位置参数。

选项终止符在哪里需要？一些可执行文件调用其他可执行文件，例如[node 可执行文件](https://nodejs.org/api/cli.html#--)。然后，可以使用选项终止符将调用者的参数与被调用者的参数分开。

这是`parseArgs()`处理选项终止符的方式：

```js
const options = {
 'verbose': {
 type: 'boolean',
 },
 'count': {
 type: 'string',
 },
};

assert.deepEqual(
 parseArgs({options, allowPositionals: true,
 args: [
 'how', '--verbose', 'are', '--', '--count', '5', 'you'
 ]
 }),
 {
 values: {__proto__:null,
 verbose: true
 },
 positionals: [ 'how', 'are', '--count', '5', 'you' ]
 }
);
```

#### 16.3.6 严格的`parseArgs()`

如果选项`.strict`为`true`（这是默认值），那么`parseArgs()`如果发生以下情况之一，将抛出异常：

+   `args`中使用的选项名称不在`options`中。

+   `args`中的选项类型错误。目前，仅当字符串选项缺少参数时才会发生这种情况。

+   `args`中有位置参数，即使`.allowPositions`为`false`（这是默认值）。

以下代码演示了每种情况：

```js
const options = {
 'str': {
 type: 'string',
 },
};

// Unknown option name
assert.throws(
 () => parseArgs({
 options,
 args: ['--unknown']
 }),
 {
 name: 'TypeError',
 message: "Unknown option '--unknown'",
 }
);

// Wrong option type (missing value)
assert.throws(
 () => parseArgs({
 options,
 args: ['--str']
 }),
 {
 name: 'TypeError',
 message: "Option '--str <value>' argument missing",
 }
);

// Unallowed positional
assert.throws(
 () => parseArgs({
 options,
 allowPositionals: false, // (the default)
 args: ['posarg']
 }),
 {
 name: 'TypeError',
 message: "Unexpected argument 'posarg'. " +
 "This command does not take positional arguments",
 }
);
```

### 16.4 `parseArgs`标记

`parseArgs()`在两个阶段处理`args`数组：

+   阶段 1：它将`args`解析为标记数组：这些标记大多是带有类型信息的`args`元素：它是一个选项吗？它是一个位置参数吗？等等。但是，如果选项有一个值，那么标记将存储选项名称和选项值，因此包含两个`args`元素的数据。

+   阶段 2：它将标记组装成通过结果属性`.values`返回的对象。

如果将`config.tokens`设置为`true`，则可以访问标记。然后，`parseArgs()`返回的对象包含一个名为`.tokens`的属性，其中包含标记。

这些是标记的属性：

```js
type Token = OptionToken | PositionalToken | OptionTerminatorToken;

interface CommonTokenProperties {
 /** Where in `args` does the token start? */
 index: number;
}

interface OptionToken extends CommonTokenProperties {
 kind: 'option';

 /** Long name of option */
 name: string;

 /** The option name as mentioned in `args` */
 rawName: string;

 /** The option’s value. `undefined` for boolean options. */
 value: string | undefined;

 /** Is the option value specified inline (e.g. --level=5)? */
 inlineValue: boolean | undefined;
}

interface PositionalToken extends CommonTokenProperties {
 kind: 'positional';

 /** The value of the positional, args[token.index] */
 value: string;
}

interface OptionTerminatorToken extends CommonTokenProperties {
 kind: 'option-terminator';
}
```

#### 16.4.1 标记示例

例如，考虑以下选项：

```js
const options = {
 'bool': {
 type: 'boolean',
 short: 'b',
 },
 'flag': {
 type: 'boolean',
 short: 'f',
 },
 'str': {
 type: 'string',
 short: 's',
 },
};
```

布尔选项的标记如下：

```js
assert.deepEqual(
 parseArgs({
 options, tokens: true,
 args: [
 '--bool', '-b', '-bf',
 ]
 }),
 {
 values: {__proto__:null,
 bool: true,
 flag: true,
 },
 positionals: [],
 tokens: [
 {
 kind: 'option',
 name: 'bool',
 rawName: '--bool',
 index: 0,
 value: undefined,
 inlineValue: undefined
 },
 {
 kind: 'option',
 name: 'bool',
 rawName: '-b',
 index: 1,
 value: undefined,
 inlineValue: undefined
 },
 {
 kind: 'option',
 name: 'bool',
 rawName: '-b',
 index: 2,
 value: undefined,
 inlineValue: undefined
 },
 {
 kind: 'option',
 name: 'flag',
 rawName: '-f',
 index: 2,
 value: undefined,
 inlineValue: undefined
 },
 ]
 }
);
```

请注意，对于选项`bool`，有三个标记，因为它在`args`中被提及三次。但是，由于解析的第二阶段，`.values`中只有一个`bool`属性。

在下一个示例中，我们将字符串选项解析为标记。`.inlineValue`现在具有布尔值（对于布尔选项，它始终为`undefined`）：

```js
assert.deepEqual(
 parseArgs({
 options, tokens: true,
 args: [
 '--str', 'yes', '--str=yes', '-s', 'yes',
 ]
 }),
 {
 values: {__proto__:null,
 str: 'yes',
 },
 positionals: [],
 tokens: [
 {
 kind: 'option',
 name: 'str',
 rawName: '--str',
 index: 0,
 value: 'yes',
 inlineValue: false
 },
 {
 kind: 'option',
 name: 'str',
 rawName: '--str',
 index: 2,
 value: 'yes',
 inlineValue: true
 },
 {
 kind: 'option',
 name: 'str',
 rawName: '-s',
 index: 3,
 value: 'yes',
 inlineValue: false
 }
 ]
 }
);
```

最后，这是解析位置参数和选项终止符的示例：

```js
assert.deepEqual(
 parseArgs({
 options, allowPositionals: true, tokens: true,
 args: [
 'command', '--', '--str', 'yes', '--str=yes'
 ]
 }),
 {
 values: {__proto__:null,
 },
 positionals: [ 'command', '--str', 'yes', '--str=yes' ],
 tokens: [
 { kind: 'positional', index: 0, value: 'command' },
 { kind: 'option-terminator', index: 1 },
 { kind: 'positional', index: 2, value: '--str' },
 { kind: 'positional', index: 3, value: 'yes' },
 { kind: 'positional', index: 4, value: '--str=yes' }
 ]
 }
);
```

#### 16.4.2 使用标记实现子命令

默认情况下，`parseArgs()`不支持`git clone`或`npm install`等子命令。但是，通过标记，相对容易实现此功能。

这是实现方式：

```js
function parseSubcommand(config) {
 // The subcommand is a positional, allow them
 const {tokens} = parseArgs({
 ...config, tokens: true, allowPositionals: true
 });
 let firstPosToken = tokens.find(({kind}) => kind==='positional');
 if (!firstPosToken) {
 throw new Error('Command name is missing: ' + config.args);
 }

 //----- Command options

 const cmdArgs = config.args.slice(0, firstPosToken.index);
 // Override `config.args`
 const commandResult = parseArgs({
 ...config, args: cmdArgs, tokens: false, allowPositionals: false
 });

 //----- Subcommand

 const subcommandName = firstPosToken.value;

 const subcmdArgs = config.args.slice(firstPosToken.index+1);
 // Override `config.args`
 const subcommandResult = parseArgs({
 ...config, args: subcmdArgs, tokens: false
 });

 return {
 commandResult,
 subcommandName,
 subcommandResult,
 };
}
```

这是`parseSubcommand()`的示例：

```js
const options = {
 'log': {
 type: 'string',
 },
 color: {
 type: 'boolean',
 }
};
const args = ['--log', 'all', 'print', '--color', 'file.txt'];
const result = parseSubcommand({options, allowPositionals: true, args});

const pn = obj => Object.setPrototypeOf(obj, null);
assert.deepEqual(
 result,
 {
 commandResult: {
 values: pn({'log': 'all'}),
 positionals: []
 },
 subcommandName: 'print',
 subcommandResult: {
 values: pn({color: true}),
 positionals: ['file.txt']
 }
 }
);
```

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/16)
