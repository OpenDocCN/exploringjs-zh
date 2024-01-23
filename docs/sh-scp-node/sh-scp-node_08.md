# 六、npm 概述（JavaScript 包管理器）

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_npm-overview.html`](https://exploringjs.com/nodejs-shell-scripting/ch_npm-overview.html)

* * *

+   6.1 npm 包管理器

+   6.2 npm 获取帮助

    +   6.2.1 在命令行上获取帮助

    +   6.2.2 在线获取帮助

+   6.3 常见的 npm 命令

+   6.4 npm 命令的缩写

* * *

### 6.1 npm 包管理器

*npm 注册表*是托管 JavaScript 包的事实标准。这些包具有特定的格式，称为*npm 包*。

因此，在 JavaScript 生态系统中，*包管理器*是一个用于安装 npm 包的命令行工具，可以从 npm 注册表或其他来源获取。

最受欢迎的包管理器称为*npm*，并与 Node.js 捆绑在一起。它的名称最初代表“Node Package Manager”。后来，当 npm 和 npm 注册表不仅用于 Node.js 包时，定义被更改为“npm 不是一个包管理器”（[来源](https://en.wikipedia.org/wiki/Npm_(software)#Acronym)）。

还有其他流行的包管理器，如 yarn 和 pnpm。所有这些包管理器默认都使用 npm 注册表。

我们通过 shell 命令`npm`使用 npm，它提供了多个子命令，如`npm install`。

### 6.2 获取 npm 帮助

#### 6.2.1 在命令行上获取帮助

我们可以使用`npm`命令来解释自身：一方面，有`-h`选项，可以在`npm`和 npm 命令之后使用。它提供简要的解释：

```js
npm -h        # brief explanation of `npm`
npm <cmd> -h  # brief explanation of `npm <cmd>`
```

另一方面，有`npm help`命令提供更长的解释：

```js
npm help         # brief explanation of `npm` (same as `npm -h`)
npm help npm     # longer explanation of `npm`
npm help <cmd>   # longer explanation of `npm <cmd>`
npm help <topic> # longer explanation of <topic>
```

帮助主题包括：

+   `folders`

+   `npmrc`

+   `package.json`

#### 6.2.2 在线获取帮助

[官方 npm 文档](https://docs.npmjs.com)也可以在线获取。

### 6.3 常见的 npm 命令

这是一些常见命令：

+   `npm init` “初始化”当前目录为一个包。也就是说，它在其中创建`package.json`文件。这个命令在§14.3.1 “设置包目录”中有解释。

+   `npm install` 全局或本地安装 npm 包。在§13 “安装 npm 包和运行 bin 脚本”中有解释。

+   `npm publish` 将包发布到注册表：它可以创建新包或更新现有包。在§14.5.3 “`npm publish`: 将包上传到 npm 注册表”中有解释。

+   `npm run`（简写为`npm run-script`）执行包脚本。包脚本在§15 “通过 npm 包脚本运行跨平台任务”中有解释。

+   `npm uninstall` 移除全局或本地安装的包。

+   `npm version` 打印记录 Node.js 和 npm 各个组件版本的`process.versions`对象：

    ```js
    {
     'my-package': '1.0.0', // current package
     npm: '8.15.0',
     node: '18.7.0',
     v8: '10.2.154.13-node.9',
     uv: '1.43.0', // libuv
     ···
     tz: '2022a', // version of tz database
     unicode: '14.0', // version of Unicode standard
     ···
    }
    ```

+   `npx`允许我们在不安装它们的情况下运行包中的 bin 脚本。在§13.4 “`npx`: 在 npm 包中运行 bin 脚本而不安装它们”中有描述。

npm 文档中有[所有 npm 命令的列表](https://docs.npmjs.com/cli/v8/commands)。

### 6.4 npm 命令的缩写

许多 npm 命令都有缩写，例如：

| 短 | 长 |
| --- | --- |
| `npm i` | `npm install` |
| `npm rm` | `npm uninstall` |
| `npm run` | `npm run-script` |

对于每个 npm 命令，[npm 文档](https://docs.npmjs.com/cli/v8/commands)还列出了所有的别名（包括缩写）。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/6)
