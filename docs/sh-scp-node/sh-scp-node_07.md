# 五、包：JavaScript 的软件分发单元

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_packages.html`](https://exploringjs.com/nodejs-shell-scripting/ch_packages.html)

* * *

+   5.1 什么是包？

    +   5.1.1 发布包：包注册表，包管理器，包名称

+   5.2 包的文件系统布局

    +   5.2.1 `package.json`

    +   5.2.2 `package.json`的`"dependencies"`属性

    +   5.2.3 `package.json`的`"bin"`属性

    +   5.2.4 `package.json`的`"license"`属性

+   5.3 归档和安装包

    +   5.3.1 从 git 安装包

    +   5.3.2 创建新包并安装依赖项

+   5.4 通过*规范*引用模块

    +   5.4.1 模块规范中的文件扩展名

+   5.5 Node.js 中的模块规范

    +   5.5.1 在 Node.js 中解析模块规范

    +   5.5.2 包导出：控制其他包看到的内容

    +   5.5.3 包导入

    +   5.5.4 `node:`协议导入

* * *

本章解释了 npm 包是什么以及它们如何与 ESM 模块交互。

**必需的知识：**我假设您对 ECMAScript 模块的语法略有了解。如果没有，您可以阅读“JavaScript for impatient programmers”中的[章节“modules”](https://exploringjs.com/impatient-js/ch_modules.html)。

### 5.1 什么是包？

在 JavaScript 生态系统中，*包*是组织软件项目的一种方式：它是一个具有标准布局的目录。包可以包含各种文件 - 例如：

+   用 JavaScript 编写的 Web 应用程序，部署在服务器上

+   JavaScript 库（用于 Node.js，浏览器，所有 JavaScript 平台等）

+   除 JavaScript 之外的其他编程语言的库：TypeScript，Rust 等

+   单元测试（例如包中的库）

+   *Bin scripts* – 基于 Node.js 的 shell 脚本 – 例如，开发工具，如编译器，测试运行器和文档生成器

+   许多其他类型的工件

包可以*依赖于*其他包（称为*依赖项*），其中包含：

+   包的 JavaScript 代码所需的库

+   开发过程中使用的 shell 脚本

+   等等。

包的依赖项安装在该包内部（我们很快就会看到）。

包之间的一个常见区别是：

+   *已发布的包*可以由我们安装：

    +   全局安装：我们可以全局安装它们，以便它们的 bin 脚本在命令行中可用。

    +   本地安装：我们可以将它们作为依赖项安装到我们自己的包中。它们的 bin 脚本可以在本地使用（我们很快就会看到）。

+   *未发布的包*永远不会成为其他包的依赖项，但它们本身有依赖项。例如，部署到服务器的 Web 应用程序。

下一小节将解释如何发布包。

#### 5.1.1 发布包：包注册表，包管理器，包名称

发布包的主要方式是将其上传到包注册表 - 一个在线软件仓库。事实上的标准是[*npm 注册表*](https://www.npmjs.com)，但这不是唯一的选择。例如，公司可以托管自己的内部注册表。

*包管理器*是一个命令行工具，它从注册表（或其他来源）下载包并在本地或全局安装它们。如果一个包包含 bin 脚本，它也会在本地或全局提供这些脚本。

最流行的包管理器称为*npm*，并与 Node.js 捆绑在一起。它的名称最初代表“Node Package Manager”。后来，当 npm 和 npm 注册表不仅用于 Node.js 包时，定义被更改为“npm 不是一个包管理器”（[来源](https://en.wikipedia.org/wiki/Npm_(software)#Acronym)）。

还有其他流行的包管理器，如 yarn 和 pnpm。所有这些包管理器默认使用 npm 注册表。

npm 注册表中的每个包都有一个名称。有两种名称：

+   *全局名称*在整个注册表中是唯一的。这是两个例子：

    ```js
    minimatch
    mocha
    ```

+   *作用域名称*由两部分组成：作用域和名称。作用域是全局唯一的，名称在作用域内是唯一的。这是两个例子：

    ```js
    @babel/core
    @rauschma/iterable
    ```

    范围从`@`符号开始，并用斜杠与名称分隔。

### 5.2 包的文件系统布局

一旦包`my-package`完全安装，它几乎总是看起来像这样：

```js
my-package/
  package.json
  node_modules/
  [More files]
