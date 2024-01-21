# 第八章：值

> 原文：[8. Values](https://exploringjs.com/es5/ch08.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 拥有我们所期望的大多数编程语言的值：布尔值、数字、字符串、数组等。JavaScript 中的所有正常值都有*属性*。⁷ 每个属性都有一个*键*（或*名称*）和一个*值*。你可以把属性看作记录的字段。你可以使用点（`.`）运算符来访问属性：

```js
> var obj = {}; // create an empty object
> obj.foo = 123;  // write property
123
> obj.foo  // read property
123
> 'abc'.toUpperCase()  // call method
'ABC'
```

## JavaScript 的类型系统

本章概述了 JavaScript 的类型系统。

### JavaScript 的类型

根据 ECMAScript 语言规范的第八章，JavaScript 只有六种类型：

> ECMAScript 语言类型对应于由 ECMAScript 程序员直接使用 ECMAScript 语言进行操作的值。ECMAScript 语言类型包括：
>
> +   未定义、空值
> +   布尔值、字符串、数字和
> +   对象

因此，构造函数在技术上并没有引入新的类型，尽管它们被认为有实例。

### 静态与动态

在语言语义和类型系统的背景下，“静态”通常意味着“在编译时”或“在不运行程序时”，而“动态”意味着“在运行时”。

### 静态类型检查与动态类型检查

在静态类型语言中，变量、参数和对象的成员（JavaScript 称之为属性）在编译时就已经知道类型。编译器可以使用这些信息进行类型检查和优化编译后的代码。

即使在静态类型语言中，变量也有动态类型，即运行时变量值的类型。动态类型可以与静态类型不同。例如（Java）：

```js
Object foo = "abc";
```

`foo`的静态类型是`Object`；它的动态类型是`String`。

JavaScript 是动态类型的；变量的类型通常在编译时不知道。

### 静态类型检查与动态类型检查

如果你有类型信息，你可以检查在操作中使用的值（调用函数、应用运算符等）是否具有正确的类型。在静态类型检查的语言中，这种检查是在编译时进行的，而在动态类型检查的语言中是在运行时进行的。一种语言可以同时进行静态类型检查和动态类型检查。如果检查失败，通常会得到某种错误或异常。

JavaScript 执行一种非常有限的动态类型检查：

```js
> var foo = null;
> foo.prop
TypeError: Cannot read property 'prop' of null
```

然而，大多数情况下，事情会悄悄地失败或者成功。例如，如果你访问一个不存在的属性，你会得到值`undefined`：

```js
> var bar = {};
> bar.prop
undefined
```

### 强制转换

在 JavaScript 中，处理类型不匹配的值的主要方法是将其*强制转换*为正确的类型。*强制转换*意味着隐式类型转换。大多数操作数都会强制转换：

```js
> '3' * '4'
12
```

JavaScript 的内置转换机制仅支持`Boolean`，`Number`，`String`和`Object`类型。没有标准的方法将一个构造函数的实例转换为另一个构造函数的实例。

### 警告

术语*强类型*和*弱类型*没有[普遍有意义的定义](http://bit.ly/1oO7t1p)。它们被使用，但通常是不正确的。最好使用*静态类型*，*静态类型检查*等。

## 原始值与对象

JavaScript 在值之间做了一个相当任意的区分：

+   *原始值*是布尔值，数字，字符串，`null`和`undefined`。

+   所有其他值都是*对象*。

两者之间的主要区别在于它们的比较方式；每个对象都有唯一的身份，只有（严格）等于自己：

```js
> var obj1 = {};  // an empty object
> var obj2 = {};  // another empty object
> obj1 === obj2
false

> var obj3 = obj1;
> obj3 === obj1
true
```

相反，编码相同值的所有原始值被认为是相同的：

```js
> var prim1 = 123;
> var prim2 = 123;
> prim1 === prim2
true
```

以下两节详细解释了原始值和对象。

## 原始值

以下是所有的*原始值*（简称*原始值*）：

+   布尔值：`true`，`false`（参见[第十章](ch10.html "第十章。布尔值"））

+   数字：`1736`，`1.351`（参见[第十一章](ch11.html "第十一章。数字"））

+   字符串：`'abc'`，`"abc"`（参见[第十二章](ch12.html "第十二章。字符串"））

+   两个“非值”：`undefined`，`null`（参见[undefined 和 null](ch08.html#undefined_null "undefined 和 null"））

原始值具有以下特征：

按值比较

比较“内容”：

```js
> 3 === 3
true
> 'abc' === 'abc'
true
```

始终不可变

属性不能被更改，添加或删除：

```js
> var str = 'abc';

> str.length = 1; // try to change property `length`
> str.length      // ⇒ no effect
3

> str.foo = 3; // try to create property `foo`
> str.foo      // ⇒ no effect, unknown property
undefined
```

（读取未知属性总是返回`undefined`。）

一组固定的类型

您不能定义自己的原始类型。

## 对象

所有非原始值都是*对象*。最常见的对象类型是：

+   *普通对象*（构造函数`Object`）可以通过*对象字面量*（参见[第十七章](ch17_split_000.html "第十七章。对象和继承"））创建：

    ```js
    {
        firstName: 'Jane',
        lastName: 'Doe'
    }
    ```

前面的对象有两个属性：属性`firstName`的值为`'Jane'`，属性`lastName`的值为`'Doe'`。

+   *数组*（构造函数`Array`）可以通过*数组字面量*（参见[第十八章](ch18.html "第十八章。数组"））创建：

    ```js
    [ 'apple', 'banana', 'cherry' ]
    ```

前面的数组有三个元素，可以通过数字索引访问。例如，'apple'的索引是 0。

+   *正则表达式*（构造函数`RegExp`）可以通过*正则表达式字面量*（参见[第十九章](ch19.html "第十九章。正则表达式"））创建：

    ```js
    /^a+b+$/
    ```

对象具有以下特征：

按引用比较

比较身份；每个对象都有自己的身份：

```js
> {} === {}  // two different empty objects
false

> var obj1 = {};
> var obj2 = obj1;
> obj1 === obj2
true
```

默认可变

通常可以自由更改，添加和删除属性（参见[点运算符（.）：通过固定键访问属性](ch17_split_000.html#dot_operator "点运算符（.）：通过固定键访问属性"））：

```js
> var obj = {};
> obj.foo = 123; // add property `foo`
> obj.foo
123
```

用户可扩展

构造函数（参见[第 3 层：构造函数-实例的工厂](ch17_split_001.html#constructors "第 3 层：构造函数-实例的工厂"）可以看作是自定义类型的实现（类似于其他语言中的类）。

## undefined 和 null

JavaScript 有两个“非值”，表示缺少信息，`undefined`和`null`：

+   `undefined`表示“没有值”（既不是原始值也不是对象）。未初始化的变量，缺少的参数和缺少的属性都具有该非值。如果没有明确返回任何内容，函数会隐式返回它。

+   `null`表示“没有对象”。它用作一个非值，期望一个对象（作为参数，在对象链中的成员等）。

`undefined`和`null`是唯一的值，任何属性访问都会导致异常：

```js
> function returnFoo(x) { return x.foo }

> returnFoo(true)
undefined
> returnFoo(0)
undefined

> returnFoo(null)
TypeError: Cannot read property 'foo' of null
> returnFoo(undefined)
TypeError: Cannot read property 'foo' of undefined
```

`undefined`有时也被用作指示不存在的元值。相比之下，`null`表示空。例如，JSON 节点访问者（请参阅[通过节点访问者转换数据](ch22.html#node_visitors "通过节点访问者转换数据")）返回：

+   `undefined`用于删除对象属性或数组元素

+   将属性或元素设置为`null`

### 未定义和 null 的出现

在这里，我们回顾了`undefined`和`null`出现的各种情况。

#### 未定义的出现

未初始化的变量是`undefined`：

```js
> var foo;
> foo
undefined
```

缺少参数是`undefined`：

```js
> function f(x) { return x }
> f()
undefined
```

如果读取不存在的属性，则会得到`undefined`：

```js
> var obj = {}; // empty object
> obj.foo
undefined
```

如果没有明确返回，函数会隐式返回`undefined`：

```js
> function f() {}
> f()
undefined

> function g() { return; }
> g()
undefined
```

#### null 的出现

+   `null`是原型链中的最后一个元素（一系列对象的链；请参阅[第 2 层：对象之间的原型关系](ch17_split_000.html#prototype_relationship "第 2 层：对象之间的原型关系")）：

    ```js
    > Object.getPrototypeOf(Object.prototype)
    null
    ```

+   如果字符串中的正则表达式没有匹配项，则`RegExp.prototype.exec()`将返回`null`：

    ```js
    > /x/.exec('aaa')
    null
    ```

### 检查未定义或 null

在接下来的几节中，我们将回顾如何分别检查`undefined`和`null`，或者检查它们是否存在。

#### 检查 null

您可以通过严格相等来检查`null`：

```js
if (x === null) ...
```

#### 检查未定义

严格相等（`===`）是检查`undefined`的规范方式：

```js
if (x === undefined) ...
```

您还可以通过`typeof`运算符（[typeof：对基元进行分类](ch09.html#typeof "typeof：对基元进行分类")）来检查`undefined`，但通常应使用前面提到的方法。

#### 检查未定义或 null

大多数函数允许您通过`undefined`或`null`指示缺少值。检查它们两者之一的一种方法是通过显式比较：

```js
// Does x have a value?
if (x !== undefined && x !== null) {
    ...
}
// Is x a non-value?
if (x === undefined || x === null) {
    ...
}
```

另一种方法是利用`undefined`和`null`都被视为`false`的事实（请参阅[真值和假值](ch10.html#truthy_falsy "真值和假值")）：

```js
// Does x have a value (is it truthy)?
if (x) {
    ...
}
// Is x falsy?
if (!x) {
    ...
}
```

### 警告

`false`，`0`，`NaN`和`''`也被视为`false`。

### 未定义和 null 的历史

单个非值可以扮演`undefined`和`null`的角色。为什么 JavaScript 有两个这样的值？原因是历史性的。

JavaScript 采用了 Java 对值进行分区为基元和对象的方法。它还使用了 Java 的“不是对象”的值，`null`。遵循 C（但不是 Java）所设定的先例，如果强制转换为数字，则`null`变为 0：

```js
> Number(null)
0
> 5 + null
5
```

请记住，JavaScript 的第一个版本没有异常处理。因此，未初始化的变量和丢失的属性等特殊情况必须通过一个值来指示。`null`本来是一个不错的选择，但当时 Brendan Eich 想要避免两件事：

+   该值不应该具有引用的含义，因为它不仅仅是关于对象。

+   该值不应该强制转换为 0，因为这样会使错误更难以发现。

因此，Eich 将`undefined`作为语言中的另一个非值。它强制转换为`NaN`：

```js
> Number(undefined)
NaN
> 5 + undefined
NaN
```

### 更改未定义

`undefined`是全局对象的一个属性（因此是全局变量；请参阅[全局对象](ch16.html#global_object "全局对象")）。在 ECMAScript 3 中，读取`undefined`时必须采取预防措施，因为很容易意外更改其值。在 ECMAScript 5 中，这是不必要的，因为`undefined`是只读的。

为了防止更改`undefined`，有两种流行的技术（它们对于旧的 JavaScript 引擎仍然相关）：

技术 1

屏蔽全局`undefined`（可能具有错误的值）：

```js
(function (undefined) {
    if (x === undefined) ...  // safe now
}());  // don’t hand in a parameter
```

在上述代码中，`undefined`保证具有正确的值，因为它是一个参数，其值未由函数调用提供。

技术 2

与始终（正确的）`undefined`相比，`void 0`（请参阅[void 运算符](ch09.html#void_operator "void 运算符")）：

```js
if (x === void 0)  // always safe
```

## 基元的包装对象

布尔值、数字和字符串这三种原始类型都有对应的构造函数：`Boolean`、`Number`、`String`。它们的实例（称为*包装对象*）包含（*包装*）原始值。这些构造函数可以以两种方式使用：

+   作为构造函数，它们创建的对象与它们包装的原始值大部分不兼容：

    ```js
    > typeof new String('abc')
    'object'
    > new String('abc') === 'abc'
    false
    ```

+   作为函数，它们将值转换为相应的原始类型（见[转换为布尔值、数字、字符串和对象的函数](ch08.html#convert_to_primitive "转换为布尔值、数字、字符串和对象的函数"））。这是推荐的转换方法：

    ```js
    > String(123)
    '123'
    ```

### 提示

最好的做法是避免使用包装对象。通常情况下，您不需要它们，因为对象不能做的事情原始值都可以做（除了被改变）。（这与 Java 不同，JavaScript 从中继承了原始值和对象之间的差异！）

### 包装对象与原始值不同

诸如`'abc'`之类的原始值与诸如`new String('abc')`之类的包装实例在根本上是不同的：

```js
> typeof 'abc'  // a primitive value
'string'
> typeof new String('abc')  // an object
'object'
> 'abc' instanceof String  // never true for primitives
false
> 'abc' === new String('abc')
false
```

包装实例是对象，JavaScript 中没有办法比较对象，甚至不能通过宽松相等`==`进行比较（见[相等运算符：===与==](ch09.html#equality_operators "相等运算符：===与=="））：

```js
> var a = new String('abc');
> var b = new String('abc');
> a == b
false
```

### 包装和解包原始值

包装对象的一个用例是：您想要向原始值添加属性。然后您包装原始值并向包装对象添加属性。在使用之前，您需要解包该值。

通过调用包装构造函数来包装原始值：

```js
new Boolean(true)
new Number(123)
new String('abc')
```

通过调用方法`valueOf()`来解包原始值。所有对象都有这个方法（如[转换为原始值](ch17_split_001.html#Object.prototype.valueOf "转换为原始值"）中所讨论的）：

```js
> new Boolean(true).valueOf()
true
> new Number(123).valueOf()
123
> new String('abc').valueOf()
'abc'
```

将包装对象转换为原始值可以正确提取数字和字符串，但不能提取布尔值：

```js
> Boolean(new Boolean(false))  // does not unwrap
true
> Number(new Number(123))  // unwraps
123
> String(new String('abc'))  // unwraps
'abc'
```

这是在[转换为布尔值](ch10.html#toboolean "转换为布尔值"）中解释的原因。

### 原始值从包装对象中借用它们的方法

原始值没有自己的方法，而是从包装对象中借用它们：

```js
> 'abc'.charAt === String.prototype.charAt
true
```

松散模式和严格模式以不同的方式处理这种借用。在松散模式下，原始值会即时转换为包装对象：

```js
String.prototype.sloppyMethod = function () {
    console.log(typeof this); // object
    console.log(this instanceof String); // true
};
''.sloppyMethod(); // call the above method
```

在严格模式下，会透明地使用包装原型中的方法：

```js
String.prototype.strictMethod = function () {
    'use strict';
    console.log(typeof this); // string
    console.log(this instanceof String); // false
};
''.strictMethod(); // call the above method
```

## 类型强制

*类型强制*意味着将一个类型的值隐式转换为另一个类型的值。JavaScript 的大多数运算符、函数和方法都会将操作数和参数强制转换为它们需要的类型。例如，乘法运算符（`*`）的操作数会被强制转换为数字：

```js
> '3' * '4'
12
```

另一个例子，如果操作数之一是字符串，加号运算符（`+`）会将另一个操作数转换为字符串：

```js
> 3 + ' times'
'3 times'
```

### 类型强制可以隐藏错误

因此，JavaScript 很少抱怨值的类型错误。例如，程序通常会将用户输入（来自在线表单或 GUI 小部件）作为字符串接收，即使用户输入的是一个数字。如果您将一个数字作为字符串处理，您将不会收到警告，只会得到意外的结果。例如：

```js
var formData = { width: '100' };

// You think formData.width is a number
// and get unexpected results
var w = formData.width;
var outer = w + 20;

// You expect outer to be 120, but it’s not
console.log(outer === 120);  // false
console.log(outer === '10020');  // true
```

在诸如前面的情况下，您应该尽早将其转换为适当的类型：

```js
var w = Number(formData.width);
```

### 转换为布尔值、数字、字符串和对象的函数

以下函数是将值转换为布尔值、数字、字符串或对象的首选方法：

`Boolean()`（见[转换为布尔值](ch10.html#toboolean "转换为布尔值"））

将一个值转换为布尔值。以下值被转换为`false`；它们被称为“假值”：

+   `undefined`，`null`

+   `false`

+   `0`，`NaN`

+   `''`

所有其他值都被视为“真值”，并转换为`true`（包括所有对象！）。

`Number()`（见[转换为数字](ch11.html#tonumber "转换为数字"））

将一个值转换为数字：

+   `undefined`变为`NaN`。

+   `null`变为`0`。

+   `false`变为`0`，`true`变为`1`。

+   字符串被解析。

+   首先将对象转换为原始值（稍后讨论），然后将其转换为数字。

`String()`（参见[转换为字符串](ch12.html#tostring "转换为字符串")）

将值转换为字符串。对于所有原始值，它都有明显的结果。例如：

```js
> String(null)
'null'
> String(123.45)
'123.45'
> String(false)
'false'
```

首先将对象转换为原始值（稍后讨论），然后将其转换为字符串。

`Object()`（参见[将任何值转换为对象](ch17_split_000.html#toobject "将任何值转换为对象")）

将对象转换为它们自己，将`undefined`和`null`转换为空对象，将原始值转换为包装的原始值。例如：

```js
> var obj = { foo: 123 };
> Object(obj) === obj
true

> Object(undefined)
{}
> Object('abc') instanceof String
true
```

请注意，`Boolean()`、`Number()`、`String()`和`Object()`都被作为函数调用。你通常不会将它们用作构造函数。然后它们创建自己的实例（参见[原始值的包装对象](ch08.html#wrapper_objects "原始值的包装对象")）。

### 算法：ToPrimitive()——将值转换为原始值

要将值转换为数字或字符串，首先将其转换为任意原始值，然后将其转换为最终类型（如[用于转换为布尔值、数字、字符串和对象的函数](ch08.html#convert_to_primitive "用于转换为布尔值、数字、字符串和对象的函数")中所讨论的）。

ECMAScript 规范有一个内部函数`ToPrimitive()`（无法从 JavaScript 中访问），它执行这种转换。了解`ToPrimitive()`使你能够配置对象如何转换为数字和字符串。它有以下签名：

```js
ToPrimitive(input, PreferredType?)
```

可选参数`PreferredType`指示转换的最终类型：它可以是`Number`或`String`，具体取决于`ToPrimitive()`的结果将被转换为数字还是字符串。

如果`PreferredType`是`Number`，则执行以下步骤：

1.  如果`input`是原始的，就返回它（没有更多的事情要做了）。

1.  否则，`input`是一个对象。调用`input.valueOf()`。如果结果是原始的，就返回它。

1.  否则，调用`input.toString()`。如果结果是原始的，就返回它。

1.  否则，抛出`TypeError`（表示无法将`input`转换为原始值）。

如果`PreferredType`是`String`，则步骤 2 和 3 会交换。`PreferredType`也可以省略；然后它被认为是日期的`String`，而对于所有其他值，则被认为是`Number`。这就是运算符`+`和`==`调用`ToPrimitive()`的方式。

#### 示例：ToPrimitive()的实际应用

`valueOf()`的默认实现返回`this`，而`toString()`的默认实现返回类型信息：

```js
> var empty = {};
> empty.valueOf() === empty
true
> empty.toString()
'[object Object]'
```

因此，`Number()`跳过`valueOf()`，将`toString()`的结果转换为数字；也就是说，它将`'[object Object]'`转换为`NaN`：

```js
> Number({})
NaN
```

以下对象自定义了`valueOf()`，它影响`Number()`，但对于`String()`没有任何改变：

```js
> var n = { valueOf: function () { return 123 } };
> Number(n)
123
> String(n)
'[object Object]'
```

以下对象自定义了`toString()`。因为结果可以转换为数字，所以`Number()`可以返回一个数字：

```js
> var s = { toString: function () { return '7'; } };
> String(s)
'7'
> Number(s)
7
```

* * *

⁷ 从技术上讲，原始值没有自己的属性，它们从包装构造函数中借用。但这是在幕后进行的，所以你通常看不到它。

