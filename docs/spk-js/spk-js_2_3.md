# 第四章：JavaScript 的创建方式

> 原文：[4. How JavaScript Was Created](https://exploringjs.com/es5/ch04.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


了解 JavaScript 的创建原因和方式有助于我们理解它的特点。

1993 年，NCSA 的 Mosaic 是第一个广受欢迎的 Web 浏览器。1994 年，成立了一家名为网景的公司，以利用新兴的万维网的潜力。网景创建了专有的 Web 浏览器 Netscape Navigator，在 1990 年代占主导地位。许多最初的 Mosaic 作者继续在 Navigator 上工作，但两者故意没有共享代码。

网景很快意识到 Web 需要变得更加动态。即使你只是想检查用户在表单中输入的正确值，也需要将数据发送到服务器以便提供反馈。1995 年，网景聘请了 Brendan Eich，并承诺让他在浏览器中实现 Scheme（一种 Lisp 方言）。²在他开始之前，网景与硬件和软件公司 Sun（后来被 Oracle 收购）合作，将其更静态的编程语言 Java 包含在 Navigator 中。因此，网景内部激烈争论的一个问题是为什么 Web 需要两种编程语言：Java 和一种脚本语言。脚本语言的支持者提出了以下解释：³

> 我们旨在为 Web 设计师和兼职程序员提供“粘合语言”，他们正在从图像、插件和 Java 小程序等组件构建 Web 内容。我们认为 Java 是由高价程序员使用的“组件语言”，而粘合程序员——Web 页面设计师——将组件组装起来，并使用[一种脚本语言]自动化它们的交互。

当时，网景管理层决定脚本语言必须具有类似于 Java 的语法。这排除了采用现有的语言，如 Perl、Python、TCL 或 Scheme。为了捍卫 JavaScript 的想法，网景需要一个原型。艾奇在 1995 年 5 月的 10 天内写了一个原型。JavaScript 的第一个代号是 Mocha，由马克·安德森创造。后来，网景营销部门出于商标原因和因为几个产品的名称已经带有前缀“Live”，将其更改为 LiveScript。1995 年 11 月底，Navigator 2.0B3 发布，包括原型，继续在早期没有进行重大更改的情况下存在。1995 年 12 月初，Java 的势头增长，Sun 授权商标 Java 给网景。语言再次更名为最终名称 JavaScript。⁴

* * *

² Brendan Eich，“流行程度”，2008 年 4 月 3 日，[`bit.ly/1lKl6fG`](http://bit.ly/1lKl6fG)。

³ Naomi Hamilton，“编程语言 A-Z：JavaScript”，Computerworld，2008 年 7 月 30 日，[`bit.ly/1lKldIe`](http://bit.ly/1lKldIe)。

⁴ Paul Krill，“JavaScript 创造者思考过去和未来”，InfoWorld，2008 年 6 月 23 日，[`bit.ly/1lKlpXO`](http://bit.ly/1lKlpXO)；Brendan Eich，“JavaScript 简史”，2010 年 7 月 21 日，[`bit.ly/1lKkI0M`](http://bit.ly/1lKkI0M)。

