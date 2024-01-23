# 八、在 Node.js 上处理文件系统

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_nodejs-file-system.html`](https://exploringjs.com/nodejs-shell-scripting/ch_nodejs-file-system.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   8.1 Node 文件系统 API 的概念、模式和约定

    +   8.1.1 访问文件的方式

    +   8.1.2 函数名称前缀

    +   8.1.3 重要类

+   8.2 读写文件

    +   8.2.1 同步将文件读入单个字符串（可选：拆分为行）

    +   8.2.2 通过流逐行读取文件

    +   8.2.3 同步将单个字符串写入文件

    +   8.2.4 将单个字符串追加到文件（同步）

    +   8.2.5 通过流将多个字符串写入文件

    +   8.2.6 通过流异步追加多个字符串到文件

+   8.3 跨平台处理行终止符

    +   8.3.1 读取行终止符

    +   8.3.2 写入行终止符

+   8.4 遍历和创建目录

    +   8.4.1 遍历目录

    +   8.4.2 创建目录（`mkdir`，`mkdir -p`）

    +   8.4.3 确保父目录存在

    +   8.4.4 创建临时目录

+   8.5 复制、重命名、移动文件或目录

    +   8.5.1 复制文件或目录

    +   8.5.2 重命名或移动文件或目录

+   8.6 删除文件或目录

    +   8.6.1 删除文件和任意目录（shell：`rm`，`rm -r`）

    +   8.6.2 删除空目录（shell：`rmdir`）

    +   8.6.3 清空目录

    +   8.6.4 删除文件或目录

+   8.7 读取和更改文件系统条目

    +   8.7.1 检查文件或目录是否存在

    +   8.7.2 检查文件的统计信息：它是目录吗？它是何时创建的？等等。

    +   8.7.3 更改文件属性：权限、所有者、组、时间戳

+   8.8 处理链接

+   8.9 进一步阅读

* * *

本章包括：

+   Node 的文件系统 API 的不同部分概述。

+   *Recipes*（代码片段）用于通过这些 API 执行各种任务。

鉴于本书的重点是 shell 脚本，我们只处理文本数据。

### 8.1 Node 的文件系统 API 的概念、模式和约定

#### 8.1.1 访问文件的方式

1.  我们可以通过字符串读取或写入文件的整个内容。

1.  我们可以打开一个用于读取或写入的流，并逐个处理文件的较小部分。流只允许顺序访问。

1.  我们可以使用文件描述符或 FileHandles，并获得顺序和随机访问，通过一个与流松散相似的 API。

    +   [*文件描述符*](https://nodejs.org/api/fs.html#file-descriptors_1)是表示文件的整数。它们通过这些函数管理（只显示同步名称，还有基于回调的版本- `fs.open()` 等）：

        +   `fs.openSync(path, flags?, mode?)` 打开给定路径上的文件的新文件描述符并返回它。

        +   `fs.closeSync(fd)` 关闭文件描述符。

        +   `fs.fchmodSync(fd, mode)`

        +   `fs.fchownSync(fd, uid, gid)`

        +   `fs.fdatasyncSync(fd)`

        +   `fs.fstatSync(fd, options?)`

        +   `fs.fsyncSync(fd)`

        +   `fs.ftruncateSync(fd, len?)`

        +   `fs.futimesSync(fd, atime, mtime)`

    +   只有同步 API 和基于回调的 API 使用文件描述符。基于 Promise 的 API 有更好的抽象，[class `FileHandle`](https://nodejs.org/api/fs.html#class-filehandle)，它基于文件描述符。实例是通过 `fsPromises.open()` 创建的。各种操作通过方法提供（而不是通过函数）：

        +   `fileHandle.close()`

        +   `fileHandle.chmod(mode)`

        +   `fileHandle.chown(uid, gid)`

        +   等等。

请注意，在本章中我们不使用（3）-（1）和（2）对我们的目的足够了。

#### 8.1.2 函数名前缀

##### 8.1.2.1 前缀“l”：符号链接

以“l”开头的函数通常操作符号链接：

+   `fs.lchmodSync()`, `fs.lchmod()`, `fsPromises.lchmod()`

+   `fs.lchownSync()`, `fs.lchown()`, `fsPromises.lchown()`

+   `fs.lutimesSync()`, `fs.lutimes()`, `fsPromises.lutimes()`

+   等等。

##### 8.1.2.2 前缀“f”：文件描述符

以“f”开头的函数通常管理文件描述符：

+   `fs.fchmodSync()`, `fs.fchmod()`

+   `fs.fchownSync()`, `fs.fchown()`

+   `fs.fstatSync()`, `fs.fstat()`

+   等等。

#### 8.1.3 重要类

几个类在 Node 的文件系统 API 中扮演重要角色。

##### 8.1.3.1 URL：字符串中文件系统路径的替代方案

每当 Node.js 函数接受一个字符串中的文件系统路径（行 A）时，它通常也接受一个 `URL` 的实例（行 B）：

```js
assert.equal(
 fs.readFileSync(
 '/tmp/text-file.txt', {encoding: 'utf-8'}), // (A)
 'Text content'
);
assert.equal(
 fs.readFileSync(
 new URL('file:///tmp/text-file.txt'), {encoding: 'utf-8'}), // (B)
 'Text content'
);
```

手动在路径和 `file:` URL 之间转换似乎很容易，但意外地有很多陷阱：百分号编码或解码，Windows 驱动器号等。因此，最好使用以下两个函数：

+   [`url.pathToFileURL()`](https://nodejs.org/api/url.html#urlpathtofileurlpath)

+   [`url.fileURLToPath()`](https://nodejs.org/api/url.html#urlfileurltopathurl)

在本章中我们不使用文件 URL。它们的用例在§7.11.1 “Class `URL`”中有描述。

##### 8.1.3.2 缓冲区

类[`Buffer`](https://nodejs.org/api/buffer.html)表示 Node.js 上的固定长度字节序列。它是 `Uint8Array` 的子类（[TypedArray](https://exploringjs.com/impatient-js/ch_typed-arrays.html)）。缓冲区在处理二进制文件时大多被使用，因此在本书中不太感兴趣。

每当 Node.js 接受一个缓冲区时，它也接受一个 Uint8Array。因此，鉴于 Uint8Arrays 是跨平台的，而 Buffers 不是，前者更可取。

缓冲区可以做 Uint8Arrays 无法做的一件事：在各种编码中编码和解码文本。如果我们需要在 Uint8Arrays 中编码或解码 UTF-8，我们可以使用类[`TextEncoder`](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder)或类[`TextDecoder`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)。这些类在大多数 JavaScript 平台上都可用：

```js
> new TextEncoder().encode('café')
Uint8Array.of(99, 97, 102, 195, 169)
> new TextDecoder().decode(Uint8Array.of(99, 97, 102, 195, 169))
'café'
```

##### 8.1.3.3 Node.js 流

一些函数接受或返回原生的 Node.js 流：

+   `stream.Readable` 是 Node 的可读流类。模块 `node:fs` 使用 `fs.ReadStream`，它是一个子类。

+   `stream.Writable` 是 Node 的可写流类。模块 `node:fs` 使用 `fs.WriteStream`，它是一个子类。

现在我们可以在 Node.js 上使用跨平台的 *web 流*，具体方法在§10 “在 Node.js 上使用 web 流”中有解释。

### 8.2 读取和写入文件

#### 8.2.1 同步读取文件为单个字符串（可选：拆分成行）

[`fs.readFileSync(filePath, options?)`](https://nodejs.org/api/fs.html#fsreadfilesyncpath-options) 将文件在 `filePath` 处同步读取为单个字符串：

```js
assert.equal(
 fs.readFileSync('text-file.txt', {encoding: 'utf-8'}),
 'there\r\nare\nmultiple\nlines'
);
```

这种方法的优缺点（与使用流相比）：

+   优点：易于使用和同步。对于许多用例来说已经足够好了。

+   缺点：不适合大文件。

    +   在我们可以处理数据之前，我们必须将其完全读取。

接下来，我们将研究如何将已读取的字符串拆分成行。

##### 8.2.1.1 不包括行终止符拆分行

以下代码将一个字符串拆分成行，同时删除行终止符。它适用于 Unix 和 Windows 行终止符：

```js
const RE_SPLIT_EOL = /\r?\n/;
function splitLines(str) {
 return str.split(RE_SPLIT_EOL);
}
assert.deepEqual(
 splitLines('there\r\nare\nmultiple\nlines'),
 ['there', 'are', 'multiple', 'lines']
);
```

“EOL”代表“行结束”。我们接受 Unix 行终止符（`'\n'`）和 Windows 行终止符（`'\r\n'`，就像前面示例中的第一个）。更多信息，请参见§8.3 “跨平台处理行终止符”。

##### 8.2.1.2 包括行终止符拆分行

以下代码将一个字符串拆分成行，同时包括行终止符。它适用于 Unix 和 Windows 行终止符（“EOL”代表“行结束”）：

```js
const RE_SPLIT_AFTER_EOL = /(?<=\r?\n)/; // (A)
function splitLinesWithEols(str) {
 return str.split(RE_SPLIT_AFTER_EOL);
}

