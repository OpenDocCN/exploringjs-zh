# 10 创建 TypeScript 应用程序

> 原文：[`exploringjs.com/ts/book/ch_typescript-apps.html`](https://exploringjs.com/ts/book/ch_typescript-apps.html)

（广告，请勿拦截。）

1.  10.1 使用 TypeScript 为 Web 浏览器编写应用程序

1.  10.2 使用 TypeScript 为服务器端运行时编写应用程序

在本章中，我将提供一些使用 TypeScript 编写应用程序的技巧。

### 10.1 使用 TypeScript 为 Web 浏览器编写应用程序

即使是纯 JavaScript，大多数项目也会使用构建工具来构建前端 Web 应用程序，出于各种原因。大多数这些构建工具都默认支持 TypeScript。

如果您：

+   想用 TypeScript 编写前端应用程序

+   有一个特定的前端框架（或纯代码）在心中

+   不想过多考虑工具

然后，您可以查看 Vite 的“入门”页面，Vite 是一个流行的前端构建工具。[“入门”页面](https://vite.dev/guide/)

但还有许多其他用于 JavaScript 的构建工具。以下是一些：

+   范围较小：esbuild、Rolldown、Rollup、Rspack、tsup、Turbopack

+   更全面：Parcel、Rsbuild、Vite

+   Monorepo 工具：Nx、Turborepo

### 10.2 使用 TypeScript 为服务器端运行时编写应用程序

+   Node.js、Deno 和 Bun 都可以直接运行 TypeScript。

+   默认情况下，Node.js 只支持 类型剥离，但通过 `--experimental-transform-types`（[`nodejs.org/api/cli.html#--experimental-transform-types`](https://nodejs.org/api/cli.html#--experimental-transform-types)）可以提供对 JSX 和枚举等不可擦除特性的实验性支持。类型剥离将保持为默认设置。

Node.js 入门：

+   我的 GitHub 仓库 [nodejs-type-stripping](https://github.com/rauschma/nodejs-type-stripping) 展示了类型剥离在 Node.js 中的工作原理。

+   “使用 TypeScript 发布 npm 包”（第九部分） 解释了如何将 TypeScript 转译为 Node.js。
