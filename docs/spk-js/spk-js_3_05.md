# 第十一章：数字

> 原文：[11. Numbers](https://exploringjs.com/es5/ch11.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 对所有数字都使用单一类型：它将它们全部视为浮点数。但是，如果小数点后没有数字，则不显示小数点：

```js
> 5.000
5
```

在内部，大多数 JavaScript 引擎都会优化并区分浮点数和整数（详情请参见[JavaScript 中的整数](ch11.html#integers "JavaScript 中的整数")）。但这是程序员看不到的东西。

JavaScript 数字是基于 IEEE 浮点算术标准（IEEE 754）的`double`（64 位）值。该标准被许多编程语言使用。

## 数字文字

数字文字可以是整数、浮点数或（整数）十六进制：

```js
> 35  // integer
35
> 3.141  // floating point
3.141
> 0xFF  // hexadecimal
255
```

### 指数

指数`eX`是“乘以 10^X”的缩写：

```js
> 5e2
500
> 5e-2
0.05
> 0.5e2
50
```

### 在文字上调用方法

对于数字文字，访问属性的点必须与小数点区分开。如果要在数字文字`123`上调用`toString()`，则有以下选项：

```js
123..toString()
123 .toString()  // space before the dot
123.0.toString()
(123).toString()
```

## 转换为数字

将值转换为数字的方式如下：

| 值 | 结果 |
| --- | --- |
| `undefined` | `NaN` |
| `null` | `0` |
| 布尔值 | `false` → `0` |
|  | `true` → `1` |
| 数字 | 与输入相同（无需转换）|
| 字符串 | 解析字符串中的数字（忽略前导和尾随空格）；空字符串转换为 0。示例：`'3.141'` → `3.141` |
| 对象 | 调用`ToPrimitive(value, Number)`（参见[算法：ToPrimitive()—将值转换为原始值](ch08.html#toprimitive "算法：ToPrimitive()—将值转换为原始值")）并转换生成的原始值。|

将空字符串转换为数字时，`NaN`可能是更好的结果。选择结果 0 是为了帮助处理空的数字输入字段，符合 1990 年代中期其他编程语言的做法。¹²

### 手动转换为数字

将任何值转换为数字的两种最常见方法是：

| `Number(value)` |（作为函数调用，而不是作为构造函数调用）|
| --- | --- |
| `+value` |  |

我更喜欢`Number()`，因为它更具描述性。以下是一些示例：

```js
> Number('')
0
> Number('123')
123
> Number('\t\v\r12.34\n ')  // ignores leading and trailing whitespace
12.34

> Number(false)
0
> Number(true)
1
```

### parseFloat()

全局函数`parseFloat()`提供了另一种将值转换为数字的方法。但是，`Number()`通常是更好的选择，我们稍后将看到。这段代码：

```js
parseFloat(str)
```

将`str`转换为字符串，修剪前导空格，然后解析最长的浮点数前缀。如果不存在这样的前缀（例如，在空字符串中），则返回`NaN`。

比较`parseFloat()`和`Number()`：

+   将`parseFloat()`应用于非字符串的效率较低，因为它在解析之前将其参数强制转换为字符串。因此，`Number()`转换为实际数字的许多值被`parseFloat()`转换为`NaN`：

    ```js
    > parseFloat(true)  // same as parseFloat('true')
    NaN
    > Number(true)
    1

    > parseFloat(null)  // same as parseFloat('null')
    NaN
    > Number(null)
    0
    ```

+   `parseFloat()`将空字符串解析为`NaN`：

    ```js
    > parseFloat('')
    NaN
    > Number('')
    0
    ```

+   `parseFloat()`解析到最后一个合法字符，这意味着您可能会得到一个您不想要的结果：

    ```js
    > parseFloat('123.45#')
    123.45
    > Number('123.45#')
    NaN
    ```

+   `parseFloat()`忽略前导空格，并在非法字符之前停止（其中包括空格）：

    ```js
    > parseFloat('\t\v\r12.34\n ')
    12.34
    ```

`Number()`忽略前导和尾随空格（但其他非法字符会导致`NaN`）。

## 特殊数字值

JavaScript 有几个特殊的数字值：

+   两个错误值，`NaN`和`Infinity`。

+   两个零值，`+0`和`-0`。JavaScript 有两个零，一个正零和一个负零，因为数字的符号和大小存储在不同的位置。在本书的大部分内容中，我假设只有一个零，并且您几乎从不在 JavaScript 中看到有两个零。

### NaN

错误值`NaN`（“不是一个数字”的缩写）是一个数字值，具有讽刺意味：

```js
> typeof NaN
'number'
```

它是由以下错误产生的：

+   无法解析数字：

    ```js
    > Number('xyz')
    NaN
    > Number(undefined)
    NaN
    ```

+   操作失败：

    ```js
    > Math.acos(2)
    NaN
    > Math.log(-1)
    NaN
    > Math.sqrt(-1)
    NaN
    ```

+   操作数之一是`NaN`（这可以确保在较长的计算过程中发生错误时，您可以在最终结果中看到它）：

    ```js
    > NaN + 3
    NaN
    > 25 / NaN
    NaN
    ```

#### 陷阱：检查值是否为 NaN

`NaN`是唯一不等于自身的值：

```js
> NaN === NaN
false
```

严格相等（`===`）也被`Array.prototype.indexOf`使用。因此，您不能通过该方法在数组中搜索`NaN`：

```js
> [ NaN ].indexOf(NaN)
-1
```

如果要检查值是否为`NaN`，则必须使用全局函数`isNaN()`：

```js
> isNaN(NaN)
true
> isNaN(33)
false
```

但是，`isNaN`不能正确处理非数字，因为它首先将它们转换为数字。该转换可能产生`NaN`，然后该函数错误地返回`true`：

```js
> isNaN('xyz')
true
```

因此，最好将`isNaN`与类型检查结合使用：

```js
function myIsNaN(value) {
    return typeof value === 'number' && isNaN(value);
}
```

或者，您可以检查值是否不等于自身（因为`NaN`是唯一具有此特性的值）。但这不够自解释：

```js
function myIsNaN(value) {
    return value !== value;
}
```

请注意，此行为由 IEEE 754 规定。如第 7.11 节“比较谓词的详细信息”中所述：¹³

> 每个 NaN 都将与任何东西（包括自身）比较无序。

### Infinity

`Infinity`是一个错误值，指示两个问题中的一个：一个数字无法表示，因为其大小太大，或者发生了除以零。

`Infinity`大于任何其他数字（除了`NaN`）。同样，`-Infinity`小于任何其他数字（除了`NaN`）。这使它们在默认值方面非常有用，例如，当您正在寻找最小值或最大值时。

#### 错误：数字的大小太大

一个数字的大小取决于其内部表示（如[数字的内部表示](ch11.html#number_representation "数字的内部表示")中所讨论的），即：

+   尾数（一个二进制数 1.f[1]f[2]...）

+   指数的 2 次幂

指数必须在（不包括）-1023 和 1024 之间。如果指数太小，数字变为 0。如果指数太大，它变为`Infinity`。2¹⁰²³仍然可以表示，但 2¹⁰²⁴不能：

```js
> Math.pow(2, 1023)
8.98846567431158e+307
> Math.pow(2, 1024)
Infinity
```

#### 错误：除以零

除以零会产生`Infinity`作为错误值：

```js
> 3 / 0
Infinity
> 3 / -0
-Infinity
```

#### 使用 Infinity 进行计算

如果您尝试用另一个`Infinity`“中和”一个`Infinity`，则会得到错误结果`NaN`：

```js
> Infinity - Infinity
NaN
> Infinity / Infinity
NaN
```

如果您尝试超出`Infinity`，您仍然会得到`Infinity`：

```js
> Infinity + Infinity
Infinity
> Infinity * Infinity
Infinity
```

#### 检查 Infinity

严格和宽松的相等对`Infinity`也适用：

```js
> var x = Infinity;
> x === Infinity
true
```

此外，全局函数`isFinite()`允许您检查一个值是否是一个实际的数字（既不是无穷大也不是`NaN`）：

```js
> isFinite(5)
true
> isFinite(Infinity)
false
> isFinite(NaN)
false
```

### 两个零

因为 JavaScript 的数字保持大小和符号分开，每个非负数都有一个负数，包括`0`。

这是因为当您以数字的方式表示数字时，它可能变得非常小，以至于无法与 0 区分，因为编码不够精确以表示差异。然后，有符号零允许您记录“从哪个方向”接近零；也就是说，在被视为零之前，数字具有什么符号。维基百科很好地总结了[有符号零](http://en.wikipedia.org/wiki/Signed_zero)的利弊：

> 据称，IEEE 754 中包含有符号零使得在一些关键问题中更容易实现数值精度，特别是在计算复杂的初等函数时。另一方面，有符号零的概念与大多数数学领域（以及大多数数学课程）中的一般假设相矛盾，即负零和零是相同的。允许负零的表示可以成为程序中的错误源，因为软件开发人员没有意识到（或可能忘记了），虽然这两个零表示在数值比较下行为相等，但它们是不同的位模式，并在一些操作中产生不同的结果。

#### 最佳实践：假装只有一个零

JavaScript 竭尽全力隐藏有两个零这一事实。鉴于通常并不重要它们是不同的，建议您配合单个零的幻觉。让我们看看这个幻觉是如何维持的。

在 JavaScript 中，通常写为`0`，这意味着`+0`。但`-0`也显示为简单的`0`。这是您在使用浏览器命令行或 Node.js REPL 时看到的情况：

```js
> -0
0
```

这是因为标准的`toString()`方法将这两个零都转换为相同的`'0'`：

```js
> (-0).toString()
'0'
> (+0).toString()
'0'
```

相等也无法区分零。甚至`===`也不行：

```js
> +0 === -0
true
```

`Array.prototype.indexOf`使用`===`搜索元素，维持了这个幻觉：

```js
> [ -0, +0 ].indexOf(+0)
0
> [ +0, -0 ].indexOf(-0)
0
```

排序运算符也认为这两个零是相等的：

```js
> -0 < +0
false
> +0 < -0
false
```

#### 区分这两个零

*您*如何实际观察到这两个零是不同的？您可以除以零（`-Infinity`和`+Infinity`可以通过`===`进行区分）：

```js
> 3 / -0
-Infinity
> 3 / +0
Infinity
```

通过`Math.pow()`（参见[数值函数](ch21.html#Math.pow "数值函数")）进行除以零的另一种方法：

```js
> Math.pow(-0, -1)
-Infinity
> Math.pow(+0, -1)
Infinity
```

`Math.atan2()`（参见[三角函数](ch21.html#Math.atan2 "Trigonometric Functions"））还显示了这两个零是不同的：

```js
> Math.atan2(-0, -1)
-3.141592653589793
> Math.atan2(+0, -1)
3.141592653589793
```

区分这两个零的规范方法是除以零。因此，用于检测负零的函数如下：

```js
function isNegativeZero(x) {
    return x === 0 && (1/x < 0);
}
```

以下是使用的函数：

```js
> isNegativeZero(0)
false
> isNegativeZero(-0)
true
> isNegativeZero(33)
false
```

## 数字的内部表示

JavaScript 数字具有 64 位精度，也称为*双精度*（某些编程语言中的`double`类型）。内部表示基于 IEEE 754 标准。64 位分布在数字的符号、指数和分数之间，如下所示：

| 符号 | 指数 ∈ [−1023, 1024] | 分数 |
| --- | --- | --- |
| 1 位 | 11 位 | 52 位 |
| 位 63 | 位 62–52 | 位 51–0 |

数字的值由以下公式计算：

(–1)^(sign) × %1.fraction × 2^(exponent)

前缀百分号（`%`）表示中间的数字以二进制表示：1，后跟二进制点，后跟二进制分数，即分数的二进制数字（自然数）。以下是此表示的一些示例：

| +0 |  | （符号：0，小数：0，指数：−1023） |
| --- | --- | --- |
| –0 |  | （符号：1，小数：0，指数：−1023） |
| 1 | = (−1)⁰ × %1.0 × 2⁰ | （符号：0，小数：0，指数：0） |
| 2 | = (−1)⁰ × %1.0 × 2¹ |  |
| 3 | = (−1)⁰ × %1.1 × 2¹ | （符号：0，小数：2⁵¹，指数：0） |
| 0.5 | = (−1)⁰ × %1.0 × 2^(−1) |  |
| −1 | = (−1)¹ × %1.0 × 2⁰ |  |

+0、−0 和 3 的编码可以解释如下：

+   ±0：鉴于分数始终以 1 为前缀，因此无法使用它来表示 0。因此，JavaScript 通过分数 0 和特殊指数−1023 来编码零。符号可以是正数或负数，这意味着 JavaScript 有两个零（参见[两个零](ch11.html#two_zeros "Two Zeros")）。

+   3：位 51 是分数的最高有效位。该位为 1。

### 特殊指数

前面提到的数字表示称为*标准化*。在这种情况下，指数 *e* 在范围内 −1023 < *e* < 1024（不包括下限和上限）。−1023 和 1024 是特殊指数：

+   1024 用于`NaN`和`Infinity`等错误值。

+   −1023 用于：

+   零（如果分数为 0，如刚才解释的那样）

+   靠近零的小数字（如果分数不为 0）。

为了同时启用两个应用程序，使用了不同的所谓*非标准化*表示：

(–1)^(sign) × %0.fraction × 2^(–1022)

要比较，标准化表示中最小（即“最接近零”的）数字是：

(–1)^(sign) × %1.fraction × 2^(–1022)

非标准化的数字更小，因为没有前导数字 1。

## 处理舍入误差

JavaScript 的数字通常以十进制浮点数输入，但在内部表示为二进制浮点数。这导致了不精确。为了理解原因，让我们忘记 JavaScript 的内部存储格式，来看看十进制浮点数和二进制浮点数可以很好地表示哪些分数。在十进制系统中，所有分数都是一个底数 *m* 除以 10 的幂：

![](eq_1101.png)

因此，在分母中只有十。这就是为什么无法将![](inleq_1102.png)精确表示为十进制浮点数的原因——无法将 3 放入分母。二进制浮点数中只有二。让我们看看哪些十进制浮点数可以很好地表示为二进制浮点数，哪些不能。如果分母中只有二，那么可以表示十进制数：

+   0.5[dec] = ![](inleq_1103.png) = ![](inleq_1104.png) = 0.1[bin]

+   0.75[dec] = ![](inleq_1105.png) = ![](inleq_1106.png) = 0.11[bin]

+   0.125[dec] = ![](inleq_1107.png) = ![](inleq_1108.png) = 0.001[bin]

其他分数无法精确表示，因为分母中有 2 以外的数字（经过质因数分解）：

+   0.1[dec] = ![](inleq_1109.png) = ![](inleq_1110.png)

+   0.2[dec] = ![](inleq_1111.png) = ![](inleq_1112.png)

通常看不到 JavaScript 内部并未精确存储 0.1。但是，通过将其乘以足够高的 10 的幂，可以使其可见：

```js
> 0.1 * Math.pow(10, 24)
1.0000000000000001e+23
```

如果将两个不精确表示的数字相加，结果有时会不精确到足以使不精确性变得可见：

```js
> 0.1 + 0.2
0.30000000000000004
```

另一个例子：

```js
> 0.1 + 1 - 1
0.10000000000000009
```

由于舍入误差，最好的做法是不直接比较非整数。而是考虑舍入误差的上限。这样的上限称为[*机器 epsilon*](http://en.wikipedia.org/wiki/Machine_epsilon)。双精度标准 epsilon 值为 2^(−53)：

```js
var EPSILON = Math.pow(2, -53);
function epsEqu(x, y) {
    return Math.abs(x - y) < EPSILON;
}
```

`epsEqu()`确保正确的结果，普通比较会不足以满足要求：

```js
> 0.1 + 0.2 === 0.3
false
> epsEqu(0.1+0.2, 0.3)
true
```

## JavaScript 中的整数

如前所述，JavaScript 只有浮点数。整数在内部以两种方式出现。首先，大多数 JavaScript 引擎将足够小的没有小数部分的数字存储为整数（例如，31 位），并尽可能长时间地保持该表示。如果数字的大小增长太大或出现小数部分，则必须切换回浮点表示。

其次，ECMAScript 规范具有整数运算符：即所有按位运算符。这些运算符将其操作数转换为 32 位整数并返回 32 位整数。对于规范，*整数*只意味着数字没有小数部分，*32 位*意味着它们在某个范围内。对于引擎，*32 位整数*意味着通常可以引入或保持实际整数（非浮点）表示。

### 整数范围

在 JavaScript 中，以下整数范围很重要：

+   安全整数（参见[安全整数](ch11.html#safe_integers "安全整数")），JavaScript 支持的最大实用整数范围：

+   53 位加上一个符号，范围（−2⁵³, 2⁵³）

+   数组索引（参见[数组索引](ch18.html#array_indices "数组索引")）：

+   32 位，无符号

+   最大长度：2³²−1

+   索引范围：[0, 2³²−1)（不包括最大长度！）

+   按位操作数（参见[按位运算符](ch11.html#bitwise_operators "按位运算符")）：

+   无符号右移运算符(`>>>`)：32 位，无符号，范围[0, 2³²)

+   所有其他按位运算符：32 位，包括符号，范围[−2³¹, 2³¹]

+   “字符代码”，UTF-16 代码单元作为数字：

+   被`String.fromCharCode()`接受（参见[字符串构造方法](ch12.html#String.fromCharCode "字符串构造方法")）

+   由`String.prototype.charCodeAt()`返回（参见[提取子字符串](ch12.html#String.prototype.charCodeAt "提取子字符串")）

+   16 位，无符号

### 将整数表示为浮点数

JavaScript 只能处理最大为 53 位的整数值（52 位的小数部分加上 1 个间接位，通过指数; 有关详细信息，请参见[数字的内部表示](ch11.html#number_representation "数字的内部表示")）。

以下表格解释了 JavaScript 如何将 53 位整数表示为浮点数：

| 位 | 范围 | 编码 |
| --- | --- | --- |
| 1 位 | 0 | (参见[数字的内部表示](ch11.html#number_representation "数字的内部表示")) |
| 1 位 | 1 | %1 × 2⁰ |
| 2 位 | 2–3 | %1.f[51] × 2¹ |
| 3 位 | 4–7 = 2²–(2³−1) | %1.f[51]f[50] × 2² |
| 4 位 | 2³–(2⁴−1) | %1.f[51]f[50]f[49] × 2³ |
| ⋯ | ⋯ | ⋯ |
| 53 位 | 2⁵²–(2⁵³−1) | %1.f[51]⋯f[0] × 2⁵² |

没有固定的位序列表示整数。相反，尾数%1.f 被指数移位，以便领先的数字 1 位于正确的位置。在某种程度上，指数计算出分数中活跃使用的数字的数量（其余数字为 0）。这意味着对于 2 位，我们使用分数的一位数字，对于 53 位，我们使用分数的所有数字。此外，我们可以将 2⁵³表示为%1.0 × 2⁵³，但是对于更高的数字，我们会遇到问题：

| 位 | 范围 | 编码 |
| --- | --- | --- |
| 54 位 | 2⁵³–(2⁵⁴−1) | %1.f[51]⋯f[0]0 × 2⁵³ |
| 55 位 | 2⁵⁴–(2⁵⁵−1) | %1.f[51]⋯f[0]00 × 2⁵⁴ |
| ⋯ |  |  |

对于 54 位，最低有效位始终为 0，对于 55 位，最低的两位始终为 0，依此类推。这意味着对于 54 位，我们只能表示每第二个数字，对于 55 位，只能表示每第四个数字，依此类推。例如：

```js
> Math.pow(2, 53) - 1  // OK
9007199254740991
> Math.pow(2, 53)  // OK
9007199254740992
> Math.pow(2, 53) + 1  // can't be represented
9007199254740992
> Math.pow(2, 53) + 2  // OK
9007199254740994
```

#### 最佳实践

如果您使用的整数的大小不超过 53 位，那么就没问题。不幸的是，在编程中经常会遇到 64 位无符号整数（Twitter ID、数据库等）。这些必须以字符串形式存储在 JavaScript 中。如果要对这样的整数执行算术运算，就需要特殊的库。有计划将更大的整数引入 JavaScript，但这需要一些时间。

### 安全整数

JavaScript 只能安全地表示范围在−2⁵³ < i < 2⁵³的整数。本节将探讨这意味着什么以及其后果。它基于 Mark S. Miller 发送给 es-discuss 邮件列表的一封邮件。

安全整数的概念集中在 JavaScript 中如何表示数学整数上。在范围(−2⁵³, 2⁵³)（不包括下限和上限）内，JavaScript 整数是*安全*的：数学整数与它们在 JavaScript 中的表示之间存在一对一的映射。

超出此范围后，JavaScript 整数是*不安全*的：两个或更多数学整数被表示为相同的 JavaScript 整数。例如，从 2⁵³开始，JavaScript 只能表示每第二个数学整数（前一节解释了原因）。因此，安全的 JavaScript 整数是可以明确表示单个数学整数的整数。

#### ECMAScript 6 中的定义

ECMAScript 6 将提供以下常量：

```js
Number.MAX_SAFE_INTEGER = Math.pow(2, 53)-1;
Number.MIN_SAFE_INTEGER = -Number.MAX_SAFE_INTEGER;
```

它还将提供一个用于确定整数是否安全的函数：

```js
Number.isSafeInteger = function (n) {
    return (typeof n === 'number' &&
        Math.round(n) === n &&
        Number.MIN_SAFE_INTEGER <= n &&
        n <= Number.MAX_SAFE_INTEGER);
}
```

对于给定值`n`，此函数首先检查`n`是否为数字和整数。如果两个检查都成功，则如果`n`大于或等于`MIN_SAFE_INTEGER`且小于或等于`MAX_SAFE_INTEGER`，则`n`是安全的。

#### 算术计算的安全结果

我们如何确保算术计算的结果是正确的？例如，以下结果显然是不正确的：

```js
> 9007199254740990 + 3
9007199254740992
```

我们有两个安全的操作数，但是一个不安全的结果：

```js
> Number.isSafeInteger(9007199254740990)
true
> Number.isSafeInteger(3)
true
> Number.isSafeInteger(9007199254740992)
false
```

以下结果也是不正确的：

```js
> 9007199254740995 - 10
9007199254740986
```

这次结果是安全的，但其中一个操作数不是：

```js
> Number.isSafeInteger(9007199254740995)
false
> Number.isSafeInteger(10)
true
> Number.isSafeInteger(9007199254740986)
true
```

因此，只有当所有操作数和结果都是安全的时，才能保证应用整数运算符`op`的结果是正确的。更正式地说：

```js
isSafeInteger(a) && isSafeInteger(b) && isSafeInteger(a op b)
```

意味着`a op b`是正确的结果。

## 转换为整数

在 JavaScript 中，所有数字都是浮点数。整数是没有小数部分的浮点数。将数字`n`转换为整数意味着找到与`n`“最接近”的整数（“最接近”的含义取决于如何进行转换）。您有几种选项可以执行此转换：

1.  `Math`函数`Math.floor()`、`Math.ceil()`和`Math.round()`（参见[Integers via Math.floor(), Math.ceil(), and Math.round()](ch11.html#integers_via_math "Integers via Math.floor(), Math.ceil(), and Math.round()")）

1.  自定义函数`ToInteger()`（参见[Integers via the Custom Function ToInteger()](ch11.html#ToInteger "Integers via the Custom Function ToInteger()")）

1.  二进制位运算符（参见[通过位运算符实现 32 位整数](ch11.html#integers_via_bitwise_operators "通过位运算符实现 32 位整数"））

1.  全局函数`parseInt()`（参见[通过 parseInt()实现整数](ch11.html#parseInt "通过 parseInt()实现整数"））

结论：#1 通常是最佳选择，#2 和#3 有特定应用，#4 适用于解析字符串，但不适用于将数字转换为整数。

### 通过 Math.floor()、Math.ceil()和 Math.round()实现整数

以下三个函数通常是将数字转换为整数的最佳方式：

+   `Math.floor()`将其参数转换为最接近的较低整数：

    ```js
    > Math.floor(3.8)
    3
    > Math.floor(-3.8)
    -4
    ```

+   `Math.ceil()`将其参数转换为最接近的更高整数：

    ```js
    > Math.ceil(3.2)
    4
    > Math.ceil(-3.2)
    -3
    ```

+   `Math.round()`将其参数转换为最接近的整数：

    ```js
    > Math.round(3.2)
    3
    > Math.round(3.5)
    4
    > Math.round(3.8)
    4
    ```

四舍五入`-3.5`的结果可能会让人惊讶：

```js
> Math.round(-3.2)
-3
> Math.round(-3.5)
-3
> Math.round(-3.8)
-4
```

因此，`Math.round(x)`与以下相同：

```js
Math.ceil(x + 0.5)
```

### 通过自定义函数 ToInteger()实现整数

将任何值转换为整数的另一个好选择是内部 ECMAScript 操作`ToInteger()`，它去除了浮点数的小数部分。如果它在 JavaScript 中可用，它将像这样工作：

```js
> ToInteger(3.2)
3
> ToInteger(3.5)
3
> ToInteger(3.8)
3
> ToInteger(-3.2)
-3
> ToInteger(-3.5)
-3
> ToInteger(-3.8)
-3
```

ECMAScript 规范将`ToInteger(number)`的结果定义为：

sign(number) × floor(abs(number))

这个公式相对复杂，因为`floor`寻找最接近的*大*整数；如果你想去掉负整数的小数部分，你必须寻找最接近的小整数。以下代码在 JavaScript 中实现了这个操作。如果数字是负数，我们避免使用`sign`操作，而是使用`ceil`：

```js
function ToInteger(x) {
    x = Number(x);
    return x < 0 ? Math.ceil(x) : Math.floor(x);
}
```

### 通过位运算符实现 32 位整数

二进制位运算符（参见[二进制位运算符](ch11.html#binary_bitwise_operators "二进制位运算符"）将（至少）一个操作数转换为 32 位整数，然后对其进行操作以产生也是 32 位整数的结果。因此，如果你适当选择另一个操作数，你可以快速地将任意数字转换为 32 位整数（有符号或无符号）。

#### 按位或（|)

如果掩码，第二个操作数，为 0，则不改变任何位，结果是第一个操作数，强制转换为有符号 32 位整数。这是执行这种强制转换的规范方式，例如，asm.js（参见[JavaScript 足够快吗？](ch02.html#asm.js "JavaScript 足够快吗？"））中使用：

```js
// Convert x to a signed 32-bit integer
function ToInt32(x) {
    return x | 0;
}
```

`ToInt32()`去除小数并应用模 2³²：

```js
> ToInt32(1.001)
1
> ToInt32(1.999)
1
> ToInt32(1)
1
> ToInt32(-1)
-1
> ToInt32(Math.pow(2, 32)+1)
1
> ToInt32(Math.pow(2, 32)-1)
-1
```

#### 移位运算符

对移位运算符也适用与按位或相同的技巧：如果你移动零位，移位操作的结果是第一个操作数，强制转换为 32 位整数。以下是通过移位运算符实现 ECMAScript 规范操作的一些示例：

```js
// Convert x to a signed 32-bit integer
function ToInt32(x) {
    return x << 0;
}

// Convert x to a signed 32-bit integer
function ToInt32(x) {
    return x >> 0;
}

// Convert x to an unsigned 32-bit integer
function ToUint32(x) {
    return x >>> 0;
}
```

这是`ToUint32()`的实际操作：

```js
> ToUint32(-1)
4294967295
> ToUint32(Math.pow(2, 32)-1)
4294967295
> ToUint32(Math.pow(2, 32))
0
```

#### 我应该使用位运算符强制转换为整数吗？

你必须自己决定，稍微提高效率是否值得让你的代码更难理解。另外要注意，位运算符人为地限制自己在 32 位，这通常既不必要也不实用。使用`Math`函数之一，可能还加上`Math.abs()`，是一个更易于理解且可能更好的选择。

### 通过 parseInt()实现整数

`parseInt()`函数：

```js
parseInt(str, radix?)
```

解析字符串`str`（非字符串被强制转换）为整数。该函数忽略前导空格，并考虑尽可能多的连续合法数字。

#### 基数

基数的范围是 2 ≤ `radix` ≤ 36。它确定要解析的数字的基数。如果基数大于 10，则除了 0-9，还使用字母作为数字（不区分大小写）。

如果`radix`缺失，则假定为 10，除非`str`以`0x`或`0X`开头，此时`radix`设置为 16（十六进制）：

```js
> parseInt('0xA')
10
```

如果`radix`已经是 16，则十六进制前缀是可选的：

```js
> parseInt('0xA', 16)
10
> parseInt('A', 16)
10
```

到目前为止，我已经描述了`parseInt()`的行为，符合 ECMAScript 规范。此外，一些引擎如果`str`以零开头，则将基数设置为 8：

```js
> parseInt('010')
8
> parseInt('0109')  // ignores digits ≥ 8
8
```

因此，最好总是明确指定基数，始终使用两个参数调用`parseInt()`。

以下是一些例子：

```js
> parseInt('')
NaN
> parseInt('zz', 36)
1295
> parseInt('   81', 10)
81

> parseInt('12**', 10)
12
> parseInt('12.34', 10)
12
> parseInt(12.34, 10)
12
```

不要使用`parseInt()`将数字转换为整数。最后一个例子让我们希望我们可以使用`parseInt()`将数字转换为整数。然而，这里有一个转换不正确的例子：

```js
> parseInt(1000000000000000000000.5, 10)
1
```

#### 解释

首先将参数转换为字符串：

```js
> String(1000000000000000000000.5)
'1e+21'
```

`parseInt`不认为`e`是一个整数数字，因此在 1 之后停止解析。这里是另一个例子：

```js
> parseInt(0.0000008, 10)
8
> String(0.0000008)
'8e-7'
```

#### 总结

`parseInt()`不应该用于将数字转换为整数：强制转换为字符串是一个不必要的绕道，即使这样，结果也不总是正确的。

`parseInt()`用于解析字符串很有用，但你必须意识到它会在第一个非法数字处停止。通过`Number()`（参见[函数 Number](ch11.html#function_number "函数 Number"））解析字符串不太宽容，但可能会产生非整数。

## 算术运算符

以下运算符适用于数字：

`number1 + number2`

数值相加，除非其中一个操作数是字符串。然后两个操作数都会被转换为字符串并连接在一起（参见[加号运算符（+）](ch09.html#plus_operator "加号运算符（+）"））：

```js
> 3.1 + 4.3
7.4
> 4 + ' messages'
'4 messages'
```

`number1 - number2`

减法。

`number1 * number2`

乘法。

`number1 / number2`

除法。

`number1 % number2`

余数：

```js
> 9 % 7
2
> -9 % 7
-2
```

### 警告

这个操作不是模运算。它返回一个与第一个操作数相同符号的值（稍后会有更多细节）。

`-number`

否定其参数。

`+number`

将其参数保持不变；非数字被转换为数字。

`++variable`, `--variable`

在增加（或减少）1 之后返回变量的当前值：

```js
> var x = 3;
> ++x
4
> x
4
```

`variable++`, `variable--`

通过 1 来增加（或减少）变量的值并返回它：

```js
> var x = 3;
> x++
3
> x
4
```

### 助记符：增量（++）和减量（--）运算符

操作数的位置可以帮助你记住它是在增加（或减少）之前还是之后返回的。如果操作数在增加运算符之前，它在增加之前返回。如果操作数在运算符之后，它会增加然后返回。（减量运算符的工作方式类似。）

陷阱：余数运算符（%）不是模运算

余数运算符的结果始终具有第一个操作数的符号（对于模运算，它是第二个操作数的符号）：

```js
> -5 % 2
-1
```

这意味着以下函数不起作用：

```js
// Wrong!
function isOdd(n) {
    return n % 2 === 1;
}
console.log(isOdd(-5)); // false
console.log(isOdd(-4)); // false
```

正确的版本是：

```js
function isOdd(n) {
    return Math.abs(n % 2) === 1;
}
console.log(isOdd(-5)); // true
console.log(isOdd(-4)); // false
```

## 位运算符

JavaScript 有几个位运算符，可以处理 32 位整数。也就是说，它们将操作数转换为 32 位整数，并产生一个 32 位整数的结果。这些运算符的用例包括处理二进制协议、特殊算法等。

### 背景知识

本节解释了一些概念，这些概念将帮助你理解位运算符。

#### 二进制补码

计算二进制补码（或反码）的两种常见方法是：

补码

通过反转 32 位数字来计算数字`x`的补码`~x`。让我们通过四位数字来说明补码。`1100`的补码是`0011`。将一个数字加上它的补码会得到一个所有数字都是 1 的数字：

```js
1 + ~1 = 0001 + 1110 = 1111
```

二进制补码

数字`x`的二进制补码`-x`是补码加一。将一个数字加上它的二进制补码会得到`0`（忽略最高位之外的溢出）。以下是一个使用四位数字的例子：

```js
1 + -1 = 0001 + 1111 = 0000
```

#### 有符号的 32 位整数

32 位整数没有显式的符号，但你仍然可以编码负数。例如，-1 可以编码为 1 的补码：将结果加 1 得到 0（在 32 位内）。正数和负数之间的边界是流动的；4294967295（2³²−1）和-1 在这里是相同的整数。但是，当你将这样的整数从 JavaScript 数字转换到 JavaScript 数字时，你必须决定一个符号，这个符号与隐式符号相对。因此，*有符号的 32 位整数*被分成两组：

+   最高位为 0：数字为零或正数。

+   最高位为 1：数字为负数。

最高位通常称为*符号位*。因此，4294967295，解释为有符号 32 位整数，当转换为 JavaScript 数字时变为-1：

```js
> ToInt32(4294967295)
-1
```

`ToInt32()`在[通过按位操作获取 32 位整数](ch11.html#integers_via_bitwise_operators "32-bit Integers via Bitwise Operators")中有解释。

### 注意

只有无符号右移操作符（`>>>`）适用于无符号 32 位整数；所有其他按位操作符适用于有符号 32 位整数。

#### 输入和输出二进制数

在以下示例中，我们通过以下两个操作使用二进制数：

+   `parseInt(str, 2)`（参见[通过 parseInt()获取整数](ch11.html#parseInt "Integers via parseInt()"））解析二进制表示法（基数为 2）的字符串`str`。例如：

    ```js
    > parseInt('110', 2)
    6
    ```

+   `num.toString(2)`（参见[Number.prototype.toString(radix?)](ch11.html#Number.prototype.toString "Number.prototype.toString(radix?)"）将数字`num`转换为二进制表示的字符串。例如：

    ```js
    > 6..toString(2)
    '110'
    ```

### 按位非操作符

`~number`计算`number`的补码：

```js
> (~parseInt('11111111111111111111111111111111', 2)).toString(2)
'0'
```

### 二进制按位操作符

JavaScript 有三个二进制按位操作符：

+   `number1 & number2`（按位与）：

    ```js
    > (parseInt('11001010', 2) & parseInt('1111', 2)).toString(2)
    '1010'
    ```

+   `number1 | number2`（按位或）：

    ```js
    > (parseInt('11001010', 2) | parseInt('1111', 2)).toString(2)
    '11001111'
    ```

+   `number1 ^ number2`（按位异或）：

    ```js
    > (parseInt('11001010', 2) ^ parseInt('1111', 2)).toString(2)
    '11000101'
    ```

直观理解二进制按位操作符有两种方式：

每位一个布尔操作。

在以下公式中，`n[i]`表示将数字`n`的第`i`位解释为布尔值（0 为`false`，1 为`true`）。例如，`2[0]`为`false`；`2[1]`为`true`：

+   And：`result[i] = number1[i] && number2[i]`

+   或：`result[i] = number1[i] || number2[i]`

+   Xor：`result[i] = number1[i] ^^ number2[i]`

操作符`^^`不存在。如果存在，它将按照以下方式工作（如果操作数中恰好有一个为`true`，则结果为`true`）：

```js
x ^^ y === (x && !y) ||(!x && y)
```

通过`number2`改变`number1`的位

+   And：仅保留`number1`中设置的那些位。这个操作也被称为*掩码*，`number2`是*掩码*。

+   或：设置`number1`中设置的所有位，并保持所有其他位不变。

+   Xor：反转`number1`中设置的所有位，并保持所有其他位不变。

### 按位移动操作符

JavaScript 有三个按位移动操作符：

+   `number << digitCount`（左移）：

    ```js
    > (parseInt('1', 2) << 1).toString(2)
    '10'
    ```

+   `number >> digitCount`（有符号右移）：

32 位二进制数被解释为有符号数（参见前面的部分）。向右移动时，符号被保留：

```js
> (parseInt('11111111111111111111111111111110', 2) >> 1).toString(2)
'-1'
```

我们已经右移了-2。结果-1 等同于一个 32 位整数，其所有数字都是 1（1 的补码）。换句话说，向右移动一个数字，负数和正数都会除以 2。

+   `number >>> digitCount`（无符号右移）：

    ```js
    > (parseInt('11100', 2) >>> 1).toString(2)
    '1110'
    ```

正如你所看到的，这个操作符从左边补零。

## 函数数字

`Number`函数可以以两种方式调用：

`Number(value)`

作为普通函数，它将`value`转换为原始数字（参见[转换为数字](ch11.html#tonumber "Converting to Number"））：

```js
> Number('123')
123
> typeof Number(3)  // no change
'number'
```

`new Number(num)`

作为构造函数，它创建一个`Number`的新实例（参见[原始值的包装对象](ch08.html#wrapper_objects "Wrapper Objects for Primitives"）），一个将`num`（在转换为数字后）包装的对象。例如：

```js
> typeof new Number(3)
'object'
```

前一种调用是常见的。

## 数字构造函数属性

对象`Number`具有以下属性：

`Number.MAX_VALUE`

可以表示的最大正数。在内部，其分数的所有数字都是 1，指数是最大的，为 1023。如果尝试通过将指数乘以 2 来增加指数，结果将是错误值`Infinity`（参见[Infinity](ch11.html#infinity "Infinity")）：

```js
> Number.MAX_VALUE
1.7976931348623157e+308
> Number.MAX_VALUE * 2
Infinity
```

`Number.MIN_VALUE`

最小的可表示正数（大于零，一个微小的分数）：

```js
> Number.MIN_VALUE
5e-324
```

`Number.NaN`

与全局`NaN`相同的值。

`Number.NEGATIVE_INFINITY`

与“-无穷大”相同的值：

```js
> Number.NEGATIVE_INFINITY === -Infinity
true
```

`Number.POSITIVE_INFINITY`

与`Infinity`相同的值：

```js
> Number.POSITIVE_INFINITY === Infinity
true
```

## 数字原型方法

所有原始数字的方法都存储在`Number.prototype`中（参见[Primitives Borrow Their Methods from Wrappers](ch08.html#primitive_methods_via_wrappers "Primitives Borrow Their Methods from Wrappers")）。

### Number.prototype.toFixed(fractionDigits?)

`Number.prototype.toFixed(fractionDigits?)`返回一个不带指数的数字表示，四舍五入到`fractionDigits`位。如果省略参数，则使用值 0：

```js
> 0.0000003.toFixed(10)
'0.0000003000'
> 0.0000003.toString()
'3e-7'
```

如果数字大于或等于 10²¹，那么这个方法的工作方式与`toString()`相同。您会得到一个用指数表示的数字：

```js
> 1234567890123456789012..toFixed()
'1.2345678901234568e+21'
> 1234567890123456789012..toString()
'1.2345678901234568e+21'
```

### Number.prototype.toPrecision(precision?)

`Number.prototype.toPrecision(precision?)`在使用类似于`toString()`的转换算法之前，将尾数修剪为`precision`位数字。如果没有给出精度，则直接使用`toString()`：

```js
> 1234..toPrecision(3)
'1.23e+3'

> 1234..toPrecision(4)
'1234'

> 1234..toPrecision(5)
'1234.0'

> 1.234.toPrecision(3)
'1.23'
```

您需要指数表示法来显示 1234，精度为三位。

### Number.prototype.toString(radix?)

对于`Number.prototype.toString(radix?)`，参数`radix`表示要显示数字的系统的基数。最常见的基数是 10（十进制）、2（二进制）和 16（十六进制）：

```js
> 15..toString(2)
'1111'
> 65535..toString(16)
'ffff'
```

基数必须至少为 2，最多为 36。任何大于 10 的基数都会导致字母字符被用作数字，这解释了最大 36，因为拉丁字母表有 26 个字符：

```js
> 1234567890..toString(36)
'kf12oi'
```

全局函数`parseInt`（参见[Integers via parseInt()](ch11.html#parseInt "Integers via parseInt()")）允许您将这些表示法转换回数字：

```js
> parseInt('kf12oi', 36)
1234567890
```

#### 十进制指数表示法

对于基数 10，`toString()`在两种情况下使用指数表示法（小数点前有一个数字）。首先，如果小数点前有超过 21 位数字：

```js
> 1234567890123456789012
1.2345678901234568e+21
> 123456789012345678901
123456789012345680000
```

其次，如果一个数字以`0.`开头，后面跟着超过五个零和一个非零数字：

```js
> 0.0000003
3e-7
> 0.000003
0.000003
```

在所有其他情况下，使用固定表示法。

### Number.prototype.toExponential(fractionDigits?)

`Number.prototype.toExponential(fractionDigits?)`强制一个数字以指数表示。`fractionDigits`是一个介于 0 和 20 之间的数字，用于确定小数点后应显示多少位数字。如果省略，则包括尽可能多的有效数字以唯一指定数字。

在这个例子中，当`toString()`也使用指数表示时，我们强制更多的精度。结果是混合的，因为当将二进制数字转换为十进制表示时，我们达到了可以实现的精度限制：

```js
> 1234567890123456789012..toString()
'1.2345678901234568e+21'

> 1234567890123456789012..toExponential(20)
'1.23456789012345677414e+21'
```

在这个例子中，数字的数量级不够大，无法通过`toString()`显示指数。然而，`toExponential()`确实显示了一个指数：

```js
> 1234..toString()
'1234'

> 1234..toExponential(5)
'1.23400e+3'

> 1234..toExponential()
'1.234e+3'
```

在这个例子中，当分数不够小时，我们得到指数表示法：

```js
> 0.003.toString()
'0.003'

> 0.003.toExponential(4)
'3.0000e-3'

> 0.003.toExponential()
'3e-3'
```

## 数字函数

以下函数操作数字：

`isFinite(number)`

检查`number`是否是实际数字（既不是`Infinity`也不是`NaN`）。详情请参见[Checking for Infinity](ch11.html#isFinite "Checking for Infinity")。

`isNaN(number)`

如果`number`是`NaN`，则返回`true`。详情请参见[Pitfall: checking whether a value is NaN](ch11.html#isNaN "Pitfall: checking whether a value is NaN")。

`parseFloat(str)`

将`str`转换为浮点数。详情请参见[parseFloat()](ch11.html#parseFloat "parseFloat()")。

`parseInt(str, radix?)`

将`str`解析为以`radix`为基数的整数（2-36）。详情请参阅[通过 parseInt()获取整数](ch11.html#parseInt "通过 parseInt()获取整数")。

## 本章的来源

在编写本章时，我参考了以下来源：

+   [“IEEE 标准 754 浮点数”](http://bit.ly/1oOc43P) 由 Steve Hollasch

+   [“数据类型和缩放（固定点块集）”](http://bit.ly/1oOc83t) 在 MATLAB 文档中

+   [“IEEE 浮点”](http://en.wikipedia.org/wiki/IEEE_754) 在维基百科上

* * *

¹² 来源：Brendan Eich，[`bit.ly/1lKzQeC`](http://bit.ly/1lKzQeC)。

¹³ Béla Varga（@netzzwerg）指出 IEEE 754 规定 NaN 不等于自身。

