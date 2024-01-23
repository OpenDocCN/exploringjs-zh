# 13 安装 npm 包并运行 bin 脚本

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_installing-packages.html`](https://exploringjs.com/nodejs-shell-scripting/ch_installing-packages.html)

* * *

+   13.1 全局安装 npm 注册表包

    +   13.1.1 哪些包是全局安装的？ `npm ls -g`（ch_installing-packages.html#which-packages-are-installed-globally-npm-ls--g）

    +   13.1.2 全局安装的包在哪里？ `npm root -g`（ch_installing-packages.html#where-are-packages-installed-globally-npm-root--g）

    +   13.1.3 全局安装的 shell 脚本在哪里？ `npm bin -g`（ch_installing-packages.html#where-are-shell-scripts-installed-globally-npm-bin--g）

    +   13.1.4 全局安装的包在哪里？npm 安装前缀

    +   13.1.5 更改全局安装包的位置

+   13.2 在本地安装 npm 注册表包

    +   13.2.1 在本地安装 bin 脚本

+   13.3 安装未发布的包

    +   13.3.1 `npm link`: 全局安装未发布的包

    +   13.3.2 `npm link`: 在本地安装全局链接的包

    +   13.3.3 `npm link`: 撤消链接

    +   13.3.4 通过本地路径安装未发布的包（ch_installing-packages.html#installing-unpublished-packages-via-local-paths）

    +   13.3.5 安装未发布包的其他方法

+   13.4 `npx`: 在不安装的情况下运行 npm 包中的 bin 脚本

    +   13.4.1 npx 缓存

* * *

`package.json` 属性 `"bin"` 允许 npm 包指定它提供的 shell 脚本（有关更多信息，请参见§14“创建跨平台 shell 脚本”）。如果我们安装了这样的包，Node.js 会确保我们可以从命令行访问这些 shell 脚本（称为*bin 脚本*）。在本章中，我们探讨了两种安装带有 bin 脚本的包的方法：

+   在本地安装带有 bin 脚本的包意味着将其安装为包内的依赖项。这些脚本只能在该包内访问。

+   全局安装带有 bin 脚本的包意味着将其安装在“全局位置”，以便脚本可以在任何地方访问-无论是当前用户还是系统的所有用户（取决于 npm 的设置方式）。

我们探讨了所有这些的含义以及我们如何在安装后运行 bin 脚本。

### 13.1 全局安装 npm 注册表包

[包`cowsay`](https://github.com/piuccio/cowsay) 具有以下 `package.json` 属性：

```js
"bin": {
 "cowsay": "./cli.js",
 "cowthink": "./cli.js"
},
```

要全局安装此包，我们使用 `npm install -g`：

```js
npm install -g cowsay
```

注意：在 Unix 上，我们可能需要使用 `sudo`（我们很快将学会如何避免这样做）：

```js
sudo npm install -g cowsay
```

之后，我们可以在命令行中使用 `cowsay` 和 `cowthink` 命令。

请注意，只有 bin 脚本在全局可用。当 Node.js 在`node_modules`目录中查找裸模块规范时，包会被忽略。

#### 13.1.1 哪些包是全局安装的？ `npm ls -g`

我们可以检查全局安装的包以及它们的位置：

```js
% npm ls -g
/usr/local/lib
├── corepack@0.12.1
├── cowsay@1.5.0
└── npm@8.15.0
```

在 Windows 上，安装路径是 `%AppData%\npm`，例如：

```js
>echo %AppData%\npm
C:\Users\jane\AppData\Roaming\npm
```

#### 13.1.2 全局安装的包在哪里？ `npm root -g`

macOS 上的结果：

```js
% npm root -g
/usr/local/lib/node_modules
```

Windows 上的结果：

```js
>npm root -g
C:\Users\jane\AppData\Roaming\npm\node_modules
```

#### 13.1.3 全局安装的 shell 脚本在哪里？ `npm bin -g`

`npm bin -g`告诉我们 npm 全局安装 shell 脚本的位置。它还确保该目录在 shell PATH 中可用。

macOS 上的结果：

```js
% npm bin -g
/usr/local/bin

% which cowsay
/usr/local/bin/cowsay
```

在 Windows 命令 shell 上的结果：

```js
>npm bin -g
C:\Users\jane\AppData\Roaming\npm

