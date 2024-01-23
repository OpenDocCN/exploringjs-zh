# 18 跨平台考虑

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_cross-platform-considerations.html`](https://exploringjs.com/nodejs-shell-scripting/ch_cross-platform-considerations.html)

* * *

+   18.1 文件系统路径

+   18.2 处理行终止符

+   18.3 检测当前平台

+   18.4 在所有平台上运行与项目相关的任务

* * *

### 18.1 文件系统路径

本书其他地方的材料：

+   §7.2.1 “路径段、路径分隔符、路径分隔符”

+   §7.9 “在不同平台上使用相同的路径”

+   §7.3 “通过模块 `'node:os'` 获取标准目录路径”

### 18.2 处理行终止符

参见 §8.3 “跨平台处理行终止符”。

### 18.3 检测当前平台

+   [`process.platform`](https://nodejs.org/api/process.html#processplatform) 包含一个标识当前平台的字符串。可能的值有：

    +   `'aix'`

    +   `'darwin'`

    +   `'freebsd'`

    +   `'linux'`

    +   `'openbsd'`

    +   `'sunos'`

    +   `'win32'`

+   模块 `'node:os'` 包含更多与平台相关的信息（处理器架构、操作系统版本等）。

### 18.4 在所有平台上运行与项目相关的任务

参见 §15 “通过 npm 包脚本运行跨平台任务”。

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/18)