assert.deepEqual(
 splitLinesWithEols('there\r\nare\nmultiple\nlines'),
 ['there\r\n', 'are\n', 'multiple\n', 'lines']
);
assert.deepEqual(
 splitLinesWithEols('first\n\nthird'),
 ['first\n', '\n', 'third']
);
assert.deepEqual(
 splitLinesWithEols('EOL at the end\n'),
 ['EOL at the end\n']
);
assert.deepEqual(
 splitLinesWithEols(''),
 ['']
);
```

行 A 包含一个带有[后行断言](https://exploringjs.com/impatient-js/ch_regexps.html#regexp-lookbehind-assertions)的正则表达式。它匹配前面有 `\r?\n` 模式的位置，但它不捕获任何内容。因此，它不会删除输入字符串被拆分成的字符串片段之间的任何内容。

在不支持后行断言的引擎上（[参见此表](https://caniuse.com/js-regexp-lookbehind)），我们可以使用以下解决方案：

```js
function splitLinesWithEols(str) {
 if (str.length === 0) return [''];
 const lines = [];
 let prevEnd = 0;
 while (prevEnd < str.length) {
 // Searching for '\n' means we’ll also find '\r\n'
 const newlineIndex = str.indexOf('\n', prevEnd);
 // If there is a newline, it’s included in the line
 const end = newlineIndex < 0 ? str.length : newlineIndex+1;
 lines.push(str.slice(prevEnd, end));
 prevEnd = end;
 }
 return lines;
}
```

这个解决方案很简单，但更冗长。

在 `splitLinesWithEols()` 的两个版本中，我们再次接受 Unix 行终止符（`'\n'`）和 Windows 行终止符（`'\r\n'`）。更多信息，请参见§8.3 “跨平台处理行终止符”。

#### 8.2.2 通过流逐行读取文件

我们也可以通过流读取文本文件：

```js
import {Readable} from 'node:stream';

