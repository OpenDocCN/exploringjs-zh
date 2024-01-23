# 十二、值

> 原文：[`exploringjs.com/impatient-js/ch_values.html`](https://exploringjs.com/impatient-js/ch_values.html)

* * *

+   12.1 什么是类型？

+   12.2 JavaScript 的类型层次结构

+   12.3 语言规范的类型

+   12.4 原始值 vs. 对象

    +   12.4.1 原始值（简称：原始值）

    +   12.4.2 对象

+   12.5 `typeof`和`instanceof`运算符：值的类型是什么？

    +   12.5.1 `typeof`

    +   12.5.2 `instanceof`

+   12.6 类和构造函数

    +   12.6.1 与原始类型相关的构造函数

+   12.7 类型转换

    +   12.7.1 类型之间的显式转换

    +   12.7.2 强制转换（类型之间的自动转换）

* * *

在本章中，我们将研究 JavaScript 具有哪些类型的值。

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png)  **支持工具：`===`**

在本章中，我们偶尔会使用严格相等运算符。`a === b`的结果为`true`，如果`a`和`b`相等。这究竟意味着什么，在§13.4.2 “严格相等（`===`和`!==`）”中有解释。

### 12.1 什么是类型？

在本章中，我认为类型是值的集合，例如，`boolean`类型是集合{ `false`，`true` }。

### 12.2 JavaScript 的类型层次结构

![图 6：JavaScript 类型的部分层次结构。缺少错误类的类，与原始类型相关的类等。该图表暗示了并非所有对象都是`Object`的实例。](img/2368843b86b1f2ca6ddd0254e1bf1dab.png)

图 6：JavaScript 类型的部分层次结构。缺少错误类的类，与原始类型相关的类等。该图表暗示了并非所有对象都是`Object`的实例。

图 6 显示了 JavaScript 的类型层次结构。我们从该图中学到了什么？

+   JavaScript 区分两种值：原始值和对象。我们很快就会看到它们之间的区别。

+   该图表区分了对象和`Object`类的实例。`Object`的每个实例也是一个对象，但反之则不然。然而，在实践中，你遇到的几乎所有对象都是`Object`的实例，例如，通过对象字面量创建的对象。有关此主题的更多详细信息，请参阅§29.7.3 “并非所有对象都是`Object`的实例”。

### 12.3 语言规范的类型

ECMAScript 规范只知道总共八种类型。这些类型的名称是（我使用的是 TypeScript 的名称，而不是规范的名称）：

+   `undefined`，唯一元素为`undefined`

+   `null`，唯一元素为`null`

+   `boolean`，元素为`false`和`true`

+   `number`所有数字的类型（例如，`-123`，`3.141`）

+   `bigint`所有大整数的类型（例如，`-123n`）

+   `string`所有字符串的类型（例如，`'abc'`）

+   `symbol`所有符号的类型（例如，`Symbol('My Symbol')`）

+   `object`所有对象的类型（不同于`Object`，`Object`类及其子类的所有实例的类型）

### 12.4 原始值 vs. 对象

规范对值进行了重要的区分：

+   *原始值*是`undefined`、`null`、`boolean`、`number`、`bigint`、`string`、`symbol`类型的元素。

+   所有其他值都是*对象*。

与 Java（在这里受到启发的 JavaScript）不同，原始值不是二等公民。它们与对象之间的区别更加微妙。简而言之：

+   原始值：是 JavaScript 中的原子数据构建块。

    +   它们是*通过值*传递的：当原始值分配给变量或传递给函数时，它们的内容被复制。

    +   它们是*通过值*比较的：当比较两个原始值时，它们的内容被比较。

+   对象：是复合数据。

    +   它们是*通过标识*（我的术语）传递的：当对象被分配给变量或传递给函数时，它们的*标识*（想象指针）被复制。

    +   它们是*通过标识*（我的术语）比较的：当比较两个对象时，它们的标识被比较。

除此之外，原始值和对象非常相似：它们都有*属性*（键值条目）并且可以在相同的位置使用。

接下来，我们将更深入地了解原始值和对象。 

#### 12.4.1 原始值（简称：原始值）

##### 12.4.1.1 原始值是不可变的

您无法更改、添加或删除原始值的属性：

```js
const str = 'abc';
assert.equal(str.length, 3);
assert.throws(
 () => { str.length = 1 },
 /^TypeError: Cannot assign to read only property 'length'/
);
```

##### 12.4.1.2 原始值是*通过值*传递的

原始值是*通过值*传递的：变量（包括参数）存储原始值的内容。将原始值分配给变量或将其作为参数传递给函数时，其内容被复制。

