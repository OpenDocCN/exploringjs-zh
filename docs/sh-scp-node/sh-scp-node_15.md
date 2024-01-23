# 12 在子进程中运行 shell 命令

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_nodejs-child-process.html`](https://exploringjs.com/nodejs-shell-scripting/ch_nodejs-child-process.html)

* * *

+   12.1 本章概述](ch_nodejs-child-process.html#overview-of-this-chapter)

    +   12.1.1 Windows vs. Unix

    +   12.1.2 我们在示例中经常使用的功能

+   12.2 异步生成进程：`spawn()`

    +   12.2.1 `spawn()`的工作原理

    +   12.2.2 何时执行 shell 命令？](ch_nodejs-child-process.html#when-is-the-shell-command-executed)

    +   12.2.3 仅命令模式 vs. 参数模式](ch_nodejs-child-process.html#spawn-argument-modes)

    +   12.2.4 向子进程的 stdin 发送数据

    +   12.2.5 手动进行管道传输](ch_nodejs-child-process.html#piping-manually)

    +   12.2.6 处理不成功的退出（包括错误）

    +   12.2.7 等待子进程退出](ch_nodejs-child-process.html#waiting-for-the-exit-of-a-child-process)

    +   12.2.8 终止子进程](ch_nodejs-child-process.html#terminating-child-processes)

+   12.3 同步生成进程：`spawnSync()`

    +   12.3.1 何时执行 shell 命令？](ch_nodejs-child-process.html#when-is-the-shell-command-executed-1)

    +   12.3.2 从 stdout 读取](ch_nodejs-child-process.html#reading-from-stdout)

    +   12.3.3 向子进程的 stdin 发送数据

    +   12.3.4 处理不成功的退出（包括错误）

+   12.4 基于`spawn()`的异步辅助函数

    +   12.4.1 `exec()`

    +   12.4.2 `execFile()`

+   12.5 基于`spawnAsync()`的同步辅助函数

    +   12.5.1 `execSync()`

    +   12.5.2 `execFileSync()`

+   12.6 有用的库](ch_nodejs-child-process.html#useful-libraries)

    +   12.6.1 tinysh：用于生成 shell 命令的辅助程序

    +   12.6.2 node-powershell：通过 Node.js 执行 Windows PowerShell 命令

+   12.7 在模块`'node:child_process'`的功能之间进行选择

* * *

在本章中，我们将探讨如何通过模块`'node:child_process'`从 Node.js 执行 shell 命令。

### 12.1 本章概述

模块`'node:child_process'`有一个用于执行 shell 命令（在*生成的*子进程中）的函数，有两个版本：

+   一个异步版本的`spawn()`。

+   一个同步版本的`spawnSync()`。

我们将首先探讨`spawn()`，然后是`spawnSync()`。最后，我们将看一下基于它们并且相对类似的以下函数：

+   基于`spawn()`：

    +   `exec()`

    +   `execFile()`

+   基于`spawnSync()`：

    +   `execSync()`

    +   `execFileSync()`

#### 12.1.1 Windows vs. Unix

本章中显示的代码在 Unix 上运行，但我也在 Windows 上进行了测试-其中大部分代码需要进行轻微更改（例如以`'\r\n'`而不是`'\n'`结尾）。

#### 12.1.2 我们在示例中经常使用的功能

以下功能在示例中经常出现。这就是为什么在这里解释一次：

+   断言：对于原始值使用`assert.equal()`，对于对象使用`assert.deepEqual()`。示例中从未显示必要的导入：

    ```js
    import * as assert from 'node:assert/strict';
    ```

+   函数`Readable.toWeb()`将 Node 的原生`stream.Readable`转换为 web 流（`ReadableStream`的实例）。这在§10“在 Node.js 上使用 web 流”中有解释。示例中始终导入`Readable`。

+   异步函数`readableStreamToString()`会消耗可读的 web 流并返回一个字符串（包装在 Promise 中）。这在 web 流章节中有解释。假定这个函数在示例中是可用的。

### 12.2 异步生成进程：`spawn()`

#### 12.2.1 `spawn()`的工作原理

```js
spawn(
 command: string,
 args?: Array<string>,
 options?: Object
): ChildProcess
```

[`spawn()`](https://nodejs.org/api/child_process.html#child_processspawncommand-args-options) 异步地在新进程中执行命令：该进程与 Node 的主 JavaScript 进程并行运行，我们可以通过各种方式与其通信（通常通过流）。

接下来，有关`spawn()`的参数和结果的文档。如果您喜欢通过示例学习，可以跳过该内容，继续阅读后面的小节。

##### 12.2.1.1 参数：`command`

`command`是一个包含 shell 命令的字符串。有两种使用该参数的模式：

+   仅命令模式：省略`args`，`command`包含整个 shell 命令。我们甚至可以使用 shell 功能，如在多个可执行文件之间进行管道传输，将 I/O 重定向到文件，变量和通配符。

    +   `options.shell`必须为`true`，因为我们需要一个 shell 来处理 shell 功能。

+   参数模式：`command`仅包含命令的名称，`args`包含其参数。

    +   如果`options.shell`为`true`，则参数中的许多元字符会被解释，并且通配符和变量名称等功能会起作用。

    +   如果`options.shell`为`false`，则字符串会直接使用，我们不必转义元字符。

这两种模式在本章后面进行了演示。

##### 12.2.1.2 参数：`options`

以下`options`最有趣：

+   `.shell: boolean|string`（默认值：`false`）

    是否应使用 shell 来执行命令？

    +   在 Windows 上，此选项几乎总是应为`true`。例如，否则无法执行`.bat`和`.cmd`文件。

    +   在 Unix 上，只有核心 shell 功能（例如管道，I/O 重定向，文件名通配符和变量）在`.shell`为`false`时不可用。

    +   如果`.shell`为`true`，我们必须小心处理用户输入并对其进行清理，因为很容易执行任意代码。如果我们想将其用作非元字符，则还必须转义元字符。

    +   我们还可以将`.shell`设置为 shell 可执行文件的路径。然后 Node.js 将使用该可执行文件来执行命令。如果我们将`.shell`设置为`true`，Node.js 将使用：

        +   Unix：`'/bin/sh'`

        +   Windows：`process.env.ComSpec`

+   `.cwd: string | URL`

    指定在执行命令时要使用的*当前工作目录*（CWD）。

+   `.stdio: Array<string|Stream>|string`

    配置标准 I/O 的设置方式。下面会有解释。

+   `.env: Object`（默认值：`process.env`）

    让我们为子进程指定 shell 变量。提示：

    +   查看`process.env`（例如在 Node.js REPL 中）以查看存在哪些变量。

    +   我们可以使用扩展运算符来非破坏性地覆盖现有变量 - 或者如果尚不存在，则创建它：

        ```js
        {env: {...process.env, MY_VAR: 'Hi!'}}
        ```

+   `.signal: AbortSignal`

    如果我们创建了一个 AbortController `ac`，我们可以将`ac.signal`传递给`spawn()`，并通过`ac.abort()`中止子进程。这在本章后面有演示。

+   `.timeout: number`

    如果子进程的执行时间超过`.timeout`毫秒，则会被终止。

##### 12.2.1.3 `options.stdio`

子进程的每个标准 I/O 流都有一个数字 ID，称为*文件描述符*：

+   标准输入（stdin）的文件描述符为 0。

+   标准输出(stdout)的文件描述符为 1。

+   标准错误(stderr)的文件描述符为 2。

可能会有更多的文件描述符，但这很少见。

`options.stdio`配置子进程的流是否以及如何被管道连接到父进程的流。它可以是一个数组，其中每个元素配置等于其索引的文件描述符。可以使用以下值作为数组元素：

+   `'pipe'`：

    +   索引 0：将`childProcess.stdin`管道连接到子进程的 stdin。请注意，尽管其名称如此，但前者是属于父进程的流。

    +   索引 1：将子进程的 stdout 管道连接到`childProcess.stdout`。

    +   索引 2：将子进程的 stderr 管道连接到`childProcess.stderr`。

+   `'ignore'`：忽略子进程的流。

+   `'inherit'`：将子进程的流管道连接到父进程的相应流。

    +   例如，如果我们希望子进程的 stderr 被记录到控制台，我们可以在索引 2 处使用`'inherit'`。

+   原生 Node.js 流：管道到该流或从该流。

+   还支持其他值，但这超出了本章的范围。

除了通过数组指定`options.stdio`之外，我们还可以缩写：

+   `'pipe'`等同于`['pipe', 'pipe', 'pipe']`（`options.stdio`的默认值）。

+   `'ignore'`等同于`['ignore', 'ignore', 'ignore']`。

+   `'inherit'`等同于`['inherit', 'inherit', 'inherit']`。

##### 12.2.1.4 结果：`ChildProcess`的实例

`spawn()`返回[`ChildProcess`](https://nodejs.org/api/child_process.html#class-childprocess)的实例。

有趣的数据属性：

+   `.exitCode: number | null`

    包含子进程退出时的代码：

    +   0（零）表示正常退出。

    +   大于零的数字表示发生了错误。

    +   `null`表示进程尚未退出。

+   `.signalCode: string | null`

    子进程被杀死的 POSIX 信号，或者如果没有被杀死则为`null`。有关更多信息，请参阅下面的`.kill()`方法的描述。

+   流：根据标准 I/O 的配置方式（请参阅前面的小节），以下流变得可用：

    +   `.stdin`

    +   `.stdout`

    +   `.stderr`

+   `.pid: number | undefined`

    子进程的*进程标识符*（PID）。如果生成失败，`.pid`为`undefined`。在调用`spawn()`后立即可用此值。

有趣的方法：

+   `.kill(signalCode?: number | string = 'SIGTERM'): boolean`

    向子进程发送 POSIX 信号（通常导致进程终止）：

    +   [`signal`的 man 页面](https://man7.org/linux/man-pages/man7/signal.7.html)包含值的列表。

    +   Windows 不支持信号，但 Node.js 模拟了其中一些 - 例如：`SIGINT`，`SIGTERM`和`SIGKILL`。有关更多信息，请参阅[Node.js 文档](https://nodejs.org/api/process.html#signal-events)。

    此方法在本章后面进行了演示。

有趣的事件：

+   `.on('exit', (exitCode: number|null, signalCode: string|null) => {})`

    此事件在子进程结束后发出：

    +   回调参数为我们提供了退出代码或信号代码：其中一个始终为非空。

    +   由于多个进程可能共享相同的流，因此其标准 I/O 流可能仍然打开。事件`'close'`在子进程退出后通知我们所有 stdio 流都已关闭。

+   `.on('error', (err: Error) => {})`

    如果进程无法被生成（请参阅示例后面）或子进程无法被杀死，则最常见地发出此事件。在此事件之后可能会或可能不会发出`'exit'`事件。

我们稍后将看到如何将事件转换为可以等待的 Promise。

#### 12.2.2 shell 命令何时执行？

在使用异步`spawn()`时，命令的子进程是异步启动的。以下代码演示了这一点：

```js
import {spawn} from 'node:child_process';

