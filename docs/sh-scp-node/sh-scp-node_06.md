# 四、Node.js 概述：架构、API、事件循环、并发性

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_nodejs-overview.html`](https://exploringjs.com/nodejs-shell-scripting/ch_nodejs-overview.html)

* * *

+   4.1 Node.js 平台

    +   4.1.1 全局 Node.js 变量

    +   4.1.2 内置 Node.js 模块

    +   4.1.3 Node.js 函数的不同风格

+   4.2 Node.js 事件循环

    +   4.2.1 运行到完成使代码更简单

    +   4.2.2 为什么 Node.js 代码在单线程中运行？

    +   4.2.3 真实的事件循环有多个阶段

    +   4.2.4 Next-tick 任务和微任务

    +   4.2.5 比较直接调度任务的不同方式

    +   4.2.6 Node.js 应用程序何时退出？

+   4.3 libuv：处理 Node.js 异步 I/O（以及更多）的跨平台库

    +   4.3.1 libuv 如何处理异步 I/O

    +   4.3.2 libuv 如何处理阻塞 I/O

    +   4.3.3 libuv 处理异步 I/O 之外的功能

+   4.4 通过用户代码逃离主线程

    +   4.4.1 工作线程

    +   4.4.2 集群

    +   4.4.3 子进程

+   4.5 本章的来源

    +   4.5.1 致谢

* * *

本章概述了 Node.js 的工作原理：

+   它的架构是什么样的。

+   它的 API 是如何结构化的。

    +   全局变量和内置模块的一些亮点。

+   它如何通过*事件循环*在单线程中运行 JavaScript。

+   在这个平台上并发 JavaScript 的选项。

### 4.1 Node.js 平台

以下图表概述了 Node.js 的结构：

![](img/a4e5a698987cc980760cc6874988c161.png)

Node.js 应用程序可用的 API 包括：

+   ECMAScript 标准库（它是语言的一部分）

+   Node.js 的 API（不是语言本身的一部分）：

    +   一些 API 是通过全局变量提供的：

        +   特别是跨平台的 web API，比如`fetch`和`CompressionStream`属于这一类别。

        +   但是一些仅适用于 Node.js 的 API 也是全局的，例如`process`。

    +   其余的 Node.js API 是通过内置模块提供的 - 例如，`'node:path'`（处理文件系统路径的函数和常量）和`'node:fs'`（与文件系统相关的功能）。

Node.js 的 API 部分是用 JavaScript 实现的，部分是用 C++实现的。后者是为了与操作系统进行接口。

Node.js 通过嵌入的 V8 JavaScript 引擎运行 JavaScript（与 Google 的 Chrome 浏览器使用的相同引擎）。

#### 4.1.1 全局 Node.js 变量

这些是[Node 的全局变量](https://nodejs.org/api/globals.html)的一些亮点：

+   `crypto`让我们可以访问与 web 兼容的[crypto API](https://developer.mozilla.org/en-US/docs/Web/API/crypto_property)。

+   `console`与浏览器中的全局变量（`console.log()`等）有很多重叠。

+   `fetch()`让我们可以使用[Fetch 浏览器 API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)。

+   `process`包含一个[类`Process`的实例](https://nodejs.org/api/process.html)，并且让我们访问命令行参数、标准输入、标准输出等。

+   `structuredClone()`是一个兼容浏览器的用于克隆对象的函数。

+   `URL`是一个处理 URL 的兼容浏览器的类。

在本章中还提到了更多的全局变量。

##### 4.1.1.1 使用模块而不是全局变量

以下内置模块提供了全局变量的替代方案：

+   `'node:console'`是全局变量`console`的替代方案：

    ```js
    console.log('Hello!');

    import {log} from 'node:console';
    log('Hello!');
    ```

+   `'node:process'`是全局变量`process`的替代方案：

    ```js
    console.log(process.argv);

    import {argv} from 'node:process';
    console.log(process.argv);
    ```

原则上，使用模块比使用全局变量更清晰。然而，使用全局变量`console`和`process`是已经建立的模式，偏离这些模式也有缺点。

#### 4.1.2 内置的 Node.js 模块

Node 的大多数 API 都是通过模块提供的。以下是一些经常使用的模块（按字母顺序排列）：

+   [`'node:assert/strict'`](https://nodejs.org/api/assert.html)：断言是检查条件是否满足并在不满足时报告错误的函数。它们可以用于应用程序代码和单元测试。这是使用此 API 的一个例子：

    ```js
    import * as assert from 'node:assert/strict';
    assert.equal(3 + 4, 7);
    assert.equal('abc'.toUpperCase(), 'ABC');

    assert.deepEqual({prop: true}, {prop: true}); // deep comparison
    assert.notEqual({prop: true}, {prop: true}); // shallow comparison
    ```

+   [`'node:child_process'`](https://nodejs.org/api/child_process.html) 用于同步或在单独的进程中运行本机命令。该模块在§12“在子进程中运行 shell 命令”中有描述。

+   [`'node:fs'`](https://nodejs.org/api/fs.html) 提供了文件系统操作，如读取、写入、复制和删除文件和目录。更多信息，请参见§8“在 Node.js 上处理文件系统”。

+   [`'node:os'`](https://nodejs.org/api/os.html) 包含特定于操作系统的常量和实用函数。其中一些在§7“在 Node.js 上处理文件系统路径和文件 URL”中有解释。

+   [`'node:path'`](https://nodejs.org/api/path.html) 是一个用于处理文件系统路径的跨平台 API。它在§7“在 Node.js 上处理文件系统路径和文件 URL”中有描述。

+   [`'node:stream'`](https://nodejs.org/api/stream.html) 包含了一个特定于 Node.js 的流 API，这些流在§9“原生 Node.js 流”中有解释。

    +   Node.js 还支持[跨平台的 Web 流 API](https://nodejs.org/api/webstreams.html)，这是§10“在 Node.js 上使用 Web 流”的主题。

+   [`'node:util'`](https://nodejs.org/api/util.html) 包含各种实用函数。

    +   [函数`util.parseArgs()`](https://nodejs.org/api/util.html#utilparseargsconfig)在§16“使用`util.parseArgs()`解析命令行参数”中有描述。

模块`'node:module'`包含函数`builtinModules()`，它返回一个包含所有内置模块的规范符号的数组：

```js
import * as assert from 'node:assert/strict';
import {builtinModules} from 'node:module';
// Remove internal modules (whose names start with underscores)
const modules = builtinModules.filter(m => !m.startsWith('_'));
modules.sort();
assert.deepEqual(
 modules.slice(0, 5),
 [
 'assert',
 'assert/strict',
 'async_hooks',
 'buffer',
 'child_process',
 ]
);
```

#### 4.1.3 Node.js 函数的不同风格

在本节中，我们使用以下导入：

```js
import * as fs from 'node:fs';
```

Node 的函数有三种不同的风格。让我们以内置模块`'node:fs'`为例：

+   使用普通函数的同步风格 - 例如：

    +   [`fs.readFileSync(path, options?): string|Buffer`](https://nodejs.org/api/fs.html#fsreadfilesyncpath-options)

+   两种异步风格：

    +   使用基于回调的异步风格的函数 - 例如：

        +   [`fs.readFile(path, options?, callback): void`](https://nodejs.org/api/fs.html#fsreadfilepath-options-callback)

    +   使用基于 Promise 的异步风格的函数 - 例如：

        +   [`fsPromises.readFile(path, options?): Promise<string|Buffer>`](https://nodejs.org/api/fs.html#fspromisesreadfilepath-options)

我们刚刚看到的三个例子演示了具有类似功能的函数的命名约定：

+   一个基于回调的函数的基本名称是：`fs.readFile()`

+   其基于 Promise 的版本具有相同的名称，但在不同的模块中：`fsPromises.readFile()`

+   其同步版本的名称是基本名称加上后缀“Sync”：`fs.readFileSync()`

让我们更仔细地看看这三种风格是如何工作的。

##### 4.1.3.1 同步函数

同步函数最简单 - 它们立即返回值并将错误作为异常抛出：

```js
try {
 const result = fs.readFileSync('/etc/passwd', {encoding: 'utf-8'});
 console.log(result);
} catch (err) {
 console.error(err);
}
```

##### 4.1.3.2 基于 Promise 的函数

基于 Promise 的函数返回用结果实现的 Promise，并用错误拒绝：

```js
import * as fsPromises from 'node:fs/promises'; // (A)

