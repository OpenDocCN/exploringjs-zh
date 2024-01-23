# 十四、[创建跨平台 shell 脚本]

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_creating-shell-scripts.html`](https://exploringjs.com/nodejs-shell-scripting/ch_creating-shell-scripts.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)



* * *

+   14.1 所需的知识

    +   14.1.1 本章的下一步是什么

+   14.2 Node.js ESM 模块作为 Unix 上独立的 shell 脚本

    +   14.2.1 Unix 上的 Node.js shell 脚本

    +   14.2.2 Unix 上的 Hashbangs

    +   14.2.3 在 Unix 上使文件可执行

    +   14.2.4 直接运行`hello.mjs`

+   14.3 使用 shell 脚本创建一个 npm 包

    +   14.3.1 设置包的目录

    +   14.3.2 添加依赖项

    +   14.3.3 向包添加内容

    +   14.3.4 在不安装的情况下运行 shell 脚本

+   14.4 npm 如何安装 shell 脚本

    +   14.4.1 在 Unix 上安装

    +   14.4.2 在 Windows 上安装

+   14.5 将示例包发布到 npm 注册表

    +   14.5.1 哪些文件被发布？哪些文件被忽略？

    +   14.5.2 检查包是否正确配置

    +   14.5.3 `npm publish`: 将包上传到 npm 注册表

    +   14.5.4 在发布之前自动执行任务

+   14.6 Unix 上任意扩展名的独立 Node.js shell 脚本

    +   14.6.1 Unix：通过自定义可执行文件设置任意文件扩展名

    +   14.6.2 Unix：通过 shell prolog 设置任意文件扩展名

+   14.7 Windows 上独立的 Node.js shell 脚本

    +   14.7.1 Windows：配置文件扩展名`.mjs`

    +   14.7.2 Windows 命令 shell：通过 shell prolog 运行 Node.js 脚本

    +   14.7.3 Windows PowerShell: 通过 shell prolog 运行 Node.js 脚本

+   14.8 为 Linux、macOS 和 Windows 创建本机二进制文件

+   14.9 Shell 路径：确保 shell 找到脚本

    +   14.9.1 Unix: `$PATH`

    +   14.9.2 在 Windows 上更改 PATH 变量（命令 shell、PowerShell）

* * *

在本章中，我们将学习如何通过 Node.js ESM 模块实现 shell 脚本。有两种常见的方法可以这样做：

+   我们可以编写一个独立的脚本并自己安装它。

+   我们可以把我们的脚本放在一个 npm 包中，并使用包管理器来安装它。这也给了我们选择将包发布到 npm 注册表的选项，这样其他人也可以安装它。

### 14.1 所需知识

你应该对以下两个主题有一定的了解：

+   ECMAScript 模块，如“JavaScript for impatient programmers”中的[章节“模块”](https://exploringjs.com/impatient-js/ch_modules.html)中所解释的。

+   npm 软件包，如§5“软件包：JavaScript 的软件分发单元”中所解释的。

#### 14.1.1 本章的下一步

Windows 实际上不支持用 JavaScript 编写的独立的 shell 脚本。因此，我们首先要了解如何为 Unix 编写带有文件扩展名的独立脚本。这些知识将帮助我们创建包含 shell 脚本的软件包。后来，我们会学到：

+   在 Windows 上编写独立的 shell 脚本的技巧。

+   在 Unix 上编写独立的 shell 脚本*不带*文件扩展名的技巧。

通过软件包安装 shell 脚本是§13“安装 npm 软件包和运行 bin 脚本”的主题。

### 14.2 Node.js ESM 模块作为 Unix 上独立的 shell 脚本

让我们将一个 ESM 模块转换为 Unix shell 脚本，这样我们就可以在不在软件包中的情况下运行它。原则上，我们可以选择 ESM 模块的两个文件扩展名：

+   `.mjs`文件总是被解释为 ESM 模块。

+   只有在最接近的`package.json`中有以下条目时，`.js`文件才会被解释为 ESM 模块：

    ```js
    "type": "module"
    ```

然而，由于我们想创建一个独立的脚本，我们不能依赖于`package.json`是否存在。因此，我们必须使用文件扩展名`.mjs`（我们稍后会介绍解决方法）。

以下文件名为`hello.mjs`：

```js
import * as os from 'node:os';
const {username} = os.userInfo();
console.log(`Hello ${username}!`);
```

我们已经可以运行这个文件：

```js
node hello.mjs
```

#### 14.2.1 Unix 上的 Node.js shell 脚本

我们需要做两件事，这样我们才能像这样运行`hello.mjs`：

```js
./hello.mjs
```

这些事情是：

+   在`hello.mjs`开头添加*哈希标记*行

+   使`hello.mjs`可执行

#### 14.2.2 Unix 上的哈希标记

在 Unix shell 脚本中，第一行是*哈希标记* - 元数据，告诉 shell 如何执行文件。例如，这是 Node.js 脚本最常见的哈希标记：

```js
#!/usr/bin/env node
```

这一行被称为“哈希标记”，因为它以井号和感叹号开头。它也经常被称为“shebang”。

如果一行以井号开头，在大多数 Unix shell（sh、bash、zsh 等）中它是一个注释。因此，这些 shell 会忽略哈希标记。Node.js 也会忽略它，但只有当它是第一行时。

为什么我们不使用这个哈希标记呢？

```js
#!/usr/bin/node
```

并非所有的 Unix 都将 Node.js 二进制文件安装在那个路径。那么这个路径呢？

```js
#!node
```

然而，并非所有的 Unix 都允许相对路径。这就是为什么我们通过绝对路径引用`env`并用它来为我们运行`node`。

有关 Unix 哈希标记的更多信息，请参见 Alex Ewerlöf 的[“Node.js shebang”](https://alexewerlof.medium.com/node-shebang-e1d4b02f731d)。

##### 14.2.2.1 将参数传递给 Node.js 二进制文件

如果我们想要传递参数，比如命令行选项给 Node.js 二进制文件怎么办？

在许多 Unix 上使用`env`的一个解决方案是使用选项`-S`，这样可以防止它将其所有参数解释为一个二进制文件的名称：

```js
#!/usr/bin/env -S node --disable-proto=throw
```

在 macOS 上，即使没有`-S`，上一个命令也可以工作；在 Linux 上通常不行。

##### 14.2.2.2 哈希标记陷阱：在 Windows 上创建哈希标记

如果我们在 Windows 上使用文本编辑器创建一个 ESM 模块，该模块应该在 Unix 或 Windows 上作为脚本运行，我们必须添加一个哈希标记。如果我们这样做，第一行将以 Windows 行终止符`\r\n`结束：

```js
#!/usr/bin/env node\r\n
```

在 Unix 上运行带有这样一个哈希标记的文件会产生以下错误：

```js
env: node\r: No such file or directory
```

也就是说，`env`认为可执行文件的名称是`node\r`。有两种方法可以解决这个问题。

首先，一些编辑器会自动检查文件中已经使用的行终止符，并继续使用它们。例如，Visual Studio Code 在右下角的状态栏中显示当前的行终止符（它称之为“行尾序列”）：

+   `LF`（换行）用于 Unix 行终止符`\n`

+   `CRLF`（回车换行）用于 Windows 行终止符`\r\n`

我们可以通过点击状态信息来切换选择行终止符。

其次，我们可以创建一个最小的文件`my-script.mjs`，其中只有 Unix 行终止符，我们在 Windows 上从不编辑它：

```js
#!/usr/bin/env node
import './main.mjs';
```

#### 14.2.3 在 Unix 上使文件可执行

为了成为一个 shell 脚本，`hello.mjs`还必须是可执行的（文件的权限），除了具有哈希标记：

```js
chmod u+x hello.mjs
```

请注意，我们使文件对于创建它的用户（`u`）是可执行的（`x`），而不是对于所有人。

#### 14.2.4 直接运行`hello.mjs`

`hello.mjs`现在是可执行的，看起来像这样：

```js
#!/usr/bin/env node

