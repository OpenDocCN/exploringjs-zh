# 三、开始使用 Node.js

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_getting-started-with-nodejs.html`](https://exploringjs.com/nodejs-shell-scripting/ch_getting-started-with-nodejs.html)

* * *

+   3.1 获取 Node.js 的帮助

+   3.2 安装 Node.js 和 npm

+   3.3 运行 Node.js 代码

    +   3.3.1 在 Node.js REPL 中评估代码

    +   3.3.2 快速打印 JavaScript 表达式的结果

    +   3.3.3 使用 Node.js 代码运行模块

    +   3.3.4 运行剪贴板中的 Node.js 代码

* * *

本章介绍了 Node.js 的第一步。

### 3.1 获取 Node.js 的帮助

+   在线：

    +   [在线文档概述](https://nodejs.org/en/docs/)

    +   [API 文档](https://nodejs.org/api/)

    +   [命令行选项](https://nodejs.org/api/cli.html)

+   命令行：

    +   在线帮助：`node -h`

    +   打印 Node.js 的版本：`node -v`

    +   各种 Node.js 组件的打印版本：

        +   `npm version`

        +   `node -p process.versions`

### 3.2 安装 Node.js 和 npm

Node.js 的安装程序还安装了包管理器 npm。它可以从[Node.js 主页](https://nodejs.org/)下载，并适用于许多操作系统。

### 3.3 运行 Node.js 代码

#### 3.3.1 在 Node.js REPL 中评估代码

Node.js REPL（读取-求值-打印循环）是一个命令行，我们可以交互式地评估 Node.js 代码。

我们可以在[JavaScript 严格模式](https://exploringjs.com/impatient-js/ch_syntax.html#strict-mode)下启动 Node.js REPL（默认情况下，对于 ESM 模块中的代码，它更安全且已打开）：

```js
node --use_strict
```

如果我们运行`node`而不带任何参数，Node.js REPL 将不使用严格模式：

```js
node
```

这是使用 Node.js REPL 的样子（`%`是 Unix shell 提示，`>`是 Node.js REPL 提示）：

```js
% node
Welcome to Node.js v18.9.0.
Type ".help" for more information.
> path.join('dir', 'sub', 'file.txt')
'dir/sub/file.txt'
>
```

所有 Node 的内置模块都可以通过 REPL 中的全局变量访问：`assert`，`path`，`fs`，`util`等。

#### 3.3.2 快速打印 JavaScript 表达式的结果

我们可以使用带有选项`--print`（缩写：`-p`）的 shell 命令`node`来打印评估 JavaScript 表达式的结果。类似于 REPL，所有内置模块都可以通过全局变量访问。例如，以下命令打印主目录的路径，并且在 Unix 和 Windows 上都有效：

```js
node -p "os.homedir()"
```

有关此命令行选项的更多信息，请参见§15.7.7“`node --eval`和`node --print`”。

#### 3.3.3 使用 Node.js 代码运行模块

例如，以下模块：

```js
// my-module.mjs
import * as os from 'node:os';
console.log(os.userInfo());
```

我们可以通过 shell 来运行它：

```js
node my-module.mjs
```

#### 3.3.4 运行剪贴板中的 Node.js 代码

我们还可以运行我们从剪贴板复制的 Node.js 代码。例如，我们可以从上一节复制`my-module.mjs`的代码，然后在 macOS 上像这样运行它：

```js
pbpaste | node --input-type=module
```

选项`--input-type=module`告诉 Node.js 将从标准输入接收的代码解释为模块。除其他外，这使我们可以使用`import`。

macOS shell 命令`pbpaste`将剪贴板的内容发送到标准输出。其他操作系统也有类似的 shell 命令：

+   Windows 命令行：`powershell get-clipboard`

+   Windows PowerShell：`get-clipboard`

+   Linux：[`xclip`](https://github.com/astrand/xclip)

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/3)
