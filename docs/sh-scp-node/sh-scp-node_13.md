# 十、在 Node.js 上使用 web 流

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_web-streams.html`](https://exploringjs.com/nodejs-shell-scripting/ch_web-streams.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)



* * *

+   10.1 什么是 web 流？

    +   10.1.1 流的种类

    +   10.1.2 管道链

    +   10.1.3 背压

    +   10.1.4 Node.js 中对 web 流的支持

+   10.2 从 ReadableStreams 读取

    +   10.2.1 通过 Readers 消费 ReadableStreams

    +   10.2.2 通过异步迭代消费 ReadableStreams

    +   10.2.3 将 ReadableStreams 管道到 WritableStreams

+   10.3 通过包装将数据源转换为 ReadableStreams

    +   10.3.1 实现底层源的第一个示例

    +   10.3.2 使用 ReadableStream 包装推送源或拉取源

+   10.4 写入 WritableStreams

    +   10.4.1 通过 Writers 写入 WritableStreams

    +   10.4.2 管道到 WritableStreams

+   10.5 通过包装将数据汇转换为 WritableStreams

    +   10.5.1 示例：跟踪 ReadableStream

    +   10.5.2 示例：收集写入字符串的 WriteStream 块

+   10.6 使用 TransformStreams

    +   10.6.1 标准 TransformStreams

+   10.7 实现自定义 TransformStreams

    +   10.7.1 示例：将任意块的流转换为行流

    +   10.7.2 提示：异步生成器也非常适合转换流

+   10.8 更深入地了解背压

    +   10.8.1 信号背压

    +   10.8.2 对背压的反应

+   10.9 字节流

    +   10.9.1 可读字节流

    +   10.9.2 示例：填充随机数据的无限可读字节流

    +   10.9.3 示例：压缩可读字节流

    +   10.9.4 示例：通过 `fetch()` 读取网页

+   10.10 Node.js 特定的辅助函数

+   10.11 进一步阅读

* * *

[*Web 流*](https://streams.spec.whatwg.org/) 是一种标准的 *流*，现在在所有主要的 web 平台上都得到支持：web 浏览器、Node.js 和 Deno。（流是一种从各种来源顺序读取和写入数据的抽象，例如文件、托管在服务器上的数据等。）

例如，[全局函数 `fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch)（用于下载在线资源）异步返回一个具有 web 流属性 `.body` 的 Response。

本章涵盖了 Node.js 上的 web 流，但我们所学的大部分内容都适用于支持它们的所有 web 平台。

### 10.1 什么是网络流？

让我们首先概述一下网络流的一些基本知识。之后，我们将快速转移到示例。

流是一种用于访问数据的数据结构，例如：

+   文件

+   托管在 Web 服务器上的数据

+   等等。

它们的两个好处是：

+   我们可以处理大量数据，因为流允许我们将它们分割成较小的片段（所谓的*chunks*），我们可以一次处理一个。

+   我们可以在处理不同数据时使用相同的数据结构，流。这样可以更容易地重用代码。