spawn(
 'echo', ['Command starts'],
 {
 stdio: 'inherit',
 shell: true,
 }
);
console.log('After spawn()');
```

这是输出：

```js
After spawn()
Command starts
```

#### 12.2.3 仅命令模式 vs. 参数模式

在本节中，我们以两种方式指定相同的命令调用：

+   仅命令模式：我们通过第一个参数`command`提供整个调用。

+   参数模式：我们通过第一个参数`command`提供命令，通过第二个参数`args`提供参数。

##### 12.2.3.1 仅命令模式

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 'echo "Hello, how are you?"',
 {
 shell: true, // (A)
 stdio: ['ignore', 'pipe', 'inherit'], // (B)
 }
);
const stdout = Readable.toWeb(
 childProcess.stdout.setEncoding('utf-8'));

// Result on Unix
assert.equal(
 await readableStreamToString(stdout),
 'Hello, how are you?\n' // (C)
);

// Result on Windows: '"Hello, how are you?"\r\n'
```

每个带参数的仅命令生成都需要`.shell`为`true`（A 行）-即使它像这个这么简单。

在 B 行，我们告诉`spawn()`如何处理标准 I/O：

+   忽略标准输入。

+   将子进程的标准输出管道到`childProcess.stdout`（属于父进程的流）。

