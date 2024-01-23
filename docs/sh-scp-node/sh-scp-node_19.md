# 十五、[通过 npm 包脚本运行跨平台任务]

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_package-scripts.html`](https://exploringjs.com/nodejs-shell-scripting/ch_package-scripts.html)

* * *

+   15.1 npm 包脚本

    +   15.1.1 运行包脚本的缩写 npm 命令

    +   15.1.2 用于运行包脚本的 shell

    +   15.1.3 防止自动运行包脚本

    +   15.1.4 在 Unix 上为包脚本获取选项完成

    +   15.1.5 列出和组织包脚本

+   15.2 包脚本的种类

    +   15.2.1 预先和后置脚本

    +   15.2.2 生命周期脚本

+   15.3 运行包脚本的 shell 环境

    +   15.3.1 当前目录

    +   15.3.2 shell 路径

+   15.4 在包脚本中使用环境变量

    +   15.4.1 获取和设置环境变量

    +   15.4.2 通过`.env`文件设置环境变量

+   15.5 包脚本的参数

+   15.6 npm 日志级别（产生多少输出）

    +   15.6.1 日志级别和打印到终端的信息

    +   15.6.2 日志级别和写入 npm 日志的信息

    +   15.6.3 配置日志

    +   15.6.4 在`npm install`期间运行的生命周期脚本的输出

    +   15.6.5 观察 npm 日志的工作方式

+   15.7 跨平台 shell 脚本

    +   15.7.1 路径和引用

    +   15.7.2 链接命令

    +   15.7.3 包脚本的退出代码

    +   15.7.4 管道和重定向输入和输出

    +   15.7.5 适用于两个平台的命令

    +   15.7.6 运行 bin 脚本和包内模块

    +   15.7.7 `node --eval`和`node --print`

+   15.8 常见操作的辅助包

    +   15.8.1 从命令行运行包脚本

    +   15.8.2 并行或顺序运行多个脚本

    +   15.8.3 文件系统操作

    +   15.8.4 将文件或目录放入垃圾箱

    +   15.8.5 复制文件树

    +   15.8.6 监视文件

    +   15.8.7 其他功能

    +   15.8.8 HTTP 服务器

+   15.9 扩展包脚本的功能

    +   15.9.1 `per-env`: 根据`$NODE_ENV`在脚本之间切换

    +   15.9.2 定义特定操作系统的脚本

+   15.10 本章的来源

* * *

`package.json`有一个属性`"scripts"`，让我们定义*包脚本*，执行与包相关的任务，如编译构件或运行测试的小型 shell 脚本。本章解释了它们以及我们如何编写它们，使它们在 Windows 和 Unix（macOS，Linux 等）上都能工作。

### 15.1 npm 包脚本

*npm 包脚本*通过`package.json`的属性`"scripts"`来定义：

```js
{
 ···
 "scripts": {
 "tsc": "tsc",
 "tscwatch": "tsc --watch",
 "tscclean": "shx rm -rf ./dist/*"
 },
 ···
}
```

`"scripts"`的值是一个对象，其中每个属性定义了一个包脚本：

+   属性键定义了脚本的名称。

+   属性值定义了脚本运行时要执行的操作。

如果我们输入：

```js
npm run <script-name>
```

然后 npm 在 shell 中执行名称为`script-name`的脚本。例如，我们可以使用：

```js
npm run tscwatch
```

在 shell 中运行以下命令：

```js
tsc --watch
```

在本章中，我们偶尔会使用`npm`选项`-s`，它是`--silent`的缩写，并告诉`npm run`产生更少的输出：

```js
npm -s run <script-name>
```

此选项在日志部分中有更详细的介绍。

#### 15.1.1 运行包脚本的更短的 npm 命令

一些包脚本可以通过更短的 npm 命令运行：

| 命令 | 等效 |
| --- | --- |
| `npm test`, `npm t` | `npm run test` |
| `npm start` | `npm run start` |
| `npm stop` | `npm run stop` |
| `npm restart` | `npm run restart` |

+   `npm start`: 如果没有包脚本`"start"`，npm 会运行`node server.js`。

+   `npm restart`: 如果没有包脚本`"restart"`，npm 会运行`"prerestart"`，`"stop"`，`"start"`，`"postrestart"`。

    +   [更多关于`npm restart`的信息。](https://docs.npmjs.com/cli/v8/commands/npm-restart)

#### 15.1.2 用于运行包脚本的 shell 是哪个？

默认情况下，npm 通过`cmd.exe`在 Windows 上运行包脚本，在 Unix 上通过`/bin/sh`运行。我们可以通过[npm 配置设置`script-shell`](https://docs.npmjs.com/cli/v8/using-npm/config#script-shell)来更改。

然而，这样做很少是一个好主意：许多现有的跨平台脚本都是为`sh`和`cmd.exe`编写的，将停止工作。

#### 15.1.3 防止包脚本自动运行

一些脚本名称保留用于*生命周期脚本*，当我们执行特定的 npm 命令时，npm 会运行它们。

例如，当我们执行`npm install`（不带参数）时，npm 会运行脚本`"postinstall"`。生命周期脚本将在后面更详细地介绍。

[如果配置设置`ignore-scripts`](https://docs.npmjs.com/cli/v8/using-npm/config#ignore-scripts)为`true`，npm 永远不会自动运行脚本，只有在我们直接调用它们时才会运行。

#### 15.1.4 在 Unix 上为包脚本获取 tab 补全

在 Unix 上，npm 支持通过[`npm completion`](https://docs.npmjs.com/cli/v8/commands/npm-completion)为命令和包脚本名称进行 tab 补全。我们可以通过将以下行添加到我们的`.profile` / `.zprofile` / `.bash_profile` /等来安装它。

```js
. <(npm completion)
```

如果您需要非 Unix 平台的 tab 补全，请搜索“npm tab completion PowerShell”等。

#### 15.1.5 列出和组织包脚本

没有名称的`npm run`会列出可用的脚本。如果存在以下脚本：

```js
"scripts": {
 "tsc": "tsc",
 "tscwatch": "tsc --watch",
 "serve": "serve ./site/"
}
```

然后它们会像这样列出：

```js
% npm run
Scripts available via `npm run-script`:
  tsc
    tsc
  tscwatch
    tsc --watch
  serve
    serve ./site/
