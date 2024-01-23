# 二十二、符号

> 原文：[`exploringjs.com/impatient-js/ch_symbols.html`](https://exploringjs.com/impatient-js/ch_symbols.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   22.1 符号是原始值，也像对象

    +   22.1.1 符号是原始值

    +   22.1.2 符号也像对象

+   22.2 符号的描述

+   22.3 使用符号的用例

    +   22.3.1 符号作为常量的值

    +   22.3.2 符号作为唯一的属性键

+   22.4 公开已知的符号

+   22.5 转换符号

* * *

### 22.1 符号是原始值，也像对象

符号是通过工厂函数`Symbol()`创建的原始值：

```js
const mySymbol = Symbol('mySymbol');
```

参数是可选的，并提供一个描述，这对于调试非常有用。

#### 22.1.1 符号是原始值

符号是原始值：

+   它们必须通过`typeof`进行分类：

    ```js
    const sym = Symbol();
    assert.equal(typeof sym, 'symbol');
    ```

+   它们可以是对象中的属性键：

    ```js
    const obj = {
     [sym]: 123,
    };
    ```

#### 22.1.2 符号也像对象

尽管符号是原始值，但它们也像对象，因为`Symbol()`创建的每个值都是唯一的，不是按值比较的：

```js
> Symbol() === Symbol()
false
```

在引入符号之前，如果我们需要唯一的值（只等于自身），对象是最佳选择：

```js
const string1 = 'abc';
const string2 = 'abc';
assert.equal(
 string1 === string2, true); // not unique

const object1 = {};
const object2 = {};
assert.equal(
 object1 === object2, false); // unique

const symbol1 = Symbol();
const symbol2 = Symbol();
assert.equal(
 symbol1 === symbol2, false); // unique
```

### 22.2 符号的描述

我们传递给符号工厂函数的参数为创建的符号提供了一个描述：

```js
const mySymbol = Symbol('mySymbol');
```

描述可以通过两种方式访问。

首先，它是由`.toString()`返回的字符串的一部分：

```js
assert.equal(mySymbol.toString(), 'Symbol(mySymbol)');
```

其次，自 ES2019 以来，我们可以通过属性`.description`检索描述：

```js
assert.equal(mySymbol.description, 'mySymbol');
```

### 22.3 使用符号的用例

符号的主要用例是：

+   常量的值

+   唯一的属性键

#### 22.3.1 符号作为常量的值

假设您想创建代表红色、橙色、黄色、绿色、蓝色和紫色的常量。一个简单的方法是使用字符串：

```js
const COLOR_BLUE = 'Blue';
```

好处是，记录该常量会产生有用的输出。坏处是，有可能错误地将与颜色无关的值误认为是颜色，因为两个具有相同内容的字符串被认为是相等的：

```js
const MOOD_BLUE = 'Blue';
assert.equal(COLOR_BLUE, MOOD_BLUE);
```

我们可以通过符号解决这个问题：

```js
const COLOR_BLUE = Symbol('Blue');
const MOOD_BLUE = Symbol('Blue');

assert.notEqual(COLOR_BLUE, MOOD_BLUE);
```

让我们使用符号值常量来实现一个函数：

```js
const COLOR_RED    = Symbol('Red');
const COLOR_ORANGE = Symbol('Orange');
const COLOR_YELLOW = Symbol('Yellow');
const COLOR_GREEN  = Symbol('Green');
const COLOR_BLUE   = Symbol('Blue');
const COLOR_VIOLET = Symbol('Violet');

function getComplement(color) {
 switch (color) {
 case COLOR_RED:
 return COLOR_GREEN;
 case COLOR_ORANGE:
 return COLOR_BLUE;
 case COLOR_YELLOW:
 return COLOR_VIOLET;
 case COLOR_GREEN:
 return COLOR_RED;
 case COLOR_BLUE:
 return COLOR_ORANGE;
 case COLOR_VIOLET:
 return COLOR_YELLOW;
 default:
 throw new Exception('Unknown color: '+color);
 }
}
assert.equal(getComplement(COLOR_YELLOW), COLOR_VIOLET);
```

#### 22.3.2 符号作为唯一的属性键

对象中属性（字段）的键在两个级别上使用：

+   程序在*基本级别*上运行。该级别的键反映了*问题域* - 程序解决问题的领域 - 例如：

    +   如果程序管理员工，属性键可能是关于职称、薪水类别、部门 ID 等。

    +   如果程序是一个国际象棋应用，属性键可能是有关国际象棋棋子、国际象棋棋盘、玩家颜色等的。

+   ECMAScript 和许多库在*元级别*上操作。它们管理数据并提供不属于问题域的服务 - 例如：

    +   标准方法`.toString()`在创建对象的字符串表示时由 ECMAScript 使用（A 行）：

        ```js
        const point = {
         x: 7,
         y: 4,
         toString() {
         return `(${this.x}, ${this.y})`;
         },
        };
        assert.equal(
         String(point), '(7, 4)'); // (A)
        ```

        `.x`和`.y`是基本级别的属性 - 它们用于解决计算点的问题。`.toString()`是一个元级别的属性 - 它与问题域无关。

    +   标准的 ECMAScript 方法`.toJSON()`

        ```js
        const point = {
         x: 7,
         y: 4,
         toJSON() {
         return [this.x, this.y];
         },
        };
        assert.equal(
         JSON.stringify(point), '[7,4]');
        ```

        `.x`和`.y`是基本级别的属性，`.toJSON()`是一个元级别的属性。

程序的基本级别和元级别必须是独立的：基本级别的属性键不应与元级别的属性键冲突。

如果我们使用名称（字符串）作为属性键，我们将面临两个挑战：

+   当一种语言首次创建时，它可以使用任何元级别的名称。基本级别的代码被迫避免使用这些名称。然而，当大量基本级别的代码已经存在时，元级别的名称就不能再自由选择了。

+   我们可以引入命名规则来区分基本级别和元级别。例如，Python 用两个下划线括住元级别名称：`__init__`，`__iter__`，`__hash__`等。然而，语言的元级别名称和库的元级别名称仍然存在于同一个命名空间中，并且可能会发生冲突。

这是 JavaScript 中后者成为问题的两个例子：

+   在 2018 年 5 月，数组方法`.flatten()`不得不改名为`.flat()`，因为前者的名称已经被库使用了（[来源](https://github.com/tc39/proposal-flatMap/commit/093eacc7fe0906e70f7626bf6c7d6e9dfc53cce9)）。

+   在 2020 年 11 月，数组方法`.item()`不得不改名为`.at()`，因为前者的名称已经被库使用了（[来源](https://github.com/tc39/proposal-relative-indexing-method#web-incompatibility-history)）。

在这里，作为属性键使用的符号有助于我们：每个符号都是唯一的，符号键永远不会与任何其他字符串或符号键冲突。

##### 22.3.2.1 示例：具有元级别方法的库

举个例子，假设我们正在编写一个库，如果对象实现了一个特殊方法，它会以不同的方式处理对象。这就是为这样一个方法定义属性键并为对象实现它的样子：

```js
const specialMethod = Symbol('specialMethod');
const obj = {
 _id: 'kf12oi',
 [specialMethod]() { // (A)
 return this._id;
 }
};
assert.equal(obj[specialMethod](), 'kf12oi');
```

A 行中的方括号使我们能够指定该方法必须具有键`specialMethod`。更多细节在§28.7.2“对象文字中的计算键”中有解释。

### 22.4 公开已知符号

在 ECMAScript 中扮演特殊角色的符号称为*公开已知符号*。例如：

+   `Symbol.iterator`: 使对象*可迭代*。它是返回迭代器的方法的键。有关此主题的更多信息，请参见§30“同步迭代”。

+   `Symbol.hasInstance`: 自定义`instanceof`的工作方式。如果对象实现了具有该键的方法，则可以在该运算符的右侧使用它。例如：

    ```js
    const PrimitiveNull = {
     Symbol.hasInstance {
     return x === null;
     }
    };
    assert.equal(null instanceof PrimitiveNull, true);
    ```

+   `Symbol.toStringTag`: 影响默认的`.toString()`方法。

    ```js
    > String({})
    '[object Object]'
    > String({ [Symbol.toStringTag]: 'is no money' })
    '[object is no money]'
    ```

    注意：通常最好重写`.toString()`。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：公开已知符号**

+   `Symbol.toStringTag`: `exercises/symbols/to_string_tag_test.mjs`

+   `Symbol.hasInstance`: `exercises/symbols/has_instance_test.mjs`

### 22.5 转换符号

如果我们将符号`sym`转换为另一个原始类型会发生什么？Tbl. 15 中有答案。

表 15：将符号转换为其他原始类型的结果。

| 转换为 | 显式转换 | 强制转换（隐式转换） |
| --- | --- | --- |
| 布尔值 | `Boolean(sym)` `→` OK | `!sym` `→` OK |
| 数字 | `Number(sym)` `→` `TypeError` | `sym*2` `→` `TypeError` |
| 字符串 | `String(sym)` `→` OK | `''+sym` `→` `TypeError` |
|  | `sym.toString()` `→` OK | ``${sym}`` `→` `TypeError` |

符号的一个关键陷阱是在将它们转换为其他类型时经常抛出异常。这背后的思想是什么？首先，将符号转换为数字是毫无意义的，应该提出警告。其次，将符号转换为字符串确实对诊断输出很有用。但是，警告意外地将符号转换为字符串也是有道理的（这是一种不同类型的属性键）：

```js
const obj = {};
const sym = Symbol();
assert.throws(
 () => { obj['__'+sym+'__'] = true },
 { message: 'Cannot convert a Symbol value to a string' });
```

缺点是异常使得使用符号变得更加复杂。在通过加号运算符组装字符串时，您必须显式转换符号：

```js
> const mySymbol = Symbol('mySymbol');
> 'Symbol I used: ' + mySymbol
TypeError: Cannot convert a Symbol value to a string
> 'Symbol I used: ' + String(mySymbol)
'Symbol I used: Symbol(mySymbol)'
```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/15)