```

这些文件系统条目的目的是什么？

+   `package.json`是每个包都必须拥有的文件：

    +   它包含描述包的元数据（名称、版本、作者等）。

    +   它列出了包的依赖项：它所需的其他包，如库和工具。对于每个依赖项，我们记录：

        +   一系列版本号。不指定特定版本允许升级和依赖项之间的代码共享。

        +   默认情况下，依赖项来自 npm 注册表。但我们也可以指定其他来源：本地目录，GZIP 文件，指向 GZIP 文件的 URL，不同于 npm 的注册表，git 存储库等。

+   `node_modules/`是包的依赖项安装的目录。每个依赖项也有一个带有其依赖项等的`node_modules`文件夹。结果是一个依赖项树。

一些包还有文件`package-lock.json`，它位于`package.json`旁边：它记录了安装的依赖项的确切版本，并且如果我们通过 npm 添加更多依赖项，它会保持更新。

#### 5.2.1 `package.json`

这是一个可以通过 npm 创建的起始`package.json`：

```js
{
 "name": "my-package",
 "version": "1.0.0",
 "description": "",
 "main": "index.js",
 "scripts": {
 "test": "echo \"Error: no test specified\" && exit 1"
 },
 "keywords": [],
 "author": "",
 "license": "ISC"
}
```

这些属性的目的是什么？

+   一些属性对于公共包（发布在 npm 注册表上）是必需的：

    +   `name`指定了这个包的名称。

    +   `version`用于版本管理，并遵循[语义化版本](https://semver.org)，由三个用点分隔的数字组成：

        +   *主要版本*在不兼容的 API 更改时递增。

        +   *次要版本*在向后兼容的方式下添加功能时递增。

        +   *补丁版本*在进行了不会真正改变功能的小更改时递增。

+   公共包的其他属性是可选的：

    +   `description`、`keywords`、`author`是可选的，使找到包变得更容易。

    +   `license`澄清了这个包如何被使用。如果包在任何方面是公共的，提供这个值是有意义的。[“选择一个开源许可证”](https://choosealicense.com)可以帮助做出这个选择。

+   `main`是用于包含库代码的包的属性。它指定了“是”包的模块（在本章后面解释）。

+   `scripts`是用于设置*package scripts*的属性——开发时 shell 命令的缩写。这些可以通过`npm run`执行。例如，脚本`test`可以通过`npm run test`执行。有关此主题的更多信息，请参阅§15“通过 npm 软件包脚本运行跨平台任务”。

其他有用的属性：

+   `dependencies`列出了软件包的依赖关系。其格式很快就会解释。

+   `devDependencies`是仅在开发过程中需要的依赖关系。

+   以下设置意味着所有具有扩展名`.js`的文件都被解释为 ECMAScript 模块。除非我们处理旧代码，否则添加它是有意义的：

    ```js
    "type": "module"
    ```

+   `bin`列出了 npm 将其安装为 shell 脚本的软件包内的 Node.js 模块的*bin scripts*。其格式很快就会解释。

+   `license`指定软件包的许可证。其格式很快就会解释。

+   通常，`name`和`version`属性是必需的，如果缺少它们，npm 会发出警告。但是，我们可以通过以下设置更改：

    ```js
    "private": true
    ```

    这可以防止软件包意外发布，并允许我们省略名称和版本。

有关`package.json`的更多信息，请参阅[npm 文档](https://docs.npmjs.com/files/package.json)。

#### 5.2.2 `package.json`的属性`"dependencies"`

这是`package.json`文件中的依赖关系的样子：

```js
"dependencies": {
 "minimatch": "⁵.1.0",
 "mocha": "¹⁰.0.0"
}
```

属性记录了软件包的名称和其版本的约束。

版本本身遵循[语义化版本](https://semver.org)标准。它们由点分隔的最多三个数字组成（第二个和第三个数字是可选的，默认为零）：

1.  *主要版本*：当软件包以不兼容的方式更改时，此数字会更改。

1.  *次要版本*：当以向后兼容的方式添加功能时，此数字会更改。

1.  *补丁版本*：当进行向后兼容的错误修复时，此数字会更改。

有关 Node 的版本范围，请参阅[semver 存储库](https://github.com/npm/node-semver#versions)。示例包括：

+   没有任何额外字符的特定版本意味着安装的版本必须完全匹配：

    ```js
    "pkg1": "2.0.1",
    ```

+   `major.minor.x`或`major.x`表示数字组件必须匹配，`x`或省略的组件可以具有任何值：

    ```js
    "pkg2": "2.x",
    "pkg3": "3.3.x",
    ```

+   `*`匹配任何版本：

    ```js
    "pkg4": "*",
    ```

+   `>=version`表示安装的版本必须是`version`或更高：

    ```js
    "pkg5": ">=1.0.2",
    ```

+   `<=version`表示安装的版本必须是`version`或更低：

    ```js
    "pkg6": "<=2.3.4",
    ```

+   `version1-version2`与`>=version1 <=version2`相同：

    ```js
    "pkg7": "1.0.0 - 2.9999.9999",
    ```

+   `^version`（如前面的示例中使用的）是一个*caret range*，意味着安装的版本可以是`version`或更高，但不得引入破坏性更改。也就是说，主要版本必须相同：

    ```js
    "pkg8": "⁴.17.21",
    ```

#### 5.2.3 `package.json`的属性`"bin"`

这是我们告诉 npm 将模块安装为 shell 脚本的方法：

```js
"bin": {
 "my-shell-script": "./src/shell/my-shell-script.mjs",
 "another-script": "./src/shell/another-script.mjs"
}
```

如果我们使用全局安装具有此`"bin"`值的软件包，Node.js 会确保命令`my-shell-script`和`another-script`在命令行上可用。

如果我们在本地安装软件包，可以在软件包脚本中使用这两个命令，或者通过[npx 命令](https://docs.npmjs.com/cli/v8/commands/npx)使用。

`"bin"`的值也可以是字符串：

```js
{
 "name": "my-package",
 "bin": "./src/main.mjs"
}
```

这是对的缩写：

```js
{
 "name": "my-package",
 "bin": {
 "my-package": "./src/main.mjs"
 }
}
```

#### 5.2.4 `package.json`的属性`"license"`

属性`"license"`的值始终是一个带有 SPDX 许可证 ID 的字符串。例如，以下值拒绝其他人以任何条款使用软件包（如果软件包未发布，则这很有用）：

```js
"license": "UNLICENSED"
```

[SPDX 网站列出了所有可用的许可证 ID](https://spdx.org/licenses/)。如果您发现很难选择一个，[“选择开源许可证”网站](https://choosealicense.com)可以帮助您——例如，如果您“希望它简单和宽松”，这是建议：

> MIT 许可证简短而直接。它允许人们几乎可以为项目做任何他们想做的事情，比如制作和分发闭源版本。
> 
> Babel、.NET 和 Rails 使用 MIT 许可证。

你可以像这样使用许可证：

```js
"license": "MIT"
```

### 5.3 存档和安装包

npm 注册表中的包通常以两种不同的方式存档：

+   在开发过程中，它们存储在 git 仓库中。

+   为了使它们可以通过 npm 安装，它们被上传到 npm 注册表。

无论哪种方式，包都会被存档，不包括它的依赖项 - 我们必须在使用之前安装它们。

如果一个包存储在 git 仓库中：

+   通常情况下，我们希望每次安装包时都使用相同的依赖树。

    +   这就是为什么通常会包含`package-lock.json`。

+   我们可以从其他工件中重新生成工件 - 例如，将 TypeScript 文件编译为 JavaScript 文件。

如果一个包发布到 npm 注册表：

+   它应该灵活地处理其依赖关系，以便升级依赖关系并在依赖树中共享包成为可能。

    +   这就是为什么`package-lock.json`永远不会上传到 npm 注册表的原因。

+   它通常包含生成的工件 - 例如，从 TypeScript 文件编译的 JavaScript 文件被包含在内，这样只使用 JavaScript 的人就不必安装 TypeScript 编译器。

开发依赖项（`package.json`中的`devDependencies`属性）只在开发过程中安装，而不是在我们从 npm 注册表安装包时安装。

请注意，git 仓库中未发布的包在开发过程中与已发布的包类似处理。

#### 5.3.1 从 git 安装包

要安装一个名为`pkg`的包，我们克隆它的存储库并：

```js
cd pkg/
npm install
```

然后执行以下步骤：

+   `node_modules`被创建并安装依赖项。安装一个依赖项也意味着下载该依赖项并安装它的依赖项（等等）。

+   有时会执行额外的设置步骤。可以通过`package.json`配置这些步骤。

如果根包没有`package-lock.json`文件，则在安装过程中会创建该文件（如前所述，依赖项没有此文件）。

在依赖树中，相同的依赖项可能存在多次，可能是不同的版本。有一些方法可以最小化重复，但这超出了本章的范围。

##### 5.3.1.1 重新安装一个包

这是一种（略显粗糙）修复依赖树中问题的方法：

```js
cd pkg/
rm -rf node_modules/
rm package-lock.json
npm install
```

请注意，这可能导致安装不同的、更新的包。我们可以通过不删除`package-lock.json`来避免这种情况。

#### 5.3.2 创建一个新的包并安装依赖项

有许多工具和技术可以设置新的包。这是一个简单的方法：

```js
mkdir my-package
cd my-package/
npm init --yes
```

之后，目录看起来像这样：

```js
my-package/
  package.json