import * as os from 'node:os';

const {username} = os.userInfo();
console.log(`Hello ${username}!`);
```

因此，我们可以这样运行它：

```js
./hello.mjs
```

遗憾的是，没有办法告诉`node`将任意扩展名的文件解释为 ESM 模块。这就是为什么我们必须使用扩展名`.mjs`。解决方法是可能的，但复杂，我们稍后会看到。

### 14.3 创建一个带有 shell 脚本的 npm 包

在本节中，我们将使用 shell 脚本创建一个 npm 包。然后我们将研究如何安装这样一个包，以便它的脚本可以在您系统的命令行上使用（Unix 或 Windows）。

完成的包可以在这里找到：

+   在 GitHub 上为[`rauschma/demo-shell-scripts`](https://github.com/rauschma/demo-shell-scripts)

+   在 npm 上为[`@rauschma/demo-shell-scripts`](https://www.npmjs.com/package/@rauschma/demo-shell-scripts)

#### 14.3.1 设置包的目录

这些命令在 Unix 和 Windows 上都适用：

```js
mkdir demo-shell-scripts
cd demo-shell-scripts
npm init --yes
```

现在有以下文件：

```js
demo-shell-scripts/
  package.json
```

##### 14.3.1.1 未发布包的`package.json`

一个选项是创建一个包并不将其发布到 npm 注册表。我们仍然可以在我们的系统上安装这样一个包（如后面所述）。在这种情况下，我们的`package.json`如下所示：

```js
{
 "private": true,
 "license": "UNLICENSED"
}
```

解释：

+   将包设为私有意味着不需要名称或版本，并且不能意外发布。

+   `"UNLICENSED"`拒绝他人以任何条件使用该包。

##### 14.3.1.2 发布包的`package.json`

如果我们想将我们的包发布到 npm 注册表，我们的`package.json`如下所示：

```js
{
 "name": "@rauschma/demo-shell-scripts",
 "version": "1.0.0",
 "license": "MIT"
}
```

对于您自己的包，您需要用适合您的包名替换`"name"`的值：

+   或者一个全局唯一的名称。这样的名称应该只用于重要的包，因为我们不希望阻止其他人使用该名称。

+   或者*作用域名称*：要发布一个包，你需要一个 npm 账户（如何获得一个账户将在后面解释）。你的账户名可以作为包名的*作用域*。例如，如果你的账户名是`jane`，你可以使用以下包名：

    ```js
    "name": "@jane/demo-shell-scripts"
    ```

#### 14.3.2 添加依赖项

接下来，我们安装一个我们想在其中一个脚本中使用的依赖项 - 包`lodash-es`（[Lodash](https://lodash.com)的 ESM 版本）：

```js
npm install lodash-es
```

这个命令：

+   创建目录`node_modules`。

+   将包`lodash-es`安装到其中。

+   在`package.json`中添加以下属性：

    ```js
    "dependencies": {
     "lodash-es": "⁴.17.21"
    }
    ```

+   创建文件`package-lock.json`。

如果我们只在开发过程中使用一个包，我们可以将其添加到`"devDependencies"`而不是`"dependencies"`，npm 只有在我们在包的目录中运行`npm install`时才会安装它，而不是如果我们将它安装为一个依赖项。单元测试库是一个典型的开发依赖。

这是我们可以安装开发依赖的两种方式：

+   通过`npm install some-package`。

+   我们可以使用`npm install some-package --save-dev`，然后手动将`some-package`的条目从`"dependencies"`移动到`"devDependencies"`。

第二种方法意味着我们可以很容易地推迟决定一个包是一个依赖还是一个开发依赖。

#### 14.3.3 向包添加内容

让我们添加一个 readme 文件和两个 shell 脚本`homedir.mjs`和`versions.mjs`：

```js
demo-shell-scripts/
  package.json
  package-lock.json
  README.md
  src/
    homedir.mjs
    versions.mjs