```

##### 15.1.5.1 添加分隔符

如果有很多包脚本，我们可以滥用脚本名称作为分隔符（脚本 `"help"` 将在下一小节中解释）：

```js
 "scripts": {
 "help": "scripts-help -w 40",
 "\n========== Building ==========": "",
 "tsc": "tsc",
 "tscwatch": "tsc --watch",
 "\n========== Serving ==========": "",
 "serve": "serve ./site/"
 },
```

现在脚本列出如下：

```js
% npm run
Scripts available via `npm run-script`:
  help
    scripts-help -w 40

========== Building ==========

  tsc
    tsc
  tscwatch
    tsc --watch

========== Serving ==========

  serve
    serve ./site/
```

请注意，在 Unix 和 Windows 上都可以使用换行符（`\n`）的技巧。

##### 15.1.5.2 打印帮助信息

包脚本 `"help"` 通过 [package `@rauschma/scripts-help`](https://github.com/rauschma/scripts-help) 的 bin 脚本 `scripts-help` 打印帮助信息。我们通过 `package.json` 属性 `"scripts-help"` 提供描述（`"tscwatch"` 的值被缩写以适应单行）：

```js
"scripts-help": {
 "tsc": "Compile the TypeScript to JavaScript.",
 "tscwatch": "Watch the TypeScript source code [...]",
 "serve": "Serve the generated website via a local server."
}
```

帮助信息如下所示：

```js
% npm -s run help
Package “demo”

╔══════╤══════════════════════════╗
║ help │ scripts-help -w 40       ║
╚══════╧══════════════════════════╝

Building

╔══════════╤══════════════════════════════════════════╗
║ tsc      │ Compile the TypeScript to JavaScript.    ║
╟──────────┼──────────────────────────────────────────╢
║ tscwatch │ Watch the TypeScript source code and     ║
║          │ compile it incrementally when and if     ║
║          │ there are changes.                       ║
╚══════════╧══════════════════════════════════════════╝

Serving

