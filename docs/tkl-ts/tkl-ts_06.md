# 四、TypeScript 是如何工作的？鸟瞰

> 原文：[`exploringjs.com/tackling-ts/ch_typescript-workflows.html`](https://exploringjs.com/tackling-ts/ch_typescript-workflows.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   4.1 TypeScript 项目的结构(ch_typescript-workflows.html#the-structure-of-typescript-projects)

    +   4.1.1 `tsconfig.json`

+   4.2 通过集成开发环境（IDE）编程 TypeScript

+   4.3 TypeScript 编译器生成的其他文件

    +   4.3.1 为了从 TypeScript 中使用 npm 包，我们需要类型信息

+   4.4 使用 TypeScript 编译器处理普通 JavaScript 文件

* * *

本章概述了 TypeScript 的工作原理：典型 TypeScript 项目的结构是什么？什么被编译了？如何使用 IDE 编写 TypeScript？

### 4.1 TypeScript 项目的结构

这是 TypeScript 项目的一种可能的文件结构：

```ts
typescript-project/
  dist/
  ts/
    src/
      main.ts
      util.ts
    test/
      util_test.ts
  tsconfig.json
```

解释：

+   目录`ts/`包含 TypeScript 文件：

    +   子目录`ts/src/`包含实际的代码。

    +   子目录`ts/test/`包含代码的测试。

+   目录`dist/`是编译器输出的存储位置。

+   TypeScript 编译器将`ts/`中的 TypeScript 文件编译为`dist/`中的 JavaScript 文件。例如：

    +   `ts/src/main.ts`被编译为`dist/src/main.js`（可能还有其他文件）

+   `tsconfig.json`用于配置 TypeScript 编译器。

#### 4.1.1 `tsconfig.json`

`tsconfig.json`的内容如下：

```ts
{
 "compilerOptions": {
 "rootDir": "ts",
 "outDir": "dist",
 "module": "commonjs",
 ···
 }
}
```

我们已经指定：

+   TypeScript 代码的根目录是`ts/`。

+   TypeScript 编译器保存其输出的目录是`dist/`。

+   输出文件的模块格式是 CommonJS。

### 4.2 通过集成开发环境（IDE）编程 TypeScript

JavaScript 的两个流行的 IDE 是：

+   [*Visual Studio Code*](https://code.visualstudio.com/)（免费）

+   [*WebStorm*](https://www.jetbrains.com/webstorm/)（需购买）

本节的观察结果是关于 Visual Studio Code 的，但也可能适用于其他 IDE。

一个重要的事实是，Visual Studio Code 以两种独立的方式处理 TypeScript 源代码：

+   检查打开文件的错误：这是通过所谓的[*语言服务器*](https://langserver.org/)完成的。语言服务器独立于特定的编辑器存在，并为 Visual Studio Code 提供与语言相关的服务：检测错误、重构、自动补全等。与服务器的通信通过基于 JSON-RPC 的协议进行（*RPC*代表*远程过程调用*）。该协议提供的独立性意味着服务器可以用几乎任何编程语言编写。

    +   要记住的重要事实是：语言服务器只列出当前打开文件的错误，不会编译 TypeScript，它只是静态分析。

+   *构建*（将 TypeScript 文件编译为 JavaScript 文件）：在这里，我们有两种选择。

    +   我们可以通过外部命令运行构建工具。例如，TypeScript 编译器`tsc`有一个`--watch`模式，可以在输入文件发生更改时将其编译为输出文件。因此，每当我们在 IDE 中保存一个 TypeScript 文件时，我们立即得到相应的输出文件。

    +   我们可以在 Visual Studio Code 中运行`tsc`。为了做到这一点，它必须安装在我们当前正在工作的项目内部，或者全局安装（通过 Node.js 包管理器 npm）。通过构建，我们可以获得完整的错误列表。有关在 Visual Studio Code 中编译 TypeScript 的更多信息，请参阅[该 IDE 的官方文档](https://code.visualstudio.com/docs/typescript/typescript-compiling)。

### 4.3 TypeScript 编译器生成的其他文件

给定一个 TypeScript 文件`main.ts`，TypeScript 编译器可以生成几种不同的产物。最常见的是：

+   JavaScript 文件：`main.js`

+   声明文件：`main.d.ts`（包含类型信息；类似于`.ts`文件但不包含 JavaScript 代码）

+   源映射文件：`main.js.map`

TypeScript 通常不是通过`.ts`文件交付的，而是通过`.js`文件和`.d.ts`文件：

+   JavaScript 代码包含实际的功能，并且可以通过纯 JavaScript 来使用。

+   声明文件可以帮助编程编辑器进行自动补全和类似的服务。这些信息使得纯 JavaScript 可以通过 TypeScript 来消耗。然而，即使我们使用纯 JavaScript，也可以从中受益，因为它可以提供更好的自动补全和更多功能。

源映射为`main.js`中的每个部分指定了`main.ts`中的哪个部分产生了它。除其他外，这些信息使得运行时环境可以执行 JavaScript 代码，同时显示 TypeScript 代码的行号在错误消息中。

#### 4.3.1 为了从 TypeScript 中使用 npm 包，我们需要类型信息

npm 注册表是一个庞大的 JavaScript 代码存储库。如果我们想要在 TypeScript 中使用 JavaScript 包，我们需要它的类型信息：

+   包本身可能包括`.d.ts`文件，甚至是完整的 TypeScript 代码。

+   如果没有，我们可能仍然可以使用它：[DefinitelyTyped](https://definitelytyped.org/)是一个为纯 JavaScript 包编写声明文件的存储库。

DefinitelyTyped 的声明文件位于`@types`命名空间中。因此，如果我们需要一个像`lodash`这样的包的声明文件，我们必须安装`@types/lodash`包。

### 4.4 使用 TypeScript 编译器处理纯 JavaScript 文件

TypeScript 编译器也可以处理纯 JavaScript 文件：

+   使用`--allowJs`选项，TypeScript 编译器会将输入目录中的 JavaScript 文件复制到输出目录中。好处是：当从 JavaScript 迁移到 TypeScript 时，我们可以从 JavaScript 和 TypeScript 文件的混合开始，然后慢慢将更多的 JavaScript 文件转换为 TypeScript。

+   使用`--checkJs`选项，编译器还会对 JavaScript 文件进行类型检查（此选项需要`--allowJs`开启才能工作）。它会尽可能地进行类型检查，考虑到可用的有限信息。可以通过文件内的注释来配置哪些文件需要进行检查：

    +   显式排除：如果 JavaScript 文件包含注释`// @ts-nocheck`，则不会进行类型检查。

    +   显式包含：如果没有`--checkJs`，则可以使用注释`// @ts-check`来对单个 JavaScript 文件进行类型检查。

+   TypeScript 编译器使用通过 JSDoc 注释指定的静态类型信息（请参见下面的示例）。如果我们认真对待，我们可以完全静态地对纯 JavaScript 文件进行类型标注，甚至可以从中派生出声明文件。

+   使用`--noEmit`选项，编译器不会产生任何输出，只进行文件的类型检查。

这是一个 JSDoc 注释的示例，为函数`add()`提供了静态类型信息：

```ts
/**
 * @param  {number} x - The first operand
 * @param  {number} y - The second operand
 * @returns {number} The sum of both operands
 */
function add(x, y) {
 return x + y;
}
```

更多信息：[Type-Checking JavaScript Files](https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html) 在 TypeScript 手册中。

[评论](https://github.com/rauschma/tackling-ts/issues/4)
