# 五、FAQ: JavaScript

> 原文：[`exploringjs.com/impatient-js/ch_faq-language.html`](https://exploringjs.com/impatient-js/ch_faq-language.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)



* * *

+   5.1 JavaScript 有哪些好的参考资料？

+   5.2 我怎样才能知道 JavaScript 的功能在哪里得到支持？

+   5.3 我可以在哪里查找 JavaScript 计划中的功能？

+   5.4 为什么 JavaScript 经常悄悄失败？

+   5.5 为什么我们不能清理 JavaScript，删除怪癖和过时的功能？

+   5.6 我怎样才能快速尝试一段 JavaScript 代码？

* * *

### 5.1 JavaScript 有哪些好的参考资料？

请参阅 §6.3 “JavaScript references”。

### 5.2 我怎样才能知道 JavaScript 的功能在哪里得到支持？

本书通常会提到一个功能是否是 ECMAScript 5 的一部分（老版本浏览器所需），或者是一个更新的版本。有关更详细的信息（包括 ES5 之前的版本），有几个在线的兼容性表格可供参考：

+   [ECMAScript compatibility tables for various engines](http://kangax.github.io/compat-table/es5/) (by [kangax](https://twitter.com/kangax), [webbedspace](https://twitter.com/webbedspace), [zloirock](https://twitter.com/zloirock))

+   [Node.js compatibility tables](https://node.green) (by [William Kapke](https://twitter.com/williamkapke))

+   Mozilla 的[MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)有每个功能的表格，描述了相关的 ECMAScript 版本和浏览器支持。

+   [“Can I use…”](https://caniuse.com/) 记录了 Web 浏览器支持的功能（包括 JavaScript 语言功能）。

### 5.3 我可以在哪里查找 JavaScript 计划中的功能？

请参阅以下来源：

+   §3.5 “The TC39 process” 描述了即将推出的功能是如何计划的。

+   §3.6 “FAQ: TC39 process” 回答了关于即将推出的功能的各种问题。

### 5.4 为什么 JavaScript 经常悄悄失败？

JavaScript 经常悄悄失败。让我们看两个例子。

第一个例子：如果操作数的运算符没有适当的类型，它们会根据需要进行转换。

```js
> '3' * '5'
15
```

第二个例子：如果算术计算失败，你会得到一个错误值，而不是异常。

```js
> 1 / 0
Infinity
```

悄悄失败的原因是历史性的：直到 ECMAScript 3 之前，JavaScript 没有异常。从那时起，它的设计者们试图避免悄悄失败。

### 5.5 为什么我们不能清理 JavaScript，删除怪癖和过时的功能？

这个问题在§3.7 “Evolving JavaScript: Don’t break the web”中得到了回答。

### 5.6 我怎样才能快速尝试一段 JavaScript 代码？

§8.1 “Trying out JavaScript code” 解释了如何做到这一点。

[Comments](https://github.com/rauschma/impatient-js/issues/24)