╔═══════╤══════════════════════════════════════════╗
║ serve │ Serve the generated website via a local  ║
║       │ server.                                  ║
╚═══════╧══════════════════════════════════════════╝
```

### 15.2 包脚本的种类

如果某些名称用于脚本，则在某些情况下会自动运行：

+   *Pre 脚本* 和 *post 脚本* 在脚本之前和之后运行。

+   *生命周期脚本* 在用户执行诸如 `npm install` 之类的操作时运行。

所有其他脚本都被称为 *直接运行脚本*。

#### 15.2.1 Pre 和 post 脚本

每当 npm 运行包脚本 `PS` 时，它会自动运行以下脚本 - 如果它们存在的话：

+   `prePS` 之前（*pre 脚本*）

+   `postPS` 之后（*post 脚本*）

以下脚本包含预先脚本 `prehello` 和后置脚本 `posthello`：

```js
"scripts": {
 "hello": "echo hello",
 "prehello": "echo BEFORE",
 "posthello": "echo AFTER"
},
```

这是我们运行 `hello` 时会发生的事情：

```js
% npm -s run hello
BEFORE
hello
AFTER
```

#### 15.2.2 生命周期脚本

npm 在执行 `npm publish` 等命令期间运行 *生命周期脚本*：

+   `npm publish`（上传包到 npm 注册表）

+   `npm pack`（为注册表包、包目录等创建归档）

+   `npm install`（无参数使用，用于安装从 npm 注册表以外的来源下载的包的依赖项）

如果任何生命周期脚本失败，整个命令将立即停止并显示错误。

生命周期脚本有哪些用例？

+   编译 TypeScript：如果一个包包含 TypeScript 代码，我们通常会在使用之前将其编译为 JavaScript 代码。虽然后者的代码通常不会被检入版本控制，但它必须上传到 npm 注册表，以便从 JavaScript 中使用该包。生命周期脚本让我们在 `npm publish` 上传包之前编译 TypeScript 代码。这确保了在 npm 注册表中，JavaScript 代码始终与我们的 TypeScript 代码同步。它还确保我们的 TypeScript 代码没有静态类型错误，因为当遇到这些错误时，编译（因此发布）会停止。

+   运行测试：我们还可以使用生命周期脚本在发布包之前运行测试。如果测试失败，包将不会被发布。

这些是最重要的生命周期脚本（有关所有生命周期脚本的详细信息，请参阅 [npm 文档](https://docs.npmjs.com/cli/v8/using-npm/scripts#life-cycle-scripts)）：

+   `"prepare"`:

    +   在创建包归档（`.tgz` 文件）之前运行：

        +   在 `npm publish` 期间

        +   在 `npm pack` 期间

    +   在从 git 或本地路径安装包时运行。

    +   在没有参数使用 `npm install` 或者全局安装包时运行。

+   `"prepack"` 在创建包归档（`.tgz` 文件）之前运行：

    +   在 `npm publish` 期间

    +   在 `npm pack` 期间

+   `"prepublishOnly"` 仅在 `npm publish` 期间运行。

+   `"install"` 在没有参数使用 `npm install` 或者全局安装包时运行。

    +   请注意，我们还可以创建一个预先脚本 `"preinstall"` 和/或一个后置脚本 `"postinstall"`。它们的名称使得在 npm 运行它们时更清晰。

以下表格总结了这些生命周期脚本何时运行：

|  | `prepublishOnly` | `prepack` | `prepare` | `install` |
| --- | --- | --- | --- | --- |
| `npm publish` | `✔` | `✔` | `✔` |  |
| `npm pack` |  | `✔` | `✔` |  |
| `npm install` |  |  | `✔` | `✔` |
| 全局安装 |  |  | `✔` | `✔` |
| 通过 git、路径安装 |  |  | `✔` |  |

**注意：** 自动执行事务总是有点棘手。我通常遵循以下规则：

+   我为自己自动化（例如通过 `prepublishOnly`）。

+   我不为其他人自动化（例如通过 `postinstall`）。

### 15.3 包脚本运行的 shell 环境

在本节中，我们偶尔会使用

```js
node -p <expr>
```

这个命令调用`expr`中的 JavaScript 代码，并将结果打印到终端 - 例如：

```js
% node -p "'hello everyone!'.toUpperCase()" 
HELLO EVERYONE!
```

#### 15.3.1 当前目录

当包脚本运行时，当前目录始终是包目录，与我们在其根目录的目录树中的位置无关。我们可以通过将以下脚本添加到`package.json`来确认：

```js
"cwd": "node -p \"process.cwd()\""
```

让我们在 Unix 上尝试`cwd`：

```js
% cd /Users/robin/new-package/src/util 
% npm -s run cwd
/Users/robin/new-package
```

以这种方式改变当前目录有助于编写包脚本，因为我们可以使用相对于包目录的路径。

#### 15.3.2 shell PATH

当模块`M`从以包`P`的名称开头的模块导入时，Node.js 会遍历`node_modules`目录，直到找到`P`的目录：

+   `M`的父目录中的第一个`node_modules`（如果存在）

+   `M`的父目录的父目录中的第二个`node_modules`（如果存在）

+   依此类推，直到达到文件系统的根目录。

也就是说，`M`继承了其祖先目录的`node_modules`目录。

类似的继承方式也发生在 bin 脚本中，当我们安装一个包时，它们存储在`node_modules/.bin`中。`npm run`会临时将条目添加到 shell PATH 变量（Unix 上为`$PATH`，Windows 上为`%Path%`）：

+   包目录中的`node_modules/.bin`

+   包目录的父目录中的`node_modules/.bin`

+   等等。

要查看这些添加，我们可以使用以下包脚本：

```js
"bin-dirs": "node -p \"JS\""
```

`JS`代表一行 JavaScript 代码：

```js
(process.env.PATH ?? process.env.Path)
.split(path.delimiter)
.filter(p => p.includes('.bin'))
```

在 Unix 上，如果我们运行`bin-dirs`，我们会得到以下输出：

```js
% npm -s run bin-dirs
[
  '/Users/robin/new-package/node_modules/.bin',
  '/Users/robin/node_modules/.bin',
  '/Users/node_modules/.bin',
  '/node_modules/.bin'
]
```

在 Windows 上，我们得到：

```js
>npm -s run bin-dirs
[
  'C:\\Users\\charlie\\new-package\\node_modules\\.bin',
  'C:\\Users\\charlie\\node_modules\\.bin',
  'C:\\Users\\node_modules\\.bin',
  'C:\\node_modules\\.bin'
]
```

### 15.4 在包脚本中使用环境变量

在诸如 Make、Grunt 和 Gulp 之类的任务运行器中，变量很重要，因为它们有助于减少冗余。遗憾的是，虽然包脚本没有自己的变量，但我们可以通过使用*环境变量*（也称为*shell 变量*）来解决这个缺陷。

我们可以使用以下命令列出特定于平台的环境变量：

+   Unix：`env`

+   Windows 命令 shell：`SET`

+   两个平台：`node -p process.env`

在 macOS 上，结果看起来像这样：

```js
TERM_PROGRAM=Apple_Terminal
SHELL=/bin/zsh
TMPDIR=/var/folders/ph/sz0384m11vxf5byk12fzjms40000gn/T/
USER=robin
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
PWD=/Users/robin/new-package
HOME=/Users/robin
LOGNAME=robin
···
```

在 Windows 命令 shell 中，结果看起来像这样：

```js
Path=C:\Windows;C:\Users\charlie\AppData\Roaming\npm;···
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROMPT=$P$G
TEMP=C:\Users\charlie\AppData\Local\Temp
TMP=C:\Users\charlie\AppData\Local\Temp
USERNAME=charlie
USERPROFILE=C:\Users\charlie
···
```

此外，npm 在运行包脚本之前会临时添加更多的环境变量。为了查看最终结果是什么样子，我们可以使用以下命令：

```js
npm run env
```

这个命令调用了一个内置的包脚本。让我们尝试一下这个`package.json`：

```js
{
 "name": "@my-scope/new-package",
 "version": "1.0.0",
 "bin": {
 "hello": "./hello.mjs"
 },
 "config": {
 "stringProp": "yes",
 "arrayProp": ["a", "b", "c"],
 "objectProp": {
 "one": 1,
 "two": 2
 }
 }
}
```

所有 npm 的临时变量的名称都以`npm_`开头。让我们按字母顺序打印这些变量：

```js
npm run env | grep npm_ | sort
```

`npm_`变量具有分层结构。在`npm_lifecycle_`下，我们找到了当前运行的包脚本的名称和定义：

```js
npm_lifecycle_event: 'env',
npm_lifecycle_script: 'env',
```

在 Windows 上，`npm_lifecycle_script`在这种情况下会`SET`。

在前缀`npm_config_`下，我们可以看到一些 npm 的配置设置（[在 npm 文档中有描述](https://docs.npmjs.com/cli/v8/using-npm/config)）。以下是一些例子：

```js
npm_config_cache: '/Users/robin/.npm',
npm_config_global_prefix: '/usr/local',
npm_config_globalconfig: '/usr/local/etc/npmrc',
npm_config_local_prefix: '/Users/robin/new-package',
npm_config_prefix: '/usr/local'
npm_config_user_agent: 'npm/8.15.0 node/v18.7.0 darwin arm64 workspaces/false',
npm_config_userconfig: '/Users/robin/.npmrc',
```

前缀`npm_package_`让我们可以访问`package.json`的内容。其顶层看起来像这样：

```js
npm_package_json: '/Users/robin/new-package/package.json',
npm_package_name: '@my-scope/new-package',
npm_package_version: '1.0.0',
```

在`npm_package_bin_`下，我们可以找到`package.json`属性`"bin"`的属性：

```js
npm_package_bin_hello: 'hello.mjs',
```

`npm_package_config_`条目让我们可以访问`"config"`的属性：

```js
npm_package_config_arrayProp: 'a\n\nb\n\nc',
npm_package_config_objectProp_one: '1',
npm_package_config_objectProp_two: '2',
npm_package_config_stringProp: 'yes',
```

这意味着`"config"`让我们设置可以在包脚本中使用的变量。下一小节将进一步探讨这一点。

请注意，对象被转换为“嵌套”条目（第 2 行和第 3 行），而数组（第 1 行）和数字（第 2 行和第 3 行）被转换为字符串。

这些是剩下的`npm_`环境变量：

```js
npm_command: 'run-script',
npm_execpath: '/usr/local/lib/node_modules/npm/bin/npm-cli.js',
npm_node_execpath: '/usr/local/bin/node',
```

#### 15.4.1 获取和设置环境变量

以下的`package.json`演示了我们如何在包脚本中访问通过`"config"`定义的变量：

```js
{
 "scripts": {
 "hi:unix": "echo $​npm_package_config_hi",
 "hi:windows": "echo %​npm_package_config_hi%"
 },
 "config": {
 "hi": "HELLO"
 }
}
```

遗憾的是，没有内置的跨平台方式可以从包脚本中访问环境变量。

然而，有一些带有 bin 脚本的包可以帮助我们。

[Package `env-var`](https://github.com/rauschma/env-var)让我们获取环境变量：

```js
"scripts": {
 "hi": "env-var echo {{npm_package_config_hi}}"
}
```

[Package `cross-env`](https://github.com/kentcdodds/cross-env)让我们设置环境变量：

```js
"scripts": {
 "build": "cross-env FIRST=one SECOND=two node ./build.mjs"
}
```

#### 15.4.2 通过`.env`文件设置环境变量

还有一些包可以让我们通过`.env`文件设置环境变量。这些文件具有以下格式：

```js
# Comment
SECRET_HOST="https://example.com"
SECRET_KEY="123456789" # another comment
```

使用与`package.json`分开的文件使我们能够将数据排除在版本控制之外。

这些是支持`.env`文件的包：

+   [Package `dotenv`](https://github.com/motdotla/dotenv)支持 JavaScript 模块的`.env`文件。我们可以预加载它：

    ```js
    node -r dotenv/config app.mjs
    ```

    我们可以导入它：

    ```js
    import dotenv from 'dotenv';
    dotenv.config();
    console.log(process.env);
    ```

+   [Package `node-env-run`](https://github.com/dkundel/node-env-run)让我们通过 shell 命令使用`.env`文件：

    ```js
    # Loads `.env` and runs an arbitrary shell script.
    # If there are CLI options, we need to use `--`.
    nodenv --exec node -- -p process.env.SECRET

    # Loads `.env` and uses `node` to run `script.mjs`.
    nodenv script.mjs
    ```

+   [Package `env-cmd`](https://github.com/toddbluhm/env-cmd)是前一个包的替代品：

    ```js
    # Loads `.env` and runs an arbitrary shell script
    env-cmd node -p process.env.SECRET
    ```

    该包还具有更多功能：在变量集之间切换，更多文件格式等。

### 15.5 包脚本的参数

让我们探讨如何将参数传递给我们通过包脚本调用的 shell 命令。我们将使用以下`package.json`：

```js
{
 ···
 "scripts": {
 "args": "log-args"
 },
 "dependencies": {
 "log-args": "¹.0.0"
 }
}
```

bin 脚本`log-args`如下所示：

```js
for (const [key,value] of Object.entries(process.env)) {
 if (key.startsWith('npm_config_arg')) {
 console.log(`${key}=${JSON.stringify(value)}`);
 }
}
console.log(process.argv.slice(2));
```

位置参数按预期工作：

```js
% npm -s run args three positional arguments
[ 'three', 'positional', 'arguments' ]
```

`npm run`使用选项并为它们创建环境变量。它们不会添加到`process.argv`中：

```js
% npm -s run args --arg1='first arg' --arg2='second arg'
npm_config_arg2="second arg"
npm_config_arg1="first arg"
[]
```

如果我们希望选项出现在`process.argv`中，我们必须使用*选项终结符*`--`。该终结符通常在包脚本名称之后插入：

```js
% npm -s run args -- --arg1='first arg' --arg2='second arg' 
[ '--arg1=first arg', '--arg2=second arg' ]
```

但我们也可以在该名称之前插入它：

```js
% npm -s run -- args --arg1='first arg' --arg2='second arg' 
[ '--arg1=first arg', '--arg2=second arg' ]
```

### 15.6 npm 日志级别（产生多少输出）

npm 支持以下日志级别：

| 日志级别 | `npm`选项 | 别名 |
| --- | --- | --- |
| 静默 | `--loglevel silent` | `-s --silent` |
| 错误 | `--loglevel error` |  |
| 警告 | `--loglevel warn` | `-q --quiet` |
| 注意 | `--loglevel notice` |  |
| http | `--loglevel http` |  |
| 时间 | `--loglevel timing` |  |
| 信息 | `--loglevel info` | `-d` |
| 详细 | `--loglevel verbose` | `-dd --verbose` |
| 荒谬 | `--loglevel silly` | `-ddd` |

日志记录指的是两种活动：

+   将信息打印到终端

+   将信息写入 npm 日志

以下各小节描述：

+   日志级别如何影响这些活动。原则上，`silent`记录最少，而`silly`记录最多。

+   如何配置日志记录。前表显示了如何通过命令行选项临时更改日志级别，但还有更多设置。我们可以将它们临时或永久更改。

#### 15.6.1 日志级别和打印到终端的信息

默认情况下，包脚本在终端输出方面相对冗长。例如，以下`package.json`文件：

```js
{
 "name": "@my-scope/new-package",
 "version": "1.0.0",
 "scripts": {
 "hello": "echo Hello",
 "err": "more does-not-exist.txt"
 },
 ···
}
```

如果日志级别高于`silent`且包脚本在没有错误的情况下退出，则会发生以下情况：

```js
% npm run hello

