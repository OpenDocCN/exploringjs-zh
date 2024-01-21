# 第三十二章：更多工具

> 原文：[32. More Tools](https://exploringjs.com/es5/ch32.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


模块系统和包管理器在第三十一章中有介绍。但还有其他重要的工具类别：

Linting

Lint 工具分析源代码并报告潜在问题和样式违规。三种流行的工具是：

+   [JSLint](http://www.jslint.com) by Douglas Crockford

+   [JSHint](http://www.jshint.com) by Anton Kovalyov

+   [ESLint](https://github.com/nzakas/eslint) by Nicholas Zakas

单元测试

理想情况下，一个单元测试框架可以在两个主要的 JavaScript 平台——浏览器和 Node.js 上运行。两个重要的框架是：

+   [Jasmine](http://pivotal.github.io/jasmine/)

+   [mocha](http://visionmedia.github.io/mocha/)

缩小

JavaScript 源代码通常会浪费空间——变量名比需要的要长，有注释，额外的空白等等。缩小工具可以去除这些浪费并将代码编译成更小的代码。移除过程的一些部分相对复杂（例如，将变量重命名为短名称）。三种流行的缩小工具是：

+   [UglifyJS](https://github.com/mishoo/UglifyJS2/)

+   [YUI Compressor](https://github.com/yui/yuicompressor)

+   [Closure Compiler](https://developers.google.com/closure/compiler/)

构建

对于大多数项目，您需要对其构件应用许多操作：lint 代码，编译代码（即使在 Web 项目中也会发生编译——例如，将诸如 LESS 或 Sass 之类的 CSS 语言编译为普通 CSS），缩小代码等等。构建工具可以帮助您完成这些操作。Unix 的两个经典示例是 make 和 Java 的 Ant。JavaScript 的两个流行的构建工具是[Grunt](http://gruntjs.com)和[Gulp](http://gulpjs.com/)。它们最引人注目的特点之一是您可以在使用它们时保持在 JavaScript 中；它们都基于 Node.js。

脚手架

脚手架工具设置一个空项目，预先配置构建文件等。[Yo](https://github.com/yeoman/yo)就是这样一个工具。它是[Yeoman](http://yeoman.io)套件的一部分，包括 yo、Bower 和 Grunt。