const nodeReadable = fs.createReadStream(
 'text-file.txt', {encoding: 'utf-8'});
const webReadableStream = Readable.toWeb(nodeReadable);
const lineStream = webReadableStream.pipeThrough(
 new ChunksToLinesStream());
for await (const line of lineStream) {
 console.log(line);
}

// Output:
// 'there\r\n'
// 'are\n'
// 'multiple\n'
// 'lines'
```

我们使用了以下外部功能：

+   [`fs.createReadStream(filePath, options?)`](https://nodejs.org/api/fs.html#fscreatereadstreampath-options) 创建了一个 Node.js 流（一个 `stream.Readable` 实例）。

+   [`stream.Readable.toWeb(streamReadable)`](https://nodejs.org/api/stream.html#streamreadabletowebstreamreadable) 将一个可读的 Node.js 流转换为 web 流（一个 `ReadableStream` 实例）。

+   TransformStream 类 `ChunksToLinesStream` 在§10.7.1 “示例：将任意块的流转换为行流”中有解释。*块*是流产生的数据片段。如果我们有一个流，其块是具有任意长度的字符串，并将其通过 ChunksToLinesStream，那么我们得到的流的块就是行。

Web 流是[异步可迭代的](https://exploringjs.com/impatient-js/ch_async-iteration.html)，这就是为什么我们可以使用 `for-await-of` 循环来迭代行。

如果我们对文本行不感兴趣，那么我们不需要 `ChunksToLinesStream`，可以迭代 `webReadableStream` 并获取任意长度的块。

更多信息：

+   Web 流在§10 “在 Node.js 上使用 web 流”中有介绍。

+   行终止符在§8.3“跨平台处理行终止符”中有介绍。

这种方法的优缺点（与读取单个字符串相比）：

+   优点：对于大文件效果很好。

    +   我们可以逐步处理数据，分成较小的片段，而不必等待所有内容被读取。

+   缺点：使用起来更复杂，且不同步。

#### 8.2.3 同步地向文件写入单个字符串

[`fs.writeFileSync(filePath, str, options?)`](https://nodejs.org/api/fs.html#fswritefilesyncfile-data-options)将`str`写入到`filePath`的文件中。如果该路径下已经存在文件，则会被覆盖。

以下代码显示了如何使用此函数：

```js
fs.writeFileSync(
 'new-file.txt',
 'First line\nSecond line\n',
 {encoding: 'utf-8'}
);
```

有关行终止符的信息，请参见§8.3“跨平台处理行终止符”。

优缺点（与使用流相比）：

+   优点：易于使用，且同步。适用于许多用例。

+   缺点：不适用于大文件。

#### 8.2.4 同步地向文件追加单个字符串

以下代码将一行文本追加到现有文件中：

```js
fs.appendFileSync(
 'existing-file.txt',
 'Appended line\n',
 {encoding: 'utf-8'}
);
```

我们也可以使用`fs.writeFileSync()`来执行此任务：

```js
fs.writeFileSync(
 'existing-file.txt',
 'Appended line\n',
 {encoding: 'utf-8', flag: 'a'}
);
```

这段代码几乎与我们用来覆盖现有内容的代码相同（有关更多信息，请参见前一节）。唯一的区别是我们添加了选项`.flag`：值`'a'`表示我们追加数据。其他可能的值（例如，如果文件尚不存在则抛出错误）在[Node.js 文档](https://nodejs.org/api/fs.html#fswritefilesyncfile-data-options)中有解释。

注意：在某些函数中，此选项称为`.flag`，在其他函数中称为`.flags`。

#### 8.2.5 通过流向文件写入多个字符串

以下代码使用流向文件写入多个字符串：

```js
import {Writable} from 'node:stream';

