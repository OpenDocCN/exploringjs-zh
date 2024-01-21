# 第二十章：日期

> 原文：[20. Dates](https://exploringjs.com/es5/ch20.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 的`Date`构造函数有助于解析、管理和显示日期。本章描述了它的工作原理。

日期 API 使用术语*UTC*（协调世界时）。在大多数情况下，UTC 是 GMT（格林尼治标准时间）的同义词，大致意味着伦敦，英国的时区。

## 日期构造函数

有四种调用`Date`构造函数的方法：

`new Date(year, month, date?, hours?, minutes?, seconds?, milliseconds?)`

从给定数据构造一个新的日期。时间相对于当前时区进行解释。`Date.UTC()`提供了类似的功能，但是相对于 UTC。参数具有以下范围：

+   `year`：对于 0 ≤ `year` ≤ 99，将添加 1900。

+   `month`：0-11（0 是一月，1 是二月，依此类推）

+   `date`：1-31

+   `hours`：0-23

+   `minutes`：0-59

+   `seconds`：0-59

+   `milliseconds`：0-999

以下是一些示例：

```js
> new Date(2001, 1, 27, 14, 55)
Date {Tue Feb 27 2001 14:55:00 GMT+0100 (CET)}
> new Date(01, 1, 27, 14, 55)
Date {Wed Feb 27 1901 14:55:00 GMT+0100 (CET)}
```

顺便说一句，JavaScript 继承了略微奇怪的约定，将 0 解释为一月，1 解释为二月，依此类推，这一点来自 Java。

`new Date(dateTimeStr)`

这是一个将日期时间字符串转换为数字的过程，然后调用`new Date(number)`。[日期时间格式](ch20.html#date_time_formats "日期时间格式")解释了日期时间格式。例如：

```js
> new Date('2004-08-29')
Date {Sun Aug 29 2004 02:00:00 GMT+0200 (CEST)}
```

非法的日期时间字符串导致将`NaN`传递给`new Date(number)`。

`new Date(timeValue)`

根据自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数创建日期。例如：

```js
> new Date(0)
Date {Thu Jan 01 1970 01:00:00 GMT+0100 (CET)}
```

这个构造函数的反函数是`getTime()`方法，它返回毫秒数：

```js
> new Date(123).getTime()
123
```

您可以使用`NaN`作为参数，这将产生`Date`的一个特殊实例，即“无效日期”：

```js
> var d = new Date(NaN);
> d.toString()
'Invalid Date'
> d.toJSON()
null
> d.getTime()
NaN
> d.getYear()
NaN
```

`new Date()`

创建当前日期和时间的对象；它与`new Date(Date.now())`的作用相同。

## 日期构造函数方法

构造函数`Date`有以下方法：

`Date.now()`

以毫秒为单位返回当前日期和时间（自 1970 年 1 月 1 日 00:00:00 UTC 起）。它产生与`new Date().getTime()`相同的结果。

`Date.parse(dateTimeString)`

将 `dateTimeString` 转换为自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数。[日期时间格式](ch20.html#date_time_formats "日期时间格式")解释了 `dateTimeString` 的格式。结果可用于调用 `new Date(number)`。以下是一些示例：

```js
> Date.parse('1970-01-01')
0
> Date.parse('1970-01-02')
86400000
```

如果无法解析字符串，此方法将返回 `NaN`：

```js
> Date.parse('abc')
NaN
```

`Date.UTC(year, month, date?, hours?, minutes?, seconds?, milliseconds?)`

将给定数据转换为自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数。它与具有相同参数的 `Date` 构造函数有两种不同之处：

+   它返回一个数字，而不是一个新的日期对象。

+   它将参数解释为世界协调时间，而不是本地时间。

## 日期原型方法

本节涵盖了 `Date.prototype` 的方法。

### 时间单位的获取器和设置器

时间单位的获取器和设置器可使用以下签名：

+   本地时间：

+   `Date.prototype.get«Unit»()` 返回根据本地时间的 `Unit`。

+   `Date.prototype.set«Unit»(number)` 根据本地时间设置 `Unit`。

+   世界协调时间：

+   `Date.prototype.getUTC«Unit»()` 返回根据世界协调时间的 `Unit`。

+   `Date.prototype.setUTC«Unit»(number)` 根据世界协调时间设置 `Unit`。

`Unit` 是以下单词之一：

+   `FullYear`：通常是四位数

+   `Month`：月份（0-11）

+   `Date`：月份中的某一天（1-31）

+   `Day`（仅获取器）：星期几（0-6）；0 代表星期日

+   `Hours`：小时（0-23）

+   `Minutes`：分钟（0-59）

+   `Seconds`：秒（0-59）

+   `Milliseconds`：毫秒（0-999）

例如：

```js
> var d = new Date('1968-11-25');
Date {Mon Nov 25 1968 01:00:00 GMT+0100 (CET)}
> d.getDate()
25
> d.getDay()
1
```

### 各种获取器和设置器

以下方法使您能够获取和设置自 1970 年 1 月 1 日以来的毫秒数以及更多内容：

+   `Date.prototype.getTime()` 返回自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数（参见[时间值：日期作为自 1970-01-01 以来的毫秒数](ch20.html#date_milliseconds "时间值：日期作为自 1970-01-01 以来的毫秒数")）。

+   `Date.prototype.setTime(timeValue)` 根据自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数设置日期（参见[时间值：日期作为自 1970-01-01 以来的毫秒数](ch20.html#date_milliseconds "时间值：日期作为自 1970-01-01 以来的毫秒数")）。

+   `Date.prototype.valueOf()` 与 `getTime()` 相同。当日期转换为数字时，将调用此方法。

+   `Date.prototype.getTimezoneOffset()` 返回本地时间和世界协调时间之间的差异（以分钟为单位）。

`Year` 单位已被弃用，推荐使用 `FullYear`：

+   `Date.prototype.getYear()` 已被弃用；请改用 `getFullYear()`。

+   `Date.prototype.setYear(number)` 已被弃用；请改用 `setFullYear()`。

### 将日期转换为字符串

请注意，转换为字符串高度依赖于实现。以下日期用于计算以下示例中的输出（在撰写本书时，Firefox 的支持最完整）：

```js
new Date(2001,9,30, 17,43,7, 856);
```

时间（可读）

+   `Date.prototype.toTimeString()`：

    ```js
    17:43:07 GMT+0100 (CET)
    ```

以当前时区的时间显示。

+   `Date.prototype.toLocaleTimeString()`：

    ```js
    17:43:07
    ```

以特定于区域的格式显示的时间。此方法由 ECMAScript 国际化 API（参见[ECMAScript 国际化 API](ch30.html#i18n_api "ECMAScript 国际化 API")）提供，并且如果没有它，就没有太多意义。

日期（可读）

+   `Date.prototype.toDateString()`：

    ```js
    Tue Oct 30 2001
    ```

日期。

+   `Date.prototype.toLocaleDateString()`：

    ```js
    10/30/2001
    ```

以特定于区域的格式显示的日期。此方法由 ECMAScript 国际化 API（参见[ECMAScript 国际化 API](ch30.html#i18n_api "ECMAScript 国际化 API")）提供，并且如果没有它，就没有太多意义。

日期和时间（可读）

+   `Date.prototype.toString()`：

    ```js
    Tue Oct 30 2001 17:43:07 GMT+0100 (CET)
    ```

日期和时间，以当前时区的时间。对于没有毫秒的任何 `Date` 实例（即秒数已满），以下表达式为真：

```js
Date.parse(d.toString()) === d.valueOf()
```

+   `Date.prototype.toLocaleString()`：

    ```js
    Tue Oct 30 17:43:07 2001
    ```

以区域特定格式的日期和时间。此方法由 ECMAScript 国际化 API 提供（请参见[ECMAScript 国际化 API](ch30.html#i18n_api "ECMAScript 国际化 API")），如果没有它，这个方法就没有太多意义。

+   `Date.prototype.toUTCString()`：

    ```js
    Tue, 30 Oct 2001 16:43:07 GMT
    ```

日期和时间，使用 UTC。

+   `Date.prototype.toGMTString()`：

已弃用；请改用`toUTCString()`。

日期和时间（机器可读）

+   `Date.prototype.toISOString()`：

    ```js
    2001-10-30T16:43:07.856Z
    ```

所有内部属性都显示在返回的字符串中。格式符合[日期时间格式](ch20.html#date_time_formats "日期时间格式")；时区始终为`Z`。

+   `Date.prototype.toJSON()`：

此方法内部调用`toISOString()`。它被`JSON.stringify()`（参见[JSON.stringify(value, replacer?, space?)](ch22.html#JSON.stringify "JSON.stringify(value, replacer?, space?)")）用于将日期对象转换为 JSON 字符串。

## 日期时间格式

本节描述了以字符串形式表示时间点的格式。有许多方法可以这样做：仅指示日期，包括一天中的时间，省略时区，指定时区等。在支持日期时间格式方面，ECMAScript 5 紧密遵循标准 ISO 8601 扩展格式。JavaScript 引擎相对完全地实现了 ECMAScript 规范，但仍然存在一些变化，因此您必须保持警惕。

最长的日期时间格式是：

```js
YYYY-MM-DDTHH:mm:ss.sssZ
```

每个部分代表日期时间数据的几个十进制数字。例如，`YYYY`表示格式以四位数年份开头。以下各小节解释了每个部分的含义。这些格式与以下方法相关：

+   `Date.parse()` 可以解析这些格式。

+   `new Date()`可以解析这些格式。

+   `Date.prototype.toISOString()`创建了上述“完整”格式的字符串：

    ```js
    > new Date().toISOString()
    '2014-09-12T23:05:07.414Z'
    ```

### 日期格式（无时间）

以下日期格式可用：

```js
YYYY-MM-DD
YYYY-MM
YYYY
```

它们包括以下部分：

+   `YYYY` 指的是年份（公历）。

+   `MM`指的是月份，从 01 到 12。

+   `DD` 指的是日期，从 01 到 31。

例如：

```js
> new Date('2001-02-22')
Date {Thu Feb 22 2001 01:00:00 GMT+0100 (CET)}
```

### 时间格式（无日期）

以下时间格式可用。如您所见，时区信息`Z`是可选的：

```js
THH:mm:ss.sss
THH:mm:ss.sssZ

THH:mm:ss
THH:mm:ssZ

THH:mm
THH:mmZ
```

它们包括以下部分：

+   `T`是格式时间部分的前缀（字面上的`T`，而不是数字）。

+   `HH`指的是小时，从 00 到 23。您可以使用 24 作为`HH`的值（指的是第二天的 00 小时），但是接下来的所有部分都必须为 0。

+   `mm` 表示分钟，从 00 到 59。

+   `ss` 表示秒，从 00 到 59。

+   `sss` 表示毫秒，从 000 到 999。

+   `Z`指的是时区，以下两者之一：

+   ``Z``表示 UTC

+   ``+``或``-``后跟时间``hh:mm``

一些 JavaScript 引擎允许您仅指定时间（其他需要日期）：

```js
> new Date('T13:17')
Date {Thu Jan 01 1970 13:17:00 GMT+0100 (CET)}
```

### 日期时间格式

日期格式和时间格式也可以结合使用。在日期时间格式中，您可以使用日期或日期和时间（或在某些引擎中仅使用时间）。例如：

```js
> new Date('2001-02-22T13:17')
Date {Thu Feb 22 2001 13:17:00 GMT+0100 (CET)}
```

## 时间值：自 1970-01-01 以来的毫秒数

日期 API 称之为`time`的东西在 ECMAScript 规范中被称为*时间值*。它是一个原始数字，以自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数编码日期。每个日期对象都将其状态存储为时间值，在内部属性`[[PrimitiveValue]]`中（与包装构造函数`Boolean`，`Number`和`String`的实例用于存储其包装的原始值的相同属性）。

### 警告

时间值中忽略了闰秒。

以下方法适用于时间值：

+   `new Date(timeValue)` 使用时间值创建日期。

+   `Date.parse(dateTimeString)`解析带有日期时间字符串的字符串并返回时间值。

+   `Date.now()`返回当前日期时间作为时间值。

+   `Date.UTC(year, month, date?, hours?, minutes?, seconds?, milliseconds?)` 解释参数相对于 UTC 并返回时间值。

+   `Date.prototype.getTime()` 返回接收器中存储的时间值。

+   `Date.prototype.setTime(timeValue)`根据指定的时间值更改日期。

+   `Date.prototype.valueOf()`返回接收者中存储的时间值。该方法确定了如何将日期转换为原始值，如下一小节所述。

JavaScript 整数的范围（53 位加上一个符号）足够大，可以表示从 1970 年前约 285,616 年开始到 1970 年后约 285,616 年结束的时间跨度。

以下是将日期转换为时间值的几个示例：

```js
> new Date('1970-01-01').getTime()
0
> new Date('1970-01-02').getTime()
86400000
> new Date('1960-01-02').getTime()
-315532800000
```

`Date`构造函数使您能够将时间值转换为日期：

```js
> new Date(0)
Date {Thu Jan 01 1970 01:00:00 GMT+0100 (CET)}
> new Date(24 * 60 * 60 * 1000)  // 1 day in ms
Date {Fri Jan 02 1970 01:00:00 GMT+0100 (CET)}
> new Date(-315532800000)
Date {Sat Jan 02 1960 01:00:00 GMT+0100 (CET)}
```

### 将日期转换为数字

通过`Date.prototype.valueOf()`将日期转换为数字，返回一个时间值。这使您能够比较日期：

```js
> new Date('1980-05-21') > new Date('1980-05-20')
true
```

您也可以进行算术运算，但要注意闰秒被忽略：

```js
> new Date('1980-05-21') - new Date('1980-05-20')
86400000
```

### 警告

使用加号运算符（`+`）将日期加到另一个日期或数字会得到一个字符串，因为将日期转换为原始值的默认方式是将日期转换为字符串（请参阅[加号运算符（+）](ch09.html#plus_operator "The Plus Operator (+)")了解加号运算符的工作原理）：

```js
> new Date('2024-10-03') + 86400000
'Thu Oct 03 2024 02:00:00 GMT+0200 (CEST)86400000'
> new Date(Number(new Date('2024-10-03')) + 86400000)
Fri Oct 04 2024 02:00:00 GMT+0200 (CEST)
```