try {
 const result = await fsPromises.readFile(
 '/etc/passwd', {encoding: 'utf-8'});
 console.log(result);
} catch (err) {
 console.error(err);
}
```

注意第 A 行中的模块说明符：基于 Promise 的 API 位于不同的模块中。

有关 Promise 的更多详细信息，请参阅[“JavaScript for impatient programmers”](https://exploringjs.com/impatient-js/ch_promises.html)。

##### 4.1.3.3 基于回调的函数

基于回调的函数将结果和错误传递给它们的最后一个参数：

```js
fs.readFile('/etc/passwd', {encoding: 'utf-8'},
 (err, result) => {
 if (err) {
 console.error(err);
 return;
 }
 console.log(result);
 }
);
```

这种风格在[Node.js 文档](https://nodejs.org/en/knowledge/getting-started/control-flow/what-are-callbacks/)中有更详细的解释。

### 4.2 Node.js 事件循环

默认情况下，Node.js 在单个线程中执行所有 JavaScript，即*主线程*。主线程不断运行*事件循环* - 一个执行 JavaScript 块的循环。每个块都是一个回调，可以被视为一个合作调度的任务。第一个任务包含我们使用的代码（来自模块或标准输入）启动 Node.js。其他任务通常稍后添加，原因是：

+   手动添加任务的代码

+   与文件系统进行 I/O（输入或输出），与网络套接字等。

+   等等。

事件循环的第一个近似值如下：

![](img/51a4aa1ae3b84e71c9266661c852658e.png)

也就是说，主线程运行类似于以下代码：

```js
while (true) { // event loop
 const task = taskQueue.dequeue(); // blocks
 task();
}
```

事件循环从*任务队列*中取出回调并在主线程中执行它们。如果任务队列为空，则出队*阻塞*（暂停主线程）。

稍后我们将探讨两个主题：

+   如何从事件循环中退出。

+   如何绕过 JavaScript 在单个线程中运行的限制。

为什么这个循环被称为*事件循环*？许多任务是响应事件添加的，例如操作系统发送的事件，当输入数据准备好被处理时。

回调是如何添加到任务队列中的？这些是常见的可能性：

+   JavaScript 代码可以将任务添加到队列中，以便稍后执行。

+   当*事件发射器*（事件源）触发事件时，事件监听器的调用将被添加到任务队列中。

+   Node.js API 中的基于回调的异步操作遵循这种模式：

    +   我们请求某些东西，并给 Node.js 一个回调函数，它可以用来向我们报告结果。

    +   最终，操作要么在主线程中运行，要么在外部线程中运行（稍后详细介绍）。

    +   完成后，回调的调用将被添加到任务队列中。

以下代码显示了异步回调操作的实际操作。它从文件系统中读取文本文件：

```js
import * as fs from 'node:fs';

