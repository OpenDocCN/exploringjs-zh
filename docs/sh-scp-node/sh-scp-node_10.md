# 七、使用 Node.js 上的文件系统路径和文件 URL

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_nodejs-path.html`](https://exploringjs.com/nodejs-shell-scripting/ch_nodejs-path.html)

* * *

+   7.1 在 Node.js 上与路径相关的功能

    +   7.1.1 访问`'node:path'` API 的三种方式

+   7.2 基本路径概念及其 API 支持

    +   7.2.1 路径段、路径分隔符、路径分隔符

    +   7.2.2 当前工作目录

    +   7.2.3 完全 vs.部分合格路径，解析路径

+   7.3 通过模块`'node:os'`获取标准目录的路径

+   7.4 连接路径

    +   7.4.1 `path.resolve()`: 连接路径以创建完全合格的路径

    +   7.4.2 `path.join()`: 连接路径并保留相对路径

+   7.5 确保路径被规范化、完全合格或相对

    +   7.5.1 `path.normalize()`: 确保路径被规范化

    +   7.5.2 `path.resolve()`（一个参数）：确保路径被规范化和完全合格

    +   7.5.3 `path.relative()`: 创建相对路径

+   7.6 解析路径：提取路径的各个部分（文件名扩展名等）

    +   7.6.1 `path.parse()`: 创建具有路径部分的对象

    +   7.6.2 `path.basename()`: 提取路径的基本部分

    +   7.6.3 `path.dirname()`: 提取路径的父目录

    +   7.6.4 `path.extname()`: 提取路径的扩展名

+   7.7 对路径进行分类

    +   7.7.1 `path.isAbsolute()`: 给定路径是否绝对？

+   7.8 `path.format()`: 从部分创建路径

    +   7.8.1 示例：更改文件名扩展名

+   7.9 在不同平台上使用相同的路径

    +   7.9.1 相对平台无关的路径

+   7.10 使用库通过*globs*匹配路径

    +   7.10.1 minimatch API

    +   7.10.2 glob 表达式的语法

+   7.11 使用`file:` URL 引用文件

    +   7.11.1 Class `URL`

    +   7.11.2 在 URL 和文件路径之间转换

    +   7.11.3 URL 的用例：访问相对于当前模块的文件

    +   7.11.4 URL 的用例：检测当前模块是否为“main”（应用程序入口点）

    +   7.11.5 路径 vs. `file:` URL

* * *

在本章中，我们将学习如何在 Node.js 上处理文件系统路径和文件 URL。

### 7.1 在 Node.js 上与路径相关的功能

在本章中，我们将探索 Node.js 上与路径相关的功能：

+   大多数与路径相关的功能都在模块 `'node:path'` 中。

+   全局变量 `process` 有用于改变*当前工作目录*的方法（这是什么，很快就会解释）。

+   模块 `'node:os'` 有返回重要目录路径的函数。

#### 7.1.1 访问 `'node:path'` API 的三种方式

模块 `'node:path'` 经常被导入如下：

```js
import * as path from 'node:path';
```

在本章中，有时会省略此导入语句。我们还省略了以下导入：

```js
import * as assert from 'node:assert/strict';
```

我们可以通过三种方式访问 Node 的路径 API：

+   我们可以访问特定于平台的 API 版本：

    +   `path.posix` 支持包括 macOS 在内的 Unix 系统。

    +   `path.win32` 支持 Windows。

+   `path` 本身始终支持当前平台。例如，这是 macOS 上 REPL 交互的一个示例：

    ```js
    > path.parse === path.posix.parse
    true
    ```

让我们看看函数 `path.parse()` 如何在两个平台上解析文件系统路径的不同之处：

```js
> path.win32.parse(String.raw`C:\Users\jane\file.txt`)
{
 dir: 'C:\\Users\\jane',
 root: 'C:\\',
 base: 'file.txt',
 name: 'file',
 ext: '.txt',
}
> path.posix.parse(String.raw`C:\Users\jane\file.txt`)
{
 dir: '',
 root: '',
 base: 'C:\\Users\\jane\\file.txt',
 name: 'C:\\Users\\jane\\file',
 ext: '.txt',
}
```

我们解析 Windows 路径 - 首先通过 `path.win32` API 正确解析，然后通过 `path.posix` API 解析。我们可以看到在后一种情况下，路径没有正确分割为其各个部分 - 例如，文件的基本名称应该是 `file.txt`（稍后会详细介绍其他属性的含义）。

### 7.2 基本路径概念及其 API 支持

#### 7.2.1 路径段、路径分隔符、路径分隔符

术语：

+   非空路径由一个或多个*路径段*组成，通常是目录或文件的名称。

+   *路径分隔符* 用于在路径中分隔两个相邻的路径段。`path.sep` 包含当前平台的路径分隔符：

    ```js
    assert.equal(
     path.posix.sep, '/' // Path separator on Unix
    );
    assert.equal(
     path.win32.sep, '\\' // Path separator on Windows
    );
    ```

+   *路径分隔符* 用于分隔路径列表中的元素。`path.delimiter` 包含当前平台的路径分隔符：

    ```js
    assert.equal(
     path.posix.delimiter, ':' // Path delimiter on Unix
    );
    assert.equal(
     path.win32.delimiter, ';' // Path delimiter on Windows
    );
    ```

如果我们检查 PATH shell 变量，我们可以看到路径分隔符和路径分隔符：

这是 macOS PATH 的一个示例（shell 变量 `$PATH`）：

```js
> process.env.PATH.split(/(?<=:)/)
[
 '/opt/homebrew/bin:',
 '/opt/homebrew/sbin:',
 '/usr/local/bin:',
 '/usr/bin:',
 '/bin:',
 '/usr/sbin:',
 '/sbin',
]
```

分隔符的长度为零，因为[回顾断言](https://exploringjs.com/impatient-js/ch_regexps.html#regexp-lookbehind-assertions) `(?<=:)` 匹配给定位置是否由冒号前导，但它不捕获任何内容。因此，路径分隔符 `':'` 包含在前面的路径中。

这是 Windows PATH 的一个示例（shell 变量 `%Path%`）：

```js
> process.env.Path.split(/(?<=;)/)
[
 'C:\\Windows\\system32;',
 'C:\\Windows;',
 'C:\\Windows\\System32\\Wbem;',
 'C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\;',
 'C:\\Windows\\System32\\OpenSSH\\;',
 'C:\\ProgramData\\chocolatey\\bin;',
 'C:\\Program Files\\nodejs\\',
]
```

#### 7.2.2 当前工作目录

许多 shell 都有*当前工作目录*（CWD）的概念 - “我当前所在的目录”：

+   如果我们使用部分合格的路径执行命令，该路径将相对于当前工作目录进行解析。

+   如果我们在命令期望路径时省略路径，将使用当前工作目录。

+   在 Unix 和 Windows 上，改变当前工作目录的命令是 `cd`。

`process` 是一个全局的 Node.js 变量。它为我们提供了获取和设置当前工作目录的方法：

+   [`process.cwd()`](https://nodejs.org/api/process.html#processcwd) 返回当前工作目录。

+   [`process.chdir(dirPath)`](https://nodejs.org/api/process.html#processchdirdirectory) 将当前工作目录更改为 `dirPath`。

    +   `dirPath` 必须存在一个目录。

    +   这种更改不会影响 shell，只会影响当前正在运行的 Node.js 进程。

Node.js 使用当前工作目录来填充缺失的部分，每当路径不是完全合格时。这使我们能够在各种函数中使用部分合格的路径，例如 `fs.readFileSync()`。

##### 7.2.2.1 Unix 上的当前工作目录

以下代码演示了在 Unix 上使用 `process.chdir()` 和 `process.cwd()`：

```js
process.chdir('/home/jane');
assert.equal(
 process.cwd(), '/home/jane'
);
```

##### 7.2.2.2 Windows 上的当前工作目录

到目前为止，我们已经在 Unix 上使用了当前工作目录。Windows 的工作方式不同：

+   每个驱动器都有一个*当前目录*。

+   有一个*当前驱动器*。

我们可以使用 `path.chdir()` 同时设置两者：

```js
process.chdir('C:\\Windows');
process.chdir('Z:\\tmp');
```

当我们重新访问一个驱动器时，Node.js 会记住该驱动器的先前当前目录：

```js
assert.equal(
 process.cwd(), 'Z:\\tmp'
);
process.chdir('C:');
assert.equal(
 process.cwd(), 'C:\\Windows'
);
```

#### 7.2.3 完全合格与部分合格的路径，解析路径

+   *完全合格的路径*不依赖于任何其他信息，可以直接使用。

+   *部分合格的路径*缺少信息：我们需要将其转换为完全合格的路径才能使用。这是通过将其与完全合格的路径*解析*来完成的。

##### 7.2.3.1 Unix 上的完全合格和部分合格路径

Unix 只知道两种路径：

+   *绝对路径*是完全合格的，并以斜杠开头：

    ```js
    /home/john/proj
    ```

+   *相对路径*是部分合格的，以文件名或点开头：

    ```js
    .   (current directory)
    ..  (parent directory)
    dir
    ./dir
    ../dir
    ../../dir/subdir
    ```

让我们使用`path.resolve()`（在后面有更详细的解释）来解析相对路径与绝对路径。结果是绝对路径：

```js
> const abs = '/home/john/proj';

