# 6 TypeScript 的使用：工作流程、工具等。

> 原文：[`exploringjs.com/ts/book/ch_typescript-workflows.html`](https://exploringjs.com/ts/book/ch_typescript-workflows.html)

（广告，请勿遮挡。）

1.  6.1 TypeScript 是 JavaScript 加上类型语法

1.  6.2 运行 TypeScript 代码的方式

    1.  6.2.1 直接运行 TypeScript

    1.  6.2.2 打包 TypeScript

    1.  6.2.3 将 TypeScript 转译为 JavaScript

    1.  6.2.4 本地导入 TypeScript 模块的文件扩展名

1.  6.3 将库包发布到 npm 注册表

    1.  6.3.1 基本：`.js` 和 `.d.ts` 文件

    1.  6.3.2 可选：源映射

1.  6.4 DefinitelyTyped：一个为无类型 npm 包提供类型的仓库

1.  6.5 使用除 `tsc` 之外的工具编译 TypeScript

    1.  6.5.1 类型剥离

    1.  6.5.2 隔离声明

1.  6.6 JSR – JavaScript 注册表

    1.  6.6.1 JSR 属于谁？

1.  6.7 编辑 TypeScript

1.  6.8 检查 JavaScript 文件中的类型

如果你是 JavaScript 程序员并且想大致了解使用 TypeScript 的感觉（在学习更多细节之前先迈出第一步），请阅读本章。你将得到以下问题的答案：

+   TypeScript 代码与 JavaScript 代码有何不同？

+   TypeScript 代码是如何运行的？

+   TypeScript 是如何帮助在 IDE 中进行编辑的？

+   等等。

本章重点介绍 TypeScript 的工作原理。如果你想了解更多关于为什么它有用的信息，请参阅“TypeScript 的销售方案”（§2）。

### 6.1 TypeScript 是 JavaScript 加上类型语法

让我们从对 TypeScript 的初步描述开始。这个描述并不完全准确（有一些例外和细节我省略了），但它应该给你一个坚实的基础：

> TypeScript 是 JavaScript 加上类型语法。

这两部分的目的有哪些？

+   JavaScript 语法是运行时存在的语法：为了运行 TypeScript 代码，必须移除类型语法——通过编译生成纯 JavaScript。该代码由 JavaScript 引擎执行。

+   类型语法仅在编辑和编译期间使用；它在运行时没有影响：

    +   一方面，它支持 *类型检查*，如果类型之间存在不一致或类型与 JavaScript 值之间存在不一致，则会报告错误。类型检查在编辑和编译期间运行。

    +   另一方面，类型通过自动完成、类型提示、重构等提高了编辑效率。

考虑以下 TypeScript 代码：

```ts
function add(x: number, y: number): number {
 return x + y;
}

```

如果我们想运行这段代码，我们必须移除类型语法，并得到由 JavaScript 引擎执行的 JavaScript：

```ts
function add(x, y) {
 return x + y;
}

```

### 6.2 TypeScript 代码的运行方式

考虑以下 TypeScript 项目：

```ts
ts-app/
 tsconfig.json
 src/
 main.ts
 util.ts
 util_test.ts
 test/
 integration_test.ts

```

+   `tsconfig.json`是一个配置文件，它告诉 TypeScript 如何检查和编译我们的代码。

+   剩余的文件是 TypeScript 源代码。

让我们探索我们可以运行此代码的不同方式。

#### 6.2.1 直接运行 TypeScript

现在大多数服务器端运行时都可以直接运行 TypeScript 代码——例如，Node.js、Deno 和 Bun。换句话说，以下在 Node.js 23.6.0+中有效：

```ts
cd ts-app/
node src/main.ts

```

#### 6.2.2 打包 TypeScript

在开发 Web 应用程序时，*打包*是一种常见的做法——即使是纯 JavaScript 项目：所有的 JavaScript 代码（应用程序代码和库代码）都被合并成一个单一的 JavaScript 文件（有时更多，但永远不会超过几个）——通常是从 HTML 文件中加载的。这有几个好处：

+   在 HTTP/2 之前，每个连接只能服务一个文件。但打包的好处现在不再相关。

    +   客户端请求并处理的每个文件，仍然会产生一点开销（即使没有打开新的连接）。

+   Web 服务器不需要服务许多（通常是小的）文件——这有助于提高效率。

+   一个大文件比许多小文件压缩得更好。

大多数打包器都支持 TypeScript——直接或通过插件。这意味着，我们通过打包器生成的 JavaScript 文件`bundle.js`运行我们的 TypeScript 代码：

```ts
ts-app/
 tsconfig.json
 src/
 main.ts
 util.ts
 util_test.ts
 test/
 integration_test.ts
 dist/
 bundle.js

```

#### 6.2.3 将 TypeScript 转换为 JavaScript

另一个选项是通过 TypeScript 编译器`tsc`将 TypeScript 应用程序编译成 JavaScript，并运行生成的代码。在服务器端 JavaScript 运行时还没有内置对 TypeScript 的支持之前，这是我们在那里运行 TypeScript 的唯一方法。

将源代码编译成源代码也称为*转换*。`tsconfig.json`指定了转换输出写入的位置。假设我们将其写入`dist/`目录：

```ts
ts-app/
 tsconfig.json
 src/
 main.ts
 util.ts
 util_test.ts
 test/
 integration_test.ts
 dist/
 src/
 main.js
 util.js
 util_test.js
 test/
 integration_test.js

```

#### 6.2.4 本地导入 TypeScript 模块的文件名扩展名

当涉及到本地导入 TypeScript 模块的文件名扩展名时，我们必须区分被转换的代码和直接运行的代码。

默认情况下，TypeScript 不会更改导入模块的指定符。因此，必须转换的代码看起来像这样（从 JavaScript 代码中导入`util.js`）：

```ts
// main.ts
import {helperFunc} from './util.js';

```

然而，如果我们直接运行这样的代码，它将不起作用。在那里，我们必须编写（从 TypeScript 代码中导入`util.ts`）：

```ts
// main.ts
import {helperFunc} from './util.ts';

```

我们还可以告诉 TypeScript 将本地导入的文件名扩展名从`.ts`更改为`.js`（更多信息）。然后之前的代码也可以被转换。

### 6.3 将库包发布到 npm 注册表

npm 注册表仍然是发布包最受欢迎的方式。尽管 Node.js 运行 TypeScript 代码，但包必须作为 JavaScript 代码部署。这使 JavaScript 代码可以使用用 TypeScript 编写的库包。然而，我们还想支持 TypeScript 功能。因此，单个库文件 `lib.ts` 通常部署为五个文件（其中四个由 TypeScript 从 `lib.ts` 编译而成）：

+   必要的：

    +   `lib.js`：`lib.ts` 的 JavaScript 部分

    +   `lib.d.ts`：`lib.ts` 的类型部分（一个 *声明文件*）

+   可选：源映射。它们将编译输出的源代码位置映射到 `lib.ts`。

    +   `lib.js.map`：`lib.js` 的源映射

    +   `lib.d.ts.map`：`lib.d.ts` 的源映射

    +   `lib.ts`：前两个源映射的目标

（关于这一切意味着什么的更多内容将在下一部分中讨论。）

例如，考虑以下库包：

```ts
ts-lib/
 package.json
 tsconfig.json
 src/
 lib.ts
 dist/
 lib.js
 lib.js.map
 lib.d.ts
 lib.d.ts.map

```

+   `package.json` 是 npm 对我们的库包的描述。其中一些数据，如所谓的 *package exports*，也被 TypeScript 使用——例如，当有人从我们的包中导入时查找类型信息。

+   `dist/` 目录中的每个文件都是由 TypeScript 生成的。虽然它被上传到 npm 注册表，但通常不会添加到版本控制系统中，因为它可以很容易地重新生成。

+   只有 `tsconfig.json` 没有上传到 npm 注册表。

#### 6.3.1 必要的：`.js` 和 `.d.ts`

看到结合在 `.lib.ts` 中的 JavaScript 和类型被拆分为只包含 JavaScript 的 `lib.js` 和只包含类型的 `lib.d.ts` 是很有趣的。为什么这样做？这使库包既能被 JavaScript 代码使用，也能被 TypeScript 代码使用：

+   JavaScript 代码可以忽略 `.d.ts` 文件。

+   TypeScript 使用它们进行类型检查、自动补全、重构等。

实际上，在幕后，许多编辑器（例如 Visual Studio Code）在编辑 JavaScript 代码时使用一种轻量级的 TypeScript 模式，这样我们也能在那里获得简单的类型检查和代码补全。

这是 TypeScript 输入 `lib.ts`

```ts
/** Add two numbers. */
export function add(x: number, y: number): number {
 return x + y; // numeric addition
}

```

它被拆分为 `lib.js` 一方面：

```ts
/** Add two numbers. */
export function add(x, y) {
 return x + y; // numeric addition
}
//# sourceMappingURL=lib.js.map

```

另一方面，`lib.d.ts`：

```ts
/** Add two numbers. */
export declare function add(x: number, y: number): number;
//# sourceMappingURL=lib.d.ts.map

```

备注：

+   这两个文件都指向它们的源映射。

+   默认情况下，这两个文件都包含注释（但我们也可以告诉 TypeScript 不要包含它们）：

    +   `lib.js` 包含所有注释，这样代码更容易阅读。

    +   `lib.d.ts` 只包含 JSDoc 注释 (`/** */`)，因为它们被许多 IDE 用于显示内联文档。

#### 6.3.2 可选：源映射

如果我们将文件 `I` 编译成文件 `O`，则 `O` 的源映射将 `O` 中的源代码位置映射到 `I` 中的源代码位置。这意味着我们可以使用 `O`，但显示来自 `I` 的信息——例如：

+   `lib.js.map`：将 `lib.js` 位置映射到 `lib.ts` 位置，并在运行前者时为我们提供后者的调试和堆栈跟踪。

+   `lib.d.ts.map`：将 `lib.d.ts` 的行映射到 `lib.ts` 的行。它使得从 `lib.ts` 的导入能够实现“转到定义”功能，带我们到那个文件。

除了堆栈跟踪之外，所有与 source-map 相关的功能都需要访问原始 TypeScript 源代码。这就是为什么如果存在 source maps，包含 `lib.ts` 是有意义的。

这就是 `lib.js.map` 的样子：

```ts
{
 "version": 3,
 "file": "lib.js",
 "sourceRoot": "",
 "sources": [
 "../../src/lib.ts"
 ],
 "names": [],
 "mappings": "AAAA,uBAAuB;AACvB,MAAM,UAAU,···"
}

```

这就是 `lib.d.ts.map` 的样子：

```ts
{
 "version": 3,
 "file": "lib.d.ts",
 "sourceRoot": "",
 "sources": [
 "../../src/lib.ts"
 ],
 "names": [],
 "mappings": "AAAA,uBAAuB;AACvB,wBAAgB,GAAG,···"
}

```

在这两种情况下，实际内容中的 `"mappings"` 都被缩略了。并且在实际的 `tsc` 输出中，JSON 总是被压缩成单行。

### 6.4 DefinitelyTyped：一个为无类型 npm 包提供类型的仓库

这些天，许多 npm 包都附带 TypeScript 类型。然而，并非所有都这样做。在这种情况下，[DefinitelyTyped](https://definitelytyped.org) 可能会帮助：如果它支持无类型的包 `pkg`，那么我们可以额外安装一个带有 `pkg` 类型的包 `@types/pkg`。

对于 Node.js 来说，一个重要的 DefinitelyTyped 包是 `@types/node`，它包含了所有其 API 的类型。如果你在 Node.js 上开发 TypeScript，你通常会把这个包作为一个开发依赖项。

### 6.5 编译 TypeScript 而不是使用 `tsc` 的工具

让我们回顾一下 `tsc` 执行的所有任务（在本节中我们将忽略 source maps）：

1.  它将 TypeScript 文件编译成 JavaScript 文件。

1.  它将 TypeScript 文件编译成类型声明文件。

1.  它检查 TypeScript 文件。

第 3 点非常复杂，只有 `tsc` 才能完成。然而，对于第 1 点和第 2 点，TypeScript 有一些稍微简单的子集，编译过程并不涉及太多除了语法处理之外的工作。这意味着我们可以为第 1 点和第 2 点使用外部、更快的工具。

甚至有 `tsconfig.json` 设置来警告我们是否没有保持在 TypeScript 的那些子集内（更多信息）。在实践中，这样做并不是很大的牺牲。

#### 6.5.1 类型剥离

*类型剥离* 是将 TypeScript 编译成 JavaScript 的简单快捷方式。这是 Node.js 运行 TypeScript 时所使用的。类型剥离之所以快速，是因为它只支持 TypeScript 的一个子集，其中有两件事是可能的：

1.  类型语法可以通过仅解析语法来检测和删除，而无需执行额外的语义分析。

1.  没有非类型语言特性会被转换。换句话说：移除类型语法就足以生成 JavaScript。

第 2 点意味着有一些 TypeScript 特性我们无法使用——例如，枚举和 JSX（TypeScript 中的类似 HTML 的语法，例如 React 所使用的）。

类型剥离的一个显著好处是它不需要任何配置（通过 `tsconfig.json` 或其他方式），因为它非常简单。这使得使用它的平台在 TypeScript 的更改方面更加稳定。

##### 6.5.1.1 类型剥离技术：用空格替换类型

`ts-blank-space` 工具（由 Ashley Claymore 为 Bloomberg 开发）开创了一种类型剥离的巧妙技术：它不是简单地移除类型语法，而是用空格替换它。这意味着输出中的源代码位置不会改变。因此，任何出现在（例如）堆栈跟踪中的位置对于输入仍然有效，并且减少了源映射的需求：你仍然需要它们进行调试和跳转到定义，但类型剥离生成的 JavaScript 相对接近原始 TypeScript，你通常也能接受。

例如 - 输入（TypeScript）：

```ts
function add(x: number, y: number): number {
 return x + y;
}

```

输出（JavaScript）：

```ts
function add(x        , y )         {
 return x + y;
}

```

如果你想进一步探索，可以查看 [ts-blank-space 演示场](https://bloomberg.github.io/ts-blank-space/play/)。

#### 6.5.2 隔离声明

“隔离声明”是一种 TypeScript 编写风格，使得外部工具生成声明文件更加容易。它主要意味着我们必须在 TypeScript 编译器 `tsc` 不需要类型注解的位置添加类型注解——多亏了其自动推导类型（所谓 *类型推断*）的能力。然而，如果外部工具不需要这种能力，它们会更简单、更快。此外，如果类型信息是本地提供的，它们也不必访问和分析外部文件。

以下示例显示了隔离声明风格如何改变代码：

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

注意，隔离声明只影响导出的结构。模块内部代码不会出现在声明文件中。

### 6.6 JSR – JavaScript 注册表

[JavaScript 注册表 JSR](https://jsr.io) 是 npm 和 npm 注册表发布包的替代方案。它的工作原理如下：

+   对于 TypeScript 包，你只需上传 `.ts` 文件。

+   安装 TypeScript 包的方法取决于平台：

    +   在支持 TypeScript 库包的 JavaScript 平台上，JSR 只会安装 TypeScript。

    +   在所有其他平台上，JSR 会自动生成 `.js` 文件和 `.d.ts` 文件，并安装这些文件以及 `.ts` 文件。为了使自动生成成为可能，TypeScript 代码必须遵循一组称为 [“无慢类型”](https://jsr.io/docs/about-slow-types) 的规则——这与 隔离声明 类似。

相比之下，使用 npm 注册表，如果你的 TypeScript 库包需要支持 Node.js，你必须上传 `.js` 文件和 `.d.ts` 文件。

JSR 还提供了一些 npm 没有的功能，例如自动生成文档。有关更多信息，请参阅官方文档中的 [“为什么选择 JSR？”](https://jsr.io/docs/why)。

#### 6.6.1 谁拥有 JSR？

引用官方文档页面 [“治理”](https://jsr.io/docs/governance/overview)：

> JSR 不属于任何个人或组织。它是一个面向所有人的社区驱动项目，为整个 JavaScript 生态系统而构建。
> 
> JSR 目前由 Deno 公司运营。我们目前正在努力建立一个治理委员会来监督项目，然后该项目将致力于将项目转移到基金会。

### 6.7 编辑 TypeScript

两个流行的 JavaScript IDE 是：

+   [*Visual Studio Code*](https://code.visualstudio.com/)（免费）

+   [*WebStorm*](https://www.jetbrains.com/webstorm/)（非商业用途免费）

本节中的观察结果主要关于 Visual Studio Code，但也可能适用于其他 IDE。

使用 Visual Studio Code，我们有两种不同的类型检查方式：

+   在 Visual Studio Code 中，任何当前打开的文件都会自动进行类型检查。为了提供这种功能，它自带了 TypeScript 的安装。

+   如果我们想要检查整个代码库的类型，我们必须调用 TypeScript 编译器 `tsc`。我们可以通过 Visual Studio Code 的 *任务* 来做这件事——这是一种调用外部工具（用于类型检查、编译、打包等）的内置方式。官方文档有关于任务的[更多信息](https://code.visualstudio.com/Docs/editor/tasks)。

### 6.8 类型检查 JavaScript 文件

可选地，TypeScript 也可以检查 JavaScript 文件。显然，这将只给我们有限的结果。然而，为了帮助 TypeScript，我们可以通过 JSDoc 注释添加类型信息——例如：

```ts
/**
 * @param {number} x - The first operand
 * @param {number} y - The second operand
 * @returns {number} The sum of both operands
 */
function add(x, y) {
 return x + y;
}

```

如果我们这样做，我们仍然在编写 TypeScript，只是使用了不同的语法。

这种方法的优点：

+   运行代码不需要构建步骤——即使在不支持 TypeScript 的平台（如浏览器）上也不需要。

    +   我们也可以使用 JSDoc 注释从 `.js` 文件生成 `.d.ts` 文件。虽然这是一个额外的构建步骤，但如何操作在 [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/declaration-files/dts-from-js.html) 中有解释。

+   它使我们能够以小步骤的方式使 JavaScript 代码库更安全——类型安全。

这种方法的缺点：

+   语法变得不那么容易使用。

为了解释其缺点——考虑我们在 TypeScript 中如何定义一个接口：

```ts
interface Point {
 x: number;
 y: number;
 /** optional property */
 z?: number;
}

```

通过 JSDoc 注释实现的方式如下所示：

```ts
/**
 * @typedef Point
 * @prop {number} x
 * @prop {number} y
 * @prop {number} [z] optional property
 */

```

更多信息请参阅 TypeScript 手册：

+   [“类型检查 JavaScript 文件”](https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html)

+   [“从 `.js` 文件创建 `.d.ts` 文件”](https://www.typescriptlang.org/docs/handbook/declaration-files/dts-from-js.html)

+   [“JSDoc 参考”](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html)
