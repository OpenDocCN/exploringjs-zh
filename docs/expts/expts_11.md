# 8 tsconfig.json 指南

> 原文：[`exploringjs.com/ts/book/ch_tsconfig-json.html`](https://exploringjs.com/ts/book/ch_tsconfig-json.html)

(广告，请勿拦截。)

1.  8.1 本章未涵盖的功能

1.  8.2 通过 `extends` 扩展基本文件

1.  8.3 输入文件在哪里？

1.  8.4 输出是什么？

    1.  8.4.1 输出文件在哪里写入？

    1.  8.4.2 输出源映射

    1.  8.4.3 输出 `.d.ts` 文件（例如，用于库）

    1.  8.4.4 微调输出文件

1.  8.5 语言和平台功能

    1.  8.5.1 `target`

    1.  8.5.2 `lib`

    1.  8.5.3 `skipLibCheck`

    1.  8.5.4 内置 Node.js API 的类型

1.  8.6 模块系统

    1.  8.6.1 TypeScript 是如何查找导入模块的？

    1.  8.6.2 直接运行 TypeScript（不生成 JS 文件）

    1.  8.6.3 导入 JSON

    1.  8.6.4 导入其他非 TypeScript 艺术品

1.  8.7 类型检查

    1.  8.7.1 `strict`

    1.  8.7.2 `exactOptionalPropertyTypes`

    1.  8.7.3 `noFallthroughCasesInSwitch`

    1.  8.7.4 `noImplicitOverride`

    1.  8.7.5 `noImplicitReturns`

    1.  8.7.6 `noPropertyAccessFromIndexSignature`

    1.  8.7.7 `noUncheckedIndexedAccess`

    1.  8.7.8 具有良好默认值的类型检查选项

1.  8.8 使用除 tsc 之外的工具编译 TypeScript

    1.  8.8.1 仅使用 tsc 进行类型检查

    1.  8.8.2 通过类型剥离生成 `.js` 文件：`erasableSyntaxOnly` 和 `verbatimModuleSyntax`

    1.  8.8.3 `erasableSyntaxOnly`：不转换语言功能

    1.  8.8.4 `verbatimModuleSyntax`：在导入和导出中强制执行 `type`

    1.  8.8.5 `isolatedDeclarations`：更有效地生成 `.d.ts` 文件

1.  8.9 从 ESM 导入 CommonJS

    1.  8.9.1 `allowSyntheticDefaultImports`：检查 CommonJS 模块的默认导入

    1.  8.9.2 `esModuleInterop`：更好的 TypeScript 到 CommonJS 代码的编译

1.  8.10 一个有良好默认值的额外选项

1.  8.11 Visual Studio Code

1.  8.12 总结：通过回答四个问题来组装你的 `tsconfig.json`

    1.  8.12.1 你是否想将新的 JavaScript 转译为旧版本的 JavaScript？

    1.  8.12.2 TypeScript 是否只应允许非类型级别的 JavaScript 功能？

    1.  8.12.3 在本地导入中你想使用哪种文件名扩展？

    1.  8.12.4 tsc 应该输出哪些文件？

1.  8.13 进一步阅读

    1.  8.13.1 其他人的 `tsconfig.json` 建议

    1.  8.13.2 本章的来源

![图标“详情”](img/837806d7ec89826c3784b2e685feb762.png) **版本：TypeScript 5.8**

本章涵盖了 TypeScript 5.8 支持的 `tsconfig.json`。

本章记录了 TypeScript 配置文件 `tsconfig.json` 的所有常见选项：

+   这项知识将使你能够理解和简化你的 `tsconfig.json`。

+   如果你没有时间阅读本章，你可以跳转到末尾的总结，在那里我展示了一个包含所有设置的起始 `tsconfig.json` 文件——以及四个问题，以确定你可以删除哪些设置。

+   我还链接到了几位知名 TypeScript 程序员提供的`tsconfig.json` 建议。（我在研究本章时查阅了它们。）

### 8.1 本章未涵盖的功能

本章仅描述如何设置所有本地模块都是 ESM 的项目。尽管如此，它也提供了一些导入 CommonJS 的技巧。

未在此处解释：

+   在你的代码库中导入和类型检查纯 JavaScript，即选项 `allowJs` 和 `checkJs`。

+   如何设置 JSX。请参阅 TypeScript 手册中的[“JSX”](https://www.typescriptlang.org/docs/handbook/jsx.html)。

+   “项目”（对 monorepos 有用）：选项 `composite` 等。有关此主题的更多信息，请参阅：

    +   TypeScript 手册中的[“项目引用”](https://www.typescriptlang.org/docs/handbook/project-references.html)章节

    +   我的博客文章[“通过 npm workspaces 和 TypeScript 项目引用创建简单的 monorepos”](https://2ality.com/2021/07/simple-monorepos.html)

### 8.2 通过 `extends` 扩展基础文件

此选项允许我们通过模块规范引用现有的 `tsconfig.json`（就像我们导入一个 JSON 文件一样）。该文件成为我们的 tsconfig 的 *基础*，它扩展了我们的 tsconfig。这意味着我们的 tsconfig 包含了基础的所有选项，但可以覆盖其中任何一个，也可以添加基础中没有提到的选项。

GitHub 仓库[tsconfig/bases](https://github.com/tsconfig/bases)列出了在 npm 命名空间 `@tsconfig` 下可用的基础，并且可以像这样使用（在它们通过 npm 本地安装后）：

```ts
{
 "extends": "@tsconfig/node-lts/tsconfig.json",
}

```

殊可惜，这些文件都不符合我的需求。但它们可以作为你 tsconfig 的灵感来源。

### 8.3 输入文件在哪里？

```ts
{
 "include": ["src/**/*"],
}

```

一方面，我们必须告诉 TypeScript 输入文件是什么。这些是可用的选项：

+   `files`：所有输入文件的详尽数组

+   `include`：通过一个模式数组指定输入文件，这些模式被解释为相对于 `tsconfig.json` 的相对路径。

+   `exclude`：指定应从 `include` 文件集中排除哪些文件 - 通过一个模式数组。

### 8.4 输出是什么？

#### 8.4.1 输出文件在哪里写入？

```ts
"compilerOptions": {
 "rootDir": "src",
 "outDir": "dist",
}

```

TypeScript 如何确定输出文件的写入位置：

+   它采用输入路径（相对于 `tsconfig.json`），

+   移除由 `rootDir` 指定的前缀

+   “追加”结果到 `outDir`。

例如，考虑以下 `tsconfig.json`：

```ts
{
 "include": ["src/**/*"],
 "compilerOptions": {
 // Specify explicitly (don’t derive from source file paths):
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

##### 8.4.1.1 将 `src/` 和 `test/` 放在一起

我喜欢有一个单独的目录 `test/`，它是 `src/` 的兄弟目录。然而，`dist/` 中的输出文件在项目目录中比 `src/` 和 `test/` 中的输入文件更深地嵌套。这意味着我们不能通过相对模块规范访问文件，如 `package.json`。

`tsconfig.json`：

```ts
{
 "include": ["src/**/*", "test/**/*"],
 "compilerOptions": {
 "rootDir": ".",
 "outDir": "dist",
 // ···
 }
}

```

这些设置的后果：

+   输入：`src/util.ts`

    +   输出：`dist/src/util.js`

+   输入：`test/integration_test.ts`

    +   输出：`dist/test/integration_test.js`

##### 8.4.1.2 `rootDir` 的默认值

`rootDir` 的默认值取决于输入文件路径。我发现这太不可预测了，所以我总是明确指定它。它是输入文件路径的最长公共前缀。

**示例 1**：默认值是 `'src'`（相对于项目目录）

`tsconfig.json`：

```ts
{
 "include": ["src/**/*"],
}

```

文件：

```ts
/tmp/my-proj/
 tsconfig.json
 src/
 main.ts
 test/
 test.ts
 dist/
 main.js
 test/
 test.js

```

**示例 2**：默认值是 `'src/core/cli'`

`tsconfig.json`：

```ts
{
 "include": ["src/**/*"],
}

```

文件：

```ts
/tmp/my-proj/
 tsconfig.json
 src/
 core/
 cli/
 main.ts
 test/
 test.ts
 dist/
 main.js
 test/
 test.js

```

**示例 3**：

`tsconfig.json`：默认值是 `'.'`

```ts
{
 "include": ["src/**/*", "test/**/*"],
}

```

文件：

```ts
/tmp/my-proj/
 tsconfig.json
 src/
 main.ts
 test/
 test.ts
 dist/
 src/
 main.js
 test/
 test.js

```

#### 8.4.2 输出源映射

```ts
"compilerOptions": {
 "sourceMap": true,
}

```

`sourceMap` 生成指向从转换后的 JavaScript 到原始 TypeScript 的源映射文件。这有助于调试，通常是个好主意。

#### 8.4.3 输出 `.d.ts` 文件（例如，用于库）

如果我们想让 TypeScript 代码消费我们的转换后的 TypeScript 代码，我们通常应该包括 `.d.ts` 文件：

```ts
"compilerOptions": {
 "declaration": true,
 "declarationMap": true, // enables importers to jump to source
}

```

可选地，我们可以在我们的 npm 包中包含 TypeScript 源代码并激活 `declarationMap`。然后导入者可以，例如，点击类型或转到值的定义，他们的编辑器将带他们到原始源代码。

##### 8.4.3.1 选项 `declarationDir`

默认情况下，每个 `.d.ts` 文件都放在其 `.js` 文件旁边。如果您想更改这一点，可以使用选项 `['declarationDir'](https://www.typescriptlang.org/tsconfig/#declarationDir)`。

#### 8.4.4 微调输出文件

```ts
"compilerOptions": {
 "newLine": "lf",
 "removeComments": false,
}

```

上面的值是默认值。

+   `newLine` 配置输出文件的换行符。允许的值有：

    +   `"lf"`: "n" (Unix)

    +   `"crlf"`: "rn" (Windows)

+   `removeComments`: 如果启用，TypeScript 文件中的所有注释在转换后的 JavaScript 文件中都将被省略。我倾向于保持默认设置，不删除注释：

    +   这有助于阅读转换后的 JavaScript——特别是如果 TypeScript 源代码没有包含在内。

    +   打包器会删除注释。

    +   在 Node.js 上，增加的负担并不重要。

### 8.5 语言和平台特性

```ts
"compilerOptions": {
 "target": "ESNext", // sets up "lib" accordingly
 "skipLibCheck": true,
}

```

#### 8.5.1 `target`

`target` 决定了哪些较新的 JavaScript 语法会被转换成旧语法。例如，如果目标为 `"ES5"`，则箭头函数 `() => {}` 会被转换成函数表达式 `function () {}`。可用的值包括：

+   `"ESNext"`

+   `"ES5"`

+   `"ES6"`

+   `"ES2015"` (等同于 `"ES6"`)

+   `"ES2016"`

+   等等。

`"ESNext"` 表示永远不会进行转换。我发现这种设置最容易处理。如果您不使用 `tsc` 并使用类型剥离（这也永远不会进行转换），这也是最好的设置。

##### 8.5.1.1 如何选择好的 `"ES20YY"` 目标

如果我们要进行转换，我们必须选择一个适用于我们的目标平台的 ECMAScript 版本。有两个表格提供了很好的概述：

+   对于浏览器：[兼容性表格](https://compat-table.github.io/compat-table/es2016plus/)

+   对于 Node.js：[node.green](https://node.green/)

此外，[官方 tsconfig 基础配置](https://github.com/tsconfig/bases)都为 `target` 提供了值。

#### 8.5.2 `lib`

`lib` 决定了哪些内置 API 的类型可用——例如 `Math` 或内置类型的函数：

+   有如 `"ES2024"` 和 `"DOM"` 这样的类别，以及如 `"DOM.Iterable"` 和 `"ES2024.Promise"` 这样的子类别。

+   哪些值可用？我们可以在以下位置查找：

    +   自动完成（例如在 Visual Studio Code 中）

    +   [TypeScript 文档](https://www.typescriptlang.org/tsconfig/#lib)

    +   [TypeScript 源代码仓库](https://github.com/microsoft/TypeScript/tree/main/src/lib)

+   值不区分大小写：Visual Studio Code 的自动完成建议包含许多大写字母；文件名则不包含。`lib` 值可以以任何一种方式书写。

TypeScript 在何时支持某个特定的 API？它必须在至少 2 个浏览器 *引擎* 中无前缀/标记地可用（即不仅仅是 2 个铬浏览器）([来源](https://github.com/microsoft/TypeScript/tree/main/src/lib))。

##### 8.5.2.1 通过 `target` 设置 `lib`

`target` 决定了 `lib` 的默认值：如果后者被省略且 `target` 是 `"ES20YY"`，则使用 `"ES20YY.Full"`。然而，这不是我们可以使用的值。如果我们想复制移除 `lib` 的效果，我们必须自己列出（例如）`es2024.full.d.ts` 在 [TypeScript 源代码仓库](https://github.com/microsoft/TypeScript/tree/main/src/lib) 中的内容：

```ts
/// <reference lib="es2024" />
/// <reference lib="dom" />
/// <reference lib="webworker.importscripts" />
/// <reference lib="scripthost" />
/// <reference lib="dom.iterable" />
/// <reference lib="dom.asynciterable" />

```

在这个文件中，我们可以观察到一种有趣的现象：

+   类别 `"ES20YY"` 通常包括其所有子类别。

+   类别 `"DOM"` 还不包括 - 例如，子类别 `"DOM.Iterable"` 还不是它的一部分。

在其他方面，`"DOM.Iterable"` 允许遍历 NodeList – 例如：

```ts
for (const x of document.querySelectorAll('div')) {}

```

#### 8.5.3 `skipLibCheck`

+   `skipLibCheck:false` – 默认情况下，TypeScript 会检查所有的 `.d.ts` 文件。这通常不是必需的，但当一个项目包含手写的 `.d.ts` 文件时，这很有帮助。

+   `skipLibCheck:true` – 如果我们将其关闭，那么 TypeScript 将只检查我们代码中使用的库功能。这可以节省时间 – 这也是为什么我选择了 `true`。

#### 8.5.4 内置 Node.js API 的类型

Node.js API 的类型必须通过 [一个 npm 包](https://www.npmjs.com/package/@types/node) 安装：

```ts
npm install @types/node

```

### 8.6 模块系统

#### 8.6.1 TypeScript 如何查找导入的模块？

这些选项影响 TypeScript 查找导入模块的方式：

```ts
"compilerOptions": {
 "module": "NodeNext",
 "noUncheckedSideEffectImports": true,
}

```

##### 8.6.1.1 选项 `module`

使用此选项，我们指定处理模块的系统。如果我们正确设置它，我们也会照顾到相关的选项 `moduleResolution`，它为此提供了良好的默认值。TypeScript 文档建议以下两个值之一：

+   Node.js: `"NodeNext"` 支持 CommonJS 和最新的 ESM 功能。

    +   暗示 `"moduleResolution": "NodeNext"`

    +   `"NodeNext"` 的缺点：它是一个移动的目标。但一般来说，功能只会增加。

    +   `"NodeNext"` 的优势：它支持良好的功能组合 – 例如 ([来源](https://github.com/microsoft/TypeScript/pull/60761))：

        +   `"Node16"` 不支持导入属性（这些属性对于导入 JSON 文件是必需的）。

        +   `"Node18"` 和 `"Node20"` 支持过时的导入断言。

        +   `require(esm)`（这对于 CommonJS 代码相关，但不适用于 ESM 代码）仅由 `"Node20"` 和 `"NodeNext"` 支持。

+   打包器: `"Preserve"` 支持 CommonJS 和最新的 ESM 功能。它与大多数打包器所做的一致。

    +   暗示 `"moduleResolution": "bundler"`

由于打包器大多模仿 Node.js 的行为，我总是使用 `"NodeNext"` 并没有遇到任何问题。

注意，在这两种情况下，TypeScript 都强迫我们提及我们导入的本地模块的完整名称。我们不能省略文件扩展名，正如当 Node.js 仅编译为 CommonJS 时常见的做法。

`module:NodeNext`意味着`target:ESNext`，但在此情况下，我更喜欢手动设置`target`，因为`module`和`target`并不像`module`和`moduleResolution`那样紧密相关。此外，`module:Bundler`并不表示任何内容。

##### 8.6.1.2 选项`noUncheckedSideEffectImports`

默认情况下，如果不存在空导入，TypeScript 不会报错。这种行为的原因是，这是某些打包器支持的一种模式，用于将非 TypeScript 工件与模块关联。而 TypeScript 只看到 TypeScript 文件。这样的导入看起来是这样的：

```ts
import './component-styles.css';

```

有趣的是，TypeScript 通常也对空导入的 TypeScript 文件表示默许。只有在从不存在文件导入时，它才会报错。

```ts
import './does-not-exist.js'; // no error!

```

将`noUncheckedSideEffectImports`设置为`true`会改变这一点。我将在稍后解释导入非 TypeScript 工件的一个替代方案。

#### 8.6.2 直接运行 TypeScript（不生成 JS 文件）

```ts
"compilerOptions": {
 "allowImportingTsExtensions": true,
 // Only needed if compiling to JavaScript:
 "rewriteRelativeImportExtensions": true,
}

```

大多数非浏览器 JavaScript 平台现在可以直接运行 TypeScript 代码，无需进行转译。

这主要影响我们在导入本地模块时使用的文件名扩展名。传统上，TypeScript 不会更改模块指定符，因此我们必须在 ESM 模块中使用文件名扩展名`.js`（这是我们的 TypeScript 编译成的 JavaScript 所使用的）：

```ts
import {someFunc} from './lib/utilities.js';

```

如果我们直接运行 TypeScript，那么这个导入语句看起来是这样的：

```ts
import {someFunc} from './lib/utilities.ts';

```

这可以通过以下设置启用：

+   `allowImportingTsExtensions`：如果此选项处于活动状态，TypeScript 不会报错，如果我们使用文件名扩展名`.ts`。

+   `rewriteRelativeImportExtensions`：通过此选项，我们还可以转译旨在直接运行的 TypeScript 代码。默认情况下，TypeScript 不会更改导入的模块指定符。此选项附带一些注意事项：

    +   只有相对路径会被重写。

    +   它们会被“天真地”重写——不考虑`baseUrl`和`paths`选项（这些选项超出了本章的范围）。

    +   在`package.json`中的`"exports"`和`"imports"`属性中路由的路径看起来不像相对路径，因此也不会被重写。

相关选项：

+   如果你只想使用 tsc 进行类型检查，那么请查看`noEmit`选项。

##### 8.6.2.1 Node 对 TypeScript 的内置支持

Node.js 现在通过*类型剥离*支持 TypeScript：

+   关于类型剥离和帮助其的选项`erasableSyntaxOnly`的更多信息

+   [Node 对 TypeScript 支持的官方文档](https://nodejs.org/api/typescript.html)

#### 8.6.3 导入 JSON

```ts
"compilerOptions": {
 "resolveJsonModule": true,
}

```

选项`resolveJsonModule`使我们能够导入 JSON 文件：

```ts
import data from './data.json' with {type: 'json'};
console.log(data.version);

```

#### 8.6.4 导入其他非 TypeScript 工件

每当我们导入一个 TypeScript 不认识的扩展名为 `ext` 的文件 `basename.ext` 时，它会寻找一个 `basename.d.ext.ts` 文件。如果找不到，它会引发错误。TypeScript 文档有一个[很好的例子](https://www.typescriptlang.org/tsconfig/#allowArbitraryExtensions)说明此类文件可能的样子。

我们有两种方法可以防止 TypeScript 对未知导入引发错误。

首先，我们可以使用选项 `allowArbitraryExtensions` 来防止在这种情况下报告任何类型的错误。

第二种方法，我们可以创建一个带有通配符指定符的 *环境模块声明* —— 一个必须位于 TypeScript 所知文件中的 `.d.ts` 文件。以下示例抑制了所有具有文件扩展名 `.css` 的导入的错误：

```ts
// ./src/globals.d.ts
declare module "*.css" {}

```

### 8.7   类型检查

```ts
"compilerOptions": {
 "strict": true,
 "exactOptionalPropertyTypes": true,
 "noFallthroughCasesInSwitch": true,
 "noImplicitOverride": true,
 "noImplicitReturns": true,
 "noPropertyAccessFromIndexSignature": true,
 "noUncheckedIndexedAccess": true,
}

```

在我看来，`strict` 是必须的。在剩余的设置中，你必须自己决定是否想要为你的代码添加额外的严格性。你可以从添加所有这些设置开始，看看哪些设置给你带来了太多麻烦。

#### 8.7.1   `strict`

编译器设置 `strict` 为类型检查提供了一个重要的最小设置。原则上，此设置默认为 `true`，但由于向后兼容性，这变得不可能。

![图标“提示”](img/0873709827ba4924e4afbb757e47a4df.png)   **编译器选项 `strict` 简要说明**

`strict`基本上意味着：尽可能多地、尽可能正确地进行类型检查。

`strict` 激活以下设置（本章节中不再提及）：

+   `alwaysStrict`: 总是在脚本文件中发出 `"use strict"`。这是一个过时的 JavaScript 功能，在 ECMAScript 模块中不再需要。

+   `noImplicitAny`: 如果设置为 `true`，我们可以在某些位置（主要是参数定义）省略类型，TypeScript 将（隐式地）推断类型 `any`。如果设置为 `false`，我们必须提供显式类型注解——这可以使用类型 `any`（显式地）。有关更多信息，请参阅“编译器选项 `noImplicitAny`”（§14.2.3）。

+   `noImplicitThis`: 如果我们在普通函数中使用 `this`，我们必须显式声明其类型。

+   `strictBindCallApply`: 如果设置为 `true`，TypeScript 将检查我们是否向 `.call()`、`.apply()` 和 `.bind()` 方法传递了正确的参数。如果设置为 `false`，我们可以向这些方法传递任何参数。

+   `strictBuiltinIteratorReturn`: 如果激活，内置迭代器具有 `TReturn` 类型 `undefined`（而不是 `any`）。

+   `strictFunctionTypes`: 如果设置为 `true`，则函数类型之间的兼容性将得到更正确的处理。

+   `strictNullChecks`: 如果设置为 `true`，则值 `undefined` 和 `null` 不是正常类型 `T` 的元素。如果我们想接受它们，我们必须使用类型 `undefined | T` 或 `null | T`。

+   `strictPropertyInitialization`：如果为 `true`，TypeScript 会警告我们在构造函数中没有初始化类实例属性。更多信息，请参阅“严格的属性初始化”（§21.4.1）。

+   `useUnknownInCatchVariables`：如果为 `true`，TypeScript 给 `catch` 变量赋予没有类型注解的类型 `unknown`（而不是 `any`）。

#### 8.7.2 `exactOptionalPropertyTypes`

如果为 `true`，则 `.colorTheme` 只能省略，不能设置为 `undefined`：

```ts
interface Settings {
 // Absent property means “system”
 colorTheme?: 'dark' | 'light';
}
const obj1: Settings = {}; // allowed
// @ts-expect-error: Type '{ colorTheme: undefined; }' is not
// assignable to type 'Settings' with
// 'exactOptionalPropertyTypes: true'. Consider adding 'undefined'
// to the types of the target's properties.
const obj2: Settings = { colorTheme: undefined };

```

此选项还会防止可选元组元素为 `undefined`（与缺失的元素相比）：

```ts
const tuple1: [number, string?] = [1];
const tuple2: [number, string?] = [1, 'hello'];
// @ts-expect-error: Type '[number, undefined]' is not assignable to
// type '[number, string?]'.
const tuple3: [number, string?] = [1, undefined];

```

##### 8.7.2.1 `exactOptionalPropertyTypes` 阻止有用的模式

我对这个选项持保留态度：一方面，启用它防止了有用的模式，例如：

```ts
type Obj = {
 num?: number,
};
function createObj(num?: number): Obj {
 // @ts-expect-error: Type '{ num: number | undefined; }' is not
 // assignable to type 'Obj' with
 // 'exactOptionalPropertyTypes: true'.
 return { num };
} 

```

##### 8.7.2.2 `exactOptionalPropertyTypes` 产生更好的类型：展开

另一方面，它更好地反映了 JavaScript 的工作方式 - 例如，展开区分了缺失的属性和值是 `undefined` 的属性：

```ts
const optionDefaults: { a: number } = { a: 1 };
// This assignment is an error with `exactOptionalPropertyTypes`
const options: { a?: number } = { a: undefined }; // (A)

const result = { ...optionDefaults, ...options };
assertType<
 { a: number }
>(result);
assert.deepEqual(
 result, { a: undefined }
);

```

如果我们在第 A 行分配了一个空对象，那么 `result` 的值将是 `{a:1}` 并匹配其类型。

`Object.assign()` 与展开类似工作。

##### 8.7.2.3 `exactOptionalPropertyTypes` 产生更好的类型：`in` 操作符

```ts
function f(obj: {prop?: number}): void {
 if ('prop' in obj) {
 // Without `exactOptionalPropertyTypes`, the type would be:
 // number | undefined
 assertType<number>(obj.prop);
 }
}

```

#### 8.7.3 `noFallthroughCasesInSwitch`

如果为 `true`，则非空 `switch` 语句必须以 `break`、`return` 或 `throw` 结尾。

#### 8.7.4 `noImplicitOverride`

如果为 `true`，则覆盖超类方法的必须具有 `override` 修饰符。

#### 8.7.5 `noImplicitReturns`

如果为 `true`，则只有当返回类型是 `void` 时才允许“隐式返回”（函数或方法的结束）。

#### 8.7.6 `noPropertyAccessFromIndexSignature`

如果为 `true`，则对于如下类型的以下类型，我们无法使用点符号访问未知属性，只能访问已知属性：

```ts
interface ObjectWithId {
 id: string,
 [key: string]: string;
}
function f(obj: ObjectWithId) {
 const value1 = obj.id; // allowed
 const value2 = obj['unknownProp']; // allowed
 // @ts-expect-error: Property 'unknownProp' comes from an index
 // signature, so it must be accessed with ['unknownProp'].
 const value3 = obj.unknownProp;
}

```

#### 8.7.7 `noUncheckedIndexedAccess`

##### 8.7.7.1 `noUncheckedIndexedAccess` 和对象

如果 `noUncheckedIndexedAccess` 为 `true`，则未知属性的类型的联合是 `undefined` 和索引签名类型的联合：

```ts
interface ObjectWithId {
 id: string,
 [key: string]: string;
}
function f(obj: ObjectWithId): void {
 assertType<string>(obj.id);
 assertType<undefined | string>(obj['unknownProp']);
}

```

`noUncheckedIndexedAccess` 对 `Record`（这是一个映射类型）也做同样的事情。

```ts
function f(obj: Record<string, number>): void {
 // Without `noUncheckedIndexedAccess`, this type would be:
 // number
 assertType<undefined | number>(obj['hello']);
}

```

##### 8.7.7.2 `noUncheckedIndexedAccess` 和数组

选项 `noUncheckedIndexedAccess` 也会影响数组处理的方式：

```ts
const arr = ['a', 'b'];
const elem = arr[0];
// Without `noUncheckedIndexedAccess`, this type would be:
// string
assertType<undefined | string>(elem);

```

数组的一个常见模式是在访问元素之前检查长度。然而，在 `noUncheckedIndexedAccess` 下，这种模式变得不方便：

```ts
function logElemAt0(arr: Array<string>) {
 if (0 < arr.length) {
 const elem = arr[0];
 assertType<undefined | string>(elem);
 console.log(elem);
 }
}

```

因此，使用不同的模式更有意义：

```ts
function logElemAt0(arr: Array<string>) {
 if (0 in arr) {
 const elem = arr[0];
 assertType<string>(elem);
 console.log(elem);
 }
}

```

#### 8.7.8 具有良好默认值的类型检查选项

默认情况下，以下选项会在编辑器中产生警告，但我们也可以选择产生编译器错误或忽略问题：

+   `allowUnreachableCode`

+   `allowUnusedLabels`

+   `noUnusedLocals`

+   `noUnusedParameters`

### 8.8 使用除 tsc 之外的工具编译 TypeScript

TypeScript 编译器 tsc 执行三个任务：

1.  类型检查

1.  生成 JavaScript 文件

1.  生成声明文件

外部工具已经变得流行，它们可以更快地完成#2 和#3。以下小节描述了帮助这些工具的配置选项。

#### 8.8.1 仅使用 tsc 进行类型检查

```ts
"compilerOptions": {
 "noEmit": true,
}

```

有时，我们只想使用 tsc 进行类型检查——例如，如果我们直接运行 TypeScript 或使用外部工具编译 TypeScript 文件（到 JavaScript 文件、声明文件等）：

+   `noEmit`：如果为`true`，我们可以运行 tsc，它将只对 TypeScript 代码进行类型检查，而不会生成任何文件。

在原则上，你不再需要提供与输出相关的设置，如`rootDir`和`outDir`。然而，一些外部工具可能需要它们。

#### 8.8.2 通过类型剥离生成`.js`文件：`erasableSyntaxOnly`和`verbatimModuleSyntax`

```ts
"compilerOptions": {
 "erasableSyntaxOnly": true,
 "verbatimModuleSyntax": true, // implies "isolatedModules"
}

```

*类型剥离*是将 TypeScript 编译成 JavaScript 的一种简单快捷的方法。当 Node.js 运行 TypeScript 时，它就是使用这种方法。类型剥离之所以快速，是因为它只支持 TypeScript 的一个子集，其中有两件事是可能的：

1.  类型语法可以通过仅解析语法来检测和删除，而无需执行额外的语义分析。

1.  没有非类型语言特性会被转换。换句话说：移除类型语法就足以生成 JavaScript。

为了帮助类型剥离，TypeScript 有两个编译器选项，如果使用不受支持的功能，它们会报告错误：

+   `verbatimModuleSyntax`禁止防止#1 的功能。

+   `erasableSyntaxOnly`禁止需要转换的特性。

注意，这些选项不会改变 tsc 生成的输出。

有用的相关知识：“类型剥离技术：用空格替换类型”（§6.5.1.1）。

#### 8.8.3 `erasableSyntaxOnly`：无转换语言特性

编译器选项`erasableSyntaxOnly`有助于类型剥离。它禁止非类型 TypeScript 特性，这些特性不是“当前”JavaScript（由目标平台支持）并且必须进行转换。这些是最重要的：

+   JSX

+   枚举

+   [类构造函数中的参数属性](https://www.typescriptlang.org/docs/handbook/2/classes.html#parameter-properties)。

+   命名空间

+   将未来 JavaScript 编译成当前 JavaScript

`erasableSyntaxOnly`还禁止了通过尖括号进行类型转换的旧方法——因为它的语法在某些情况下使得类型剥离变得不可能（[来源](https://github.com/microsoft/TypeScript/pull/61244)）：

```ts
<someType>someValue // not allowed

```

然而，替代的`as`总是更好的选择：

```ts
someValue as someType // allowed

```

#### 8.8.4 `verbatimModuleSyntax`：在导入和导出中强制使用`type`

编译器选项`verbatimModuleSyntax`迫使我们向仅类型导入和导出添加关键字`type`。

当通过 type stripping 将 TypeScript 编译成 JavaScript 时，我们需要删除 TypeScript 部分。其中大部分部分很容易检测到。例外的是导入和导出——例如，没有语义分析，我们不知道一个导入是（TypeScript）类型还是（JavaScript）值。如果仅类型导入和导出使用关键字`type`标记，则不需要此类分析。

##### 8.8.4.1 导入类型

这就是`type`关键字在导入中的样子：

```ts
// Input: TypeScript
import { type SomeInterface, SomeClass } from './my-module.js';

// Output: JavaScript
import { SomeClass } from './my-module.js';

```

注意，一个类既是值也是类型。在这种情况下，不需要`type`关键字，因为这部分语法可以保持为纯 JavaScript。

我们也可以将`type`应用于整个导入：

```ts
import type { Type1, Type2, Type3 } from './types.js';

```

##### 8.8.4.2 导出类型

内联类型导出：

```ts
export type MyType = {};
export interface MyInterface {}

```

导出条款：

```ts
type Type1 = {};
type Type2 = {};
export {
 type Type1,
 type Type2,
}

type Type3 = {};
type Type4 = {};
export type {
 Type3,
 Type4,
}

```

可惜，默认导出仅适用于接口：

```ts
export default interface DefaultInterface {} // OK

type DefaultType = {}
export default DefaultType; // error
export default type DefaultType; // error

export default type {} // error

```

我们可以使用以下解决方案：

```ts
type DefaultType = {}
export {
 type DefaultType as default,
}

```

为什么存在这种不一致性？`type`在`export default`之后允许作为（JavaScript 级别）标识符。

##### 8.8.4.3 `isolatedModules`

激活`verbatimModuleSyntax`也会激活`isolatedModules`，这就是为什么我们只需要前者设置。后者阻止我们使用一些相对不为人知的特性，这些特性也可能有问题。

作为旁注，此选项使 esbuild 能够将 TypeScript 编译成 JavaScript ([source](https://esbuild.github.io/content-types/#isolated-modules))。

#### 8.8.5 `isolatedDeclarations`：更高效地生成`.d.ts`文件

```ts
"compilerOptions": {
 // Only allowed if `declaration` or `composite` are true
 "isolatedDeclarations": true,
}

```

选项`isolatedDeclarations`通过迫使我们添加更多类型注解，从而帮助外部工具将 TypeScript 文件编译成声明文件，这样在编译时就不需要类型推断（简单类型推断仍然允许——更多内容将在后面介绍）。这有几个好处：

+   工具不需要知道类型推断的逻辑——这使得它们更简单。提取声明变成了一种语法操作，并且实际上不需要考虑类型级别。

+   不必运行类型推断可以节省时间。

+   我们可以单独查看单个文件：类型推断有时需要访问其他文件来计算其结果，这引入了对这些文件的依赖。

    +   顺便说一下，这也解释了编译器选项名称的由来。

`isolatedDeclarations`仅产生编译器错误，它不会改变 tsc 输出的内容。它只影响导出的结构——因为只有那些会出现在声明文件中。模块内部代码不受影响。

让我们看看受`isolatedDeclarations`影响的三个结构。

##### 8.8.5.1 顶级函数的返回类型

对于顶级函数，我们通常应该明确指定返回类型：

```ts
// OK: return type stated explicitly
export function f1(): string {
 return 123..toString();
}
// Error: return type requires inference
export function f2() {
 return 123..toString();
}
// OK: return type trivial to determine
export function f3() {
 return 123;
}

```

##### 8.8.5.2 变量声明的类型

更复杂的变量声明必须具有类型注解。请注意，这仅影响顶层声明 - 例如：函数内部的变量声明不会出现在声明文件中，因此无关紧要。

```ts
// OK: type trivial to determine
export const value1 = 123;
// Error: type requires inference
export const value2 = 123..toString();
// OK: type stated explicitly
export const value3: string = 123..toString();

```

##### 8.8.5.3 类实例字段的类型

类实例字段必须具有类型注解（即使 `tsc` 可以在构造函数中推断它们的类型）：

```ts
export class C {
 str: string; // required
 constructor(str: string) {
 this.str = str;
 }
}

```

##### 8.8.5.4 `isolatedDeclarations` 需要 `declaration` 或 `composite`

我很乐意始终使用 `isolatedDeclarations`，但 TypeScript 只允许在选项 `declaration` 或选项 `composite` 启用时使用它。[Jake Bailey 解释了原因](https://github.com/microsoft/TypeScript/issues/58262#issuecomment-2597014286)：

> 在实现层面，`isolatedDeclarations` 诊断是由声明转换器产生的额外声明诊断，我们只有在启用 `declaration` 时才会运行它。
> 
> 理论上可以实施，使得 `isolatedDeclarations` 启用这些检查（诊断实际上是我们运行转换器然后丢弃生成的 AST），但这与原始设计有所不同。

##### 8.8.5.5 进一步阅读

TypeScript 5.5 版本的发布说明中有一个关于独立声明的[全面部分](https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/#isolated-declarations)。

### 8.9 从 ESM 导入 CommonJS

一个关键问题影响从 ESM 模块导入 CommonJS 模块：

+   在 ESM 中，默认导出是模块命名空间对象的 `.default` 属性。

+   在 CommonJS 中，模块对象 *就是* 默认导出 - 例如，有许多 CommonJS 模块将 `module.exports` 设置为函数。

让我们看看两种有助于此的选项。

#### 8.9.1 `allowSyntheticDefaultImports`：检查 CommonJS 模块的默认导入

此选项仅影响类型检查，不影响 TypeScript 生成的 JavaScript 代码：如果启用，CommonJS 模块的默认导入将引用 `module.exports`（而不是 `module.exports.default`） - 但仅在没有 `module.exports.default` 的情况下。

这反映了 Node.js 处理 CommonJS 模块的默认导入的方式（[来源](https://nodejs.org/api/esm.html#interoperability-with-commonjs)）：“当导入 CommonJS 模块时，`module.exports` 对象作为默认导出提供。如果提供静态分析，则可能存在命名导出，作为与更好生态系统兼容性的便利。”

**我们需要这个选项吗？** 是的，但如果 `moduleResolution` 是 `"bundler"` 或 `module` 是 `"NodeNext"`（这会激活 `esModuleInterop`，进而激活 `allowSyntheticDefaultImports`），则它将自动激活。

#### 8.9.2 `esModuleInterop`：更好的 TypeScript 到 CommonJS 代码编译

此选项会影响生成的 CommonJS 代码：

+   如果 `false`：

    +   `import * as m from 'm'`被编译为`const m = require('m')`。

    +   `import m from 'm'`被（大致）编译为`const m = require('m')`，并且对`m`的每次访问都被编译为`m.default`。

+   如果为`true`：

    +   `import * as m from 'm'`将一个新对象分配给`m`，该对象具有与`module.exports`相同的属性，以及一个属性`.default`，它引用`module.exports`。

    +   `import m from 'm'`将一个新对象分配给`m`，该对象具有单个属性`.default`，它引用`module.exports`。对`m`的每次访问都被编译为`m.default`。

+   如果一个 CommonJS 模块具有标记属性`.__esModule`，则它始终被导入，好像`esModuleInterop`被关闭一样。

**我们需要这个选项吗？** 不，因为我们只编写 ESM 模块。

### 8.10 一个具有良好默认值的另一个选项

我们通常可以忽略此选项：

+   `moduleDetection`：此选项配置 TypeScript 如何确定一个文件是脚本还是模块。通常可以省略，因为其默认的`"auto"`在大多数情况下都工作得很好。只有当你代码库中的模块既没有导入也没有导出时，你才需要明确将其设置为`"force"`。如果`module`是`"NodeNext"`且`package.json`中包含`"type":"module"`，那么即使这些文件也被解释为模块。

### 8.11 Visual Studio Code

如果你对自动创建的导入中本地导入的模块规范不满意，那么你可以查看以下两个设置：

```ts
javascript.preferences.importModuleSpecifierEnding
typescript.preferences.importModuleSpecifierEnding

```

默认情况下，VSC 现在应该足够智能，能够在必要时添加文件名扩展名。

### 8.12 摘要：通过回答四个问题来组装你的`tsconfig.json`

这是一个包含所有设置的初始`tsconfig.json`文件。以下子部分解释了根据你的需求应该移除哪些部分。

或者，你可以通过命令行或在线使用我的交互式[tsconfig 配置器](https://github.com/rauschma/tsconfigurator)。

```ts
{
 "include": ["src/**/*"],
 "compilerOptions": {
 // Specified explicitly (not derived from source file paths)
 "rootDir": "src",
 "outDir": "dist",

 //========== Target and module ==========
 // Nothing is ever transpiled
 "target": "ESNext", // sets up "lib" accordingly
 "module": "NodeNext", // sets up "moduleResolution"
 // Don’t check .d.ts files
 "skipLibCheck": true,
 // Emptily imported modules must exist
 "noUncheckedSideEffectImports": true,
 // Allow importing JSON
 "resolveJsonModule": true,

 //========== Type checking ==========
 // Essential: activates several useful options
 "strict": true,
 // Beyond "strict": less important
 "exactOptionalPropertyTypes": true,
 "noFallthroughCasesInSwitch": true,
 "noImplicitOverride": true,
 "noImplicitReturns": true,
 "noPropertyAccessFromIndexSignature": true,
 "noUncheckedIndexedAccess": true,

 //========== Only JS at non-type level (type stripping etc.) ==========
 // Forbid non-JavaScript language constructs such as:
 // JSX, enums, constructor parameter properties, namespaces
 "erasableSyntaxOnly": true,
 // Enforce keyword `type` for type imports etc.
 "verbatimModuleSyntax": true, // implies "isolatedModules"

 //========== Use filename extension .ts in imports ==========
 "allowImportingTsExtensions": true,
 // Only needed if compiling to JavaScript
 "rewriteRelativeImportExtensions": true, // from .ts to .js

 //========== Emitted files ==========
 // tsc only type-checks, doesn’t emit any files
 "noEmit": true,
 //----- Output: .js -----
 "sourceMap": true, // .js.map files
 //----- Output: .d.ts -----
 "declaration": true, // .d.ts files
 // “Go to definition” jumps to TS source etc.
 "declarationMap": true, // .d.ts.map files
 // - Enforces constraints that enable efficient .d.ts generation:
 //   no inferred return types for exported functions etc.
 // - Even though this option would be generally useful, it requires
 //   that `declaration` and/or `composite` are true.
 "isolatedDeclarations": true,
 }
}

```

#### 8.12.1 你想将新的 JavaScript 转换为旧的 JavaScript 吗？

TypeScript 可以将新的 JavaScript 特性转换为仅使用较老“目标”特性的代码。这有助于支持较老的 JavaScript 引擎。如果你希望这样做，你必须更改`"target"`：

```ts
"compilerOptions": {
 // Transpile new JavaScript to old JavaScript
 "target": "ES20YY", // sets up "lib" accordingly
}

```

#### 8.12.2 应该只允许 TypeScript 在非类型级别使用 JavaScript 特性吗？

换句话说：是否应该使所有非 JavaScript 语法都可擦除？如果是的话，那么以下是一些被禁止的主要特性：JSX、枚举、构造函数参数属性和命名空间。

初始 tsconfig 只允许可擦除语法。如果你想使用上述任何特性，那么请移除“仅在非类型级别使用 JS”部分。

#### 8.12.3 在本地导入中你想使用哪种文件名扩展名？

启动 tsconfig 允许在导入中使用`.ts`。如果您想使用`.js`，可以移除“在导入中使用文件扩展名.ts”部分。

#### 8.12.4 tsc 应该输出哪些文件？

+   如果`tsc`应该输出某些文件，请移除属性`"noEmit"`。

+   如果`tsc`不应该输出 JavaScript，请移除子部分“输出：.js”。

+   如果`tsc`不应该输出声明，请移除子部分“输出：.d.ts”。

如果没有文件输出，您可以移除以下属性：

+   `"rootDir"`

+   `"outDir"`

输出文件：

| 文件 | `tsconfig.json` |
| --- | --- |
| `*.js` | 默认（通过`"noEmit": true`禁用） |
| `*.js.map` | `"sourceMap": true` |
| `*.d.ts` | `"declaration": true` |
| `*.d.ts.map` | `"declarationMap": true` |

只有在源文件被输出时，才会输出源映射（`.map`）。

### 8.13 进一步阅读

#### 8.13.1 其他人对`tsconfig.json`的建议

+   Matt Pocock 的[“TSConfig 速查表”](https://www.totaltypescript.com/tsconfig-cheat-sheet)

+   Pelle Wessman 的[`base.json`](https://github.com/voxpelli/tsconfig/blob/main/base.json)

+   Sindre Sorhus 的[`tsconfig.json`](https://github.com/sindresorhus/tsconfig/blob/main/tsconfig.json)

#### 8.13.2 本章来源

一些来源已在之前提到。以下是我使用的额外来源：

+   [官方 TSConfig 文档](https://www.typescriptlang.org/tsconfig/)

+   [TypeScript 5.7 公告中的“相对路径重写部分”](https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/#path-rewriting-for-relative-paths)

+   esbuild 文档对编译 TypeScript 做出了[有趣的观察](https://esbuild.github.io/content-types/#typescript)。
