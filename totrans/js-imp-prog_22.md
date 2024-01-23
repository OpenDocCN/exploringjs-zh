# 18 大整数 - 任意精度整数 [ES2020]（高级）

> 原文：[`exploringjs.com/impatient-js/ch_bigints.html`](https://exploringjs.com/impatient-js/ch_bigints.html)

* * *

+   18.1 为什么 bigints？

+   18.2 Bigints

    +   18.2.1 超越 53 位整数

    +   18.2.2 示例：使用 bigints

+   18.3 大整数文字

    +   18.3.1 [大整数文字中的下划线（`_`）作为分隔符[ES2021]](ch_bigints.html#numeric-separator-bigint-literals)

+   18.4 重用 bigints 的数字运算符（重载）

    +   18.4.1 算术运算符

    +   18.4.2 排序运算符

    +   18.4.3 位运算符

    +   18.4.4 松散相等（`==`）和不等（`!=`）

    +   18.4.5 严格相等（`===`）和不等（`!==`）

+   18.5 包装构造函数`BigInt`

    +   18.5.1 `BigInt`作为构造函数和作为函数

    +   18.5.2 `BigInt.prototype.*` 方法

    +   18.5.3 `BigInt.*` 方法

    +   18.5.4 强制转换和 64 位整数

+   18.6 将 bigints 强制转换为其他原始类型

+   18.7 64 位值的 TypedArrays 和 DataView 操作

+   18.8 Bigints 和 JSON

    +   18.8.1 将 bigints 转换为字符串

    +   18.8.2 解析大整数

+   18.9 常见问题：Bigints

    +   18.9.1 我如何决定何时使用数字，何时使用 bigints？

    +   18.9.2 为什么不像 bigints 一样增加数字的精度？

* * *

在本章中，我们将看一下*bigints*，JavaScript 的整数，其存储空间会根据需要增长和缩小。

### 18.1 为什么 bigints？

在 ECMAScript 2020 之前，JavaScript 处理整数如下：

+   浮点数和整数只有一个类型：64 位浮点数（IEEE 754 双精度）。

+   在幕后，大多数 JavaScript 引擎透明地支持整数：如果一个数字没有小数位并且在某个范围内，它可以在内部存储为真正的整数。这种表示称为*小整数*，通常适合 32 位。例如，在 V8 引擎的 64 位版本上，小整数的范围是从-2³¹到 2³¹-1（来源）。

+   JavaScript 数字也可以表示超出小整数范围的整数，作为浮点数。在这里，安全范围是加/减 53 位。有关此主题的更多信息，请参见§16.9.3“安全整数”。

有时，我们需要超过有符号的 53 位 - 例如：

+   Twitter 使用 64 位整数作为推文的 ID（来源）。在 JavaScript 中，这些 ID 必须存储为字符串。

+   金融技术使用所谓的*大整数*（具有任意精度的整数）来表示货币金额。在内部，金额被乘以一个数，以便十进制数消失。例如，美元金额被乘以 100，以便消失掉美分。

### 18.2 Bigints

*Bigint*是一个新的整数原始数据类型。大整数的存储大小没有固定的位数；它们的大小会根据它们所表示的整数而调整：

+   小整数的位数比大整数少。

+   可以表示的整数没有负的下限或正的上限。

大整数文字是一个由一个或多个数字组成的序列，后面跟着一个 `n` - 例如：

```js
123n
```

诸如 `-` 和 `*` 这样的运算符被重载并且可以与大整数一起使用：

```js
> 123n * 456n
56088n
```

大整数是原始值。`typeof` 为它们返回一个新的结果：

```js
> typeof 123n
'bigint'
```

#### 18.2.1 超过 53 位的整数

JavaScript 数字在内部表示为一个乘以指数的分数（有关详细信息，请参见§16.8“背景：浮点精度”）。因此，如果我们超过了最高的*安全整数* 2⁵³−1，仍然有*一些*整数可以表示，但它们之间存在间隙：

```js
> 2**53 - 2 // safe
9007199254740990
> 2**53 - 1 // safe
9007199254740991

> 2**53 // unsafe, same as next integer
9007199254740992
> 2**53 + 1
9007199254740992
> 2**53 + 2
9007199254740994
> 2**53 + 3
9007199254740996
> 2**53 + 4
9007199254740996
> 2**53 + 5
9007199254740996
```

大整数使我们能够超过 53 位：

```js
> 2n**53n
9007199254740992n
> 2n**53n + 1n
9007199254740993n
> 2n**53n + 2n
9007199254740994n
```

#### 18.2.2 示例：使用大整数

这就是使用大整数的样子（基于提案中的一个例子）：

```js
/**
 * Takes a bigint as an argument and returns a bigint
 */
function nthPrime(nth) {
 if (typeof nth !== 'bigint') {
 throw new TypeError();
 }
 function isPrime(p) {
 for (let i = 2n; i < p; i++) {
 if (p % i === 0n) return false;
 }
 return true;
 }
 for (let i = 2n; ; i++) {
 if (isPrime(i)) {
 if (--nth === 0n) return i;
 }
 }
}

assert.deepEqual(
 [1n, 2n, 3n, 4n, 5n].map(nth => nthPrime(nth)),
 [2n, 3n, 5n, 7n, 11n]
);
```

### 18.3 大整数文字

与数字文字一样，大整数文字支持几种基数：

+   十进制：`123n`

+   十六进制：`0xFFn`

+   二进制：`0b1101n`

+   八进制：`0o777n`

负大整数是通过在数字前加上一元减号操作符来产生的：`-0123n`

#### 18.3.1 大整数文字中的下划线（`_`）作为分隔符 [ES2021]

就像在数字文字中一样，我们可以在大整数文字中使用下划线（`_`）作为分隔符：

```js
const massOfEarthInKg = 6_000_000_000_000_000_000_000_000n;
```

大整数经常用于在金融技术领域表示货币。分隔符在这里也有帮助：

```js
const priceInCents = 123_000_00n; // 123 thousand dollars
```

与数字文字一样，有两个限制：

+   我们只能在两个数字之间放一个下划线。

+   我们最多可以连续使用一个下划线。

### 18.4 重用大整数的数字运算符（重载）

对于大多数运算符，我们不允许混合使用大整数和数字。如果这样做，就会抛出异常：

```js
> 2n + 1
TypeError: Cannot mix BigInt and other types, use explicit conversions
```

这个规则的原因是没有一般的方法来强制一个数字和一个大整数转换为一个公共类型：数字无法表示超过 53 位的大整数，大整数无法表示分数。因此，这些异常警告我们可能导致意外结果的拼写错误。

例如，以下表达式的结果应该是 `9007199254740993n` 还是 `9007199254740992`？

```js
2**53 + 1n
```

以下表达式的结果也不清楚：

```js
2n**53n * 3.3
```

#### 18.4.1 算术运算符

二进制 `+`，二进制 `-`，`*`，`**` 的工作方式与预期相同：

```js
> 7n * 3n
21n
```

混合使用大整数和字符串是可以的：

```js
> 6n + ' apples'
'6 apples'
```

`/`，`%` 向零舍入（就像 `Math.trunc()`）：

```js
> 1n / 2n
0n
```

一元 `-` 的工作方式与预期相同：

```js
> -(-64n)
64n
```

大整数不支持一元 `+`，因为很多代码依赖于它将其操作数强制转换为数字：

```js
> +23n
TypeError: Cannot convert a BigInt value to a number
```

#### 18.4.2 排序运算符

排序运算符 `<`，`>`，`>=`，`<=` 的工作方式与预期相同：

```js
> 17n <= 17n
true
> 3n > -1n
true
```

比较大整数和数字不会带来任何风险。因此，我们可以混合使用大整数和数字：

```js
> 3n > -1
true
```

#### 18.4.3 按位运算符

##### 18.4.3.1 数字的按位运算符

按位运算符将数字解释为 32 位整数。这些整数可以是无符号的，也可以是有符号的。如果它们是有符号的，那么一个整数的负数是它的*二进制补码*（将一个整数加上它的二进制补码 - 忽略溢出 - 会得到零）：

```js
> 2**32-1 >> 0
-1
```

由于这些整数具有固定的大小，它们的最高位表示它们的符号：

```js
> 2**31 >> 0 // highest bit is 1
-2147483648
> 2**31 - 1 >> 0 // highest bit is 0
2147483647
```

##### 18.4.3.2 大整数的按位运算符

对于大整数，按位运算符将负号解释为无限的二进制补码 - 例如：

+   `-1` 是 `···111111`（1 无限扩展到左边）

+   `-2` 是 `···111110`

+   `-3` 是 `···111101`

+   `-4` 是 `···111100`

也就是说，负号更像是一个外部标志，而不是实际表示为一个位。

##### 18.4.3.3 按位取反（`~`）

按位取反（`~`）反转所有位：

```js
> ~0b10n
-3n
> ~0n
-1n
> ~-2n
1n
```

##### 18.4.3.4 二进制按位运算符（`&`，`|`，`^`）

将二进制按位运算符应用于大整数的工作方式类似于将它们应用于数字：

```js
> (0b1010n |  0b0111n).toString(2)
'1111'
> (0b1010n &  0b0111n).toString(2)
'10'

> (0b1010n | -1n).toString(2)
'-1'
> (0b1010n & -1n).toString(2)
'1010'
```

##### 18.4.3.5 按位有符号移位运算符（`<<` 和 `>>`）

bigint 的有符号移位运算符保留数字的符号：

```js
> 2n << 1n
4n
> -2n << 1n
-4n

> 2n >> 1n
1n
> -2n >> 1n
-1n
```

回想一下，`-1n` 是一个向左无限延伸的一系列数字。这就是为什么将其向左移动不会改变它的原因：

```js
> -1n >> 20n
-1n
```

##### 18.4.3.6 位无符号右移运算符 (`>>>`)

bigint 没有无符号右移运算符：

```js
> 2n >>> 1n
TypeError: BigInts have no unsigned right shift, use >> instead
```

为什么？无符号右移的想法是从“左边”移入一个零。换句话说，假设是有限数量的二进制数字。

然而，对于 bigint，没有“左”，它们的二进制数字无限延伸。这在处理负数时尤为重要。

有符号右移即使在有无限位数的情况下也可以工作，因为最高位数字被保留。因此，它可以适应 bigint。

#### 18.4.4 宽松相等 (`==`) 和不等 (`!=`)

宽松相等 (`==`) 和不等 (`!=`) 强制转换值：

```js
> 0n == false
true
> 1n == true
true

> 123n == 123
true

> 123n == '123'
true
```

#### 18.4.5 严格相等 (`===`) 和不等 (`!==`)

严格相等 (`===`) 和不等 (`!==`) 只有在它们具有相同类型时才被认为是相等的：

```js
> 123n === 123
false
> 123n === 123n
true
```

### 18.5 包装构造函数 `BigInt`

类似于数字，bigint 有关联的包装构造函数 `BigInt`。

#### 18.5.1 `BigInt` 作为构造函数和作为函数

+   `new BigInt()`: 抛出 `TypeError`。

+   `BigInt(x)` 将任意值 `x` 转换为 bigint。这类似于 `Number()`，但有几个不同之处，这些不同之处在 tbl. 13 中总结，并在以下小节中详细解释。

表 13：将值转换为 bigint。

| `x` | `BigInt(x)` |
| --- | --- |
| `undefined` | 抛出 `TypeError` |
| `null` | 抛出 `TypeError` |
| 布尔值 | `false` `→` `0n`, `true` `→` `1n` |
| 数字 | 例子：`123` `→` `123n` |
|  | 非整数 `→` 抛出 `RangeError` |
| bigint | `x` (不变) |
| 字符串 | 例子：`'123'` `→` `123n` |
|  | 无法解析的 `→` 抛出 `SyntaxError` |
| 符号 | 抛出 `TypeError` |
| 对象 | 可配置的（例如通过 `.valueOf()`） |

##### 18.5.1.1 转换 `undefined` 和 `null`

如果 `x` 是 `undefined` 或 `null`，则抛出 `TypeError`：

```js
> BigInt(undefined)
TypeError: Cannot convert undefined to a BigInt
> BigInt(null)
TypeError: Cannot convert null to a BigInt
```

##### 18.5.1.2 转换字符串

如果一个字符串不表示整数，`BigInt()` 抛出 `SyntaxError`（而 `Number()` 返回错误值 `NaN`）：

```js
> BigInt('abc')
SyntaxError: Cannot convert abc to a BigInt
```

后缀 `'n'` 是不允许的：

```js
> BigInt('123n')
SyntaxError: Cannot convert 123n to a BigInt
```

bigint 文字的所有基数都是允许的：

```js
> BigInt('123')
123n
> BigInt('0xFF')
255n
> BigInt('0b1101')
13n
> BigInt('0o777')
511n
```

##### 18.5.1.3 非整数数字会产生异常

```js
> BigInt(123.45)
RangeError: The number 123.45 cannot be converted to a BigInt because
it is not an integer
> BigInt(123)
123n
```

##### 18.5.1.4 转换对象

如何将对象转换为 bigint 可以进行配置 - 例如，通过覆盖 `.valueOf()`：

```js
> BigInt({valueOf() {return 123n}})
123n
```

#### 18.5.2 `BigInt.prototype.*` 方法

`BigInt.prototype` 包含了原始 bigint “继承”的方法：

+   `BigInt.prototype.toLocaleString(locales?, options?)`

+   `BigInt.prototype.toString(radix?)`

+   `BigInt.prototype.valueOf()`

#### 18.5.3 `BigInt.*` 方法

+   `BigInt.asIntN(width, theInt)`

    将 `theInt` 转换为 `width` 位（有符号）。这会影响值在内部的表示方式。

+   `BigInt.asUintN(width, theInt)`

    将 `theInt` 转换为 `width` 位（无符号）。

#### 18.5.4 强制转换和 64 位整数

强制转换允许我们创建具有特定位数的整数值。如果我们想要限制自己只使用 64 位整数，我们必须始终进行强制转换：

```js
const uint64a = BigInt.asUintN(64, 12345n);
const uint64b = BigInt.asUintN(64, 67890n);
const result = BigInt.asUintN(64, uint64a * uint64b);
```

### 18.6 将 bigint 强制转换为其他原始类型

这个表格展示了如果我们将 bigint 转换为其他原始类型会发生什么：

| 转换为 | 显式转换 | 强制转换（隐式转换） |
| --- | --- | --- |
| 布尔值 | `Boolean(0n)` `→` `false` | `!0n` `→` `true` |
|  | `Boolean(int)` `→` `true` | `!int` `→` `false` |
| 数字 | `Number(7n)` `→` `7` (例子) | `+int` `→` `TypeError` (1) |
| 字符串 | `String(7n)` `→` `'7'` (例子) | `''+7n` `→` `'7'` (例子) |

注释：

+   (1) 由于很多代码依赖于它将其操作数强制转换为数字，因此 bigint 不支持一元 `+`。

### 18.7 64 位值的 TypedArrays 和 DataView 操作

由于 bigint，Typed Arrays 和 DataViews 可以支持 64 位值。

+   类型化数组构造函数：

    +   `BigInt64Array`

    +   `BigUint64Array`

+   DataView 方法：

    +   `DataView.prototype.getBigInt64()`

    +   `DataView.prototype.setBigInt64()`

    +   `DataView.prototype.getBigUint64()`

    +   `DataView.prototype.setBigUint64()`

### 18.8 Bigints 和 JSON

JSON 标准是固定的，不会改变。好处是旧的 JSON 解析代码永远不会过时。坏处是 JSON 无法扩展以包含 bigint。

序列化 bigint 会抛出异常：

```js
> JSON.stringify(123n)
TypeError: Do not know how to serialize a BigInt
> JSON.stringify([123n])
TypeError: Do not know how to serialize a BigInt
```

#### 18.8.1 序列化 bigint

因此，我们最好的选择是将 bigint 存储为字符串：

```js
const bigintPrefix = '[[bigint]]';

function bigintReplacer(_key, value) {
 if (typeof value === 'bigint') {
 return bigintPrefix + value;
 }
 return value;
}

const data = { value: 9007199254740993n };
assert.equal(
 JSON.stringify(data, bigintReplacer),
 '{"value":"[[bigint]]9007199254740993"}'
);
```

#### 18.8.2 解析 bigint

以下代码显示了如何解析诸如我们在前面示例中生成的字符串。

```js
function bigintReviver(_key, value) {
 if (typeof value === 'string' && value.startsWith(bigintPrefix)) {
 return BigInt(value.slice(bigintPrefix.length));
 }
 return value;
}

const str = '{"value":"[[bigint]]9007199254740993"}';
assert.deepEqual(
 JSON.parse(str, bigintReviver),
 { value: 9007199254740993n }
);
```

### 18.9 常见问题：Bigints

#### 18.9.1 我如何决定何时使用数字，何时使用 bigint？

我的建议：

+   对于最多 53 位和数组索引，请使用数字。原因是：它们已经随处可见，并且大多数引擎都可以高效处理它们（特别是如果它们适合 31 位）。出现的情况包括：

    +   `Array.prototype.forEach()`

    +   `Array.prototype.entries()`

+   对于大数值，请使用 bigint：如果您的无小数数值不适合 53 位，那么您别无选择，只能转为 bigint。

所有现有的 web API 只返回和接受数字，并且只会在特定情况下升级为 bigint。

#### 18.9.2 为什么不像 bigint 一样增加数字的精度？

可以想象将`number`分成`integer`和`double`，但这将给语言增加许多新的复杂性（几个仅限整数的运算符等）。我在[a Gist](https://gist.github.com/rauschma/13d48d1c49615ce2396ce7c9e45d4cd1)中勾勒了后果。

* * *

**致谢：**

+   感谢 Daniel Ehrenberg 对此内容的早期版本进行审查。

+   感谢 Dan Callahan 对此内容的早期版本进行审查。

[评论](https://github.com/rauschma/impatient-js/issues/50)