const nodeWritable = fs.createWriteStream(
 'new-file.txt', {encoding: 'utf-8'});
const webWritableStream = Writable.toWeb(nodeWritable);

const writer = webWritableStream.getWriter();
try {
 await writer.write('First line\n');
 await writer.write('Second line\n');
 await writer.close();
} finally {
 writer.releaseLock()
}
```

我们使用了以下函数：

+   [`fs.createWriteStream(path, options?)`](https://nodejs.org/api/fs.html#fscreatewritestreampath-options)创建一个 Node.js 流（`stream.Writable`的实例）。

+   [`stream.Writable.toWeb(streamWritable)`](https://nodejs.org/api/stream.html#streamwritabletowebstreamwritable)将可写的 Node.js 流转换为 Web 流（`WritableStream`的实例）。

更多信息：

+   可写流和写入器在§10“在 Node.js 上使用 Web 流”中有介绍。

+   行终止符在§8.3“跨平台处理行终止符”中有介绍。

优缺点（与写入单个字符串相比）：

+   优点：对于大文件效果很好，因为我们可以逐步写入数据，分成较小的片段。

+   缺点：使用起来更复杂，且不同步。

#### 8.2.6 通过流（异步地）向文件追加多个字符串

以下代码使用流向现有文件追加文本：

```js
import {Writable} from 'node:stream';

const nodeWritable = fs.createWriteStream(
 'existing-file.txt', {encoding: 'utf-8', flags: 'a'});
const webWritableStream = Writable.toWeb(nodeWritable);

const writer = webWritableStream.getWriter();
try {
 await writer.write('First appended line\n');
 await writer.write('Second appended line\n');
 await writer.close();
} finally {
 writer.releaseLock()
}
```

这段代码几乎与我们用来覆盖现有内容的代码相同（有关更多信息，请参见前一节）。唯一的区别是我们添加了选项`.flags`：值`'a'`表示我们追加数据。其他可能的值（例如，如果文件尚不存在则抛出错误）在[Node.js 文档](https://nodejs.org/api/fs.html#fswritefilesyncfile-data-options)中有解释。

注意：在某些函数中，此选项称为`.flag`，在其他函数中称为`.flags`。

### 8.3 处理跨平台的行终止符

遗憾的是，并非所有平台都具有标记*行终止符*（EOL）的相同*行终止符*字符：

+   在 Windows 上，EOL 是`'\r\n'`。

+   在 Unix（包括 macOS）上，EOL 是`'\n'`。

为了以适用于所有平台的方式处理 EOL，我们可以使用几种策略。

#### 8.3.1 读取行终止符

在读取文本时，最好能够识别两种 EOL。

当将文本拆分成行时，可能会是什么样子？我们可以在行尾包含 EOL（以任何格式）。这样，如果我们修改这些行并将其写入文件，我们可以尽可能少地进行更改。

在处理带有 EOL 的行时，有时将它们移除是有用的，例如通过以下函数：

```js
const RE_EOL_REMOVE = /\r?\n$/;
function removeEol(line) {
 const match = RE_EOL_REMOVE.exec(line);
 if (!match) return line;
 return line.slice(0, match.index);
}