> path.resolve(abs, '.')
'/home/john/proj'
> path.resolve(abs, '..')
'/home/john'
> path.resolve(abs, 'dir')
'/home/john/proj/dir'
> path.resolve(abs, './dir')
'/home/john/proj/dir'
> path.resolve(abs, '../dir')
'/home/john/dir'
> path.resolve(abs, '../../dir/subdir')
'/home/dir/subdir'
```

##### 7.2.3.2 Windows 上的完全合格和部分合格路径

Windows 区分四种路径（有关更多信息，请参阅[Microsoft 的文档](https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file#fully-qualified-vs-relative-paths)）：

+   有绝对路径和相对路径。

+   这两种路径都可以有驱动器号（“卷标”)或者没有。

带有驱动器号的绝对路径是完全合格的。所有其他路径都是部分合格的。

**解析没有驱动器号的绝对路径**与完全合格路径`full`，会获取`full`的驱动器号：

```js
> const full = 'C:\\Users\\jane\\proj';

> path.resolve(full, '\\Windows')
'C:\\Windows'
```

**解析没有驱动器号的相对路径**与完全合格路径，可以看作是更新后者：

```js
> const full = 'C:\\Users\\jane\\proj';

> path.resolve(full, '.')
'C:\\Users\\jane\\proj'
> path.resolve(full, '..')
'C:\\Users\\jane'
> path.resolve(full, 'dir')
'C:\\Users\\jane\\proj\\dir'
> path.resolve(full, '.\\dir')
'C:\\Users\\jane\\proj\\dir'
> path.resolve(full, '..\\dir')
'C:\\Users\\jane\\dir'
> path.resolve(full, '..\\..\\dir')
'C:\\Users\\dir'
```

**解析带有驱动器号的相对路径**与完全合格路径`full`取决于`rel`的驱动器号：

+   与`full`相同的驱动器号？将`rel`解析为`full`。

+   与`full`不同的驱动器号？将`rel`解析为`rel`驱动器的当前目录。

看起来如下：

```js
// Configure current directories for C: and Z:
process.chdir('C:\\Windows\\System');
process.chdir('Z:\\tmp');

const full = 'C:\\Users\\jane\\proj';

// Same drive letter
assert.equal(
 path.resolve(full, 'C:dir'),
 'C:\\Users\\jane\\proj\\dir'
);
assert.equal(
 path.resolve(full, 'C:'),
 'C:\\Users\\jane\\proj'
);

