# 一、关于这本书

> 原文：[`exploringjs.com/deep-js/ch_about-book.html`](https://exploringjs.com/deep-js/ch_about-book.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   1.1 这本书的主页在哪里？

+   1.2 这本书包括什么？

+   1.3 我用我的钱能得到什么？

+   1.4 我如何预览内容？

+   1.5 我如何报告错误？

+   1.6 阅读提示

+   1.7 符号和约定

    +   1.7.1 什么是类型签名？为什么我在这本书中看到静态类型？

    +   1.7.2 带图标的注释是什么意思？

+   1.8 致谢

* * *

### 1.1 这本书的主页在哪里？

“深入 JavaScript”的主页是[`exploringjs.com/deep-js/`](https://exploringjs.com/deep-js/)

### 1.2 这本书包括什么？

这本书深入探讨了 JavaScript：

+   它教授了更好地使用这种语言的实用技术。

+   它教授了这种语言的工作原理和原因。它所教授的内容牢固地基于 ECMAScript 规范（本书对此进行了解释和参考）。

+   它只涵盖语言（忽略特定平台的功能，如浏览器 API），但不是详尽无遗。相反，它专注于一些重要的主题。

### 1.3 我用我的钱能得到什么？

如果你购买这本书，你会得到：

+   当前内容有四个无 DRM 的版本：

    +   PDF 文件

    +   无广告 HTML 的 ZIP 存档

    +   EPUB 文件

    +   MOBI 文件

+   任何未来添加到这个版本的内容。我能添加多少取决于这本书的销售情况。

当前价格是介绍性的。随着内容的增加，价格会上涨。

### 1.4 我如何预览内容？

[在这本书的主页](https://exploringjs.com/deep-js/#previews)，有关于这本书所有版本的广泛预览。

### 1.5 我如何报告错误？

+   这本书的 HTML 版本在每章的末尾有一个评论链接。

+   它们跳转到 GitHub 的问题页面，[你也可以直接访问](https://github.com/rauschma/deep-js/issues)。

### 1.6 阅读提示

+   你可以按任何顺序阅读章节。每一章都是独立的，但偶尔会有参考其他章节的进一步信息。

+   一些章节的标题标有“(可选)”，意思是它们不是必要的。如果你跳过它们，你仍然会理解章节的其余部分。

### 1.7 符号和约定

#### 1.7.1 什么是类型签名？为什么我在这本书中看到静态类型？

例如，你可能会看到：

```js
Number.isFinite(num: number): boolean
```

这被称为`Number.isFinite()`的*类型签名*。这种符号，特别是`num`的静态类型`number`和结果的`boolean`，并不是真正的 JavaScript。这种符号是从编译成 JavaScript 的语言 TypeScript 中借用的（它主要是 JavaScript 加上静态类型）。

为什么使用这种符号？它可以帮助你快速了解一个函数是如何工作的。这种符号在[2ality 博客文章](https://2ality.com/2018/04/type-notation-typescript.html)中有详细解释，但通常相对直观。

#### 1.7.2 带图标的注释是什么意思？

![](img/6b17215a2abf7f098e996c026fba60fd.png)  **阅读说明**

解释了如何最好地阅读内容。

![](img/0c2cffc9b09a6e4a1ac19d36b7230eed.png)  **外部内容**

指向额外的外部内容。

![](img/c17f3239b9e1f5d335bb0adf8211d9d3.png)  **提示**

提供了与当前内容相关的提示。

![](img/c619d77adc13394437a61f6e9047056c.png)  **问题**

问及并回答与当前内容相关的问题（常见问题解答）。

![](img/c43c94e3731896959890f3b542413c88.png)  **警告**

警告关于陷阱等。

![](img/290901f5575b7fa8e8b287bbaf550458.png) **细节**

提供额外的细节，补充当前的内容。类似于脚注。

### 1.8 致谢

+   感谢 Allen Wirfs-Brock 通过 Twitter 和博客评论给予的建议。这些帮助使本书变得更好。

+   更多为本书作出贡献的人在各章中得到了感谢。

[评论](https://github.com/rauschma/deep-js/issues/1)
