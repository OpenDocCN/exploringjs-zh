# 二十六、动态评估代码：eval()，new Function()（高级）

> 原文：[`exploringjs.com/impatient-js/ch_dynamic-code-evaluation.html`](https://exploringjs.com/impatient-js/ch_dynamic-code-evaluation.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   26.1 `eval()`

+   26.2 `new Function()`

+   26.3 Recommendations

* * *

在本章中，我们将看两种动态评估代码的方式：`eval()` 和 `new Function()`。

### 26.1 `eval()`

给定一个包含 JavaScript 代码的字符串 `str`，`eval(str)` 评估该代码并返回结果：

```js
> eval('2 ** 4')
16
```

有两种调用 `eval()` 的方式：

+   *直接*，通过函数调用。然后在其参数中评估代码在当前范围内。

+   *间接地*，而不是通过函数调用。然后在全局范围内评估其代码。

“不通过函数调用”意味着“任何看起来不同于 `eval(···)`”：

+   `eval.call(undefined, '···')`（使用函数的 `.call()` 方法）

+   `eval?.()`（使用可选链）

+   `(0, eval)('···')`（使用逗号运算符）

+   `globalThis.eval('···')`

+   `const e = eval; e('···')`

+   等等。

以下代码说明了区别：

```js
globalThis.myVariable = 'global';
function func() {
 const myVariable = 'local';

 // Direct eval
 assert.equal(eval('myVariable'), 'local');

 // Indirect eval
 assert.equal(eval.call(undefined, 'myVariable'), 'global');
}
```

在全局上下文中评估代码更安全，因为代码访问的内部更少。

### 26.2 `new Function()`

`new Function()` 创建一个函数对象，并按以下方式调用：

```js
const func = new Function('«param_1»', ···, '«param_n»', '«func_body»');
```

前面的语句等同于下一条语句。请注意，`«param_1»` 等不再在字符串文字中。

```js
const func = function («param_1», ···, «param_n») {
 «func_body»
};
```

在下一个示例中，我们首先通过 `new Function()` 创建相同的函数，然后通过函数表达式创建：

```js
const times1 = new Function('a', 'b', 'return a * b');
const times2 = function (a, b) { return a * b };
```

![](img/0ac255e56dc93a43365d8502301c8688.png) **`new Function()` 创建非严格模式函数**

默认情况下，通过 `new Function()` 创建的函数是松散模式。如果我们希望函数体处于严格模式，我们必须手动切换它。

### 26.3 推荐

尽量避免动态评估代码：

+   这是一个安全风险，因为它可能使攻击者能够以您代码的权限执行任意代码。

+   它可能会被关闭 - 例如，在浏览器中，通过[内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)。

通常，JavaScript 是动态的，因此您不需要 `eval()` 或类似的东西。在下面的示例中，我们使用 `eval()`（行 A）可以很好地在没有它的情况下实现（行 B）。

```js
const obj = {a: 1, b: 2};
const propKey = 'b';

assert.equal(eval('obj.' + propKey), 2); // (A)
assert.equal(obj[propKey], 2); // (B)
```

如果您必须动态评估代码：

+   更喜欢 `new Function()` 而不是 `eval()`：它总是在全局上下文中执行其代码，并且函数为评估的代码提供了一个清晰的接口。

+   更喜欢间接的 `eval` 而不是直接的 `eval`：在全局上下文中评估代码更安全。

[评论](https://github.com/rauschma/impatient-js/issues/51)