// Different drive letter
assert.equal(
 path.resolve(full, 'Z:dir'),
 'Z:\\tmp\\dir'
);
assert.equal(
 path.resolve(full, 'Z:'),
 'Z:\\tmp'
);
```

### 7.3 通过模块`'node:os'`获取标准目录的路径

模块`'node:os'`为我们提供了两个重要目录的路径：

+   [`os.homedir()`](https://nodejs.org/api/os.html#oshomedir)返回当前用户的主目录路径，例如：

    ```js
    > os.homedir() // macOS
    '/Users/rauschma'
    > os.homedir() // Windows
    'C:\\Users\\axel'
    ```

+   [`os.tmpdir()`](https://nodejs.org/api/os.html#ostmpdir)返回操作系统用于临时文件的目录路径，例如：

    ```js
    > os.tmpdir() // macOS
    '/var/folders/ph/sz0384m11vxf5byk12fzjms40000gn/T'
    > os.tmpdir() // Windows
    'C:\\Users\\axel\\AppData\\Local\\Temp'
    ```

### 7.4 连接路径

有两个用于连接路径的函数：

+   `path.resolve()`总是返回完全合格的路径

+   `path.join()` 保留相对路径

#### 7.4.1 `path.resolve()`: 连接路径以创建完全合格的路径

```js
path.resolve(...paths: Array<string>): string
```

连接`paths`并返回完全合格的路径。它使用以下算法：

+   从当前工作目录开始。

+   将`path[0]`解析为先前的结果。

+   将`path[1]`解析为先前的结果。

+   对所有剩余的路径执行相同的操作。

+   返回最终结果。

没有参数，`path.resolve()`返回当前工作目录的路径：

```js
> process.cwd()
'/usr/local'
> path.resolve()
'/usr/local'
```

一个或多个相对路径用于解析，从当前工作目录开始：

```js
> path.resolve('.')
'/usr/local'
> path.resolve('..')
'/usr'
> path.resolve('bin')
'/usr/local/bin'
> path.resolve('./bin', 'sub')
'/usr/local/bin/sub'
> path.resolve('../lib', 'log')
'/usr/lib/log'
```

任何完全合格的路径都会替换先前的结果：

```js
> path.resolve('bin', '/home')
'/home'
```

这使我们能够解析部分合格的路径与完全合格的路径：

```js
> path.resolve('/home/john', 'proj', 'src')
'/home/john/proj/src'
```

#### 7.4.2 `path.join()`: 连接路径同时保留相对路径

```js
path.join(...paths: Array<string>): string
```

从`paths[0]`开始，并将其余路径解释为上升或下降的指令。与`path.resolve()`相反，此函数保留部分合格的路径：如果`paths[0]`是部分合格的，则结果也是部分合格的。如果它是完全合格的，则结果也是完全合格的。

下降的例子：

```js
> path.posix.join('/usr/local', 'sub', 'subsub')
'/usr/local/sub/subsub'
> path.posix.join('relative/dir', 'sub', 'subsub')
'relative/dir/sub/subsub'
```

双点上升：

```js
> path.posix.join('/usr/local', '..')
'/usr'
> path.posix.join('relative/dir', '..')
'relative'
```

单个点不起作用：

```js
> path.posix.join('/usr/local', '.')
'/usr/local'
> path.posix.join('relative/dir', '.')
'relative/dir'
```

如果第一个参数之后的参数是完全合格的路径，则将其解释为相对路径：

```js
> path.posix.join('dir', '/tmp')
'dir/tmp'
> path.win32.join('dir', 'C:\\Users')
'dir\\C:\\Users'
```

使用多于两个参数：

```js
> path.posix.join('/usr/local', '../lib', '.', 'log')
'/usr/lib/log'
```

### 7.5 确保路径被规范化，完全合格或相对

#### 7.5.1 `path.normalize()`: 确保路径被规范化

```js
path.normalize(path: string): string
```

在 Unix 上，`path.normalize()`：

+   删除单个点（`。`）的路径段。

+   解析双点（`..`）的路径段。

+   将多个路径分隔符转换为单个路径分隔符。

例如：

```js
// Fully qualified path
assert.equal(
 path.posix.normalize('/home/./john/lib/../photos///pet'),
 '/home/john/photos/pet'
);

// Partially qualified path
assert.equal(
 path.posix.normalize('./john/lib/../photos///pet'),
 'john/photos/pet'
);
```

在 Windows 上，`path.normalize()`：

+   删除单点（`.`）的路径段。

+   解析双点（`..`）的路径段。

+   将每个路径分隔符斜杠（`/`）转换为首选路径分隔符（`\`）。

+   将多个路径分隔符序列转换为单个反斜杠。

例如：

```js
// Fully qualified path
assert.equal(
 path.win32.normalize('C:\\Users/jane\\doc\\..\\proj\\\\src'),
 'C:\\Users\\jane\\proj\\src'
);

// Partially qualified path
assert.equal(
 path.win32.normalize('.\\jane\\doc\\..\\proj\\\\src'),
 'jane\\proj\\src'
);
```

请注意，使用单个参数的`path.join()`也会规范化并且与`path.normalize()`的工作方式相同：

```js
> path.posix.normalize('/home/./john/lib/../photos///pet')
'/home/john/photos/pet'
> path.posix.join('/home/./john/lib/../photos///pet')
'/home/john/photos/pet'

> path.posix.normalize('./john/lib/../photos///pet')
'john/photos/pet'
> path.posix.join('./john/lib/../photos///pet')
'john/photos/pet'
```

#### 7.5.2 `path.resolve()`（一个参数）：确保路径被规范化和完全合格

我们已经遇到了`path.resolve()`。使用单个参数调用它，它会规范化路径并确保它们是完全合格的。

在 Unix 上使用`path.resolve()`：

```js
> process.cwd()
'/usr/local'

> path.resolve('/home/./john/lib/../photos///pet')
'/home/john/photos/pet'
> path.resolve('./john/lib/../photos///pet')
'/usr/local/john/photos/pet'
```

在 Windows 上使用`path.resolve()`：

```js
> process.cwd()
'C:\\Windows\\System'

