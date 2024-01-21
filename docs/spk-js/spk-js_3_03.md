# 第九章 运算符

> 原文：[9. Operators](https://exploringjs.com/es5/ch09.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章概述了运算符。

## 运算符和对象

所有运算符都会强制转换（如[类型强制转换](ch08.html#type_coercion "类型强制转换")中所讨论的）它们的操作数为适当的类型。大多数运算符只能处理原始值（例如，算术运算符和比较运算符）。这意味着在对它们进行任何操作之前，对象都会被转换为原始值。其中一个不幸的例子是加号运算符，许多语言用它来进行数组连接。然而，在 JavaScript 中并非如此，这个运算符会将数组转换为字符串并将它们连接起来：

```js
> [1, 2] + [3]
'1,23'
> String([1, 2])
'1,2'
> String([3])
'3'
```

### 注意

在 JavaScript 中没有办法重载或自定义运算符，甚至没有相等性。

## 赋值运算符

有几种使用普通赋值运算符的方法：

`x = value`

分配给先前声明的变量`x`

`var x = value`

将变量声明与赋值结合

`obj.propKey = value`

设置属性

`obj['propKey'] = value`

设置属性

`arr[index] = value`

设置数组元素⁸

赋值是一个求值为赋值的表达式。这允许您链接赋值。例如，以下语句将`0`分配给`y`和`x`：

```js
x = y = 0;
```

### 复合赋值运算符

*复合赋值运算符*写为`op=`，其中`op`是几个二进制运算符之一，`=`是赋值运算符。以下两个表达式是等价的：

```js
myvar op= value
myvar = myvar op value
```

换句话说，复合赋值运算符`op=`将`op`应用于两个操作数，并将结果分配给第一个操作数。让我们看一个使用加法运算符（`+`）的复合赋值的示例：

```js
> var x = 2;
> x += 3
5
> x
5
```

以下都是复合赋值运算符：

+   算术运算（参见[算术运算符](ch11.html#arithmetic_operators "算术运算符")）：`*=`, `/=`, `%=`, `+=`, `-=`

+   按位操作（参见[二进制按位运算符](ch11.html#binary_bitwise_operators "二进制按位运算符")）：`<<=`, `>>=`, `>>>=`, `&=`, `^=`, `|=`

+   字符串连接（参见[连接：加号（+）运算符](ch12.html#string_plus "连接：加号（+）运算符")）：`+=`

## 相等运算符：===与==

JavaScript 有两种确定两个值是否相等的方法：

+   严格相等（`===`）和严格不等（`!==`）仅认为具有相同类型的值相等。

+   正常（或“宽松”）相等（`==`）和不等（`!=`）在比较之前尝试转换不同类型的值，就像严格（不）相等一样。

宽松相等在两个方面存在问题。首先，它的转换方式令人困惑。其次，由于运算符如此宽容，类型错误可能会隐藏更长时间。

始终使用严格相等，避免宽松相等。只有在您想知道为什么应该避免它时，才需要了解后者。

相等是不可定制的。JavaScript 中的运算符不能被重载，也不能定制相等的工作方式。有一些操作，您经常需要影响比较——例如，`Array.prototype.sort()`（参见[排序和反转元素（破坏性）](ch18.html#Array.prototype.sort "排序和反转元素（破坏性）")）。该方法可选择接受一个回调，该回调执行数组元素之间的所有比较。

### 严格相等（===, !==）

具有不同类型的值永远不会严格相等。如果两个值具有相同的类型，则以下断言成立：

+   `undefined === undefined`

+   `null === null`

+   两个数字：

    ```js
    x === x  // unless x is NaN
    +0 === -0
    NaN !== NaN  // read explanation that follows
    ```

+   两个布尔值，两个字符串：显而易见的结果

+   两个对象（包括数组和函数）：`x === y`当且仅当`x`和`y`是同一个对象时；也就是说，如果要比较不同的对象，您必须实现自己的比较算法： 

    ```js
    > var b = {}, c = {};
    > b === c
    false
    > b === b
    true
    ```

+   其他一切：不严格相等。

#### 陷阱：NaN

特殊的数字值`NaN`（参见[NaN](ch11.html#nan "NaN")）不等于自身：

```js
> NaN === NaN
false
```

因此，您需要使用其他方法来检查它，这些方法在[陷阱：检查值是否为 NaN](ch11.html#isNaN "陷阱：检查值是否为 NaN")中有描述。

#### 严格不等 (!==)

严格不等比较：

```js
x !== y
```

等同于严格相等比较的否定：

```js
!(x === y)
```

### 正常（宽松）相等（==, !=）

通过正常相等比较的算法工作如下。如果两个操作数具有相同的类型（六种规范类型之一——Undefined、Null、Boolean、Number、String 和 Object），则通过严格相等比较它们。

否则，如果操作数是：

1.  `undefined`和`null`，那么它们被认为是宽松相等的：

```js
> undefined == null
true
```

1.  一个字符串和一个数字，然后将字符串转换为数字，并通过严格相等比较两个操作数。

1.  一个布尔值和一个非布尔值，然后将布尔值转换为数字并进行宽松比较（再次）。

1.  一个对象和一个数字或字符串，然后尝试将对象转换为原始值（通过[算法：ToPrimitive()—将值转换为原始值](ch08.html#toprimitive "算法：ToPrimitive()—将值转换为原始值")中描述的算法）并进行宽松比较。

否则——如果上述任何情况都不适用——宽松比较的结果是`false`。

#### 宽松不等号（!=）

一个不等式比较：

```js
x != y
```

等同于等式比较的否定：

```js
!(x == y)
```

#### 陷阱：宽松相等与转换为布尔值不同

第三步意味着相等和转换为布尔值（参见[转换为布尔值](ch10.html#toboolean "转换为布尔值")）的工作方式不同。如果转换为布尔值，大于 1 的数字变为`true`（例如，在`if`语句中）。但这些数字并不宽松相等于`true`。注释解释了结果是如何计算的：

```js
> 2 == true  // 2 === 1
false
> 2 == false  // 2 === 0
false

> 1 == true  // 1 === 1
true
> 0 == false  // 0 === 0
true
```

同样，虽然空字符串等于`false`，但并非所有非空字符串都等于`true`：

```js
> '' == false   // 0 === 0
true
> '1' == true  // 1 === 1
true
> '2' == true  // 2 === 1
false
> 'abc' == true  // NaN === 1
false
```

#### 陷阱：宽松相等和字符串

一些宽松性可能是有用的，这取决于你的需求：

```js
> 'abc' == new String('abc')  // 'abc' == 'abc'
true
> '123' == 123  // 123 === 123
true
```

其他情况可能有问题，因为 JavaScript 如何将字符串转换为数字（参见[转换为数字](ch11.html#tonumber "转换为数字")）：

```js
> '\n\t123\r ' == 123  // usually not OK
true
> '' == 0  // 0 === 0
true
```

#### 陷阱：宽松相等和对象

如果你将一个对象与一个非对象进行比较，它会被转换为原始值，这会导致奇怪的结果：

```js
> {} == '[object Object]'
true
> ['123'] == 123
true
> [] == 0
true
```

然而，只有两个对象是相等的，如果它们是同一个对象。这意味着你不能真正比较两个包装对象：

```js
> new Boolean(true) === new Boolean(true)
false
> new Number(123) === new Number(123)
false
> new String('abc') == new String('abc')
false
```

### 没有`==`的有效用例

有时你会读到关于宽松相等（`==`）的有效用例。本节列出了它们，并指出了更好的替代方案。

#### 用例：检查 undefined 或 null

以下比较确保`x`既不是`undefined`也不是`null`：

```js
if (x != null) ...
```

虽然这是一种简洁的写法，但它会让初学者感到困惑，而专家也无法确定它是否是打字错误。因此，如果你想检查`x`是否有值，请使用标准的真值检查（在[真值和假值](ch10.html#truthy_falsy "真值和假值")中介绍）：

```js
if (x) ...
```

如果你想更精确，你应该对两个值进行显式检查：

```js
if (x !== undefined && x !== null) ...
```

#### 用例：处理字符串中的数字

如果你不确定一个值`x`是一个数字还是一个数字字符串，你可以使用以下检查：

```js
if (x == 123) ...
```

前面的检查是为了确保`x`是`123`或`'123'`。同样，这是非常紧凑的，而且最好是明确的：

```js
if (Number(x) === 123) ...
```

#### 用例：比较包装实例和原始值

宽松相等允许你比较原始值和包装原始值：

```js
> 'abc' == new String('abc')
true
```

有三个理由反对这种方法。首先，宽松相等在包装原始值之间不起作用：

```js
> new String('abc') == new String('abc')
false
```

其次，你应该无论如何避免使用包装器。第三，如果你使用它们，最好是明确的：

```js
if (wrapped.valueOf() === 'abc') ...
```

## 排序运算符

JavaScript 知道以下排序运算符：

+   小于（`<`）

+   小于或等于（`<=`）

+   大于（`>`）

+   大于或等于（`>=`）

这些运算符适用于数字和字符串：

```js
> 7 >= 5
true
> 'apple' < 'orange'
true
```

对于字符串来说，它们并不是非常有用，因为它们区分大小写，而且不能很好地处理重音等特性（有关详细信息，请参见[比较字符串](ch12.html#comparing_strings "比较字符串")）。

### 算法

你评估一个比较：

```js
x < y
```

通过以下步骤：

1.  确保两个操作数都是原始值。对象`obj`通过内部操作`ToPrimitive(obj, Number)`（参见[算法：ToPrimitive()—将值转换为原始值](ch08.html#toprimitive "算法：ToPrimitive()—将值转换为原始值")）转换为原始值，该操作调用`obj.valueOf()`和可能的`obj.toString()`来实现。

1.  如果两个操作数都是字符串，那么通过按字典顺序比较表示字符串的 JavaScript 字符的 16 位代码单元（参见第二十四章）来比较它们。

1.  否则，将两个操作数转换为数字并进行数字比较。

其他排序运算符类似处理。

## 加号运算符（+）

粗略地说，加号运算符检查它的操作数。 如果其中一个是字符串，则另一个也被转换为字符串，并且两者被连接：

```js
> 'foo' + 3
'foo3'
> 3 + 'foo'
'3foo'

> 'Colors: ' + [ 'red', 'green', 'blue' ]
'Colors: red,green,blue'
```

否则，两个操作数都转换为数字（参见[转换为数字](ch11.html#tonumber "转换为数字")）并相加：

```js
> 3 + 1
4
> 3 + true
4
```

这意味着评估的顺序很重要：

```js
> 'foo' + (1 + 2)
'foo3'
> ('foo' + 1) + 2
'foo12'
```

### 算法

你评估一个加法：

```js
value1 + value2
```

通过以下步骤进行： 

1.  确保两个操作数都是原始值。 对象`obj`通过内部操作`ToPrimitive(obj)`（参见[算法：ToPrimitive()—将值转换为原始值](ch08.html#toprimitive "算法：ToPrimitive()—将值转换为原始值")）转换为原始值，该操作调用`obj.valueOf()`和可能的`obj.toString()`来执行此操作。 对于日期，首先调用`obj.toString()`。

1.  如果任一操作数是字符串，则将两者转换为字符串并返回结果的连接。

1.  否则，将两个操作数转换为数字，并返回结果的总和。

## 布尔值和数字的运算符

以下运算符只有单一类型的操作数，并且也产生该类型的结果。 它们在其他地方有所涉及。

布尔运算符：

+   二进制逻辑运算符（参见[二进制逻辑运算符：And (&&)和 Or (||)](ch10.html#binary_logical_operators "二进制逻辑运算符：And (&&)和 Or (||)")）：

    ```js
    x && y, x || y
    ```

+   逻辑非（参见[逻辑非(!)](ch10.html#logical_not "逻辑非(!)")）：

    ```js
    !x
    ```

数字运算符：

+   算术运算符（参见[算术运算符](ch11.html#arithmetic_operators "算术运算符"）：

    ```js
    x + y, x - y, x * y, x / y, x % y
    ++x, --x, x++, x--
    -x, +x
    ```

+   按位运算符（参见[按位运算符](ch11.html#bitwise_operators "按位运算符"）：

    ```js
    ~x
    x & y, x | y, x ^ y
    x << y, x >> y, x >>> y
    ```

## 特殊运算符

在这里，我们将回顾特殊运算符，即条件、逗号和`void`运算符。

### 条件运算符（？：）

条件运算符是一个表达式：

```js
«condition» ? «if_true» : «if_false»
```

如果条件为`true`，则结果为`if_true`； 否则，结果为`if_false`。 例如：

```js
var x = (obj ? obj.prop : null);
```

不需要在运算符周围加括号，但这样做会使其更易于阅读。

### 逗号运算符

```js
«left» , «right»
```

逗号运算符评估两个操作数并返回`right`的结果。 粗略地说，它对表达式做了分号对语句所做的事情。

这个例子演示了第二个操作数成为运算符的结果：

```js
> 123, 'abc'
'abc'
```

这个例子演示了两个操作数都被评估：

```js
> var x = 0;
> var y = (x++, 10);

> x
1
> y
10
```

逗号运算符很令人困惑。 最好不要聪明，而是在您可以的情况下编写两个单独的语句。

### void 运算符

`void`运算符的语法是：

```js
void «expr»
```

评估`expr`并返回`undefined`。 以下是一些例子：

```js
> void 0
undefined
> void (0)
undefined

> void 4+7  // same as (void 4)+7
NaN
> void (4+7)
undefined

> var x;
> x = 3
3
> void (x = 5)
undefined
> x
5
```

因此，如果您将`void`实现为一个函数，它看起来如下：

```js
function myVoid(expr) {
    return undefined;
}
```

`void`运算符与其操作数密切相关，因此根据需要使用括号。 例如，`void 4+7`绑定为`(void 4)+7`。

#### void 用于什么？

在 ECMAScript 5 下，`void`很少有用。 它的主要用例是：

`void 0`作为`undefined`的同义词

后者可以更改，而前者将始终具有正确的值。 但是，在 ECMAScript 5 下，`undefined`相对安全，这使得这种用例不那么重要（有关详细信息，请参见[更改 undefined](ch08.html#changing_undefined "更改 undefined")）。

丢弃表达式的结果

在某些情况下，返回`undefined`而不是表达式的结果很重要。 然后可以使用`void`来丢弃该结果。 其中一种情况涉及`javascript:` URL，应该避免使用链接，但对于书签很有用。 当您访问这些 URL 之一时，许多浏览器会用 URL 的“内容”评估结果替换当前文档，但前提是结果不是`undefined`。 因此，如果您想要打开一个新窗口而不更改当前显示的内容，可以执行以下操作：

```js
javascript:void window.open("http://example.com/")
```

前缀 IIFE

IIFE 必须被解析为表达式。确保这一点的几种方法之一是用`void`作为前缀（参见[IIFE 变体：前缀运算符](ch16.html#iife_prefix "IIFE Variation: Prefix Operators")）⁹

#### 为什么 JavaScript 有 void 运算符？

根据 JavaScript 的创始人 Brendan Eich，他将其添加到语言中以帮助处理`javascript:`链接（前面提到的用例之一）：

> 我在 Netscape 2 发布之前向 JS 添加了`void`运算符，以便轻松丢弃 javascript: URL 中的任何非 undefined 值。¹⁰

## 通过 typeof 和 instanceof 对值进行分类

如果你想对一个值进行分类，不幸的是你必须区分原始值和对象（参见第八章中的内容）：

+   typeof 运算符区分原始值和对象，并确定原始值的类型。

+   `instanceof`运算符确定一个对象是否是给定构造函数的实例。有关 JavaScript 中面向对象编程的更多信息，请参阅第十七章。

### typeof：对原始值进行分类

typeof 运算符：

```js
typeof «value»
```

返回描述`value`是什么类型的字符串。以下是一些例子：

```js
> typeof undefined
'undefined'
> typeof 'abc'
'string'
> typeof {}
'object'
> typeof []
'object'
```

`typeof`用于区分原始值和对象，并对原始值进行分类（`instanceof`无法处理原始值）。不幸的是，这个运算符的结果并不完全符合逻辑，而且只是松散地对应于 ECMAScript 规范的类型（在[JavaScript 的类型](ch08.html#javascript_types "JavaScript’s Types")中有解释）：

| 操作数 | 结果 |
| --- | --- |
| `undefined`，未声明的变量 | `'undefined'` |
| `null` | `'object'` |
| 布尔值 | `'boolean'` |
| 数值 | `'number'` |
| 字符串值 | `'string'` |
| 函数 | `'function'` |
| 所有其他正常值 | `'object'` |
| （引擎创建的值） | JavaScript 引擎允许创建值，对于这些值，`typeof`返回任意字符串（与表中列出的所有结果不同）。

#### 陷阱：typeof null

不幸的是，`typeof null`是`'object'`。这被认为是一个错误（`null`不是内部类型 Object 的成员），但无法修复，因为这样做会破坏现有的代码。因此，你必须谨慎处理`null`。例如，以下函数检查`value`是否是一个对象：

```js
function isObject(value) {
    return (value !== null
       && (typeof value === 'object'
           || typeof value === 'function'));
}
```

试一试：

```js
> isObject(123)
false
> isObject(null)
false
> isObject({})
true
```

#### typeof null 的历史

第一个 JavaScript 引擎将 JavaScript 值表示为 32 位字。这样的字的最低 3 位用作类型标记，以指示该值是对象、整数、双精度、字符串还是布尔值（正如你所看到的，即使这个早期引擎已经尽可能将数字存储为整数）。

对象的类型标记为 000。为了表示值`null`，引擎使用了机器语言的 NULL 指针，一个所有位都为零的字。`typeof`检查类型标记以确定值的类型，这就是为什么它报告`null`是一个对象的原因。¹¹

#### 检查变量是否存在

检查：

```js
typeof x === 'undefined'
```

有两种用例：

1.  它确定`x`是否`undefined`。

1.  它确定变量`x`是否存在。

以下是两种用例的示例：

```js
> var foo;
> typeof foo === 'undefined'
true

> typeof undeclaredVariable === 'undefined'
true
```

对于第一个用例，直接与`undefined`进行比较通常是更好的选择。但是，对于第二个用例，这种方法行不通。

```js
> var foo;
> foo === undefined
true

> undeclaredVariable === undefined
ReferenceError: undeclaredVariable is not defined
```

### instanceof：检查对象是否是给定构造函数的实例

`instanceof`运算符：

```js
«value» instanceof «Constr»
```

确定`value`是由构造函数`Constr`还是子构造函数创建的。以下是一些例子：

```js
> {} instanceof Object
true
> [] instanceof Array  // constructor of []
true
> [] instanceof Object  // super-constructor of []
true
```

预期的是，`instanceof`对非值`undefined`和`null`返回`false`：

```js
> undefined instanceof Object
false
> null instanceof Object
false
```

但对于所有其他原始值也是`false`：

```js
> 'abc' instanceof Object
false
> 123 instanceof Object
false
```

有关`instanceof`的详细信息，请参阅[instanceof 运算符](ch17_split_001.html#operator_instanceof "The instanceof Operator")。

## 对象运算符

以下三个运算符适用于对象。它们在其他地方有解释：

`new`（参见[第三层：构造函数——实例的工厂](ch17_split_001.html#constructors "第三层：构造函数——实例的工厂")）

调用构造函数，例如，`new Point(3, 5)`

`delete`（参见[删除属性](ch17_split_000.html#operator_delete "删除属性")）

删除属性，例如，`delete obj.prop`

`in`（参见[迭代和属性检测](ch17_split_000.html#iterate_and_detect_properties "迭代和属性检测")）

检查对象是否具有给定属性，例如，`'prop' in obj`

* * *

⁸ 严格来说，设置数组元素是设置属性的特例。

⁹ 感谢 Brandon Benvie (@benvie)，他告诉我如何使用`void`来进行 IIFEs。

¹⁰ 来源：[`en.wikipedia.org/wiki/Bookmarklet`](http://en.wikipedia.org/wiki/Bookmarklet)

¹¹ 感谢 Tom Schuster (@evilpies) 指引我到第一个 JavaScript 引擎的源代码。