+   将子进程的标准错误输出管道到父进程的标准错误输出。

在这种情况下，我们只对子进程的输出感兴趣。因此，一旦我们处理了输出，我们就完成了。在其他情况下，我们可能需要等到子进程退出。如何做到这一点，稍后会有演示。

在仅命令模式下，我们看到 shell 的更多特殊之处 - 例如，Windows 命令 shell 输出包括双引号（最后一行）。

##### 12.2.3.2 参数模式

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 'echo', ['Hello, how are you?'],
 {
 shell: true,
 stdio: ['ignore', 'pipe', 'inherit'],
 }
);
const stdout = Readable.toWeb(
 childProcess.stdout.setEncoding('utf-8'));

// Result on Unix
assert.equal(
 await readableStreamToString(stdout),
 'Hello, how are you?\n'
);
// Result on Windows: 'Hello, how are you?\r\n'
```

##### 12.2.3.3 `args`中的元字符

让我们探讨一下如果`args`中有元字符会发生什么：

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';

async function echoUser({shell, args}) {
 const childProcess = spawn(
 `echo`, args,
 {
 stdio: ['ignore', 'pipe', 'inherit'],
 shell,
 }
 );
 const stdout = Readable.toWeb(
 childProcess.stdout.setEncoding('utf-8'));
 return readableStreamToString(stdout);
}

// Results on Unix
assert.equal(
 await echoUser({shell: false, args: ['$USER']}), // (A)
 '$USER\n'
);
assert.equal(
 await echoUser({shell: true, args: ['$USER']}), // (B)
 'rauschma\n'
);
assert.equal(
 await echoUser({shell: true, args: [String.raw`\$USER`]}), // (C)
 '$USER\n'
);
```

+   如果我们不使用 shell，例如美元符号（`$`）等元字符没有效果（A 行）。

