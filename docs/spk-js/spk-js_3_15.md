# 第二十一章：数学

> 原文：[21. Math](https://exploringjs.com/es5/ch21.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


`Math`对象用作多个数学函数的命名空间。本章提供了一个概述。

## 数学属性

`Math`的属性如下：

`Math.E`

欧拉常数（e）

`Math.LN2`

2 的自然对数

`Math.LN10`

10 的自然对数

`Math.LOG2E`

e 的底数 2 对数

`Math.LOG10E`

e 的十进制对数

`Math.PI`

圆的周长与直径的比值（3.14159 ...），π

`Math.SQRT1_2`

一半的平方根，![](inleq_2101.png)

`Math.SQRT2`

二的平方根，![](inleq_2102.png)

## 数值函数

`Math`的数值函数包括以下内容：

`Math.abs(x)`

返回`x`的绝对值。

`Math.ceil(x)`

返回大于等于`x`的最小整数：

```js
> Math.ceil(3.999)
4
> Math.ceil(3.001)
4
> Math.ceil(-3.001)
-3
> Math.ceil(3.000)
3
```

有关将浮点数转换为整数的更多信息，请参阅[转换为整数](ch11.html#converting_to_integer "Converting to Integer")。

`Math.exp(x)`

返回 e^x，其中 e 是欧拉常数（`Math.E`）。这是`Math.log()`的反函数。

`Math.floor(x)`

返回小于等于`x`的最大整数：

```js
> Math.floor(3.999)
3
> Math.floor(3.001)
3
> Math.floor(-3.001)
-4
> Math.floor(3.000)
3
```

有关将浮点数转换为整数的更多信息，请参阅[转换为整数](ch11.html#converting_to_integer "Converting to Integer")。

`Math.log(x)`

返回`x`的自然（以 e 为底）对数 ln(`x`)。这是`Math.exp()`的反函数。

`Math.pow(x, y)`

返回 x^y，`x`的`y`次幂：

```js
> Math.pow(9, 2)
81
> Math.pow(36, 0.5)
6
```

`Math.round(x)`

返回`x`四舍五入到最接近的整数（如果在两个整数之间，则为较大的整数）：

```js
> Math.round(3.999)
4
> Math.round(3.001)
3
> Math.round(3.5)
4
> Math.round(-3.5)
-3
```

有关将浮点数转换为整数的更多信息，请参阅[转换为整数](ch11.html#converting_to_integer "Converting to Integer")。

`Math.sqrt(x)`

返回![](inleq_2103.png)，`x`的平方根：

```js
> Math.sqrt(256)
16
```

## 三角函数

三角函数方法接受弧度作为角度并返回。以下函数向您展示了如何实现转换，如果需要的话：

+   从度到弧度：

    ```js
    function toRadians(degrees) {
        return degrees / 180 * Math.PI;
    }
    ```

以下是交互：

```js
> toRadians(180)
3.141592653589793
> toRadians(90)
1.5707963267948966
```

+   从弧度到度：

    ```js
    function toDegrees(radians) {
        return radians / Math.PI * 180;
    }
    ```

以下是交互：

```js
> toDegrees(Math.PI * 2)
360
> toDegrees(Math.PI)
180
```

三角函数方法如下：

`Math.acos(x)`

返回`x`的反余弦值。

`Math.asin(x)`

返回`x`的反正弦值。

`Math.atan(x)`

返回`x`的反正切值。

`Math.atan2(y, x)`

返回商的反正切值![](inleq_2104.png)。

`Math.cos(x)`

返回`x`的余弦值。

`Math.sin(x)`

返回`x`的正弦值。

`Math.tan(x)`

返回`x`的正切值。

## 其他函数

以下是剩余的`Math`函数：

`min(x1?, x2?, ...)`

返回参数中的最小数：

```js
> Math.min()
Infinity
> Math.min(27)
27
> Math.min(27, -38)
-38
> Math.min(27, -38, -43)
-43
```

通过`apply()`在数组上使用它（参见[func.apply(thisValue, argArray)](ch15.html#functional_apply "func.apply(thisValue, argArray)")）：

```js
> Math.min.apply(null, [27, -38, -43])
-43
```

`max(x1?, x2?, ...)`

返回参数中的最大数：

```js
> Math.max()
-Infinity
> Math.max(7)
7
> Math.max(7, 10)
10
> Math.max(7, 10, -333)
10
```

通过`apply()`在数组上使用它（参见[func.apply(thisValue, argArray)](ch15.html#functional_apply "func.apply(thisValue, argArray)")）：

```js
> Math.max.apply(null, [7, 10, -333])
10
```

`Math.random()`

```js
/**
 * Compute a random integer within the given range.
 *
 * @param [lower] Optional lower bound. Default: zero.
 * @returns A random integer i, lower ≤ i < upper
 */
function getRandomInteger(lower, upper) {
    if (arguments.length === 1) {
        upper = lower;
        lower = 0;
    }
    return Math.floor(Math.random() * (upper - lower)) + lower;
}
```
