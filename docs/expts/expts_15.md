# 12 TypeScript 迁移策略

> 原文：[`exploringjs.com/ts/book/ch_migrating-to-typescript.html`](https://exploringjs.com/ts/book/ch_migrating-to-typescript.html)

（广告，请勿拦截。）

1.  12.1 策略：混合 JavaScript/TypeScript 代码库

1.  12.2 策略：向纯 JavaScript 文件添加类型信息

1.  12.3 策略：在激活编译器选项之前进行代码检查

1.  12.4 策略：错误太多？使用快照测试

1.  12.5 帮助迁移到 TypeScript 的工具

1.  12.6 结论和进一步阅读

本章概述了将代码库从 JavaScript 迁移到 TypeScript 的策略，并提到了进一步阅读的材料。

### 12.1 策略：混合 JavaScript/TypeScript 代码库

如果我们使用编译器选项 `--allowJs`，TypeScript 编译器支持 JavaScript 和 TypeScript 文件的混合：

+   TypeScript 文件被编译。

+   JavaScript 文件简单地复制到输出目录（经过一些简单的类型检查后）。

起初，只有 JavaScript 文件。然后，一个接一个地，我们将文件转换为 TypeScript。在此过程中，我们的代码库持续被编译。

这就是 `tsconfig.json` 的样子：

```ts
{
 "compilerOptions": {
 ···
 "allowJs": true
 }
}

```

**更多信息：**

+   [“逐步将 JavaScript 迁移到 TypeScript”](https://medium.com/@clayallsopp/incrementally-migrating-javascript-to-typescript-565020e49c88) by [Clay Allsopp](https://twitter.com/clayallsopp).

### 12.2 策略：向纯 JavaScript 文件添加类型信息

此方法如下操作：

+   我们继续使用当前的构建基础设施。

+   我们运行 TypeScript 编译器，但仅作为类型检查器（编译器选项 `--noEmit`）。除了编译器选项 `--allowJs`（允许并复制 JavaScript 文件）外，我们还需要使用编译器选项 `--checkJs`（用于类型检查 JavaScript 文件）。

+   我们通过 JSDoc 注释（见下例）和声明文件添加类型信息。

+   一旦 TypeScript 的类型检查器不再抱怨，我们使用编译器构建代码库。现在从 `.js` 文件切换到 `.ts` 文件并不紧急，因为整个代码库已经完全静态类型化。我们甚至现在可以生成类型文件（文件名扩展名 `.d.ts`）。

这是我们通过 JSDoc 注释指定纯 JavaScript 的静态类型的方式：

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

```ts
/** @typedef {{ prop1: string, prop2: string, prop3?: number }} SpecialType */
/** @typedef {(data: string, index?: number) => boolean} Predicate */

```

**更多信息：**

+   “类型检查 JavaScript 文件” (§6.8)

+   [“如何在 Unsplash 逐步迁移到 TypeScript”](https://medium.com/unsplash/how-we-gradually-migrated-to-typescript-at-unsplash-7a34caa24ef1) by [Oliver Joseph Ash](https://twitter.com/OliverJAsh)

### 12.3 策略：在激活编译器选项之前进行 linting

迁移到 TypeScript 的项目通常从默认的类型检查开始，然后分阶段添加更多检查 - 例如：

1.  初始状态：默认检查

1.  激活`strictNullChecks`（防止除非类型是`T | undefined`或`T | null`，否则使用`undefined`和`null`)

1.  激活`noImplicitAny`（防止省略参数的类型等）

1.  激活`strict`（包括前两个编译器选项 - 可以删除 - 并添加[额外的选项](https://www.typescriptlang.org/tsconfig/#strict))

与编译器选项相比，我们可以按文件激活 linting 选项。这在从较宽松的类型检查切换到更严格的类型检查时很有帮助：我们可以在切换之前进行 linting。以下是一些 TypeScript linter [typescript-eslint](https://typescript-eslint.io)提供的有用规则的示例：

+   准备`noImplicitAny`：

    +   [no-unsafe-call](https://typescript-eslint.io/rules/no-unsafe-call/)

    +   [no-unsafe-member-access](https://typescript-eslint.io/rules/no-unsafe-member-access/)

    +   [no-unsafe-argument](https://typescript-eslint.io/rules/no-unsafe-argument/)

    +   [no-unsafe-assignment](https://typescript-eslint.io/rules/no-unsafe-assignment/)

    +   [no-explicit-any](https://typescript-eslint.io/rules/no-explicit-any/)

+   准备`erasableSyntaxOnly`：

    +   [eslint-plugin-erasable-syntax-only](https://github.com/JoshuaKGoldberg/eslint-plugin-erasable-syntax-only)

### 12.4 策略：错误太多？使用快照测试

在大型 JavaScript 项目中，切换到 TypeScript 可能会产生过多的错误 - 无论我们选择哪种方法。然后，对 TypeScript 错误进行快照测试可能是一个选项：

+   我们第一次在整个代码库上运行 TypeScript 编译器。

+   编译器产生的错误成为我们的初始快照。

+   在我们处理代码库时，我们将新的错误输出与之前的快照进行比较：

    +   有时现有的错误会消失。然后我们可以创建一个新的快照。

    +   有时会出现新的错误。然后我们要么修复这些错误，要么创建一个新的快照。

**更多信息：**

+   [“如何逐步迁移 10k 行代码到 TypeScript”](https://dylanvann.com/incrementally-migrating-to-typescript/) by [Dylan Vann](https://twitter.com/atomarranger)

### 12.5 帮助迁移到 TypeScript 的工具

+   [ts-migrate](https://github.com/airbnb/ts-migrate)帮助从 JavaScript 迁移到 TypeScript 的第一步：“生成的代码将通过构建，但需要进一步改进类型安全性。会有很多`//@ts-expect-error`和`any`需要随着时间的推移进行修复。总的来说，这比从头开始要好得多。”

+   [type-coverage](https://github.com/plantain-00/type-coverage) 显示了有多少百分比的标识符具有 `any` 类型。

### 12.6 结论和进一步阅读

我们已经快速浏览了迁移到 TypeScript 的策略。还有两个小贴士：

+   从实验开始你的迁移：在你的代码库中玩耍，尝试各种策略，然后再决定采用其中之一。

+   然后制定一个清晰的行动计划。与你的团队讨论优先级：

    +   有时，快速完成迁移可能更为重要。

    +   有时，在迁移过程中保持代码完全功能可能更为重要。

    +   等等……

**进一步阅读：**

+   [从 JavaScript 迁移](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html) 在 TypeScript 手册中