function handleResult(err, result) {
 if (err) {
 console.error(err);
 return;
 }
 console.log(result); // (A)
}
fs.readFile('reminder.txt', 'utf-8',
 handleResult
);
console.log('AFTER'); // (B)
```

这是输出：

```js
AFTER
Don’t forget!
```

`fs.readFile()`在另一个线程中执行读取文件的代码。在这种情况下，代码成功并将此回调添加到任务队列中：

```js
() => handleResult(null, 'Don’t forget!')
```

#### 4.2.1 运行到完成使代码更简单

Node.js 运行 JavaScript 代码的一个重要规则是：每个任务在其他任务运行之前都会完成（“运行到完成”）。我们可以在上一个示例中看到这一点：'AFTER'在 B 行之前被记录，因为初始任务在调用`handleResult()`的任务运行之前完成。

运行到完成意味着任务的生命周期不重叠，我们不必担心共享数据在后台被更改。这简化了 Node.js 代码。下一个示例演示了这一点。它实现了一个简单的 HTTP 服务器：

```js
// server.mjs
import * as http from 'node:http';

let requestCount = 1;
const server = http.createServer(
 (_req, res) => { // (A)
 res.writeHead(200);
 res.end('This is request number ' + requestCount); // (B)
 requestCount++; // (C)
 }
);
server.listen(8080);
```

我们通过`node server.mjs`运行此代码。之后，代码启动并等待 HTTP 请求。我们可以通过使用 Web 浏览器访问`http://localhost:8080`来发送请求。每次重新加载该 HTTP 资源时，Node.js 会调用从 A 行开始的回调函数。它提供了变量`requestCount`当前值的消息（B 行），并增加它（C 行）。