>where cowsay
C:\Users\jane\AppData\Roaming\npm\cowsay
C:\Users\jane\AppData\Roaming\npm\cowsay.cmd
```

没有文件名扩展名的可执行文件`cowsay`是针对基于 Unix 的 Windows 环境（如 Cygwin、MinGW 和 MSYS）的。

Windows PowerShell 返回`gcm cowsay`的路径：

```js
C:\Users\jane\AppData\Roaming\npm\cowsay.ps1
```

#### 13.1.4 全局安装的包在哪里？npm 安装前缀

npm 的*安装前缀*决定了全局安装包和 bin 脚本的安装位置。

这是 macOS 上的安装前缀：

```js
% npm config get prefix
/usr/local
```

因此：

+   包安装在`/usr/local/lib/node_modules`中

+   Bin 脚本安装在`/usr/local/bin`中

这是 Windows 上的安装前缀：

```js
>npm config get prefix
C:\Users\jane\AppData\Roaming\npm
```

因此：

+   包安装在`C:\Users\jane\AppData\Roaming\npm\node_modules`中

+   Bin 脚本安装在`C:\Users\jane\AppData\Roaming\npm`中

#### 13.1.5 改变全局安装包位置

在这一部分，我们将研究两种改变全局安装包位置的方法：

+   更改 npm 安装前缀

+   使用 Node.js 版本管理器

##### 13.1.5.1 改变 npm 安装前缀

改变全局安装包位置的一种方法是改变 npm 的安装前缀。

Unix：

```js
mkdir ~/npm-global
npm config set prefix '~/npm-global'
```

Windows 命令 shell：

```js
mkdir "%UserProfile%\npm-global"
npm config set prefix "%UserProfile%\npm-global"
```

Windows PowerShell：

```js
mkdir "$env:UserProfile\npm-global"
npm config set prefix "$env:UserProfile\npm-global"
```

配置数据保存在主目录中的`.npmrc`文件中。

从现在开始，全局安装将被添加到我们刚刚指定的目录中。

之后，我们仍然需要将`npm bin -g`目录添加到我们的 shell PATH 中，以便我们的 shell 可以找到我们全局安装的 bin 脚本。

**更改 npm 前缀的一个缺点：**如果我们告诉 npm 升级自己，它现在也会安装到新位置。

##### 13.1.5.2 使用 Node.js 版本管理器

Node.js 版本管理器可以让我们同时安装多个 Node.js 版本并在它们之间切换。流行的版本管理器包括：

+   Unix：[nvm](https://github.com/nvm-sh/nvm)

+   跨平台：[Volta](https://volta.sh)

### 13.2 安装 npm 注册包到本地

要*本地*安装 npm 注册包（如`cowsay`），我们需要执行以下操作：

```js
cd my-package/
npm install cowsay
```

这将向`package.json`添加以下数据：

```js
"dependencies": {
 "cowsay": "¹.5.0",
 ···
}
```

此外，该包被下载到以下目录：

```js
my-package/node_modules/cowsay/
```

在 Unix 上，npm 为 bin 脚本添加了这些符号链接：

```js
my-package/node_modules/.bin/cowsay -> ../cowsay/cli.js
my-package/node_modules/.bin/cowthink -> ../cowsay/cli.js
```

在 Windows 上，npm 将这些文件添加到`my-package\node_modules\.bin\`中：

```js
cowsay
cowsay.cmd
cowsay.ps1
cowthink
cowthink.cmd
cowthink.ps1
```

没有扩展名的文件是针对基于 Unix 的 Windows 环境（如 Cygwin、MinGW 和 MSYS）的脚本。

`npm bin`告诉我们本地安装的 bin 脚本的位置 - 例如：

```js
% npm bin
/Users/john/my-package/node_modules/.bin
```

注意：本地安装的包始终安装在`package.json`文件旁边的`node_modules`目录中。如果当前目录中不存在`package.json`，npm 会在祖先目录中搜索并在那里安装包。要检查 npm 在本地安装包的位置，我们可以使用`npm root`命令 - 例如（Unix）：

```js
% cd $HOME
% npm root
/Users/john/node_modules
```

John 的主目录中没有`package.json`，但 npm 无法在祖先目录中安装任何内容，这就是为什么`npm root`显示这个目录。在当前位置本地安装包将导致创建`package.json`并像往常一样进行安装。

#### 13.2.1 运行本地安装的 bin 脚本

（本小节中的所有命令都在`my-package`目录中执行。）

##### 13.2.1.1 直接运行 bin 脚本

我们可以从 shell 中如下运行`cowsay`：

```js
./node_modules/.bin/cowsay Hello
```

在 Unix 上，我们可以设置一个辅助程序：

```js
alias npm-exec='PATH=$(npm bin):$PATH'
```

然后以下命令有效：

```js
npm-exec cowsay Hello
```

##### 13.2.1.2 通过包脚本运行 bin 脚本

我们还可以在`package.json`中添加一个包脚本：

```js
{
 ···
 "scripts": {
 "cowsay": "cowsay"
 },
 ···
}
```

现在我们可以在 shell 中执行这个命令：

```js
npm run cowsay Hello
```

这是因为 npm 在 Unix 上临时将以下条目添加到`$PATH`中：

```js
/Users/john/my-package/node_modules/.bin
/Users/john/node_modules/.bin
/Users/node_modules/.bin
/node_modules/.bin
```

在 Windows 上，类似的条目被添加到`%Path%`或`$env:Path`中：

```js
C:\Users\jane\my-package\node_modules\.bin
C:\Users\jane\node_modules\.bin
C:\Users\node_modules\.bin
C:\node_modules\.bin
```

以下命令列出了包脚本运行时存在的环境变量及其值：

```js
npm run env
```

##### 13.2.1.3 通过 npx 运行 bin 脚本

在一个包内，可以使用 npx 来访问 bin 脚本：

```js
npx cowsay Hello
npx cowthink Hello
```

稍后再详细介绍 npx。

### 13.3 安装未发布的包

有时，我们有一个包，要么我们还没有发布，要么永远不会发布，并且想要安装它。

#### 13.3.1 `npm link`：全局安装未发布的包

假设我们有一个未发布的包，其名称是 `@my-scope/unpublished-package`，存储在目录 `/tmp/unpublished-package/` 中。我们可以按如下方式全局提供它：

```js
cd /tmp/unpublished-package/
npm link
```

如果我们这样做：

+   npm 将一个符号链接添加到全局的 `node_modules`（由 `npm root -g` 返回）- 例如：

    ```js
    /usr/local/lib/node_modules/@my-scope/unpublished-package
    -> ../../../../../tmp/unpublished-package
    ```

+   在 Unix 上，npm 还会从全局 bin 目录（由 `npm bin -g` 返回）到每个 bin 脚本添加一个符号链接。该链接不是直接的，而是通过全局 `node_modules` 目录：

    ```js
    /usr/local/bin/my-command
    -> ../lib/node_modules/@my-scope/unpublished-package/src/my-command.js
    ```

+   在 Windows 上，它添加了通常的 3 个脚本（通过相对路径引用全局 `node_modules` 中的链接包）：

    ```js
    C:\Users\jane\AppData\Roaming\npm\my-command
    C:\Users\jane\AppData\Roaming\npm\my-command.cmd
    C:\Users\jane\AppData\Roaming\npm\my-command.ps1
    ```

由于链接包的引用方式，其中的任何更改都会立即生效。当它发生变化时，无需重新链接它。

要检查全局安装是否成功，我们可以使用 `npm ls -g` 列出所有全局安装的包。

#### 13.3.2 `npm link`：在本地安装全局链接的包

在我们全局安装了未发布的包之后（参见前一小节），我们可以选择在我们的一个包中（可以是已发布的或未发布的）中将其安装为本地包：

```js
cd /tmp/other-package/
npm link @my-scope/unpublished-package
```

这创建了以下链接：

```js
/tmp/other-package/node_modules/@my-scope/unpublished-package
-> ../../../unpublished-package
```

默认情况下，未发布的包不会被添加为 `package.json` 的依赖项。其背后的原因是 `npm link` 经常用于临时使用注册表包的未发布版本- 这些不应该出现在依赖项中。

#### 13.3.3 `npm link`：取消链接

取消本地链接：

```js
cd /tmp/other-package/
npm uninstall @my-scope/unpublished-package
```

取消全局链接：

```js
cd /tmp/unpublished-package/
npm uninstall -g
```

#### 13.3.4 通过本地路径安装未发布的包

另一种在本地安装未发布的包的方法是使用 `npm install` 并通过本地路径引用它（而不是通过包名）：

```js
cd /tmp/other-package/
npm install ../unpublished-package
```

这有两个效果。

首先，创建以下符号链接：

```js
/tmp/other-package/node_modules/@my-scope/unpublished-package
-> ../../../unpublished-package
```

其次，将依赖项添加到 `package.json` 中：

```js
"dependencies": {
 "@my-scope/unpublished-package": "file:../unpublished-package",
 ···
}
```

这种安装未发布的包的方法也适用于全局：

```js
cd /tmp/unpublished-package/
npm install -g .
```

#### 13.3.5 安装未发布的包的其他方法

+   [Yalc](https://github.com/wclr/yalc) 让我们将包发布到本地的“Yalc 仓库”（类似本地注册表）。从该仓库中，我们可以将包安装为依赖项，例如，一个名为 `my-package/` 的包。它们被复制到目录 `my-package/.yalc` 中，并且 `file:` 或 `link:` 依赖项被添加到 `package.json` 中。

+   [`relative-deps`](https://github.com/mweststrate/relative-deps) 支持 `package.json` 中的 `"relativeDependencies"`，如果存在的话，会覆盖正常的依赖关系。与 `npm link` 和本地路径安装相比：

    +   正常的依赖关系不需要更改。

    +   相对依赖项被安装为来自 npm 注册表的依赖项（而不是通过符号链接）。

    `relative-deps` 还有助于保持本地安装的相对依赖项及其原始依赖项同步。

+   [`npx link`](https://hirok.io/posts/avoid-npm-link) 是 `npm link` 的一个更安全的版本，它不需要全局安装，还有其他好处。

### 13.4 `npx`：在不安装它们的情况下运行 npm 包中的 bin 脚本

[npx](https://docs.npmjs.com/cli/v8/commands/npx) 是一个与 npm 捆绑在一起的用于运行 bin 脚本的 shell 命令。

它最常见的用法是：

```js
npx <package-name> arg1 arg2 ...
```

这个命令将名称为 `package-name` 的包安装到 npx 缓存中，并运行与包同名的 bin 脚本- 例如：

```js
npx cowsay Hello
```

这意味着我们可以在不先安装它们的情况下运行 bin 脚本。npx 最适用于一次性调用 bin 脚本- 例如，许多框架提供用于设置新项目的 bin 脚本，这些通常通过 npx 运行。

npx 第一次使用包后，它将在其缓存中可用，并且后续调用速度更快。但是，我们无法确定包在缓存中停留的时间有多长。因此，npx 不能替代全局或本地安装 bin 脚本。

如果一个包带有与其包名称不同的 bin 脚本，我们可以像这样访问它们：

```js
npx --package=<package-name> <bin-script> arg1 arg2 ...
```

例如：

```js
npx --package=cowsay cowthink Hello
```

#### 13.4.1 npx 缓存

npx 的缓存位于哪里？

在 Unix 上，我们可以通过以下命令找到：

```js
npx --package=cowsay node -p \
  "process.env.PATH.split(':').find(p => p.includes('_npx'))"