> path.resolve('C:\\Users/jane\\doc\\..\\proj\\\\src')
'C:\\Users\\jane\\proj\\src'
> path.resolve('.\\jane\\doc\\..\\proj\\\\src')
'C:\\Windows\\System\\jane\\proj\\src'
```

#### 7.5.3 `path.relative()`: 创建相对路径

```js
path.relative(sourcePath: string, destinationPath: string): string
```

返回一个相对路径，使我们从`sourcePath`到`destinationPath`：

```js
> path.posix.relative('/home/john/', '/home/john/proj/my-lib/README.md')
'proj/my-lib/README.md'
> path.posix.relative('/tmp/proj/my-lib/', '/tmp/doc/zsh.txt')
'../../doc/zsh.txt'
```

在 Windows 上，如果`sourcePath`和`destinationPath`位于不同的驱动器上，则会得到一个完全合格的路径：

```js
> path.win32.relative('Z:\\tmp\\', 'C:\\Users\\Jane\\')
'C:\\Users\\Jane'
```

此函数还适用于相对路径：

```js
> path.posix.relative('proj/my-lib/', 'doc/zsh.txt')
'../../doc/zsh.txt'
```

### 7.6 解析路径：提取路径的各个部分（文件扩展名等）

#### 7.6.1 `path.parse()`: 创建具有路径部分的对象

```js
type PathObject = {
 dir: string,
 root: string,
 base: string,
 name: string,
 ext: string,
};
path.parse(path: string): PathObject
```

提取`path`的各个部分，并以具有以下属性的对象返回它们：

+   `.base`：路径的最后一部分

    +   `.ext`：基本的文件扩展名

    +   `.name`：没有扩展名的基本部分。这部分也被称为路径的*stem*。

+   `.root`：路径的开始（第一个段之前）

+   `.dir`：基本所在的目录-没有基本的路径

稍后，我们将看到函数`path.format()`，它是`path.parse()`的反函数：它将具有路径部分的对象转换为路径。

##### 7.6.1.1 `path.parse()`在 Unix 上的示例

这是在 Unix 上使用`path.parse()`的样子：

```js
> path.posix.parse('/home/jane/file.txt')
{
 dir: '/home/jane',
 root: '/',
 base: 'file.txt',
 name: 'file',
 ext: '.txt',
}
```

以下图表可视化了各个部分的范围：

```js
 /      home/jane / file   .txt
| root |           | name | ext  |
| dir              | base        |
```

例如，我们可以看到`.dir`是没有基本的路径。而`.base`是`.name`加上`.ext`。

##### 7.6.1.2 `path.parse()`在 Windows 上的示例

这是`path.parse()`在 Windows 上的工作方式：

```js
> path.win32.parse(String.raw`C:\Users\john\file.txt`)
{
 dir: 'C:\\Users\\john',
 root: 'C:\\',
 base: 'file.txt',
 name: 'file',
 ext: '.txt',
}
```

这是结果的图表：

```js
 C:\    Users\john \ file   .txt
| root |            | name | ext  |
| dir               | base        |
```

#### 7.6.2 `path.basename()`: 提取路径的基本部分

```js
path.basename(path, ext?)
```

返回`path`的基本部分：

```js
> path.basename('/home/jane/file.txt')
'file.txt'
```

可选地，此函数还可以删除后缀：

```js
> path.basename('/home/jane/file.txt', '.txt')
'file'
> path.basename('/home/jane/file.txt', 'txt')
'file.'
> path.basename('/home/jane/file.txt', 'xt')
'file.t'
```

删除扩展名是区分大小写的-即使在 Windows 上也是如此！

```js
> path.win32.basename(String.raw`C:\Users\john\file.txt`, '.txt')
'file'
> path.win32.basename(String.raw`C:\Users\john\file.txt`, '.TXT')
'file.txt'
```

#### 7.6.3 `path.dirname()`: 提取路径的父目录

```js
path.dirname(path)
```

返回`path`中文件或目录的父目录：

```js
> path.win32.dirname(String.raw`C:\Users\john\file.txt`)
'C:\\Users\\john'
> path.win32.dirname('C:\\Users\\john\\dir\\')
'C:\\Users\\john'

> path.posix.dirname('/home/jane/file.txt')
'/home/jane'
> path.posix.dirname('/home/jane/dir/')
'/home/jane'
```

#### 7.6.4 `path.extname()`: 提取路径的扩展名

```js
path.extname(path)
```

返回`path`的扩展名：

```js
> path.extname('/home/jane/file.txt')
'.txt'
> path.extname('/home/jane/file.')
'.'
> path.extname('/home/jane/file')
''
> path.extname('/home/jane/')
''
> path.extname('/home/jane')
''
```

### 7.7 对路径进行分类

#### 7.7.1 `path.isAbsolute()`: 给定路径是否是绝对路径？

```js
path.isAbsolute(path: string): boolean
```

如果`path`是绝对路径则返回`true`，否则返回`false`。

在 Unix 上的结果很直接：

```js
> path.posix.isAbsolute('/home/john')
true
> path.posix.isAbsolute('john')
false
```

在 Windows 上，“绝对”并不一定意味着“完全合格”（只有第一个路径是完全合格的）：

```js
> path.win32.isAbsolute('C:\\Users\\jane')
true
> path.win32.isAbsolute('\\Users\\jane')
true
> path.win32.isAbsolute('C:jane')
false
> path.win32.isAbsolute('jane')
false
```

### 7.8 `path.format()`: 从部分创建路径

```js
type PathObject = {
 dir: string,
 root: string,
 base: string,
 name: string,
 ext: string,
};
path.format(pathObject: PathObject): string
```

从路径对象创建路径：

```js
> path.format({dir: '/home/jane', base: 'file.txt'})
'/home/jane/file.txt'
```

#### 7.8.1 示例：更改文件扩展名

我们可以使用`path.format()`来更改路径的扩展名：

```js
function changeFilenameExtension(pathStr, newExtension) {
 if (!newExtension.startsWith('.')) {
 throw new Error(
 'Extension must start with a dot: '
 + JSON.stringify(newExtension)
 );
 }
 const parts = path.parse(pathStr);
 return path.format({
 ...parts,
 base: undefined, // prevent .base from overriding .name and .ext
 ext: newExtension,
 });
}

