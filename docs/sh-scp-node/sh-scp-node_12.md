# 九、原生 Node.js 流

> 链接：[`exploringjs.com/nodejs-shell-scripting/ch_nodejs-streams.html`](https://exploringjs.com/nodejs-shell-scripting/ch_nodejs-streams.html)

* * *

+   9.1 总结：异步迭代和异步生成器

+   9.2 流

    +   9.2.1 管道

    +   9.2.2 文本编码

    +   9.2.3 辅助函数：`readableToString()`

    +   9.2.4 一些初步说明

+   9.3 可读流

    +   9.3.1 创建可读流

    +   9.3.2 通过`for-await-of`从可读流中读取块

    +   9.3.3 通过模块`'node:readlines'`从可读流中读取行

+   9.4 通过异步生成器转换可读流(ch_nodejs-streams.html#transforming-Readable-via-async-generator)

    +   9.4.1 从异步可迭代对象中的块转换为编号行

+   9.5 可写流

    +   9.5.1 为文件创建可写流

    +   9.5.2 写入可写流

+   9.6 快速参考：与流相关的功能(ch_nodejs-streams.html#quick-reference-stream-related-functionality)

+   9.7 进一步阅读和本章的来源

* * *

本章是对 Node 的原生流的介绍。它们支持[异步迭代](https://exploringjs.com/impatient-js/ch_async-iteration.html)，这使它们更容易使用，这也是我们在本章中主要使用的。

请注意，跨平台的*web 流*在§10“在 Node.js 上使用 web 流”中有所涵盖。我们在本书中主要使用这些。因此，如果您愿意，可以跳过当前章节。

### 9.1 总结：异步迭代和异步生成器

[异步迭代](https://exploringjs.com/impatient-js/ch_async-iteration.html)是一种异步检索数据容器内容的协议（意味着当前的“任务”在检索项目之前可能会暂停）。

[异步生成器](https://exploringjs.com/impatient-js/ch_async-iteration.html#async-generators)有助于异步迭代。例如，这是一个异步生成器函数：

```js
/**
 * @returns an asynchronous iterable
 */
async function* asyncGenerator(asyncIterable) {
 for await (const item of asyncIterable) { // input
 if (···) {
 yield '> ' + item; // output
 }
 }
}
```

+   `for-await-of`循环遍历输入的`asyncIterable`。这个循环也适用于普通的异步函数。

+   `yield`将值提供给此生成器返回的异步可迭代对象。

在本章的其余部分，请仔细注意函数是异步函数还是异步生成器函数：

```js
/** @returns a Promise */
async function asyncFunction() { /*···*/ }

/** @returns an async iterable */
async function* asyncGeneratorFunction() { /*···*/ }
```

### 9.2 流

流是一种模式，其核心思想是“分而治之”大量数据：如果我们将其分割成较小的部分并一次处理一部分，我们就可以处理它。

Node.js 支持几种流，例如：

+   *可读流*是我们可以从中读取数据的流。换句话说，它们是数据的来源。一个例子是*可读文件流*，它允许我们读取文件的内容。

+   *可写流*是我们可以写入数据的流。换句话说，它们是数据的接收端。一个例子是*可写文件流*，它允许我们向文件写入数据。

+   *转换流*既可读又可写。作为可写流，它接收数据块，*转换*（更改或丢弃）它们，然后将它们作为可读流输出。

#### 9.2.1 管道

为了在多个步骤中处理流数据，我们可以*管道*（连接）流：

1.  输入通过可读流接收。

1.  每个处理步骤都是通过转换流执行的。

1.  对于最后的处理步骤，我们有两个选项：

    +   我们可以将最近的可读流中的数据写入可写流。也就是说，可写流是我们管道的最后一个元素。

    +   我们可以以其他方式处理最近的可读流中的数据。

部分（2）是可选的。

#### 9.2.2 文本编码

创建文本流时，最好始终指定编码：

+   Node.js 文档中有[支持的编码及其默认拼写的列表](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings) - 例如：

    +   'utf8'

    +   'utf16le'

    +   'base64'

+   也允许一些不同的拼写。您可以使用[`Buffer.isEncoding()`](https://nodejs.org/api/buffer.html#buffer_class_method_buffer_isencoding_encoding)来检查哪些是：

    ```js
    > buffer.Buffer.isEncoding('utf8')
    true
    > buffer.Buffer.isEncoding('utf-8')
    true
    > buffer.Buffer.isEncoding('UTF-8')
    true
    > buffer.Buffer.isEncoding('UTF:8')
    false
    ```

编码的默认值是`null`，等同于`'utf8'`。

#### 9.2.3 辅助函数：`readableToString()`

我们偶尔会使用以下辅助函数。您不需要理解它的工作原理，只需（大致）了解它的作用。

```js
import * as stream from 'stream';

/**
 * Reads all the text in a readable stream and returns it as a string,
 * via a Promise.
 * @param  {stream.Readable} readable
 */
function readableToString(readable) {
 return new Promise((resolve, reject) => {
 let data = '';
 readable.on('data', function (chunk) {
 data += chunk;
 });
 readable.on('end', function () {
 resolve(data);
 });
 readable.on('error', function (err) {
 reject(err);
 });
 });
}
```

此函数是通过基于事件的 API 实现的。稍后我们将看到一个更简单的方法 - 通过异步迭代。

#### 9.2.4 一些初步说明

+   在本章中，我们只会使用文本流。

+   在示例中，我们偶尔会遇到`await`被用于顶层。在这种情况下，我们假设我们在[模块内](https://github.com/tc39/proposal-top-level-await)或在异步函数的主体内。

+   每当有换行符时，我们都支持：

    +   Unix：`'\n'`（LF）

    +   Windows：`'\r\n'`（CR LF）当前平台的换行符可以通过模块`os`中的常量`EOL`访问。

### 9.3 可读流

#### 9.3.1 创建可读流

##### 9.3.1.1 从文件创建可读流

我们可以使用[`fs.createReadStream()`](https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options)来创建可读流：

```js
import * as fs from 'fs';

const readableStream = fs.createReadStream(
 'tmp/test.txt', {encoding: 'utf8'});

assert.equal(
 await readableToString(readableStream),
 'This is a test!\n');
```

##### 9.3.1.2 `Readable.from()`: 从可迭代对象创建可读流

静态方法[`Readable.from(iterable, options?)`](https://nodejs.org/api/stream.html#stream_stream_readable_from_iterable_options)创建一个可读流，其中包含`iterable`中包含的数据。`iterable`可以是同步可迭代对象或异步可迭代对象。参数`options`是可选的，可以用于指定文本编码等其他内容。

```js
import * as stream from 'stream';

function* gen() {
 yield 'One line\n';
 yield 'Another line\n';
}
const readableStream = stream.Readable.from(gen(), {encoding: 'utf8'});
assert.equal(
 await readableToString(readableStream),
 'One line\nAnother line\n');
```

###### 9.3.1.2.1 从字符串创建可读流

`Readable.from()`接受任何可迭代对象，因此也可以用于将字符串转换为流：

```js
import {Readable} from 'stream';

const str = 'Some text!';
const readable = Readable.from(str, {encoding: 'utf8'});
assert.equal(
 await readableToString(readable),
 'Some text!');
```

[目前](https://github.com/nodejs/node/blob/master/lib/internal/streams/from.js)，`Readable.from()`将字符串视为任何其他可迭代对象，因此会迭代其代码点。从性能上讲，这并不理想，但对于大多数用例来说应该是可以的。我期望`Readable.from()`经常与字符串一起使用，所以也许将来会有优化。

#### 9.3.2 通过`for-await-of`从可读流中读取块

每个可读流都是异步可迭代的，这意味着我们可以使用`for-await-of`循环来读取其内容：

```js
import * as fs from 'fs';

async function logChunks(readable) {
 for await (const chunk of readable) {
 console.log(chunk);
 }
}

const readable = fs.createReadStream(
 'tmp/test.txt', {encoding: 'utf8'});
logChunks(readable);

// Output:
// 'This is a test!\n'
```

##### 9.3.2.1 在字符串中收集可读流的内容

以下函数是本章开头所见函数的简化重新实现。

```js
import {Readable} from 'stream';

async function readableToString2(readable) {
 let result = '';
 for await (const chunk of readable) {
 result += chunk;
 }
 return result;
}

const readable = Readable.from('Good morning!', {encoding: 'utf8'});
assert.equal(await readableToString2(readable), 'Good morning!');
```

请注意，在这种情况下，我们必须使用异步函数，因为我们想要返回一个 Promise。

#### 9.3.3 通过模块`'node:readlines'`从可读流中读取行

内置模块`'node:readline'`让我们可以从可读流中读取行：

```js
import * as fs from 'node:fs';
import * as readline from 'node:readline/promises';

const filePath = process.argv[2]; // first command line argument

const rl = readline.createInterface({
 input: fs.createReadStream(filePath, {encoding: 'utf-8'}),
});
for await (const line of rl) {
 console.log('>', line);
}
rl.close();
```

### 9.4 通过异步生成器转换可读流

异步迭代提供了一个优雅的替代方案，用于在多个步骤中处理流式数据的转换流：

+   输入是一个可读流。

+   第一个转换是通过一个异步生成器执行的，该生成器遍历可读流并在适当时产生。

+   可选地，我们可以通过使用更多的异步生成器来进一步转换。

+   最后，我们有几种处理最后一个生成器返回的异步可迭代对象的选项：

    +   我们可以通过`Readable.from()`将其转换为可读流（稍后可以传输到可写流）。

    +   我们可以使用异步函数来处理它。

    +   等等。

总之，这些是这样的处理管道的组成部分：

可读的

→ 第一个异步生成器 [→ … → 最后一个异步生成器]

→ 可读或异步函数

#### 9.4.1 从块到异步可迭代对象中的编号行

在下一个示例中，我们将看到一个刚刚解释过的处理管道的示例。

```js
import {Readable} from 'stream';

/**
 * @param  chunkIterable An asynchronous or synchronous iterable
 * over “chunks” (arbitrary strings)
 * @returns An asynchronous iterable over “lines”
 * (strings with at most one newline that always appears at the end)
 */
async function* chunksToLines(chunkIterable) {
 let previous = '';
 for await (const chunk of chunkIterable) {
 let startSearch = previous.length;
 previous += chunk;
 while (true) {
 // Works for EOL === '\n' and EOL === '\r\n'
 const eolIndex = previous.indexOf('\n', startSearch);
 if (eolIndex < 0) break;
 // Line includes the EOL
 const line = previous.slice(0, eolIndex+1);
 yield line;
 previous = previous.slice(eolIndex+1);
 startSearch = 0;
 }
 }
 if (previous.length > 0) {
 yield previous;
 }
}

async function* numberLines(lineIterable) {
 let lineNumber = 1;
 for await (const line of lineIterable) {
 yield lineNumber + ' ' + line;
 lineNumber++;
 }
}

async function logLines(lineIterable) {
 for await (const line of lineIterable) {
 console.log(line);
 }
}

const chunks = Readable.from(
 'Text with\nmultiple\nlines.\n',
 {encoding: 'utf8'});
await logLines(numberLines(chunksToLines(chunks))); // (A)

// Output:
// '1 Text with\n'
// '2 multiple\n'
// '3 lines.\n'
```

处理管道在 A 行设置。步骤是：

+   `chunksToLines()`: 从具有块的异步可迭代对象转换为具有行的异步可迭代对象。

+   `numberLines()`: 从具有行的异步可迭代对象转换为具有编号行的异步可迭代对象。

+   `logLines()`: 记录异步可迭代对象中的项目。

观察：

+   `chunksToLines()`和`numberLines()`的输入和输出都是异步可迭代对象。这就是为什么它们是异步生成器（由`async`和`*`指示）。

+   `logLines()`的输入是异步可迭代对象。这就是为什么它是一个异步函数（由`async`指示）。

### 9.5 可写流

#### 9.5.1 创建文件的可写流

我们可以使用[`fs.createWriteStream()`](https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options)来创建可写流：

```js
const writableStream = fs.createWriteStream(
 'tmp/log.txt', {encoding: 'utf8'});
```

#### 9.5.2 向可写流写入数据

在本节中，我们将探讨向可写流写入数据的方法：

1.  通过其方法`.write()`直接向可写流写入数据。

1.  使用模块`stream`中的函数`pipeline()`将可读流传输到可写流。

为了演示这些方法，我们使用它们来实现相同的函数`writeIterableToFile()`。

可读流的`.pipe()`方法也支持管道传输，但它有一个缺点，最好避免使用它。

##### 9.5.2.1 `writable.write(chunk)`

在向流中写入数据时，有两种基于回调的机制可以帮助我们：

+   事件`'drain'`表示背压已经解除。

+   函数`finished()`在流：

    +   不再可读或可写

    +   已经遇到错误或过早关闭事件

在下一个示例中，我们将这些机制转换为 Promise，以便我们可以通过异步函数使用它们：

```js
import * as util from 'util';
import * as stream from 'stream';
import * as fs from 'fs';
import {once} from 'events';

const finished = util.promisify(stream.finished); // (A)

async function writeIterableToFile(iterable, filePath) {
 const writable = fs.createWriteStream(filePath, {encoding: 'utf8'});
 for await (const chunk of iterable) {
 if (!writable.write(chunk)) { // (B)
 // Handle backpressure
 await once(writable, 'drain');
 }
 }
 writable.end(); // (C)
 // Wait until done. Throws if there are errors.
 await finished(writable);
}

await writeIterableToFile(
 ['One', ' line of text.\n'], 'tmp/log.txt');
assert.equal(
 fs.readFileSync('tmp/log.txt', {encoding: 'utf8'}),
 'One line of text.\n');
```

`stream.finished()`的默认版本是基于回调的，但可以通过`util.promisify()`（A 行）转换为基于 Promise 的版本。

我们使用了以下两种模式：

+   在处理背压的情况下向可写流写入数据（B 行）：

    ```js
    if (!writable.write(chunk)) {
     await once(writable, 'drain');
    }
    ```

+   关闭可写流并等待写入完成（C 行）：

    ```js
    writable.end();
    await finished(writable);
    ```

##### 9.5.2.2 通过`stream.pipeline()`将可读流传输到可写流

在 A 行，我们使用`stream.pipeline()`的 Promise 版本将可读流`readable`传输到可写流`writable`：

```js
import * as stream from 'stream';
import * as fs from 'fs';
const pipeline = util.promisify(stream.pipeline);

async function writeIterableToFile(iterable, filePath) {
 const readable = stream.Readable.from(
 iterable, {encoding: 'utf8'});
 const writable = fs.createWriteStream(filePath);
 await pipeline(readable, writable); // (A)
}
await writeIterableToFile(
 ['One', ' line of text.\n'], 'tmp/log.txt');
// ···
```

##### 9.5.2.3 不推荐：`readable.pipe(destination)`

可读的`.pipe()`方法也支持管道传输，但有[一个警告](https://nodejs.org/api/stream.html#stream_readable_pipe_destination)：如果可读流发出错误，则可写流不会自动关闭。`pipeline()`没有这个警告。

### 9.6 快速参考：与流相关的功能

模块`os`：

+   `const EOL: string`（自 0.7.8 起）(https://nodejs.org/api/os.html#os_os_eol)

    包含当前平台使用的行尾字符序列。

模块`buffer`：

+   `Buffer.isEncoding(encoding: string): boolean`（自 0.9.1 起）(https://nodejs.org/api/buffer.html#buffer_class_method_buffer_isencoding_encoding)

    如果`encoding`正确命名了受支持的 Node.js 文本编码之一，则返回`true`。[支持的编码](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings)包括：

    +   `'utf8'`

    +   `'utf16le'`

    +   `'ascii'`

    +   `'latin1`

    +   `'base64'`

    +   `'hex'`（每个字节表示为两个十六进制字符）

模块`stream`：

+   `Readable.prototype[Symbol.asyncIterator](): AsyncIterableIterator<any>`（自 10.0.0 起）(https://nodejs.org/api/stream.html#stream_readable_symbol_asynciterator)

    可读流是异步可迭代的。例如，您可以在异步函数或异步生成器中使用`for-await-of`循环来迭代它们。

+   `finished(stream: ReadableStream | WritableStream | ReadWriteStream, callback: (err?: ErrnoException | null) => void): () => Promise<void>` [(自 10.0.0 起)](https://nodejs.org/api/stream.html#stream_stream_finished_stream_options_callback)

    当读取/写入完成或出现错误时，返回的 Promise 将被解决。

    此 promisified 版本的创建方式如下：

    ```js
    const finished = util.promisify(stream.finished);
    ```

+   `pipeline(...streams: Array<ReadableStream|ReadWriteStream|WritableStream>): Promise<void>` [(自 10.0.0 起)](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback)

    流之间的管道。当管道完成或出现错误时，返回的 Promise 将被解决。

    此 promisified 版本的创建方式如下：

    ```js
    const pipeline = util.promisify(stream.pipeline);
    ```

+   `Readable.from(iterable: Iterable<any> | AsyncIterable<any>, options?: ReadableOptions): Readable` [(自 12.3.0 起)](https://nodejs.org/api/stream.html#stream_stream_readable_from_iterable_options)

    将可迭代对象转换为可读流。

    ```js
    interface ReadableOptions {
     highWaterMark?: number;
     encoding?: string;
     objectMode?: boolean;
     read?(this: Readable, size: number): void;
     destroy?(this: Readable, error: Error | null,
     callback: (error: Error | null) => void): void;
     autoDestroy?: boolean;
    }
    ```

    这些选项与`Readable`构造函数的选项相同，并在[此处](https://nodejs.org/api/stream.html#stream_new_stream_readable_options)有文档记录。

模块`fs`：

+   `createReadStream(path: string | Buffer | URL, options?: string | {encoding?: string; start?: number}): ReadStream` [(自 2.3.0 起)](https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options)

    创建可读流。还有更多选项可用。

+   `createWriteStream(path: PathLike, options?: string | {encoding?: string; flags?: string; mode?: number; start?: number}): WriteStream` [(自 2.3.0 起)](https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options)

    使用`.flags`选项，您可以指定是要写入还是追加，以及文件存在或不存在时会发生什么。还有更多选项可用。

本节中的静态类型信息基于[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/node)。

### 9.7 进一步阅读和本章的来源

+   [Node.js 文档中的“流与异步生成器和异步迭代器兼容性”章节](https://nodejs.org/api/stream.html#stream_streams_compatibility_with_async_generators_and_async_iterators)

+   [“JavaScript for impatient programmers”中的“异步函数”章节](https://exploringjs.com/impatient-js/ch_async-functions.html)

+   [“JavaScript for impatient programmers”中的“异步迭代”章节](https://exploringjs.com/impatient-js/ch_async-iteration.html)

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/9)
