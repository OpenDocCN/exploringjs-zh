# 15 布尔值

> 原文：[`exploringjs.com/impatient-js/ch_booleans.html`](https://exploringjs.com/impatient-js/ch_booleans.html)

* * *

+   15.1 转换为布尔值

+   15.2 假值和真值

    +   15.2.1 检查真值或假值

+   15.3 基于真值的存在性检查

    +   15.3.1 陷阱：基于真值的存在性检查不精确

    +   15.3.2 用例：是否提供了参数？

    +   15.3.3 用例：属性是否存在？

+   15.4 条件运算符（`? :`）

+   15.5 二进制逻辑运算符：And（`x && y`），Or（`x || y`）

    +   15.5.1 值保留

    +   15.5.2 短路

    +   15.5.3 逻辑与（`x && y`）

    +   15.5.4 逻辑或（`||`）

+   15.6 逻辑非（`!`）

* * *

原始类型*boolean*包括两个值 - `false` 和 `true`：

```js
> typeof false
'boolean'
> typeof true
'boolean'
```

### 15.1 转换为布尔值

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **“转换为[type]” 的含义**

“转换为[type]”是“将任意值转换为[type]类型的值”的简称。

有三种方法可以将任意值`x`转换为布尔值。

+   `Boolean(x)`

    最具描述性；推荐使用。

+   `x ? true : false`

    使用条件运算符（在本章后面解释）。

+   `!!x`

    使用逻辑非运算符（`!`）。此运算符将其操作数强制转换为布尔值。然后再次应用它以获得非否定的结果。

Tbl. 4 描述了各种值如何转换为布尔值。

表 4：将值转换为布尔值。

| `x` | `Boolean(x)` |
| --- | --- |
| `undefined` | `false` |
| `null` | `false` |
| boolean | `x` (no change) |
| number | `0` `→` `false`, `NaN` `→` `false` |
|  | 其他数字 `→` `true` |
| bigint | `0` `→` `false` |
|  | 其他数字 `→` `true` |
| string | `''` `→` `false` |
|  | 其他字符串 `→` `true` |
| symbol | `true` |
| object | 始终为 `true` |

### 15.2 假值和真值

在检查`if`语句、`while`循环或`do-while`循环的条件时，JavaScript 的工作方式与您可能期望的不同。例如，考虑以下条件：

```js
if (value) {}
```

在许多编程语言中，此条件等同于：

```js
if (value === true) {}
```

然而，在 JavaScript 中，它等同于：

```js
if (Boolean(value) === true) {}
```

也就是说，JavaScript 检查将`value`转换为布尔值时是否为`true`。这种检查非常常见，因此引入了以下名称：

+   如果将值转换为布尔值时为`true`，则称该值为*真值*。

+   如果将值转换为布尔值时为`false`，则称该值为*假值*。

每个值都是真值或假值。通过查阅表 4，我们可以列出所有假值的详尽列表：

+   `undefined`

+   `null`

+   布尔值：`false`

+   数字：`0`，`NaN`

+   Bigint：`0n`

+   字符串：`''`

所有其他值（包括所有对象）都是真值：

```js
> Boolean('abc')
true
> Boolean([])
true
> Boolean({})
true
```

#### 15.2.1 检查真值或假值

```js
if (x) {
 // x is truthy
}

if (!x) {
 // x is falsy
}

if (x) {
 // x is truthy
} else {
 // x is falsy
}

const result = x ? 'truthy' : 'falsy';
```

在最后一行使用的条件运算符在本章后面有解释。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：真值**

`exercises/booleans/truthiness_exrc.mjs`

### 15.3 基于真值的存在性检查

在 JavaScript 中，如果你读取一个不存在的东西（例如，一个缺失的参数或一个缺失的属性），通常会得到`undefined`作为结果。在这些情况下，存在性检查等同于将一个值与`undefined`进行比较。例如，以下代码检查对象`obj`是否具有属性`.prop`：

```js
if (obj.prop !== undefined) {
 // obj has property .prop
}
```

由于`undefined`是假值，我们可以将这个检查缩短为：

```js
if (obj.prop) {
 // obj has property .prop
}
```

#### 15.3.1 陷阱：基于真实性的存在性检查不够精确

基于真实性的存在性检查有一个陷阱：它们不是很精确。考虑这个之前的例子：

```js
if (obj.prop) {
 // obj has property .prop
}
```

如果：

+   `obj.prop`是丢失的（在这种情况下，JavaScript 返回`undefined`）。

然而，如果：

+   `obj.prop`是`undefined`。

+   `obj.prop`是任何其他假值（`null`，`0`，`''`，等等）。

实际上，这很少会引起问题，但你必须意识到这个陷阱。

#### 15.3.2 用例：是否提供了参数？

真实性检查经常用于确定函数的调用者是否提供了参数：

```js
function func(x) {
 if (!x) {
 throw new Error('Missing parameter x');
 }
 // ···
}
```

好的一面是，这种模式已经被建立并且很简短。它可以正确地抛出`undefined`和`null`的错误。

不好的一面是，之前提到的陷阱：代码也会对所有其他假值抛出错误。

另一种方法是检查`undefined`：

```js
if (x === undefined) {
 throw new Error('Missing parameter x');
}
```

#### 15.3.3 用例：属性是否存在？

真实性检查也经常用于确定属性是否存在：

```js
function readFile(fileDesc) {
 if (!fileDesc.path) {
 throw new Error('Missing property: .path');
 }
 // ···
}
readFile({ path: 'foo.txt' }); // no error
```

这种模式也已经建立，并且有一个通常的警告：它不仅在属性丢失时抛出错误，而且在属性存在并且具有任何假值时也会抛出错误。

如果你真的想检查属性是否存在，你必须使用 in 运算符：

```js
if (! ('path' in fileDesc)) {
 throw new Error('Missing property: .path');
}
```

### 15.4 条件运算符（`? :`）

条件运算符是`if`语句的表达式版本。它的语法是：

```js
«condition» ? «thenExpression» : «elseExpression»
```

它的评估如下：

+   如果`condition`为真值，则评估并返回`thenExpression`。

+   否则，评估并返回`elseExpression`。

条件运算符也被称为*三元运算符*，因为它有三个操作数。

例子：

```js
> true ? 'yes' : 'no'
'yes'
> false ? 'yes' : 'no'
'no'
> '' ? 'yes' : 'no'
'no'
```

以下代码演示了通过条件选择的两个分支“then”和“else”，只有一个分支被评估。另一个分支不会被评估。

```js
const x = (true ? console.log('then') : console.log('else'));

// Output:
// 'then'
```

### 15.5 二进制逻辑运算符：与（`x && y`），或（`x || y`）

二进制逻辑运算符`&&`和`||`是*值保留*和*短路*的。

#### 15.5.1 值保留

*值保留*意味着操作数被解释为布尔值，但返回不变：

```js
> 12 || 'hello'
12
> 0 || 'hello'
'hello'
```

#### 15.5.2 短路

*短路*意味着如果第一个操作数已经确定了结果，那么第二个操作数就不会被评估。唯一延迟评估其操作数的其他运算符是条件运算符。通常，在执行操作之前会评估所有操作数。

例如，逻辑与（`&&`）如果第一个操作数为假，则不评估第二个操作数：

```js
const x = false && console.log('hello');
// No output
```

如果第一个操作数为真值，则执行`console.log()`：

```js
const x = true && console.log('hello');

// Output:
// 'hello'
```

#### 15.5.3 逻辑与（`x && y`）

表达式`a && b`（“`a`和`b`”）的评估如下：

1.  评估`a`。

1.  结果是否为假？返回它。

1.  否则，评估`b`并返回结果。

换句话说，以下两个表达式大致等价：

```js
a && b
!a ? a : b
```

例子：

```js
> false && true
false
> false && 'abc'
false

> true && false
false
> true && 'abc'
'abc'

> '' && 'abc'
''
```

#### 15.5.4 逻辑或（`||`）

表达式`a || b`（“`a`或`b`”）的评估如下：

1.  评估`a`。

1.  结果是否为真值？返回它。

1.  否则，评估`b`并返回结果。

换句话说，以下两个表达式大致等价：

```js
a || b
a ? a : b
```

例子：

```js
> true || false
true
> true || 'abc'
true

> false || true
true
> false || 'abc'
'abc'

> 'abc' || 'def'
'abc'
```

##### 15.5.4.1 逻辑或（`||`）的传统用例：提供默认值

ECMAScript 2020 引入了空值合并运算符（`??`）用于默认值。在此之前，逻辑或被用于此目的：

```js
const valueToUse = receivedValue || defaultValue;
```

有关`??`和在这种情况下`||`的缺点的更多信息，请参见[§14.4“空值合并运算符（`??`）用于默认值[ES2020]”](ch_undefined-null.html#nullish-coalescing-operator)。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **传统练习：通过或运算符（`||`）提供默认值**

`exercises/booleans/default_via_or_exrc.mjs`

### 15.6 逻辑非 (`!`)

表达式`!x`（“非`x`”）的求值如下：

1.  求值`x`。

1.  它是真值吗？返回`false`。

1.  否则，返回`true`。

例子：

```js
> !false
true
> !true
false

> !0
true
> !123
false

> !''
true
> !'abc'
false
```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **Quiz**

参见 quiz app。

[Comments](https://github.com/rauschma/impatient-js/issues/10)
