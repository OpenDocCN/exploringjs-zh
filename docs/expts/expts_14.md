# 11 使用文档注释和 TypeDoc 记录 TypeScript API

> 原文：[`exploringjs.com/ts/book/ch_jsdoc.html`](https://exploringjs.com/ts/book/ch_jsdoc.html)

（广告，请勿拦截。）

1.  11.1 文档注释

    1.  11.1.1 JSDoc 注释

    1.  11.1.2 TSDoc 注释

    1.  11.1.3 TypeDoc

1.  11.2 生成文档

1.  11.3 从文档注释中引用文件的部分

    1.  11.3.1 为什么这很有用？

    1.  11.3.2 真实世界的例子

1.  11.4 进一步阅读

本章描述了如何通过 *文档注释*（多行注释，其内容遵循标准格式）来记录 TypeScript API。我们将使用可安装的 npm 命令行工具 [TypeDoc](https://typedoc.org) 来完成这项任务。

### 11.1 文档注释

#### 11.1.1 JSDoc 注释

在 JavaScript 中记录 API 的默认标准是[JSDoc 注释](https://jsdoc.app)（其灵感来源于[Javadoc 注释](https://en.wikipedia.org/wiki/Javadoc)）。它们是多行注释，其中开界定符后面跟着第二个星号：

```ts
/**
 * Adds two numbers
 * 
 * @param {number} x - The first operand
 * @param {number} y - The second operand
 * @returns {number} The sum of both operands
 */
function add(x: number, y: number): number {
 return x + y;
}

```

参数名称如`x`之后的连字符是可选的。

TypeScript 本身支持 JSDoc 注释作为在纯 JavaScript 代码中指定类型的方式（[更多信息](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html)）。

#### 11.1.2 TSDoc 注释

[TSDoc](https://tsdoc.org) 适应 JSDoc 注释，以便它们更适合 TypeScript – 例如，注释中的类型是不允许的，连字符是强制性的：

```ts
/**
 * Adds two numbers
 * 
 * @param x - The first operand
 * @param y - The second operand
 * @returns The sum of both operands
 */
function add(x: number, y: number): number {
 return x + y;
}

```

#### 11.1.3 TypeDoc

API 文档生成器 [TypeDoc](https://typedoc.org) 使用文档注释来生成 HTML API 文档。TSDoc 注释是首选，但也支持 JSDoc 注释。TypeDoc 的功能包括：

+   支持在文档注释中使用 Markdown，包括代码块的语法高亮。

+   文档注释可以引用外部文件中的代码片段 – 这可以更容易地进行测试。

+   从用 Markdown 编写的文件生成的网页（[“外部文档”](https://typedoc.org/documents/External_Documents.html)）。

### 11.2 生成文档

我们可以使用以下`package.json`脚本来将 TSDoc 注释转换为 API 文档：

```ts
"scripts": {
 "\n========== TypeDoc ==========": "",
 "api": "shx rm -rf docs/api/ && typedoc --out docs/api/ --readme none
 --entryPoints src --entryPointStrategy expand --exclude '**/*_test.ts'",
},

```

`"api"` 的条目是一行；我将其拆分以便更好地显示。

作为补充措施，我们可以从 `docs/` 提供 GitHub 页面：

+   仓库中的文件：`my-package/docs/api/index.html`

+   在线文件（用户`robin`）：`https://robin.github.io/my-package/api/index.html`

您可以在线查看`@rauschma/helpers`的 API 文档（警告：文档尚不完整）。

### 11.3 从文档注释中引用文件的部分

自版本 0.27.7 以来，TypeDoc 允许我们通过 doc 标签 `{@includeCode}` 引用外部文件的部分：

文件 `util.ts`：

```ts
/**
 * {@includeCode ./util_test.ts#utilFunc}
 */
function utilFunc(): void {}

```

注意文件路径后的哈希 (`#`) 和名称 `utilFunc`：它指的是 `util_test.ts` 内的 *区域*。区域是通过注释标记源文件中行序列的一种方式。Visual Studio Code 也支持区域，它们可以被折叠（[文档](https://code.visualstudio.com/docs/editor/codebasics#_folding)）。

这就是 `util_test.ts` 内区域的样貌：

```ts
test('utilFunc', () => {
 //#region utilFunc
 // ...
 //#endregion utilFunc
});

```

#### 11.3.1 为什么这很有用？

文件名已经暗示了此功能的使用场景：它使我们能够发布包含所有代码示例（如区域 `utilFunc`）的文档的文档。

在过去，TypeDoc 只允许我们包含完整文件，这意味着每个示例一个文件 – 测试模板在文档中显示。

#### 11.3.2 现实世界示例

+   以下文件是来自现实世界代码的摘录。

+   在线查看结果：[Function arrayToChunks](https://rauschma.github.io/helpers/api/functions/collection_array.arrayToChunks.html)

    +   另一个示例：[Class UnexpectedValueError](https://rauschma.github.io/helpers/api/classes/typescript_error.UnexpectedValueError.html)

文件 `array.ts`：

```ts
/**
 * Split `arr` into chunks with length `chunkLen` and return them
 * in an Array.
 * {@includeCode ./array_test.ts#arrayToChunks}
 */
export function arrayToChunks<T>(
 arr: Array<T>, chunkLen: number
): Array<Array<T>> {
 // ···
}

```

文件 `array_test.ts`：

```ts
// ···
test('arrayToChunks', () => {
 //#region arrayToChunks
 const arr = ['a', 'b', 'c', 'd'];
 assert.deepEqual(
 arrayToChunks(arr, 1),
 [['a'], ['b'], ['c'], ['d']],
 );
 //#endregion arrayToChunks

 assert.deepEqual(
 arrayToChunks(arr, 2),
 [['a', 'b'], ['c', 'd']],
 );
});

```

### 11.4 进一步阅读

+   [关于 `{@includeCode}` 的文档](https://typedoc.org/documents/Tags.__include_.html)

+   相关工具：

    +   [Markcheck](https://github.com/rauschma/markcheck) 检查 Markdown 文件中的代码示例。

    +   [jsdoctest](https://github.com/yamadapc/jsdoctest) 运行 JSDoc 示例（用 JavaScript 编写）作为 doctests。
