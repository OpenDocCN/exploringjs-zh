# 9 创建 Web 应用程序通过 TypeScript 和 webpack

> 原文：[`exploringjs.com/tackling-ts/ch_webpack-typescript.html`](https://exploringjs.com/tackling-ts/ch_webpack-typescript.html)

* * *

+   9.1 所需知识

+   9.2 限制

+   9.3 存储库`ts-demo-webpack`

+   9.4 `package.json`

+   9.5 `webpack.config.js`

+   9.6 `tsconfig.json`

+   9.7 `index.html`

+   9.8 `main.ts`

+   9.9 安装、构建和运行 Web 应用程序

    +   9.9.1 在 Visual Studio Code 中构建

+   9.10 在没有加载器的情况下使用 webpack：`webpack-no-loader.config.js`

* * *

本章描述了如何通过 TypeScript 和 webpack 创建 Web 应用程序。我们将仅使用 DOM API，而不是特定的前端框架。

![](img/ef04c4c2601874494d82453a9b525b87.png) **GitHub 存储库：`ts-demo-webpack`**

[存储库`ts-demo-webpack`](https://github.com/rauschma/ts-demo-webpack)我们在本章中使用的，可以从 GitHub 下载。

### 9.1 所需知识

你应该对以下内容有大致的了解：

+   [npm](https://docs.npmjs.com/about-npm/)

+   [webpack](https://webpack.js.org)

### 9.2 限制

在本章中，我们坚持使用 TypeScript 最好支持的内容：CommonJS 模块，捆绑为脚本文件。

### 9.3 存储库`ts-demo-webpack`

这是存储库`ts-demo-webpack`的结构：

```ts
ts-demo-webpack/
  build/   (created on demand)
  html/
    index.html
  package.json
  ts/
    src/
      main.ts
  tsconfig.json
  webpack.config.js
```

Web 应用程序的构建方式如下：

+   输入：

    +   `ts/`中的 TypeScript 文件

    +   所有通过 npm 安装并由 TypeScript 文件导入的 JavaScript 代码

    +   `html/`中的 HTML 文件

+   输出 - 目录`build/`中的完整 Web 应用程序：

    +   TypeScript 文件被编译为 JavaScript 代码，与通过 npm 安装的 JavaScript 组合，并写入脚本文件`build/main-bundle.js`。这个过程称为*捆绑*，`main-bundle.js`是一个捆绑文件。

    +   每个 HTML 文件都被复制到`build/`中。

两个输出任务都由 webpack 处理：

+   通过 webpack 的*插件*`copy-webpack-plugin`将`html/`中的文件复制到`build/`中。

+   本章探讨了两种不同的捆绑工作流程：

    +   要么 webpack 直接将 TypeScript 文件编译成捆绑包，借助*加载器*`ts-loader`。

    +   或者我们自己编译 TypeScript 文件，将其编译为 JavaScript 文件，放在目录`dist/`中（就像我们在上一章中所做的那样）。然后 webpack 不需要加载器，只需要捆绑 JavaScript 文件。

    本章大部分内容都是关于使用`ts-loader`的 webpack。最后，我们简要地看一下其他工作流程。

### 9.4 `package.json`

`package.json`包含项目的元数据：

```ts
{
 "private": true,
 "scripts": {
 "tsc": "tsc",
 "tscw": "tsc --watch",
 "wp": "webpack",
 "wpw": "webpack --watch",
 "serve": "http-server build"
 },
 "dependencies": {
 "@types/lodash": "···",
 "copy-webpack-plugin": "···",
 "http-server": "···",
 "lodash": "···",
 "ts-loader": "···",
 "typescript": "···",
 "webpack": "···",
 "webpack-cli": "···"
 }
}
```

属性的工作如下：

+   `"private": true`表示如果我们不提供包名称和包版本，npm 不会抱怨。

+   脚本：

    +   `tsc, tscw`：这些脚本直接调用 TypeScript 编译器。如果我们使用`ts-loader`，则不需要它们。但是，如果我们在不使用`ts-loader`的情况下使用 webpack（如本章末尾所示），它们是有用的。

    +   `wp`：运行 webpack 一次，编译所有内容。

    +   `wpw`：以监视模式运行 webpack，它会监视输入文件，并只编译更改的文件。

    +   `serve`：运行服务器`http-server`并提供完全组装的 Web 应用程序的目录`build/`。

+   依赖项：

    +   与 webpack 相关的四个软件包：

        +   `webpack`：webpack 的核心

        +   `webpack-cli`：核心的命令行界面

        +   `ts-loader`：用于将`.ts`文件编译为 JavaScript 的*加载器*

        +   `copy-webpack-plugin`：一个*插件*，将文件从一个位置复制到另一个位置

    +   `ts-loader`所需：`typescript`

    +   为 Web 应用提供服务：`http-server`

    +   TypeScript 代码使用的库加上类型定义：`lodash`，`@types/lodash`

### 9.5 `webpack.config.js`

这是我们配置 webpack 的方式：

```ts
const path = require('path');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
 ···
 entry: {
 main: "./ts/src/main.ts",
 },
 output: {
 path: path.resolve(__dirname, 'build'),
 filename: "[name]-bundle.js",
 },
 resolve: {
 // Add ".ts" and ".tsx" as resolvable extensions.
 extensions: [".ts", ".tsx", ".js"],
 },
 module: {
 rules: [
 // all files with a `.ts` or `.tsx` extension will be handled by `ts-loader`
 { test: /\.tsx?$/, loader: "ts-loader" },
 ],
 },
 plugins: [
 new CopyWebpackPlugin([
 {
 from: './html',
 }
 ]),
 ],
};
```

属性：

+   `entry`：*入口点*是 webpack 开始收集输出包数据的文件。首先将入口点文件添加到包中，然后是入口点的导入，然后是导入的导入，依此类推。属性`entry`的值是一个对象，其属性键指定入口点的名称，其属性值指定入口点的路径。

+   `output`指定输出包的路径。当存在多个入口点（因此存在多个输出包）时，`[name]`主要有用。在组装路径时，它将被入口点的名称替换。 

+   `resolve`配置 webpack 如何将模块的*规范符*（ID）转换为文件的位置。

+   `module`配置*加载程序*（处理文件的插件）等。

+   `plugins`配置*插件*，可以以各种方式更改和增强 webpack 的行为。

有关配置 webpack 的更多信息，请参阅[webpack 网站](https://webpack.js.org/concepts/)。

### 9.6 `tsconfig.json`

此文件配置 TypeScript 编译器：

```ts
{
 "compilerOptions": {
 "rootDir": "ts",
 "outDir": "dist",
 "target": "es2019",
 "lib": [
 "es2019",
 "dom"
 ],
 "module": "commonjs",
 "esModuleInterop": true,
 "strict": true,
 "sourceMap": true
 }
}
```

如果我们使用`ts-loader`与 webpack，则不需要选项`outDir`。但是，如果我们在本章后面解释的情况下使用 webpack 而不使用加载程序，则需要它。

### 9.7 `index.html`

这是 Web 应用程序的 HTML 页面：

```ts
<!doctype html>
<html>
<head>
 <meta charset="UTF-8">
 <title>ts-demo-webpack</title>
</head>
<body>
 <div id="output"></div>
 <script src="main-bundle.js"></script>
</body>
</html>
```

具有 ID`"output"`的`<div>`是 Web 应用程序显示其输出的位置。`main-bundle.js`包含捆绑代码。

### 9.8 `main.ts`

这是 Web 应用程序的 TypeScript 代码：

```ts
import template from 'lodash/template';

const outputElement = document.getElementById('output');
if (outputElement) {
 const compiled = template(`
 <h1><%- heading %></h1>
 Current date and time: <%- dateTimeString %>
 `.trim());
 outputElement.innerHTML = compiled({
 heading: 'ts-demo-webpack',
 dateTimeString: new Date().toISOString(),
 });
}
```

+   步骤 1：我们使用[Lodash 的函数`template()`](https://lodash.com/docs/4.17.15#template)将具有自定义模板语法的字符串转换为函数`compiled()`，该函数将数据映射到 HTML。字符串定义了两个要通过数据填充的空白：

    +   `<%- heading %>`

    +   `<%- dateTimeString %>`

+   步骤 2：将`compiled()`应用于数据（具有两个属性的对象）以生成 HTML。

### 9.9 安装，构建和运行 Web 应用程序

首先，我们需要安装我们的 web 应用程序依赖的所有 npm 包：

```ts
npm install
```

然后，我们需要通过`package.json`中的脚本运行在上一步安装的 webpack：

```ts
npm run wpw
```

从现在开始，webpack 会监视存储库中的文件以进行更改，并在检测到任何更改时重新构建 web 应用程序。

在另一个命令行中，我们现在可以启动一个 Web 服务器，该服务器在本地主机上提供`build/`的内容：

```ts
npm run serve
```

如果我们转到 Web 服务器打印的 URL，我们可以看到 Web 应用程序正在运行。

请注意，简单的重新加载可能不足以在更改后看到结果-由于缓存。您可能需要在重新加载时按住 shift 键来强制重新加载。

#### 9.9.1 在 Visual Studio Code 中构建

除了从命令行构建外，我们还可以通过 Visual Studio Code 内部进行构建，通过所谓的*构建任务*：

+   从“终端”菜单中执行“配置默认构建任务…”。

+   选择“npm: wpw”。

+   *问题匹配器*处理工具输出到*问题*（信息，警告和错误）列表的转换。默认情况下，在这种情况下工作良好。如果要明确，可以在`.vscode/tasks.json`中指定一个值：

    ```ts
    "problemMatcher": ["$tsc-watch"],
    ```

我们现在可以通过“终端”菜单中的“运行构建任务…”来启动 webpack。

### 9.10 在没有加载程序的情况下使用 webpack：`webpack-no-loader.config.js`

除了使用`ts-loader`之外，我们还可以首先将 TypeScript 文件编译为 JavaScript 文件，然后通过 webpack 捆绑这些文件。前两个步骤中的第一个步骤的工作原理在上一章中有描述。

现在我们不必配置`ts-loader`，我们的 webpack 配置文件更简单：

```ts
const path = require('path');

module.exports = {
 entry: {
 main: "./dist/src/main.js",
 },
 output: {
 path: path.join(__dirname, 'build'),
 filename: '[name]-bundle.js',
 },
 plugins: [
 new CopyWebpackPlugin([
 {
 from: './html',
 }
 ]),
 ],
};
```

请注意，`entry.main`是不同的。在另一个配置文件中，它是：

```ts
"./ts/src/main.ts"
```

为什么我们要在捆绑之前生成中间文件？一个好处是我们可以使用 Node.js 运行一些 TypeScript 代码的单元测试。

[评论](https://github.com/rauschma/tackling-ts/issues/9)