```

我们必须告诉 npm 关于这两个 shell 脚本，这样它才能为我们安装它们。这就是`package.json`中的`"bin"`属性的作用：

```js
"bin": {
 "homedir": "./src/homedir.mjs",
 "versions": "./src/versions.mjs"
}
```

如果我们安装这个包，两个名为`homedir`和`versions`的 shell 脚本将变得可用。

你可能更喜欢使用`.js`作为 shell 脚本的文件扩展名。然后，你需要在`package.json`中添加以下两个属性，而不是之前的属性：

```js
"type": "module",
"bin": {
 "homedir": "./src/homedir.js",
 "versions": "./src/versions.js"
}
```

第一个属性告诉 Node.js 应该将`.js`文件解释为 ESM 模块（而不是 CommonJS 模块 - 这是默认值）。

`homedir.mjs`的样子如下：

```js
#!/usr/bin/env node
import {homedir} from 'node:os';

console.log('Homedir: ' + homedir());
```

这个模块以前面提到的 hashbang 开始，这是在 Unix 上使用它时所必需的。它从内置模块`node:os`中导入函数`homedir()`，调用它并将结果记录到控制台（即标准输出）。

请注意，`homedir.mjs`不需要可执行；npm 在安装时确保`"bin"`脚本的可执行性（我们很快就会看到如何做到这一点）。

`versions.mjs`的内容如下：

```js
#!/usr/bin/env node

