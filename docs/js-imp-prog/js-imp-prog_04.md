# 三、JavaScript 的历史和演变

> 原文：[`exploringjs.com/impatient-js/ch_history.html`](https://exploringjs.com/impatient-js/ch_history.html)

* * *

+   3.1 JavaScript 是如何创建的

+   3.2 JavaScript 标准化

+   3.3 ECMAScript 版本的时间表

+   3.4 Ecma 技术委员会 39（TC39）

+   3.5 TC39 流程

    +   3.5.1 提示：考虑单个功能和阶段，而不是 ECMAScript 版本

+   3.6 常见问题：TC39 流程

    +   3.6.1 我的最喜欢的提议功能怎么样？

    +   3.6.2 是否有 ECMAScript 功能的官方列表？

+   3.7 JavaScript 的演变：不要破坏 Web

* * *

### 3.1 JavaScript 是如何创建的

JavaScript 是由 Brendan Eich 于 1995 年 5 月在 10 天内创建的。Eich 在 Netscape 工作，并为他们的 Web 浏览器*Netscape Navigator*实现了 JavaScript。

这个想法是客户端 Web 的主要交互部分应该用 Java 实现。JavaScript 应该是这些部分的粘合语言，并且也使 HTML 稍微更具交互性。鉴于它作为 Java 的辅助角色，JavaScript 必须看起来像 Java。这排除了现有的解决方案，如 Perl，Python，TCL 等。

最初，JavaScript 的名称多次更改：

+   它的代号是*Mocha*。

+   在 Netscape Navigator 2.0 beta 版（1995 年 9 月），它被称为*LiveScript*。

+   在 Netscape Navigator 2.0 beta 3（1995 年 12 月），它得到了最终的名字，*JavaScript*。

### 3.2 JavaScript 标准化

JavaScript 有两个标准：

+   ECMA-262 由 Ecma International 主办。这是主要标准。

+   ISO/IEC 16262 由国际标准化组织（ISO）和国际电工委员会（IEC）主办。这是次要标准。

这些标准描述的语言称为*ECMAScript*，而不是*JavaScript*。选择了不同的名称，因为 Sun（现在是 Oracle）拥有后者的商标。 “ECMAScript”中的“ECMA”来自主要标准的主办组织。

该组织的原始名称是*ECMA*，是*European Computer Manufacturers Association*的首字母缩写。后来更改为*Ecma International*（“Ecma”是一个专有名词，而不是首字母缩写），因为该组织的活动已经扩展到欧洲以外。最初的全大写首字母缩写解释了 ECMAScript 的拼写。

原则上，JavaScript 和 ECMAScript 意思相同。有时会做出以下区分：

+   术语*JavaScript*指的是语言及其实现。

+   术语*ECMAScript*指的是语言标准和语言版本。

因此，*ECMAScript 6*是语言的一个版本（第 6 版）。

### 3.3 ECMAScript 版本的时间表

这是 ECMAScript 版本的简要时间表：

+   ECMAScript 1（1997 年 6 月）：标准的第一个版本。

+   ECMAScript 2（1998 年 6 月）：与 ISO 标准保持一致的小更新。

+   ECMAScript 3（1999 年 12 月）：添加了许多核心功能-“[…]正则表达式，更好的字符串处理，新的控制语句[do-while，switch]，try/catch 异常处理，[…]”

+   ECMAScript 4（2008 年 7 月放弃）：本来会是一次大规模升级（包括静态类型、模块、命名空间等），但最终变得过于雄心勃勃，分裂了语言的管理者。

+   ECMAScript 5（2009 年 12 月）：带来了一些小的改进-一些标准库功能和*严格模式*。

+   ECMAScript 5.1（2011 年 6 月）：另一个小更新，以保持 Ecma 和 ISO 标准同步。

+   ECMAScript 6（2015 年 6 月）：一个大型更新，实现了 ECMAScript 4 的许多承诺。这个版本是第一个官方名称为*ECMAScript 2015*的版本，名称是基于出版年份的。

+   ECMAScript 2016（2016 年 6 月）：首次年度发布。较短的发布周期导致新功能较少，与大型 ES6 相比。

+   ECMAScript 2017（2017 年 6 月）：第二次年度发布。

+   随后的 ECMAScript 版本（ES2018 等）总是在 6 月份正式通过。

### 3.4 Ecma 技术委员会 39（TC39）

TC39 是推动 JavaScript 发展的委员会。严格来说，它的成员是公司：Adobe、Apple、Facebook、Google、Microsoft、Mozilla、Opera、Twitter 等。也就是说，通常是激烈竞争的公司正在为了语言的利益而共同合作。

每两个月，TC39 都会举行会议，由成员指定的代表和受邀专家参加。这些会议的记录是公开的，存储在[GitHub 存储库](https://github.com/tc39/tc39-notes/)中。

### 3.5 TC39 流程

随着 ECMAScript 6，当时使用的发布流程出现了两个问题：

+   如果发布之间的时间太长，那么早期准备好的功能就必须等待很长时间才能发布。而准备晚的功能则面临着为了赶上截止日期而被匆忙发布的风险。

+   功能通常是在实现和使用之前设计的。因此，与实现和使用相关的设计缺陷通常太晚才被发现。

为了解决这些问题，TC39 制定了新的*TC39 流程*：

+   ECMAScript 功能是独立设计的，并经历从 0（“草案”）到 4（“完成”）的阶段。

+   特别是后期阶段需要原型实现和实际测试，从而在设计和实现之间形成反馈循环。

+   ECMAScript 版本每年发布一次，并包括在发布截止日期之前达到第 4 阶段的所有功能。

结果是：更小的、增量式的发布，其功能已经经过了现场测试。图 1 说明了 TC39 的流程。

![图 1：每个 ECMAScript 功能提案都经历了从 0 到 4 的阶段。冠军是支持功能作者的 TC39 成员。Test 262 是一套测试，用于检查 JavaScript 引擎是否符合语言规范。](img/eada37ac5c0c14718840a0024395b1e5.png)

图 1：每个 ECMAScript 功能提案都经历了从 0 到 4 的阶段。*冠军*是支持功能作者的 TC39 成员。Test 262 是一套测试，用于检查 JavaScript 引擎是否符合语言规范。

ES2016 是第一个根据 TC39 流程设计的 ECMAScript 版本。

#### 3.5.1 提示：以单独的功能和阶段为思考对象，而不是 ECMAScript 版本

直到 ES6 为止，最常见的是根据 ECMAScript 版本来思考 JavaScript - 例如，“这个浏览器支持 ES6 吗？”

从 ES2016 开始，最好是考虑单独的功能：一旦一个功能达到第 4 阶段，你就可以安全地使用它（如果它受到你所针对的 JavaScript 引擎的支持）。你不必等到下一个 ECMAScript 发布版本。

### 3.6 常见问题：TC39 流程

#### 3.6.1 我最喜欢的提议功能进展如何？

如果你想知道各种提议功能处于哪个阶段，请查阅[GitHub 存储库`proposals`](https://github.com/tc39/proposals)。

#### 3.6.2 是否有一个官方的 ECMAScript 功能列表？

是的，TC39 存储库列出了[已完成的提案](https://github.com/tc39/proposals/blob/master/finished-proposals.md)，并提到它们是在哪个 ECMAScript 版本中引入的。

### 3.7 JavaScript 的发展：不要破坏网络

有时会提出一个想法，即通过删除旧功能和怪癖来清理 JavaScript。虽然这个想法的吸引力是显而易见的，但它有重大的缺点。

假设我们创建了一个不向后兼容并修复了所有缺陷的 JavaScript 的新版本。结果，我们会遇到以下问题：

+   JavaScript 引擎变得臃肿：它们需要支持旧版本和新版本。对于 IDE 和构建工具也是如此。

+   程序员需要了解，并不断意识到，不同版本之间的差异。

+   你可以选择将现有的全部代码迁移到新版本（这可能是很多工作）。或者你可以混合使用版本，重构变得更加困难，因为你不能在不改变代码的情况下在不同版本之间移动代码。

+   你必须以某种方式指定每一段代码的版本 - 无论是文件还是嵌入在网页中的代码 - 它是用哪个版本编写的。每种可行的解决方案都有利有弊。例如，*严格模式*是 ES5 的一个稍微更清洁的版本。它之所以没有像应该的那样受欢迎的一个原因是：通过在文件或函数开头的指令中选择进入是一件麻烦事。

那么解决方案是什么？我们可以两全其美吗？ES6 选择的方法称为“一种 JavaScript”：

+   新版本始终完全向后兼容（但偶尔可能会有一些微小的、几乎察觉不到的清理）。

+   旧功能不会被移除或修复。相反，它们的更好版本被引入。一个例子是通过`let`声明变量 - 这是`var`的改进版本。

+   如果语言的某些方面发生了变化，那是在新的语法结构内完成的。也就是说，你是隐式选择进入的。例如，`yield`只是 ES6 中生成器内的关键字。并且所有模块和类中的代码（都是在 ES6 中引入的）都隐式地处于严格模式。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/23)