回调函数的每次调用都是一个新任务，变量`requestCount`在任务之间共享。由于运行到完成，它很容易读取和更新。不需要与其他同时运行的任务同步，因为没有其他任务。

#### 4.2.2 为什么 Node.js 代码在单个线程中运行？

为什么 Node.js 代码默认在单个线程（带有事件循环）中运行？这有两个好处：

+   正如我们已经看到的，如果只有一个线程，任务之间共享数据会更简单。

+   在传统的多线程代码中，需要较长时间才能完成的操作会阻塞当前线程，直到操作完成。此类操作的示例包括读取文件或处理 HTTP 请求。执行许多此类操作是昂贵的，因为每次都必须创建一个新线程。使用事件循环，每次操作的成本更低，特别是如果每个操作都不做太多。这就是为什么基于事件循环的 Web 服务器可以处理比基于线程的服务器更高的负载。

鉴于 Node 的一些异步操作在主线程之外的线程中运行（稍后会详细介绍），并通过任务队列向 JavaScript 报告，Node.js 实际上并不是单线程的。相反，我们使用单个线程来协调并发和异步运行的操作（在主线程中）。

这就是我们对事件循环的第一次了解。**如果您只需要一个表面的解释，可以随意跳过本节的其余部分**。继续阅读以了解更多细节。

#### 4.2.3 真实的事件循环有多个阶段