assert.equal(
 removeEol('Windows EOL\r\n'),
 'Windows EOL'
);
assert.equal(
 removeEol('Unix EOL\n'),
 'Unix EOL'
);
assert.equal(
 removeEol('No EOL'),
 'No EOL'
);
```

#### 8.3.2 写入行终止符

在写入行终止符时，我们有两个选项：

+   模块`'node:os'`中的常量`EOL`（https://nodejs.org/api/os.html#oseol）包含当前平台的 EOL。

+   我们可以检测输入文件的 EOL 格式，并在更改该文件时使用它。

### 8.4 遍历和创建目录

#### 8.4.1 遍历目录

以下函数遍历目录并列出其所有后代（其子目录、其子目录的子目录等）：

```js
import * as path from 'node:path';

function* traverseDirectory(dirPath) {
 const dirEntries = fs.readdirSync(dirPath, {withFileTypes: true});
 // Sort the entries to keep things more deterministic
 dirEntries.sort(
 (a, b) => a.name.localeCompare(b.name, 'en')
 );
 for (const dirEntry of dirEntries) {
 const fileName = dirEntry.name;
 const pathName = path.join(dirPath, fileName);
 yield pathName;
 if (dirEntry.isDirectory()) {
 yield* traverseDirectory(pathName);
 }
 }
}
```

我们使用了这个功能：

+   [`fs.readdirSync(thePath, options?)`](https://nodejs.org/api/fs.html#fsreaddirsyncpath-options)返回`thePath`处目录的子目录。

    +   如果选项`.withFileTypes`是`true`，函数返回*directory entries*，即[`fs.Dirent`](https://nodejs.org/api/fs.html#class-fsdirent)的实例。这些具有属性，如：

        +   `dirent.name`

        +   `dirent.isDirectory()`

        +   `dirent.isFile()`

        +   `dirent.isSymbolicLink()`

    +   如果选项`.withFileTypes`是`false`或缺失，函数返回文件名的字符串。

以下代码展示了`traverseDirectory()`的操作：

```js
for (const filePath of traverseDirectory('dir')) {
 console.log(filePath);
}

// Output:
// 'dir/dir-file.txt'
// 'dir/subdir'
// 'dir/subdir/subdir-file1.txt'
// 'dir/subdir/subdir-file2.csv'
```

#### 8.4.2 创建目录（`mkdir`, `mkdir -p`）

我们可以使用[以下函数](https://nodejs.org/api/fs.html#fsmkdirsyncpath-options)来创建目录：

```js
fs.mkdirSync(thePath, options?): undefined | string
```

`options.recursive`决定函数如何创建`thePath`处的目录：

+   如果`.recursive`缺失或为`false`，`mkdirSync()`返回`undefined`，并且如果：

    +   `thePath`处已经存在一个目录（或文件）。

    +   `thePath`的父目录不存在。

+   如果`.recursive`是`true`：

    +   如果`thePath`处已经有一个目录，那没关系。

    +   `thePath`的祖先目录将根据需要创建。

    +   `mkdirSync()`返回第一个新创建目录的路径。

这是`mkdirSync()`的操作：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
fs.mkdirSync('dir/sub/subsub', {recursive: true});
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/sub',
 'dir/sub/subsub',
 ]
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`处目录的所有后代。

#### 8.4.3 确保父目录存在

如果我们想要根据需要设置嵌套文件结构，我们不能总是确定在创建新文件时祖先目录是否存在。这时以下函数会有所帮助：

```js
import * as path from 'node:path';

function ensureParentDirectory(filePath) {
 const parentDir = path.dirname(filePath);
 if (!fs.existsSync(parentDir)) {
 fs.mkdirSync(parentDir, {recursive: true});
 }
}
```