import {pick} from 'lodash-es';

console.log(
 pick(process.versions, ['node', 'v8', 'unicode'])
);
```

我们从 Lodash 中导入`pick()`函数，并用它来显示`process.versions`对象的三个属性。

#### 14.3.4 在不安装的情况下运行 shell 脚本

我们可以这样运行，例如，`homedir.mjs`：

```js
cd demo-shell-scripts/
node src/homedir.mjs
```

### 14.4 npm 如何安装 shell 脚本

#### 14.4.1 在 Unix 上安装

例如，`homedir.mjs`这样的脚本在 Unix 上不需要可执行，因为 npm 通过可执行符号链接来安装它：

+   如果我们全局安装包，链接将被添加到`$PATH`中列出的目录中。

+   如果我们将包作为依赖项本地安装，链接将被添加到`node_modules/.bin/`中

#### 14.4.2 在 Windows 上安装

要在 Windows 上安装`homedir.mjs`，npm 会创建三个文件：

+   `homedir.bat`是一个使用`node`来执行`homedir.mjs`的命令 shell 脚本。

+   `homedir.ps1`对 PowerShell 也是一样的。

+   `homedir`对 Cygwin、MinGW 和 MSYS 也是一样的。

npm 会将这些文件添加到一个目录中：

+   如果我们全局安装包，文件将被添加到列在`%Path%`中的目录中。

+   如果我们将包作为依赖项本地安装，文件将被添加到`node_modules/.bin/`中

### 14.5 将示例包发布到 npm 注册表

让我们将包`@rauschma/demo-shell-scripts`（之前创建的）发布到 npm。在使用`npm publish`上传包之前，我们应该检查一切是否配置正确。

#### 14.5.1 发布了哪些文件？哪些文件被忽略了？

在发布时排除和包含文件时使用以下机制：

+   顶层文件`.gitignore`中列出的文件会被排除。

    +   我们可以用与`.gitignore`相同的格式覆盖`.npmignore`。

+   `package.json`属性`"files"`包含一个数组，其中包含要包括的文件的名称。这意味着我们可以选择列出要排除的文件（在`.npmignore`中）或要包括的文件。

+   一些文件和目录默认被排除在外 - 例如：

    +   `node_modules`

    +   `.*.swp`

    +   `._*`

    +   `.DS_Store`

    +   `.git`

    +   `.gitignore`

    +   `.npmignore`

    +   `.npmrc`

    +   `npm-debug.log`

    除了这些默认值，点文件（文件名以点开头的文件）也会被包括进来。

+   以下文件永远不会被排除：

    +   `package.json`

    +   `README.md`及其变体

    +   `CHANGELOG`及其变体

    +   `LICENSE`，`LICENCE`

npm 文档中有关于发布时包含和排除的[更多细节](https://docs.npmjs.com/cli/v8/using-npm/developers#keeping-files-out-of-your-package)。

#### 14.5.2 检查包是否正确配置

在上传包之前，我们可以检查几件事情。

##### 14.5.2.1 检查将要上传的文件

`npm install`的*dry run*会在不上传任何内容的情况下运行该命令：

```js
npm publish --dry-run
```

这会显示将要上传的文件以及有关包的几项统计数据。

我们也可以创建一个包的存档，就像它在 npm 注册表上存在一样：

```js
npm pack
```

此命令在当前目录中创建文件`rauschma-demo-shell-scripts-1.0.0.tgz`。

##### 14.5.2.2 全局安装软件包-而不上传

我们可以使用以下两个命令之一在全局安装我们的软件包而不将其发布到 npm 注册表：

```js
npm link
npm install . -g
```

要查看是否有效，我们可以打开一个新的 shell 并检查这两个命令是否可用。我们还可以列出所有全局安装的软件包：

```js
npm ls -g
```

##### 14.5.2.3 本地安装软件包（作为依赖项）-而不上传

要将我们的包安装为依赖项，我们必须执行以下命令（当我们在目录`demo-shell-scripts`中时）：

```js
cd ..
mkdir sibling-directory
cd sibling-directory
npm init --yes
npm install ../demo-shell-scripts
```

现在我们可以运行，例如，`homedir`，使用以下两个命令之一：

```js
npx homedir
./node_modules/.bin/homedir
```

#### 14.5.3 `npm publish`：将软件包上传到 npm 注册表

在我们上传软件包之前，我们需要创建一个 npm 用户帐户。 npm 文档[描述了如何做到这一点](https://docs.npmjs.com/creating-a-new-npm-user-account)。

然后我们最终可以发布我们的软件包：

```js
npm publish --access public
```

我们必须指定公共访问权限，因为默认值是：

+   对于未经范围限定的软件包，使用`public`

+   对于受范围限制的软件包使用`restricted`。此设置使软件包[*private*](https://docs.npmjs.com/about-private-packages)-这是一个付费的 npm 功能，主要由公司使用，并且与`package.json`中的`"private":true`不同。引用 npm：“使用 npm 私有软件包，您可以使用 npm 注册表来托管仅对您和选择的协作者可见的代码，允许您在项目中管理和使用私有代码以及公共代码。”

选项`--access`只在第一次发布时有效。之后，我们可以省略它，并且需要使用[`npm access`](https://docs.npmjs.com/cli/v8/commands/npm-access)来更改访问级别。

我们可以通过[`publishConfig.access`](https://docs.npmjs.com/cli/v8/using-npm/config#access)在`package.json`中更改初始`npm publish`的默认值：

```js
"publishConfig": {
 "access": "public"
}
```

##### 14.5.3.1 每次上传都需要一个新版本

一旦我们使用特定版本上传了软件包，我们就不能再使用该版本，我们必须增加版本的三个组件中的任何一个：

```js
major.minor.patch
```

+   如果我们进行了重大更改，则增加`major`。

+   如果我们进行了向后兼容的更改，则增加`minor`。

+   如果我们进行了不会真正改变 API 的小修复，则增加`patch`。

#### 14.5.4 每次发布前自动执行任务

可能有一些步骤我们想要在上传软件包之前每次执行-例如：

+   运行单元测试

+   将 TypeScript 代码编译为 JavaScript 代码

这可以通过`package.json`属性`“scripts”`自动完成。该属性可以如下所示：

```js
"scripts": {
 "build": "tsc",
 "test": "mocha --ui qunit",
 "dry": "npm publish --dry-run",
 "prepublishOnly": "npm run test && npm run build"
}
```

`mocha`是一个单元测试库。`tsc`是 TypeScript 编译器。

在`npm publish`之前运行以下软件包脚本：

+   `"prepare"`被运行：

    +   在`npm pack`之前

    +   在`npm publish`之前

    +   在本地`npm install`没有参数的情况下

+   `"prepublishOnly"`仅在`npm publish`之前运行。

有关此主题的更多信息，请参见§15“通过 npm 软件包脚本运行跨平台任务”。

### 14.6 在 Unix 上使用任意扩展名的独立 Node.js shell 脚本

#### 14.6.1 Unix：通过自定义可执行文件使用任意文件名扩展名

Node.js 二进制文件`node`使用文件扩展名来检测文件是哪种类型的模块。目前没有命令行选项来覆盖它。默认值是 CommonJS，这不是我们想要的。

但是，我们可以创建我们自己的可执行文件来运行 Node.js，并将其命名为`node-esm`，然后我们可以将我们以前的独立脚本`hello.mjs`重命名为`hello`（没有任何扩展名），如果我们将第一行更改为：

```js
#!/usr/bin/env node-esm
```

以前，`env`的参数是`node`。

这是 Andrea Giammarchi 提出的[node-esm 的实现](https://gist.github.com/WebReflection/8840ec29d296f2fa98d8be0102f08590)：

```js
#!/usr/bin/env sh
input_file=$1
shift
exec node --input-type=module - $@ < $input_file
```

此可执行文件通过标准输入将脚本内容发送到`node`。命令行选项`--input-type=module`告诉 Node.js 它接收的文本是一个 ESM 模块。

我们还使用以下 Unix shell 功能：

+   `$1`包含传递给`node-esm`的第一个参数-脚本的路径。

+   我们通过`shift`删除参数`$0`（`node-esm`的路径）并将剩余的参数传递给`node`。

+   `exec`用`node`运行替换当前进程。这确保脚本以与`node`相同的代码退出。

+   连字符（`-`）将 Node 的参数与脚本的参数分开。

在使用`node-esm`之前，我们必须确保它是可执行的，并且可以通过`$PATH`找到。如何做到这一点将在后面解释。

#### 14.6.2 Unix：通过 shell prolog 任意文件扩展名

我们已经看到，我们无法为文件指定模块类型，只能为标准输入指定。因此，我们可以编写一个 Unix shell 脚本`hello`，使用 Node.js 将自身作为 ESM 模块运行（基于[sambal.org 的工作](https://sambal.org/2014/02/passing-options-node-shebang-line/)）：

```js
#!/bin/sh
':' // ; cat "$0" | node --input-type=module - $@ ; exit $?

