# 十七、Shell 脚本配方

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_shell-scripting-recipes.html`](https://exploringjs.com/nodejs-shell-scripting/ch_shell-scripting-recipes.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   17.1 通过 nodemon 交互式编辑代码片段

    +   17.1.1 nodemon

    +   17.1.2 尝试使用 nodemon 而不安装它

+   17.2 检测当前模块是否为“main”（应用程序入口点）

+   17.3 相对于当前模块访问文件

* * *

### 17.1 通过 nodemon 交互式编辑代码片段

本节描述了在处理 JavaScript 代码片段时使用 Node.js 的技巧。

#### 17.1.1 nodemon

例如，假设我们想要尝试[标准 Node.js 函数`util.format()`](https://nodejs.org/api/util.html#util_util_format_format_args)。我们创建文件`mysnippet.mjs`，内容如下：

```js
import * as util from 'node:util';
console.log(util.format('Hello %s!', 'world'));
```

我们如何在处理它时运行`mysnippet.mjs`？

首先安装[npm 包*nodemon*](https://nodemon.io)：

```js
npm install -g nodemon
```

然后我们可以使用它来持续运行`mysnippet.mjs`：

```js
nodemon mysnippet.mjs
```

每当我们保存`mysnippet.mjs`，nodemon 会再次运行它。这意味着我们可以在编辑器中编辑该文件，并在保存时查看更改的结果。

#### 17.1.2 尝试使用 nodemon 而不安装它

甚至可以通过 npx（Node.js 工具）尝试使用 nodemon 而不安装它：

```js
npx nodemon mysnippet.mjs
```

### 17.2 检测当前模块是否为“main”（应用程序入口点）

参见§7.11.4“URL 用例：检测当前模块是否为“main”（应用程序入口点）”。

### 17.3 相对于当前模块访问文件

参见§7.11.3“URL 用例：访问相对于当前模块的文件”。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/17)
