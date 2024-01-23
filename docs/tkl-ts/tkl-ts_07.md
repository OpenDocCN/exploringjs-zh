# 五、尝试 TypeScript

> 原文：[`exploringjs.com/tackling-ts/ch_trying-out-typescript.html`](https://exploringjs.com/tackling-ts/ch_trying-out-typescript.html)

* * *

+   5.1 TypeScript Playground

+   5.2 TS Node

* * *

本章提供了快速尝试 TypeScript 的技巧。

### 5.1 TypeScript Playground

[*TypeScript Playground*](http://www.typescriptlang.org/play/)是一个用于 TypeScript 代码的在线编辑器。其功能包括：

+   支持完整的 IDE 风格编辑：自动补全等。

+   显示静态类型错误。

+   显示将 TypeScript 代码编译成 JavaScript 的结果。它还可以在浏览器中执行结果。

Playground 非常适用于快速实验和演示。它可以将 TypeScript 代码片段和编译器设置保存到 URL 中，非常适合与他人分享这些片段。以下是一个示例 URL：

> [`https://www.typescriptlang.org/play/#code/MYewdgzgLgBFDuBLYBTGBeGAKAHgLhmgCdEwBzASgwD4YcYBqOgbgChXRIQAbFAOm4gyWBMhRYA5AEMARsAkUKzIA`](https://www.typescriptlang.org/play/#code/MYewdgzgLgBFDuBLYBTGBeGAKAHgLhmgCdEwBzASgwD4YcYBqOgbgChXRIQAbFAOm4gyWBMhRYA5AEMARsAkUKzIA)

### 5.2 TS Node

[TS Node](https://github.com/TypeStrong/ts-node)是 Node.js 的 TypeScript 版本。其用例包括：

+   TS Node 提供了一个用于 TypeScript 的 REPL（命令行）：

    ```ts
    $ ts-node
    > const twice = (x: string) => x + x;
    > twice('abc')
    'abcabc'
    > twice(123)
    Error TS2345: Argument of type '123' is not assignable
    to parameter of type 'string'.
    ```

+   TS Node 使一些 JavaScript 工具能够直接执行 TypeScript 代码。它会自动将 TypeScript 代码编译成 JavaScript 代码，并将其传递给工具，而无需我们做任何事情。以下 shell 命令演示了如何在[JavaScript 单元测试框架 Mocha](https://mochajs.org)中使用它：

    ```ts
    mocha --require ts-node/register --ui qunit testfile.ts
    ```

使用`npx ts-node`来运行 REPL 而无需安装它。

[Comments](https://github.com/rauschma/tackling-ts/issues/5)