assert.equal(
 changeFilenameExtension('/tmp/file.md', '.html'),
 '/tmp/file.html'
);
assert.equal(
 changeFilenameExtension('/tmp/file', '.html'),
 '/tmp/file.html'
);
assert.equal(
 changeFilenameExtension('/tmp/file/', '.html'),
 '/tmp/file.html'
);
```

如果我们知道原始文件名的扩展名，我们也可以使用正则表达式来更改文件名的扩展名：

```js
> '/tmp/file.md'.replace(/\.md$/i, '.html')
'/tmp/file.html'
> '/tmp/file.MD'.replace(/\.md$/i, '.html')
'/tmp/file.html'
```

### 7.9 在不同平台上使用相同的路径

有时我们希望在不同平台上使用相同的路径。然后我们面临两个问题：

+   路径分隔符可能不同。

+   文件结构可能不同：主目录和临时文件目录可能位于不同位置等。

例如，考虑一个在一个包含数据的目录上运行的 Node.js 应用程序。假设该应用程序可以配置两种类型的路径：

+   系统中任何地方都是完全合格的路径

+   数据目录内的路径

由于前面提到的问题：

+   我们不能在不同平台之间重用完全合格的路径。

    +   有时我们需要绝对路径。这些必须针对数据目录的“实例”进行配置，并存储在外部（或内部并被版本控制忽略）。这些路径保持不变，不会随数据目录移动。

+   我们可以重用指向数据目录的路径。这些路径可以存储在配置文件中（数据目录内或外）和应用程序代码中的常量中。为此：

    +   我们必须将它们存储为相对路径。

    +   我们必须确保每个平台上的路径分隔符是正确的。

    下一小节解释了如何实现这两个目标。

#### 7.9.1 相对平台无关的路径

相对平台无关的路径可以存储为路径段的数组，并按以下方式转换为完全合格的特定平台的路径：

```js
const universalRelativePath = ['static', 'img', 'logo.jpg'];

const dataDirUnix = '/home/john/data-dir';
assert.equal(
 path.posix.resolve(dataDirUnix, ...universalRelativePath),
 '/home/john/data-dir/static/img/logo.jpg'
);

const dataDirWindows = 'C:\\Users\\jane\\data-dir';
assert.equal(
 path.win32.resolve(dataDirWindows, ...universalRelativePath),
 'C:\\Users\\jane\\data-dir\\static\\img\\logo.jpg'
);
```

要创建相对于特定平台的路径，我们可以使用：

```js
const dataDir = '/home/john/data-dir';
const pathInDataDir = '/home/john/data-dir/static/img/logo.jpg';
assert.equal(
 path.relative(dataDir, pathInDataDir),
 'static/img/logo.jpg'
);
```

以下函数将相对于特定平台的路径转换为平台无关的路径：

```js
import * as path from 'node:path';

function splitRelativePathIntoSegments(relPath) {
 if (path.isAbsolute(relPath)) {
 throw new Error('Path isn’t relative: ' + relPath);
 }
 relPath = path.normalize(relPath);
 const result = [];
 while (true) {
 const base = path.basename(relPath);
 if (base.length === 0) break;
 result.unshift(base);
 const dir = path.dirname(relPath);
 if (dir === '.') break;
 relPath = dir;
 }
 return result;
}
```

在 Unix 上使用`splitRelativePathIntoSegments()`：

```js
> splitRelativePathIntoSegments('static/img/logo.jpg')
[ 'static', 'img', 'logo.jpg' ]
> splitRelativePathIntoSegments('file.txt')
[ 'file.txt' ]
```

在 Windows 上使用`splitRelativePathIntoSegments()`：

```js
> splitRelativePathIntoSegments('static/img/logo.jpg')
[ 'static', 'img', 'logo.jpg' ]
> splitRelativePathIntoSegments('C:static/img/logo.jpg')
[ 'static', 'img', 'logo.jpg' ]