```

`package.json`中包含了我们已经看到的起始内容。

##### 5.3.2.1 安装依赖项

现在，`my-package`没有任何依赖项。假设我们想要使用库`lodash-es`。这是我们将其安装到我们的包中的方法：

```js
npm install lodash-es
```

该命令执行以下步骤：

+   该包被下载到`my-package/node_modules/lodash-es`中。

+   也会安装它的依赖项。然后是它的依赖项的依赖项。等等。

+   `package.json`中添加了一个新属性：

    ```js
    "dependencies": {
     "lodash-es": "⁴.17.21"
    }
    ```

+   `package-lock.json`会更新为安装的确切版本。

### 5.4 通过*标识符*引用模块

ECMAScript 模块中的代码通过`import`语句（A 行和 B 行）访问：

```js
// Static import
import {namedExport} from 'https://example.com/some-module.js'; // (A)
console.log(namedExport);

// Dynamic import
import('https://example.com/some-module.js') // (B)
.then((moduleNamespace) => {
 console.log(moduleNamespace.namedExport);
});
```

静态导入和动态导入都使用*模块标识符*来引用模块：

+   A 行中`from`后面的字符串。

+   B 行中的字符串参数。

有三种类型的模块标识符：

+   *绝对标识符*是完整的 URL - 例如：

    ```js
    'https://www.unpkg.com/browse/yargs@17.3.1/browser.mjs'
    'file:///opt/nodejs/config.mjs'
    ```

    绝对标识符主要用于访问直接托管在网络上的库。

+   *相对标识符*是相对 URL（以`'/'`、`'./'`或`'../'`开头） - 例如：

    ```js
    './sibling-module.js'
    '../module-in-parent-dir.mjs'
    '../../dir/other-module.js'
    ```

    每个模块都有一个 URL，其协议取决于其位置（`file:`、`https:`等）。如果它使用相对标识符，JavaScript 会通过将其解析为模块的 URL 来将该标识符转换为完整的 URL。

    相对标识符主要用于访问同一代码库中的其他模块。

+   *裸 specifier*是以包的名称开头的路径（没有协议和域）。这些名称可以选择后跟*子路径*：

    ```js
    'some-package'
    'some-package/sync'
    'some-package/util/files/path-tools.js'
    ```

    裸 specifier 也可以指向具有作用域名称的包：

    ```js
    '@some-scope/scoped-name'
    '@some-scope/scoped-name/async'
    '@some-scope/scoped-name/dir/some-module.mjs'
    ```

    每个裸 specifier 都指向包内的一个模块；如果没有子路径，则指向其包的指定“主”模块。裸 specifier 永远不会直接使用，而是总是*解析* - 转换为绝对 specifier。解析的工作方式取决于平台。我们很快就会了解更多。

#### 5.4.1 模块 specifier 中的文件扩展名

+   绝对 specifier 和相对 specifier 总是带有文件扩展名-通常是`.js`或`.mjs`。

+   有三种裸 specifier 的样式：

    +   样式 1：没有子路径

    +   样式 2：没有文件扩展名的子路径。在这种情况下，子路径的作用类似于包名称的修饰符：

        ```js
        'my-parser/sync'
        'my-parser/async'

        'assertions'
        'assertions/strict'
        ```

    +   样式 3：带有文件扩展名的子路径。在这种情况下，包被视为模块的集合，子路径指向其中一个：

        ```js
        'large-package/misc/util.js'
        'large-package/main/parsing.js'
        'large-package/main/printing.js'
        ```

裸 specifier 样式 3 的注意事项：文件扩展名的解释取决于依赖项，可能与导入包不同。例如，导入包可能对 ESM 模块使用`.mjs`，对 CommonJS 模块使用`.js`，而依赖项导出的 ESM 模块可能具有带有文件扩展名`.js`的裸路径。

### 5.5 Node.js 中的模块 specifier

让我们看看 Node.js 中模块 specifier 的工作原理。

#### 5.5.1 Node.js 中解析模块 specifier

[*Node.js 解析算法*](https://nodejs.org/api/esm.html#resolution-algorithm)的工作如下：

+   参数：

    +   导入模块的 URL

    +   模块 specifier

+   结果：模块 specifier 的解析 URL

这是算法：

+   如果 specifier 是绝对的，解析已经完成。三个协议最常见：

    +   `file:`用于本地文件

    +   `https:`用于远程文件

    +   `node:`用于内置模块（稍后讨论）

+   如果 specifier 是相对的，它将根据导入模块的 URL 进行解析。

+   如果一个 specifier 是裸的：

    +   如果以`'#'`开头，则通过在*包导入*中查找它并解析结果来解析它（稍后将解释）。

    +   否则，它是一个具有以下格式之一的裸 specifier（子路径是可选的）：

        +   `«package»/sub/path`

        +   `@«scope»/«scoped-package»/sub/path`

        解析算法遍历当前目录及其祖先，直到找到一个具有与裸 specifier 开头匹配的子目录`node_modules`，即：

        +   `node_modules/«package»/`

        +   `node_modules/@«scope»/«scoped-package»/`

        该目录是包的目录。默认情况下，包 ID 后的（可能为空的）子路径被解释为相对于包目录。默认值可以通过下面将要解释的*包出口*来覆盖。

解析算法的结果必须指向一个文件。这就解释了为什么绝对 specifier 和相对 specifier 总是带有文件扩展名。裸 specifier 大多数情况下没有，因为它们是在包出口中查找的缩写。

模块文件通常具有这些文件扩展名：

+   如果文件的扩展名为`.mjs`，它总是一个 ES 模块。

+   如果文件的扩展名为`.js`，则最接近的`package.json`具有此条目，则它是一个 ES 模块：

    +   `"type": "module"`

如果 Node.js 执行通过 stdin、`--eval`或`--print`提供的代码，我们使用[以下命令行选项](https://nodejs.org/api/cli.html#--input-typetype)以便它被解释为 ES 模块：

```js
--input-type=module
```

#### 5.5.2 包出口：控制其他包看到什么

在本小节中，我们正在处理具有以下文件布局的包：

```js
my-lib/
  dist/
    src/
      main.js
      util/
        errors.js
      internal/
        internal-module.js
    test/