> @my-scope/new-package@1.0.0 hello
> echo Hello

Hello
```

如果日志级别高于`silent`且包脚本失败，则会发生以下情况：

```js
% npm run err      

> @my-scope/new-package@1.0.0 err
> more does-not-exist.txt

does-not-exist.txt: No such file or directory
```

使用日志级别`silent`，输出变得不那么混乱：

```js
% npm -s run hello
Hello

% npm -s run err
does-not-exist.txt: No such file or directory
```

一些错误被`-s`吞没：

```js
% npm -s run abc
%
```

我们至少需要日志级别`error`才能看到它们：

```js
% npm --loglevel error run abc
npm ERR! Missing script: "abc"
npm ERR! 
npm ERR! To see a list of scripts, run:
npm ERR!   npm run

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/robin/.npm/_logs/2072-08-30T14_59_40_474Z-debug-0.log
```

不幸的是，日志级别`silent`也会抑制`npm run`的输出（无参数）：

```js
% npm -s run
%
```

#### 15.6.2 日志级别和写入 npm 日志的信息

默认情况下，日志将被写入 npm 缓存目录，我们可以通过`npm config`获取其路径：

```js
% npm config get cache
/Users/robin/.npm
```

日志目录的内容如下：

```js
% ls -1 /Users/robin/.npm/_logs
2072-08-28T11_44_38_499Z-debug-0.log
2072-08-28T11_45_45_703Z-debug-0.log
2072-08-28T11_52_04_345Z-debug-0.log
```

日志中的每一行都以行索引和日志级别开头。这是一个使用日志级别`notice`写入的日志的示例。有趣的是，即使是比`notice`更详细的日志级别（如`silly`）也会显示出来：

```js
0 verbose cli /usr/local/bin/node /usr/local/bin/npm
1 info using npm@8.15.0
···
33 silly logfile done cleaning log files
34 timing command:run Completed in 9ms
···
```

如果`npm run`返回错误，相应的日志以这种方式结束：

```js
34 timing command:run Completed in 7ms
35 verbose exit 1
36 timing npm Completed in 28ms
37 verbose code 1
```

如果没有错误，相应的日志记录以这种方式结束：

```js
34 timing command:run Completed in 7ms
35 verbose exit 0
36 timing npm Completed in 26ms
37 info ok
```

#### 15.6.3 配置日志记录

`npm config list --long`打印各种设置的默认值。这些是与日志记录相关的设置的默认值：

```js
% npm config list --long | grep log
loglevel = "notice"
logs-dir = null
logs-max = 10
```

如果`logs-dir`的值为`null`，npm 将使用 npm 缓存目录内的目录`_logs`（如前所述）。

+   `logs-dir`允许我们覆盖默认设置，使 npm 将其日志写入我们选择的目录。

+   `logs-max`允许我们配置 npm 在删除旧文件之前写入日志目录的文件数。如果将`logs-max`设置为 0，则不会写入任何日志。

+   `loglevel`允许我们配置 npm 的日志级别。

要永久更改这些设置，我们还可以使用`npm config` - 例如：

+   获取当前的日志级别：

    ```js
    npm config get loglevel
    ```

+   永久设置当前的日志级别：

    ```js
    npm config set loglevel silent
    ```

+   永久重置日志级别为内置默认值：

    ```js
    npm config delete loglevel
    ```

我们还可以通过命令行选项临时更改设置 - 例如：

```js
npm --loglevel silent run build
```

其他更改设置的方式（例如使用环境变量）由[npm 文档](https://docs.npmjs.com/cli/v8/using-npm/config)解释。

#### 15.6.4 在`npm install`期间运行的生命周期脚本的输出

在`npm install`期间运行的生命周期脚本的输出（无参数）是隐藏的。我们可以通过（临时或永久）将`foreground-scripts`设置为`true`来更改这一点。

#### 15.6.5 npm 日志工作的观察

+   只有日志级别为`silent`时，使用`npm run`时才会关闭额外的输出。

+   日志级别对于是否创建日志文件以及写入到日志文件中的内容没有影响。

+   错误消息不会被写入日志。

### 15.7 跨平台 shell 脚本

用于包脚本的两个最常用的 shell 是：

+   Unix 上的`sh`

+   Windows 上的`cmd.exe`

在本节中，我们研究了在两个 shell 中都有效的构造。

#### 15.7.1 路径和引用

提示：

+   使用由斜杠分隔的相对路径段：Windows 接受斜杠作为分隔符，即使在该平台上通常使用反斜杠。

+   双引号参数：虽然`sh`支持单引号，但 Windows 命令 shell 不支持。不幸的是，当我们在包脚本定义中使用双引号时，我们必须对其进行转义：

    ```js
    "dir": "mkdir \"\my dir""
    ```

#### 15.7.2 链接命令

有两种方式可以链接在两个平台上都有效的命令：

+   `&&`之后的命令仅在前一个命令成功时执行（退出代码为 0）。

+   `||`之后的命令仅在前一个命令失败时执行（退出代码不为 0）。

忽略退出代码的链接在不同平台上有所不同：

+   Unix：`;`

+   Windows 命令 shell：`&`

以下交互演示了在 Unix 上`&&`和`||`的工作方式（在 Windows 上，我们会使用`dir`而不是`ls`）：

```js
% ls unknown && echo "SUCCESS" || echo "FAILURE"
ls: unknown: No such file or directory
FAILURE

