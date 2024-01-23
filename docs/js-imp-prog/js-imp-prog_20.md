# 十六、数字

> 原文：[`exploringjs.com/impatient-js/ch_numbers.html`](https://exploringjs.com/impatient-js/ch_numbers.html)

* * *

+   16.1 数字既用于浮点数也用于整数

+   16.2 数字文字

    +   16.2.1 整数文字

    +   16.2.2 浮点数文字

    +   16.2.3 语法陷阱：整数字面值的属性

    +   16.2.4 [数字文字中的下划线 (`_`) 作为分隔符 [ES2021]](ch_numbers.html#numeric-separator-number-literals)

+   16.3 算术运算符

    +   16.3.1 二进制算术运算符

    +   16.3.2 一元加号 (`+`) 和否定 (`-`)

    +   16.3.3 递增 (`++`) 和递减 (`--`)

+   16.4 转换为数字

+   16.5 错误值

    +   16.5.1 错误值：`NaN`

    +   16.5.2 错误值：`Infinity`

+   16.6 数字的精度：小心处理小数部分

+   16.7 （高级）

+   16.8 背景：浮点数精度

    +   16.8.1 浮点数的简化表示

+   16.9 JavaScript 中的整数

    +   16.9.1 转换为整数

    +   16.9.2 JavaScript 中整数的范围

    +   16.9.3 安全整数

+   16.10 位运算符

    +   16.10.1 在内部，按位运算符使用 32 位整数

    +   16.10.2 按位取反

    +   16.10.3 二进制位运算符

    +   16.10.4 位移运算符

    +   16.10.5 `b32()`: 以二进制表示无符号 32 位整数

+   16.11 快速参考：数字

    +   16.11.1 数字的全局函数

    +   16.11.2 `Number` 的静态属性

    +   16.11.3 `Number` 的静态方法

    +   16.11.4 `Number.prototype` 的方法

    +   16.11.5 来源

* * *

JavaScript 有两种数值类型：

+   *Numbers* 是 64 位浮点数，也用于较小的整数（范围在正负 53 位之内）。

+   *Bigints* 代表具有任意精度的整数。

本章涵盖了数字。大整数将在本书的后面进行介绍。

### 16.1 数字既用于浮点数也用于整数

在 JavaScript 中，类型 `number` 用于整数和浮点数：

```js
98
123.45
```

然而，所有数字都是*双精度*，64 位浮点数，根据*IEEE 浮点算术标准*（IEEE 754）实现。

整数就是没有小数部分的浮点数：

```js
> 98 === 98.0
true
```

请注意，在底层，大多数 JavaScript 引擎通常能够使用真正的整数，带有所有相关的性能和存储大小优势。

### 16.2 数字文字

让我们来看看数字的文字。

#### 16.2.1 整数字面值

几个*整数字面值*让我们可以用不同的基数表示整数：

```js
// Binary (base 2)
assert.equal(0b11, 3); // ES6

// Octal (base 8)
assert.equal(0o10, 8); // ES6

// Decimal (base 10)
assert.equal(35, 35);

// Hexadecimal (base 16)
assert.equal(0xE7, 231);
```

#### 16.2.2 浮点数文字

浮点数只能用十进制表示。

分数：

```js
> 35.0
35
```

指数：`eN` 表示 ×10^N

```js
> 3e2
300
> 3e-2
0.03
> 0.3e2
30
```

#### 16.2.3 语法陷阱：整数字面值的属性

访问整数字面值的属性会导致一个陷阱：如果整数字面值紧接着一个点，那么该点会被解释为十进制点：

```js
7.toString(); // syntax error
```

有四种方法可以避开这个陷阱：

```js
7.0.toString()
(7).toString()
7..toString()
7 .toString()  // space before dot
```

#### 16.2.4 下划线 (`_`) 作为数字字面值中的分隔符 [ES2021]

将数字分组以使长数字更易读具有悠久的传统。例如：

+   1825 年，伦敦有 1,335,000 名居民。

+   地球和太阳之间的距离是 149,600,000 公里。

自 ES2021 以来，我们可以在数字字面值中使用下划线作为分隔符：

```js
const inhabitantsOfLondon = 1_335_000;
const distanceEarthSunInKm = 149_600_000;
```

对于其他进制，分组也很重要：

```js
const fileSystemPermission = 0b111_111_000;
const bytes = 0b1111_10101011_11110000_00001101;
const words = 0xFAB_F00D;
```

我们还可以在分数和指数中使用分隔符：

```js
const massOfElectronInKg = 9.109_383_56e-31;
const trillionInShortScale = 1e1_2;
```

##### 16.2.4.1 我们可以在哪里放置分隔符？

分隔符的位置受到两种限制：

+   我们只能在两个数字之间放下划线。因此，以下所有数字字面值都是非法的：

    ```js
    3_.141
    3._141

    1_e12
    1e_12

    _1464301  // valid variable name!
    1464301_

    0_b111111000
    0b_111111000
    ```

+   我们不能连续使用多个下划线：

    ```js
    123__456 // two underscores – not allowed
    ```

这些限制背后的动机是保持解析简单，避免奇怪的边缘情况。

##### 16.2.4.2 使用分隔符解析数字

以下用于解析数字的函数不支持分隔符：

+   `Number()`

+   `Number.parseInt()`

+   `Number.parseFloat()`

例如：

```js
> Number('123_456')
NaN
> Number.parseInt('123_456')
123
```

其理念是数字分隔符是为了代码。其他类型的输入应该以不同的方式处理。

### 16.3 算术运算符

#### 16.3.1 二进制算术运算符

Tbl. 5 列出了 JavaScript 的二进制算术运算符。

表 5：二进制算术运算符。

| 运算符 | 名称 |  | 例子 |
| --- | --- | --- | --- |
| `n + m` | 加法 | ES1 | `3 + 4` `→` `7` |
| `n - m` | 减法 | ES1 | `9 - 1` `→` `8` |
| `n * m` | 乘法 | ES1 | `3 * 2.25` `→` `6.75` |
| `n / m` | 除法 | ES1 | `5.625 / 5` `→` `1.125` |
| `n % m` | 余数 | ES1 | `8 % 5` `→` `3` |
|  |  |  | `-8 % 5` `→` `-3` |
| `n ** m` | 指数 | ES2016 | `4 ** 2` `→` `16` |

##### 16.3.1.1 `%` 是余数运算符

`%` 是一个余数运算符，而不是模运算符。其结果具有第一个操作数的符号：

```js
> 5 % 3
2
> -5 % 3
-2
```

有关余数和模运算之间的区别的更多信息，请参阅博文[“余数运算符 vs. 模运算符（带有 JavaScript 代码）”](https://2ality.com/2019/08/remainder-vs-modulo.html) 在 2ality 上。

#### 16.3.2 一元加 (`+`) 和否定 (`-`)

Tbl. 6 总结了两个运算符 *一元加* (`+`) 和 *否定* (`-`)。

表 6：一元加 (`+`) 和否定 (`-`) 运算符。

| 运算符 | 名称 |  | 例子 |
| --- | --- | --- | --- |
| `+n` | 一元加 | ES1 | `+(-7)` `→` `-7` |
| `-n` | 一元否定 | ES1 | `-(-7)` `→` `7` |

这两个运算符都会将它们的操作数强制转换为数字：

```js
> +'5'
5
> +'-12'
-12
> -'9'
-9
```

因此，一元加让我们将任意值转换为数字。

#### 16.3.3 递增 (`++`) 和递减 (`--`)

递增运算符 `++` 存在前缀版本和后缀版本。在两个版本中，它都会破坏性地将其操作数加一。因此，它的操作数必须是可以更改的存储位置。

递减运算符 `--` 的工作方式相同，但是从其操作数中减去一个。下面的两个例子解释了前缀和后缀版本之间的区别。

Tbl. 7 总结了递增和递减运算符。

表 7：递增运算符和递减运算符。

| 运算符 | 名称 |  | 例子 |
| --- | --- | --- | --- |
| `v++` | 递增 | ES1 | `let v=0; [v++, v]` `→` `[0, 1]` |
| `++v` | 递增 | ES1 | `let v=0; [++v, v]` `→` `[1, 1]` |
| `v--` | 递减 | ES1 | `let v=1; [v--, v]` `→` `[1, 0]` |
| `--v` | 递减 | ES1 | `let v=1; [--v, v]` `→` `[0, 0]` |

接下来，我们将看一些使用这些运算符的例子。

前缀 `++` 和前缀 `--` 改变它们的操作数，然后返回它们。

```js
let foo = 3;
assert.equal(++foo, 4);
assert.equal(foo, 4);

let bar = 3;
assert.equal(--bar, 2);
assert.equal(bar, 2);
```

后缀 `++` 和后缀 `--` 返回它们的操作数，然后改变它们。

```js
let foo = 3;
assert.equal(foo++, 3);
assert.equal(foo, 4);

let bar = 3;
assert.equal(bar--, 3);
assert.equal(bar, 2);
```

##### 16.3.3.1 操作数：不仅仅是变量

我们还可以将这些运算符应用于属性值：

```js
const obj = { a: 1 };
++obj.a;
assert.equal(obj.a, 2);
```

以及数组元素：

```js
const arr = [ 4 ];
arr[0]++;
assert.deepEqual(arr, [5]);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：数字运算符**

`exercises/numbers-math/is_odd_test.mjs`

### 16.4 转换为数字

这些是将值转换为数字的三种方法：

+   `Number(value)`

+   `+value`

+   `parseFloat(value)`（避免；与其他两种方法不同！）

建议：使用描述性的`Number()`。Tbl. 8 总结了它的工作原理。

表 8：将值转换为数字。

| `x` | `Number(x)` |
| --- | --- |
| `未定义` | `NaN` |
| `null` | `0` |
| 布尔值 | `false` `→` `0`，`true` `→` `1` |
| 数字 | `x`（无变化） |
| 大整数 | `-1n` `→` `-1`，`1n` `→` `1`，等等。 |
| 字符串 | `''` `→` `0` |
|  | 其他`→`解析的数字，忽略前导/尾随空格 |
| 符号 | 抛出`TypeError` |
| 对象 | 可配置的（例如通过`.valueOf()`） |

例子：

```js
assert.equal(Number(123.45), 123.45);

assert.equal(Number(''), 0);
assert.equal(Number('\n 123.45 \t'), 123.45);
assert.equal(Number('xyz'), NaN);

assert.equal(Number(-123n), -123);
```

对象如何转换为数字可以进行配置-例如，通过覆盖`.valueOf()`：

```js
> Number({ valueOf() { return 123 } })
123
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：转换为数字**

`exercises/numbers-math/parse_number_test.mjs`

### 16.5 错误值

当发生错误时，会返回两个数字值：

+   `NaN`

+   `Infinity`

#### 16.5.1 错误值：`NaN`

`NaN`是“不是一个数字”的缩写。讽刺的是，JavaScript 认为它是一个数字：

```js
> typeof NaN
'number'
```

何时返回`NaN`？

如果无法解析数字，则返回`NaN`：

```js
> Number('$$$')
NaN
> Number(undefined)
NaN
```

如果无法执行操作，则返回`NaN`：

```js
> Math.log(-1)
NaN
> Math.sqrt(-1)
NaN
```

如果操作数或参数是`NaN`，则返回`NaN`（以传播错误）：

```js
> NaN - 3
NaN
> 7 ** NaN
NaN
```

##### 16.5.1.1 检查`NaN`

`NaN`是唯一一个不严格等于自身的 JavaScript 值：

```js
const n = NaN;
assert.equal(n === n, false);
```

这是检查值`x`是否为`NaN`的几种方法：

```js
const x = NaN;

assert.equal(Number.isNaN(x), true); // preferred
assert.equal(Object.is(x, NaN), true);
assert.equal(x !== x, true);
```

在最后一行，我们使用比较技巧来检测`NaN`。

##### 16.5.1.2 在数组中查找`NaN`

一些数组方法无法找到`NaN`：

```js
> [NaN].indexOf(NaN)
-1
```

其他可以：

```js
> [NaN].includes(NaN)
true
> [NaN].findIndex(x => Number.isNaN(x))
0
> [NaN].find(x => Number.isNaN(x))
NaN
```

遗憾的是，没有简单的经验法则。我们必须检查每种方法如何处理`NaN`。

#### 16.5.2 错误值：`Infinity`

何时返回错误值`Infinity`？

如果数字太大，则返回无穷大：

```js
> Math.pow(2, 1023)
8.98846567431158e+307
> Math.pow(2, 1024)
Infinity
```

如果除以零，则返回无穷大：

```js
> 5 / 0
Infinity
> -5 / 0
-Infinity
```

##### 16.5.2.1 `Infinity`作为默认值

`Infinity`大于所有其他数字（除了`NaN`），使其成为一个很好的默认值：

```js
function findMinimum(numbers) {
 let min = Infinity;
 for (const n of numbers) {
 if (n < min) min = n;
 }
 return min;
}

assert.equal(findMinimum([5, -1, 2]), -1);
assert.equal(findMinimum([]), Infinity);
```

##### 16.5.2.2 检查`Infinity`

这是检查值`x`是否为`Infinity`的两种常见方法：

```js
const x = Infinity;

assert.equal(x === Infinity, true);
assert.equal(Number.isFinite(x), false);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：比较数字**

`exercises/numbers-math/find_max_test.mjs`

### 16.6 数字的精度：小心处理小数

在内部，JavaScript 浮点数使用基数 2 来表示（根据 IEEE 754 标准）。这意味着十进制小数（基数 10）不能总是精确表示：

```js
> 0.1 + 0.2
0.30000000000000004
> 1.3 * 3
3.9000000000000004
> 1.4 * 100000000000000
139999999999999.98
```

因此，在 JavaScript 中进行算术运算时，我们需要考虑舍入误差。

继续阅读对这一现象的解释。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验：基础**

请参阅测验应用程序的解释。

### 16.7 （高级）

本章的所有其余部分都是高级的。

### 16.8 背景：浮点精度

在 JavaScript 中，使用数字进行计算并不总是产生正确的结果-例如：

```js
> 0.1 + 0.2
0.30000000000000004
```

要理解为什么，我们需要探索 JavaScript 如何在内部表示浮点数。它使用三个整数来表示，总共占用 64 位存储空间（双精度）：

| 组件 | 大小 | 整数范围 |
| --- | --- | --- |
| 符号 | 1 位 | [0, 1] |
| 分数 | 52 位 | [0, 2⁵²−1] |
| 指数 | 11 位 | [−1023, 1024] |

由这些整数表示的浮点数计算如下：

> （-1）^（符号）× 0b1.fraction × 2^(exponent)

这种表示无法编码零，因为它的第二个组件（涉及分数）总是有一个前导 1。因此，零通过特殊指数-1023 和分数 0 来编码。

#### 16.8.1 浮点数的简化表示

为了使进一步讨论更容易，我们简化了先前的表示：

+   我们使用基数 10（十进制）而不是基数 2（二进制），因为大多数人更熟悉十进制。

+   *分数*是一个被解释为分数的自然数（小数点后的数字）。我们切换到*尾数*，一个被解释为自身的整数。因此，指数的使用方式不同，但其基本作用并未改变。

+   由于尾数是一个整数（带有自己的符号），我们不再需要单独的符号。

新的表示方法如下：

> 尾数 × 10^(指数)

让我们尝试一下这种表示方法来表示一些浮点数。

+   对于整数−123，我们主要需要尾数：

    ```js
    > -123 * (10 ** 0)
    -123
    ```

+   对于数字 1.5，我们想象尾数后有一个点。我们使用负指数将该点向左移动一位：

    ```js
    > 15 * (10 ** -1)
    1.5
    ```

+   对于数字 0.25，我们将小数点向左移动两位：

    ```js
    > 25 * (10 ** -2)
    0.25
    ```

具有负指数的表示也可以写成分母中具有正指数的分数：

```js
> 15 * (10 ** -1) === 15 / (10 ** 1)
true
> 25 * (10 ** -2) === 25 / (10 ** 2)
true
```

这些分数有助于理解为什么有些数字我们的编码无法表示：

+   `1/10` 可以表示。它已经具有所需的格式：分母中的 10 的幂。

+   `1/2` 可以表示为`5/10`。我们通过将分子和分母乘以 5，将分母中的 2 转换为 10 的幂。

+   `1/4` 可以表示为`25/100`。我们通过将分子和分母乘以 25，将分母中的 4 转换为 10 的幂。

+   `1/3` 无法表示。没有办法将分母转换为 10 的幂。（10 的质因数是 2 和 5。因此，任何只有这些质因数的分母都可以通过乘以足够多的 2 和 5 来转换为 10 的幂。如果分母有其他质因数，那么我们就无能为力了。）

为了结束我们的探讨，我们再次切换到基数 2：

+   `0.5 = 1/2` 可以用基数 2 表示，因为分母已经是 2 的幂。

+   `0.25 = 1/4` 可以用基数 2 表示，因为分母已经是 2 的幂。

+   `0.1 = 1/10` 无法表示，因为分母无法转换为 2 的幂。

+   `0.2 = 2/10` 无法表示，因为分母无法转换为 2 的幂。

现在我们可以看到为什么`0.1 + 0.2`不能产生正确的结果：在内部，这两个操作数都无法精确表示。

精确计算小数部分的唯一方法是在内部切换到基数 10。对于许多编程语言，基数 2 是默认值，基数 10 是一个选项。例如，Java 有类[`BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html)，Python 有模块[`decimal`](https://docs.python.org/3/library/decimal.html)。有计划向 JavaScript 添加类似的功能：[ECMAScript 提案“Decimal”](https://github.com/tc39/proposal-decimal)。

### 16.9 JavaScript 中的整数

整数是没有小数部分的正常（浮点）数：

```js
> 1 === 1.0
true
> Number.isInteger(1.0)
true
```

在本节中，我们将看一些处理这些伪整数的工具。JavaScript 还支持*bigints*，这些是真正的整数。

#### 16.9.1 转换为整数

将数字转换为整数的推荐方法是使用`Math`对象的其中一种四舍五入方法：

+   `Math.floor(n)`: 返回最大的整数`i` ≤ `n`

    ```js
    > Math.floor(2.1)
    2
    > Math.floor(2.9)
    2
    ```

+   `Math.ceil(n)`: 返回最小的整数`i` ≥ `n`

    ```js
    > Math.ceil(2.1)
    3
    > Math.ceil(2.9)
    3
    ```

+   `Math.round(n)`: 返回与`n`“最接近”的整数，其中`__.5`四舍五入为上述整数，例如：

    ```js
    > Math.round(2.4)
    2
    > Math.round(2.5)
    3
    ```

+   `Math.trunc(n)`: 移除`n`的任何小数部分（小数点后），因此将其转换为整数。

    ```js
    > Math.trunc(2.1)
    2
    > Math.trunc(2.9)
    2
    ```

有关四舍五入的更多信息，请参阅§17.3 “Rounding”。

#### 16.9.2 JavaScript 中整数的范围

这些是 JavaScript 中整数的重要范围：

+   **安全整数：**可以被 JavaScript“安全”表示（在下一小节中会详细介绍）

    +   精度：53 位加符号

    +   范围：（−2⁵³, 2⁵³）

+   **数组索引**

    +   精度：32 位，无符号

    +   范围：[0, 2³²−1)（不包括最大长度）

    +   类型化数组具有 53 位的更大范围（安全且无符号）

+   **按位运算符**（按位或等）

    +   精度：32 位

    +   无符号右移（`>>>`）的范围：无符号，[0, 2³²)

    +   所有其他按位运算符的范围：有符号，[−2³¹, 2³¹)

#### 16.9.3 安全整数

这是 JavaScript 中*安全*的整数范围（53 位加上符号）：

> [–(2⁵³)+1, 2⁵³–1]

如果一个整数由一个 JavaScript 数字精确表示，则该整数是*安全*的。鉴于 JavaScript 数字被编码为乘以 2 的指数幂的分数，更高的整数也可以表示，但它们之间存在间隙。

例如（18014398509481984 是 2⁵⁴）：

```js
> 18014398509481984
18014398509481984
> 18014398509481985
18014398509481984
> 18014398509481986
18014398509481984
> 18014398509481987
18014398509481988
```

`Number`的以下属性有助于确定整数是否安全：

```js
assert.equal(Number.MAX_SAFE_INTEGER, (2 ** 53) - 1);
assert.equal(Number.MIN_SAFE_INTEGER, -Number.MAX_SAFE_INTEGER);

assert.equal(Number.isSafeInteger(5), true);
assert.equal(Number.isSafeInteger('5'), false);
assert.equal(Number.isSafeInteger(5.1), false);
assert.equal(Number.isSafeInteger(Number.MAX_SAFE_INTEGER), true);
assert.equal(Number.isSafeInteger(Number.MAX_SAFE_INTEGER+1), false);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：检测安全整数**

`exercises/numbers-math/is_safe_integer_test.mjs`

##### 16.9.3.1 安全计算

让我们来看看涉及不安全整数的计算。

以下结果是不正确和不安全的，即使它的操作数都是安全的：

```js
> 9007199254740990 + 3
9007199254740992
```

以下结果是安全的，但不正确。第一个操作数是不安全的；第二个操作数是安全的：

```js
> 9007199254740995 - 10
9007199254740986
```

因此，表达式`a op b`的结果是正确的当且仅当：

```js
isSafeInteger(a) && isSafeInteger(b) && isSafeInteger(a op b)
```

也就是说，操作数和结果都必须是安全的。

### 16.10 按位运算符

#### 16.10.1 按位运算符在内部使用 32 位整数

在内部，JavaScript 的按位运算符使用 32 位整数。它们通过以下步骤产生结果：

+   输入（JavaScript 数字）：首先将 1-2 个操作数转换为 JavaScript 数字（64 位浮点数），然后转换为 32 位整数。

+   计算（32 位整数）：实际操作处理 32 位整数并产生 32 位整数。

+   输出（JavaScript 数字）：在返回结果之前，它被转换回 JavaScript 数字。

##### 16.10.1.1 操作数和结果的类型

对于每个按位运算符，本书提到了它的操作数和结果的类型。每种类型总是以下两种之一：

| 类型 | 描述 | 大小 | 范围 |
| --- | --- | --- | --- |
| Int32 | 有符号 32 位整数 | 32 位包括符号 | −2³¹, 2³¹) |
| Uint32 | 无符号 32 位整数 | 32 位 | [0, 2³²) |

考虑到前面提到的步骤，我建议假装按位运算符在内部使用无符号 32 位整数（步骤“计算”），而 Int32 和 Uint32 只影响 JavaScript 数字如何转换为整数和从整数转换为（步骤“输入”和“输出”）。

##### 16.10.1.2 将 JavaScript 数字显示为无符号 32 位整数

在探索按位运算符时，有时将 JavaScript 数字显示为二进制表示的无符号 32 位整数会有所帮助。这就是`b32()`的作用（其实现稍后会显示）：

```js
assert.equal(
 b32(-1),
 '11111111111111111111111111111111');
assert.equal(
 b32(1),
 '00000000000000000000000000000001');
assert.equal(
 b32(2 ** 31),
 '10000000000000000000000000000000');
```

#### 16.10.2 按位非

表 9：按位非运算符。

| 操作 | 名称 | 类型签名 |  |
| --- | --- | --- | --- |
| `~num` | 按位非，*补码* | Int32 `→` Int32 | ES1 |

按位非运算符（tbl. [9）反转其操作数的每个二进制位：

```js
> b32(~0b100)
'11111111111111111111111111111011'
```

所谓的*补码*对于某些算术运算类似于负数。例如，将整数加上它的补码总是`-1`：

```js
> 4 + ~4
-1
> -11 + ~-11
-1
```

#### 16.10.3 二进制按位运算符

表 10：二进制按位运算符。

| 操作 | 名称 | 类型签名 |  |
| --- | --- | --- | --- |
| `num1 & num2` | 按位与 | Int32 × Int32 `→` Int32 | ES1 |
| `num1 ¦ num2` | 按位或 | Int32 × Int32 `→` Int32 | ES1 |
| `num1 ^ num2` | 按位异或 | Int32 × Int32 `→` Int32 | ES1 |

二进制按位运算符（tbl. 10）将它们的操作数的位组合起来产生它们的结果：

```js
> (0b1010 & 0b0011).toString(2).padStart(4, '0')
'0010'
> (0b1010 | 0b0011).toString(2).padStart(4, '0')
'1011'
> (0b1010 ^ 0b0011).toString(2).padStart(4, '0')
'1001'
```

#### 16.10.4 按位移动运算符

表 11：位移运算符。

| 操作 | 名称 | 类型签名 |  |
| --- | --- | --- | --- |
| `num << count` | 左移 | Int32 × Uint32 `→` Int32 | ES1 |
| `num >> count` | 有符号右移 | Int32 × Uint32 `→` Int32 | ES1 |
| `num >>> count` | 无符号右移 | Uint32 × Uint32 `→` Uint32 | ES1 |

位移运算符（见表 11）将二进制数字向左或向右移动：

```js
> (0b10 << 1).toString(2)
'100'
```

`>>`保留最高位，`>>>`不保留：

```js
> b32(0b10000000000000000000000000000010 >> 1)
'11000000000000000000000000000001'
> b32(0b10000000000000000000000000000010 >>> 1)
'01000000000000000000000000000001'
```

#### 16.10.5 `b32()`: 以二进制表示无符号 32 位整数

我们现在已经使用了`b32()`几次。以下代码是它的一个实现：

```js
/**
 * Return a string representing n as a 32-bit unsigned integer,
 * in binary notation.
 */
function b32(n) {
 // >>> ensures highest bit isn’t interpreted as a sign
 return (n >>> 0).toString(2).padStart(32, '0');
}
assert.equal(
 b32(6),
 '00000000000000000000000000000110');
```

`n >>> 0`表示我们将`n`向右移动零位。因此，原则上，`>>>`运算符什么也不做，但它仍然将`n`强制转换为无符号 32 位整数：

```js
> 12 >>> 0
12
> -12 >>> 0
4294967284
> (2**32 + 1) >>> 0
1
```

### 16.11 快速参考：数字

#### 16.11.1 用于数字的全局函数

JavaScript 有以下四个用于数字的全局函数：

+   `isFinite()`

+   `isNaN()`

+   `parseFloat()`

+   `parseInt()`

然而，最好使用`Number`的相应方法（`Number.isFinite()`等），它们有更少的陷阱。它们是在 ES6 中引入的，并在下面讨论。

#### 16.11.2 `Number`的静态属性

+   `.EPSILON: number` ^([ES6])

    1 和下一个可表示的浮点数之间的差异。一般来说，[机器 epsilon](https://en.wikipedia.org/wiki/Machine_epsilon)提供了浮点运算中舍入误差的上限。

    +   大约为：2.2204460492503130808472633361816 × 10^(-16)

+   `.MAX_SAFE_INTEGER: number` ^([ES6])

    JavaScript 可以明确表示的最大整数（2⁵³−1）。

+   `.MAX_VALUE: number` ^([ES1])

    最大的正有限 JavaScript 数字。

    +   大约为：1.7976931348623157 × 10³⁰⁸

+   `.MIN_SAFE_INTEGER: number` ^([ES6])

    JavaScript 可以明确表示的最小整数（−2⁵³+1）。

+   `.MIN_VALUE: number` ^([ES1])

    最小的正 JavaScript 数字。大约为 5 × 10^(−324)。

+   `.NaN: number` ^([ES1])

    与全局变量`NaN`相同。

+   `.NEGATIVE_INFINITY: number` ^([ES1])

    与`-Number.POSITIVE_INFINITY`相同。

+   `.POSITIVE_INFINITY: number` ^([ES1])

    与全局变量`Infinity`相同。

#### 16.11.3 `Number`的静态方法

+   `.isFinite(num: number): boolean` ^([ES6])

    如果`num`是一个实际数字（既不是`Infinity`也不是`-Infinity`也不是`NaN`），则返回`true`。

    ```js
    > Number.isFinite(Infinity)
    false
    > Number.isFinite(-Infinity)
    false
    > Number.isFinite(NaN)
    false
    > Number.isFinite(123)
    true
    ```

+   `.isInteger(num: number): boolean` ^([ES6])

    如果`num`是一个数字并且没有小数部分，则返回`true`。

    ```js
    > Number.isInteger(-17)
    true
    > Number.isInteger(33)
    true
    > Number.isInteger(33.1)
    false
    > Number.isInteger('33')
    false
    > Number.isInteger(NaN)
    false
    > Number.isInteger(Infinity)
    false
    ```

+   `.isNaN(num: number): boolean` ^([ES6])

    如果`num`是值`NaN`，则返回`true`：

    ```js
    > Number.isNaN(NaN)
    true
    > Number.isNaN(123)
    false
    > Number.isNaN('abc')
    false
    ```

+   `.isSafeInteger(num: number): boolean` ^([ES6])

    如果`num`是一个数字并且明确表示一个整数，则返回`true`。

+   `.parseFloat(str: string): number` ^([ES6])

    将其参数强制转换为字符串并将其解析为浮点数。对于将字符串转换为数字，通常使用`Number()`（它忽略前导和尾随空格）比使用`Number.parseFloat()`（它忽略前导空格和非法的尾随字符，并且可能隐藏问题）更好。

    ```js
    > Number.parseFloat(' 123.4#')
    123.4
    > Number(' 123.4#')
    NaN
    ```

+   `.parseInt(str: string, radix=10): number` ^([ES6])

    将其参数强制转换为字符串并将其解析为整数，忽略前导空格和非法的尾随字符：

    ```js
    > Number.parseInt('  123#')
    123
    ```

    参数`radix`指定要解析的数字的基数：

    ```js
    > Number.parseInt('101', 2)
    5
    > Number.parseInt('FF', 16)
    255
    ```

    不要使用此方法将数字转换为整数：强制转换为字符串是低效的。在第一个非数字之前停止不是去除数字的小数部分的好算法。这里有一个它出错的例子：

    ```js
    > Number.parseInt(1e21, 10) // wrong
    1
    ```

    最好使用`Math`的一个舍入函数将数字转换为整数：

    ```js
    > Math.trunc(1e21) // correct
    1e+21
    ```

#### 16.11.4 `Number.prototype`的方法

（`Number.prototype`是存储数字方法的地方。）

+   `.toExponential(fractionDigits?: number): string` ^([ES3])

    返回表示数字的指数表示的字符串。使用`fractionDigits`，我们可以指定应显示与指数相乘的数字的位数（默认情况下，显示所需的位数）。

    示例：数字太小，无法通过`.toString()`获得正指数。

    ```js
    > 1234..toString()
    '1234'

    > 1234..toExponential() // 3 fraction digits
    '1.234e+3'
    > 1234..toExponential(5)
    '1.23400e+3'
    > 1234..toExponential(1)
    '1.2e+3'
    ```

    示例：小数部分不够小，无法通过`.toString()`获得负指数。

    ```js
    > 0.003.toString()
    '0.003'
    > 0.003.toExponential()
    '3e-3'
    ```

+   `.toFixed(fractionDigits=0): string` ^([ES3])

    返回不带指数的数字表示，四舍五入到`fractionDigits`位数。

    ```js
    > 0.00000012.toString() // with exponent
    '1.2e-7'

    > 0.00000012.toFixed(10) // no exponent
    '0.0000001200'
    > 0.00000012.toFixed()
    '0'
    ```

    如果数字大于或等于 10²¹，甚至`.toFixed()`也会使用指数：

    ```js
    > (10 ** 21).toFixed()
    '1e+21'
    ```

+   `.toPrecision(precision?: number): string` ^([ES3])

    类似于`.toString()`，但`precision`指定应显示多少位数字。如果缺少`precision`，则使用`.toString()`。

    ```js
    > 1234..toPrecision(3)  // requires exponential notation
    '1.23e+3'

    > 1234..toPrecision(4)
    '1234'

    > 1234..toPrecision(5)
    '1234.0'

    > 1.234.toPrecision(3)
    '1.23'
    ```

+   `.toString(radix=10): string` ^([ES1])

    返回数字的字符串表示。

    默认情况下，我们得到一个以 10 为底的数字作为结果：

    ```js
    > 123.456.toString()
    '123.456'
    ```

    如果我们希望数字以不同的基数表示，可以通过`radix`指定：

    ```js
    > 4..toString(2) // binary (base 2)
    '100'
    > 4.5.toString(2)
    '100.1'

    > 255..toString(16) // hexadecimal (base 16)
    'ff'
    > 255.66796875.toString(16)
    'ff.ab'

    > 1234567890..toString(36)
    'kf12oi'
    ```

    `Number.parseInt()`提供了反向操作：它将包含给定基数的整数（无小数部分！）数字的字符串转换为数字。

    ```js
    > Number.parseInt('kf12oi', 36)
    1234567890
    ```

#### 16.11.5 来源

+   维基百科

+   [TypeScript 的内置类型](https://github.com/Microsoft/TypeScript/blob/master/lib/)

+   [JavaScript 的 MDN 网页文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

+   [ECMAScript 语言规范](https://tc39.github.io/ecma262/)

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验：高级**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/11)