```

[*包出口*](https://nodejs.org/api/packages.html#packages_package_entry_points)通过`package.json`中的`"exports"`属性指定，并支持两个重要功能：

+   隐藏包的内部：

    +   没有`"exports"`属性，包`my-lib`中的每个模块都可以在包名后使用相对路径访问 - 例如：

        ```js
        'my-lib/dist/src/internal/internal-module.js'
        ```

    +   一旦属性存在，只能使用其中列出的指定符。其他所有内容都对外部隐藏。

+   更好的模块指定符：包出口让我们为较短和/或名称更好的模块定义裸指定符子路径。

回想一下裸指定符的三种样式：

+   样式 1：没有子路径的裸指定符

+   样式 2：没有扩展名的裸指定符

+   样式 3：带有扩展名的裸指定符子路径

包出口帮助我们处理所有三种样式

##### 5.5.2.1 样式 1：配置哪个文件代表（包的裸指定符）

`package.json`：

```js
{
 "main": "./dist/src/main.js",
 "exports": {
 ".": "./dist/src/main.js"
 }
}
```

我们只提供`"main"`是为了向后兼容（与旧的捆绑器和 Node.js 12 及更旧版本）。否则，`"."`的条目就足够了。

有了这些包出口，我们现在可以这样从`my-lib`导入。

```js
import {someFunction} from 'my-lib';
```

这导入了`someFunction()`从这个文件：

```js
my-lib/dist/src/main.js
```

##### 5.5.2.2 样式 2：将不带扩展名的子路径映射到模块文件

`package.json`：

```js
{
 "exports": {
 "./util/errors": "./dist/src/util/errors.js"
 }
}
```

我们将指定符子路径`'util/errors'`映射到一个模块文件。这使得以下导入成为可能：

```js
import {UserError} from 'my-lib/util/errors';
```

##### 5.5.2.3 样式 2：更好的不带扩展名的子路径为子树

前一小节解释了如何为不带扩展名的子路径创建单个映射。还有一种方法可以通过单个条目创建多个这样的映射：

`package.json`：

```js
{
 "exports": {
 "./lib/*": "./dist/src/*.js"
 }
}
```

任何位于`./dist/src/`下的文件现在都可以在不带文件扩展名的情况下导入：

```js
import {someFunction} from 'my-lib/lib/main';
import {UserError}    from 'my-lib/lib/util/errors';
```

请注意这个`"exports"`条目中的星号：

```js
"./lib/*": "./dist/src/*.js"
```

这些更多的指令是如何将子路径映射到实际路径，而不是匹配文件路径片段的通配符。

##### 5.5.2.4 样式 3：将带有扩展名的子路径映射到模块文件

`package.json`：

```js
{
 "exports": {
 "./util/errors.js": "./dist/src/util/errors.js"
 }
}
```

我们将指定符子路径`'util/errors.js'`映射到一个模块文件。这使得以下导入成为可能：

```js
import {UserError} from 'my-lib/util/errors.js';
```

##### 5.5.2.5 样式 3：更好的带有扩展名的子路径为子树

`package.json`：

```js
{
 "exports": {
 "./*": "./dist/src/*"
 }
}
```

在这里，我们缩短了`my-package/dist/src`下整个子树的模块指定符：

```js
import {InternalError} from 'my-package/util/errors.js';
```

没有出口，导入语句将是：

```js
import {InternalError} from 'my-package/dist/src/util/errors.js';
```

请注意这个`"exports"`条目中的星号：

```js
"./*": "./dist/src/*"
```

这些不是文件系统通配符，而是如何将外部模块指定符映射到内部模块指定符的指令。

##### 5.5.2.6 暴露子树同时隐藏其中的部分

通过以下技巧，我们暴露了`my-package/dist/src/`目录中的所有内容，但除了`my-package/dist/src/internal/`

```js
"exports": {
 "./*": "./dist/src/*",
 "./internal/*": null
}
```

请注意，这个技巧在*不带*文件名扩展名的情况下导出子树时也适用。

##### 5.5.2.7 条件包出口

我们还可以使出口[*条件*](https://nodejs.org/api/packages.html#packages_conditional_exports)：然后给定的路径根据包在其中使用的上下文而映射到不同的值。

**Node.js vs. 浏览器。** 例如，我们可以为 Node.js 和浏览器提供不同的实现：

```js
"exports": {
 ".": {
 "node": "./main-node.js",
 "browser": "./main-browser.js",
 "default": "./main-browser.js"
 }
}
```

`"default"`条件在没有其他键匹配时匹配，并且必须放在最后。每当我们区分平台时，建议使用它，因为它负责新的和/或未知的平台。

**开发 vs. 生产。** 条件包出口的另一个用例是在“开发”和“生产”环境之间切换：

```js
"exports": {
 ".": {
 "development": "./main-development.js",
 "production": "./main-production.js",
 }
}
```

在 Node.js 中，我们可以这样指定环境：

```js
node --conditions development app.mjs
```

#### 5.5.3 包导入

[包导入](https://nodejs.org/api/packages.html#imports)让一个包为模块指定符定义缩写，它可以在内部自己使用（其中包出口为其他包定义了缩写）。这是一个例子：

`package.json`：

```js
{
 "imports": {
 "#some-pkg": {
 "node": "some-pkg-node-native",
 "default": "./polyfills/some-pkg-polyfill.js"
 }
 },
 "dependencies": {
 "some-pkg-node-native": "¹.2.3"
 }
}
```

包导入`#`是*条件*的（具有与条件包出口相同的功能）：