import * as os from 'node:os';

const {username} = os.userInfo();
console.log(`Hello ${username}!`);
```

我们在这里使用的大多数 shell 功能在本章的开头都有描述。`$?`包含上次执行的 shell 命令的退出代码。这使`hello`能够以与`node`相同的代码退出。

此脚本使用的关键技巧是第二行既是 Unix shell 脚本代码又是 JavaScript 代码：

+   作为 shell 脚本代码，它运行[引用命令`':'`](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#Bourne-Shell-Builtins)，除了扩展其参数和执行重定向外，什么也不做。它的唯一参数是路径`//`。然后将当前文件的内容传递给`node`二进制文件。

+   作为 JavaScript 代码，它是字符串`':'`（被解释为表达式语句并且什么也不做），然后是一个注释。

将 shell 代码从 JavaScript 中隐藏的另一个好处是，当处理和显示语法时，JavaScript 编辑器不会感到困惑。

### 14.7 在 Windows 上独立的 Node.js shell 脚本

#### 14.7.1 Windows：配置文件扩展名`.mjs`

在 Windows 上创建独立的 Node.js shell 脚本的一个选项是使用文件扩展名`.mjs`并配置文件以便通过`node`运行。遗憾的是，这仅适用于命令 shell，而不适用于 PowerShell。

