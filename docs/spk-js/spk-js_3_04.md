# 第十章：布尔值

> 原文：[10. Booleans](https://exploringjs.com/es5/ch10.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


原始布尔类型包括值`true`和`false`：

```js
> typeof false
'boolean'
> typeof true
'boolean'
```

## 转换为布尔值

值转换为布尔值的方式如下：

| 值 | 转换为布尔值 |
| --- | --- |
| `undefined` | `false` |
| `null` | `false` |
| 布尔值 | 与输入相同（无需转换） |
| 数字 | `0`、`NaN` → `false` |
|  | 其他数字 → `true` |
| 字符串 | `''` → `false` |
|  | 其他字符串 → `true` |
| 对象 | `true`（总是！） |

### 手动转换为布尔值

任何值都可以通过三种方式转换为布尔值：

| `Boolean(value)` | （作为函数调用，而不是构造函数） |
| `value ? true : false` |  |
| `!!value` | 单个`not`转换为取反的布尔值；使用两次进行非取反转换。 |

我更喜欢`Boolean()`，因为它更具描述性。以下是一些例子：

```js
> Boolean(undefined)
false
> Boolean(null)
false

> Boolean(0)
false
> Boolean(1)
true
> Boolean(2)
true

> Boolean('')
false
> Boolean('abc')
true
> Boolean('false')
true
```

### 真值和假值

在 JavaScript 需要布尔值的地方，你可以提供任何类型的值，它会自动转换为布尔值。因此，在 JavaScript 中有两组值：一组转换为`false`，而另一组转换为`true`。这些组被称为*假值*和*真值*。根据前面的表格，以下是所有的假值：

+   `undefined`、`null`

+   布尔值：`false`

+   数字：`0`、`NaN`

+   字符串：`''`

所有其他值，包括*所有*对象，甚至是空对象、空数组和`new Boolean(false)`，都是真值。因为`undefined`和`null`是假值，你可以使用`if`语句来检查变量`x`是否有值：

```js
if (x) {
    // x has a value
}
```

需要注意的是，前面的检查将所有假值解释为“没有值”，不仅仅是`undefined`和`null`。但如果你可以接受这个限制，你就可以使用一种简洁和成熟的模式。

#### 陷阱：所有对象都是真值

所有对象都是真值：

```js
> Boolean(new Boolean(false))
true
> Boolean([])
true
> Boolean({})
true
```

这与对象转换为数字或字符串的方式不同，你可以通过实现`valueOf()`和`toString()`方法来控制结果。

```js
> Number({ valueOf: function () { return 123 } })
123
> String({ toString: function () { return 'abc' } })
'abc'
```

#### 历史：为什么对象总是真值？

由于历史原因，布尔值的转换方式不同。在 ECMAScript 1 中，决定不允许对象配置该转换（例如，通过`toBoolean()`方法）。其理由是布尔运算符`||`和`&&`会保留其操作数的值。因此，如果你链式使用这些运算符，相同的值可能会被多次检查真值或假值。对于原始值来说，这些检查是廉价的，但如果对象能够配置它们的布尔值转换，那么对于对象来说将会很昂贵。ECMAScript 1 通过使对象始终为真值来避免这种成本。

## 逻辑运算符

在本节中，我们将介绍 And（&&）、Or（||）和 Not（!）逻辑运算符的基础知识。

### 二进制逻辑运算符：And（&&）和 Or（||）

二进制逻辑运算符有：

保持值不变

它们总是返回两个操作数中的一个，不会改变：

```js
> 'abc' || 123
'abc'
> false || 123
123
```

短路

如果第一个操作数已经确定了结果，则不会评估第二个操作数。例如（`console.log`的结果是`undefined`）：

```js
> true || console.log('Hello')
true
> false || console.log('Hello')
Hello
undefined
```

这是运算符的不常见行为。通常，在调用运算符之前会评估所有操作数（就像函数一样）。

### 逻辑与（&&）

如果第一个操作数可以转换为`false`，则返回它。否则，返回第二个操作数：

```js
> true && false
false
> false && 'def'
false
> '' && 'def'
''
> 'abc' && 'def'
'def'
```

### 逻辑或（||）

如果第一个操作数可以转换为`true`，则返回它。否则，返回第二个操作数：

```js
> true || false
true
> true || 'def'
true
> 'abc' || 'def'
'abc'
> '' || 'def'
'def'
```

#### 模式：提供默认值

有时会出现这样的情况：一个值（参数、函数的结果等）可以是非值（`undefined`、`null`）或实际值。如果要为前一种情况提供默认值，可以使用或运算符：

```js
theValue || defaultValue
```

前面的表达式在`theValue`为真值时求值为`theValue`，否则为`defaultValue`。通常的警告适用：如果`theValue`具有除`undefined`和`null`之外的假值，则也将返回`defaultValue`。让我们看看使用该模式的三个示例。

#### 示例 1：参数的默认值

函数`saveText()`的参数`text`是可选的，如果省略了，则应该是空字符串：

```js
function saveText(text) {
    text = text || '';
    ...
}
```

这是`||`作为默认运算符的最常见用法。有关可选参数的更多信息，请参阅[可选参数](ch15.html#optional_parameters "可选参数")。

#### 示例 2：属性的默认值

对象`options`可能有也可能没有属性`title`。如果缺少，则在设置标题时应使用值`'Untitled'`：

```js
setTitle(options.title || 'Untitled');
```

#### 示例 3：函数结果的默认值

函数`countOccurrences`计算`regex`在`str`中匹配的次数：

```js
function countOccurrences(regex, str) {
    // Omitted: check that /g is set for `regex`
    return (str.match(regex) || []).length;
}
```

问题在于`match()`（请参见[String.prototype.match: Capture Groups or Return All Matching Substrings](ch19.html#String.prototype.match "String.prototype.match: Capture Groups or Return All Matching Substrings")）要么返回一个数组，要么返回`null`。由于`||`，在后一种情况下使用了默认值。因此，您可以安全地在两种情况下访问属性`length`。

### 逻辑非（！）

逻辑非运算符`!`将其操作数转换为布尔值，然后对其取反：

```js
> !true
false
> !43
false
> !''
true
> !{}
false
```

## 相等运算符，排序运算符

其他运算符在其他地方有所涵盖：

+   相等运算符：`===`，`!==`，`==`，`!=`（参见[相等运算符：===与==](ch09.html#equality_operators "相等运算符：===与==")）

+   排序运算符：`>`，`>=`，`<`，`<=`（参见[排序运算符](ch09.html#ordering_operators "排序运算符")）

## 布尔函数

函数`Boolean`可以以两种方式调用：

`Boolean(value)`

作为普通函数，它将`value`转换为原始布尔值（请参见[转换为布尔值](ch10.html#toboolean "转换为布尔值")）：

```js
> Boolean(0)
false
> typeof Boolean(false)  // no change
'boolean'
```

`new Boolean(bool)`

作为构造函数，它创建了`Boolean`的新实例（参见[原始包装对象](ch08.html#wrapper_objects "原始包装对象")），一个将`bool`（在将其转换为布尔值后）包装起来的对象。例如：

```js
> typeof new Boolean(false)
'object'
```

前面的调用是常见的。