> splitRelativePathIntoSegments('file.txt')
[ 'file.txt' ]
> splitRelativePathIntoSegments('C:file.txt')
[ 'file.txt' ]
```

### 7.10 使用库通过*globs*匹配路径

[npm 模块`'minimatch'`](https://github.com/isaacs/minimatch)让我们可以根据称为*glob 表达式*、*glob 模式*或*glob*的模式匹配路径：

```js
import minimatch from 'minimatch';
assert.equal(
 minimatch('/dir/sub/file.txt', '/dir/sub/*.txt'), true
);
assert.equal(
 minimatch('/dir/sub/file.txt', '/**/file.txt'), true
);
```

通配符的用例：

+   指定目录中应由脚本处理的文件。

+   指定要忽略哪些文件。

更多的通配符库：

+   [multimatch](https://github.com/sindresorhus/multimatch)扩展了 minimatch，支持多个模式。

+   [micromatch](https://github.com/micromatch/micromatch)是 minimatch 和 multimatch 的替代品，具有类似的 API。

+   [globby](https://github.com/sindresorhus/globby)是基于[fast-glob](https://github.com/mrmlnc/fast-glob)的库，添加了便利功能。

#### 7.10.1 minimatch API

minimatch 的整个 API 在[项目的自述文件](https://github.com/isaacs/minimatch)中有文档。在本小节中，我们将重点关注最重要的功能。

Minimatch 将通配符编译为 JavaScript `RegExp`对象，并使用它们进行匹配。

##### 7.10.1.1 `minimatch()`: 编译和匹配一次

```js
minimatch(path: string, glob: string, options?: MinimatchOptions): boolean
```

如果`glob`匹配`path`，则返回`true`，否则返回`false`。

两个有趣的选项：

+   `.dot: boolean`（默认值：`false`）

    如果为`true`，通配符符号如`*`和`**`将匹配“不可见”的路径段（其名称以点开头）：

    ```js
    > minimatch('/usr/local/.tmp/data.json', '/usr/**/data.json')
    false
    > minimatch('/usr/local/.tmp/data.json', '/usr/**/data.json', {dot: true})
    true

    > minimatch('/tmp/.log/events.txt', '/tmp/*/events.txt')
    false
    > minimatch('/tmp/.log/events.txt', '/tmp/*/events.txt', {dot: true})
    true
    ```

+   `.matchBase: boolean`（默认值：`false`）

    如果为`true`，不带斜杠的模式将与路径的基本名称匹配：

    ```js
    > minimatch('/dir/file.txt', 'file.txt')
    false
    > minimatch('/dir/file.txt', 'file.txt', {matchBase: true})
    true
    ```

##### 7.10.1.2 `new minimatch.Minimatch()`: 编译一次，多次匹配

类`minimatch.Minimatch`使我们只需将通配符编译为正则表达式一次，就可以多次进行匹配：

```js
new Minimatch(pattern: string, options?: MinimatchOptions)
```

这是如何使用这个类的：

```js
import minimatch from 'minimatch';
const {Minimatch} = minimatch;
const glob = new Minimatch('/dir/sub/*.txt');
assert.equal(
 glob.match('/dir/sub/file.txt'), true
);
assert.equal(
 glob.match('/dir/sub/notes.txt'), true
);
```

#### 7.10.2 通配符表达式的语法

本小节涵盖了语法的基本要点。但还有更多功能。这些在这里记录：

+   [Minimatch 的单元测试](https://github.com/isaacs/minimatch/tree/main/test)有许多通配符的示例。

+   Bash 参考手册有[关于文件名扩展的部分](https://www.gnu.org/software/bash/manual/bash.html#Filename-Expansion)。

##### 7.10.2.1 匹配 Windows 路径

即使在 Windows 上，通配符段也是由斜杠分隔的-但它们匹配反斜杠和斜杠（这些是 Windows 上合法的路径分隔符）：

```js
> minimatch('dir\\sub/file.txt', 'dir/sub/file.txt')
true
```

##### 7.10.2.2 Minimatch 不会规范化路径

Minimatch 不会为我们规范化路径：

```js
> minimatch('./file.txt', './file.txt')
true
> minimatch('./file.txt', 'file.txt')
false
> minimatch('file.txt', './file.txt')
false
```

因此，如果我们不自己创建路径，我们必须规范化路径：

```js
> path.normalize('./file.txt')
'file.txt'
```

##### 7.10.2.3 不带通配符符号的模式：路径分隔符必须对齐

不带通配符符号的模式（更灵活匹配）必须精确匹配。特别是路径分隔符必须对齐：

```js
> minimatch('/dir/file.txt', '/dir/file.txt')
true
> minimatch('dir/file.txt', 'dir/file.txt')
true
> minimatch('/dir/file.txt', 'dir/file.txt')
false

> minimatch('/dir/file.txt', 'file.txt')
false
```

也就是说，我们必须决定是绝对路径还是相对路径。

使用`.matchBase`选项，我们可以匹配不带斜杠的模式与路径的基本名称：

```js
> minimatch('/dir/file.txt', 'file.txt', {matchBase: true})
true
```

##### 7.10.2.4 星号（`*`）匹配任何（部分）单个段

通配符符号*（`*`）匹配任何路径段或段的任何部分：

```js
> minimatch('/dir/file.txt', '/*/file.txt')
true
> minimatch('/tmp/file.txt', '/*/file.txt')
true

> minimatch('/dir/file.txt', '/dir/*.txt')
true
> minimatch('/dir/data.txt', '/dir/*.txt')
true
```

星号不匹配以点开头的“隐藏文件”。如果我们想匹配这些文件，我们必须在星号前加上点：

```js
> minimatch('file.txt', '*')
true
> minimatch('.gitignore', '*')
false
> minimatch('.gitignore', '.*')
true
> minimatch('/tmp/.log/events.txt', '/tmp/*/events.txt')
false
```

选项`.dot`让我们关闭这种行为：

```js
> minimatch('.gitignore', '*', {dot: true})
true
> minimatch('/tmp/.log/events.txt', '/tmp/*/events.txt', {dot: true})
true
```

##### 7.10.2.5 双星号（`**`）匹配零个或多个段

´`**/`匹配零个或多个段：

```js
> minimatch('/file.txt', '/**/file.txt')
true
> minimatch('/dir/file.txt', '/**/file.txt')
true
> minimatch('/dir/sub/file.txt', '/**/file.txt')
true
```

如果我们想匹配相对路径，模式仍然不能以路径分隔符开头：

```js
> minimatch('file.txt', '/**/file.txt')
false
```

双星号不匹配以点开头的“隐藏”路径段：

```js
> minimatch('/usr/local/.tmp/data.json', '/usr/**/data.json')
false
```

我们可以通过选项`.dot`关闭该行为：

```js
> minimatch('/usr/local/.tmp/data.json', '/usr/**/data.json', {dot: true})
true
```

##### 7.10.2.6 否定通配符

如果我们以感叹号开头的通配符，它将匹配感叹号后的模式不匹配的情况：

```js
> minimatch('file.txt', '!**/*.txt')
false
> minimatch('file.js', '!**/*.txt')
true
```

##### 7.10.2.7 替代模式

大括号内逗号分隔的模式匹配，如果其中一个模式匹配：

```js
> minimatch('file.txt', 'file.{txt,js}')
true
> minimatch('file.js', 'file.{txt,js}')
true
```

##### 7.10.2.8 整数范围

由双点分隔的一对整数定义了整数范围，并且如果其任何元素匹配，则匹配：

```js
> minimatch('file1.txt', 'file{1..3}.txt')
true
> minimatch('file2.txt', 'file{1..3}.txt')
true
> minimatch('file3.txt', 'file{1..3}.txt')
true
> minimatch('file4.txt', 'file{1..3}.txt')
false
```

还支持用零填充：

```js
> minimatch('file1.txt', 'file{01..12}.txt')
false
> minimatch('file01.txt', 'file{01..12}.txt')
true
> minimatch('file02.txt', 'file{01..12}.txt')
true
> minimatch('file12.txt', 'file{01..15}.txt')
true
```

### 7.11 使用`file:` URL 引用文件

在 Node.js 中有两种常见的引用文件的方式：

+   字符串中的路径

+   具有协议`file:`的`URL`实例

例如：

```js
assert.equal(
 fs.readFileSync(
 '/tmp/data.txt', {encoding: 'utf-8'}),
 'Content'
);
assert.equal(
 fs.readFileSync(
 new URL('file:///tmp/data.txt'), {encoding: 'utf-8'}),
 'Content'
);
```

#### 7.11.1 `URL`类

在本节中，我们更详细地了解了`URL`类。有关此类的更多信息：

+   Node.js 文档：部分[“WHATWG URL API”](https://nodejs.org/api/url.html#the-whatwg-url-api)

+   WHATWG URL 标准的[“API”部分](https://url.spec.whatwg.org/#api)

在本章中，我们通过全局变量访问`URL`类，因为这是在其他 Web 平台上使用的方式。但它也可以被导入：

```js
import {URL} from 'node:url';
```

##### 7.11.1.1 URIs vs. 相对引用

URL 是 URI 的一个子集。URI 的标准 RFC 3986 区分了[两种*URI 引用*](https://datatracker.ietf.org/doc/html/rfc3986#section-4.1)：

+   *URI*以[方案](https://datatracker.ietf.org/doc/html/rfc3986#section-3.1)开头，后跟冒号分隔符。

+   所有其他 URI 引用都是*相对引用*。

##### 7.11.1.2 `URL`的构造函数

`URL`类可以通过两种方式实例化：

+   `new URL(uri: string)`

    `uri`必须是一个 URI。它指定了新实例的 URI。

+   `new URL(uriRef: string, baseUri: string)`

    `baseUri`必须是一个 URI。如果`uriRef`是相对引用，它将根据`baseUri`解析，并且结果将成为新实例的 URI。

    如果`uriRef`是一个 URI，则它完全替换`baseUri`作为实例所基于的数据。

在这里我们可以看到类的实际应用：

```js
// If there is only one argument, it must be a proper URI
assert.equal(
 new URL('https://example.com/public/page.html').toString(),
 'https://example.com/public/page.html'
);
assert.throws(
 () => new URL('../book/toc.html'),
 /^TypeError \[ERR_INVALID_URL\]: Invalid URL$/
);