[*Web streams*](https://streams.spec.whatwg.org/#intro)（“web”通常被省略）是一个相对较新的标准，起源于 Web 浏览器，但现在也受到 Node.js 和 Deno 的支持（如此[MDN 兼容性表](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API#browser_compatibility)所示）。

在网络流中，chunks 通常是：

+   文本流：字符串

+   二进制流：Uint8Arrays（[一种 TypedArray](https://exploringjs.com/impatient-js/ch_typed-arrays.html)）

#### 10.1.1 流的种类

有三种主要类型的网络流：

+   一个 ReadableStream 用于从*source*读取数据。执行此操作的代码称为*consumer*。

+   一个 WritableStream 用于向*sink*写入数据。执行此操作的代码称为*producer*。

+   TransformStream 由两个流组成：

    +   它从其*writable side*接收输入，即 WritableStream。

    +   它将输出发送到其*readable side*，即 ReadableStream。

    这个想法是通过“管道传输”TransformStream 来转换数据。也就是说，我们将数据写入可写端，并从可读端读取转换后的数据。以下 TransformStreams 内置在大多数 JavaScript 平台中（稍后会详细介绍）：

    +   因为 JavaScript 字符串是 UTF-16 编码的，所以在 JavaScript 中，UTF-8 编码的数据被视为二进制数据。`TextDecoderStream`将这样的数据转换为字符串。

    +   `TextEncoderStream`将 JavaScript 字符串转换为 UTF-8 数据。

    +   `CompressionStream`将二进制数据压缩为 GZIP 和其他压缩格式。

    +   `DecompressionStream`从 GZIP 和其他压缩格式中解压缩二进制数据。

ReadableStreams，WritableStreams 和 TransformStreams 可用于传输文本或二进制数据。在本章中，我们将主要进行前者。 *字节流*用于二进制数据，在最后简要提到。

#### 10.1.2 管道链路

*Piping*是一种操作，它让我们将一个 ReadableStream 连接到一个 WritableStream：只要 ReadableStream 产生数据，此操作就会读取该数据并将其写入 WritableStream。如果我们连接了两个流，我们就可以方便地将数据从一个位置传输到另一个位置（例如复制文件）。但是，我们也可以连接多于两个流，并获得可以以各种方式处理数据的*管道链路*。这是一个管道链路的例子：

+   它以一个 ReadableStream 开始。

+   接下来是一个或多个 TransformStreams。

+   链路以 WritableStream 结束。

通过将前者连接到后者的可写端，将一个 ReadableStream 连接到 TransformStream。类似地，通过将前者的可读端连接到后者的可写端，将一个 TransformStream 连接到另一个 TransformStream。并且通过将前者的可读端连接到后者的可写端，将一个 TransformStream 连接到一个 WritableStream。

#### 10.1.3 背压

管道链路中的一个问题是，成员可能会收到比它目前能处理的更多数据。 *背压*是解决这个问题的一种技术：它使数据的接收者能够告诉发送者应该暂时停止发送数据，以便接收者不会被压倒。

另一种看待背压的方式是作为一个信号，通过管道链路向后传播，从被压倒的成员到链路的开始。例如，考虑以下管道链路：

```js
ReadableStream -pipeTo-> TransformStream -pipeTo-> WriteableStream
```

这是背压通过这个链路传播的方式：

+   最初，WriteableStream 发出信号，表明它暂时无法处理更多数据。

+   管道停止从 TransformStream 中读取。

+   输入在 TransformStream 中积累（被缓冲）。

+   TransformStream 发出满的信号。

+   管道停止从 ReadableStream 中读取。

我们已经到达管道链的开头。因此，在 ReadableStream 中没有数据积累（也被缓冲），WritableStream 有时间恢复。一旦它恢复，它会发出信号表明它已准备好再次接收数据。该信号也会通过链返回，直到它到达 ReadableStream，数据处理恢复。

在这第一次对背压的探讨中，为了让事情更容易理解，省略了一些细节。这些将在以后进行讨论。

#### 10.1.4 Node.js 中对 web 流的支持

在 Node.js 中，Web 流可以从两个来源获得：

+   来自[模块`'node:stream/web'`](https://nodejs.org/api/webstreams.html)

+   通过全局变量（就像在 Web 浏览器中）

目前，只有一个 API 在 Node.js 中直接支持 web 流 – [Fetch API](https://nodejs.org/api/globals.html#fetch)：

```js
const response = await fetch('https://example.com');
const readableStream = response.body;
```

对于其他事情，我们需要使用模块`'node:stream'`中以下静态方法之一，将 Node.js 流转换为 Web 流，反之亦然：

+   Node.js 的 Readable 可以转换为 WritableStreams，反之亦然：

    +   `Readable.toWeb(nodeReadable)`

    +   `Readable.fromWeb(webReadableStream, options?)`

+   Node.js 的 Writable 可以转换为 ReadableStreams，反之亦然：

    +   `Writable.toWeb(nodeWritable)`

    +   `Writable.fromWeb(webWritableStream, options?)`

+   Node.js 的 Duplex 可以转换为 TransformStreams，反之亦然：

    +   `Duplex.toWeb(nodeDuplex)`

    +   `Duplex.fromWeb(webTransformStream, options?)`

还有一个 API 部分支持 web 流：FileHandles 有方法[`.readableWebStream()`](https://nodejs.org/dist/latest-v18.x/docs/api/fs.html#filehandlereadablewebstream)。

### 10.2 从 ReadableStreams 中读取

ReadableStreams 让我们从各种来源读取数据块。它们具有以下类型（随意浏览此类型及其属性的解释；当我们在示例中遇到它们时，它们将再次被解释）：

```js
interface ReadableStream<TChunk> {
 getReader(): ReadableStreamDefaultReader<TChunk>;
 readonly locked: boolean;
 [Symbol.asyncIterator](): AsyncIterator<TChunk>;

 cancel(reason?: any): Promise<void>;

 pipeTo(
 destination: WritableStream<TChunk>,
 options?: StreamPipeOptions
 ): Promise<void>;
 pipeThrough<TChunk2>(
 transform: ReadableWritablePair<TChunk2, TChunk>,
 options?: StreamPipeOptions
 ): ReadableStream<TChunk2>;

 // Not used in this chapter:
 tee(): [ReadableStream<TChunk>, ReadableStream<TChunk>];
}

interface StreamPipeOptions {
 signal?: AbortSignal;
 preventClose?: boolean;
 preventAbort?: boolean;
 preventCancel?: boolean;
}
```

这些属性的解释：

+   `.getReader()`返回一个 Reader – 通过它我们可以从 ReadableStream 中读取。ReadableStreams 返回 Readers 类似于[可迭代对象](https://exploringjs.com/impatient-js/ch_sync-iteration.html)返回迭代器。

+   `.locked`: 一次只能有一个活动的 Reader 读取 ReadableStream。当一个 Reader 正在使用时，ReadableStream 被锁定，无法调用`.getReader()`。

+   `[Symbol.asyncIterator](https://exploringjs.com/impatient-js/ch_async-iteration.html)`: 这个方法使得 ReadableStreams 可以[异步迭代](https://exploringjs.com/impatient-js/ch_async-iteration.html)。目前只在一些平台上实现。

+   `.cancel(reason)`取消流，因为消费者对它不再感兴趣。`reason`被传递给 ReadableStream 的*底层源*的`.cancel()`方法（稍后会详细介绍）。返回的 Promise 在此操作完成时实现。

+   `.pipeTo()`将其 ReadableStream 的内容传送到 WritableStream。返回的 Promise 在此操作完成时实现。`.pipeTo()`确保背压、关闭、错误等都正确地通过管道链传播。我们可以通过它的第二个参数指定选项：

    +   `.signal`让我们向这个方法传递一个 AbortSignal，这使我们能够通过 AbortController 中止管道传输。

    +   `.preventClose`: 如果为`true`，它会阻止在 ReadableStream 关闭时关闭 WritableStream。当我们想要将多个 ReadableStream 管道到同一个 WritableStream 时，这是有用的。

    +   其余选项超出了本章的范围。它们在[web 流规范](https://streams.spec.whatwg.org/#rs-prototype)中有文档记录。

+   `.pipeThrough()`将其 ReadableStream 连接到一个 ReadableWritablePair（大致是一个 TransformStream，稍后会详细介绍）。它返回生成的 ReadableStream（即 ReadableWritablePair 的可读端）。

以下小节涵盖了三种消费 ReadableStreams 的方式：

+   通过 Readers 进行读取

+   通过异步迭代进行读取

+   将 ReadableStreams 连接到 WritableStreams

#### 10.2.1 通过 Reader 消费 ReadableStreams

我们可以使用*Readers*从 ReadableStreams 中读取数据。它们具有以下类型（随意浏览此类型及其属性的解释；当我们在示例中遇到它们时，它们将再次被解释）：

```js
interface ReadableStreamGenericReader {
 readonly closed: Promise<undefined>;
 cancel(reason?: any): Promise<void>;
}
interface ReadableStreamDefaultReader<TChunk>
 extends ReadableStreamGenericReader
{
 releaseLock(): void;
 read(): Promise<ReadableStreamReadResult<TChunk>>;
}

interface ReadableStreamReadResult<TChunk> {
 done: boolean;
 value: TChunk | undefined;
}
```

这些属性的解释：

+   `.closed`：此 Promise 在流关闭后被满足。如果流出现错误或者在流关闭之前 Reader 的锁被释放，它将被拒绝。

+   `.cancel()`：在活动的 Reader 中，此方法取消关联的 ReadableStream。

+   `.releaseLock()`停用 Reader 并解锁其流。

+   `.read()`返回一个 Promise，用于 ReadableStreamReadResult（一个包装的块），它有两个属性：

    +   `.done`是一个布尔值，只要可以读取块，就为`false`，在最后一个块之后为`true`。

    +   `.value`是块（或在最后一个块之后是`undefined`）。

如果您了解迭代的工作原理，ReadableStreamReadResult 可能会很熟悉：ReadableStreams 类似于可迭代对象，Readers 类似于迭代器，而 ReadableStreamReadResults 类似于迭代器方法`.next()`返回的对象。

以下代码演示了使用 Readers 的协议：

```js
const reader = readableStream.getReader(); // (A)
assert.equal(readableStream.locked, true); // (B)
try {
 while (true) {
 const {done, value: chunk} = await reader.read(); // (C)
 if (done) break;
 // Use `chunk`
 }
} finally {
 reader.releaseLock(); // (D)
}
```

**获取 Reader。**我们不能直接从`readableStream`中读取，我们首先需要获取一个*Reader*（行 A）。每个 ReadableStream 最多可以有一个 Reader。获取 Reader 后，`readableStream`被锁定（行 B）。在我们可以再次调用`.getReader()`之前，我们必须调用`.releaseLock()`（行 D）。

**读取块。**`.read()`返回一个带有属性`.done`和`.value`的对象的 Promise（行 C）。在读取最后一个块之后，`.done`为`true`。这种方法类似于 JavaScript 中[异步迭代](https://exploringjs.com/impatient-js/ch_async-iteration.html)的工作方式。

##### 10.2.1.1 示例：通过 ReadableStream 读取文件

在下面的示例中，我们从文本文件`data.txt`中读取块（字符串）：

```js
import * as fs from 'node:fs';
import {Readable} from 'node:stream';

const nodeReadable = fs.createReadStream(
 'data.txt', {encoding: 'utf-8'});
const webReadableStream = Readable.toWeb(nodeReadable); // (A)

const reader = webReadableStream.getReader();
try {
 while (true) {
 const {done, value} = await reader.read();
 if (done) break;
 console.log(value);
 }
} finally {
 reader.releaseLock();
}
// Output:
// 'Content of text file\n'
```

我们将 Node.js Readable 转换为 web ReadableStream（行 A）。然后我们使用先前解释的协议来读取块。

##### 10.2.1.2 示例：使用 ReadableStream 内容组装字符串

在下一个示例中，我们将所有 ReadableStream 的块连接成一个字符串并返回它：

```js
/**
 * Returns a string with the contents of `readableStream`.
 */
async function readableStreamToString(readableStream) {
 const reader = readableStream.getReader();
 try {
 let result = '';
 while (true) {
 const {done, value} = await reader.read();
 if (done) {
 return result; // (A)
 }
 result += value;
 }
 } finally {
 reader.releaseLock(); // (B)
 }
}
```

方便的是，`finally`子句总是被执行 - 无论我们如何离开`try`子句。也就是说，如果我们返回一个结果（行 A），锁将被正确释放（行 B）。

#### 10.2.2 通过异步迭代消费 ReadableStreams

ReadableStreams 也可以通过[异步迭代](https://exploringjs.com/impatient-js/ch_async-iteration.html)进行消费：

```js
const iterator = readableStream[Symbol.asyncIterator]();
let exhaustive = false;
try {
 while (true) {
 let chunk;
 ({done: exhaustive, value: chunk} = await iterator.next());
 if (exhaustive) break;
 console.log(chunk);
 }
} finally {
 // If the loop was terminated before we could iterate exhaustively
 // (via an exception or `return`), we must call `iterator.return()`.
 // Check if that was the case.
 if (!exhaustive) {
 iterator.return();
 }
}
```

值得庆幸的是，`for-await-of`循环为我们处理了异步迭代的所有细节：

```js
for await (const chunk of readableStream) {
 console.log(chunk);
}
```

##### 10.2.2.1 示例：使用异步迭代读取流

让我们重新尝试从文件中读取文本。这次，我们使用异步迭代而不是 Reader：

```js
import * as fs from 'node:fs';
import {Readable} from 'node:stream';

const nodeReadable = fs.createReadStream(
 'text-file.txt', {encoding: 'utf-8'});
const webReadableStream = Readable.toWeb(nodeReadable);
for await (const chunk of webReadableStream) {
 console.log(chunk);
}
// Output:
// 'Content of text file'
```

##### 10.2.2.2 示例：使用 ReadableStream 内容组装字符串

我们以前使用 Reader 来组装一个包含 ReadableStream 内容的字符串。有了异步迭代，代码变得更简单了：

```js
/**
 * Returns a string with the contents of `readableStream`.
 */
async function readableStreamToString2(readableStream) {
 let result = '';
 for await (const chunk of readableStream) {
 result += chunk;
 }
 return result;
}
```

##### 10.2.2.3 注意事项：浏览器不支持对 ReadableStreams 进行异步迭代

目前，Node.js 和 Deno 支持对 ReadableStreams 进行异步迭代，但 Web 浏览器不支持：有[一个 GitHub 问题](https://github.com/whatwg/streams/issues/778)链接到错误报告。

鉴于尚不完全清楚浏览器将如何支持异步迭代，包装比填充更安全。以下代码基于[Chromium bug 报告中的建议](https://bugs.chromium.org/p/chromium/issues/detail?id=929585#c10)：

```js
async function* getAsyncIterableFor(readableStream) {
 const reader = readableStream.getReader();
 try {
 while (true) {
 const {done, value} = await reader.read();
 if (done) return;
 yield value;
 }
 } finally {
 reader.releaseLock();
 }
}
```

#### 10.2.3 将可读流导入可写流

可读流有两种管道方法：

+   `readableStream.pipeTo(writeableStream)`同步返回一个 Promise `p`。它异步读取`readableStream`的所有块，并将它们写入`writableStream`。完成后，它会实现`p`。

    当我们探索可写流时，我们将看到`.pipeTo()`的示例，因为它提供了一种方便的方式将数据传输到其中。

+   `readableStream.pipeThrough(transformStream)`将`readableStream`导入`transformStream.writable`并返回`transformStream.readable`（每个 TransformStream 都有这些属性，它们指向其可写侧和可读侧）。另一种看待这个操作的方式是，我们通过连接`transformStream`到`readableStream`创建一个新的可读流。

    当我们探索 TransformStreams 时，我们将看到`.pipeThrough()`的示例，因为这是它们主要使用的方法。

### 10.3 将数据源通过包装转换为可读流

如果我们想通过一个可读流读取外部源，我们可以将其包装在一个适配器对象中，并将该对象传递给`ReadableStream`构造函数。适配器对象被称为可读流的*底层源*（当我们更仔细地看 backpressure 时，将解释排队策略）：

```js
new ReadableStream(underlyingSource?, queuingStrategy?)
```

这是底层源的类型（随意浏览此类型及其属性的解释；当我们在示例中遇到它们时，它们将再次解释）：

```js
interface UnderlyingSource<TChunk> {
 start?(
 controller: ReadableStreamController<TChunk>
 ): void | Promise<void>;
 pull?(
 controller: ReadableStreamController<TChunk>
 ): void | Promise<void>;
 cancel?(reason?: any): void | Promise<void>;

 // Only used in byte streams and ignored in this section:
 type: 'bytes' | undefined;
 autoAllocateChunkSize: bigint;
}
```

这是当可读流调用这些方法时：

+   在调用`ReadableStream`的构造函数后立即调用`.start(controller)`。

+   每当可读流的内部队列中有空间时，都会调用`.pull(controller)`。直到队列再次满了为止，它会被重复调用。此方法只会在`.start()`完成后调用。如果`.pull()`没有入队任何内容，它将不会再次被调用。

+   如果可读流的消费者通过`readableStream.cancel()`或`reader.cancel()`取消它，将调用`.cancel(reason)`。`reason`是传递给这些方法的值。

这些方法中的每一个都可以返回一个 Promise，并且在 Promise 解决之前不会采取进一步的步骤。如果我们想要做一些异步操作，这是有用的。

`.start()`和`.pull()`的参数`controller`让它们访问流。它具有以下类型：

```js
type ReadableStreamController<TChunk> =
 | ReadableStreamDefaultController<TChunk>
 | ReadableByteStreamController<TChunk> // ignored here
;

interface ReadableStreamDefaultController<TChunk> {
 enqueue(chunk?: TChunk): void;
 readonly desiredSize: number | null;
 close(): void;
 error(err?: any): void;
}
```

现在，块是字符串。我们稍后将介绍字节流，其中 Uint8Arrays 很常见。这些方法的作用是：

+   `.enqueue(chunk)`将`chunk`添加到可读流的内部队列。

+   `.desiredSize`指示`.enqueue()`写入的队列中有多少空间。如果队列已满，则为零，如果超过了最大大小，则为负。因此，如果期望大小为零或负，则我们必须停止入队。

    +   如果流关闭，其期望大小为零。

    +   如果流处于错误模式，其期望大小为`null`。

+   `.close()`关闭可读流。消费者仍然可以清空队列，但之后，流将结束。底层源调用此方法很重要-否则，读取其流将永远不会结束。

+   `.error(err)`将流置于错误模式：以后与它的所有交互都将以错误值`err`失败。

#### 10.3.1 实现底层源的第一个示例

在我们实现底层源的第一个示例中，我们只提供了`.start()`方法。我们将在下一小节中看到`.pull()`的用例。

```js
const readableStream = new ReadableStream({
 start(controller) {
 controller.enqueue('First line\n'); // (A)
 controller.enqueue('Second line\n'); // (B)
 controller.close(); // (C)
 },
});
for await (const chunk of readableStream) {
 console.log(chunk);
}

// Output:
// 'First line\n'
// 'Second line\n'
```

我们使用控制器创建一个具有两个块（行 A 和行 B）的流。关闭流很重要（行 C）。否则，`for-await-of`循环永远不会结束！

请注意，这种入队的方式并不完全安全：存在超出内部队列容量的风险。我们很快将看到如何避免这种风险。

#### 10.3.2 使用 ReadableStream 包装推送源或拉取源

一个常见的场景是将推送源或拉取源转换为 ReadableStream。源是推送还是拉取决定了我们将如何与 UnderlyingSource 连接到 ReadableStream：

+   推送源：这样的源在有新数据时通知我们。我们使用`.start()`来设置监听器和支持数据结构。如果我们收到太多数据，期望的大小不再是正数，我们必须告诉我们的源暂停。如果以后调用了`.pull()`，我们可以取消暂停。对外部源在期望的大小变为非正数时暂停的反应称为*应用背压*。

+   拉取源：我们向这样的源请求新数据-通常是异步的。因此，我们通常在`.start()`中不做太多事情，并在调用`.pull()`时检索数据。

接下来我们将看到两种来源的例子。

##### 10.3.2.1 示例：从具有背压支持的推送源创建一个 ReadableStream

在下面的示例中，我们将一个 ReadableStream 包装在一个套接字周围-它向我们推送数据（它调用我们）。[这个例子](https://streams.spec.whatwg.org/#example-rs-push-backpressure)来自 web 流规范：

```js
function makeReadableBackpressureSocketStream(host, port) {
 const socket = createBackpressureSocket(host, port);

 return new ReadableStream({
 start(controller) {
 socket.ondata = event => {
 controller.enqueue(event.data);

 if (controller.desiredSize <= 0) {
 // The internal queue is full, so propagate
 // the backpressure signal to the underlying source.
 socket.readStop();
 }
 };

 socket.onend = () => controller.close();
 socket.onerror = () => controller.error(
 new Error('The socket errored!'));
 },

 pull() {
 // This is called if the internal queue has been emptied, but the
 // stream’s consumer still wants more data. In that case, restart
 // the flow of data if we have previously paused it.
 socket.readStart();
 },

 cancel() {
 socket.close();
 },
 });
}
```

##### 10.3.2.2 示例：从拉取源创建一个 ReadableStream

工具函数`iterableToReadableStream()`接受一个块的可迭代对象，并将其转换为一个 ReadableStream：

```js
/**
 * @param  iterable an iterable (asynchronous or synchronous)
 */
 function iterableToReadableStream(iterable) {
 return new ReadableStream({
 start() {
 if (typeof iterable[Symbol.asyncIterator] === 'function') {
 this.iterator = iterable[Symbol.asyncIterator]();
 } else if (typeof iterable[Symbol.iterator] === 'function') {
 this.iterator = iterable[Symbol.iterator]();
 } else {
 throw new Error('Not an iterable: ' + iterable);
 }
 },

 async pull(controller) {
 if (this.iterator === null) return;
 // Sync iterators return non-Promise values,
 // but `await` doesn’t mind and simply passes them on
 const {value, done} = await this.iterator.next();
 if (done) {
 this.iterator = null;
 controller.close();
 return;
 }
 controller.enqueue(value);
 },

 cancel() {
 this.iterator = null;
 controller.close();
 },
 });
}
```

让我们使用一个异步生成器函数来创建一个异步可迭代对象，并将该可迭代对象转换为一个 ReadableStream：

```js
async function* genAsyncIterable() {
 yield 'how';
 yield 'are';
 yield 'you';
}
const readableStream = iterableToReadableStream(genAsyncIterable());
for await (const chunk of readableStream) {
 console.log(chunk);
}

// Output:
// 'how'
// 'are'
// 'you'
```

`iterableToReadableStream()`也适用于同步可迭代对象：

```js
const syncIterable = ['hello', 'everyone'];
const readableStream = iterableToReadableStream(syncIterable);
for await (const chunk of readableStream) {
 console.log(chunk);
}

// Output:
// 'hello'
// 'everyone'
```

可能会有一个静态的辅助方法`ReadableStream.from()`，提供这个功能（[请参阅其拉取请求以获取更多信息](https://github.com/whatwg/streams/pull/1083)）。

### 10.4 向 WritableStreams 写入

WritableStreams 让我们向各种接收器写入数据块。它们具有以下类型（随意浏览此类型和其属性的解释；当我们在示例中遇到它们时，它们将再次解释）：

```js
interface WritableStream<TChunk> {
 getWriter(): WritableStreamDefaultWriter<TChunk>;
 readonly locked: boolean;

 close(): Promise<void>;
 abort(reason?: any): Promise<void>;
}
```

这些属性的解释：

+   `.getWriter()`返回一个 Writer-通过它我们可以向 WritableStream 写入数据的对象。

+   `.locked`：WritableStream 一次只能有一个活动的 Writer。当一个 Writer 正在使用时，WritableStream 被锁定，无法调用`.getWriter()`。

+   `.close()`关闭流：

    +   *底层接收器*（稍后会详细介绍）在关闭之前仍将接收所有排队的块。

    +   从现在开始，所有的写入尝试都将无声地失败（没有错误）。

    +   该方法返回一个 Promise，如果接收器成功写入所有排队的块并关闭，将实现该 Promise。如果在这些步骤中发生任何错误，它将被拒绝。

+   `.abort()`中止流：

    +   它将流置于错误模式。

    +   返回的 Promise 在接收器成功关闭时实现，如果发生错误则拒绝。

以下小节涵盖了向 WritableStreams 发送数据的两种方法：

+   通过 Writers 向 WritableStreams 写入

+   将数据传输到 WritableStreams

#### 10.4.1 通过 Writers 向 WritableStreams 写入

我们可以使用*Writers*向 WritableStreams 写入。它们具有以下类型（随意浏览此类型和其属性的解释；当我们在示例中遇到它们时，它们将再次解释）：

```js
interface WritableStreamDefaultWriter<TChunk> {
 readonly desiredSize: number | null;
 readonly ready: Promise<undefined>;
 write(chunk?: TChunk): Promise<void>;
 releaseLock(): void;

 close(): Promise<void>;
 readonly closed: Promise<undefined>;
 abort(reason?: any): Promise<void>;
}
```

这些属性的解释：

+   `.desiredSize`指示 WriteStream 队列中有多少空间。如果队列已满，则为零，如果超过最大大小，则为负数。因此，如果期望的大小为零或负数，我们必须停止写入。

    +   如果流关闭，它的期望大小为零。

    +   如果流处于错误模式，它的期望大小为`null`。

+   `.ready`返回一个 Promise，在期望的大小从非正数变为正数时实现。这意味着没有背压活动，可以写入数据。如果期望的大小后来再次变为非正数，则会创建并返回一个新的待处理 Promise。

+   `.write()`将一个块写入流。它返回一个 Promise，在写入成功后实现，如果有错误则拒绝。

+   `.releaseLock()`释放 Writer 对其流的锁定。

+   `.close()`具有与关闭 Writer 流相同的效果。

+   `.closed`返回一个 Promise，在流关闭时被实现。

+   `.abort()`具有与中止 Writer 流相同的效果。

以下代码显示了使用 Writers 的协议：

```js
const writer = writableStream.getWriter(); // (A)
assert.equal(writableStream.locked, true); // (B)
try {
 // Writing the chunks (explained later)
} finally {
 writer.releaseLock(); // (C)
}
```

我们不能直接向`writableStream`写入，我们首先需要获取一个*Writer*（A 行）。每个 WritableStream 最多只能有一个 Writer。在获取了 Writer 之后，`writableStream`被锁定（B 行）。在我们可以再次调用`.getWriter()`之前，我们必须调用`.releaseLock()`（C 行）。

有三种写入块的方法。

##### 10.4.1.1 写入方法 1：等待`.write()`（处理背压效率低下）

第一种写入方法是等待每个`.write()`的结果：

```js
await writer.write('Chunk 1');
await writer.write('Chunk 2');
await writer.close();
```

由`.write()`返回的 Promise 在我们传递给它的块成功写入时实现。“成功写入”具体意味着什么取决于 WritableStream 的实现方式 - 例如，对于文件流，该块可能已发送到操作系统，但仍然驻留在缓存中，因此实际上尚未写入磁盘。

由`.close()`返回的 Promise 在流关闭时实现。

这种写入方法的一个缺点是等待写入成功意味着队列没有被使用。因此，数据吞吐量可能会较低。

##### 10.4.1.2 写入方法 2：忽略`.write()`拒绝（忽略背压）

在第二种写入方法中，我们忽略了`.write()`返回的 Promise，只等待`.close()`返回的 Promise：

```js
writer.write('Chunk 1').catch(() => {}); // (A)
writer.write('Chunk 2').catch(() => {}); // (B)
await writer.close(); // reports errors
```

`.write()`的同步调用将块添加到 WritableStream 的内部队列中。通过不等待返回的 Promises，我们不必等待每个块被写入。但是，等待`.close()`确保队列为空，并且所有写入都成功后我们才继续。

在 A 行和 B 行调用`.catch()`是必要的，以避免在写入过程中出现问题时出现有关未处理的 Promise 拒绝的警告。这样的警告通常会记录在控制台上。我们可以忽略`.write()`报告的错误，因为`.close()`也会向我们报告这些错误。

通过使用一个忽略 Promise 拒绝的辅助函数，可以改进先前的代码：

```js
ignoreRejections(
 writer.write('Chunk 1'),
 writer.write('Chunk 2'),
);
await writer.close(); // reports errors

function ignoreRejections(...promises) {
 for (const promise of promises) {
 promise.catch(() => {});
 }
}
```

这种方法的一个缺点是忽略了背压：我们只是假设队列足够大，可以容纳我们写入的所有内容。

##### 10.4.1.3 写入方法 3：等待`.ready`（高效处理背压）

在这种写入方法中，我们通过等待 Writer getter`.ready`来有效地处理背压：

```js
await writer.ready; // reports errors
// How much room do we have?
console.log(writer.desiredSize);
writer.write('Chunk 1').catch(() => {});

await writer.ready; // reports errors
// How much room do we have?
console.log(writer.desiredSize);
writer.write('Chunk 2').catch(() => {});

await writer.close(); // reports errors
```

`.ready`中的 Promise 在流从有背压到无背压的转换时实现。

##### 10.4.1.4 示例：通过 Writer 写入文件

在这个例子中，我们通过 WritableStream 创建一个文本文件`data.txt`：

```js
import * as fs from 'node:fs';
import {Writable} from 'node:stream';

const nodeWritable = fs.createWriteStream(
 'new-file.txt', {encoding: 'utf-8'}); // (A)
const webWritableStream = Writable.toWeb(nodeWritable); // (B)

const writer = webWritableStream.getWriter();
try {
 await writer.write('First line\n');
 await writer.write('Second line\n');
 await writer.close();
} finally {
 writer.releaseLock()
}
```

在 A 行，我们为文件`data.txt`创建了一个 Node.js 流。在 B 行，我们将这个流转换为 web 流。然后我们使用 Writer 将字符串写入其中。

#### 10.4.2 向 WritableStreams 进行管道传输

除了使用 Writers，我们还可以通过将 ReadableStreams 传输到 WritableStreams 来向 WritableStreams 写入：

```js
await readableStream.pipeTo(writableStream);
```

由`.pipeTo()`返回的 Promise 在传输成功完成时实现。

##### 10.4.2.1 管道传输是异步进行的

管道传输是在当前任务完成或暂停后执行的。以下代码演示了这一点：

```js
const readableStream = new ReadableStream({ // (A)
 start(controller) {
 controller.enqueue('First line\n');
 controller.enqueue('Second line\n');
 controller.close();
 },
});
const writableStream = new WritableStream({ // (B)
 write(chunk) {
 console.log('WRITE: ' + JSON.stringify(chunk));
 },
 close() {
 console.log('CLOSE WritableStream');
 },
});

console.log('Before .pipeTo()');
const promise = readableStream.pipeTo(writableStream); // (C)
promise.then(() => console.log('Promise fulfilled'));
console.log('After .pipeTo()');

// Output:
// 'Before .pipeTo()'
// 'After .pipeTo()'
// 'WRITE: "First line\n"'
// 'WRITE: "Second line\n"'
// 'CLOSE WritableStream'
// 'Promise fulfilled'
```

在 A 行我们创建一个 ReadableStream。在 B 行我们创建一个 WritableStream。

我们可以看到`.pipeTo()`（行 C）立即返回。在一个新的任务中，块被读取和写入。然后`writableStream`被关闭，最后，`promise`被实现。

##### 10.4.2.2 示例：将数据管道到文件的可写流

在下面的示例中，我们为一个文件创建一个 WritableStream，并将一个 ReadableStream 管道传递给它：

```js
const webReadableStream = new ReadableStream({ // (A)
 async start(controller) {
 controller.enqueue('First line\n');
 controller.enqueue('Second line\n');
 controller.close();
 },
});

const nodeWritable = fs.createWriteStream( // (B)
 'data.txt', {encoding: 'utf-8'});
const webWritableStream = Writable.toWeb(nodeWritable); // (C)

await webReadableStream.pipeTo(webWritableStream); // (D)
```

在 A 行，我们创建了一个 ReadableStream。在 B 行，我们为文件`data.txt`创建了一个 Node.js 流。在 C 行，我们将这个流转换为 web 流。在 D 行，我们将我们的`webReadableStream`管道传递给文件的 WritableStream。

##### 10.4.2.3 示例：将两个 ReadableStreams 写入到一个 WritableStream

在下面的示例中，我们将两个 ReadableStreams 写入单个 WritableStream。

```js
function createReadableStream(prefix) {
 return new ReadableStream({
 async start(controller) {
 controller.enqueue(prefix + 'chunk 1');
 controller.enqueue(prefix + 'chunk 2');
 controller.close();
 },
 });
}

const writableStream = new WritableStream({
 write(chunk) {
 console.log('WRITE ' + JSON.stringify(chunk));
 },
 close() {
 console.log('CLOSE');
 },
 abort(err) {
 console.log('ABORT ' + err);
 },
});

await createReadableStream('Stream 1: ')
 .pipeTo(writableStream, {preventClose: true}); // (A)
await createReadableStream('Stream 2: ')
 .pipeTo(writableStream, {preventClose: true}); // (B)
await writableStream.close();

// Output
// 'WRITE "Stream 1: chunk 1"'
// 'WRITE "Stream 1: chunk 2"'
// 'WRITE "Stream 2: chunk 1"'
// 'WRITE "Stream 2: chunk 2"'
// 'CLOSE'
```

我们告诉`.pipeTo()`在 ReadableStream 关闭后不关闭 WritableStream（行 A 和行 B）。因此，在行 A 之后，WritableStream 保持打开状态，我们可以将另一个 ReadableStream 管道传递给它。

### 10.5 将数据接收端通过包装转换为可写流

如果我们想通过 WritableStream 写入到外部接收端，我们可以将其包装在一个适配器对象中，并将该对象传递给`WritableStream`的构造函数。适配器对象被称为 WritableStream 的*底层接收端*（当我们更仔细地看反压时，排队策略将在稍后解释）：

```js
new WritableStream(underlyingSink?, queuingStrategy?)
```

这是底层接收端的类型（随意浏览此类型和其属性的解释；当我们在示例中遇到它们时，它们将再次解释）：

```js
interface UnderlyingSink<TChunk> {
 start?(
 controller: WritableStreamDefaultController
 ): void | Promise<void>;
 write?(
 chunk: TChunk,
 controller: WritableStreamDefaultController
 ): void | Promise<void>;
 close?(): void | Promise<void>;;
 abort?(reason?: any): void | Promise<void>;
}
```

这些属性的解释：

+   `.start(controller)` 在我们调用`WritableStream`的构造函数后立即调用。如果我们做一些异步操作，我们可以返回一个 Promise。在这个方法中，我们可以准备写入。

+   `.write(chunk, controller)` 当一个新的块准备写入外部接收端时调用。我们可以通过返回一个 Promise 来施加反压，一旦反压消失就会实现。

+   `.close()` 在调用`writer.close()`后调用，并且所有排队的写入都成功。在这个方法中，我们可以在写入后进行清理。

+   如果调用了`writeStream.abort()`或`writer.abort()`，则会调用`.abort(reason)`。`reason`是传递给这些方法的值。

`.start()`和`.write()`的参数`controller`让它们错误 WritableStream。它具有以下类型：

```js
interface WritableStreamDefaultController {
 readonly signal: AbortSignal;
 error(err?: any): void;
}
```

+   `.signal` 是一个 AbortSignal，如果我们想在流被中止时中止写入或关闭操作，我们可以监听它。

+   `.error(err)` 错误 WritableStream：它被关闭，并且以后所有与它的交互都会失败，错误值为`err`。

#### 10.5.1 示例：跟踪一个可读流

在下一个示例中，我们将一个 ReadableStream 管道到一个 WritableStream，以便检查 ReadableStream 如何生成块：

```js
const readableStream = new ReadableStream({
 start(controller) {
 controller.enqueue('First chunk');
 controller.enqueue('Second chunk');
 controller.close();
 },
});
await readableStream.pipeTo(
 new WritableStream({
 write(chunk) {
 console.log('WRITE ' + JSON.stringify(chunk));
 },
 close() {
 console.log('CLOSE');
 },
 abort(err) {
 console.log('ABORT ' + err);
 },
 })
);
// Output:
// 'WRITE "First chunk"'
// 'WRITE "Second chunk"'
// 'CLOSE'
```

#### 10.5.2 示例：收集写入到 WriteStream 的块到一个字符串中

在下一个示例中，我们创建了`WriteStream`的一个子类，它将所有写入的块收集到一个字符串中。我们可以通过`.getString()`方法访问该字符串：

```js
class StringWritableStream extends WritableStream {
 #string = '';
 constructor() {
 super({
 // We need to access the `this` of `StringWritableStream`.
 // Hence the arrow function (and not a method).
 write: (chunk) => {
 this.#string += chunk;
 },
 });
 }
 getString() {
 return this.#string;
 }
}
const stringStream = new StringWritableStream();
const writer = stringStream.getWriter();
try {
 await writer.write('How are');
 await writer.write(' you?');
 await writer.close();
} finally {
 writer.releaseLock()
}
assert.equal(
 stringStream.getString(),
 'How are you?'
);
```

这种方法的一个缺点是我们混合了两个 API：`WritableStream`的 API 和我们新的字符串流 API。另一种选择是委托给 WritableStream 而不是扩展它：

```js
function StringcreateWritableStream() {
 let string = '';
 return {
 stream: new WritableStream({
 write(chunk) {
 string += chunk;
 },
 }),
 getString() {
 return string;
 },
 };
}

const stringStream = StringcreateWritableStream();
const writer = stringStream.stream.getWriter();
try {
 await writer.write('How are');
 await writer.write(' you?');
 await writer.close();
} finally {
 writer.releaseLock()
}
assert.equal(
 stringStream.getString(),
 'How are you?'
);
```

这个功能也可以通过类来实现（而不是作为对象的工厂函数）。

### 10.6 使用 TransformStreams

一个 TransformStream：

+   通过其*writable side*接收输入，即 WritableStream。

+   然后可能会或可能不会转换这个输入。

+   结果可以通过一个 ReadableStream 来读取，它的*可读端*。

使用 TransformStreams 最常见的方式是“管道传递”它们：

```js
const transformedStream = readableStream.pipeThrough(transformStream);
```

`.pipeThrough()` 将`readableStream`管道到`transformStream`的可写端，并返回其可读端。换句话说：我们已经创建了一个新的`ReadableStream`，它是`readableStream`的转换版本。

`.pipeThrough()` 不仅接受 TransformStreams，还接受任何具有以下形式的对象：

```js
interface ReadableWritablePair<RChunk, WChunk> {
 readable: ReadableStream<RChunk>;
 writable: WritableStream<WChunk>;
}
```

#### 10.6.1 标准 TransformStreams

Node.js 支持以下标准 TransformStreams：

+   [编码（WHATWG 标准）](https://encoding.spec.whatwg.org) – `TextEncoderStream`和`TextDecoderStream`：

    +   这些流支持 UTF-8，但也支持[许多“旧编码”](https://encoding.spec.whatwg.org/#names-and-labels)。

    +   一个 Unicode 代码点被编码为多达四个 UTF-8 代码单元（字节）。在字节流中，编码的代码点可能会跨越块。`TextDecoderStream`可以正确处理这些情况。

    +   大多数 JavaScript 平台都可以使用（[`TextEncoderStream`](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoderStream#browser_compatibility), [`TextDecoderStream`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream#browser_compatibility)）。

+   [压缩流（W3C 草案社区组报告）](https://wicg.github.io/compression/) – [`CompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/CompressionStream), [`DecompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/DecompressionStream):

    +   [当前支持的压缩格式](https://wicg.github.io/compression/#supported-formats): `deflate`（ZLIB 压缩数据格式），`deflate-raw`（DEFLATE 算法），`gzip`（GZIP 文件格式）。

    +   在许多 JavaScript 平台上都可以使用（[`CompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/CompressionStream#browser_compatibility), [`DecompressionStream`](https://developer.mozilla.org/en-US/docs/Web/API/DecompressionStream#browser_compatibility)）。

##### 10.6.1.1 示例：解码一系列 UTF-8 编码的字节流

在下面的示例中，我们解码了一系列 UTF-8 编码的字节流：

```js
const response = await fetch('https://example.com');
const readableByteStream = response.body;
const readableStream = readableByteStream
 .pipeThrough(new TextDecoderStream('utf-8'));
for await (const stringChunk of readableStream) {
 console.log(stringChunk);
}
```

`response.body`是一个 ReadableByteStream，其块是`Uint8Array`的实例（[TypedArrays](https://exploringjs.com/impatient-js/ch_typed-arrays.html)）。我们通过`TextDecoderStream`将该流传输，以获得具有字符串块的流。

请注意，单独翻译每个字节块（例如通过[`TextDecoder`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)）是行不通的，因为[一个 Unicode 代码点在 UTF-8 中被编码为多达四个字节](https://exploringjs.com/impatient-js/ch_unicode.html#utf-8-unicode-transformation-format-8)，而这些字节可能不都在同一个块中。

##### 10.6.1.2 示例：创建一个用于标准输入的可读文本流

以下 Node.js 模块记录通过标准输入发送给它的所有内容：

```js
// echo-stdin.mjs
import {Readable} from 'node:stream';

const webStream = Readable.toWeb(process.stdin)
 .pipeThrough(new TextDecoderStream('utf-8'));
for await (const chunk of webStream) {
 console.log('>>>', chunk);
}
```

我们可以通过存储在`process.stdin`中的流访问标准输入（`process`是一个全局 Node.js 变量）。如果我们不为此流设置编码并通过`Readable.toWeb()`进行转换，我们将获得一个字节流。我们通过 TextDecoderStream 将其传输，以获得一个文本流。

请注意，我们逐步处理标准输入：一旦另一个块可用，我们就会记录它。换句话说，我们不会等到标准输入完成。当数据要么很大要么只是间歇性发送时，这是很有用的。

### 10.7 实现自定义 TransformStreams

我们可以通过将 Transformer 对象传递给`TransformStream`的构造函数来实现自定义 TransformStream。这样的对象具有以下类型（随意略过此类型和其属性的解释；当我们在示例中遇到它们时，它们将再次解释）：

```js
interface Transformer<TInChunk, TOutChunk> {
 start?(
 controller: TransformStreamDefaultController<TOutChunk>
 ): void | Promise<void>;
 transform?(
 chunk: TInChunk,
 controller: TransformStreamDefaultController<TOutChunk>
 ): void | Promise<void>;
 flush?(
 controller: TransformStreamDefaultController<TOutChunk>
 ): void | Promise<void>;
}
```

这些属性的解释：

+   `.start(controller)`在我们调用`TransformStream`的构造函数之后立即调用。在这里，我们可以在转换开始之前准备好一些东西。

+   `.transform(chunk, controller)`执行实际的转换。它接收一个输入块，并可以使用其参数`controller`来排队一个或多个转换后的输出块。它也可以选择不排队任何内容。

+   `.flush(controller)`在所有输入块成功转换后调用。在这里，我们可以在转换完成后执行清理工作。

这些方法中的每一个都可以返回一个 Promise，并且在 Promise 解决之前不会采取进一步的步骤。如果我们想要执行一些异步操作，这是很有用的。

参数`controller`具有以下类型：

```js
interface TransformStreamDefaultController<TOutChunk> {
 enqueue(chunk?: TOutChunk): void;
 readonly desiredSize: number | null;
 terminate(): void;
 error(err?: any): void;
}
```

+   `.enqueue(chunk)`将`chunk`添加到 TransformStream 的可读端（输出）。

+   `.desiredSize`返回可读端（输出）的 TransformStream 内部队列的期望大小。

+   `.terminate()`关闭可读端（输出）并错误可写端（输入）的 TransformStream。如果转换器对可写端（输入）的剩余块不感兴趣并希望跳过它们，则可以使用它。

+   `.error(err)`错误 TransformStream：以后所有与它的交互都将以错误值`err`失败。

TransformStream 中的背压如何？该类将背压从其可读端（输出）传播到其可写端（输入）。假设转换不会改变数据量太多。因此，Transform 可以忽略背压。但是，可以通过`transformStreamDefaultController.desiredSize`检测到它，并通过从`transformer.transform()`返回一个 Promise 来传播它。

#### 10.7.1 示例：将任意块的流转换为行流

`TransformStream`的以下子类将流转换为每个块都包含一行文本的流。也就是说，除了最后一个块可能以行尾（EOL）字符串结束之外，每个块都以行尾（EOL）字符串结束：Unix（包括 macOS）上为`'\n'`，Windows 上为`'\r\n'`。

```js
class ChunksToLinesTransformer {
 #previous = '';

 transform(chunk, controller) {
 let startSearch = this.#previous.length;
 this.#previous += chunk;
 while (true) {
 // Works for EOL === '\n' and EOL === '\r\n'
 const eolIndex = this.#previous.indexOf('\n', startSearch);
 if (eolIndex < 0) break;
 // Line includes the EOL
 const line = this.#previous.slice(0, eolIndex+1);
 controller.enqueue(line);
 this.#previous = this.#previous.slice(eolIndex+1);
 startSearch = 0;
 }
 }

 flush(controller) {
 // Clean up and enqueue any text we’re still holding on to
 if (this.#previous.length > 0) {
 controller.enqueue(this.#previous);
 }
 }
}
class ChunksToLinesStream extends TransformStream {
 constructor() {
 super(new ChunksToLinesTransformer());
 }
}

const stream = new ReadableStream({
 async start(controller) {
 controller.enqueue('multiple\nlines of\ntext');
 controller.close();
 },
});
const transformStream = new ChunksToLinesStream();
const transformed = stream.pipeThrough(transformStream);

for await (const line of transformed) {
 console.log('>>>', JSON.stringify(line));
}

// Output:
// '>>> "multiple\n"'
// '>>> "lines of\n"'
// '>>> "text"'
```

请注意，[Deno 的内置`TextLineStream`](https://doc.deno.land/https://deno.land/std@0.141.0/streams/mod.ts/~/TextLineStream)提供类似的功能。

提示：我们也可以通过异步生成器进行这种转换。它将异步迭代 ReadableStream 并返回一个包含行的异步可迭代对象。其实现在§9.4“通过异步生成器转换可读流”中显示。

#### 10.7.2 提示：异步生成器也非常适合转换流

由于 ReadableStreams 是异步可迭代的，我们可以使用[异步生成器](https://exploringjs.com/impatient-js/ch_async-iteration.html#async-generators)来转换它们。这导致非常优雅的代码：

```js
const stream = new ReadableStream({
 async start(controller) {
 controller.enqueue('one');
 controller.enqueue('two');
 controller.enqueue('three');
 controller.close();
 },
});

async function* prefixChunks(prefix, asyncIterable) {
 for await (const chunk of asyncIterable) {
 yield '> ' + chunk;
 }
}

const transformedAsyncIterable = prefixChunks('> ', stream);
for await (const transformedChunk of transformedAsyncIterable) {
 console.log(transformedChunk);
}

// Output:
// '> one'
// '> two'
// '> three'
```

### 10.8 仔细观察背压

让我们仔细观察背压。考虑以下管道链：

```js
rs.pipeThrough(ts).pipeTo(ws);
```

`rs`是一个 ReadableStream，`ts`是一个 TransformStream，`ws`是一个 WritableStream。这些是由前一个表达式创建的连接（`.pipeThrough`使用`.pipeTo`将`rs`连接到`ts`的可写端）：

```js
rs -pipeTo-> ts{writable,readable} -pipeTo-> ws
```

观察：

+   `rs`的基础源可以被视为在`rs`之前的管道链成员。

+   `ws`的基础接收器可以被视为在`ws`之后的管道链成员。

+   每个流都有一个内部缓冲区：ReadableStreams 在其基础源之后进行缓冲。WritableStreams 在其基础接收器之前进行缓冲。

假设`ws`的基础接收器速度慢，`ws`的缓冲区最终满了。然后发生以下步骤：

+   `ws`发出满的信号。

+   `pipeTo`停止从`ts.readable`读取。

+   `ts.readable`发出满的信号。

+   `ts`停止从`ts.writable`移动块到`ts.readable`。

+   `ts.writable`发出满的信号。

+   `pipeTo`停止从`rs`读取。

+   `rs`向其基础源发出满的信号。

+   基础源暂停。

这个例子说明我们需要两种功能：

+   接收数据的实体需要能够发出背压信号。

+   发送数据的实体需要对信号做出反应，施加背压。

让我们探索这些功能在 web 流 API 中是如何实现的。

#### 10.8.1 发出背压

背压由接收数据的实体发出信号。Web 流有两个这样的实体：

+   WritableStream 通过 Writer 方法`.write()`接收数据。

+   当其基础源调用 ReadableStreamDefaultController 方法`.enqueue()`时，ReadableStream 接收数据。

在这两种情况下，输入都通过队列进行缓冲。施加背压的信号是队列已满。让我们看看如何检测到这一点。

这些是队列的位置：

+   一个 WritableStream 的队列在 WritableStreamDefaultController 中内部存储（[参见 web 流标准](https://streams.spec.whatwg.org/#ws-default-controller-internal-slots)）。

+   一个 ReadableStream 的队列在 ReadableStreamDefaultController 中内部存储（[参见 web 流标准](https://streams.spec.whatwg.org/#rs-default-controller-internal-slots)）。

队列的*期望大小*是一个数字，表示队列中还有多少空间：

+   如果队列中仍有空间，则为正。

+   如果队列已达到其最大大小，则为零。

+   如果队列已超过其最大大小，则为负。

因此，如果期望的大小为零或更少，我们必须施加背压。它可以通过包含队列的对象的 getter`.desiredSize`获得。

期望的大小是如何计算的？通过指定所谓的*排队策略*的对象。`ReadableStream`和`WritableStream`具有默认的排队策略，可以通过它们的构造函数的可选参数进行覆盖。[接口`QueuingStrategy`](https://streams.spec.whatwg.org/#dictdef-queuingstrategy)有两个属性：

+   方法`.size(chunk)`返回`chunk`的大小。

    +   队列的当前大小是它包含的块的大小之和。

+   属性`.highWaterMark`指定队列的最大大小。

队列的期望大小是高水位标记减去队列的当前大小。

#### 10.8.2 对背压的反应

发送数据的实体需要对信号背压做出反应，通过施加背压。

##### 10.8.2.1 通过 Writer 写入 WritableStream 的代码

+   我们可以在`writer.ready`中等待 Promise。在等待期间，我们被阻塞，期望的背压得到了实现。一旦队列中有空间，Promise 就会被实现。当`writer.desiredSize`的值大于零时，实现会被触发。

+   或者，我们可以等待`writer.write()`返回的 Promise。如果我们这样做，队列甚至不会被填满。

如果我们愿意，我们还可以根据`writer.desiredSize`来确定我们的块的大小。

##### 10.8.2.2 ReadableStream 的底层源

可以传递给 ReadableStream 的底层源对象包装了外部源。在某种程度上，它也是管道链的成员；在其 ReadableStream 之前的成员。

+   只有在队列中有空间时，才会要求底层拉取源提供新数据。在没有空间时，会自动施加背压，因为没有数据被拉取。

+   在入队后，底层推送源应检查`controller.desiredSize`：如果为零或更少，则应通过暂停其外部源来施加背压。

##### 10.8.2.3 WritableStream 的底层接收端

可以传递给 WritableStream 的底层接收端对象包装了外部接收端。在某种程度上，它也是管道链的成员；在其 WritableStream 之后的成员。

每个外部接收端以不同的方式（在某些情况下根本不）信号背压。底层接收端可以通过从方法`.write()`返回一个被实现的 Promise 来施加背压，一旦写入完成。在[web 流标准中有一个例子](https://streams.spec.whatwg.org/#example-ws-backpressure)，演示了这是如何工作的。

##### 10.8.2.4 一个 transformStream（`.writable` `→` `.readable`）

TransformStream 通过为前者实现底层接收端和为后者实现底层源，将其可写端连接到其可读端。它具有一个内部插槽`.[[backpressure]]`，指示内部背压当前是否处于活动状态。

+   可写端的底层接收器的`.write()`方法会异步等待，直到没有内部背压，然后将另一个块提供给 TransformStream 的转换器（web streams 标准：[`TransformStreamDefaultSinkWriteAlgorithm`](https://streams.spec.whatwg.org/#transform-stream-default-sink-write-algorithm)）。然后转换器可以通过其 TransformStreamDefaultController 加入一些内容。请注意，`.write()`返回一个 Promise，在方法完成时会被满足。在此之前，WriteStream 通过其队列缓冲传入的写请求。因此，可写端的背压通过该队列及其期望的大小来表示。

+   如果通过 TransformStreamDefaultController 将一个块加入队列，并且可读端的队列变满了，TransformStream 的背压就会被激活（web streams 标准：[`TransformStreamDefaultControllerEnqueue`](https://streams.spec.whatwg.org/#transform-stream-default-controller-enqueue)）。

+   如果从读取器中读取了一些内容，`ReadableStream`的背压可能会被取消（web streams 标准：[`ReadableStreamDefaultReaderRead`](https://streams.spec.whatwg.org/#readable-stream-default-reader-read)）：

    +   如果队列中现在有空间，可能是时候调用底层源的`.pull()`了（web streams 标准：[`.[[PullSteps]]`](https://streams.spec.whatwg.org/#rs-default-controller-private-pull)）。

    +   可读端的底层源的`.pull()`会取消背压（web streams 标准：[`TransformStreamDefaultSourcePullAlgorithm`](https://streams.spec.whatwg.org/#transform-stream-default-source-pull)）。

##### 10.8.2.5 `.pipeTo()`（ReadableStream `→` WritableStream）

`.pipeTo()`通过读取器从 ReadableStream 读取块，并通过写入器将它们写入 WritableStream。当`writer.desiredSize`为零或更小时，它会暂停（web streams 标准：[`ReadableStreamPipeTo`](https://streams.spec.whatwg.org/#readable-stream-pipe-to)的第 15 步）。

### 10.9 字节流

到目前为止，我们只使用过*文本流*，流的块是字符串。但是 web streams API 也支持*字节流*，用于二进制数据，其中块是 Uint8Arrays（[TypedArrays](https://exploringjs.com/impatient-js/ch_typed-arrays.html)）：

+   `ReadableStream`有一个特殊的`'bytes'`模式。

+   `WritableStream`本身不关心块是字符串还是 Uint8Arrays。因此，实例是文本流还是字节流取决于底层接收器可以处理什么类型的块。

+   `TransformStream`可以处理什么类型的块也取决于其 Transformer。

接下来，我们将学习如何创建可读的字节流。

#### 10.9.1 可读的字节流

`ReadableStream`构造函数创建的流的类型取决于可选的属性`.type`和可选的第一个参数`underlyingSource`：

+   如果`.type`被省略或没有提供底层源，则新实例是一个文本流。

+   如果`.type`是字符串`'bytes'`，则新实例是一个字节流：

    ```js
    const readableByteStream = new ReadableStream({
     type: 'bytes',
     async start() { /*...*/ }
     // ...
    });
    ```

如果一个 ReadableStream 处于`'bytes'`模式，会发生什么变化？

在默认模式下，底层源可以返回任何类型的块。在字节模式下，块必须是 ArrayBufferViews，即 TypedArrays（例如 Uint8Arrays）或 DataViews。

此外，可读的字节流可以创建两种读取器：

+   `.getReader()`返回一个`ReadableStreamDefaultReader`的实例。

+   `.getReader({mode: 'byob'})` 返回一个 `ReadableStreamBYOBReader` 的实例。

“BYOB” 代表 “Bring Your Own Buffer”，意味着我们可以传递一个缓冲区（ArrayBufferView）给 `reader.read()`。之后，该 ArrayBufferView 将被分离并且不再可用。但是`.read()` 返回其数据在一个新的 ArrayBufferView 中，该 ArrayBufferView 具有相同的类型并访问相同的 ArrayBuffer 的相同区域。

此外，可读的字节流具有不同的控制器：它们是`ReadableByteStreamController`的实例（而不是`ReadableStreamDefaultController`）。除了强制底层源将 ArrayBufferViews（TypedArrays 或 DataViews）入队之外，它还通过[其属性`.byobRequest`](https://streams.spec.whatwg.org/#rbs-controller-prototype)支持 ReadableStreamBYOBReaders。底层源将其数据写入存储在此属性中的 BYOBRequest。Web 流标准在[其“创建流的示例”部分](https://streams.spec.whatwg.org/#creating-examples)中有两个使用`.byobRequest`的示例。

#### 10.9.2 示例：填充随机数据的无限可读的字节流

在下一个示例中，创建一个无限可读的字节流，用随机数据填充其块（灵感来自：[`example4.mjs` in “在 Node.js 中实现 Web 流 API”](https://www.jasnell.me/posts/webstreams#creating-and-using-a-readablestream)）。

```js
import {promisify} from 'node:util';
import {randomFill} from 'node:crypto';
const asyncRandomFill = promisify(randomFill);

const readableByteStream = new ReadableStream({
 type: 'bytes',
 async pull(controller) {
 const byobRequest = controller.byobRequest;
 await asyncRandomFill(byobRequest.view);
 byobRequest.respond(byobRequest.view.byteLength);
 },
});

const reader = readableByteStream.getReader({mode: 'byob'});
const buffer = new Uint8Array(10); // (A)
const firstChunk = await reader.read(buffer); // (B)
console.log(firstChunk);
```

由于`readableByteStream`是无限的，我们无法循环读取它。这就是为什么我们只读取它的第一个块（B 行）。

我们在 A 行创建的缓冲区在 B 行之后被传输，因此无法读取。

#### 10.9.3 示例：压缩可读的字节流

在下面的示例中，我们创建一个可读的字节流，并将其通过一个将其压缩为 GZIP 格式的流：

```js
const readableByteStream = new ReadableStream({
 type: 'bytes',
 start(controller) {
 // 256 zeros
 controller.enqueue(new Uint8Array(256));
 controller.close();
 },
});
const transformedStream = readableByteStream.pipeThrough(
 new CompressionStream('gzip'));
await logChunks(transformedStream);

async function logChunks(readableByteStream) {
 const reader = readableByteStream.getReader();
 try {
 while (true) {
 const {done, value} = await reader.read();
 if (done) break;
 console.log(value);
 }
 } finally {
 reader.releaseLock();
 }
}
```

#### 10.9.4 示例：通过`fetch()`读取网页

`fetch()`的结果解析为一个响应对象，其属性`.body`是一个可读的字节流。我们通过`TextDecoderStream`将该字节流转换为文本流：

```js
const response = await fetch('https://example.com');
const readableByteStream = response.body;
const readableStream = readableByteStream.pipeThrough(
 new TextDecoderStream('utf-8'));
for await (const stringChunk of readableStream) {
 console.log(stringChunk);
}
```

### 10.10 Node.js 特定的辅助函数

Node.js 是唯一支持以下辅助函数的 Web 平台，它称之为[*实用消费者*](https://nodejs.org/api/webstreams.html#utility-consumers)：

```js
import {
 arrayBuffer,
 blob,
 buffer,
 json,
 text,
} from 'node:stream/consumers';
```

这些函数将 Web ReadableStreams、Node.js Readables 和 AsyncIterators 转换为被满足的 Promise：

+   ArrayBuffers（`arrayBuffer()`）

+   Blobs（`blob()`）

+   Node.js 缓冲区（`buffer()`）

+   JSON 对象（`json()`）

+   字符串（`text()`）

假定二进制数据为 UTF-8 编码：

```js
import * as streamConsumers from 'node:stream/consumers';

const readableByteStream = new ReadableStream({
 type: 'bytes',
 start(controller) {
 // TextEncoder converts strings to UTF-8 encoded Uint8Arrays
 const encoder = new TextEncoder();
 const view = encoder.encode('"😀"');
 assert.deepEqual(
 view,
 Uint8Array.of(34, 240, 159, 152, 128, 34)
 );
 controller.enqueue(view);
 controller.close();
 },
});
const jsonData = await streamConsumers.json(readableByteStream);
assert.equal(jsonData, '😀');
```

字符串流按预期工作：

```js
import * as streamConsumers from 'node:stream/consumers';

const readableByteStream = new ReadableStream({
 start(controller) {
 controller.enqueue('"😀"');
 controller.close();
 },
});
const jsonData = await streamConsumers.json(readableByteStream);
assert.equal(jsonData, '😀');
```

### 10.11 进一步阅读

本节提到的所有材料都是本章的来源。

本章不涵盖 Web 流 API 的每个方面。您可以在此处找到更多信息：

+   [“WHATWG 流标准”](https://streams.spec.whatwg.org/) by Adam Rice, Domenic Denicola, Mattias Buelens, and 吉野剛史 (Takeshi Yoshino)

+   [“Web Streams API”](https://nodejs.org/api/webstreams.html) in Node.js 文档

更多材料：

+   Web 流 API：

    +   [“在 Node.js 中实现 Web 流 API”](https://www.jasnell.me/posts/webstreams) by James M. Snell

    +   [“流 API”](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 在 MDN 上

    +   [“流-权威指南”](https://web.dev/streams/) by Thomas Steiner

+   背压：

    +   [“Node.js 流中的背压”](https://enlear.academy/nodejs-backpressuring-in-streams-52638f505e1b) by Vladimir Topolev

    +   [“流中的背压”](https://nodejs.org/en/docs/guides/backpressuring-in-streams/) in Node.js 文档

+   Unicode（代码点，UTF-8，UTF-16 等）：[“Unicode 简介”章节](https://exploringjs.com/impatient-js/ch_unicode.html) in “JavaScript for impatient programmers”

+   [“异步迭代”章节](https://exploringjs.com/impatient-js/ch_async-iteration.html) in “JavaScript for impatient programmers”

+   [“Typed Arrays：处理二进制数据”章节](https://exploringjs.com/impatient-js/ch_typed-arrays.html) in “JavaScript for impatient programmers”

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/10)
