# 3 解构算法

> 原文：[`exploringjs.com/deep-js/ch_destructuring-algorithm.html`](https://exploringjs.com/deep-js/ch_destructuring-algorithm.html)

* * *

+   3.1 为模式匹配算法做准备

    +   3.1.1 使用声明性规则指定匹配算法

    +   3.1.2 基于声明性规则评估表达式

+   3.2 模式匹配算法

    +   3.2.1 模式

    +   3.2.2 变量的规则

    +   3.2.3 对象模式的规则

    +   3.2.4 数组模式的规则

+   3.3 空对象模式和数组模式

+   3.4 应用算法

    +   3.4.1 背景：通过匹配传递参数

    +   3.4.2 使用`move2()`

    +   3.4.3 使用`move1()`

    +   3.4.4 结论：默认值是模式部分的特性

* * *

在本章中，我们以不同的角度看待解构：作为一种递归模式匹配算法。

该算法将使我们更好地理解默认值。这在最后将会很有用，我们将尝试弄清楚以下两个函数的区别：

```js
function move({x=0, y=0} = {})         { ··· }
function move({x, y} = { x: 0, y: 0 }) { ··· }
```

### 3.1 为模式匹配算法做准备

解构赋值看起来像这样：

```js
«pattern» = «value»
```

我们想使用`pattern`从`value`中提取数据。

我们现在将看一个执行这种赋值的算法。这个算法在函数式编程中被称为*模式匹配*（简称：*匹配*）。它指定了运算符`←`（“匹配”）来将`pattern`与`value`匹配并在此过程中赋值给变量：

```js
«pattern» ← «value»
```

我们只会探讨解构赋值，但解构变量声明和解构参数定义的工作方式类似。我们也不会涉及高级特性：计算属性键，属性值简写，以及对象属性和数组元素作为赋值目标，超出了本章的范围。

匹配运算符的规范包括下降到两个操作数的结构的声明性规则。声明性符号可能需要一些时间来适应，但它使规范更加简洁。

#### 3.1.1 使用声明性规则指定匹配算法

本章中使用的声明性规则操作输入，并通过副作用产生算法的结果。这是一个这样的规则（我们稍后会再次看到）：

+   (2c) `{key: «pattern», «properties»} ← obj`

    ```js
    «pattern» ← obj.key
    {«properties»} ← obj
    ```

此规则有以下部分：

+   (2c)是规则的*编号*。该编号用于引用规则。

+   *head*（第一行）描述了输入必须是什么样子，以便应用此规则。

+   *body*（剩余行）描述了规则应用时发生的情况。

在规则（2c）中，头部表示如果存在至少一个属性（其键为`key`）和零个或多个剩余属性的对象模式，则可以应用此规则。此规则的效果是继续执行，将属性值模式与`obj.key`匹配，并将剩余属性与`obj`匹配。

让我们考虑本章中的另一条规则：

+   (2e) `{} ← obj`（没有剩余属性）

    ```js
    // We are finished
    ```

在规则(2e)中，头部意味着如果空对象模式`{}`与值`obj`匹配，则执行此规则。主体意味着在这种情况下，我们已经完成了。

规则(2c)和规则(2e)一起形成了一个声明性循环，它遍历箭头左侧模式的属性。

#### 3.1.2 基于声明性规则评估表达式

完整的算法是通过一系列声明性规则指定的。假设我们想要评估以下匹配表达式：

```js
{first: f, last: l} ← obj
```

为了应用一系列规则，我们从上到下遍历它们，并执行第一个适用的规则。如果在该规则的主体中有匹配的表达式，则再次应用规则。依此类推。

有时头部包括一个条件，该条件还确定了规则是否适用-例如：

+   (3a) `[«elements»] ← non_iterable`（非法值）

    `if (!isIterable(non_iterable))`

    ```js
    throw new TypeError();
    ```

### 3.2 模式匹配算法

#### 3.2.1 模式

模式要么是：

+   一个变量：`x`

+   一个对象模式：`{«properties»}`

+   一个数组模式：`[«elements»]`

接下来的三个部分指定了处理这三种情况的匹配表达式的规则。

#### 3.2.2 变量规则

+   1.  `x ← value`（包括`undefined`和`null`）

    ```js
    x = value
    ```

#### 3.2.3 对象模式规则

+   (2a) `{«properties»} ← undefined`（非法值）

    ```js
    throw new TypeError();
    ```

+   (2b) `{«properties»} ← null`（非法值）

    ```js
    throw new TypeError();
    ```

+   (2c) `{key: «pattern», «properties»} ← obj`

    ```js
    «pattern» ← obj.key
    {«properties»} ← obj
    ```

+   (2d) `{key: «pattern» = default_value, «properties»} ← obj`

    ```js
    const tmp = obj.key;
    if (tmp !== undefined) {
     «pattern» ← tmp
    } else {
     «pattern» ← default_value
    }
    {«properties»} ← obj
    ```

+   (2e) `{} ← obj`（没有剩余属性）

    ```js
    // We are finished
    ```

规则 2a 和 2b 处理非法值。规则 2c-2e 循环遍历模式的属性。在规则 2d 中，我们可以看到默认值提供了一个与`obj`中没有匹配属性对应的替代方案。

#### 3.2.4 数组模式规则

**数组模式和可迭代对象。** 数组解构的算法从数组模式和可迭代对象开始：

+   (3a) `[«elements»] ← non_iterable`（非法值）

    `if (!isIterable(non_iterable))`

    ```js
    throw new TypeError();
    ```

+   (3b) `[«elements»] ← iterable`

    `if (isIterable(iterable))`

    ```js
    const iterator = iterable[Symbol.iterator]();
    «elements» ← iterator
    ```

辅助函数：

```js
function isIterable(value) {
 return (value !== null
 && typeof value === 'object'
 && typeof value[Symbol.iterator] === 'function');
}
```

**数组元素和迭代器。** 算法继续进行：

+   模式的元素（箭头左侧）

+   从可迭代对象（箭头右侧）获得的迭代器

这些是规则：

+   (3c) `«pattern», «elements» ← iterator`

    ```js
    «pattern» ← getNext(iterator) // undefined after last item
    «elements» ← iterator
    ```

+   (3d) `«pattern» = default_value, «elements» ← iterator`

    ```js
    const tmp = getNext(iterator);  // undefined after last item
    if (tmp !== undefined) {
     «pattern» ← tmp
    } else {
     «pattern» ← default_value
    }
    «elements» ← iterator
    ```

+   (3e) `, «elements» ← iterator`（空位，省略）

    ```js
    getNext(iterator); // skip
    «elements» ← iterator
    ```

+   (3f) `...«pattern» ← iterator`（总是最后一部分！）

    ```js
    const tmp = [];
    for (const elem of iterator) {
     tmp.push(elem);
    }
    «pattern» ← tmp
    ```

+   (3g) `← iterator`（没有剩余元素）

    ```js
    // We are finished
    ```

辅助函数：

```js
function getNext(iterator) {
 const {done,value} = iterator.next();
 return (done ? undefined : value);
}
```

迭代器完成类似于对象中缺少的属性。

### 3.3 空对象模式和数组模式

算法规则的有趣结果：我们可以使用空对象模式和空数组模式进行解构。

给定一个空对象模式`{}`：如果要解构的值既不是`undefined`也不是`null`，则什么也不会发生。否则，将抛出`TypeError`。

```js
const {} = 123; // OK, neither undefined nor null
assert.throws(
 () => {
 const {} = null;
 },
 /^TypeError: Cannot destructure 'null' as it is null.$/)
```

给定一个空的数组模式`[]`：如果要解构的值是可迭代的，则什么也不会发生。否则，将抛出`TypeError`。

```js
const [] = 'abc'; // OK, iterable
assert.throws(
 () => {
 const [] = 123; // not iterable
 },
 /^TypeError: 123 is not iterable$/)
```

换句话说：空解构模式强制值具有某些特征，但没有其他效果。

### 3.4 应用算法

在 JavaScript 中，通过对象模拟命名参数：调用者使用对象文字，被调用者使用解构。这个模拟在[“JavaScript for impatient programmers”](https://exploringjs.com/impatient-js/ch_callables.html#named-parameters)中有详细解释。以下代码显示了一个例子：函数`move1()`有两个命名参数`x`和`y`：

```js
function move1({x=0, y=0} = {}) { // (A)
 return [x, y];
}
assert.deepEqual(
 move1({x: 3, y: 8}), [3, 8]);
assert.deepEqual(
 move1({x: 3}), [3, 0]);
assert.deepEqual(
 move1({}), [0, 0]);
assert.deepEqual(
 move1(), [0, 0]);
```

A 行中有三个默认值：

+   前两个默认值允许我们省略`x`和`y`。

+   第三个默认值允许我们调用`move1()`而不带参数（就像最后一行一样）。

但是为什么我们要像前面的代码片段中定义参数呢？为什么不像下面这样？

```js
function move2({x, y} = { x: 0, y: 0 }) {
 return [x, y];
}
```

要看`move1()`为什么是正确的，我们将在两个示例中使用这两个函数。在这之前，让我们看看参数的传递如何可以通过匹配来解释。

#### 3.4.1 背景：通过匹配传递参数

在函数调用中，*形式参数*（在函数定义内部）与*实际参数*（在函数调用内部）进行匹配。例如，考虑以下函数定义和以下函数调用。

```js
function func(a=0, b=0) { ··· }
func(1, 2);
```

参数`a`和`b`的设置类似于以下解构。

```js
[a=0, b=0] ← [1, 2]
```

#### 3.4.2 使用`move2()`

让我们看看`move2()`的解构是如何工作的。

**示例 1.** 函数调用`move2()`导致这种解构：

```js
[{x, y} = { x: 0, y: 0 }] ← []
```

左侧的单个数组元素在右侧没有匹配，这就是为什么`{x,y}`与默认值匹配而不是与右侧数据匹配的原因（规则 3b，3d）：

```js
{x, y} ← { x: 0, y: 0 }
```

左侧包含*属性值简写*。这是一个缩写形式：

```js
{x: x, y: y} ← { x: 0, y: 0 }
```

这种解构导致以下两个赋值（规则 2c，1）：

```js
x = 0;
y = 0;
```

这就是我们想要的。但是，在下一个示例中，我们就没有那么幸运了。

**示例 2.** 让我们检查函数调用`move2({z: 3})`，这导致以下解构：

```js
[{x, y} = { x: 0, y: 0 }] ← [{z: 3}]
```

右侧有一个索引为 0 的数组元素。因此，默认值被忽略，下一步是（规则 3d）：

```js
{x, y} ← { z: 3 }
```

这导致`x`和`y`都设置为`undefined`，这不是我们想要的。问题在于`{x,y}`不再与默认值匹配，而是与`{z:3}`匹配。

#### 3.4.3 使用`move1()`

让我们尝试`move1()`。

**示例 1：** `move1()`

```js
[{x=0, y=0} = {}] ← []
```

我们在右侧没有一个索引为 0 的数组元素，并且使用默认值（规则 3d）：

```js
{x=0, y=0} ← {}
```

左侧包含属性值简写，这意味着这种解构等同于：

```js
{x: x=0, y: y=0} ← {}
```

左侧的属性`x`和属性`y`都没有在右侧匹配。因此，使用默认值，并且接下来执行以下解构（规则 2d）：

```js
x ← 0
y ← 0
```

这导致以下赋值（规则 1）：

```js
x = 0
y = 0
```

在这里，我们得到了我们想要的。让我们看看我们的运气是否在下一个示例中持续。

**示例 2：** `move1({z: 3})`

```js
[{x=0, y=0} = {}] ← [{z: 3}]
```

数组模式的第一个元素在右侧有一个匹配，并且该匹配用于继续解构（规则 3d）：

```js
{x=0, y=0} ← {z: 3}
```

与示例 1 一样，在右侧没有属性`x`和`y`，并且使用默认值：

```js
x = 0
y = 0
```

它按预期工作！这次，模式中的`x`和`y`与`{z:3}`匹配不是问题，因为它们有自己的本地默认值。

#### 3.4.4 结论：默认值是模式部分的一个特性

这些示例表明默认值是模式部分（对象属性或数组元素）的一个特性。如果一个部分没有匹配或与`undefined`匹配，则使用默认值。也就是说，模式与默认值匹配，而不是与实际值匹配。

[评论](https://github.com/rauschma/deep-js/issues/3)
