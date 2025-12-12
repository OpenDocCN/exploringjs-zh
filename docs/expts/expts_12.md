# 9 使用 TypeScript 发布 npm 包

> 原文：[`exploringjs.com/ts/book/ch_typescript-npm-packages.html`](https://exploringjs.com/ts/book/ch_typescript-npm-packages.html)

（广告，请勿拦截。）

1.  9.1 文件系统布局

    1.  9.1.1 `.gitignore`

    1.  9.1.2 单元测试

1.  9.2 `tsconfig.json`

    1.  9.2.1 输出在哪里？

    1.  9.2.2 输出

1.  9.3 `package.json`

    1.  9.3.1 使用 `.js` 为 ESM 模块

    1.  9.3.2 应上传哪些文件到 npm 注册表？

    1.  9.3.3 包导出

    1.  9.3.4 包导入

    1.  9.3.5 包脚本

    1.  9.3.6 开发依赖

    1.  9.3.7 二进制脚本：用 JavaScript 编写的 shell 命令

1.  9.4 Linting npm 包

1.  9.5 进一步阅读

在本章中，我们将通过 TypeScript 创建一个基于 ESM 的 npm 库包：

+   从 TypeScript 4.7（2022-05-24）以来，所描述的设置一直对我很好。

+   我们只会使用 tsc，但设置已准备好用于其他工具。更多信息，请参阅“使用 tsc 之外的工具编译 TypeScript”（§8.8）。

+   如果你想要创建一个包含可执行文件而不是库的包，请查看“二进制脚本：用 JavaScript 编写的 shell 命令”（§9.3.7）。

![图标“GitHub”](img/25fa4b537bb7b3fd4dddaa93103f1bda.png) **示例包：**[`@rauschma/helpers`](https://github.com/rauschma/helpers)

此包使用本章中描述的设置。

### 9.1 文件系统布局

我们的 npm 包具有以下文件系统布局：

```ts
my-package/
 README.md
 LICENSE
 package.json
 tsconfig.json
 docs/
 api/
 src/
 test/
 dist/
 test/

```

注释：

+   通常包括一个 `README.md` 和一个 `LICENSE` 是一个好主意。

+   `package.json` 描述了包，将在后面描述。

+   `tsconfig.json` 配置 TypeScript，将在后面描述。

+   `docs/api/` 是通过 TypeDoc 生成的 API 文档。请参阅“通过文档注释和 TypeDoc 记录 TypeScript API”（§11）。

+   `src/` 是 TypeScript 源代码的位置。

+   `src/test/` 是用于集成测试的位置——跨越多个模块的测试。关于单元测试的更多信息即将到来。

    +   为什么我们不把 `src/` 和 `test/` 放在一起？那样会有一个负面后果，即输出文件会比输入文件在项目目录中更深地嵌套（更多信息）。

+   `dist/` 是 TypeScript 写入其输出的位置。

#### 9.1.1 `.gitignore`

我使用 Git 进行版本控制。这是我的 `.gitignore`（位于 `my-package/` 内）

```ts
node_modules
dist
.DS_Store

```

为什么有这些条目？

+   `node_modules`：目前最常见的做法似乎是不将 `node_modules` 目录提交到版本控制中。

+   `dist`: TypeScript 的编译输出不会提交到 Git，但会上传到 npm 注册表。关于这一点稍后详细说明。

+   `.DS_Store`：这个条目是关于我作为一个懒惰的 macOS 用户。由于它只在该操作系统上需要，你可以争论说 Mac 用户应该通过全局配置设置来添加它，并将其从项目特定的 gitignores 中排除。

#### 9.1.2 单元测试

我已经开始将特定模块的单元测试放在该模块旁边：

```ts
src/
 util.ts
 util_test.ts

```

由于单元测试有助于理解模块的工作方式，如果它们容易找到，那么它们就很有用。

##### 9.1.2.1 测试技巧：自我引用包

如果一个 npm 包有`"exports"`，它可以通过其包名来*自我引用*：

```ts
// src/misc/errors.ts
import {helperFunc} from 'my-package/misc/errors.js';

```

Node.js 文档有更多关于[自我引用的信息](https://nodejs.org/api/packages.html#self-referencing-a-package-using-its-name)，并指出：“只有当`package.json`有`"exports"`时，才能进行自我引用，并且只允许导入`"exports"`（在`package.json`中）允许的内容。”

自我引用的好处：

+   对于测试（可以展示导入包将如何使用代码）来说，这很有用。

+   它检查你的包导出是否设置正确。

### 9.2 `tsconfig.json`

“摘要：通过回答四个问题来组装`tsconfig.json`”（§8.12）通过询问我们四个问题帮助我们创建`tsconfig.json`文件。让我们为我们的 npm 包回答这些问题：

+   Q: 你是否希望将新的 JavaScript 转换为较旧的 JavaScript？

    +   A: 不。如果我们能限制自己只使用目标平台支持的 JavaScript 功能，那么输出会更简单，更接近输入。

+   Q: TypeScript 是否只应允许非类型级别的 JavaScript 功能？

    +   A: 是的，因为这使事情变得简单，并允许我们在需要时使用类型剥离。这是一种前瞻性的编写 TypeScript 的方式。

+   Q: 你想在本地导入中使用哪种文件名扩展名？

    +   A: ESM 需要文件名扩展名，而 TypeScript 传统上在转换期间不会更改模块指定符。因此，TypeScript ESM 代码最初使用`.js`。现在 TypeScript 转换可以将`.ts`转换为`.js`。因此，我们可以使用`.ts` – 优点是相同的代码在某些平台上（Node.js、Deno、Bun 等）也可以在不进行转换的情况下运行。唯一的缺点是当我们自我引用模块（见上一节）或使用`import()`时，我们仍然必须使用`.js`。

+   Q: tsc 应该输出哪些文件？

    +   A: `.js`、`.js.map`、`.d.ts`、`.d.ts.map`（关于这一点稍后详细介绍）

#### 9.2.1 输出在哪里？

`tsconfig.json`：

```ts
{
 "include": ["src/**/*"],
 "compilerOptions": {
 "rootDir": "src",
 "outDir": "dist",
 // ···
 }
}

```

这些设置的后果：

+   输入：`src/util.ts`

    +   输出：`dist/util.js`

+   输入：`src/test/integration_test.ts`

    +   输出：`dist/test/integration_test.js`

如果我们想将 `src/` 和 `test/` 放在一起，请参阅“将 `src/` 和 `test/` 放在一起”（§8.4.1.1） 获取更多信息。

#### 9.2.2 输出

给定一个 TypeScript 文件 `util.ts`，tsc 将以下输出写入 `dist/`：

```ts
src/
 util.ts
dist/
 util.js
 util.js.map
 util.d.ts
 util.d.ts.map

```

这些文件的目的：

+   `util.js`：包含在 `util.ts` 中的 JavaScript 代码

+   `util.js.map`：JavaScript 代码的 *源映射*。当运行 `util.js` 时，它启用以下功能：

    +   在调试器中，我们看到 TypeScript 代码。

    +   调试信息包含 TypeScript 源代码位置。

+   `util.d.ts`：在 `util.ts` 中定义的类型

+   `util.d.ts.map`：*声明映射* – `util.d.ts` 的源映射。它使得支持它的 TypeScript 编辑器（例如）能够跳转到类型定义的 TypeScript 源代码。我发现这对于库来说很有用。这就是为什么我将 TypeScript 源代码包含在它们的包中。

### 9.3 `package.json`

`package.json` 中的某些设置也会影响 TypeScript。我们将在下一节中查看这些设置。相关材料：

+   “使用 Node.js 进行 Shell 脚本编写”中的[“包：JavaScript 的软件分发单元”](https://exploringjs.com/nodejs-shell-scripting/ch_packages.html)章节提供了对 npm 包的全面概述。

+   您也可以查看 `@rauschma/helpers` 的 `package.json` 文件[链接](https://github.com/rauschma/helpers/blob/main/package.json)。

#### 9.3.1 使用 `.js` 为 ESM 模块

默认情况下，`.js` 文件被解释为 CommonJS 模块。以下设置让我们可以使用该文件扩展名来使用 ESM 模块：

```ts
"type": "module",

```

#### 9.3.2 应上传哪些文件到 npm 注册表？

我们必须指定应上传到 npm 注册表的文件。虽然也存在 `.npmignore` 文件，但明确列出要 *包含* 的内容更安全。这是通过 `package.json` 属性 `"files"` 实现的：

```ts
"files": [
 "package.json",
 "README.md",
 "LICENSE",

 "src/**/*.ts",

 "dist/**/*.js",
 "dist/**/*.js.map",
 "dist/**/*.d.ts",
 "dist/**/*.d.ts.map",

 "!src/test/",
 "!src/**/*_test.ts",

 "!dist/test/",
 "!dist/**/*_test.js",
 "!dist/**/*_test.js.map",
 "!dist/**/*_test.d.ts",
 "!dist/**/*_test.d.ts.map"
],

```

在 `.gitignore` 文件中，我们已经忽略了 `dist/` 目录，因为它包含可以自动生成的信息。然而，在这里我们明确地将其包含在内，因为其大部分内容必须包含在 npm 包中。

以感叹号 (`!`) 开头的模式定义了要排除的文件。在这种情况下，我们排除了测试：

+   其中一些位于 `src/` 的模块旁边。

+   剩余的测试位于 `src/test/`。

#### 9.3.3 包导出

如果我们想让包支持旧代码，有几个 `package.json` 属性，我们必须考虑：

+   `"main"`：之前由 Node.js 使用

+   `"module"`：之前由打包器使用

+   `"types"`：之前由 TypeScript 使用

+   `"typesVersions"`：之前由 TypeScript 使用

相比之下，对于现代代码，我们只需要：

```ts
"exports": {
 // Package exports go here
},

```

在我们深入了解细节之前，有两个问题我们必须考虑：

+   我们的包是否只通过裸导入导入，还是它将支持子路径导入？

    ```ts
    import {someFunc} from 'my-package'; // bare import
    import {someFunc} from 'my-package/sub/path'; // subpath import

    ```

+   如果我们导出子路径：它们是否将具有文件扩展名？

回答后一个问题的小贴士：

+   无扩展名风格有着悠久的历史。尽管 ESM 要求本地导入使用文件扩展名，但这一点并没有太大变化。

+   无扩展名风格的不利之处（引用[Node.js 文档](https://nodejs.org/api/packages.html#extensions-in-subpaths)）：“随着导入映射现在为浏览器和其他 JavaScript 运行时提供包解析的标准，使用无扩展名风格可能会导致导入映射定义膨胀。显式文件扩展名可以通过使导入映射能够利用包文件夹映射来映射多个子路径，而不是为每个包子路径导出单独的映射条目，从而避免这个问题。这也反映了在相对和绝对导入指定符中使用完整指定符路径的要求。”

我目前是这样决定的：

+   我的大部分包根本没有任何子路径。

+   如果包是模块的集合，我使用扩展名导出它们。

+   如果模块更像是包的不同版本（例如，同步与异步），那么我导出它们时不使用扩展名。

然而，我没有强烈的偏好，未来可能会改变主意。

##### 9.3.3.1 指定包导出

```ts
// Bare export
".": "./dist/main.js",

// Subpaths with extensions
"./misc/errors.js": "./dist/misc/errors.js", // single file
"./misc/*": "./dist/misc/*", // subtree

// Extensionless subpaths
"./misc/errors": "./dist/misc/errors.js", // single file
"./misc/*": "./dist/misc/*.js", // subtree

```

注意事项：

+   如果模块不多，多个单文件条目比一个子树条目更易于理解。

+   默认情况下，`.d.ts`文件必须位于`.js`文件旁边。但可以通过[`types`导入条件](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html#packagejson-exports-imports-and-self-referencing)来更改这一点。

关于这个主题的更多信息，请参阅“探索 JavaScript”中的[“包导出：控制其他包可见的内容”](https://exploringjs.com/nodejs-shell-scripting/ch_packages.html#package-exports-controlling-what-other-packages-see)部分。

#### 9.3.4 包导入

TypeScript 也支持 Node 的包导入。它们允许我们为路径定义别名。这些别名的优点是它们从包的顶级开始。以下是一个例子：

```ts
"imports": {
 "#root/*": "./*"
},

```

我们可以这样使用包导入：

```ts
import pkg from '#root/package.json' with { type: 'json' };
console.log(pkg.version);

```

包导入在 JavaScript 输出文件比 TypeScript 输入文件更深嵌套时特别有用。在这种情况下，我们无法使用相对路径来访问顶级文件。

关于包导入的更多信息，请参阅[Node.js 文档](https://nodejs.org/api/packages.html#imports)。

#### 9.3.5 包脚本

[包脚本](https://docs.npmjs.com/cli/v11/using-npm/scripts)允许我们为 shell 命令定义别名，例如`build`，并通过`npm run build`执行它们。我们可以通过`npm run`（不带脚本名称）来获取这些别名的列表。

这些是我认为对我的库项目有用的命令：

```ts
"scripts": {
 "\n========== Building ==========": "",
 "build": "npm run clean && tsc",
 "watch": "tsc --watch",
 "clean": "shx rm -rf ./dist/*",
 "\n========== Testing ==========": "",
 "test": "mocha --enable-source-maps --ui qunit",
 "testall": "mocha --enable-source-maps --ui qunit \"./dist/**/*_test.js\"",
 "\n========== Publishing ==========": "",
 "publishd": "npm publish --dry-run",
 "prepublishOnly": "npm run build"
},

```

说明：

+   `build`: 在每次构建之前，我都会清理目录 `dist/`。为什么？当重命名 TypeScript 文件时，旧输出文件不会被删除。这对于测试文件尤其成问题，并且经常让我头疼。每当这种情况发生时，我可以通过 `npm run build` 来修复问题。

+   `test`, `testall`:

    +   `--enable-source-maps` 启用 Node.js 中的源映射支持，因此堆栈跟踪中的行号更准确。

    +   测试运行器 [Mocha](https://mochajs.org) 支持多种测试风格。我更喜欢 `--ui qunit` ([示例](https://github.com/rauschma/helpers/blob/main/src/string/string_test.ts))。

+   `publishd`: 我们通过 `npm publish` 发布 npm 包。`npm run publishd` 调用该命令的“dry run”版本，它不会进行任何更改，但会提供有用的反馈——例如，它显示哪些文件将成为包的一部分。

+   `prepublishOnly`: 在 `npm publish` 将文件上传到 npm 注册表之前，它会调用此脚本。通过在发布前构建，我们确保不会上传过时文件和旧文件。

为什么使用命名的分隔符？这样做可以使 `npm run` 的输出更容易阅读。

#### 9.3.6 开发依赖项

即使我的包没有正常依赖项，它往往有以下开发依赖项：

```ts
"devDependencies": {
 "@types/mocha": "¹⁰.0.6",
 "@types/node": "²⁰.12.12",
 "mocha": "¹⁰.4.0",
 "shx": "⁰.3.4",
 "typedoc": "⁰.27.6"
},

```

说明：

+   `@types/node`: 在单元测试中，我使用 `node:assert` 进行断言，例如 `assert.deepEqual()`。此依赖项提供了该类型和其他 Node 模块的类型。

+   `shx`: 提供跨平台的 Unix shell 命令版本。我经常使用：

    ```ts
    shx rm -rf
    shx chmod u+x

    ```

我还在我的项目中本地安装以下两个命令行工具，以确保它们一定可用。`npm run` 的好处是它会将本地安装的命令添加到 shell 路径——这意味着它们可以在包脚本中使用，就像它们是全球安装的一样。

+   `mocha` 和 `@types/mocha`: 我仍然更喜欢 [Mocha](https://mochajs.org) 的 API 和 CLI 用户体验，但 [Node 的内置测试运行器](https://nodejs.org/api/test.html) 已成为有趣的替代方案。

+   `typedoc`: 我使用 TypeDoc 来生成 API 文档。

#### 9.3.7 Bin 脚本：用 JavaScript 编写的 shell 命令

`package.json` 可以包含 `"bin"` 属性，该属性设置可执行文件。查看我的项目 [TSConfigurator](https://github.com/rauschma/tsconfigurator) 以获取完整示例。该项目在 `package.json` 中具有以下属性：

```ts
"bin": {
 "tsconfigurator": "./dist/tsconfigurator.js"
},

```

我们可以使用 Node.js 版本来限制 bin 脚本的使用：

```ts
"engines": {
 "node": ">=23.6.0"
},

```

以下包脚本很有用（从 `"build"` 调用，在 `"tsc"` 之后）：

```ts
"scripts": {
 ···
 "chmod": "shx chmod u+x ./dist/tsconfigurator.js"
}

```

如果一个包提供可执行文件而不是库，我们不需要发出 `.d.ts` 文件。如果我们对 `.js` 使用类型剥离，我们可能也不需要 `.js.map` 文件。

### 9.4 检查 npm 包

检查包的公共接口：

+   [publint](https://publint.dev)：确保 npm 包在 Vite、Webpack、Rollup、Node.js 等各种环境中具有最广泛的兼容性。

+   [arethetypeswrong](https://github.com/arethetypeswrong/arethetypeswrong.github.io)：该项目试图分析 npm 包内容中 TypeScript 类型的问题，特别是与 ESM 相关的模块解析问题。

+   [npm-package-json-lint](https://npmpackagejsonlint.org)：`package.json` 文件的“可配置的检查器”

+   [vitest-package-exports](https://github.com/antfu/vitest-package-exports)：[...] 获取一个包的所有导出 API 并防止意外的破坏性更改。尽管其名称如此，此工具[不需要 Vitest](https://github.com/antfu/vitest-package-exports?tab=readme-ov-file#does-this-requires-vitest)。

在内部 linting npm 包：

+   [installed-check](https://github.com/voxpelli/node-installed-check)：验证已安装的模块是否符合 `package.json` 中指定的要求[Node.js 的 `engines` 版本范围](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#engines)。

+   [Knip](https://knip.dev)：查找并修复未使用的文件、依赖和导出。支持 JavaScript 和 TypeScript。

+   [Node Modules Inspector](https://github.com/antfu/node-modules-inspector)：可视化你的 `node_modules`，检查依赖项，等等。

+   [Madge](https://www.npmjs.com/package/madge)：创建模块依赖关系的可视化图，查找循环依赖，等等。

### 9.5 进一步阅读

+   JavaScript 模块（ESM）：在《探索 JavaScript》中，第 [“模块”](https://exploringjs.com/js/book/ch_modules.html) 章节中

+   npm 包：在《使用 Node.js 的 Shell 脚本》中，第 [“包：JavaScript 的软件分发单元”](https://exploringjs.com/nodejs-shell-scripting/ch_packages.html) 章节中

也很有用：

+   Node.js 文档中的 [“模块：包”](https://nodejs.org/api/packages.html) 章节内容

+   TypeScript 手册中的 [“`package.json "exports"`”](https://www.typescriptlang.org/docs/handbook/modules/reference.html#packagejson-exports) 部分
