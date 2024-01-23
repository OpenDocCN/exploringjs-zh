# 十七、数学

> 原文：[`exploringjs.com/impatient-js/ch_math.html`](https://exploringjs.com/impatient-js/ch_math.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   17.1 数据属性

+   17.2 指数、根、对数

+   17.3 舍入

+   17.4 三角函数

+   17.5 其他各种函数

+   17.6 来源

* * *

`Math`是一个具有数据属性和用于处理数字的方法的对象。您可以将其视为一个简陋的模块：它是在 JavaScript 拥有模块之前创建的。

### 17.1 数据属性

+   `Math.E: number` ^([ES1])

    欧拉数，自然对数的底数，约为 2.7182818284590452354。

+   `Math.LN10: number` ^([ES1])

    10 的自然对数，约为 2.302585092994046。

+   `Math.LN2: number` ^([ES1])

    2 的自然对数，约为 0.6931471805599453。

+   `Math.LOG10E: number` ^([ES1])

    以 10 为底的*e*的对数，约为 0.4342944819032518。

+   `Math.LOG2E: number` ^([ES1])

    *e*的以 2 为底的对数，约为 1.4426950408889634。

+   `Math.PI: number` ^([ES1])

    数学常数π，圆的周长与直径的比值，约为 3.1415926535897932。

+   `Math.SQRT1_2: number` ^([ES1])

    1/2 的平方根，约为 0.7071067811865476。

+   `Math.SQRT2: number` ^([ES1])

    2 的平方根，约为 1.4142135623730951。

### 17.2 指数、根、对数

+   `Math.cbrt(x: number): number` ^([ES6])

    返回`x`的立方根。

    ```js
    > Math.cbrt(8)
    2
    ```

+   `Math.exp(x: number): number` ^([ES1])

    返回*e*^(`x`)（*e*为欧拉数）。是`Math.log()`的反函数。

    ```js
    > Math.exp(0)
    1
    > Math.exp(1) === Math.E
    true
    ```

+   `Math.expm1(x: number): number` ^([ES6])

    返回`Math.exp(x)-1`。是`Math.log1p()`的反函数。非常小的数字（接近 0 的分数）以更高的精度表示。因此，当`.exp()`返回接近 1 的值时，此函数返回更精确的值。

+   `Math.log(x: number): number` ^([ES1])

    返回`x`的自然对数（以*e*为底，即欧拉数）。是`Math.exp()`的反函数。

    ```js
    > Math.log(1)
    0
    > Math.log(Math.E)
    1
    > Math.log(Math.E ** 2)
    2
    ```

+   `Math.log1p(x: number): number` ^([ES6])

    返回`Math.log(1 + x)`。是`Math.expm1()`的反函数。非常小的数字（接近 0 的分数）以更高的精度表示。因此，当`.log()`的参数接近 1 时，您可以提供此函数更精确的参数。

+   `Math.log10(x: number): number` ^([ES6])

    返回`x`的以 10 为底的对数。是`10 ** x`的反函数。

    ```js
    > Math.log10(1)
    0
    > Math.log10(10)
    1
    > Math.log10(100)
    2
    ```

+   `Math.log2(x: number): number` ^([ES6])

    返回`x`的以 2 为底的对数。是`2 ** x`的反函数。

    ```js
    > Math.log2(1)
    0
    > Math.log2(2)
    1
    > Math.log2(4)
    2
    ```

+   `Math.pow(x: number, y: number): number` ^([ES1])

    返回`x`^(`y`), `x`的`y`次方。与`x ** y`相同。

    ```js
    > Math.pow(2, 3)
    8
    > Math.pow(25, 0.5)
    5
    ```

+   `Math.sqrt(x: number): number` ^([ES1])

    返回`x`的平方根。是`x ** 2`的反函数。

    ```js
    > Math.sqrt(9)
    3
    ```

### 17.3 舍入

舍入意味着将任意数字转换为整数（没有小数部分的数字）。以下函数实现了不同的舍入方法。

+   `Math.ceil(x: number): number` ^([ES1])

    返回最小的（最接近-∞）整数`i`，使得`x` ≤ `i`。

    ```js
    > Math.ceil(2.1)
    3
    > Math.ceil(2.9)
    3
    ```

+   `Math.floor(x: number): number` ^([ES1])

    返回最大的（最接近+∞）整数`i`，使得`i` ≤ `x`。

    ```js
    > Math.floor(2.1)
    2
    > Math.floor(2.9)
    2
    ```

+   `Math.round(x: number): number` ^([ES1])

    返回最接近`x`的整数。如果`x`的小数部分是`.5`，则`.round()`会向上舍入（到更接近正无穷大的整数）：

    ```js
    > Math.round(2.4)
    2
    > Math.round(2.5)
    3
    ```

+   `Math.trunc(x: number): number` ^([ES6])

    去除`x`的小数部分并返回结果整数。

    ```js
    > Math.trunc(2.1)
    2
    > Math.trunc(2.9)
    2
    ```

Tbl. 12 显示了几个代表性输入的舍入函数的结果。

表 12：`Math`的舍入函数。请注意，由于“更大”始终意味着“更接近正无穷大”，因此负数会导致结果发生变化。

|  | `-2.9` | `-2.5` | `-2.1` | `2.1` | `2.5` | `2.9` |
| --- | --- | --- | --- | --- | --- | --- |
| `Math.floor` | `-3` | `-3` | `-3` | `2` | `2` | `2` |
| `Math.ceil` | `-2` | `-2` | `-2` | `3` | `3` | `3` |
| `Math.round` | `-3` | `-2` | `-2` | `2` | `3` | `3` |
| `Math.trunc` | `-2` | `-2` | `-2` | `2` | `2` | `2` |

### 17.4 三角函数

所有角度均以弧度指定。使用以下两个函数在度和弧度之间进行转换。

```js
function degreesToRadians(degrees) {
 return degrees / 180 * Math.PI;
}
assert.equal(degreesToRadians(90), Math.PI/2);

function radiansToDegrees(radians) {
 return radians / Math.PI * 180;
}
assert.equal(radiansToDegrees(Math.PI), 180);
```

+   `Math.acos(x: number): number` ^([ES1])

    返回 `x` 的反余弦（反余弦）。

    ```js
    > Math.acos(0)
    1.5707963267948966
    > Math.acos(1)
    0
    ```

+   `Math.acosh(x: number): number` ^([ES6])

    返回 `x` 的反双曲余弦。

+   `Math.asin(x: number): number` ^([ES1])

    返回 `x` 的反正弦（反正弦）。

    ```js
    > Math.asin(0)
    0
    > Math.asin(1)
    1.5707963267948966
    ```

+   `Math.asinh(x: number): number` ^([ES6])

    返回 `x` 的反双曲正弦。

+   `Math.atan(x: number): number` ^([ES1])

    返回 `x` 的反正切（反正切）。

+   `Math.atanh(x: number): number` ^([ES6])

    返回 `x` 的反双曲正切。

+   `Math.atan2(y: number, x: number): number` ^([ES1])

    返回商 y/x 的反正切。

+   `Math.cos(x: number): number` ^([ES1])

    返回 `x` 的余弦。

    ```js
    > Math.cos(0)
    1
    > Math.cos(Math.PI)
    -1
    ```

+   `Math.cosh(x: number): number` ^([ES6])

    返回 `x` 的双曲余弦。

+   `Math.hypot(...values: number[]): number` ^([ES6])

    返回 `values` 的平方和的平方根（毕达哥拉斯定理）：

    ```js
    > Math.hypot(3, 4)
    5
    ```

+   `Math.sin(x: number): number` ^([ES1])

    返回 `x` 的正弦。

    ```js
    > Math.sin(0)
    0
    > Math.sin(Math.PI / 2)
    1
    ```

+   `Math.sinh(x: number): number` ^([ES6])

    返回 `x` 的双曲正弦。

+   `Math.tan(x: number): number` ^([ES1])

    返回 `x` 的正切。

    ```js
    > Math.tan(0)
    0
    > Math.tan(1)
    1.5574077246549023
    ```

+   `Math.tanh(x: number): number;` ^([ES6])

    返回 `x` 的双曲正切。

### 17.5 其他各种功能

+   `Math.abs(x: number): number` ^([ES1])

    返回 `x` 的绝对值。

    ```js
    > Math.abs(3)
    3
    > Math.abs(-3)
    3
    > Math.abs(0)
    0
    ```

+   `Math.clz32(x: number): number` ^([ES6])

    计算 32 位整数 `x` 中前导零位的数量。用于 DSP 算法。

    ```js
    > Math.clz32(0b01000000000000000000000000000000)
    1
    > Math.clz32(0b00100000000000000000000000000000)
    2
    > Math.clz32(2)
    30
    > Math.clz32(1)
    31
    ```

+   `Math.max(...values: number[]): number` ^([ES1])

    将 `values` 转换为数字并返回最大值。

    ```js
    > Math.max(3, -5, 24)
    24
    ```

+   `Math.min(...values: number[]): number` ^([ES1])

    将 `values` 转换为数字并返回最小值。

    ```js
    > Math.min(3, -5, 24)
    -5
    ```

+   `Math.random(): number` ^([ES1])

    返回一个伪随机数 `n`，其中 0 ≤ `n` < 1。

    ```js
    /** Returns a random integer i with 0 <= i < max */
    function getRandomInteger(max) {
     return Math.floor(Math.random() * max);
    }
    ```

+   `Math.sign(x: number): number` ^([ES6])

    返回一个数字的符号：

    ```js
    > Math.sign(-8)
    -1
    > Math.sign(0)
    0
    > Math.sign(3)
    1
    ```

### 17.6 来源

+   维基百科

+   [TypeScript 的内置类型](https://github.com/Microsoft/TypeScript/blob/master/lib/)

+   [MDN web JavaScript 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

+   [ECMAScript 语言规范](https://tc39.github.io/ecma262/)

[评论](https://github.com/rauschma/impatient-js/issues/12)