+   在 shell 中，`$USER`被解释为一个变量（B 行）。

+   如果我们不想要这个，我们必须通过反斜杠转义美元符号（C 行）。

其他元字符（如星号（`*`））也会产生类似的效果。

这是 Unix shell 元字符的两个例子。Windows shell 有它们自己的元字符和它们自己的转义方式。

##### 12.2.3.4 一个更复杂的 shell 命令

让我们使用更多的 shell 特性（这需要仅命令模式）：

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';
import {EOL} from 'node:os';

const childProcess = spawn(
 `(echo cherry && echo apple && echo banana) | sort`,
 {
 stdio: ['ignore', 'pipe', 'inherit'],
 shell: true,
 }
);
const stdout = Readable.toWeb(
 childProcess.stdout.setEncoding('utf-8'));
assert.equal(
 await readableStreamToString(stdout),
 'apple\nbanana\ncherry\n'
);
```

#### 12.2.4 将数据发送到子进程的标准输入

到目前为止，我们只读取了子进程的标准输出。但是我们也可以将数据发送到标准输入：

```js
import {Readable, Writable} from 'node:stream';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 `sort`, // (A)
 {
 stdio: ['pipe', 'pipe', 'inherit'],
 }
);
const stdin = Writable.toWeb(childProcess.stdin); // (B)
const writer = stdin.getWriter(); // (C)
try {
 await writer.write('Cherry\n');
 await writer.write('Apple\n');
 await writer.write('Banana\n');
} finally {
 writer.close();
}

const stdout = Readable.toWeb(
 childProcess.stdout.setEncoding('utf-8'));
assert.equal(
 await readableStreamToString(stdout),
 'Apple\nBanana\nCherry\n'
);
```

我们使用 shell 命令`sort`（A 行）来为我们对文本行进行排序。

在 B 行，我们使用`Writable.toWeb()`将本机 Node.js 流转换为网络流（更多信息，请参见§10“在 Node.js 上使用网络流”）。

如何通过写入器（C 行）向 WritableStream 写入也在网络流章节中有解释。

#### 12.2.5 手动进行管道传输

我们之前让 shell 执行以下命令：

```js
(echo cherry && echo apple && echo banana) | sort
```

在下面的例子中，我们手动进行管道传输，从 echo（A 行）到 sorting（B 行）：

```js
import {Readable, Writable} from 'node:stream';
import {spawn} from 'node:child_process';

const echo = spawn( // (A)
 `echo cherry && echo apple && echo banana`,
 {
 stdio: ['ignore', 'pipe', 'inherit'],
 shell: true,
 }
);
const sort = spawn( // (B)
 `sort`,
 {
 stdio: ['pipe', 'pipe', 'inherit'],
 shell: true,
 }
);

//==== Transferring chunks from echo.stdout to sort.stdin ====

const echoOut = Readable.toWeb(
 echo.stdout.setEncoding('utf-8'));
const sortIn = Writable.toWeb(sort.stdin);

const sortInWriter = sortIn.getWriter();
try {
 for await (const chunk of echoOut) { // (C)
 await sortInWriter.write(chunk);
 }
} finally {
 sortInWriter.close();
}

//==== Reading sort.stdout ====

const sortOut = Readable.toWeb(
 sort.stdout.setEncoding('utf-8'));
assert.equal(
 await readableStreamToString(sortOut),
 'apple\nbanana\ncherry\n'
);
```

例如`echoOut`这样的 ReadableStreams 是异步可迭代的。这就是为什么我们可以使用`for-await-of`循环来读取它们的*chunks*（流数据的片段）。更多信息，请参见§10“在 Node.js 上使用网络流”。

#### 12.2.6 处理不成功的退出（包括错误）

有三种主要的不成功的退出方式：

+   子进程无法生成。

+   Shell 中发生了错误。

+   一个进程被终止。

##### 12.2.6.1 子进程无法生成

以下代码演示了如果子进程无法生成会发生什么。在这种情况下，原因是 shell 的路径没有指向可执行文件（A 行）。

```js
import {spawn} from 'node:child_process';

