# 第三十章：库

> 原文：[30. Libraries](https://exploringjs.com/es5/ch30.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章介绍了 JavaScript 库。首先解释了 shim 和 polyfill 是什么，这两种特殊的库。然后列出了一些核心库。最后，指向了其他与库相关的资源。

## Shim 与 Polyfill 的区别

Shim 和 polyfill 是在旧的 JavaScript 引擎上改进新功能的库：

+   *Shim*是一个库，它将新的 API 引入到旧的环境中，只使用该环境的手段。

+   *填充物*是浏览器 API 的替代品。它通常检查浏览器是否支持 API。如果不支持，填充物会安装自己的实现。这样你就可以在任何情况下使用 API。术语*填充物*来自于家居改进产品；根据[Remy Sharp](http://bit.ly/MmZZmZ)的说法：

> Polyfilla 是一种在美国被称为 Spackling Paste 的英国产品。考虑到这一点：把浏览器想象成有裂缝的墙。这些[填充物]有助于填平裂缝，使我们得到一个漂亮平滑的浏览器墙来使用。

示例包括：

+   [“HTML5 跨浏览器填充物”](http://bit.ly/1oOGuTE)：由 Paul Irish 编制的列表。

+   [es5-shim](http://bit.ly/1oOGxi4)是一个（非填充物）shim，它在 ECMAScript 3 引擎上改进了 ECMAScript 5 的功能。它纯粹与语言相关，无论是在 Node.js 上还是在浏览器上都是有意义的。

## 四种语言库

以下库已经相当成熟并且接近语言。了解它们是有用的：

+   ECMAScript 国际化 API 有助于处理与国际化相关的任务：排序和搜索字符串、数字格式化以及日期和时间格式化。下一节将更详细地解释此 API。

+   [Underscore.js](http://underscorejs.org)通过数组、对象、函数等工具函数来补充 JavaScript 相对稀疏的标准库。由于 Underscore 早于 ECMAScript 5，因此与标准库存在一些重叠。然而，这是一个特性：在旧版浏览器上，您可以获得通常只有 ECMAScript-5 才有的功能；在 ECMAScript 5 上，相关函数简单地转发到标准库。

+   [Lo-Dash](http://lodash.com)是 Underscore.js API 的另一种实现，具有一些额外的功能。访问网站了解它是否比 Underscore.js 更适合您。

+   [XRegExp](http://xregexp.com)是一个具有多个高级功能的正则表达式库，例如命名捕获和自由间隔（允许您将正则表达式分布在多行并逐行记录）。在幕后，增强的正则表达式被转换为普通的正则表达式，这意味着您在使用 XRegExp 时不会付出性能代价。

## ECMAScript 国际化 API

ECMAScript 国际化 API 是一个标准的 JavaScript API，用于处理与国际化相关的任务：排序和搜索字符串、数字格式化以及日期和时间格式化。本节简要概述并指向更多阅读材料。

### ECMAScript 国际化 API，第 1 版

API 的第一版提供了以下服务：

+   *排序*支持两种场景：对一组字符串进行排序和在一组字符串中进行搜索。排序是由区域设置参数化的，并且了解 Unicode。

+   *数字格式化*。参数包括：

+   格式化样式：十进制、货币（由其他参数确定的货币种类和如何引用）

+   区域设置（直接指定或最佳匹配，通过匹配器对象搜索）

+   编号系统（西方数字、阿拉伯数字、泰国数字等）

+   精度：整数位数、小数位数、有效数字位数

+   分组分隔符打开或关闭

+   *日期和时间格式*。参数包括：

+   要格式化的信息以及使用哪种样式（短、长、数字等）

+   一个区域设置

+   一个时区

大部分功能通过全局变量`Intl`中的对象访问，但 API 还增强了以下方法：

+   `String.prototype.localeCompare`

+   `Number.prototype.toLocaleString`

+   `Date.prototype.toLocaleString`

+   `Date.prototype.toLocaleDateString`

+   `Date.prototype.toLocaleTimeString`

### 它是什么样的标准？

标准“ECMAScript 国际化 API”（EIA）的编号是 ECMA-402。它由 Ecma International 托管，该协会还托管 ECMA-262，即 ECMAScript 语言规范。这两个标准都由 TC39 维护。因此，EIA 是最接近语言的标准，而不是 ECMA-262 的一部分。该 API 已经设计用于与 ECMAScript 5 和 ECMAScript 6 一起使用。一套一致性测试补充了标准，并确保 API 的各种实现是兼容的（ECMA-262 有类似的测试套件）。

### 我什么时候可以使用它？

大多数现代浏览器已经支持它，或者正在支持它的过程中。David Storey 创建了一个详细的[兼容性表](http://bit.ly/1oOGIdo)（指示哪些浏览器支持哪些区域设置等）。

### 进一步阅读

ECMAScript 国际化 API 的[规范](http://bit.ly/1oOGQth)由 Norbert Lindenberg 编辑。它以 PDF、HTML 和 EPUB 格式提供。此外，还有几篇全面的介绍性文章：

+   [“ECMAScript 国际化 API”](http://bit.ly/1oOGT8C) by Norbert Lindenberg

+   [“ECMAScript 国际化 API”](http://bit.ly/1oOGYcc) by David Storey

+   [“使用 JavaScript 国际化 API”](http://bit.ly/1oOH2sz) by Marcos Caceres

## JavaScript 资源目录

本节描述了收集 JavaScript 资源信息的网站。有几种这样的目录。

以下是 JavaScript 的一般目录列表：

+   [“JavaScriptOO：您应该关注的每个 JavaScript 项目”](http://www.javascriptoo.com/)

+   [JSDB](http://www.jsdb.io/)：最好的 JavaScript 库集合

+   [JSter](http://jster.net/)：JavaScript 库和开发工具目录

+   [“HTML5、JavaScript 和 CSS 资源的主列表”](http://bit.ly/1oOH7MW)

专门的目录包括：

+   [“Microjs：出色的微框架和微库，用于娱乐和利润”](http://microjs.com/)

+   [“Unheap：整洁的 jQuery 插件存储库”](http://www.unheap.com/)

显然，您可以直接浏览包管理器的注册表：

+   [npm](https://npmjs.org/)（Node Packaged Modules）

+   [Bower](http://bower.io/)

CDN（内容交付网络）和 CDN 内容的目录包括：

+   [jsDelivr](http://www.jsdelivr.com/)：JavaScript 库、jQuery 插件、CSS 框架、字体等的免费 CDN

+   [“cdnjs：JavaScript 和 CSS 的缺失 CDN”](http://cdnjs.com/)（托管不太流行的库）

### 致谢

以下人员为本节做出了贡献：Kyle Simpson（@getify），Gildas Lormeau（@check_ca），Fredrik Sogaard（@fredrik_sogaard），Gene Loparco（@gloparco），Manuel Strehl（@m_strehl）和 Elijah Manor（@elijahmanor）。

