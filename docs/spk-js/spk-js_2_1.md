# 第二章 为什么选择 JavaScript？

> 原文：[2. Why JavaScript?](https://exploringjs.com/es5/ch02.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


有很多编程语言。为什么你要使用 JavaScript？本章将从七个重要方面来看，这些方面在你选择编程语言时很重要，并且认为 JavaScript 总体上做得很好：

1.  它是免费提供的吗？

1.  它是一种优雅的编程语言吗？

1.  在实践中有用吗？

1.  它有好的工具，特别是好的*集成开发环境*（IDE）吗？

1.  它对你想做的事情来说足够快吗？

1.  它被广泛使用吗？

1.  它有未来吗？

## JavaScript 是免费提供的吗？

JavaScript 可以说是最开放的编程语言：它的规范 ECMA-262 是 ISO 标准。许多独立方实现都紧密遵循这一规范。其中一些实现是开源的。此外，语言的演变由 TC39 委员会负责，该委员会由包括所有主要浏览器供应商在内的几家公司组成。其中许多公司通常是竞争对手，但他们为了语言的利益而共同合作。

## JavaScript 优雅吗？

是和不是。我用不同范式的几种编程语言写了大量代码。因此，我很清楚 JavaScript 并不是优雅的巅峰。然而，它是一种非常灵活的语言，有一个相当优雅的核心，并且使你能够使用面向对象编程和函数式编程的混合。

JavaScript 引擎之间的语言兼容性曾经是一个问题，但现在不再是了，部分得益于[测试 262 套件](https://github.com/tc39/test262)，该套件检查引擎是否符合 ECMAScript 规范。相比之下，浏览器和 DOM 的差异仍然是一个挑战。这就是为什么通常最好依赖框架来隐藏这些差异。

## JavaScript 有用吗？

世界上最美丽的编程语言是无用的，除非它能让你编写你需要的程序。

### 图形用户界面

在图形用户界面领域，JavaScript 受益于成为*HTML5*的一部分。在本节中，我使用 HTML5 这个术语来表示“浏览器平台”（HTML、CSS 和浏览器 JavaScript API）。HTML5 得到了广泛的部署，并不断取得进展。它正在慢慢地成为一个完整的层，用于编写功能齐全的跨平台应用程序；类似于 Java 平台，它几乎像一个嵌入式操作系统。HTML5 的卖点之一是它让你编写跨平台的图形用户界面。这些总是一种妥协：你放弃了一些质量，以换取不受限于单一操作系统。过去，“跨平台”意味着 Windows、Mac OS 或 Linux。但现在我们有了两个额外的交互平台：Web 和移动。使用 HTML5，你可以通过诸如[PhoneGap](http://phonegap.com)、[Chrome Apps](http://developer.chrome.com/apps/)和[TideSDK](http://www.tidesdk.org/)等技术来针对所有这些平台。

此外，一些平台将 Web 应用程序作为本地应用程序或允许你本地安装它们——例如 Chrome OS、Firefox OS 和 Android。

### 其他补充 JavaScript 的技术

除了 HTML5 之外，还有更多的技术可以补充 JavaScript，使语言更有用：

库

JavaScript 有大量的库，可以让你完成各种任务，从解析 JavaScript（通过[Esprima](http://esprima.org)）到处理和显示 PDF 文件（通过[PDF.js](https://github.com/mozilla/pdf.js)）。

[Node.js](http://nodejs.org)

Node.js 平台让你编写服务器端代码和 shell 脚本（构建工具、测试运行器等）。

JSON（JavaScript 对象表示法，在第二十二章中介绍）

JSON 是一种根植于 JavaScript 的数据格式，在 Web 上交换数据变得流行（例如网络服务的结果）。

NoSQL 数据库（例如[CouchDB](http://couchdb.apache.org)和[MongoDB](http://www.mongodb.org)）

这些数据库紧密集成了 JSON 和 JavaScript。

## JavaScript 有好的工具吗？

JavaScript 正在获得更好的构建工具（例如[Grunt](http://gruntjs.com)）和测试工具（例如[mocha](http://visionmedia.github.io/mocha/)）。Node.js 使得可以通过 shell 运行这些类型的工具（不仅仅在浏览器中）。在这个领域的一个风险是分裂，因为我们逐渐得到了太多这样的工具。

JavaScript 的 IDE 空间仍处于萌芽阶段，但正在迅速成长。网络开发的复杂性和动态性使得这个空间成为创新的肥沃土壤。两个开源的例子是[Brackets](http://brackets.io)和[Light Table](http://www.lighttable.com)。

此外，浏览器正变得越来越强大的开发环境。特别是 Chrome 最近取得了令人印象深刻的进展。有趣的是看到 IDE 和浏览器在未来将整合到多大程度。

## JavaScript 足够快吗？

JavaScript 引擎取得了巨大的进步，从缓慢的解释器发展为快速的即时编译器。它们现在已经足够快，适用于大多数应用。此外，已经在开发新的想法，使 JavaScript 足够快以适用于其余的应用：

+   [asm.js](http://asmjs.org/)是 JavaScript 的（非常静态的）子集，在当前引擎上运行速度很快，大约相当于编译后的 C++的 70%。例如，它可以用于实现网络应用的性能关键算法部分。它还被用于将基于 C++的游戏移植到网络平台上。

+   [ParallelJS](http://www.2ality.com/2013/12/paralleljs.html)可以并行化使用新数组方法`mapPar`、`filterPar`和`reducePar`（现有数组方法`map`、`filter`和`reduce`的可并行化版本）的 JavaScript 代码。为了使并行化工作，回调必须以特殊的方式编写；主要限制是不能改变在回调中未创建的数据。

## JavaScript 被广泛使用吗？

通常广泛使用的语言有两个好处。首先，这样的语言有更好的文档和支持。其次，更多的程序员知道它，这在你需要雇佣某人或者寻找基于该语言的工具的客户时非常重要。

JavaScript 被广泛使用，并获得了前述两个好处：

+   如今，JavaScript 的文档和支持以各种形式呈现：书籍、播客、博客文章、电子邮件通讯、论坛等等。第三十三章指引您前往重要资源。

+   JavaScript 开发人员需求量大，但他们的人数也在不断增加。

## JavaScript 有未来吗？

有几件事表明 JavaScript 有一个光明的未来：

+   语言正在稳步发展；ECMAScript 6 看起来不错。

+   有许多与 JavaScript 相关的创新（例如前述的 asm.js 和 ParallelJS，微软的 TypeScript 等）。

+   JavaScript 作为一个不可或缺的部分所在的网络平台正在迅速成熟。

+   JavaScript 得到了众多公司的支持，没有单个人或公司控制它。

## 结论

考虑到使一种语言具有吸引力的前述因素，JavaScript 的表现非常出色。它当然并不完美，但目前很难超越它，而且情况只会变得更好。