% ls package.json && echo "SUCCESS" || echo "FAILURE"
package.json
SUCCESS
```

#### 15.7.3 包脚本的退出代码

退出代码可以通过 shell 变量访问：

+   Unix：`$?`

+   Windows 命令 shell：`%errorlevel%`

`npm run`返回与上次执行的 shell 脚本相同的退出代码：

```js
{
 ···
 "scripts": {
 "hello": "echo Hello",
 "err": "more does-not-exist.txt"
 }
}
```

以下交互发生在 Unix 上：

```js
% npm -s run hello ; echo $?
Hello
0
% npm -s run err ; echo $?
does-not-exist.txt: No such file or directory
1
```

#### 15.7.4 管道和重定向输入和输出

+   在命令之间进行管道传输：`|`

+   将输出写入文件：`cmd > stdout-saved-to-file.txt`

+   从文件中读取输入：`cmd < stdin-from-file.txt`

#### 15.7.5 在两个平台上都有效的命令

以下命令在两个平台上都存在（但在选项方面有所不同）：

+   `cd`

+   `echo`。在 Windows 上要注意：双引号会被打印出来，而不是被忽略。

+   `exit`

+   `mkdir`

+   `more`

+   `rmdir`

+   `sort`

#### 15.7.6 运行 bin 脚本和包内部模块

以下的`package.json`演示了在依赖项中调用 bin 脚本的三种方式：

```js
{
 "scripts": {
 "hi1": "./node_modules/.bin/cowsay Hello",
 "hi2": "cowsay Hello",
 "hi3": "npx cowsay Hello"
 },
 "dependencies": {
 "cowsay": "¹.5.0"
 }
}
```

解释：

+   `hi1`：依赖项中的 bin 脚本安装在目录`node_modules/.bin`中。

+   `hi2`：正如我们所见，npm 在执行包脚本时会将`node_modules/.bin`添加到 shell PATH 中。这意味着我们可以像全局安装一样使用本地 bin 脚本。

+   `hi3`：当`npx`运行脚本时，它还会将`node_modules/.bin`添加到 shell PATH 中。

在 Unix 上，我们可以直接调用包本地脚本 - 如果它们有 hashbangs 并且是可执行的。然而，在 Windows 上这种方法行不通，这就是为什么最好通过`node`来调用它们：

```js
"build": "node ./build.mjs"
```

#### 15.7.7 `node --eval`和`node --print`

当一个包脚本的功能变得太复杂时，通常最好通过 Node.js 模块来实现它 - 这样可以轻松编写跨平台代码。

但是，我们也可以使用`node`命令来运行小的 JavaScript 片段，这对于以跨平台的方式执行小任务非常有用。相关的选项是：

+   [`node --eval <expr>`](https://nodejs.org/api/cli.html#-e---eval-script)评估 JavaScript 表达式`expr`。

    +   缩写：`node -e`

+   [`node --print <expr>`](https://nodejs.org/api/cli.html#-p---print-script)评估 JavaScript 表达式`expr`并将结果打印到终端。

    +   缩写：`node -p`

以下命令在 Unix 和 Windows 上都适用（只有注释是 Unix 特定的）：

```js
# Print a string to the terminal (cross-platform echo)
node -p "'How are you?'"