// Resolve a relative reference against a base URI 
assert.equal(
 new URL(
 '../book/toc.html',
 'https://example.com/public/page.html'
 ).toString(),
 'https://example.com/book/toc.html'
);
```

##### 7.11.1.3 相对引用解析为`URL`实例

让我们重新访问`URL`构造函数的这个变体：

```js
new URL(uriRef: string, baseUri: string)
```

参数`baseUri`被强制转换为字符串。因此，任何对象都可以使用-只要在强制转换为字符串时成为有效的 URL：

```js
const obj = { toString() {return 'https://example.com'} };
assert.equal(
 new URL('index.html', obj).href,
 'https://example.com/index.html'
);
```

这使我们能够相对于`URL`实例解析相对引用：

```js
const url = new URL('https://example.com/dir/file1.html');
assert.equal(
 new URL('../file2.html', url).href,
 'https://example.com/file2.html'
);
```

以这种方式使用构造函数，它与`path.resolve()` loosly 类似。

##### 7.11.1.4 `URL`实例的属性

`URL`实例具有以下属性：

```js
type URL = {
 protocol: string,
 username: string,
 password: string,
 hostname: string,
 port: string,
 host: string,
 readonly origin: string,

 pathname: string,

 search: string,
 readonly searchParams: URLSearchParams,
 hash: string,

 href: string,
 toString(): string,
 toJSON(): string,
}
```

##### 7.11.1.5 将 URL 转换为字符串

有三种常见的方法可以将 URL 转换为字符串：

```js
const url = new URL('https://example.com/about.html');

assert.equal(
 url.toString(),
 'https://example.com/about.html'
);
assert.equal(
 url.href,
 'https://example.com/about.html'
);
assert.equal(
 url.toJSON(),
 'https://example.com/about.html'
);
```

方法`.toJSON()`使我们能够在 JSON 数据中使用 URL：

```js
const jsonStr = JSON.stringify({
 pageUrl: new URL('https://exploringjs.com')
});
assert.equal(
 jsonStr, '{"pageUrl":"https://exploringjs.com"}'
);
```

##### 7.11.1.6 获取`URL`属性

`URL`实例的属性不是自有数据属性，它们是通过 getter 和 setter 实现的。在下一个示例中，我们使用实用函数`pickProps()`（其代码在最后显示）将这些 getter 返回的值复制到普通对象中：

```js
const props = pickProps(
 new URL('https://jane:pw@example.com:80/news.html?date=today#misc'),
 'protocol', 'username', 'password', 'hostname', 'port', 'host',
 'origin', 'pathname', 'search', 'hash', 'href'
);
assert.deepEqual(
 props,
 {
 protocol: 'https:',
 username: 'jane',
 password: 'pw',
 hostname: 'example.com',
 port: '80',
 host: 'example.com:80',
 origin: 'https://example.com:80',
 pathname: '/news.html',
 search: '?date=today',
 hash: '#misc',
 href: 'https://jane:pw@example.com:80/news.html?date=today#misc'
 }
);
function pickProps(input, ...keys) {
 const output = {};
 for (const key of keys) {
 output[key] = input[key];
 }
 return output;
}
```

遗憾的是，路径名是一个单一的原子单位。也就是说，我们不能使用`URL`类来访问其部分（基础，扩展名等）。

##### 7.11.1.7 设置 URL 的部分

我们还可以通过设置`.hostname`等属性来更改 URL 的部分：

```js
const url = new URL('https://example.com');
url.hostname = '2ality.com';
assert.equal(
 url.href, 'https://2ality.com/'
);
```

我们可以使用 setter 从部分创建 URL（[Haroen Viaene 的想法](https://twitter.com/haroenv/status/1545357986046017539)）：

```js
// Object.assign() invokes setters when transferring property values
const urlFromParts = (parts) => Object.assign(
 new URL('https://example.com'), // minimal dummy URL
 parts // assigned to the dummy
);