```js
const x = 123;
const y = x;
// `y` is the same as any other number 123
assert.equal(y, 123);
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **观察通过值传递和通过引用传递的区别**

由于原始值是不可变的并且按值比较（见下一小节），所以无法观察通过值传递和通过标识传递（在 JavaScript 中用于对象）之间的区别。

##### 12.4.1.3 原始值是*通过值*比较的

原始值是*通过值*比较的：当比较两个原始值时，我们比较它们的内容。

```js
assert.equal(123 === 123, true);
assert.equal('abc' === 'abc', true);
```

要了解这种比较方式的特殊之处，请继续阅读并了解对象是如何比较的。

#### 12.4.2 对象

对象在§28“对象”和接下来的章节中有详细介绍。在这里，我们主要关注它们与原始值的区别。

让我们首先探讨创建对象的两种常见方法：

+   对象文字：

    ```js
    const obj = {
     first: 'Jane',
     last: 'Doe',
    };
    ```

    对象文字以大括号`{}`开始和结束。它创建一个具有两个属性的对象。第一个属性具有键`'first'`（字符串）和值`'Jane'`。第二个属性具有键`'last'`和值`'Doe'`。有关对象文字的更多信息，请参阅§28.3.1“对象文字：属性”。

+   数组文字：

    ```js
    const fruits = ['strawberry', 'apple'];
    ```

    数组文字以方括号`[]`开始和结束。它创建一个包含两个*元素*`'strawberry'`和`'apple'`的数组。有关数组文字的更多信息，请参阅§31.3.1“创建、读取、写入数组”。

##### 12.4.2.1 对象默认是可变的

默认情况下，您可以自由更改、添加和删除对象的属性：

```js
const obj = {};

obj.count = 2; // add a property
assert.equal(obj.count, 2);

obj.count = 3; // change a property
assert.equal(obj.count, 3);
```

##### 12.4.2.2 对象是*通过标识*传递的

对象是*通过标识*（我的术语）传递的：变量（包括参数）存储对象的*标识*。

对象的标识就像是指向对象在堆上实际数据的指针（或透明引用）（想象 JavaScript 引擎的共享主内存）。

将对象分配给变量或将其作为参数传递给函数时，其标识被复制。每个对象文字在堆上创建一个新对象并返回其标识。

```js
const a = {}; // fresh empty object
// Pass the identity in `a` to `b`:
const b = a;

// Now `a` and `b` point to the same object
// (they “share” that object):
assert.equal(a === b, true);

// Changing `a` also changes `b`:
a.name = 'Tessa';
assert.equal(b.name, 'Tessa');
```

JavaScript 使用*垃圾回收*来自动管理内存：

```js
let obj = { prop: 'value' };
obj = {};
```

现在`obj`的旧值`{ prop: 'value' }`是*垃圾*（不再使用）。JavaScript 将自动*垃圾回收*它（从内存中删除），在某个时间点（可能永远不会，如果有足够的空闲内存）。

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **详细信息：通过标识传递**

“通过标识传递”意味着对象的标识（透明引用）是按值传递的。这种方法也称为[“通过共享传递”](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)。

##### 12.4.2.3 对象通过标识进行比较

对象通过标识进行比较（我的术语）：只有当两个变量包含相同的对象标识时，它们才相等。如果它们引用具有相同内容的不同对象，则它们不相等。

```js
const obj = {}; // fresh empty object
assert.equal(obj === obj, true); // same identity
assert.equal({} === {}, false); // different identities, same content
```

### 12.5 `typeof`和`instanceof`运算符：值的类型是什么？

两个运算符`typeof`和`instanceof`让您确定给定值`x`的类型：

```js
if (typeof x === 'string') ···
if (x instanceof Array) ···
```

它们有何不同？

+   `typeof`区分规范中的 7 种类型（减去一个遗漏，加上一个添加）。

+   `instanceof`测试哪个类创建了给定的值。

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **经验法则：`typeof`用于原始值；`instanceof`用于对象**

#### 12.5.1 `typeof`

表 2：`typeof`运算符的结果。

| `x` | `typeof x` |
| --- | --- |
| `undefined` | `'undefined'` |
| `null` | `'object'` |
| 布尔值 | `'boolean'` |
| 数字 | `'number'` |
| 大整数 | `'bigint'` |
| 字符串 | `'string'` |
| 符号 | `'symbol'` |
| 函数 | `'function'` |
| 所有其他对象 | `'object'` |

Tbl. 2 列出了`typeof`的所有结果。它们大致对应于语言规范的 7 种类型。遗憾的是，存在两个差异，它们是语言怪癖：

+   `typeof null`返回`'object'`而不是`'null'`。这是一个错误。不幸的是，它无法修复。TC39 试图这样做，但它在网络上破坏了太多代码。

+   函数的`typeof`应该是`'object'`（函数是对象）。为函数引入一个单独的类别是令人困惑的。

以下是使用`typeof`的几个例子：

```js
> typeof undefined
'undefined'
> typeof 123n
'bigint'
> typeof 'abc'
'string'
> typeof {}
'object'
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：关于`typeof`的两个练习**

