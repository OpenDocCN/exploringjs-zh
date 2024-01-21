# 第五章 标准化：ECMAScript

> 原文：[5. Standardization: ECMAScript](https://exploringjs.com/es5/ch05.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 推出后，微软在 Internet Explorer 3.0（1996 年 8 月）中实现了相同的语言，但名称不同，称为 JScript。为了限制微软，网景决定标准化 JavaScript，并要求标准组织 [Ecma International](http://en.wikipedia.org/wiki/Ecma) 托管标准。ECMA-262 的规范工作始于 1996 年 11 月。由于 Sun（现在是 Oracle）对 *Java* 一词拥有商标，因此标准化的官方名称不能是 *JavaScript*。因此，选择了 *ECMAScript*，源自 *JavaScript* 和 *Ecma*。但是，该名称仅用于指代语言的版本（其中一个指的是规范）。每个人仍然称该语言为 *JavaScript*。

ECMA-262 由 Ecma 的 [技术委员会 39](http://bit.ly/1oNTQiP)（TC39）管理和发展。其成员是微软、Mozilla 和 Google 等公司，它们指定员工参与委员会工作；例如 Brendan Eich、Allen Wirfs-Brock（ECMA-262 的编辑）和 David Herman。为了推进 ECMAScript 的设计，TC39 在开放渠道（如邮件列表 [es-discuss](https://mail.mozilla.org/listinfo/es-discuss)）上举行讨论，并定期举行会议。会议由 TC39 成员和受邀专家参加。2013 年初，与会人数从 15 到 25 人不等。

以下是 ECMAScript 版本（或 ECMA-262 的 *版本*）及其主要特性的列表：

ECMAScript 1（1997 年 6 月）

第一版

ECMAScript 2（1998 年 8 月）

编辑更改以使 ECMA-262 与标准 ISO/IEC 16262 保持一致

ECMAScript 3（1999 年 12 月）

`do-while`，正则表达式，新的字符串方法（`concat`，`match`，`replace`，`slice`，使用正则表达式的`split`等），异常处理等

ECMAScript 4（2008 年 7 月放弃）

ECMAScript 4 被开发为 JavaScript 的下一个版本，原型是用 ML 编写的。然而，TC39 无法就其功能集达成一致。为防止僵局，委员会于 2008 年 7 月底举行会议，并达成了一致，总结在 [四点](http://mzl.la/1oNTUiG) 中：

1.  开发了 ECMAScript 3 的增量更新（成为 ECMAScript 5）。

1.  开发了一个比 ECMAScript 4 更少，但比 ECMAScript 3 的增量更新更多的主要新版本。新版本的代号是 Harmony，因为它的构思是在一个和谐的会议中产生的。Harmony 将分为 ECMAScript 6 和 ECMAScript 7。

1.  ECMAScript 4 中将被删除的特性包括包，命名空间和早期绑定。

1.  其他想法将与 TC39 共识开发。

因此，ECMAScript 4 的开发人员同意使 Harmony 比 ECMAScript 4 更少激进，而 TC39 的其他成员同意继续推动事情向前发展。

ECMAScript 5（2009 年 12 月）

添加了严格模式，获取器和设置器，新的数组方法，对 JSON 的支持等（参见 第二十五章）

ECMAScript 5.1（2011 年 6 月）

编辑更改以使 ECMA-262 与国际标准 ISO/IEC 16262:2011 的第三版保持一致

ECMAScript 6

目前正在开发中，预计将在 2014 年底得到批准。大多数引擎可能会在批准时支持最重要的 ECMAScript 6 特性。完整支持可能需要更长时间。

达成共识并创建标准并不总是容易的，但由于前述各方的协作努力，JavaScript 是一种真正开放的语言，由多个供应商实现，具有非常高的兼容性。这种兼容性是通过非常详细但具体的规范实现的。例如，规范经常使用伪代码，并且它由一个测试套件[test262](http://test262.ecmascript.org/)补充，该测试套件检查 ECMAScript 实现的兼容性。有趣的是，ECMAScript 并不由万维网联盟（W3C）管理。TC39 和 W3C 在 JavaScript 和 HTML5 之间存在重叠时进行合作。

