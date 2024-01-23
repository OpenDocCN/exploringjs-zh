# 8 通过 TypeScript 创建基于 CommonJS 的 npm 包

> 原文：[`exploringjs.com/tackling-ts/ch_npm-cjs-typescript.html`](https://exploringjs.com/tackling-ts/ch_npm-cjs-typescript.html)

* * *

+   8.1 所需知识

+   8.2 限制

+   8.3 存储库`ts-demo-npm-cjs`

+   8.4 `.gitignore`

+   8.5 `.npmignore`

+   8.6 `package.json`

    +   8.6.1 脚本

    +   8.6.2 `dependencies` vs. `devDependencies`

    +   8.6.3 更多关于`package.json`的信息

+   8.7 `tsconfig.json`

+   8.8 TypeScript 代码

    +   8.8.1 `index.ts`

    +   8.8.2 `index_test.ts`

* * *

本章描述了如何使用 TypeScript 为基于 CommonJS 模块格式的 npm 包创建包。

![](img/ef04c4c2601874494d82453a9b525b87.png)  **GitHub 存储库：`ts-demo-npm-cjs`**

在本章中，我们正在探索[存储库`ts-demo-npm-cjs`](https://github.com/rauschma/ts-demo-npm-cjs)，可以在 GitHub 上下载。（我故意没有将其发布为 npm 包。）

### 8.1 所需知识

您应该对以下内容有大致了解：

+   [*CommonJS 模块*](https://nodejs.org/api/modules.html) – 一种起源于服务器端 JavaScript 并为其设计的模块格式。它由服务器端 JavaScript 平台[*Node.js*](https://nodejs.org/)推广。CommonJS 模块先于 JavaScript 的内置[ECMAScript 模块](https://exploringjs.com/impatient-js/ch_modules.html)出现，并且仍然被工具（IDE、构建工具等）广泛使用和支持良好。

+   [TypeScript 的模块](https://www.typescriptlang.org/docs/handbook/modules.html) – 其语法基于 ECMAScript 模块。但它们经常被编译为 CommonJS 模块。

+   [npm 包](https://docs.npmjs.com/packages-and-modules/) – 通过 npm 包管理器安装的包含文件的目录。它们可以包含 CommonJS 模块、ECMAScript 模块和各种其他文件。

### 8.2 限制

在本章中，我们正在使用 TypeScript 目前最好支持的内容：

+   我们所有的 TypeScript 代码都被编译为带有文件扩展名`.js`的 CommonJS 模块。

+   所有外部导入也是 CommonJS 模块。

特别是在 Node.js 上，TypeScript 目前并不真正支持 ECMAScript 模块和除`.js`以外的文件扩展名。

### 8.3 存储库`ts-demo-npm-cjs`

这就是存储库`ts-demo-npm-cjs`的结构：

```ts
ts-demo-npm-cjs/
  .gitignore
  .npmignore
  dist/   (created on demand)
  package.json
  ts/
    src/
      index.ts
    test/
      index_test.ts
  tsconfig.json
```

除了用于包的`package.json`之外，存储库还包含：

+   `ts/src/index.ts`：包的实际代码

+   `ts/test/index_test.ts`：`index.ts`的测试

+   `tsconfig.json`：TypeScript 编译器的配置数据

`package.json`包含用于编译的脚本：

+   输入：目录`ts/`（TypeScript 代码）

+   输出：目录`dist/`（CommonJS 模块；该目录尚不存在于存储库中）

这是两个 TypeScript 文件的编译结果所在的地方：

```ts
ts/src/index.ts       --> dist/src/index.js
ts/test/index_test.ts --> dist/test/index_test.js
```

### 8.4 `.gitignore`

这个文件列出了我们不想检入 git 的目录：

```ts
node_modules/
dist/
```

解释：

+   `node_modules/`是通过`npm install`设置的。

+   `dist/`目录中的文件是由 TypeScript 编译器创建的（稍后会详细介绍）。

### 8.5 `.npmignore`

在确定应该上传哪些文件到 npm 注册表时，我们有不同于 git 的需求。因此，除了`.gitignore`之外，我们还需要文件`.npmignore`：

```ts
ts/
```

两个不同之处是：

+   我们希望上传 TypeScript 编译成 JavaScript 的结果（目录`dist/`）。

+   我们不想上传 TypeScript 源文件（目录`ts/`）。

请注意，npm 默认忽略`node_modules/`目录。

### 8.6 `package.json`

`package.json`看起来像这样：

```ts
{
 ···
 "type": "commonjs",
 "main": "./dist/src/index.js",
 "types": "./dist/src/index.d.ts",
 "scripts": {
 "clean": "shx rm -rf dist/*",
 "build": "tsc",
 "watch": "tsc --watch",
 "test": "mocha --ui qunit",
 "testall": "mocha --ui qunit dist/test",
 "prepack": "npm run clean && npm run build"
 },
 "// devDependencies": {
 "@types/node": "Needed for unit test assertions (assert.equal() etc.)",
 "shx": "Needed for development-time package.json scripts"
 },
 "devDependencies": {
 "@types/lodash": "···",
 "@types/mocha": "···",
 "@types/node": "···",
 "mocha": "···",
 "shx": "···"
 },
 "dependencies": {
 "lodash": "···"
 }
}
```

让我们来看看这些属性：

+   `type`：值`"commonjs"`表示`.js`文件被解释为 CommonJS 模块。

+   `主要`：如果有所谓的*裸导入*，只提到当前包的名称，那么这就是将被导入的模块。

+   `types`指向一个声明文件，其中包含当前包的所有类型定义。

接下来的两个小节涵盖了剩余的属性。

#### 8.6.1 脚本

属性`scripts`定义了可以通过`npm run`调用的各种命令。例如，脚本`clean`通过`npm run clean`调用。前面的`package.json`包含以下脚本：

+   `clean`使用[跨平台包`shx`](https://github.com/shelljs/shx)通过其对 Unix shell 命令`rm`的实现来删除编译结果。 `shx`支持各种 shell 命令，而无需为我们可能想要使用的每个命令单独安装包。

+   `build`和`watch`使用 TypeScript 编译器`tsc`根据`tsconfig.json`编译 TypeScript 文件。 `tsc`必须全局或本地安装（在当前包内），通常通过 npm 包`typescript`。

+   `test`和`testall`使用[单元测试框架 Mocha](https://mochajs.org/)来运行一个测试或所有测试。

+   `prepack`：这个脚本在打包 tarball 之前运行（由于`npm pack`，`npm publish`或从 git 安装）。

请注意，当我们使用 IDE 时，我们不需要脚本`build`和`watch`，因为我们可以让 IDE 构建构件。但它们对于脚本`prepack`是必需的。

#### 8.6.2 `dependencies` vs. `devDependencies`

`dependencies`应该只包含导入包时需要的包。这不包括用于运行测试等的包。

以`@types/`开头的包为那些没有 TypeScript 类型定义的包提供了 TypeScript 类型定义。没有前者，我们就不能使用后者。这些是正常的依赖项还是开发依赖项？这取决于：

+   如果我们包的类型定义引用另一个包中的类型定义，则该包是正常的依赖项。

+   否则，该包只在开发时需要，并且是开发依赖项。

#### 8.6.3 更多关于`package.json`的信息

+   [“Awesome npm scripts”](https://github.com/RyanZim/awesome-npm-scripts)提供了编写跨平台脚本的技巧。

+   [`package.json`的 npm 文档](https://docs.npmjs.com/files/package.json)解释了该文件的各种属性。

+   [`scripts`的 npm 文档](https://docs.npmjs.com/misc/scripts)解释了`package.json`属性`scripts`。

### 8.7 `tsconfig.json`

```ts
{
 "compilerOptions": {
 "rootDir": "ts",
 "outDir": "dist",
 "target": "es2019",
 "lib": [
 "es2019"
 ],
 "module": "commonjs",
 "esModuleInterop": true,
 "strict": true,
 "declaration": true,
 "sourceMap": true
 }
}
```

+   `rootDir`：我们的 TypeScript 文件位于哪里？

+   `outDir`：编译结果应该放在哪里？

+   `target`：目标 ECMAScript 版本是什么？如果 TypeScript 代码使用目标版本不支持的功能，则将其编译为仅使用支持功能的等效代码。

+   `lib`：TypeScript 应该意识到哪些平台功能？可能包括 ECMAScript 标准库和浏览器的 DOM。Node.js API 通过包`@types/node`以不同的方式得到支持。

+   `module`：指定编译输出的格式。

其余选项由[官方`tsconfig.json`文档](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)解释。

### 8.8 TypeScript 代码

#### 8.8.1 `index.ts`

该文件提供了包的实际功能：

```ts
import endsWith from 'lodash/endsWith';

export function removeSuffix(str: string, suffix: string) {
 if (!endsWith(str, suffix)) {
 throw new Error(JSON.stringify(suffix)} + ' is not a suffix of ' +
 JSON.stringify(str));
 }
 return str.slice(0, -suffix.length);
}
```

它使用[库 Lodash 的`endsWith()`函数](https://lodash.com/docs/4.17.15#endsWith)。这就是为什么 Lodash 是一个正常的依赖项-它在运行时需要。

#### 8.8.2 `index_test.ts`

该文件包含了对`index.ts`的单元测试。

```ts
import { strict as assert } from 'assert';
import { removeSuffix } from '../src/index';

test('removeSuffix()', () => {
 assert.equal(
 removeSuffix('myfile.txt', '.txt'),
 'myfile');
 assert.throws(() => removeSuffix('myfile.txt', 'abc'));
});
```

我们可以这样运行测试：

```ts
npm t dist/test/index_test.js
```

+   npm 命令`t`是 npm 命令`test`的缩写。

+   npm 命令`test`是`run test`的缩写（运行`package.json`中的`test`脚本）。

如您所见，我们正在运行测试的编译版本（在`dist/`目录中），而不是 TypeScript 代码。

有关单元测试框架 Mocha 的更多信息，请参阅[其主页](https://mochajs.org/)。

[评论](https://github.com/rauschma/tackling-ts/issues/8)