这里我们可以看到`ensureParentDirectory()`的操作（A 行）：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
const filePath = 'dir/sub/subsub/new-file.txt';
ensureParentDirectory(filePath); // (A)
fs.writeFileSync(filePath, 'content', {encoding: 'utf-8'});
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/sub',
 'dir/sub/subsub',
 'dir/sub/subsub/new-file.txt',
 ]
);
```

#### 8.4.4 创建临时目录

[`fs.mkdtempSync(pathPrefix, options?)`](https://nodejs.org/api/fs.html#fsmkdtempsyncprefix-options)创建一个临时目录：它在`pathPrefix`后附加 6 个随机字符，创建一个新路径的目录并返回该路径。

`pathPrefix`不应以大写的“X”结尾，因为一些平台会用随机字符替换尾随的 X。

如果我们想要在操作系统特定的全局临时目录中创建临时目录，我们可以使用[函数`os.tmpdir()`](https://nodejs.org/api/os.html#ostmpdir)：

```js
import * as os from 'node:os';
import * as path from 'node:path';

const pathPrefix = path.resolve(os.tmpdir(), 'my-app');
 // e.g. '/var/folders/ph/sz0384m11vxf/T/my-app'

const tmpPath = fs.mkdtempSync(pathPrefix);
 // e.g. '/var/folders/ph/sz0384m11vxf/T/my-app1QXOXP'
```

重要的是要注意，当 Node.js 脚本终止时，临时目录不会自动删除。我们要么自己删除它，要么依赖操作系统定期清理其全局临时目录（可能会或可能不会这样做）。

### 8.5 复制、重命名、移动文件或目录

#### 8.5.1 复制文件或目录

[`fs.cpSync(srcPath, destPath, options?)`](https://nodejs.org/api/fs.html#fscpsyncsrc-dest-options)：从`srcPath`复制文件或目录到`destPath`。有趣的选项：

+   `.recursive`（默认：`false`）：只有在此选项为`true`时，目录（包括空目录）才会被复制。

+   `.force`（默认：`true`）：如果为`true`，则覆盖现有文件。如果为`false`，则保留现有文件。

    +   在后一种情况下，将`.errorOnExist`设置为`true`会导致如果文件路径冲突则抛出错误。

+   `.filter`是一个函数，让我们控制哪些文件被复制。

+   `.preserveTimestamps`（默认：`false`）：如果为`true`，`destPath`中的复制品将获得与`srcPath`中原始文件相同的时间戳。

这是函数的操作：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir-orig',
 'dir-orig/some-file.txt',
 ]
);
fs.cpSync('dir-orig', 'dir-copy', {recursive: true});
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir-copy',
 'dir-copy/some-file.txt',
 'dir-orig',
 'dir-orig/some-file.txt',
 ]
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`目录中所有后代。

#### 8.5.2 重命名或移动文件或目录

[`fs.renameSync(oldPath, newPath)`](https://nodejs.org/api/fs.html#fsrenamesyncoldpath-newpath)将文件或目录从`oldPath`重命名或移动到`newPath`。

让我们使用这个函数来重命名一个目录：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'old-dir-name',
 'old-dir-name/some-file.txt',
 ]
);
fs.renameSync('old-dir-name', 'new-dir-name');
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'new-dir-name',
 'new-dir-name/some-file.txt',
 ]
);
```

在这里，我们使用该函数来移动一个文件：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/subdir',
 'dir/subdir/some-file.txt',
 ]
);
fs.renameSync('dir/subdir/some-file.txt', 'some-file.txt');
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/subdir',
 'some-file.txt',
 ]
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`目录中所有后代。

### 8.6 删除文件或目录

#### 8.6.1 删除文件和任意目录（shell：`rm`，`rm -r`）

[`fs.rmSync(thePath, options?)`](https://nodejs.org/api/fs.html#fsrmsyncpath-options)删除`thePath`上的文件或目录。有趣的选项：

+   `.recursive`（默认：`false`）：只有在此选项为`true`时，才会删除目录（包括空目录）。

+   `.force`（默认：`false`）：如果为`false`，则如果`thePath`上没有文件或目录，将抛出异常。

让我们使用`fs.rmSync()`来删除一个文件：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/some-file.txt',
 ]
);
fs.rmSync('dir/some-file.txt');
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
```

在这里，我们使用`fs.rmSync()`递归地删除非空目录。

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/subdir',
 'dir/subdir/some-file.txt',
 ]
);
fs.rmSync('dir/subdir', {recursive: true});
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`目录中所有后代。

