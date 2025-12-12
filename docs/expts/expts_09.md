# 7 尝试 TypeScript 而不安装它

> 原文：[`exploringjs.com/ts/book/ch_trying-out-typescript.html`](https://exploringjs.com/ts/book/ch_trying-out-typescript.html)

（广告，请勿屏蔽。）

1.  7.1 TypeScript 操场

1.  7.2 通过 `node --watch` 创建简单的 TypeScript 操场

    1.  7.2.1 更多配置：`tsconfig.json` 和 `package.json`

    1.  7.2.2 进一步阅读

1.  7.3 通过 Node.js 运行复制的 TypeScript 代码

本章提供了快速尝试 TypeScript 的技巧。

### 7.1 TypeScript 操场

[TypeScript 操场](http://www.typescriptlang.org/play/) 是 TypeScript 代码的在线编辑器。功能包括：

+   支持完整的 IDE 风格编辑：自动完成等。

+   显示静态类型错误。

+   显示将 TypeScript 代码编译成 JavaScript 和声明（`.d.ts`）的结果。

+   可以在浏览器中运行 JavaScript 输出。

Playground 对于快速实验和演示非常有用。它可以将 TypeScript 代码片段和编译器设置保存到 URL 中，这对于与他人分享此类片段非常棒。这是一个此类 URL 的示例：

```ts
https://www.typescriptlang.org/play/?#code/«base64»

```

许多社交媒体服务限制每条帖子的字符数，但不限制 URL 的字符数。因此，我们可以使用 Playground URL 来分享不适合帖子的代码。

### 7.2 通过 `node --watch` 创建简单的 TypeScript 操场

这是基本方法：

+   我们创建一个文件 `playground.mts`

    +   文件名扩展名为 `.mts`，这样我们就不需要 `package.json` 文件来告诉 Node.js `.ts` 表示 ESM 模块。

+   我们在终端中运行以下命令：

    ```ts
    node --watch ./playground.mts

    ```

+   我们编辑 `playground.mts`。每次我们保存它时，Node.js 都会重新运行它并显示其输出。

注意，Node.js 不会对代码进行类型检查。但我们在 TypeScript 编辑器中获得的类型检查应该在这种情况下足够了（因为我们只处理单个文件）。

#### 7.2.1 更多配置：`tsconfig.json` 和 `package.json`

对于更复杂的实验，我们可能需要两个额外的文件：

+   一个包含我们首选设置的 `tsconfig.json`

+   一个包含 `"type":"module"` 的 `package.json`，这样我们就可以使用文件名扩展名 `.ts`。

我已经创建了一个 GitHub 仓库 `nodejs-type-stripping`，其中两者都已正确设置。

#### 7.2.2 进一步阅读

+   相关的 Node.js 命令行选项包括：

    +   `--watch`（https://nodejs.org/api/cli.html#--watch）监视文件及其导入的更改，并在发生更改时重新运行文件。

    +   `--watch-path`（https://nodejs.org/api/cli.html#--watch-path）覆盖默认的监视导入，并告诉 Node.js 监视哪些文件。

    +   [`--watch-preserve-output`](https://nodejs.org/api/cli.html#--watch-preserve-output) 禁止在文件重新运行前清除控制台。

+   “直接运行 TypeScript（不生成 JS 文件）” (§8.6.2)

### 7.3 运行通过 Node.js 复制的 TypeScript 代码

对于简单的实验，只需复制 TypeScript 代码并通过 Node.js 运行即可：

```ts
«paste-command» | node --input-type=module-typescript

```

«paste-command» 依赖于您的操作系统：

+   MacOS: `pbpaste`

+   Windows PowerShell: `Get-Clipboard`

+   Linux: 有许多选项。在 Ubuntu 上，我通过应用中心（snap）安装了 [wl-clipboard](https://github.com/bugaevc/wl-clipboard)，它对我工作得很好。

在 macOS 上，我在我的 `.zprofile` 文件中添加了以下行：

```ts
alias pbts='pbpaste | node --input-type=module-typescript'

```