真实的事件循环有多个任务队列，它从中读取多个阶段（[您可以在 GitHub 存储库`nodejs/node`中查看一些 JavaScript 代码](https://github.com/nodejs/node/blob/main/lib/internal/process/task_queues.js)）。以下图表显示了其中最重要的阶段：

![](img/5d5cabb99a6fc7037b402c23a71f36e1.png)

图表中显示的事件循环阶段的作用是什么？

+   “定时器”阶段调用了*定时任务*，这些任务是通过以下方式添加到其队列中的：

    +   [`setTimeout(task, delay=1)`](https://nodejs.org/api/timers.html#settimeoutcallback-delay-args)会在`delay`毫秒后运行回调函数`task`。

    +   [`setInterval(task, delay=1)`](https://nodejs.org/api/timers.html#setintervalcallback-delay-args)会重复运行回调函数`task`，每次暂停持续`delay`毫秒。

+   “轮询”阶段检索和处理 I/O 事件，并从其队列中运行与 I/O 相关的任务。

+   “检查”阶段（“立即”阶段）执行通过以下方式安排的任务：

    +   [`setImmediate(task)`](https://nodejs.org/api/timers.html#setimmediatecallback-args)会尽快运行回调函数`task`（“轮询”阶段之后“立即”）。

每个阶段运行直到其队列为空，或者直到处理了最大数量的任务。除了“轮询”阶段外，每个阶段在处理其运行期间添加的任务之前会等待其下一个轮次。

##### 4.2.3.1 “轮询”阶段

+   如果轮询队列不为空，轮询阶段将遍历并运行其任务。

+   一旦轮询队列为空：

    +   如果有`setImmediate()`任务，处理将进入“检查”阶段。

    +   如果有准备好的定时器任务，处理将进入“定时器”阶段。

    +   否则，此阶段将阻塞整个主线程，并等待直到将新任务添加到轮询队列（或直到此阶段结束，见下文）。这些任务会立即处理。

如果此阶段花费的时间超过系统相关的时间限制，它将结束并运行下一个阶段。

#### 4.2.4 下一个任务和微任务

在每次调用任务后，会运行一个“子循环”，其中包括两个阶段：

![](img/433fc416f821b72a5dd9a3168a34f704.png)

子阶段处理：

+   通过 process.nextTick()排队的 next-tick 任务。

+   Microtasks，通过 queueMicrotask()、Promise reactions 等方式加入队列。

Next-tick 任务是 Node.js 特有的，Microtasks 是跨平台的 Web 标准（参见[MDN 的支持表](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask#browser_compatibility)）。

这个子循环一直运行，直到两个队列都为空。在其运行期间添加的任务会立即处理 - 子循环不会等到下一轮才执行。

#### 4.2.5 比较直接调度任务的不同方式

我们可以使用以下函数和方法将回调添加到其中一个任务队列中：

+   定时任务（“timers”阶段）

    +   setTimeout()（Web 标准）

    +   setInterval()（Web 标准）

+   非定时任务（“check”阶段）

    +   setImmediate()（Node.js 特有）

+   在当前任务之后立即运行的任务：

    +   process.nextTick()（Node.js 特有）

    +   queueMicrotask()：（Web 标准）

需要注意的是，通过延迟计划任务时，我们指定了任务将运行的最早可能时间。Node.js 并不总是能够在准确的预定时间运行它们，因为它只能在任务之间检查是否有定时任务到期。因此，长时间运行的任务可能会导致定时任务延迟。

##### 4.2.5.1 Next-tick 任务和 microtasks vs. normal tasks

考虑以下代码：

```js
function enqueueTasks() {
 Promise.resolve().then(() => console.log('Promise reaction 1'));
 queueMicrotask(() => console.log('queueMicrotask 1'));
 process.nextTick(() => console.log('nextTick 1'));
 setImmediate(() => console.log('setImmediate 1')); // (A)
 setTimeout(() => console.log('setTimeout 1'), 0);

 Promise.resolve().then(() => console.log('Promise reaction 2'));
 queueMicrotask(() => console.log('queueMicrotask 2'));
 process.nextTick(() => console.log('nextTick 2'));
 setImmediate(() => console.log('setImmediate 2')); // (B)
 setTimeout(() => console.log('setTimeout 2'), 0);
}

setImmediate(enqueueTasks);
```

我们使用 setImmediate()来避免 ESM 模块的一个特殊情况：它们在 microtasks 中执行，这意味着如果我们在 ESM 模块的顶层排队 microtasks，它们会在 next-tick 任务之前运行。正如我们将在接下来看到的，在大多数其他情境中是不同的。

这是前面代码的输出：

```js
nextTick 1
nextTick 2
Promise reaction 1
queueMicrotask 1
Promise reaction 2
queueMicrotask 2
setTimeout 1
setTimeout 2
setImmediate 1
setImmediate 2
```

观察：

+   所有 next-tick 任务都会在 enqueueTasks()之后立即执行。

+   它们后面是所有的 microtasks，包括 Promise reactions。

+   “timers”阶段在 immediate 阶段之后。这时定时任务被执行。

+   我们在 immediate（“check”）阶段（A 行和 B 行）添加了 immediate 任务。它们出现在输出的最后，这意味着它们不是在当前阶段执行的，而是在下一个 immediate 阶段执行的。

##### 4.2.5.2 在它们的阶段排队 next-tick 任务和 microtasks

下一个代码检查了如果我们在 next-tick 阶段排队 next-tick 任务，以及在 microtask 阶段排队 microtask 会发生什么：

```js
setImmediate(() => {
 setImmediate(() => console.log('setImmediate 1'));
 setTimeout(() => console.log('setTimeout 1'), 0);

 process.nextTick(() => {
 console.log('nextTick 1');
 process.nextTick(() => console.log('nextTick 2'));
 });

 queueMicrotask(() => {
 console.log('queueMicrotask 1');
 queueMicrotask(() => console.log('queueMicrotask 2'));
 process.nextTick(() => console.log('nextTick 3'));
 });
});
```

这是输出：

```js
nextTick 1
nextTick 2
queueMicrotask 1
queueMicrotask 2
nextTick 3
setTimeout 1
setImmediate 1
```

观察：

+   Next-tick 任务会首先执行。

+   “nextTick 2”在 next-tick 阶段排队并立即执行。只有在 next-tick 队列为空时，执行才会继续。

+   对于 microtasks 也是如此。

+   我们在 microtask 阶段排队了“nextTick 3”，执行循环回到了 next-tick 阶段。这些子阶段会重复，直到它们的队列都为空。然后执行才会移动到下一个全局阶段：首先是“timers”阶段（“setTimeout 1”）。然后是 immediate 阶段（“setImmediate 1”）。

##### 4.2.5.3 饿死事件循环阶段

以下代码探讨了哪种类型的任务可以通过无限递归*饿死*事件循环阶段（阻止它们运行）：

```js
import * as fs from 'node:fs/promises';

function timers() { // OK
 setTimeout(() => timers(), 0);
}
function immediate() { // OK
 setImmediate(() => immediate());
}

function nextTick() { // starves I/O
 process.nextTick(() => nextTick());
}

function microtasks() { // starves I/O
 queueMicrotask(() => microtasks());
}

timers();
console.log('AFTER'); // always logged
console.log(await fs.readFile('./file.txt', 'utf-8'));
```

“timers”阶段和 immediate 阶段不会执行在它们的阶段排队的任务。这就是为什么 timers()和 immediate()不会饿死 fs.readFile()，后者在“poll”阶段报告回来（这里也有一个 Promise reaction，但我们在这里忽略它）。

由于 next-tick 任务和 microtasks 的调度方式，nextTick()和 microtasks()都会阻止最后一行的输出。

#### 4.2.6 Node.js 应用何时退出？

在事件循环的每次迭代结束时，Node.js 都会检查是否是退出的时候。它会保持待处理的*超时*（定时任务）的引用计数：

+   通过 setImmediate()、setInterval()或 setTimeout()调度定时任务会增加引用计数。

+   运行定时任务会减少引用计数。

如果引用计数在事件循环迭代结束时为零，Node.js 会退出。

我们可以看到在以下示例中：

```js
function timeout(ms) {
 return new Promise(
 (resolve, _reject) => {
 setTimeout(resolve, ms); // (A)
 }
 );
}
await timeout(3_000);
```

Node.js 等待`timeout()`返回的 Promise 被实现。为什么？因为我们在 A 行安排的任务使事件循环保持活动状态。

相比之下，创建 Promise 不会增加引用计数：

```js
function foreverPending() {
 return new Promise(
 (_resolve, _reject) => {}
 );
}
await foreverPending(); // (A)
```

在这种情况下，在 A 行的`await`期间，执行暂时离开了这个（主）任务。在事件循环结束时，引用计数为零，Node.js 退出。但是，退出不成功。也就是说，退出代码不是 0，而是 13（[“未完成的顶级等待”](https://nodejs.org/api/process.html#exit-codes)）。

我们可以手动控制超时是否保持事件循环活动：默认情况下，通过`setImmediate()`，`setInterval()`和`setTimeout()`安排的任务在挂起状态时会保持事件循环活动。这些函数返回[class `Timeout`](https://nodejs.org/api/timers.html#class-timeout)的实例，其方法`.unref()`更改了默认设置，使得活动的超时不会阻止 Node.js 退出。方法`.ref()`恢复默认设置。

[Tim Perry 提到了`.unref()`的一个用例](https://httptoolkit.tech/blog/unblocking-node-with-unref/)：他的库使用`setInterval()`重复运行后台任务。该任务阻止应用程序退出。他通过`.unref()`解决了这个问题。

### 4.3 libuv：处理 Node.js 异步 I/O（以及更多）的跨平台库

libuv 是用 C 编写的库，支持许多平台（Windows，macOS，Linux 等）。Node.js 使用它来处理 I/O 和更多内容。

#### 4.3.1 libuv 如何处理异步 I/O

网络 I/O 是异步的，不会阻塞当前线程。这种 I/O 包括：

+   TCP

+   UDP

+   终端 I/O

+   管道（Unix 域套接字，Windows 命名管道等）

为了处理异步 I/O，libuv 使用本机内核 API 并订阅 I/O 事件（Linux 上的 epoll；BSD Unix 包括 macOS 上的 kqueue；SunOS 上的事件端口；Windows 上的 IOCP）。然后在发生时得到通知。所有这些活动，包括 I/O 本身，都发生在主线程上。

#### 4.3.2 libuv 如何处理阻塞 I/O

一些本机 I/O API 是阻塞的（不是异步的）-例如，文件 I/O 和一些 DNS 服务。libuv 从线程池中的线程调用这些 API（所谓的“工作池”）。这使得主线程可以异步使用这些 API。

#### 4.3.3 libuv 功能超出 I/O

libuv 不仅帮助 Node.js 处理 I/O。其他功能包括：

+   在线程池中运行任务

+   信号处理

+   高分辨率时钟

+   线程和同步原语

另外，libuv 有自己的事件循环，你可以在 GitHub 存储库`libuv/libuv`中查看其源代码（[函数`uv_run()`](https://github.com/libuv/libuv/blob/v1.x/src/unix/core.c)）。

### 4.4 通过用户代码逃离主线程

如果我们想让 Node.js 对 I/O 保持响应，我们应该避免在主线程任务中执行长时间运行的计算。有两种选择：

+   分区：我们可以将计算分成较小的部分，并通过`setImmediate()`运行每个部分。这使得事件循环能够在这些部分之间执行 I/O。

    +   一个优点是我们可以在每个部分执行 I/O。

    +   一个缺点是我们仍然减慢了事件循环。

+   卸载：我们可以在不同的线程或进程中执行我们的计算。

    +   缺点是我们不能从主线程以外的线程执行 I/O，与外部代码的通信变得更加复杂。

    +   优点是我们不会减慢事件循环，我们可以更好地利用多个处理器核心，并且其他线程中的错误不会影响主线程。

下面的小节涵盖了一些卸载的选项。

#### 4.4.1 Worker 线程

[Worker Threads](https://nodejs.org/api/worker_threads.html)实现了[跨平台 Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Worker#browser_compatibility)，但有一些区别-例如：

+   必须从模块导入 Worker Threads，通过全局变量访问 Web Workers。

+   在工作线程中，通过浏览器全局对象的方法来监听消息和发布消息。在 Node.js 中，我们使用`parentPort`进行导入。

+   我们可以从工作线程中使用大多数 Node.js API。在浏览器中，我们的选择更有限（无法使用 DOM 等）。

+   在 Node.js 中，可以传输更多的对象（[所有类扩展内部类`JSTransferable`的对象](https://github.com/nodejs/node/issues/37080)）比在浏览器中。

一方面，Worker Threads 确实是线程：它们比进程更轻量，并在同一进程中运行。

另一方面：

+   每个工作线程都运行自己的事件循环。

+   每个工作线程都有自己的 JavaScript 引擎实例和自己的 Node.js 实例 - 包括单独的全局变量。

    +   （具体来说，每个工作线程都是一个[*V8 隔离*](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/bindings/core/v8/V8BindingDesign.md)，它有自己的 JavaScript 堆，但与其他线程共享操作系统堆。）

+   线程之间共享数据是有限的：

    +   我们可以通过 SharedArrayBuffers 共享二进制数据/数字。

    +   [`Atomics`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics)提供原子操作和同步原语，有助于使用 SharedArrayBuffers 时。

    +   [通道消息 API](https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API)允许我们通过双向通道发送数据（“消息”）。数据可以是*克隆*（复制）或*传输*（移动）。后者更有效，并且仅受[少数数据结构](https://developer.mozilla.org/en-US/docs/Glossary/Transferable_objects#supported_objects)支持。

更多信息，请参阅[worker threads 的 Node.js 文档](https://nodejs.org/api/worker_threads.html)。

#### 4.4.2 集群

[Cluster](https://nodejs.org/api/cluster.html)是一个 Node.js 特定的 API。它允许我们运行 Node.js 进程的*集群*，我们可以用来分发工作负载。这些进程是完全隔离的，但共享服务器端口。它们可以通过通道传递 JSON 数据进行通信。

如果我们不需要进程隔离，可以使用更轻量的 Worker Threads。

#### 4.4.3 子进程

[Child process](https://nodejs.org/api/child_process.html)是另一个 Node.js 特定的 API。它允许我们生成运行本机命令（通常通过本机 shell）的新进程。此 API 在§12“在子进程中运行 shell 命令”中有介绍。

### 4.5 本章的来源

Node.js 事件循环：

+   Node.js 文档：[“Node.js 事件循环，定时器和`process.nextTick()`”](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

+   [“要真正理解 Node.js 事件循环，你应该知道的事情”](https://medium.com/the-node-js-collection/what-you-should-know-to-really-understand-the-node-js-event-loop-and-its-metrics-c4907b19da4c) by Daniel Khan

+   [“Node.js 如何决定是退出事件循环还是再次运行？”](https://stackoverflow.com/questions/46914025/node-exits-without-error-and-doesnt-await-promise-event-callback/46916601#46916601) by Mark Meyer

事件循环的视频（刷新了本章所需的一些背景知识）：

+   [“Node 的事件循环从内部到外部”](https://www.youtube.com/watch?v=P9csgxBgaZ8)（由 Sam Roberts）解释了为什么操作系统增加了对异步 I/O 的支持；哪些操作是异步的，哪些不是（必须在线程池中运行）等。

+   [“Node.js 事件循环：并非单线程”](https://www.youtube.com/watch?v=zphcsoSJMvM)（由 Bryan Hughes）包含了多任务处理的简要历史（协作多任务处理，抢占式多任务处理，对称多线程，异步多任务处理）；进程与线程；同步运行 I/O 与在线程池中运行等。

libuv：

+   libuv 文档：

    +   [“设计概述”](http://docs.libuv.org/en/latest/design.html)

    +   [“libuv 基础知识”](http://docs.libuv.org/en/latest/guide/basics.html)

+   [“深入了解 libuv”](https://www.youtube.com/watch?v=sGTRmPiXD4Y) 由 Saúl Ibarra Corretgé

+   [“I/O 多路复用（select vs. poll vs. epoll/kqueue）-问题和算法”](https://nima101.github.io/io_multiplexing) 由 Nima Aghdaii

+   [“开发人员启动 I/O 操作。接下来会发生什么，你将不会相信。”](https://cjihrig.com/node_libuv_io) 由 Colin J. Ihrig

    +   跟踪 JavaScript 函数调用，从 JavaScript 到 Node 的核心再到 libuv，然后返回。

JavaScript 并发：

+   在 Node.js 文档中“不要阻塞事件循环（或工作线程池）”中的部分[“不阻塞事件循环的复杂计算”](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/#complex-calculations-without-blocking-the-event-loop)

+   [“理解 Node.js 中的工作线程”](https://nodesource.com/blog/worker-threads-nodejs/) 由 Liz Parody

+   [“2021 年 Web Workers 的现状”](https://www.smashingmagazine.com/2021/06/web-workers-2021/) 由 Surma

+   视频[“Node.js：通往工作线程的道路”](https://www.youtube.com/watch?v=-ssCzHoUI7M) 由 Anna Henningsen

#### 4.5.1 致谢

+   我非常感谢[Dominic Elm](https://twitter.com/elmd_)审阅本章并提供重要反馈。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/4)
