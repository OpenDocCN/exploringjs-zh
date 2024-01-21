# 第二十七章：调试的语言机制

> 原文：[27. Language Mechanisms for Debugging](https://exploringjs.com/es5/ch27.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


以下三种语言结构有助于调试。它们显然应该由适当的调试器补充：

+   `debugger`语句的行为类似于断点，并启动调试器。

+   `console.log(x)`将值`x`记录到 JavaScript 引擎的控制台中。

+   `console.trace()`将堆栈跟踪打印到引擎的控制台中。

控制台 API 提供了更多的调试帮助，并在[The Console API](ch23.html#console_api "The Console API")中有更详细的文档。异常处理在第十四章中有解释。

