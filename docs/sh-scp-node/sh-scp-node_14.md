# 11 流配方

> [`exploringjs.com/nodejs-shell-scripting/ch_stream-recipes.html`](https://exploringjs.com/nodejs-shell-scripting/ch_stream-recipes.html)

* * *

+   11.1 写入标准输出(stdout)

    +   11.1.1 通过`console.log()`写入 stdout

    +   11.1.2 通过 Node.js 流写入 stdout

    +   11.1.3 通过 Web 流写入 stdout

+   11.2 写入标准错误(stderr)

+   11.3 从标准输入(stdin)读取

    +   11.3.1 通过 Node.js 流从 stdin 读取

    +   11.3.2 通过 Web 流从 stdin 读取

    +   11.3.3 通过模块`'node:readline'`从 stdin 读取

+   11.4 Node.js 流配方

+   11.5 Web 流配方

* * *

### 11.1 写入标准输出(stdout)

这是写入 stdout 的三个选项：

+   我们可以通过`console.log()`写入它。

+   我们可以通过 Node.js 流写入它。

+   我们可以通过 Web 流写入它。

#### 11.1.1 通过`console.log()`写入 stdout

[`console.log(format, ...args)`](https://nodejs.org/docs/latest/api/console.html#consolelogdata-args)写入 stdout 并始终附加换行符`'\n'`（即使在 Windows 上也是如此）。第一个参数可以包含占位符，这些占位符的解释方式与`util.format()`相同：

```js
console.log('String: %s Number: %d Percent: %%', 'abc', 123);

const obj = {one: 1, two: 2};
console.log('JSON: %j Object: %o', obj, obj);

// Output:
// 'String: abc Number: 123 Percent: %'
// 'JSON: {"one":1,"two":2} Object: { one: 1, two: 2 }'
```

第一个参数之后的所有参数始终显示在输出中，即使没有足够的占位符。

#### 11.1.2 通过 Node.js 流写入 stdout

`process.stdout`是`stream.Readable`的一个实例。这意味着我们可以像使用其他 Node.js 流一样使用它-例如：

```js
process.stdout.write('two');
process.stdout.write(' words');
process.stdout.write('\n');
```

前面的代码等同于：

```js
console.log('two words');
```

请注意，这种情况下末尾没有换行符，因为`console.log()`总是会添加一个。

如果我们使用`.write()`来处理大量数据，我们应该考虑回压，如§9.5.2.1“`writable.write(chunk)`”中所解释的那样。

以下配方适用于`process.stdout`：§11.4“Node.js 流配方”。

#### 11.1.3 通过 Web 流写入 stdout

我们可以将`process.stdout`转换为 Web 流并写入其中：

```js
import {Writable} from 'node:stream';
const webOut = Writable.toWeb(process.stdout);
const writer = webOut.getWriter();
try {
 await writer.write('First line\n');
 await writer.write('Second line\n');
 await writer.close();
} finally {
 writer.releaseLock()
}
```

以下配方适用于`webOut`：§11.5“Web 流配方”。

### 11.2 写入标准错误(stderr)

写入 stderr 的工作方式与写入 stdout 类似：

+   我们可以通过`console.error()`写入它。

+   我们可以通过 Node.js 流写入它。

+   我们可以通过 Web 流写入它。

有关更多信息，请参阅前一节。

### 11.3 从标准输入(stdin)读取

这些是从 stdin 读取的选项：

+   我们可以通过 Node.js 流从中读取。

+   我们可以通过 Web 流从中读取。

+   我们可以使用模块`'node:readline'`。

#### 11.3.1 通过 Node.js 流从 stdin 读取

`process.stdin`是`stream.Writable`的一个实例。这意味着我们可以像使用其他 Node.js 流一样使用它：

```js
// Switch to text mode (otherwise we get chunks of binary data)
process.stdin.setEncoding('utf-8');
for await (const chunk of process.stdin) {
 console.log('>', chunk);
}
```

以下配方适用于`webIn`：§11.4“Node.js 流配方”。

#### 11.3.2 通过 Web 流从 stdin 读取

我们首先必须将`process.stdin`转换为 Web 流：

```js
import {Readable} from 'node:stream';
// Switch to text mode (otherwise we get chunks of binary data)
process.stdin.setEncoding('utf-8');
const webIn = Readable.toWeb(process.stdin);
for await (const chunk of webIn) {
 console.log('>', chunk);
}
```

以下配方适用于`webIn`：§11.5“Web 流配方”。

#### 11.3.3 通过模块`'node:readline'`从 stdin 读取

内置模块`'node:readline'`允许我们提示用户以交互方式输入信息-例如：

```js
import * as fs from 'node:fs';
import * as readline from 'node:readline/promises';

const rl = readline.createInterface({
 input: process.stdin,
 output: process.stdout,
});

const filePath = await rl.question('Please enter a file path: ');
fs.writeFileSync(filePath, 'Hi!', {encoding: 'utf-8'})

rl.close();
```

有关模块`'node:readline'`的更多信息，请参见：

+   §9.3.3“通过模块'node:readlines'从可读流中读取行”

+   [官方文档](https://nodejs.org/api/readline.html)。

### 11.4 Node.js 流配方

可读流：

+   §9.3.1.2“`Readable.from()`: 从可迭代对象创建可读流”

+   §9.3.2“通过`for-await-of`从可读流中读取块”

    +   §9.3.2.1“在字符串中收集可读流的内容”

+   §9.3.3“通过模块'node:readlines'从可读流中读取行”

+   §9.4“通过异步生成器转换可读流”

    +   §9.4.1“在异步可迭代对象中从块转换为编号行”

可写流：

+   §9.5.2“写入可写流”

+   §9.5.2.2“通过`stream.pipeline()`将可读流传输到可写流”

### 11.5 网络流配方

从中创建一个 ReadableStream：

+   字符串：§10.3.1“实现基础源的第一个示例”

+   可迭代对象：§10.3.2.2“示例：从拉取源创建一个 ReadableStream”

从 ReadableStream 中读取：

+   §10.2.1“通过读取器消耗 ReadableStreams”

+   §10.2.2“通过异步迭代消耗 ReadableStreams”

    +   §10.2.2.2“示例：组装包含 ReadableStream 内容的字符串”

+   §10.2.3“将 ReadableStreams 传输到 WritableStreams”

转换 ReadableStreams：

+   §10.6“使用 TransformStreams”

+   §10.7.2“提示：异步生成器也非常适合转换流”

+   §10.7.1“示例：将任意块的流转换为行流”

使用 WritableStreams：

+   §10.4“写入可写流”

+   §10.5.2“示例：在字符串中收集写入到 WriteStream 的块”

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/11)
