# 第六章：历史 JavaScript 里程碑

> 原文：[6. Historical JavaScript Milestones](https://exploringjs.com/es5/ch06.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 花了很长时间才产生影响。许多与 JavaScript 相关的技术存在了一段时间，直到它们被主流发现。本节描述了从 JavaScript 的创建到今天发生的事情。在整个过程中，只提到了最受欢迎的项目，而忽略了许多项目，即使它们是第一个。例如，列出了 Dojo Toolkit，但也有较少人知道的[qooxdoo](http://qooxdoo.org/)，它是在同一时间创建的。还列出了 Node.js，尽管[Jaxer](https://github.com/aptana/Jaxer)在它之前就存在：

1997 年—[动态 HTML](http://bit.ly/1oNVOzH)

动态 HTML 允许您动态更改网页的内容和外观。您可以通过操作页面的文档对象模型（DOM）来实现这一点，这是一种树状数据结构。您可以做的事情包括更改内容，更改样式，显示和隐藏元素。动态 HTML 首次出现在 Internet Explorer 4 和 Netscape Navigator 4 中。

1999 年—[XMLHttpRequest](http://www.w3.org/TR/XMLHttpRequest/)

此 API 允许客户端脚本向服务器发送 HTTP 或 HTTPS 请求并返回数据，通常以文本格式（XML，HTML，JSON）返回。它是在 Internet Explorer 5 中引入的。

2001 年—[JSON](http://json.org/)，基于 JavaScript 的数据交换格式

2001 年，道格拉斯·克罗克福德命名并记录了 JSON（JavaScript 对象表示法），其主要思想是使用 JavaScript 语法以文本格式存储数据。 JSON 使用 JavaScript 文字来表示对象，数组，字符串，数字和布尔值以表示结构化数据。例如：

```js
{
    "first": "Jane",
    "last": "Porter",
    "married": true,
    "born": 1890,
    "friends": [ "Tarzan", "Cheeta" ]
}
```

多年来，JSON 已成为 XML 的受欢迎的轻量级替代品，特别是在需要表示结构化数据而不是标记时。自然地，JSON 易于通过 JavaScript 消耗（参见第二十二章）。

2004 年—[Dojo Toolkit](http://dojotoolkit.org/)，用于大规模编程 JavaScript 的框架

Dojo Toolkit 通过提供必要的基础设施来促进大规模编程：继承库，模块系统，用于桌面式图形小部件的 API 等。

2005 年—[Ajax](http://bit.ly/1oNW3Lf)，基于浏览器的桌面级应用程序

Ajax 是一组技术，为网页带来了与桌面应用程序相媲美的交互水平。一个令人印象深刻的例子是 2005 年 2 月推出的 Google 地图。该应用程序允许您在世界地图上平移和缩放，但只有当前可见的内容才会下载到浏览器中。在 Google 地图推出后，杰西·詹姆斯·加勒特注意到它与其他交互式网站共享某些特征。他将这些特征称为*Ajax*，这是*异步 JavaScript 和 XML*的简称。Ajax 的两个基石是在后台异步加载内容（通过`XMLHttpRequest`）并动态更新当前页面的结果（通过动态 HTML）。这是一个相当大的可用性改进，可以避免始终执行完整的页面重新加载。

Ajax 标志着 JavaScript 和动态 Web 应用程序的主流突破。有趣的是要注意这花了多长时间——在那时，Ajax 的成分已经可用多年。自 Ajax 诞生以来，其他数据格式变得流行（JSON 取代 XML），使用其他协议（例如，除 HTTP 外还使用 Web Sockets），并且双向通信是可能的。但基本技术仍然是相同的。然而，这个术语*Ajax*如今使用得少得多，大多数情况下已经被更全面的术语*HTML5*和*Web 平台*（两者都意味着 JavaScript 加上浏览器 API）所取代。

2005 年—[Apache CouchDB](http://couchdb.apache.org/)，一个以 JavaScript 为中心的数据库

大致上，CouchDB 是一个 JSON 数据库：您可以向其提供 JSON 对象，无需事先指定模式。此外，您可以通过执行 map/reduce 操作的 JavaScript 函数定义视图和索引。因此，CouchDB 非常适合 JavaScript，因为您可以直接使用本机数据。与关系数据库相比，没有映射相关的阻抗不匹配。与对象数据库相比，您避免了许多复杂性，因为只存储数据，而不是行为。CouchDB 只是几个类似的[NoSQL 数据库](http://bit.ly/1oNYfCp)中的一个。它们中的大多数都具有出色的 JavaScript 支持。

2006 年—[jQuery](http://jquery.com/)，帮助 DOM 操作

浏览器 DOM 是客户端 Web 开发中最痛苦的部分之一。jQuery 通过在浏览器差异上进行抽象和提供强大的流畅式 API 来使 DOM 操作变得有趣，从而使 DOM 操作变得有趣。

2007 年—[WebKit](https://www.webkit.org/)，将移动 Web 推向主流

基于 KDE 的先前工作，WebKit 是苹果于 2003 年推出的 HTML 引擎。它于 2005 年开源。随着 iPhone 于 2007 年的推出，移动 Web 突然变得主流，并与非移动 Web 相比几乎没有限制。

2008 年—[V8](http://code.google.com/p/v8/)，证明 JavaScript 可以很快

当谷歌推出其 Chrome 网络浏览器时，其中一个亮点是一个名为 V8 的快速 JavaScript 引擎。它改变了 JavaScript 速度慢的看法，并引发了与其他浏览器供应商的速度竞赛，我们至今仍在受益。V8 是开源的，可以在需要快速嵌入式语言时作为独立组件使用。

2009 年—[Node.js](http://nodejs.org/)，在服务器上实现 JavaScript

Node.js 允许您实现在负载下表现良好的服务器。为此，它使用事件驱动的非阻塞 I/O 和 JavaScript（通过 V8）。Node.js 的创始人 Ryan Dahl 提到了选择 JavaScript 的以下原因：

+   “因为它是裸露的，没有 I/O API。”[因此，Node.js 可以引入自己的非阻塞 API。]

+   “Web 开发人员已经在使用它。”[JavaScript 是一种广为人知的语言，特别是在 Web 环境中。]

+   “DOM API 是基于事件的。每个人都已经习惯了在没有线程和事件循环上运行。”[开发人员习惯于异步编码风格。]

Dahl 能够在事件驱动服务器和服务器端 JavaScript（主要是[CommonJS](http://www.commonjs.org/)项目）的先前工作基础上构建。

Node.js 对 JavaScript 程序员的吸引力不仅在于能够使用熟悉的语言进行编程；您可以在客户端和服务器上使用相同的语言。这意味着您可以共享更多的代码（例如，用于验证数据）并使用诸如[*同构 JavaScript*](http://bit.ly/1gWhLIs)之类的技术。同构 JavaScript 是关于在客户端或服务器上组装网页，具有许多好处：可以在服务器上呈现页面以实现更快的初始显示、SEO 以及在不支持 JavaScript 或版本过旧的浏览器上运行。但它们也可以在客户端上更新，从而实现更具响应性的用户界面。

2009 年—[PhoneGap](http://phonegap.com/)，使用 HTML5 编写本机应用程序

PhoneGap 是由一家名为 Nitobi 的公司创建的，后来被 Adobe 收购。PhoneGap 的开源基础称为 Cordova。PhoneGap 最初的使命是通过 HTML5 实现原生移动应用程序。自那时起，支持已扩展到非移动操作系统。目前支持的平台包括 Android，Bada，BlackBerry，Firefox OS，iOS，Mac OS X，Tizen，Ubuntu，Windows（桌面）和 Windows Phone。除了 HTML5 API 之外，还有专门用于访问加速计，相机和联系人等[本机功能](http://bit.ly/1oO22Q9)的 PhoneGap 特定 API。

2009 年—[Chrome OS](http://bit.ly/1oO27U2)，使浏览器成为操作系统

对于 Chrome OS 来说，Web 平台就是本机平台。这种方法有几个优点：

+   创建操作系统要容易得多，因为所有用户界面技术都已经存在。

+   许多开发人员已经（大部分）知道如何为操作系统编写应用程序。

+   管理应用程序很简单。这有助于公共场所的安装，如网吧和学校。

移动操作系统[webOS](http://bit.ly/1oO2e1N)的推出（起源于 Palm，现在由 LG Electronics 拥有）早于 Chrome OS 的推出，但后者更明显地体现了“浏览器作为操作系统”的理念（这就是为什么它被选为里程碑的原因）。webOS 既少，因为它非常专注于手机和平板电脑。又多，因为它内置了 Node.js，可以让您用 JavaScript 实现服务。在 Web 操作系统类别中，最近的一个新进是 Mozilla 的[Firefox OS](http://mzl.la/1oO2i1J)，它针对手机和平板电脑。 [Mozilla 的维基](http://mzl.la/1oO2n5m)提到 Web 操作系统对 Web 的好处：

> 我们还需要一个目标，以便确定和专注我们的努力。最近，我们看到了 pdf.js 项目[通过 HTML5 渲染 PDF，无需插件]暴露出一些需要填补的小差距，以便使“HTML5”成为 PDF 的超集。现在，我们希望迈出更大的一步，找到阻止 Web 开发人员能够构建与 iPhone，Android 和 WP7 为等同的本机应用程序的差距。

2011 年—[Windows 8](http://bit.ly/1oO2qhJ)，一流的 HTML5 应用程序

当微软推出 Windows 8 时，它让所有人都感到惊讶，因为该操作系统广泛集成了 HTML5。在 Windows 8 中，HTML5 应用程序与通过现有技术（如.NET 和 C++）实现的应用程序平等。为了证明这一点，微软用 HTML5（加上对本机 API 的调用）编写了几个重要的 Windows 8 应用程序，包括应用商店和电子邮件应用程序。

* * *

⁵ Ajax 是一个缩写词，但不是一个首字母缩写，这就是为什么它没有被写成 AJAX。
