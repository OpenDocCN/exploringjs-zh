# 二、说明

> 原文：[`exploringjs.com/nodejs-shell-scripting/ch_instructions.html`](https://exploringjs.com/nodejs-shell-scripting/ch_instructions.html)

* * *

+   2.1 如何阅读本书

+   2.2 本书中断言的使用方式

* * *

本章包含在阅读本书时有用的信息。

### 2.1 如何阅读本书

您可以以两种方式阅读本书：

+   就像一本指南：从头开始阅读并继续阅读。

+   就像一本参考书：只阅读你感兴趣的章节，跳过其余部分。

本书考虑了这两种方式，因此跳过内容不应该是问题。如果在任何时候书中有相关信息，我会指出它。

### 2.2 本书中断言的使用方式

始终假定已经进行了以下导入（类似于在 Node.js REPL 中可用非严格的`assert`）：

```js
import * as assert from 'node:assert/strict';
```

这个模块实现了[*断言*](https://exploringjs.com/impatient-js/ch_assertion-api.html) - 这在本书的示例中经常使用。它们看起来像这样：

```js
// Comparing primitive values:
assert.equal(3 + 4, 7);
assert.equal('abc'.toUpperCase(), 'ABC');

// Comparing objects:
assert.notEqual({prop: 1}, {prop: 1}); // shallow comparison
assert.deepEqual({prop: 1}, {prop: 1}); // deep comparison
assert.notDeepEqual({prop: 1}, {prop: 2}); // deep comparison
```

[评论](https://github.com/rauschma/nodejs-shell-scripting/issues/2)