+   `exercises/values/typeof_exrc.mjs`

+   奖励：`exercises/values/is_object_test.mjs`

#### 12.5.2 `instanceof`

此运算符回答问题：值`x`是否由类`C`创建？

```js
x instanceof C
```

例如：

```js
> (function() {}) instanceof Function
true
> ({}) instanceof Object
true
> [] instanceof Array
true
```

原始值不是任何东西的实例：

```js
> 123 instanceof Number
false
> '' instanceof String
false
> '' instanceof Object
false
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：`instanceof`**

`exercises/values/instanceof_exrc.mjs`

### 12.6 类和构造函数

JavaScript 的原始对象工厂是*构造函数*：普通函数，如果通过`new`运算符调用它们，则返回自己的“实例”。

ES6 引入了*类*，主要是构造函数的更好语法。

在本书中，我将*构造函数*和*类*这两个术语互换使用。

类可以被视为将规范中的单一类型`object`分成子类型 - 它们给我们比规范中有限的 7 种类型更多的类型。每个类都是由它创建的对象的类型。

#### 12.6.1 与原始类型相关联的构造函数

每种原始类型（除了规范内部类型`undefined`和`null`）都有一个关联的*构造函数*（考虑类）：

+   构造函数`Boolean`与布尔值相关联。

+   构造函数`Number`与数字相关联。

+   构造函数`String`与字符串相关联。

+   构造函数`Symbol`与符号相关联。

这些函数各自扮演多个角色 - 例如，`Number`：

+   您可以将其作为函数使用并将值转换为数字：

    ```js
    assert.equal(Number('123'), 123);
    ```

+   `Number.prototype`提供了数字的属性 - 例如，方法`.toString()`：

    ```js
    assert.equal((123).toString, Number.prototype.toString);
    ```

+   `Number`是用于数字的工具函数的命名空间/容器对象 - 例如：

    ```js
    assert.equal(Number.isInteger(123), true);
    ```

+   最后，您还可以将`Number`用作类并创建数字对象。这些对象与实际数字不同，应该避免使用。

    ```js
    assert.notEqual(new Number(123), 123);
    assert.equal(new Number(123).valueOf(), 123);
    ```

##### 12.6.1.1 包装原始值

与原始类型相关的构造函数也称为*包装类型*，因为它们提供了将原始值转换为对象的规范方式。在这个过程中，原始值被“包装”在对象中。

```js
const prim = true;
assert.equal(typeof prim, 'boolean');
assert.equal(prim instanceof Boolean, false);

const wrapped = Object(prim);
assert.equal(typeof wrapped, 'object');
assert.equal(wrapped instanceof Boolean, true);

assert.equal(wrapped.valueOf(), prim); // unwrap
```

包装在实践中很少重要，但在语言规范中内部使用，以赋予原始属性。

### 12.7 类型之间的转换

在 JavaScript 中，有两种方式将值转换为其他类型：

+   显式转换：通过诸如`String()`之类的函数。

+   *强制转换*（自动转换）：当操作接收到无法处理的操作数/参数时发生。

#### 12.7.1 显式类型转换

与原始类型相关联的函数明确地将值转换为该类型：

```js
> Boolean(0)
false
> Number('123')
123
> String(123)
'123'
```

您也可以使用`Object()`将值转换为对象：

```js
> typeof Object(123)
'object'
```

以下表格更详细地描述了这种转换方式：

| `x` | `Object(x)` |
| --- | --- |
| `undefined` | `{}` |
| `null` | `{}` |
| boolean | `new Boolean(x)` |
| number | `new Number(x)` |
| bigint | `BigInt`的一个实例（`new`抛出`TypeError`） |
| string | `new String(x)` |
| symbol | `Symbol`的一个实例（`new`抛出`TypeError`） |
| object | `x` |

#### 12.7.2 强制转换（类型之间的自动转换）

对于许多操作，如果它们的类型不匹配，JavaScript 会自动转换操作数/参数。这种自动转换称为*强制转换*。

例如，乘法运算符将其操作数强制转换为数字：

```js
> '7' * '3'
21
```

许多内置函数也会进行强制转换。例如，`Number.parseInt()`在解析之前会将其参数强制转换为字符串。这解释了以下结果：

```js
> Number.parseInt(123.45)
123
```

数字`123.45`在解析之前被转换为字符串`'123.45'`。解析在第一个非数字字符之前停止，这就是为什么结果是`123`。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：将值转换为原始值**

`exercises/values/conversion_exrc.mjs`

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

参见测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/7)
