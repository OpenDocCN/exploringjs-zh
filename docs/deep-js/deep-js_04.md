# 二、JavaScript 中的类型强制转换

> 原文：[`exploringjs.com/deep-js/ch_type-coercion.html`](https://exploringjs.com/deep-js/ch_type-coercion.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   2.1 什么是类型强制转换？

    +   2.1.1 处理类型强制转换

+   2.2 实现 ECMAScript 规范中的强制转换的操作

    +   2.2.1 转换为原始类型和对象

    +   2.2.2 转换为数值类型

    +   2.2.3 转换为属性键

    +   2.2.4 转换为数组索引

    +   2.2.5 转换为 Typed Array 元素

+   2.3 插曲：用 JavaScript 表达规范算法

+   2.4 强制转换算法示例

    +   2.4.1 `ToPrimitive()`

    +   2.4.2 `ToString()`和相关操作

    +   2.4.3 `ToPropertyKey()`

    +   2.4.4 `ToNumeric()`和相关操作

+   2.5 强制转换的操作

    +   2.5.1 加法运算符（`+`）

    +   2.5.2 抽象相等比较（`==`）

+   2.6 术语表：与类型转换相关的术语

* * *

在本章中，我们将深入探讨 JavaScript 中*类型强制转换*的作用。我们将相对深入地研究这个主题，例如，看看 ECMAScript 规范如何处理强制转换。

### 2.1 什么是类型强制转换？

每个操作（函数、运算符等）都期望其参数具有某些类型。如果一个值对于参数没有正确的类型，例如，函数的三种常见选项是：

1.  该函数可以抛出异常：

    ```js
    function multiply(x, y) {
     if (typeof x !== 'number' || typeof y !== 'number') {
     throw new TypeError();
     }
     // ···
    }
    ```

1.  该函数可以返回错误值：

    ```js
    function multiply(x, y) {
     if (typeof x !== 'number' || typeof y !== 'number') {
     return NaN;
     }
     // ···
    }
    ```

1.  该函数可以将其参数转换为有用的值：

    ```js
    function multiply(x, y) {
     if (typeof x !== 'number') {
     x = Number(x);
     }
     if (typeof y !== 'number') {
     y = Number(y);
     }
     // ···
    }
    ```

在（3）中，该操作执行隐式类型转换。这就是所谓的*类型强制转换*。

JavaScript 最初没有异常，这就是为什么它对大多数操作使用强制转换和错误值的原因：

```js
// Coercion
assert.equal(3 * true, 3);

// Error values
assert.equal(1 / 0, Infinity);
assert.equal(Number('xyz'), NaN);
```

然而，也有一些情况（特别是涉及较新功能时），如果参数没有正确的类型，它会抛出异常：

+   访问`null`或`undefined`的属性：

    ```js
    > undefined.prop
    TypeError: Cannot read property 'prop' of undefined
    > null.prop
    TypeError: Cannot read property 'prop' of null
    > 'prop' in null
    TypeError: Cannot use 'in' operator to search for 'prop' in null
    ```

+   使用符号：

    ```js
    > 6 / Symbol()
    TypeError: Cannot convert a Symbol value to a number
    ```

+   混合大整数和数字：

    ```js
    > 6 / 3n
    TypeError: Cannot mix BigInt and other types
    ```

+   调用或函数调用不支持该操作的值：

    ```js
    > 123()
    TypeError: 123 is not a function
    > (class {})()
    TypeError: Class constructor  cannot be invoked without 'new'

    > new 123
    TypeError: 123 is not a constructor
    > new (() => {})
    TypeError: (intermediate value) is not a constructor
    ```

+   更改只读属性（在严格模式下会抛出）：

    ```js
    > 'abc'.length = 1
    TypeError: Cannot assign to read only property 'length'
    > Object.freeze({prop:3}).prop = 1
    TypeError: Cannot assign to read only property 'prop'
    ```

#### 2.1.1 处理类型强制转换

处理强制转换的两种常见方法是：

+   调用者可以显式转换值，使其具有正确的类型。例如，在以下交互中，我们想要将两个编码为字符串的数字相乘：

    ```js
    let x = '3';
    let y = '2';
    assert.equal(Number(x) * Number(y), 6);
    ```

+   调用者可以让操作为其进行转换：

    ```js
    let x = '3';
    let y = '2';
    assert.equal(x * y, 6);
    ```

我通常更喜欢前者，因为它澄清了我的意图：我希望`x`和`y`不是数字，但想要将两个数字相乘。

### 2.2 实现 ECMAScript 规范中的强制转换的操作

以下各节描述了 ECMAScript 规范中使用的最重要的内部函数，用于将实际参数转换为期望的类型。

例如，在 TypeScript 中，我们会这样写：

```js
function isNaN(number: number) {
 // ···
}
```

在规范中，这看起来像[如下](https://tc39.es/ecma262/#sec-multiplicative-operators-runtime-semantics-evaluation)（转换为 JavaScript，以便更容易理解）：

```js
function isNaN(number) {
 let num = ToNumber(number);
 // ···
}
```

#### 2.2.1 转换为原始类型和对象

每当期望原始类型或对象时，将使用以下转换函数：

+   `ToBoolean()`

+   `ToNumber()`

+   `ToBigInt()`

+   `ToString()`

+   `ToObject()`

这些内部函数在 JavaScript 中有非常相似的类似物：

```js
> Boolean(0)
false
> Boolean(1)
true

> Number('123')
123
```

在引入了与数字并存的 bigint 之后，规范通常在先前使用`ToNumber()`的地方使用`ToNumeric()`。继续阅读以获取更多信息。

#### 2.2.2 转换为数值类型

目前，JavaScript 有[两种内置的数值类型](https://tc39.es/ecma262/#sec-numeric-types)：number 和 bigint。

+   `ToNumeric()`返回一个数值`num`。它的调用者通常调用规范类型`num`的方法`mthd`：

    ```js
    Type(num)::mthd(···)
    ```

    除其他外，以下操作使用`ToNumeric`：

    +   前缀和后缀`++`运算符

    +   `*`运算符

+   `ToInteger(x)`在期望没有小数的数字时使用。结果的范围通常在之后进一步限制。

    +   它调用`ToNumber(x)`并删除小数（类似于`Math.trunc()`）。

    +   使用`ToInteger()`的操作：

        +   `Number.prototype.toString(radix?)`

        +   `String.prototype.codePointAt(pos)`

        +   `Array.prototype.slice(start, end)`

        +   等等。

+   `ToInt32()`，`ToUint32()`将数字强制转换为 32 位整数，并被位运算符使用（见 tbl. 1）。

    +   `ToInt32()`：有符号，范围[−2³¹, 2³¹−1]（包括限制）

    +   `ToUint32()`：无符号（因此有`U`），范围[0, 2³²−1]（包括限制）

表 1：位数运算符的操作数的强制转换（BigInt 运算符不限制位数）。

| 运算符 | 左操作数 | 右操作数 | 结果类型 |
| --- | --- | --- | --- |
| `<<` | `ToInt32()` | `ToUint32()` | `Int32` |
| signed `>>` | `ToInt32()` | `ToUint32()` | `Int32` |
| unsigned `>>>` | `ToInt32()` | `ToUint32()` | `Uint32` |
| `&`, `^`, `&#124;` | `ToInt32()` | `ToUint32()` | `Int32` |
| `~` | — | `ToInt32()` | `Int32` |

#### 2.2.3 转换为属性键

`ToPropertyKey()`返回一个字符串或符号，并被以下使用：

+   括号运算符`[]`

+   对象字面量中的计算属性键

+   `in`运算符的左操作数

+   `Object.defineProperty(_, P, _)`

+   `Object.fromEntries()`

+   `Object.getOwnPropertyDescriptor()`

+   `Object.prototype.hasOwnProperty()`

+   `Object.prototype.propertyIsEnumerable()`

+   `Reflect`的几种方法

#### 2.2.4 转换为数组索引

+   `ToLength()`主要用于字符串索引。

    +   `ToIndex()`的辅助函数

    +   结果范围`l`：0 ≤ `l` ≤ 2⁵³−1

+   `ToIndex()`用于 Typed Array 索引。

    +   与`ToLength()`的主要区别：如果参数超出范围，则抛出异常。

    +   结果范围`i`：0 ≤ `i` ≤ 2⁵³−1

+   `ToUint32()`用于数组索引。

    +   结果范围`i`：0 ≤ `i` < 2³²−1（上限被排除，为`.length`留出空间）

#### 2.2.5 转换为 Typed Array 元素

当我们设置 Typed Array 元素的值时，将使用以下转换函数之一：

+   `ToInt8()`

+   `ToUint8()`

+   `ToUint8Clamp()`

+   `ToInt16()`

+   `ToUint16()`

+   `ToInt32()`

+   `ToUint32()`

+   `ToBigInt64()`

+   `ToBigUint64()`

### 2.3 中场休息：用 JavaScript 表达规范算法

在本章的其余部分，我们将遇到几种规范算法，但是“实现”为 JavaScript。以下列表显示了一些经常使用的模式如何从规范转换为 JavaScript：

+   规范：如果 Type(value)是 String

    JavaScript：`if (TypeOf(value) === 'string')`

    （非常宽松的翻译；`TypeOf()`在下面定义）

+   规范：如果 IsCallable(method)为 true

    JavaScript：`if (IsCallable(method))`

    （`IsCallable()`在下面定义）

+   规范：让 numValue 成为 ToNumber(value)

    JavaScript：`let numValue = Number(value)`

+   规范：让 isArray 成为 IsArray(O)

    JavaScript：`let isArray = Array.isArray(O)`

+   规范：如果 O 具有[[NumberData]]内部插槽

    JavaScript：`if ('__NumberData__' in O)`

+   规范：让 tag 成为 Get(O, @@toStringTag)

    JavaScript：`let tag = O[Symbol.toStringTag]`

+   规范：返回字符串连接的“[object ”、tag 和“]”。

    JavaScript：`return '[object ' + tag + ']';`

`let`（而不是`const`）用于匹配规范的语言。

有一些东西被省略了 - 例如，[ReturnIfAbrupt shorthands](https://tc39.es/ecma262/#sec-returnifabrupt-shorthands) `?` 和 `!`。

```js
/**
 * An improved version of typeof
 */
function TypeOf(value) {
 const result = typeof value;
 switch (result) {
 case 'function':
 return 'object';
 case 'object':
 if (value === null) {
 return 'null';
 } else {
 return 'object';
 }
 default:
 return result;
 }
}

function IsCallable(x) {
 return typeof x === 'function';
}
```

### 2.4 示例强制算法

#### 2.4.1 `ToPrimitive()`

[The operation `ToPrimitive()`](https://tc39.es/ecma262/#sec-toprimitive) 是许多强制算法的中间步骤（本章后面将看到其中一些）。它将任意值转换为原始值。

`ToPrimitive()`在规范中经常使用，因为大多数操作符只能使用原始值。例如，我们可以使用加法操作符（`+`）来添加数字和连接字符串，但不能用它来连接数组。

这是 JavaScript 版本的`ToPrimitive()`的样子：

```js
/**
 * @param  hint Which type is preferred for the result:
 *             string, number, or don’t care?
 */
function ToPrimitive(input: any,
 hint: 'string'|'number'|'default' = 'default') {
 if (TypeOf(input) === 'object') {
 let exoticToPrim = input[Symbol.toPrimitive]; // (A)
 if (exoticToPrim !== undefined) {
 let result = exoticToPrim.call(input, hint);
 if (TypeOf(result) !== 'object') {
 return result;
 }
 throw new TypeError();
 }
 if (hint === 'default') {
 hint = 'number';
 }
 return OrdinaryToPrimitive(input, hint);
 } else {
 // input is already primitive
 return input;
 }
 }
```

`ToPrimitive()`允许对象通过`Symbol.toPrimitive`（第 A 行）覆盖转换为原始值。如果对象没有这样做，则将其传递给`OrdinaryToPrimitive()`：

```js
function OrdinaryToPrimitive(O: object, hint: 'string' | 'number') {
 let methodNames;
 if (hint === 'string') {
 methodNames = ['toString', 'valueOf'];
 } else {
 methodNames = ['valueOf', 'toString'];
 }
 for (let name of methodNames) {
 let method = O[name];
 if (IsCallable(method)) {
 let result = method.call(O);
 if (TypeOf(result) !== 'object') {
 return result;
 }
 }
 }
 throw new TypeError();
}
```

##### 2.4.1.1 调用`ToPrimitive()`的提示是什么？

参数`hint`可以有三个值中的一个：

+   `'number'`表示：如果可能的话，`input`应该转换为数字。

+   `'string'`表示：如果可能的话，`input`应该转换为字符串。

+   `'default'`表示：对于数字或字符串没有偏好。

以下是各种操作如何使用`ToPrimitive()`的几个示例：

+   `hint === 'number'`。以下操作更偏向于数字：

    +   `ToNumeric()`

    +   `ToNumber()`

    +   `ToBigInt()`，`BigInt()`

    +   抽象关系比较（`<`）

+   `hint === 'string'`。以下操作更偏向于字符串：

    +   `ToString()`

    +   `ToPropertyKey()`

+   `hint === 'default'`。以下操作对返回的原始值的类型是中立的：

    +   抽象相等比较（`==`）

    +   加法操作符（`+`）

    +   `new Date(value)`（`value`可以是数字或字符串）

正如我们所见，默认行为是将`'default'`处理为`'number'`。只有`Symbol`和`Date`的实例覆盖了这种行为（稍后会介绍）。

##### 2.4.1.2 将对象转换为原始值时调用的方法是哪些？

如果通过`Symbol.toPrimitive`未覆盖对原始值的转换，`OrdinaryToPrimitive()`将调用以下两种方法中的一个或两个：

+   如果`hint`指示我们希望原始值是字符串，则首先调用`'toString'`。

+   如果`hint`指示我们希望原始值是数字，则首先调用`'valueOf'`。

以下代码演示了这是如何工作的：

```js
const obj = {
 toString() { return 'a' },
 valueOf() { return 1 },
};

// String() prefers strings:
assert.equal(String(obj), 'a');

// Number() prefers numbers:
assert.equal(Number(obj), 1);
```

具有属性键`Symbol.toPrimitive`的方法覆盖了正常的转换为原始值。标准库中只有两次这样做：

+   `Symbol.prototypeSymbol.toPrimitive`

    +   如果接收者是`Symbol`的实例，则此方法始终返回包装的符号。

    +   其理由是`Symbol`的实例具有返回字符串的`.toString()`方法。但是，即使`hint`是`'string'`，也不应该调用`.toString()`，以免意外将`Symbol`的实例转换为字符串（这是一种完全不同类型的属性键）。

+   `Date.prototypeSymbol.toPrimitive`

    +   下面会更详细解释。

##### 2.4.1.3 `Date.prototype[Symbol.toPrimitive]()`

这是日期处理转换为原始值的方式：

```js
Date.prototype[Symbol.toPrimitive] = function (
 hint: 'default' | 'string' | 'number') {
 let O = this;
 if (TypeOf(O) !== 'object') {
 throw new TypeError();
 }
 let tryFirst;
 if (hint === 'string' || hint === 'default') {
 tryFirst = 'string';
 } else if (hint === 'number') {
 tryFirst = 'number';
 } else {
 throw new TypeError();
 }
 return OrdinaryToPrimitive(O, tryFirst);
 };
```

与默认算法唯一的区别是`'default'`变为`'string'`（而不是`'number'`）。如果我们使用将`hint`设置为`'default'`的操作，就可以观察到这一点：

+   The `==` operator 如果另一个操作数是除`undefined`、`null`和`boolean`之外的原始值，则将对象强制转换为原始值（使用默认提示）。在以下交互中，我们可以看到将日期强制转换的结果是一个字符串：

    ```js
    const d = new Date('2222-03-27');
    assert.equal(
     d == 'Wed Mar 27 2222 01:00:00 GMT+0100'
     + ' (Central European Standard Time)',
     true);
    ```

+   `+`运算符将两个操作数强制转换为原始值（使用默认提示）。如果其中一个结果是字符串，则执行字符串连接（否则执行数字相加）。在下面的交互中，我们可以看到将日期强制转换的结果是一个字符串，因为操作符返回一个字符串。

    ```js
    const d = new Date('2222-03-27');
    assert.equal(
     123 + d,
     '123Wed Mar 27 2222 01:00:00 GMT+0100'
     + ' (Central European Standard Time)');
    ```

#### 2.4.2 `ToString()`和相关操作

这是`ToString()`的 JavaScript 版本：

```js
function ToString(argument) {
 if (argument === undefined) {
 return 'undefined';
 } else if (argument === null) {
 return 'null';
 } else if (argument === true) {
 return 'true';
 } else if (argument === false) {
 return 'false';
 } else if (TypeOf(argument) === 'number') {
 return Number.toString(argument);
 } else if (TypeOf(argument) === 'string') {
 return argument;
 } else if (TypeOf(argument) === 'symbol') {
 throw new TypeError();
 } else if (TypeOf(argument) === 'bigint') {
 return BigInt.toString(argument);
 } else {
 // argument is an object
 let primValue = ToPrimitive(argument, 'string'); // (A)
 return ToString(primValue);
 }
}
```

请注意，此函数在将对象转换为字符串之前使用`ToPrimitive()`作为中间步骤，然后将原始结果转换为字符串（第 A 行）。

`ToString()`以有趣的方式偏离了`String()`的工作方式：如果`argument`是一个符号，前者会抛出`TypeError`，而后者不会。为什么会这样？符号的默认值是将它们转换为字符串会抛出异常：

```js
> const sym = Symbol('sym');

> ''+sym
TypeError: Cannot convert a Symbol value to a string
> `${sym}`
TypeError: Cannot convert a Symbol value to a string
```

`String()`和`Symbol.prototype.toString()`中都覆盖了默认值（它们都在下一小节中描述）：

```js
> String(sym)
'Symbol(sym)'
> sym.toString()
'Symbol(sym)'
```

##### 2.4.2.1 `String()`

```js
function String(value) {
 let s;
 if (value === undefined) {
 s = '';
 } else {
 if (new.target === undefined && TypeOf(value) === 'symbol') {
 // This function was function-called and value is a symbol
 return SymbolDescriptiveString(value);
 }
 s = ToString(value);
 }
 if (new.target === undefined) {
 // This function was function-called
 return s;
 }
 // This function was new-called
 return StringCreate(s, new.target.prototype); // simplified!
}
```

`String()`的工作方式不同，取决于是通过函数调用还是通过`new`调用。它使用[`new.target`](https://exploringjs.com/es6/ch_classes.html#sec_allocating-and-initializing-instances)来区分这两种情况。

这些是辅助函数`StringCreate()`和`SymbolDescriptiveString()`：

```js
/** 
 * Creates a String instance that wraps `value`
 * and has the given protoype.
 */
function StringCreate(value, prototype) {
 // ···
}

function SymbolDescriptiveString(sym) {
 assert.equal(TypeOf(sym), 'symbol');
 let desc = sym.description;
 if (desc === undefined) {
 desc = '';
 }
 assert.equal(TypeOf(desc), 'string');
 return 'Symbol('+desc+')';
}
```

##### 2.4.2.2 `Symbol.prototype.toString()`

除了`String()`之外，我们还可以使用方法`.toString()`将符号转换为字符串。其规范如下。

```js
Symbol.prototype.toString = function () {
 let sym = thisSymbolValue(this);
 return SymbolDescriptiveString(sym);
};
function thisSymbolValue(value) {
 if (TypeOf(value) === 'symbol') {
 return value;
 }
 if (TypeOf(value) === 'object' && '__SymbolData__' in value) {
 let s = value.__SymbolData__;
 assert.equal(TypeOf(s), 'symbol');
 return s;
 }
}
```

##### 2.4.2.3 `Object.prototype.toString`

`.toString()`的默认规范如下：

```js
Object.prototype.toString = function () {
 if (this === undefined) {
 return '[object Undefined]';
 }
 if (this === null) {
 return '[object Null]';
 }
 let O = ToObject(this);
 let isArray = Array.isArray(O);
 let builtinTag;
 if (isArray) {
 builtinTag = 'Array';
 } else if ('__ParameterMap__' in O) {
 builtinTag = 'Arguments';
 } else if ('__Call__' in O) {
 builtinTag = 'Function';
 } else if ('__ErrorData__' in O) {
 builtinTag = 'Error';
 } else if ('__BooleanData__' in O) {
 builtinTag = 'Boolean';
 } else if ('__NumberData__' in O) {
 builtinTag = 'Number';
 } else if ('__StringData__' in O) {
 builtinTag = 'String';
 } else if ('__DateValue__' in O) {
 builtinTag = 'Date';
 } else if ('__RegExpMatcher__' in O) {
 builtinTag = 'RegExp';
 } else {
 builtinTag = 'Object';
 }
 let tag = O[Symbol.toStringTag];
 if (TypeOf(tag) !== 'string') {
 tag = builtinTag;
 }
 return '[object ' + tag + ']';
};
```

如果我们将普通对象转换为字符串，则使用此操作：

```js
> String({})
'[object Object]'
```

默认情况下，如果我们将类的实例转换为字符串，则也会使用它：

```js
class MyClass {}
assert.equal(
 String(new MyClass()), '[object Object]');
```

通常，我们会重写`.toString()`以配置`MyClass`的字符串表示形式，但我们也可以更改字符串中“`object`”后面的内容，使用方括号：

```js
class MyClass {}
MyClass.prototype[Symbol.toStringTag] = 'Custom!';
assert.equal(
 String(new MyClass()), '[object Custom!]');
```

比较`.toString()`的重写版本与`Object.prototype`中的原始版本是很有趣的：

```js
> ['a', 'b'].toString()
'a,b'
> Object.prototype.toString.call(['a', 'b'])
'[object Array]'

> /^abc$/.toString()
'/^abc$/'
> Object.prototype.toString.call(/^abc$/)
'[object RegExp]'
```

#### 2.4.3 `ToPropertyKey()`

`ToPropertyKey()`被方括号运算符等使用。它的工作方式如下：

```js
function ToPropertyKey(argument) {
 let key = ToPrimitive(argument, 'string'); // (A)
 if (TypeOf(key) === 'symbol') {
 return key;
 }
 return ToString(key);
}
```

再次，对象在使用原始值之前被转换为原始值。

#### 2.4.4 `ToNumeric()`和相关操作

`ToNumeric()`被乘法运算符(`*`)等使用。它的工作方式如下：

```js
function ToNumeric(value) {
 let primValue = ToPrimitive(value, 'number');
 if (TypeOf(primValue) === 'bigint') {
 return primValue;
 }
 return ToNumber(primValue);
}
```

##### 2.4.4.1 `ToNumber()`

`ToNumber()`的工作方式如下：

```js
function ToNumber(argument) {
 if (argument === undefined) {
 return NaN;
 } else if (argument === null) {
 return +0;
 } else if (argument === true) {
 return 1;
 } else if (argument === false) {
 return +0;
 } else if (TypeOf(argument) === 'number') {
 return argument;
 } else if (TypeOf(argument) === 'string') {
 return parseTheString(argument); // not shown here
 } else if (TypeOf(argument) === 'symbol') {
 throw new TypeError();
 } else if (TypeOf(argument) === 'bigint') {
 throw new TypeError();
 } else {
 // argument is an object
 let primValue = ToPrimitive(argument, 'number');
 return ToNumber(primValue);
 }
}
```

`ToNumber()`的结构类似于`ToString()`的结构。

### 2.5 强制转换操作

#### 2.5.1 加法运算符(`+`)

这是 JavaScript 的加法运算符的规范：

```js
function Addition(leftHandSide, rightHandSide) {
 let lprim = ToPrimitive(leftHandSide);
 let rprim = ToPrimitive(rightHandSide);
 if (TypeOf(lprim) === 'string' || TypeOf(rprim) === 'string') { // (A)
 return ToString(lprim) + ToString(rprim);
 }
 let lnum = ToNumeric(lprim);
 let rnum = ToNumeric(rprim);
 if (TypeOf(lnum) !== TypeOf(rnum)) {
 throw new TypeError();
 }
 let T = Type(lnum);
 return T.add(lnum, rnum); // (B)
}
```

此算法的步骤：

+   两个操作数都被转换为原始值。

+   如果其中一个结果是字符串，则两者都将转换为字符串并连接（第 A 行）。

+   否则，两个操作数将转换为数值并相加（第 B 行）。`Type()`返回`lnum`的 ECMAScript 规范类型。`.add()`是[数值类型](https://tc39.es/ecma262/#sec-numeric-types)的一个方法。

#### 2.5.2 抽象相等比较(`==`)

```js
/** Loose equality (==) */
function abstractEqualityComparison(x, y) {
 if (TypeOf(x) === TypeOf(y)) {
 // Use strict equality (===)
 return strictEqualityComparison(x, y);
 }

 // Comparing null with undefined
 if (x === null && y === undefined) {
 return true;
 }
 if (x === undefined && y === null) {
 return true;
 }

 // Comparing a number and a string
 if (TypeOf(x) === 'number' && TypeOf(y) === 'string') {
 return abstractEqualityComparison(x, Number(y));
 }
 if (TypeOf(x) === 'string' && TypeOf(y) === 'number') {
 return abstractEqualityComparison(Number(x), y);
 }

 // Comparing a bigint and a string
 if (TypeOf(x) === 'bigint' && TypeOf(y) === 'string') {
 let n = StringToBigInt(y);
 if (Number.isNaN(n)) {
 return false;
 }
 return abstractEqualityComparison(x, n);
 }
 if (TypeOf(x) === 'string' && TypeOf(y) === 'bigint') {
 return abstractEqualityComparison(y, x);
 }

 // Comparing a boolean with a non-boolean
 if (TypeOf(x) === 'boolean') {
 return abstractEqualityComparison(Number(x), y);
 }
 if (TypeOf(y) === 'boolean') {
 return abstractEqualityComparison(x, Number(y));
 }

 // Comparing an object with a primitive
 // (other than undefined, null, a boolean)
 if (['string', 'number', 'bigint', 'symbol'].includes(TypeOf(x))
 && TypeOf(y) === 'object') {
 return abstractEqualityComparison(x, ToPrimitive(y));
 }
 if (TypeOf(x) === 'object'
 && ['string', 'number', 'bigint', 'symbol'].includes(TypeOf(y))) {
 return abstractEqualityComparison(ToPrimitive(x), y);
 }

 // Comparing a bigint with a number
 if ((TypeOf(x) === 'bigint' && TypeOf(y) === 'number')
 || (TypeOf(x) === 'number' && TypeOf(y) === 'bigint')) {
 if ([NaN, +Infinity, -Infinity].includes(x)
 || [NaN, +Infinity, -Infinity].includes(y)) {
 return false;
 }
 if (isSameMathematicalValue(x, y)) {
 return true;
 } else {
 return false;
 }
 }

 return false;
}
```

以下操作在此处未显示：

+   [`strictEqualityComparison()`](https://tc39.es/ecma262/#sec-strict-equality-comparison)

+   [`StringToBigInt()`](https://tc39.es/ecma262/#sec-stringtobigint)

+   [`isSameMathematicalValue()`](https://tc39.es/ecma262/#mathematical-value)

### 2.6 与类型转换相关的术语表

现在我们已经更仔细地了解了 JavaScript 的类型转换工作方式，让我们用与类型转换相关的术语简要总结一下：

+   在*类型转换*中，我们希望输出值具有给定类型。如果输入值已经具有该类型，则简单地返回它。否则，它将被转换为具有所需类型的值。

+   *显式类型转换*意味着程序员使用操作（函数、运算符等）来触发类型转换。显式转换可以是：

    +   *已检查*：如果值无法转换，则会抛出异常。

    +   *未经检查*：如果一个值无法转换，就会返回一个错误值。

+   *类型转换*取决于编程语言。例如，在 Java 中，它是显式的检查类型转换。

+   *类型强制*是隐式类型转换：一个操作会自动将其参数转换为它所需的类型。可以是检查的、未经检查的或介于两者之间的。

[来源：[维基百科](https://en.wikipedia.org/wiki/Type_conversion)]

[评论](https://github.com/rauschma/deep-js/issues/2)