```

返回类似于这样的路径：

```js
/Users/john/.npm/_npx/8f497369b2d6166e/node_modules/.bin
```

在 Windows 上，我们可以使用（一行分成两行）：

```js
npx --package=cowsay node -p
  "process.env.Path.split(';').find(p => p.includes('_npx'))"
```

返回类似于这样的路径（单个路径分成两行）：

```js
C:\Users\jane\AppData\Local\npm-cache\_npx\
  8f497369b2d6166e\node_modules\.bin
```

请注意，npx 的缓存与 npm 用于安装模块的缓存不同：

+   Unix：

    +   npm 缓存：`$HOME/.npm/_cacache/`

    +   npx 缓存：`$HOME/.npm/_npx/`

+   Windows（PowerShell）：

    +   npm 缓存：`$env:UserProfile\AppData\Local\npm-cache\_npx\`

    +   npx 缓存：`$env:UserProfile\AppData\Local\npm-cache\_cacache\`

两个缓存的父目录可以通过以下方式确定：

```js
npm config get cache
```

有关 npm 缓存的更多信息，请参阅[npm 文档](https://docs.npmjs.com/cli/v8/commands/npm-cache)。

与 npx 缓存相比，npm 缓存中的数据永远不会被删除，只会被添加。我们可以在 Unix 上通过以下方式检查其大小：

```js
du -sh $(npm config get cache)/_cacache/
```

在 Windows PowerShell 上：

```js
DiskUsage /d:0 "$(npm config get cache)\_cacache"
```

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/13)