另一个缺点是我们无法以这种方式传递参数给脚本：

```js
>more args.mjs
console.log(process.argv);

>.\args.mjs one two
[
  'C:\\Program Files\\nodejs\\node.exe',
  'C:\\Users\\jane\\args.mjs'
]

>node args.mjs one two
[
  'C:\\Program Files\\nodejs\\node.exe',
  'C:\\Users\\jane\\args.mjs',
  'one',
  'two'
]
```

我们如何配置 Windows，使命令 shell 直接运行诸如`args.mjs`之类的文件？

*文件关联*指定在 shell 中输入其名称时打开文件的应用程序。如果我们将文件扩展名`.mjs`与 Node.js 二进制文件关联，我们可以在 shell 中运行 ESM 模块。其中一种方法是通过设置应用程序，如 Tim Fisher 在[“如何更改 Windows 中的文件关联”](https://www.lifewire.com/how-to-change-file-associations-in-windows-2624477)中所解释的那样。

如果我们还将`.MJS`添加到变量`%PATHEXT%`中，甚至在引用 ESM 模块时可以省略文件扩展名。此环境变量可以通过设置应用程序永久更改-搜索“variables”。

#### 14.7.2 Windows 命令 shell：通过 shell prolog 运行 Node.js 脚本

在 Windows 上，我们面临的挑战是没有像 hashbangs 这样的机制。因此，我们必须使用类似于我们在 Unix 上用于无扩展名文件的解决方法：创建一个通过 Node.js 在自身内部运行 JavaScript 代码的脚本。

命令 shell 脚本的文件扩展名是`.bat`。我们可以通过`script.bat`或`script`运行名为`script.bat`的脚本。

如果我们将其转换为命令 shell 脚本`hello.bat`，则`hello.mjs`看起来是这样的：

```js
:: /*
@echo off
more +5 %~f0 | node --input-type=module - %*
exit /b %errorlevel%
*/

import * as os from 'node:os';
const {username} = os.userInfo();
console.log(`Hello ${username}!`);
```

将此代码作为文件通过`node`运行需要两个不存在的功能：

+   使用命令行选项来覆盖默认情况下将无扩展名的文件解释为 ESM 模块。

+   跳过文件开头的行。

因此，我们别无选择，只能将文件的内容传递给`node`。我们还使用以下命令 shell 功能：

+   `%~f0`包含当前脚本的完整路径，包括其文件扩展名。相比之下，`%0`包含用于调用脚本的命令。因此，前者的 shell 变量使我们能够通过`hello`或`hello.bat`调用脚本。

+   `%*`包含命令的参数-我们将其传递给`node`。

+   `%errorlevel%`包含上次执行的命令的退出代码。我们使用该值以与`node`指定的相同代码退出。

#### 14.7.3 Windows PowerShell：通过 shell prolog 运行 Node.js 脚本

我们可以使用与上一节中使用的类似的技巧，将`hello.mjs`转换为 PowerShell 脚本`hello.ps1`，如下所示：

```js
Get-Content $PSCommandPath | Select-Object -Skip 3 | node --input-type=module - $args
exit $LastExitCode
<#
import * as os from 'node:os';
const {username} = os.userInfo();
console.log(`Hello ${username}!`);
// #>
```

我们可以通过以下方式运行此脚本：

```js
.\hello.ps1
.\hello
```

但是，在我们这样做之前，我们需要设置一个允许我们运行 PowerShell 脚本的执行策略（[有关执行策略的更多信息](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies)）：

+   Windows 客户端上的默认策略是“受限”，不允许我们运行任何脚本。

+   策略`RemoteSigned`允许我们运行未签名的本地脚本。下载的脚本必须经过签名。这是 Windows 服务器上的默认设置。

以下命令让我们运行本地脚本：

```js
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### 14.8 为 Linux、macOS 和 Windows 创建本机二进制文件

[npm 包`pkg`](https://github.com/vercel/pkg)将 Node.js 包转换为本机二进制文件，即使在未安装 Node.js 的系统上也可以运行。它支持以下平台：Linux、macOS 和 Windows。

### 14.9 Shell 路径：确保 shell 找到脚本

在大多数 shell 中，我们可以输入文件名而不直接引用文件，它们会在几个目录中搜索具有该名称的文件并运行它。这些目录通常在一个特殊的 shell 变量中列出：

+   在大多数 Unix shell 中，我们通过`$PATH`访问它。

+   在 Windows 命令 shell 中，我们通过`%Path%`访问它。

+   在 PowerShell 中，我们通过`$Env:PATH`访问它。

我们需要 PATH 变量有两个目的：

+   如果我们想要安装我们自定义的 Node.js 可执行文件`node-esm`。

+   如果我们想要运行一个独立的 shell 脚本而不直接引用其文件。

#### 14.9.1 Unix：`$PATH`

大多数 Unix shell 都有一个名为`$PATH`的变量，列出了当我们输入命令时 shell 查找可执行文件的所有路径。它的值可能如下所示：

```js
$ echo $PATH
/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin
```

以下命令适用于大多数 shell（[来源](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path/26059#26059)），并且在我们离开当前 shell 之前更改`$PATH`：

```js
export PATH="$PATH:$HOME/bin"
```

如果两个 shell 变量中有一个包含空格，则需要引号。

##### 14.9.1.1 永久更改`$PATH`

在 Unix 上，`$PATH`的配置取决于 shell。您可以通过以下方式找出自己正在运行的 shell：

```js
echo $0
```

MacOS 使用 Zsh，永久配置`$PATH`的最佳位置是启动脚本`$HOME/.zprofile`- [像这样](https://stackoverflow.com/questions/11530090/adding-a-new-entry-to-the-path-variable-in-zsh)：

```js
path+=('/Library/TeX/texbin')
export PATH
```

#### 14.9.2 更改 Windows 上的 PATH 变量（命令 shell，PowerShell）

在 Windows 上，可以通过“设置”应用程序永久配置命令 shell 和 PowerShell 的默认环境变量-搜索“variables”。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/14)