const childProcess = spawn(
 'echo hello',
 {
 stdio: ['inherit', 'inherit', 'pipe'],
 shell: '/bin/does-not-exist', // (A)
 }
);
childProcess.on('error', (err) => { // (B)
 assert.equal(
 err.toString(),
 'Error: spawn /bin/does-not-exist ENOENT'
 );
});
```

这是我们第一次使用事件来处理子进程。在 B 行，我们为`'error'`事件注册了一个事件监听器。当前代码片段完成后，子进程开始。这有助于防止竞争条件：当我们开始监听时，我们可以确保事件尚未被触发。

##### 12.2.6.2 Shell 中发生了错误

如果 shell 代码包含错误，我们不会收到`'error'`事件（B 行），而是会收到一个带有非零退出代码的`'exit'`事件（A 行）：

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 'does-not-exist',
 {
 stdio: ['inherit', 'inherit', 'pipe'],
 shell: true,
 }
);
childProcess.on('exit',
 async (exitCode, signalCode) => { // (A)
 assert.equal(exitCode, 127);
 assert.equal(signalCode, null);
 const stderr = Readable.toWeb(
 childProcess.stderr.setEncoding('utf-8'));
 assert.equal(
 await readableStreamToString(stderr),
 '/bin/sh: does-not-exist: command not found\n'
 );
 }
);
childProcess.on('error', (err) => { // (B)
 console.error('We never get here!');
});
```

##### 12.2.6.3 进程被终止

如果在 Unix 上终止进程，退出代码是`null`（C 行），信号代码是一个字符串（D 行）：

```js
import {Readable} from 'node:stream';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 'kill $$', // (A)
 {
 stdio: ['inherit', 'inherit', 'pipe'],
 shell: true,
 }
);
console.log(childProcess.pid); // (B)
childProcess.on('exit', async (exitCode, signalCode) => {
 assert.equal(exitCode, null); // (C)
 assert.equal(signalCode, 'SIGTERM'); // (D)
 const stderr = Readable.toWeb(
 childProcess.stderr.setEncoding('utf-8'));
 assert.equal(
 await readableStreamToString(stderr),
 '' // (E)
 );
});
```

请注意，没有错误输出（E 行）。

子进程不是自己终止（A 行），我们也可以暂停它更长时间，然后通过我们在 B 行记录的进程 ID 手动终止它。

如果我们在 Windows 上杀死一个子进程会发生什么？

+   `exitCode` 是 `1`。

+   `signalCode` 是 `null`。

#### 12.2.7 等待子进程退出

有时我们只想等到命令执行完毕。这可以通过事件和 Promise 来实现。

##### 12.2.7.1 通过事件等待

```js
import * as fs from 'node:fs';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 `(echo first && echo second) > tmp-file.txt`,
 {
 shell: true,
 stdio: 'inherit',
 }
);
childProcess.on('exit', (exitCode, signalCode) => { // (A)
 assert.equal(exitCode, 0);
 assert.equal(signalCode, null);
 assert.equal(
 fs.readFileSync('tmp-file.txt', {encoding: 'utf-8'}),
 'first\nsecond\n'
 );
});
```

我们使用标准的 Node.js 事件模式，并为 `'exit'` 事件注册了一个监听器（A 行）。

##### 12.2.7.2 通过 Promises 等待

```js
import * as fs from 'node:fs';
import {spawn} from 'node:child_process';

const childProcess = spawn(
 `(echo first && echo second) > tmp-file.txt`,
 {
 shell: true,
 stdio: 'inherit',
 }
);

const {exitCode, signalCode} = await onExit(childProcess); // (A)

assert.equal(exitCode, 0);
assert.equal(signalCode, null);
assert.equal(
 fs.readFileSync('tmp-file.txt', {encoding: 'utf-8'}),
 'first\nsecond\n'
);
```

我们在 A 行使用的辅助函数 `onExit()` 返回一个 Promise，如果触发了 `'exit'` 事件，它就会被满足：

```js
export function onExit(eventEmitter) {
 return new Promise((resolve, reject) => {
 eventEmitter.once('exit', (exitCode, signalCode) => {
 if (exitCode === 0) { // (B)
 resolve({exitCode, signalCode});
 } else {
 reject(new Error(
 `Non-zero exit: code ${exitCode}, signal ${signalCode}`));
 }
 });
 eventEmitter.once('error', (err) => { // (C)
 reject(err);
 });
 });
}
```

如果 `eventEmitter` 失败，返回的 Promise 被拒绝，`await` 在 A 行抛出异常。`onExit()` 处理两种失败情况：

+   `exitCode` 不是零（B 行）。发生了这种情况：

    +   如果有 shell 错误。那么 `exitCode` 大于零。

    +   如果在 Unix 上杀死子进程。那么 `exitCode` 是 `null`，`signalCode` 是非空的。

        +   在 Windows 上杀死子进程会产生一个 shell 错误。

+   一个 `'error'` 事件被触发（C 行）。如果孩子进程无法被生成，就会发生这种情况。

#### 12.2.8 终止子进程

##### 12.2.8.1 通过 AbortController 终止子进程

在这个例子中，我们使用 AbortController 来终止一个 shell 命令：

```js
import {spawn} from 'node:child_process';

const abortController = new AbortController(); // (A)

const childProcess = spawn(
 `echo Hello`,
 {
 stdio: 'inherit',
 shell: true,
 signal: abortController.signal, // (B)
 }
);
childProcess.on('error', (err) => {
 assert.equal(
 err.toString(),
 'AbortError: The operation was aborted'
 );
});
abortController.abort(); // (C)
```

