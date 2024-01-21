# 第三十一章：模块系统和包管理器

> 原文：[31. Module Systems and Package Managers](https://exploringjs.com/es5/ch31.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 没有内置模块支持，但社区已经创建了令人印象深刻的解决方法。要管理模块，可以使用所谓的*包管理器*，它们处理发现、安装、依赖管理等。

## 模块系统

JavaScript 模块的两个最重要（但不幸兼容）的标准是：

CommonJS 模块（CJS）

这个标准的主要体现是[Node.js 模块](http://nodejs.org/api/modules.html)（Node.js 模块具有一些超出 CJS 的特性）。它的特点包括：

+   紧凑的语法

+   设计用于同步加载

+   主要用途：服务器

异步模块定义（AMD）

这个标准最流行的实现是[RequireJS](http://requirejs.org)。它的特点包括：

+   略微更复杂的语法，使 AMD 可以在没有`eval()`或静态编译步骤的情况下工作

+   设计用于异步加载

+   主要用途：浏览器

## 包管理器

在包管理器方面，[npm](https://npmjs.org)（Node Packaged Modules）是 Node.js 的规范选择。对于浏览器，有两个流行的选项（还有其他）：

+   [Bower](http://bower.io)是一个支持 AMD 和 CJS 的 Web 包管理器。

+   [Browserify](http://browserify.org)是一个基于 npm 的工具，它将 npm 包编译成浏览器可用的东西。

## 快速而肮脏的模块

对于正常的 Web 开发，您应该使用 RequireJS 或 Browserify 等模块系统。但有时您只是想快速拼凑一下。然后以下简单的模块模式可以帮助：

```js
var moduleName = function () {
    function privateFunction () { ... }
    function publicFunction(...) {
        privateFunction();
        otherModule.doSomething();  // implicit import
    }
    return { // exports
        publicFunction: publicFunction
    };
}();
```

上面是一个存储在全局变量`moduleName`中的模块。它执行以下操作：

+   隐式导入一个依赖项（模块`otherModule`）

+   有一个私有函数`privateFunction`

+   导出`publicFunction`

要在网页上使用模块，只需通过`<script>`标签加载其文件和其依赖项的文件：

```js
<script src="modules/otherModule.js"></script>
<script src="modules/moduleName.js"></script>
<script type="text/javascript">
    moduleName.publicFunction(...);
</script>
```

如果在加载模块时没有访问其他模块（这是`moduleName`的情况），那么加载模块的顺序就不重要了。

以下是我的评论和建议：

+   我用了一段时间这个模块模式，直到我发现我并没有发明它，它有一个官方名称。Christian Heilmann 推广了它，并称之为[“揭示模块模式”](http://bit.ly/1c1InUg)。

+   如果您使用此模式，请保持简单。可以随意使用模块名称污染全局范围，但请尽量找到唯一的名称。这只是为了进行一些小技巧，所以没有必要变得复杂（嵌套命名空间，模块分割成多个文件等）。