# Print the value of an environment variable
# (Alas, we can’t change variables via `process.env`)
node -p process.env.USER # only Unix
node -p process.env.USERNAME # only Windows
node -p "process.env.USER ?? process.env.USERNAME"

# Print all environment variables
node -p process.env

# Print the current working directory
node -p "process.cwd()"

# Print the path of the current home directory
node -p "os.homedir()"

# Print the path of the current temporary directory
node -p "os.tmpdir()"

# Print the contents of a text file
node -p "fs.readFileSync('package.json', 'utf-8')"

# Write a string to a file
node -e "fs.writeFileSync('file.txt', 'Text content', 'utf-8')"
```

如果我们需要特定于平台的行终止符，我们可以使用`os.EOL`，例如，我们可以在前一个命令中用`'Text content'`替换：

```js
`line 1${os.EOL}line2${os.EOL}`
```

观察：

+   如果 JavaScript 代码包含括号，将其放在双引号中很重要，否则 Unix 会报错。

+   所有内置模块都可以通过变量访问。这就是为什么我们不需要导入`os`或`fs`。

+   `fs`支持更多的文件系统操作。这些在§8“在 Node.js 上使用文件系统”中有文档记录。

### 15.8 常见操作的辅助软件包

#### 15.8.1 从命令行运行软件包脚本

[npm-quick-run](https://github.com/bahmutov/npm-quick-run)提供了一个 bin 脚本`nr`，让我们可以使用缩写来运行软件包脚本，例如：

+   `nr m -w`执行`"npm run mocha -- -w"`（如果`"mocha"`是以“m”开头的第一个软件包脚本）。

+   `nr c:o`运行软件包脚本`"cypress:open"`。

+   等等。

#### 15.8.2 同时或顺序运行多个脚本

同时运行 shell 脚本：

+   Unix：`&`

+   Windows 命令 shell：`start`

以下两个软件包为我们提供了跨平台的选项和相关功能：

+   [concurrently](https://github.com/open-cli-tools/concurrently)同时运行多个 shell 命令，例如：

    ```js
    concurrently "npm run clean" "npm run build"
    ```

+   [npm-run-all](https://github.com/mysticatea/npm-run-all/)提供了几种功能，例如：

    +   调用软件包脚本的更方便的方式。以下两个命令是等价的：

        ```js
        npm-run-all clean lint build
        npm run clean && npm run lint && npm run build
        ```

    +   同时运行软件包脚本：

        ```js
        npm-run-all --parallel lint build
        ```

    +   使用通配符运行多个脚本，例如，`watch:*`代表所有以`watch:`开头的软件包脚本（`watch:html`、`watch:js`等）：

        ```js
        npm-run-all "watch:*"
        npm-run-all --parallel "watch:*"
        ```

#### 15.8.3 文件系统操作

[Package `shx`](https://github.com/shelljs/shx)让我们可以使用“Unix 语法”来运行各种文件系统操作。它在 Unix 和 Windows 上的所有操作都有效。

创建目录：

```js
"create-asset-dir": "shx mkdir ./assets"
```

删除目录：

```js
"remove-asset-dir": "shx rm -rf ./assets"
```

清空目录（双引号是为了安全起见，关于通配符`*`）：

```js
"tscclean": "shx rm -rf \"./dist/*\""
```

复制文件：

```js
"copy-index": "shx cp ./html/index.html ./out/index.html"
```

删除文件：

```js
"remove-index": "shx rm ./out/index.html"
```

`shx`基于 JavaScript 库 ShellJS，其存储库列出了[所有支持的命令](https://github.com/shelljs/shelljs#command-reference)。除了我们已经看到的 Unix 命令之外，它还模拟：`cat`、`chmod`、`echo`、`find`、`grep`、`head`、`ln`、`ls`、`mv`、`pwd`、`sed`、`sort`、`tail`、`touch`、`uniq`等。

#### 15.8.4 将文件或目录放入垃圾箱

[Package `trash-cli`](https://github.com/sindresorhus/trash-cli)适用于 macOS（10.12+）、Linux 和 Windows（8+）。它将文件和目录放入垃圾箱，并支持路径和 glob 模式。以下是使用它的示例：

```js
trash tmp-file.txt
trash tmp-dir
trash "*.jpg"
```

#### 15.8.5 复制文件树

[Package `copyfiles`](https://github.com/calvinmetcalf/copyfiles)让我们可以复制文件树。

`copyfiles`的用例如下：在 TypeScript 中，我们可以导入非代码资产，如 CSS 和图像。TypeScript 编译器将代码编译到“dist”（输出）目录，但忽略非代码资产。这个跨平台的 shell 命令将它们复制到 dist 目录：

```js
copyfiles --up 1 "./ts/**/*.{css,png,svg,gif}" ./dist
```

TypeScript 编译：

```js
my-pkg/ts/client/picker.ts  -> my-pkg/dist/client/picker.js
```

`copy-assets`复制：

```js
my-pkg/ts/client/picker.css -> my-pkg/dist/client/picker.css
my-pkg/ts/client/icon.svg   -> my-pkg/dist/client/icon.svg
```

#### 15.8.6 监视文件

[Package `onchange`](https://github.com/Qard/onchange)监视文件并在每次更改时运行 shell 命令，例如：

```js
onchange 'app/**/*.js' 'test/**/*.js' -- npm test
```

一个常见的替代方案（还有许多其他）：

+   [`nodemon`](https://github.com/remy/nodemon)

#### 15.8.7 其他功能

+   [cli-error-notifier](https://github.com/micromata/cli-error-notifier) 如果脚本失败（具有非零退出代码），则显示本机桌面通知。它支持许多操作系统。

#### 15.8.8 HTTP 服务器

在开发过程中，通常需要一个 HTTP 服务器。以下包（以及许多其他包）可以帮助：

+   [`http-server`](https://github.com/http-party/http-server)

+   [`live-server`](https://github.com/tapio/live-server)

+   [`serve`](https://github.com/vercel/serve)

### 15.9 扩展包脚本的功能

#### 15.9.1 `per-env`: 根据 `$NODE_ENV` 在不同脚本之间切换

[The bin script `per-env`](https://github.com/ericclemmons/per-env) 允许我们运行一个包脚本 `SCRIPT`，并根据环境变量 `NODE_ENV` 的值自动在 `SCRIPT:development`、`SCRIPT:staging` 和 `SCRIPT:production` 之间切换：

```js
{
 "scripts": {
 // If NODE_ENV is missing, the default is "development"
 "build": "per-env",

 "build:development": "webpack -d --watch",
 "build:staging": "webpack -p",
 "build:production": "webpack -p"
 },
 // Processes spawned by `per-env` inherit environment-specific
 // variables, if defined.
 "per-env": {
 "production": {
 "DOCKER_USER": "my",
 "DOCKER_REPO": "project"
 }
 }
}
```

#### 15.9.2 定义特定操作系统的脚本

[The bin script `cross-os`](https://github.com/milewski/cross-os) 根据当前操作系统在不同脚本之间切换。

```js
{
 "scripts": {
 "user": "cross-os user"
 },
 "cross-os": {
 "user": {
 "darwin": "echo $USER",
 "win32": "echo %USERNAME%",
 "linux": "echo $USER"
 }
 },
 ···
}
```

支持的属性值有：`darwin`、`freebsd`、`linux`、`sunos`、`win32`。

### 15.10 本章的来源

+   [npm documentation](https://docs.npmjs.com)

+   [Node.js documentation](https://nodejs.org/api/)

+   [“Awesome npm scripts”](https://github.com/RyanZim/awesome-npm-scripts) 由 [Ryan Zimmerman](https://github.com/RyanZim) 和 [Michael Kühnel](https://github.com/mischah) 编写

+   [“Three Things You Didn’t Know You Could Do with npm Scripts”](https://www.twilio.com/blog/npm-scripts) 由 [Dominik Kundel](https://twitter.com/dkundel)

+   [“Helpers and tips for npm run scripts”](https://michael-kuehnel.de/tooling/2018/03/22/helpers-and-tips-for-npm-run-scripts.html) 由 [Michael Kühnel](https://twitter.com/mkuehnel)

[Comments](https://github.com/rauschma/nodejs-shell-scripting/issues/15)