#### 8.6.2 删除空目录（shell：`rmdir`）

[`fs.rmdirSync(thePath, options?)`](https://nodejs.org/api/fs.html#fsrmdirsyncpath-options)删除空目录（如果目录不为空，则会抛出异常）。

以下代码显示了这个函数的工作原理：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/subdir',
 ]
);
fs.rmdirSync('dir/subdir');
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`目录中所有后代。

#### 8.6.3 清除目录

将其输出保存到目录`dir`的脚本通常需要在开始之前*清除*`dir`：删除`dir`中的每个文件，使其为空。以下函数可以实现这一点。

```js
import * as path from 'node:path';

function clearDirectory(dirPath) {
 for (const fileName of fs.readdirSync(dirPath)) {
 const pathName = path.join(dirPath, fileName);
 fs.rmSync(pathName, {recursive: true});
 }
}
```

我们使用了两个文件系统函数：

+   `fs.readdirSync(dirPath)`返回`dirPath`目录中所有子项的名称。在§8.4.1“遍历目录”中有解释。

+   `fs.rmSync(pathName, options?)`删除文件和目录（包括非空目录）。在§8.6.1“删除文件和任意目录（shell：`rm`，`rm -r`）”中有解释。

这是使用`clearDirectory()`的一个例子：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/dir-file.txt',
 'dir/subdir',
 'dir/subdir/subdir-file.txt'
 ]
);
clearDirectory('dir');
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 ]
);
```

#### 8.6.4 将文件或目录移到垃圾箱

