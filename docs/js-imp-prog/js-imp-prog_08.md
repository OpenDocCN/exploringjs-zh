# 六、使用 JavaScript：整体概况

> 原文：[`exploringjs.com/impatient-js/ch_big-picture.html`](https://exploringjs.com/impatient-js/ch_big-picture.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   6.1 你在这本书中学到了什么？

+   6.2 浏览器和 Node.js 的结构

+   6.3 JavaScript 参考

+   6.4 进一步阅读

* * *

在这一章中，我想描绘整体概况：你在这本书中学到了什么，它如何融入 Web 开发的整体格局？

### 6.1 你在这本书中学到了什么？

这本书教授 JavaScript 语言。它专注于语言本身，但偶尔也会提到 JavaScript 可以使用的两个平台：

+   Web 浏览器

+   Node.js

Node.js 在 Web 开发中有三种重要的用途：

+   你可以用它来用 JavaScript 编写服务器端软件。

+   你也可以用它来编写命令行软件（比如 Unix shell、Windows PowerShell 等）。许多与 JavaScript 相关的工具都是基于（并通过）Node.js 执行的。

+   Node 的软件注册表 npm 已成为安装工具（如编译器和构建工具）和库的主要方式-甚至用于客户端开发。

### 6.2 浏览器和 Node.js 的结构

![图 2：两个 JavaScript 平台 web 浏览器和 Node.js 的结构。API“标准库”和“平台 API”托管在具有 JavaScript 引擎和特定平台“核心”的基础层之上。](img/e6d9b16bd5efb55d1d6f2c5283d1de6d.png)

图 2：两个 JavaScript 平台*web 浏览器*和*Node.js*的结构。API“标准库”和“平台 API”托管在具有 JavaScript 引擎和特定平台“核心”的基础层之上。

两个 JavaScript 平台*web 浏览器*和*Node.js*的结构相似（图 2）：

+   基础层包括 JavaScript 引擎和特定平台的“核心”功能。

+   两个 API 托管在这个基础之上：

    +   JavaScript 标准库是 JavaScript 的一部分，运行在引擎之上。

    +   平台 API 也可以从 JavaScript 中使用-它提供对特定平台功能的访问。例如：

        +   在浏览器中，如果你想做任何与用户界面相关的事情，你需要使用特定平台 API：响应鼠标点击，播放声音等。

        +   在 Node.js 中，特定平台 API 允许你读写文件，通过 HTTP 下载数据等。

### 6.3 JavaScript 参考

当你对 JavaScript 有疑问时，通常可以通过网络搜索来解决。我可以推荐以下在线资源：

+   [MDN web 文档](https://developer.mozilla.org/en-US/)：涵盖各种 Web 技术，如 CSS、HTML、JavaScript 等。是一个很好的参考资料。

+   [Node.js 文档](https://nodejs.org/en/docs/)：记录了 Node.js API。

+   [ExploringJS.com](https://exploringjs.com)：我在 JavaScript 上的其他书籍比这本书更详细，并且可以免费在线阅读。你可以按 ECMAScript 版本查找功能：

    +   ES1–ES5：[*Speaking JavaScript*](http://speakingjs.com/)

    +   ES6：[*探索 ES6*](https://exploringjs.com/es6.html)

    +   ES2016–ES2017：[*探索 ES2016 和 ES2017*](https://exploringjs.com/es2016-es2017.html)

    +   等等。

### 6.4 进一步阅读

+   额外章节提供了对 Web 开发的更全面的了解。

[评论](https://github.com/rauschma/impatient-js/issues/32)
