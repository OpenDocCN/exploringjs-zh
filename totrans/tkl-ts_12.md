# 10 迁移到 TypeScript 的策略

> 原文：[`exploringjs.com/tackling-ts/ch_migrating-to-typescript.html`](https://exploringjs.com/tackling-ts/ch_migrating-to-typescript.html)

* * *

+   10.1 三种策略

+   10.2 策略：混合 JavaScript/TypeScript 代码库

+   10.3 策略：向普通 JavaScript 文件添加类型信息

+   10.4 策略：通过快照测试 TypeScript 错误迁移大型项目

+   10.5 结论

* * *

本章概述了从 JavaScript 迁移到 TypeScript 的策略。它还提到了进一步阅读的材料。

### 10.1 三种策略

这是迁移到 TypeScript 的三种策略：

+   我们可以支持代码库中 JavaScript 和 TypeScript 文件的混合。我们从只有 JavaScript 文件开始，然后逐渐将更多文件切换到 TypeScript。

+   我们可以保留当前（非 TypeScript）的构建流程和我们只有 JavaScript 的代码库。我们通过 JSDoc 注释添加静态类型信息，并将 TypeScript 用作类型检查器（而不是编译器）。一旦一切都正确类型化，我们就切换到 TypeScript 进行构建。

+   对于大型项目，在迁移过程中可能会出现太多 TypeScript 错误。然后快照测试可以帮助我们找到已修复的错误和新错误。

**更多信息：**

+   [“从 JavaScript 迁移”](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html) in the TypeScript Handbook

### 10.2 策略：混合 JavaScript/TypeScript 代码库

如果我们使用编译器选项`--allowJs`，TypeScript 编译器支持 JavaScript 和 TypeScript 文件的混合：

+   TypeScript 文件被编译。

+   JavaScript 文件只是简单地复制到输出目录（经过一些简单的类型检查）。

起初，只有 JavaScript 文件。然后，我们逐个将文件切换到 TypeScript。在此过程中，我们的代码库将继续被编译。

这是`tsconfig.json`的样子：

```ts
{
 "compilerOptions": {
 ···
 "allowJs": true
 }
}
```

**更多信息：**

+   [“逐步将 JavaScript 迁移到 TypeScript”](https://medium.com/@clayallsopp/incrementally-migrating-javascript-to-typescript-565020e49c88) by [Clay Allsopp](https://twitter.com/clayallsopp)。

### 10.3 策略：向普通 JavaScript 文件添加类型信息

这种方法的工作方式如下：

+   我们继续使用我们当前的构建基础设施。

+   我们运行 TypeScript 编译器，但只作为类型检查器（编译器选项`--noEmit`）。除了编译器选项`--allowJs`（用于允许和复制 JavaScript 文件），我们还必须使用编译器选项`--checkJs`（用于对 JavaScript 文件进行类型检查）。

+   我们通过 JSDoc 注释（见下面的示例）和声明文件添加类型信息。

+   一旦 TypeScript 的类型检查器不再抱怨，我们就可以使用编译器构建代码库。现在不急于从`.js`文件切换到`.ts`文件，因为整个代码库已经完全静态类型化。我们甚至现在可以生成类型文件（文件扩展名`.d.ts`）。

这是我们如何通过 JSDoc 注释为普通 JavaScript 指定静态类型的方式：

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

```ts
/** @typedef  {{  prop1:  string,  prop2:  string,  prop3?:  number  }}  SpecialType */
/** @typedef  {(data:  string,  index?:  number)  =>  boolean}  Predicate */
```

**更多信息：**

+   §4.4 “使用 TypeScript 编译器处理普通 JavaScript 文件”

+   [“我们如何逐渐迁移到 TypeScript 在 Unsplash”](https://medium.com/unsplash/how-we-gradually-migrated-to-typescript-at-unsplash-7a34caa24ef1) by [Oliver Joseph Ash](https://twitter.com/OliverJAsh)

### 10.4 策略：通过快照测试 TypeScript 错误迁移大型项目

在大型 JavaScript 项目中，切换到 TypeScript 可能会产生太多错误 - 无论我们选择哪种方法。然后，快照测试 TypeScript 错误可能是一个选择：

+   我们第一次在整个代码库上运行了 TypeScript 编译器。

+   编译器产生的错误成为我们的初始快照。

+   当我们在代码库上工作时，我们将新的错误输出与之前的快照进行比较：

    +   有时现有的错误会消失。然后我们可以创建一个新的快照。

    +   有时会出现新的错误。然后我们要么修复这些错误，要么创建一个新的快照。

**更多信息：**

+   [“如何逐步将 10 万行代码迁移到 TypeScript”](https://dylanvann.com/incrementally-migrating-to-typescript/) by [Dylan Vann](https://twitter.com/atomarranger)

### 10.5 结论

我们已经快速了解了迁移到 TypeScript 的策略。再给两个建议：

+   开始你的迁移实验：在提交到其中一个之前，尝试各种策略并玩弄你的代码库。

+   然后制定一个明确的前进计划。与团队讨论优先级：

    +   有时，快速完成迁移可能更重要。

    +   有时，在迁移过程中代码保持完全可用可能更重要。

    +   等等...

[评论](https://github.com/rauschma/tackling-ts/issues/10)
