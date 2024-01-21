# 前言

> 原文：[Preface](https://exploringjs.com/es5/pr02.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


由于它在网络上的普及和其他因素，JavaScript 变得难以避免。但这并不意味着它受到了良好的喜爱。通过这本书，我希望能说服你，虽然在使用它时你必须接受相当多的怪癖，但 JavaScript 是一种不错的语言，可以让你非常高效地工作，并且编程起来也很有趣。

尽管我从诞生开始就一直关注它的发展，但我花了很长时间才开始喜欢 JavaScript。然而，当我最终开始喜欢它时，事实证明，我的先前经验已经为我做好了充分的准备，因为我曾经使用过 Scheme、Java（包括 GWT）、Python、Perl 和 Self（所有这些都对 JavaScript 产生了影响）。

2010 年，我意识到了 Node.js，这让我希望最终能够在服务器和客户端都使用 JavaScript。因此，我将 JavaScript 作为我的主要编程语言。在学习它的过程中，我开始写一本记录我的发现的书。这就是你现在正在阅读的书。在我的博客上，我发表了这本书的部分内容和其他关于 JavaScript 的材料。这在几个方面帮助了我：积极的反应鼓励我继续前行，使写作这本书不再孤单；博客文章的评论给了我额外的信息和建议（正如这本书中的所有地方所承认的）；它让人们意识到我的工作，最终导致了 O'Reilly 出版了这本书。

因此，这本书已经历了三年多的时间。在这段时间里，我不断完善了它的内容。我很高兴这本书终于完成了，并希望人们会发现它对学习 JavaScript 很有用。O'Reilly 同意免费在线阅读，这应该有助于让更广泛的读者群体能够接触到这本书。

## 关于本书你需要知道的事情

这本书适合你吗？以下几点可以帮助你确定：

这本书是为谁写的

这本书是由程序员写给程序员的。因此，为了理解它，你应该已经了解面向对象的编程，例如通过主流编程语言如 Java、PHP、C++、Python、Ruby、Objective-C、C#或 Perl。

因此，这本书的目标受众是想要快速而正确地学习 JavaScript 的程序员，以及想要深化他们的技能和/或查找特定主题的 JavaScript 程序员。

未涵盖的内容

这本书专注于 JavaScript 语言本身。例如，你不会在这里找到关于编程网页浏览器（DOM、异步编程等）的信息。然而，第三十三章指向相关材料。

本书的组织结构

这本书分为四个部分，但主要的两个部分是：

+   JavaScript 快速入门

+   深入 JavaScript

这些部分是完全独立的！你可以把它们当作是独立的书籍：前者更像是一本指南，后者更像是一本参考书。[本书的四个部分](pr02.html#book_parts "本书的四个部分")告诉你更多关于本书结构的信息。

这本书使用 JavaScript 的哪个版本

这本书教授的是 ECMAScript 5，这是所有现代引擎都支持的 JavaScript 的当前版本。如果你不得不使用旧的网络浏览器，那么第二十五章解释了 ECMAScript 5 独有的功能。

## 阅读本书的提示

学习 JavaScript 最重要的提示是*不要被细节困扰*。是的，当涉及到语言时有很多细节，而这本书涵盖了大部分。但我也会向你指出一个相对简单而优雅的“大局”，

### 本书的四个部分

这本书分为四个部分：

[第 I 部分](pt01.html "第 I 部分。JavaScript 快速入门")

这部分教你“基本 JavaScript”，这是 JavaScript 的一个子集，尽可能小，同时仍然能让您高效工作。这部分是独立的；它不依赖于其他部分，其他部分也不依赖于它。

[第二部分](pt02.html "第二部分. 背景")

这部分将 JavaScript 放在历史和技术背景中：它是何时、为什么以及如何创建的？它与其他编程语言有什么关系？我们今天所处的重要步骤是什么？

[第三部分](pt03.html "第三部分. 深入 JavaScript")

这部分更多的是一个参考：寻找你感兴趣的主题，跳进去，探索一下。许多简短的例子应该可以防止事情变得太枯燥。

[第四部分](pt04.html "第四部分. 技巧、工具和库")

这部分提供了使用 JavaScript 的技巧：最佳实践、高级技术和学习资源。它还描述了一些重要的工具和库。

### JavaScript 命令行

在阅读本书时，您可能希望准备好一个命令行。这样可以让您交互式地尝试代码。最受欢迎的选择是：

[Node.js](http://nodejs.org)

Node.js 带有一个交互式命令行。您可以通过调用 shell 命令`node`来启动它。

浏览器

所有主要浏览器都有控制台，用于输入在当前页面上下文中评估的 JavaScript。只需在线搜索您的浏览器名称和“控制台”。

### 符号约定

本书中始终使用以下符号约定。

#### 描述语法

问号（？）用于标记可选参数。例如：

```js
parseInt(str, radix?)
```

法语引号（guillemets）表示元代码。您可以将这样的元代码视为空白，由实际代码填充。例如：

```js
try {
    «try_statements»
}
```

“白色”方括号标记可选的语法元素。例如：

```js
break ⟦«label»⟧
```

在 JavaScript 注释中，我有时使用反引号来区分 JavaScript 和英语：

```js
foo(x, y); // calling function `foo` with parameters `x` and `y`
```

#### 引用方法

我通过它们的完整路径引用内置方法：

```js
«Constructor».prototype.«methodName»()
```

例如，`Array.prototype.join()`指的是数组方法`join()`；也就是说，JavaScript 将`Array`实例的方法存储在对象`Array.prototype`中。这样做的原因在[第 3 层：构造函数-实例的工厂](ch17_split_001.html#constructors "第 3 层：构造函数-实例的工厂")中有解释。

#### 命令行交互

每当我介绍一个新概念时，我经常通过 JavaScript 命令行中的交互来说明它。看起来是这样的：

```js
> 3 + 4
7
```

大于号后面的文本是由人类输入的输入。其他所有内容都是 JavaScript 引擎输出的。此外，我使用`console.log()`方法将数据打印到控制台，特别是在（非命令行）源代码中：

```js
var x = 3;
x++;
console.log(x); // 4
```

#### 提示、注释和警告

### 提示

这个元素表示提示或建议。

### 注意

这个元素表示一般说明。

### 警告

这个元素表示警告或注意。

### 快速查找文档

虽然您显然可以将本书用作参考，但有时在线查找信息会更快。我推荐的一个资源是[Mozilla 开发者网络](https://developer.mozilla.org/en-US/)（MDN）。您可以搜索网络以找到 MDN 的文档。例如，以下网络搜索找到了数组的`push()`方法的文档：

```js
mdn array push
```

## Safari® Books Online

### 注意

[Safari Books Online](http://my.safaribooksonline.com/?portal=oreilly)是一个按需数字图书馆，提供来自技术和商业领域领先作者的专业内容，包括书籍和视频形式。

技术专业人员、软件开发人员、网页设计师以及商业和创意专业人士将 Safari Books Online 作为他们进行研究、解决问题、学习和认证培训的主要资源。

Safari Books Online 为[组织](http://www.safaribooksonline.com/organizations-teams)、[政府机构](http://www.safaribooksonline.com/government)和[个人](http://www.safaribooksonline.com/individuals)提供一系列[产品组合](http://www.safaribooksonline.com/subscriptions)和定价方案。订阅者可以访问来自 O’Reilly Media、Prentice Hall Professional、Addison-Wesley Professional、Microsoft Press、Sams、Que、Peachpit Press、Focal Press、Cisco Press、John Wiley & Sons、Syngress、Morgan Kaufmann、IBM Redbooks、Packt、Adobe Press、FT Press、Apress、Manning、New Riders、McGraw-Hill、Jones & Bartlett、Course Technology 等出版商的数千本图书、培训视频和预出版手稿，这些都可以在一个完全可搜索的数据库中找到。有关 Safari Books Online 的更多信息，请访问我们的[网站](http://www.safaribooksonline.com/)。

## 如何联系我们

请将有关本书的评论和问题发送给出版商：

| O’Reilly Media, Inc. |
| --- |
| 1005 Gravenstein Highway North |
| Sebastopol, CA 95472 |
| 800-998-9938（美国或加拿大）|
| 707-829-0515（国际或本地）|
| 707-829-0104（传真）|

我们为这本书准备了一个网页，列出勘误、示例和任何额外信息。您可以在[`oreil.ly/speaking-js`](http://oreil.ly/speaking-js)上访问此页面。

要就本书发表评论或提出技术问题，请发送电子邮件至[bookquestions@oreilly.com](mailto:bookquestions@oreilly.com)。

有关我们的图书、课程、会议和新闻的更多信息，请访问我们的网站[`www.oreilly.com`](http://www.oreilly.com)。

在 Facebook 上找到我们：[`facebook.com/oreilly`](http://facebook.com/oreilly)

在 Twitter 上关注我们：[`twitter.com/oreillymedia`](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[`www.youtube.com/oreillymedia`](http://www.youtube.com/oreillymedia)

## 致谢

我要感谢以下所有帮助使这本书成为可能的人。

### 为 JavaScript 做准备

以下人员按时间顺序奠定了我对 JavaScript 的理解基础：

+   François Bry 教授，Sven Panne 和 Tim Geisler（Scheme）

+   Don Batory 教授（技术写作，编程语言设计）

+   Martin Wirsing 教授，Alexander Knapp，Matthias Hölzl，Hubert Baumeister 以及慕尼黑大学信息学院的其他前同事（形式方法，各种软件工程主题）

### JavaScript 帮助

es-discuss 邮件列表的参与者

他们的回答帮助我理解了 JavaScript 的设计。我对他们的耐心和不懈的努力深表感谢。有四个人脱颖而出：Brendan Eich，Allen Wirfs-Brock，Mark Miller 和 David Herman。

我博客[2ality](http://www.2ality.com)的读者

我在博客上发布了这本书的片段，得到了大量有用的反馈。其中有几个名字：Ben Alman，Brandon Benvie，Mathias Bynens，Andrea Giammarchi，Matthias Reuter 和 Rick Waldron。

更多来源在各章中得到了确认。

### 审阅者

我非常感谢审阅本书的以下人员。他们提供了重要的反馈和更正。按字母顺序排列：

+   Mathias Bynens

+   Raymond Camden

+   Cody Lindley

+   Shelley Powers

+   Andreas Schroeder

+   Alex Stangl

+   Béla Varga

+   Edward Yue Shung Wong