+   如果当前包在 Node.js 上使用，则模块指定符`'#some-pkg'`指的是包`some-pkg-node-native`。

+   在其他地方，`'#some-pkg'`指的是当前包内的`./polyfills/some-pkg-polyfill.js`文件。

（只有包引入可以引用外部包，包导出不能这样做。）

包引入的用例是什么？

+   通过相同的模块标识符引用不同的特定于平台的实现模块（如上所示）。

+   当前包内部模块的别名 - 避免使用相对路径（在嵌套目录中可能会变得复杂）。

在使用打包工具时要小心包引入：这个功能相对较新，你的打包工具可能不支持它。

#### 5.5.4 `node:`协议导入

Node.js 有许多内置模块，比如'path'和'fs'。它们都可以作为 ES 模块和 CommonJS 模块使用。它们的一个问题是它们可能会被安装在`node_modules`中的模块覆盖，这既是安全风险（如果意外发生）也是一个问题，如果 Node.js 想要在未来引入新的内置模块并且它们的名称已经被 npm 包占用。

我们可以使用[node:协议](https://nodejs.org/api/esm.html#node-imports)来明确表示我们想要导入一个内置模块。例如，以下两个导入语句在大多数情况下是等效的（如果没有安装名为'fs'的 npm 模块）：

```js
import * as fs from 'node:fs/promises';
import * as fs from 'fs/promises';
```

使用`node:`协议的另一个好处是我们立即看到导入的模块是内置的。考虑到有多少内置模块，这在阅读代码时很有帮助。

由于`node:`标识符具有协议，它们被认为是绝对的。这就是为什么它们不会在`node_modules`中查找的原因。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/5)