我们创建一个 AbortController（A 行），将其信号传递给 `spawn()`（B 行），并通过 AbortController 终止 shell 命令（C 行）。

子进程是异步启动的（在当前代码片段执行后）。这就是为什么我们可以在进程甚至开始之前中止，以及为什么在这种情况下我们看不到任何输出。

##### 12.2.8.2 通过 `.kill()` 终止子进程

在下一个例子中，我们通过方法 `.kill()` 终止一个子进程（最后一行）：

```js
import {spawn} from 'node:child_process';

const childProcess = spawn(
 `echo Hello`,
 {
 stdio: 'inherit',
 shell: true,
 }
);
childProcess.on('exit', (exitCode, signalCode) => {
 assert.equal(exitCode, null);
 assert.equal(signalCode, 'SIGTERM');
});
childProcess.kill(); // default argument value: 'SIGTERM'
```

再次，在孩子进程开始之前我们就杀死了它（异步！），并且没有输出。

### 12.3 同步生成进程：`spawnSync()`

```js
spawnSync(
 command: string,
 args?: Array<string>,
 options?: Object
): Object
```

[`spawnSync()`](https://nodejs.org/api/child_process.html#child_processspawnsynccommand-args-options) 是 `spawn()` 的同步版本 - 它会等待子进程退出，然后同步返回一个对象。

参数大多与`spawn()`相同。`options` 有一些额外的属性 - 例如：

+   `.input: string | TypedArray | DataView`

    如果这个属性存在，它的值将被发送到子进程的标准输入。

+   `.encoding: string`（默认：`'buffer'`）

    指定用于所有标准 I/O 流的编码。

该函数返回一个对象。它最有趣的属性是：

+   `.stdout: Buffer | string`

    包含写入子进程标准输出流的内容。

+   `.stderr: Buffer | string`

    包含写入子进程标准错误流的内容。

+   `.status: number | null`

    包含子进程的退出代码或 `null`。退出代码或信号代码中的一个是非空的。

+   `.signal: string | null`

    包含孩子进程的信号代码或 `null`。退出代码或信号代码中的一个是非空的。

+   `.error?: Error`

    只有在生成失败时才会创建这个属性，然后包含一个错误对象。

使用异步的 `spawn()` 时，子进程并行运行，我们可以通过流读取标准 I/O。相反，同步的 `spawnSync()` 收集流的内容并将其同步返回给我们（见下一小节）。

#### 12.3.1 shell 命令何时执行？

使用同步的 `spawnSync()` 时，命令的子进程是同步启动的。以下代码演示了这一点：

```js
import {spawnSync} from 'node:child_process';

spawnSync(
 'echo', ['Command starts'],
 {
 stdio: 'inherit',
 shell: true,
 }
);
console.log('After spawnSync()');
```

这是输出：

```js
Command starts
After spawnSync()
```

#### 12.3.2 从标准输出读取

以下代码演示了如何读取标准输出：

```js
import {spawnSync} from 'node:child_process';

const result = spawnSync(
 `echo rock && echo paper && echo scissors`,
 {
 stdio: ['ignore', 'pipe', 'inherit'], // (A)
 encoding: 'utf-8', // (B)
 shell: true,
 }
);
console.log(result);
assert.equal(
 result.stdout, // (C)
 'rock\npaper\nscissors\n'
);
assert.equal(result.stderr, null); // (D)
```

在 A 行，我们使用 `options.stdio` 告诉 `spawnSync()` 我们只对标准输出感兴趣。我们忽略标准输入，并将标准错误传输到父进程。

因此，我们只能得到标准输出的结果属性（C 行），标准错误的属性是 `null`（D 行）。

由于我们无法访问`spawnSync()`内部使用的流来处理子进程的标准 I/O，我们通过`options.encoding`（B 行）告诉它使用哪种编码。

#### 12.3.3 向子进程的 stdin 发送数据

我们可以通过选项属性`.input`（A 行）向子进程的标准输入流发送数据：

```js
import {spawnSync} from 'node:child_process';

const result = spawnSync(
 `sort`,
 {
 stdio: ['pipe', 'pipe', 'inherit'],
 encoding: 'utf-8',
 input: 'Cherry\nApple\nBanana\n', // (A)
 }
);
assert.equal(
 result.stdout,
 'Apple\nBanana\nCherry\n'
);
```

#### 12.3.4 处理不成功的退出（包括错误）

有三种主要的不成功的退出情况（当退出代码不为零时）：

+   子进程无法被生成。

+   shell 中发生错误。

+   进程被终止。

##### 12.3.4.1 子进程无法生成

如果生成失败，`spawn()`会发出一个`'error'`事件。相比之下，`spawnSync()`将`result.error`设置为一个错误对象：

```js
import {spawnSync} from 'node:child_process';

const result = spawnSync(
 'echo hello',
 {
 stdio: ['ignore', 'inherit', 'pipe'],
 encoding: 'utf-8',
 shell: '/bin/does-not-exist',
 }
);
assert.equal(
 result.error.toString(),
 'Error: spawnSync /bin/does-not-exist ENOENT'
);
```

##### 12.3.4.2 shell 中发生错误

如果在 shell 中发生错误，退出代码`result.status`大于零，`result.signal`为`null`：

```js
import {spawnSync} from 'node:child_process';

const result = spawnSync(
 'does-not-exist',
 {
 stdio: ['ignore', 'inherit', 'pipe'],
 encoding: 'utf-8',
 shell: true,
 }
);
assert.equal(result.status, 127);
assert.equal(result.signal, null);
assert.equal(
 result.stderr, '/bin/sh: does-not-exist: command not found\n'
);
```

##### 12.3.4.3 进程被终止

如果在 Unix 上终止子进程，`result.signal`包含信号的名称，`result.status`为`null`：

```js
import {spawnSync} from 'node:child_process';

const result = spawnSync(
 'kill $$',
 {
 stdio: ['ignore', 'inherit', 'pipe'],
 encoding: 'utf-8',
 shell: true,
 }
);

assert.equal(result.status, null);
assert.equal(result.signal, 'SIGTERM');
assert.equal(result.stderr, ''); // (A)
```

请注意，没有输出发送到标准错误流（A 行）。

如果我们在 Windows 上终止一个子进程：

+   `result.status`为 1

+   `result.signal`为`null`

+   `result.stderr`为`''`

### 12.4 基于`spawn()`的异步辅助函数

在本节中，我们将看到基于`spawn()`的两个异步函数：

+   `exec()`

+   `execFile()`

在本章中，我们忽略了[`fork()`](https://nodejs.org/api/child_process.html#child_processforkmodulepath-args-options)。引用[Node.js 文档](https://nodejs.org/api/child_process.html#child-process)：

> `fork()`生成一个新的 Node.js 进程，并调用一个指定的模块，建立了一个 IPC 通信通道，允许在父进程和子进程之间发送消息。

#### 12.4.1 `exec()`

```js
exec(
 command: string,
 options?: Object,
 callback?: (error, stdout, stderr) => void
): ChildProcess
```

[`exec()`](https://nodejs.org/api/child_process.html#child_processexeccommand-options-callback)在新生成的 shell 中运行一个命令。与`spawn()`的主要区别在于：

+   除了返回一个 ChildProcess，`exec()`还通过回调函数传递结果：错误对象或 stdout 和 stderr 的内容。

+   错误原因：子进程无法生成，shell 错误，子进程被终止。

    +   相比之下，`spawn()`只在子进程无法被生成时发出`'error'`事件。另外两种失败是通过退出代码和（在 Unix 上）信号代码来处理的。

+   没有参数`args`。

+   `options.shell`的默认值为`true`。

```js
import {exec} from 'node:child_process';

const childProcess = exec(
 'echo Hello',
 (error, stdout, stderr) => {
 if (error) {
 console.error('error: ' + error.toString());
 return;
 }
 console.log('stdout: ' + stdout); // 'stdout: Hello\n'
 console.error('stderr: ' + stderr); // 'stderr: '
 }
);
```

`exec()`可以通过[`util.promisify()`](https://nodejs.org/api/util.html#utilpromisifyoriginal)转换为基于 Promise 的函数：

+   ChildProcess 成为返回的 Promise 的属性。

+   Promise 的解决方式如下：

    +   完成值：`{stdout, stderr}`

    +   拒绝值：与回调函数的参数`error`相同，但有两个额外的属性：`.stdout`和`.stderr`。

```js
import * as util from 'node:util';
import * as child_process from 'node:child_process';

const execAsync = util.promisify(child_process.exec);

try {
 const resultPromise = execAsync('echo Hello');
 const {childProcess} = resultPromise;
 const obj = await resultPromise;
 console.log(obj); // { stdout: 'Hello\n', stderr: '' }
} catch (err) {
 console.error(err);
}
```

#### 12.4.2 `execFile()`

[`execFile(file, args?, options?, callback?): ChildProcess`](https://nodejs.org/api/child_process.html#child_processexecfilefile-args-options-callback)

与`exec()`类似，具有以下区别：

+   支持参数`args`。

+   `options.shell`的默认值为`false`。

与`exec()`类似，`execFile()`可以通过`util.promisify()`转换为基于 Promise 的函数。

### 12.5 基于`spawnAsync()`的同步辅助函数

#### 12.5.1 `execSync()`

```js
execSync(
 command: string,
 options?: Object
): Buffer | string
```

[`execSync()`](https://nodejs.org/api/child_process.html#child_processexecsynccommand-options)在一个新的子进程中运行一个命令，并同步等待该进程退出。与`spawnSync()`的主要区别在于：

+   只返回 stdout 的内容。

+   三种失败通过异常报告：子进程无法生成，shell 错误，子进程被终止。

    +   相比之下，`spawnSync()`的结果只有一个`.error`属性，如果子进程无法被生成。另外两种失败是通过退出代码和（在 Unix 上）信号代码来处理的。

+   没有参数`args`。

+   `options.shell`的默认值为`true`。

```js
import {execSync} from 'node:child_process';

try {
 const stdout = execSync('echo Hello');
 console.log('stdout: ' + stdout); // 'stdout: Hello\n'
} catch (err) {
 console.error('Error: ' + err.toString());
}
```

#### 12.5.2 `execFileSync()`

[`execFileSync(file, args?, options?): Buffer | string`](https://nodejs.org/api/child_process.html#child_processexecfilesyncfile-args-options)

与`execSync()`类似，但有以下区别：

+   支持参数`args`。

+   `options.shell`的默认值是`false`。

### 12.6 有用的库

#### 12.6.1 tinysh：生成 shell 命令的辅助程序

[tinysh](https://github.com/antonmedv/tinysh)由 Anton Medvedev 是一个帮助生成 shell 命令的小型库-例如：

```js
import sh from 'tinysh';

console.log(sh.ls('-l'));
console.log(sh.cat('README.md'));
```

我们可以通过使用`.call()`将对象作为`this`传递来覆盖默认选项：

```js
sh.tee.call({input: 'Hello, world!'}, 'file.txt');
```

我们可以使用任何属性名称，tinysh 会使用该名称执行 shell 命令。它通过[代理](https://exploringjs.com/deep-js/ch_proxies.html)实现了这一壮举。这是实际库的略微修改版本：

```js
import {execFileSync} from 'node:child_process';
const sh = new Proxy({}, {
 get: (_, bin) => function (...args) { // (A)
 return execFileSync(bin, args,
 {
 encoding: 'utf-8',
 shell: true,
 ...this // (B)
 }
 );
 },
});
```

在 A 行中，我们可以看到如果从`sh`获取名为`bin`的属性，则返回一个调用`execFileSync()`并使用`bin`作为第一个参数的函数。

在 B 行中传播`this`使我们能够通过`.call()`指定选项。默认值首先出现，以便可以通过`this`进行覆盖。

#### 12.6.2 node-powershell：通过 Node.js 执行 Windows PowerShell 命令

在 Windows 上使用[node-powershell 库](https://github.com/rannn505/child-shell/tree/master/packages/node-powershell)的示例如下：

```js
import { PowerShell } from 'node-powershell';
PowerShell.$`echo "hello from PowerShell"`;
```

### 12.7 在模块`'node:child_process'`的函数之间进行选择

一般约束：

+   在执行命令时，其他异步任务是否应该运行？

    +   使用任何异步函数。

+   您是否只执行一个命令（没有后台异步任务）？

    +   使用任何同步函数。

+   您想通过流访问子进程的 stdin 或 stdout 吗？

    +   只有异步函数才能让您访问流：在这种情况下，`spawn()`更简单，因为它没有提供传递错误和标准 I/O 内容的回调。

+   您想在字符串中捕获 stdout 或 stderr 吗？

    +   异步选项：`exec()`和`execFile()`

    +   同步选项：`spawnSync()`，`execSync()`，`execFileSync()`

异步函数-在`spawn()`和`exec()`或`execFile()`之间进行选择：

+   `exec()`和`execFile()`有两个好处：

    +   由于它们都通过第一个回调参数报告，因此更容易处理失败。

    +   获取 stdout 和 stderr 作为字符串更容易-由于回调。

+   如果这些好处对您不重要，您可以选择`spawn()`。它的签名更简单，没有（可选的）回调。

同步函数-在`spawnSync()`和`execSync()`或`execFileSync()`之间进行选择：

+   `execSync()`和`execFileSync()`有两个特点：

    +   它们返回一个包含 stdout 内容的字符串。

    +   由于它们都通过异常报告，因此更容易处理失败。

+   如果您需要比`execSync()`和`execFileSync()`通过它们的返回值和异常提供的更多信息，则选择`spawnSync()`。

在`exec()`和`execFile()`之间进行选择（选择`execSync()`和`execFileSync()`时适用相同的参数）：

+   `options.shell`在`exec()`中的默认值为`true`，但在`execFile()`中为`false`。

+   `execFile()`支持`args`，`exec()`不支持。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/12)