[库`trash`](https://github.com/sindresorhus/trash)将文件和文件夹移动到垃圾箱。它适用于 macOS，Windows 和 Linux（在 Linux 上支持有限，需要帮助）。这是它自述文件中的一个例子：

```js
import trash from 'trash';

await trash(['*.png', '!rainbow.png']);
```

`trash()`接受字符串数组或字符串作为其第一个参数。任何字符串都可以是 glob 模式（带有星号和其他元字符）。

### 8.7 读取和更改文件系统条目

#### 8.7.1 检查文件或目录是否存在

[`fs.existsSync(thePath)`](https://nodejs.org/api/fs.html#fsexistssyncpath)如果`thePath`上存在文件或目录，则返回`true`：

```js
assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/some-file.txt',
 ]
);
assert.equal(
 fs.existsSync('dir'), true
);
assert.equal(
 fs.existsSync('dir/some-file.txt'), true
);
assert.equal(
 fs.existsSync('dir/non-existent-file.txt'), false
);
```

函数`traverseDirectory(dirPath)`列出`dirPath`目录中所有后代。

#### 8.7.2 检查文件的统计信息：它是一个目录吗？它是什么时候创建的？等等。

[`fs.statSync(thePath, options?)`](https://nodejs.org/api/fs.html#fsstatsyncpath-options)返回一个`fs.Stats`实例，其中包含有关`thePath`上的文件或目录的信息。

有趣的`options`：

+   `.throwIfNoEntry`（默认：`true`）：如果在`path`上没有实体会发生什么？

    +   如果此选项为`true`，则会抛出异常。

    +   如果为`false`，则返回`undefined`。

+   `.bigint`（默认：`false`）：如果为`true`，则此函数将使用 bigints 作为数值（例如时间戳，请参见下文）。

[`fs.Stats`](https://nodejs.org/api/fs.html#class-fsstats)实例的属性：

+   它是什么类型的文件系统条目？

    +   `stats.isFile()`

    +   `stats.isDirectory()`

    +   `stats.isSymbolicLink()`

+   `stats.size` 是以字节为单位的大小

+   时间戳：

    +   有三种时间戳：

        +   `stats.atime`：上次访问时间

        +   `stats.mtime`：上次修改时间

        +   `stats.birthtime`：创建时间

    +   这些时间戳中的每一个都可以用三种不同的单位指定，例如`atime`：

        +   `stats.atime`：`Date`的实例

        +   `stats.atimeMS`：自 POSIX 纪元以来的毫秒数

        +   `stats.atimeNs`：自 POSIX 纪元以来的纳秒数（需要选项`.bigint`）

在以下示例中，我们使用`fs.statSync()`来实现一个`isDirectory()`函数：

```js
function isDirectory(thePath) {
 const stats = fs.statSync(thePath, {throwIfNoEntry: false});
 return stats !== undefined && stats.isDirectory();
}

assert.deepEqual(
 Array.from(traverseDirectory('.')),
 [
 'dir',
 'dir/some-file.txt',
 ]
);

assert.equal(
 isDirectory('dir'), true
);
assert.equal(
 isDirectory('dir/some-file.txt'), false
);
assert.equal(
 isDirectory('non-existent-dir'), false
);
```

函数`traverseDirectory(dirPath)` 列出`dirPath`目录的所有后代。

#### 8.7.3 更改文件属性：权限、所有者、组、时间戳

让我们简要地看一下更改文件属性的函数：

+   [`fs.chmodSync(path, mode)`](https://nodejs.org/api/fs.html#fschmodsyncpath-mode) 改变文件的权限。

+   [`fs.chownSync(path, uid, gid)`](https://nodejs.org/api/fs.html#fschownsyncpath-uid-gid) 改变文件的所有者和组。

+   [`fs.utimesSync(path, atime, mtime)`](https://nodejs.org/api/fs.html#fsutimessyncpath-atime-mtime) 改变文件的时间戳：

    +   `atime`：上次访问时间

    +   `mtime`：上次修改时间

### 8.8 处理链接

用于处理硬链接的函数：

+   [`fs.linkSync(existingPath, newPath)`](https://nodejs.org/api/fs.html#fslinksyncexistingpath-newpath) 创建一个硬链接。

+   [`fs.unlinkSync(path)`](https://nodejs.org/api/fs.html#fsunlinksyncpath) 删除一个硬链接，可能也会删除它指向的文件（如果它是指向该文件的最后一个硬链接）。

用于处理符号链接的函数：

+   [`fs.symlinkSync(target, path, type?)`](https://nodejs.org/api/fs.html#fssymlinksynctarget-path-type) 从`path`到`target`创建一个符号链接。

+   [`fs.readlinkSync(path, options?)`](https://nodejs.org/api/fs.html#fsreadlinksyncpath-options) 返回`path`处符号链接的目标。

以下函数在不解除引用符号链接的情况下操作符号链接（注意名称前缀“l”）：

+   [`fs.lchmodSync(path, mode)`](https://nodejs.org/api/fs.html#fslchmodsyncpath-mode) 改变`path`处符号链接的权限。

+   [`fs.lchownSync(path, uid, gid)`](https://nodejs.org/api/fs.html#fslchownsyncpath-uid-gid) 改变`path`处符号链接的用户和组。

+   [`fs.lutimesSync(path, atime, mtime)`](https://nodejs.org/api/fs.html#fslutimessyncpath-atime-mtime) 改变`path`处符号链接的时间戳。

+   [`fs.lstatSync(path, options?)`](https://nodejs.org/api/fs.html#fslstatsyncpath-options) 返回`path`处符号链接的统计信息（时间戳等）。

其他有用的函数：

+   [`fs.realpathSync(path, options?)`](https://nodejs.org/api/fs.html#fsrealpathsyncpath-options) 通过解析点（`.`）、双点（`..`）和符号链接来计算规范路径名。

影响符号链接处理方式的函数选项：

+   [`fs.cpSync(src, dest, options?)`](https://nodejs.org/api/fs.html#fscpsyncsrc-dest-options)：

    +   `.dereference`（默认：`false`）：如果为`true`，则复制符号链接指向的文件，而不是符号链接本身。

    +   `.verbatimSymlinks`（默认：`false`）：如果为`false`，则复制的符号链接的目标将被更新，以便仍然指向相同的位置。如果为`true`，目标不会改变。

### 8.9 进一步阅读

+   “JavaScript 快速编程者”有几章关于编写异步代码：

    +   [“JavaScript 中的异步编程基础”](https://exploringjs.com/impatient-js/ch_async-js.html)

    +   [“用于异步编程的 Promise”](https://exploringjs.com/impatient-js/ch_promises.html)

    +   [“异步函数”](https://exploringjs.com/impatient-js/ch_async-functions.html)

    +   [“异步迭代”](https://exploringjs.com/impatient-js/ch_async-iteration.html)

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/8)