const url = urlFromParts({
 protocol: 'https:',
 hostname: '2ality.com',
 pathname: '/p/about.html',
});
assert.equal(
 url.href, 'https://2ality.com/p/about.html'
);
```

##### 7.11.1.8 通过`.searchParams`管理搜索参数

我们可以使用属性`.searchParams`来管理 URL 的搜索参数。其值是[`URLSearchParams`](https://nodejs.org/api/url.html#class-urlsearchparams)的实例。

我们可以用它来读取搜索参数：

```js
const url = new URL('https://example.com/?topic=js');
assert.equal(
 url.searchParams.get('topic'), 'js'
);
assert.equal(
 url.searchParams.has('topic'), true
);
```

我们也可以通过它更改搜索参数：

```js
url.searchParams.append('page', '5');
assert.equal(
 url.href, 'https://example.com/?topic=js&page=5'
);

url.searchParams.set('topic', 'css');
assert.equal(
 url.href, 'https://example.com/?topic=css&page=5'
);
```

#### 7.11.2 在 URL 和文件路径之间进行转换

手动在文件路径和 URL 之间进行转换是很诱人的。例如，我们可以尝试通过`myUrl.pathname`将`URL`实例`myUrl`转换为文件路径。然而，这并不总是有效 - 最好使用[这个函数](https://nodejs.org/api/url.html#urlfileurltopathurl)：

```js
url.fileURLToPath(url: URL | string): string
```

以下代码将该函数的结果与`.pathname`的值进行比较：

```js
import * as url from 'node:url';

//::::: Unix :::::

const url1 = new URL('file:///tmp/with%20space.txt');
assert.equal(
 url1.pathname, '/tmp/with%20space.txt');
assert.equal(
 url.fileURLToPath(url1), '/tmp/with space.txt');

const url2 = new URL('file:///home/thor/Mj%C3%B6lnir.txt');
assert.equal(
 url2.pathname, '/home/thor/Mj%C3%B6lnir.txt');
assert.equal(
 url.fileURLToPath(url2), '/home/thor/Mjölnir.txt');

//::::: Windows :::::

const url3 = new URL('file:///C:/dir/');
assert.equal(
 url3.pathname, '/C:/dir/');
assert.equal(
 url.fileURLToPath(url3), 'C:\\dir\\');
```

[这个函数](https://nodejs.org/api/url.html#url_url_pathtofileurl_path)是`url.fileURLToPath()`的逆操作：

```js
url.pathToFileURL(path: string): URL
```

它将`path`转换为文件 URL：

```js
> url.pathToFileURL('/home/john/Work Files').href
'file:///home/john/Work%20Files'
```

#### 7.11.3 URL 的用例：访问相对于当前模块的文件

URL 的一个重要用例是访问当前模块的同级文件：

```js
function readData() {
 const url = new URL('data.txt', import.meta.url);
 return fs.readFileSync(url, {encoding: 'UTF-8'});
}
```

这个函数使用[`import.meta.url`](https://exploringjs.com/impatient-js/ch_modules.html#import.meta.url-on-node.js)，其中包含当前模块的 URL（通常在 Node.js 上是`file:` URL）。

使用`fetch()`会使先前的代码更加跨平台。然而，截至 Node.js 18.9.0，`fetch()`对于`file:` URL 尚不起作用：

```js
> await fetch('file:///tmp/file.txt')
TypeError: fetch failed
 cause: Error: not implemented... yet...
```

#### 7.11.4 URL 的用例：检测当前模块是否为“main”（应用程序入口点）

ESM 模块可以以两种方式使用：

1.  它可以作为其他模块可以导入值的库使用。

1.  它可以作为我们通过 Node.js 运行的脚本使用 - 例如，从命令行。在这种情况下，它被称为*主模块*。

如果我们希望一个模块以两种方式使用，我们需要一种方法来检查当前模块是否为主模块，因为只有在这种情况下我们才执行脚本功能。在本章中，我们将学习如何执行该检查。

##### 7.11.4.1 确定 CommonJS 模块是否为主要模块

使用 CommonJS，我们可以使用以下模式来检测当前模块是否为入口点（来源：[Node.js 文档](https://nodejs.org/api/modules.html#accessing-the-main-module)）：

```js
if (require.main === module) {
 // Main CommonJS module
}
```

##### 7.11.4.2 确定 ESM 模块是否为主要模块

到目前为止，ESM 模块没有简单的内置方法来检查模块是否为主模块。相反，我们必须使用以下解决方法（基于[Rich Harris 的一条推文](https://twitter.com/Rich_Harris/status/1355289863130673153)）：

```js
import * as url from 'node:url';

if (import.meta.url.startsWith('file:')) { // (A)
 const modulePath = url.fileURLToPath(import.meta.url);
 if (process.argv[1] === modulePath) { // (B)
 // Main ESM module
 }
}
```

解释：

+   [`import.meta.url`](https://exploringjs.com/impatient-js/ch_modules.html#import.meta.url)包含当前执行的 ESM 模块的 URL。

+   如果我们确定我们的代码始终在本地运行（这在将来可能变得不太常见），我们可以省略 A 行中的检查。如果我们这样做，而代码没有在本地运行，至少我们会得到一个异常（而不是静默失败）- 这要归功于`url.fileURLToPath()`（见下一项）。

+   我们使用`url.fileURLToPath()`将 URL 转换为本地路径。如果协议不是`file:`，此函数会抛出异常。

+   `process.argv[1]`包含初始模块的路径。B 行中的比较有效，因为这个值始终是绝对路径 - Node.js 设置如下（[源代码](https://github.com/nodejs/node/blob/36fbbe0b86131fa2dcca558872b02335586e0089/lib/internal/bootstrap/pre_execution.js#L100-L107)）：

    ```js
    process.argv[1] = path.resolve(process.argv[1]);
    ```

#### 7.11.5 路径 vs. `file:` URL

当 shell 脚本接收到文件的引用或导出文件的引用（例如在屏幕上记录它们）时，它们几乎总是路径。但是，有两种情况我们需要 URL（如前面的小节中所讨论的）：

+   访问相对于当前模块的文件

+   检测当前模块是否作为脚本运行

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/7)
