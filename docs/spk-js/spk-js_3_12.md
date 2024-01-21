# 第十八章：数组

> 原文：[18. Arrays](https://exploringjs.com/es5/ch18.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


数组是从索引（从零开始的自然数）到任意值的映射。值（映射的范围）称为数组的*元素*。创建数组的最方便的方法是通过数组字面量。这样的字面量列举了数组元素；元素的位置隐含地指定了它的索引。

在本章中，我将首先介绍基本的数组机制，如索引访问和`length`属性，然后再介绍数组方法。

## 概述

本节提供了数组的快速概述。详细内容将在后面解释。

作为第一个例子，我们通过数组字面量创建一个数组 `arr`（参见[创建数组](ch18.html#creating_arrays "Creating Arrays"））并访问元素（参见[数组索引](ch18.html#array_indices "Array Indices"）：

```js
> var arr = [ 'a', 'b', 'c' ]; // array literal
> arr[0]  // get element 0
'a'
> arr[0] = 'x';  // set element 0
> arr
[ 'x', 'b', 'c' ]
```

我们可以使用数组属性 `length`（参见[length](ch18.html#array_length "length")）来删除和追加元素：

```js
> var arr = [ 'a', 'b', 'c' ];
> arr.length
3
> arr.length = 2;  // remove an element
> arr
[ 'a', 'b' ]
> arr[arr.length] = 'd';  // append an element
> arr
[ 'a', 'b', 'd' ]
```

数组方法 `push()` 提供了另一种追加元素的方式：

```js
> var arr = [ 'a', 'b' ];
> arr.push('d')
3
> arr
[ 'a', 'b', 'd' ]
```

### 数组是映射，不是元组

ECMAScript 标准将数组规定为从索引到值的映射（字典）。换句话说，数组可能不是连续的，并且可能有空洞。例如：

```js
> var arr = [];
> arr[0] = 'a';
'a'
> arr[2] = 'b';
'b'
> arr
[ 'a', , 'b' ]
```

前面的数组有一个空洞：索引 1 处没有元素。[数组中的空洞](ch18.html#array_holes "Holes in Arrays") 更详细地解释了空洞。

请注意，大多数 JavaScript 引擎会在内部优化没有空洞的数组，并将它们连续存储。

### 数组也可以有属性

数组仍然是对象，可以有对象属性。这些属性不被视为实际数组的一部分；也就是说，它们不被视为数组元素：

```js
> var arr = [ 'a', 'b' ];
> arr.foo = 123;
> arr
[ 'a', 'b' ]
> arr.foo
123
```

## 创建数组

你可以通过数组字面量创建一个数组：

```js
var myArray = [ 'a', 'b', 'c' ];
```

数组中的尾随逗号会被忽略：

```js
> [ 'a', 'b' ].length
2
> [ 'a', 'b', ].length
2
> [ 'a', 'b', ,].length  // hole + trailing comma
3
```

### 数组构造函数

有两种使用构造函数 `Array` 的方式：可以创建一个给定长度的空数组，或者数组的元素是给定的值。对于这个构造函数，`new` 是可选的：以普通函数的方式调用它（不带 `new`）与以构造函数的方式调用它是一样的。

#### 创建一个给定长度的空数组

给定长度的空数组中只有空洞！因此，很少有意义使用这个版本的构造函数：

```js
> var arr = new Array(2);
> arr.length
2
> arr  // two holes plus trailing comma (ignored!)
[ , ,]
```

一些引擎在以这种方式调用 `Array()` 时可能会预先分配连续的内存，这可能会稍微提高性能。但是，请确保增加的冗余性值得！

#### 初始化带有元素的数组（避免！）

这种调用 `Array` 的方式类似于数组字面量：

```js
// The same as ['a', 'b', 'c']:
var arr1 = new Array('a', 'b', 'c');
```

问题在于你不能创建只有一个数字的数组，因为那会被解释为创建一个 `length` 为该数字的数组：

```js
> new Array(2)  // alas, not [ 2 ]
[ , ,]

> new Array(5.7)  // alas, not [ 5.7 ]
RangeError: Invalid array length

> new Array('abc')  // ok
[ 'abc' ]
```

### 多维数组

如果你需要为元素创建多个维度，你必须嵌套数组。当你创建这样的嵌套数组时，最内层的数组可以根据需要增长。但是，如果你想直接访问元素，你至少需要创建外部数组。在下面的例子中，我为井字游戏创建了一个三乘三的矩阵。该矩阵完全填满了数据（而不是让行根据需要增长）：

```js
// Create the Tic-tac-toe board
var rows = [];
for (var rowCount=0; rowCount < 3; rowCount++) {
    rows[rowCount] = [];
    for (var colCount=0; colCount < 3; colCount++) {
        rows[rowCount][colCount] = '.';
    }
}

// Set an X in the upper right corner
rows[0][2] = 'X';  // [row][column]

// Print the board
rows.forEach(function (row) {
    console.log(row.join(' '));
});
```

以下是输出：

```js
. . X
. . .
. . .
```

我希望这个例子能够演示一般情况。显然，如果矩阵很小并且具有固定的维度，你可以通过数组字面量来设置它：

```js
var rows = [ ['.','.','.'], ['.','.','.'], ['.','.','.'] ];
```

## 数组索引

当你使用数组索引时，你必须牢记以下限制：

+   索引是范围在 0 ≤ `i` < 2³²−1 的数字 *i*。

+   最大长度为 2³²−1。

超出范围的索引被视为普通的属性键（字符串！）。它们不会显示为数组元素，也不会影响属性 `length`。例如：

```js
> var arr = [];

> arr[-1] = 'a';
> arr
[]
> arr['-1']
'a'

> arr[4294967296] = 'b';
> arr
[]
> arr['4294967296']
'b'
```

### `in` 操作符和索引

`in` 操作符用于检测对象是否具有给定键的属性。但它也可以用于确定数组中是否存在给定的元素索引。例如：

```js
> var arr = [ 'a', , 'b' ];
> 0 in arr
true
> 1 in arr
false
> 10 in arr
false
```

### 删除数组元素

除了删除属性之外，`delete` 操作符还可以删除数组元素。删除元素会创建空洞（`length` 属性不会更新）：

```js
> var arr = [ 'a', 'b' ];
> arr.length
2
> delete arr[1]  // does not update length
true
> arr
[ 'a',  ]
> arr.length
2
```

你也可以通过减少数组的长度来删除尾随的数组元素（参见[length](ch18.html#array_length "length")了解详情）。要删除元素而不创建空洞（即，后续元素的索引被减少），你可以使用 `Array.prototype.splice()`（参见[添加和删除元素（破坏性）](ch18.html#Array.prototype.push "Adding and Removing Elements (Destructive)")）。在这个例子中，我们删除索引为 1 的两个元素：

```js
> var arr = ['a', 'b', 'c', 'd'];
> arr.splice(1, 2) // returns what has been removed
[ 'b', 'c' ]
> arr
[ 'a', 'd' ]
```

### 数组索引详解

### 提示

这是一个高级部分。通常情况下，您不需要知道这里解释的细节。

*数组索引并非看起来那样。* 到目前为止，我一直假装数组索引是数字。这也是 JavaScript 引擎在内部实现数组的方式。然而，ECMAScript 规范对索引的看法不同。引用[第 15.4 节](http://bit.ly/1fwoCFg)的话来说：

+   如果且仅当`ToString``(ToUint32(P))`等于`P`且`ToUint32(P)`不等于 2³²−1 时，属性键`P`（一个字符串）才是*数组索引*。这意味着什么将在下面解释。

+   属性键为数组索引的数组属性称为*元素*。

换句话说，在规范中，括号中的所有值都被转换为字符串，并解释为属性键，甚至是数字。以下互动演示了这一点：

```js
> var arr = ['a', 'b'];
> arr['0']
'a'
> arr[0]
'a'
```

要成为数组索引，属性键`P`（一个字符串！）必须等于以下计算结果：

1.  将`P`转换为数字。

1.  将数字转换为 32 位无符号整数。

1.  将整数转换为字符串。

这意味着数组索引必须是 32 位范围内的字符串化整数*i*，其中 0 ≤ *i* < 2³²−1。规范明确排除了上限（如前面引用的）。它保留给了最大长度。要了解这个定义是如何工作的，让我们使用[通过位运算符实现 32 位整数](ch11.html#integers_via_bitwise_operators "通过位运算符实现 32 位整数")中的`ToUint32()`函数。

首先，不包含数字的字符串总是转换为 0，这在字符串化后不等于字符串：

```js
> ToUint32('xyz')
0
> ToUint32('?@#!')
0
```

其次，超出范围的字符串化整数也会转换为完全不同的整数，与字符串化后不相等：

```js
> ToUint32('-1')
4294967295
> Math.pow(2, 32)
4294967296
> ToUint32('4294967296')
0
```

第三，字符串化的非整数数字会转换为整数，这些整数又是不同的：

```js
> ToUint32('1.371')
1
```

请注意，规范还强制规定数组索引不得具有指数：

```js
> ToUint32('1e3')
1000
```

它们不包含前导零：

```js
> var arr = ['a', 'b'];
> arr['0']  // array index
'a'
> arr['00'] // normal property
undefined
```

## 长度

`length`属性的基本功能是跟踪数组中的最高索引：

```js
> [ 'a', 'b' ].length
2
> [ 'a', , 'b' ].length
3
```

因此，`length`不计算元素的数量，因此您必须编写自己的函数来执行此操作。例如：

```js
function countElements(arr) {
    var elemCount = 0;
    arr.forEach(function () {
        elemCount++;
    });
    return elemCount;
}
```

为了计算元素（非空洞），我们已经利用了`forEach`跳过空洞的事实。以下是互动：

```js
> countElements([ 'a', 'b' ])
2
> countElements([ 'a', , 'b' ])
2
```

### 手动增加数组的长度

手动增加数组的长度对数组几乎没有影响；它只会创建空洞：

```js
> var arr = [ 'a', 'b' ];
> arr.length = 3;
> arr  // one hole at the end
[ 'a', 'b', ,]
```

最后的结果末尾有两个逗号，因为尾随逗号是可选的，因此总是被忽略。

我们刚刚做的并没有添加任何元素：

```js
> countElements(arr)
2
```

然而，`length`属性确实作为指针，指示在哪里插入新元素。例如：

```js
> arr.push('c')
4
> arr
[ 'a', 'b', , 'c' ]
```

因此，通过`Array`构造函数设置数组的初始长度会创建一个完全空的数组：

```js
> var arr = new Array(2);
> arr.length
2
> countElements(arr)
0
```

### 减少数组的长度

如果您减少数组的长度，则新长度及以上的所有元素都将被删除：

```js
> var arr = [ 'a', 'b', 'c' ];
> 1 in arr
true
> arr[1]
'b'

> arr.length = 1;
> arr
[ 'a' ]
> 1 in arr
false
> arr[1]
undefined
```

#### 清除数组

如果将数组的长度设置为 0，则它将变为空。这样可以清除数组。例如：

```js
function clearArray(arr) {
    arr.length = 0;
}
```

以下是互动：

```js
> var arr = [ 'a', 'b', 'c' ];
> clearArray(arr)
> arr
[]
```

但是，请注意，这种方法可能会很慢，因为每个数组元素都会被显式删除。具有讽刺意味的是，创建一个新的空数组通常更快：

```js
arr = [];
```

#### 清除共享数组

您需要知道的是，将数组的长度设置为零会影响共享数组的所有人：

```js
> var a1 = [1, 2, 3];
> var a2 = a1;
> a1.length = 0;

> a1
[]
> a2
[]
```

相比之下，分配一个空数组不会：

```js
> var a1 = [1, 2, 3];
> var a2 = a1;
> a1 = [];

> a1
[]
> a2
[ 1, 2, 3 ]
```

### 最大长度

最大数组长度为 2³²−1：

```js
> var arr1 = new Array(Math.pow(2, 32));  // not ok
RangeError: Invalid array length

> var arr2 = new Array(Math.pow(2, 32)-1);  // ok
> arr2.push('x');
RangeError: Invalid array length
```

## 数组中的空洞

数组是从索引到值的映射。这意味着数组可以有*空洞*，即长度小于数组中缺失的索引。在这些索引中读取元素会返回`undefined`。

### 提示

建议避免数组中的空洞。JavaScript 对它们的处理不一致（即，一些方法忽略它们，其他方法不会）。幸运的是，通常你不需要知道如何处理空洞：它们很少有用，并且会对性能产生负面影响。

### 创建空洞

通过给数组索引赋值可以创建空洞：

```js
> var arr = [];
> arr[0] = 'a';
> arr[2] = 'c';
> 1 in arr  // hole at index 1
false
```

也可以通过在数组字面量中省略值来创建空洞：

```js
> var arr = ['a',,'c'];
> 1 in arr  // hole at index 1
false
```

### 警告

需要两个尾随逗号来创建尾随的空洞，因为最后一个逗号总是被忽略：

```js
> [ 'a', ].length
1
> [ 'a', ,].length
2
```

### 稀疏数组与密集数组

本节将检查空洞和`undefined`作为元素之间的区别。鉴于读取空洞会返回`undefined`，两者非常相似。

带有空洞的数组称为*稀疏*数组。没有空洞的数组称为*密集*数组。密集数组是连续的，并且在每个索引处都有一个元素——从零开始，到`length`-1 结束。让我们比较以下两个数组，一个是稀疏数组，一个是密集数组。这两者非常相似：

```js
var sparse = [ , , 'c' ];
var dense  = [ undefined, undefined, 'c' ];
```

空洞几乎就像在相同索引处有一个`undefined`元素。两个数组的长度都是一样的：

```js
> sparse.length
3
> dense.length
3
```

但是稀疏数组没有索引为 0 的元素：

```js
> 0 in sparse
false
> 0 in dense
true
```

通过`for`进行迭代对两个数组来说是一样的：

```js
> for (var i=0; i<sparse.length; i++) console.log(sparse[i]);
undefined
undefined
c
> for (var i=0; i<dense.length; i++) console.log(dense[i]);
undefined
undefined
c
```

通过`forEach`进行迭代会跳过空洞，但不会跳过未定义的元素：

```js
> sparse.forEach(function (x) { console.log(x) });
c
> dense.forEach(function (x) { console.log(x) });
undefined
undefined
c
```

### 哪些操作会忽略空洞，哪些会考虑它们？

涉及数组的一些操作会忽略空洞，而另一些会考虑它们。本节解释了细节。

#### 数组迭代方法

`forEach()`会跳过空洞：

```js
> ['a',, 'b'].forEach(function (x,i) { console.log(i+'.'+x) })
0.a
2.b
```

`every()`也会跳过空洞（类似的：`some()`）：

```js
> ['a',, 'b'].every(function (x) { return typeof x === 'string' })
true
```

`map()`会跳过，但保留空洞：

```js
> ['a',, 'b'].map(function (x,i) { return i+'.'+x })
[ '0.a', , '2.b' ]
```

`filter()`消除空洞：

```js
> ['a',, 'b'].filter(function (x) { return true })
[ 'a', 'b' ]
```

#### 其他数组方法

`join()`将空洞、`undefined`和`null`转换为空字符串：

```js
> ['a',, 'b'].join('-')
'a--b'
> [ 'a', undefined, 'b' ].join('-')
'a--b'
```

`sort()`在排序时保留空洞：

```js
> ['a',, 'b'].sort()  // length of result is 3
[ 'a', 'b', ,  ]
```

#### for-in 循环

`for-in`循环正确列出属性键（它们是数组索引的超集）：

```js
> for (var key in ['a',, 'b']) { console.log(key) }
0
2
```

#### Function.prototype.apply()

`apply()`将每个空洞转换为一个值为`undefined`的参数。以下交互演示了这一点：函数`f()`将其参数作为数组返回。当我们传递一个带有三个空洞的数组给`apply()`以调用`f()`时，后者接收到三个`undefined`参数：

```js
> function f() { return [].slice.call(arguments) }
> f.apply(null, [ , , ,])
[ undefined, undefined, undefined ]
```

这意味着我们可以使用`apply()`来创建一个带有`undefined`的数组：

```js
> Array.apply(null, Array(3))
[ undefined, undefined, undefined ]
```

### 警告

`apply()`将空洞转换为`undefined`在空数组中，但不能用于在任意数组中填补空洞（可能包含或不包含空洞）。例如，任意数组`[2]`：

```js
> Array.apply(null, [2])
[ , ,]
```

数组不包含任何空洞，所以`apply()`应该返回相同的数组。但实际上它返回一个长度为 2 的空数组（它只包含两个空洞）。这是因为`Array()`将单个数字解释为数组长度，而不是数组元素。

### 从数组中移除空洞

正如我们所见，`filter()`会移除空洞：

```js
> ['a',, 'b'].filter(function (x) { return true })
[ 'a', 'b' ]
```

使用自定义函数将任意数组中的空洞转换为`undefined`：

```js
function convertHolesToUndefineds(arr) {
    var result = [];
    for (var i=0; i < arr.length; i++) {
        result[i] = arr[i];
    }
    return result;
}
```

使用该函数：

```js
> convertHolesToUndefineds(['a',, 'b'])
[ 'a', undefined, 'b' ]
```

## 数组构造方法

`Array.isArray(obj)`

如果`obj`是数组则返回`true`。它正确处理跨*realms*（窗口或框架）的对象——与`instanceof`相反（参见[Pitfall: crossing realms (frames or windows)](ch17_split_001.html#cross-realm_instanceof "Pitfall: crossing realms (frames or windows)")）。

## 数组原型方法

在接下来的章节中，数组原型方法按功能分组。对于每个子章节，我会提到这些方法是*破坏性*的（它们会改变被调用的数组）还是*非破坏性*的（它们不会修改它们的接收者；这样的方法通常会返回新的数组）。

## 添加和删除元素（破坏性）

本节中的所有方法都是破坏性的：

`Array.prototype.shift()`

删除索引为 0 的元素并返回它。随后元素的索引减 1：

```js
> var arr = [ 'a', 'b' ];
> arr.shift()
'a'
> arr
[ 'b' ]
```

`Array.prototype.unshift(elem1?, elem2?, ...)`

将给定的元素添加到数组的开头。它返回新的长度：

```js
> var arr = [ 'c', 'd' ];
> arr.unshift('a', 'b')
4
> arr
[ 'a', 'b', 'c', 'd' ]
```

`Array.prototype.pop()`

移除数组的最后一个元素并返回它：

```js
> var arr = [ 'a', 'b' ];
> arr.pop()
'b'
> arr
[ 'a' ]
```

`Array.prototype.push(elem1?, elem2?, ...)`

将给定的元素添加到数组的末尾。它返回新的长度：

```js
> var arr = [ 'a', 'b' ];
> arr.push('c', 'd')
4
> arr
[ 'a', 'b', 'c', 'd' ]
```

`apply()`（参见[Function.prototype.apply(thisValue, argArray)](ch17_split_000.html#oop_apply "Function.prototype.apply(thisValue, argArray)")）使您能够破坏性地将数组`arr2`附加到另一个数组`arr1`：

```js
> var arr1 = [ 'a', 'b' ];
> var arr2 = [ 'c', 'd' ];

> Array.prototype.push.apply(arr1, arr2)
4
> arr1
[ 'a', 'b', 'c', 'd' ]
```

`Array.prototype.splice(start, deleteCount?, elem1?, elem2?, ...)`

从`start`开始，删除`deleteCount`个元素并插入给定的元素。换句话说，您正在用`elem1`、`elem2`等替换位置`start`处的`deleteCount`个元素。该方法返回已被移除的元素：

```js
> var arr = [ 'a', 'b', 'c', 'd' ];
> arr.splice(1, 2, 'X');
[ 'b', 'c' ]
> arr
[ 'a', 'X', 'd' ]
```

特殊的参数值：

+   `start`可以为负数，这种情况下它将被加到长度以确定起始索引。因此，`-1`指的是最后一个元素，依此类推。

+   `deleteCount`是可选的。如果省略（以及所有后续参数），则删除从索引`start`开始的所有元素及之后的所有元素。

在此示例中，我们删除最后两个索引后的所有元素：

```js
> var arr = [ 'a', 'b', 'c', 'd' ];
> arr.splice(-2)
[ 'c', 'd' ]
> arr
[ 'a', 'b' ]
```

## 排序和颠倒元素（破坏性）

这些方法也是破坏性的：

`Array.prototype.reverse()`

颠倒数组中元素的顺序并返回对原始（修改后的）数组的引用：

```js
> var arr = [ 'a', 'b', 'c' ];
> arr.reverse()
[ 'c', 'b', 'a' ]
> arr // reversing happened in place
[ 'c', 'b', 'a' ]
```

`Array.prototype.sort(compareFunction?)`

对数组进行排序并返回它：

```js
> var arr = ['banana', 'apple', 'pear', 'orange'];
> arr.sort()
[ 'apple', 'banana', 'orange', 'pear' ]
> arr  // sorting happened in place
[ 'apple', 'banana', 'orange', 'pear' ]
```

请记住，排序通过将值转换为字符串进行比较，这意味着数字不会按数字顺序排序：

```js
> [-1, -20, 7, 50].sort()
[ -1, -20, 50, 7 ]
```

您可以通过提供可选参数`compareFunction`来解决这个问题，它控制排序的方式。它具有以下签名：

```js
function compareFunction(a, b)
```

此函数比较`a`和`b`并返回：

+   如果`a`小于`b`，则返回小于零的整数（例如，`-1`）

+   如果`a`等于`b`，则返回零

+   如果`a`大于`b`，则返回大于零的整数（例如，`1`）

### 比较数字

对于数字，您可以简单地返回`a-b`，但这可能会导致数值溢出。为了防止这种情况发生，您需要更冗长的代码：

```js
function compareCanonically(a, b) {
    if (a < b) {
        return -1;
    } else if (a > b) {
        return 1;
    } else {
        return 0;
    }
}
```

我不喜欢嵌套的条件运算符。但在这种情况下，代码要简洁得多，我很想推荐它：

```js
function compareCanonically(a, b) {
    return return a < b ? -1 (a > b ? 1 : 0);
}
```

使用该函数：

```js
> [-1, -20, 7, 50].sort(compareCanonically)
[ -20, -1, 7, 50 ]
```

### 比较字符串

对于字符串，您可以使用`String.prototype.localeCompare`（参见[比较字符串](ch12.html#comparing_strings "比较字符串")）：

```js
> ['c', 'a', 'b'].sort(function (a,b) { return a.localeCompare(b) })
[ 'a', 'b', 'c' ]
```

### 比较对象

参数`compareFunction`对于排序对象也很有用：

```js
var arr = [
    { name: 'Tarzan' },
    { name: 'Cheeta' },
    { name: 'Jane' } ];

function compareNames(a,b) {
    return a.name.localeCompare(b.name);
}
```

使用`compareNames`作为比较函数，`arr`按`name`排序：

```js
> arr.sort(compareNames)
[ { name: 'Cheeta' },
  { name: 'Jane' },
  { name: 'Tarzan' } ]
```

## 连接、切片、连接（非破坏性）

以下方法对数组执行各种非破坏性操作：

`Array.prototype.concat(arr1?, arr2?, ...)`

创建一个新数组，其中包含接收器的所有元素，后跟数组`arr1`的所有元素，依此类推。如果其中一个参数不是数组，则将其作为元素添加到结果中（例如，这里的第一个参数`'c'`）：

```js
> var arr = [ 'a', 'b' ];
> arr.concat('c', ['d', 'e'])
[ 'a', 'b', 'c', 'd', 'e' ]
```

调用`concat()`的数组不会改变：

```js
> arr
[ 'a', 'b' ]
```

`Array.prototype.slice(begin?, end?)`

将数组元素复制到一个新数组中，从`begin`开始，直到`end`之前的元素：

```js
> [ 'a', 'b', 'c', 'd' ].slice(1, 3)
[ 'b', 'c' ]
```

如果缺少`end`，则使用数组长度：

```js
> [ 'a', 'b', 'c', 'd' ].slice(1)
[ 'b', 'c', 'd' ]
```

如果两个索引都缺失，则复制数组：

```js
> [ 'a', 'b', 'c', 'd' ].slice()
[ 'a', 'b', 'c', 'd' ]
```

如果任一索引为负数，则将数组长度加上它。因此，`-1`指的是最后一个元素，依此类推：

```js
> [ 'a', 'b', 'c', 'd' ].slice(1, -1)
[ 'b', 'c' ]
> [ 'a', 'b', 'c', 'd' ].slice(-2)
[ 'c', 'd' ]
```

`Array.prototype.join(separator?)`

通过对所有数组元素应用`toString()`并在结果之间放置`separator`字符串来创建一个字符串。如果省略`separator`，则使用`,`：

```js
> [3, 4, 5].join('-')
'3-4-5'
> [3, 4, 5].join()
'3,4,5'
> [3, 4, 5].join('')
'345'
```

`join()`将`undefined`和`null`转换为空字符串：

```js
> [undefined, null].join('#')
'#'
```

数组中的空位也会转换为空字符串：

```js
> ['a',, 'b'].join('-')
'a--b'
```

## 搜索值（非破坏性）

以下方法在数组中搜索值：

`Array.prototype.indexOf(searchValue, startIndex?)`

从`startIndex`开始搜索数组中的`searchValue`。它返回第一次出现的索引，如果找不到则返回-1。如果`startIndex`为负数，则将数组长度加上它；如果缺少`startIndex`，则搜索整个数组：

```js
> [ 3, 1, 17, 1, 4 ].indexOf(1)
1
> [ 3, 1, 17, 1, 4 ].indexOf(1, 2)
3
```

搜索时使用严格相等（参见[相等运算符：===与==](ch09.html#equality_operators "相等运算符：===与==")），这意味着`indexOf()`无法找到`NaN`：

```js
> [NaN].indexOf(NaN)
-1
```

`Array.prototype.lastIndexOf(searchElement, startIndex?)`

在`startIndex`开始向后搜索`searchElement`，返回第一次出现的索引或-1（如果找不到）。如果`startIndex`为负数，则将数组长度加上它；如果缺失，则搜索整个数组。搜索时使用严格相等（参见[相等运算符：===与==](ch09.html#equality_operators "相等运算符：===与==")）：

```js
> [ 3, 1, 17, 1, 4 ].lastIndexOf(1)
3
> [ 3, 1, 17, 1, 4 ].lastIndexOf(1, -3)
1
```

## 迭代（非破坏性）

迭代方法使用一个函数来迭代数组。我区分三种迭代方法，它们都是非破坏性的：*检查方法*主要观察数组的内容；*转换方法*从接收器派生一个新数组；*减少方法*基于接收器的元素计算结果。

### 检查方法

本节中描述的每个方法都是这样的：

```js
arr.examinationMethod(callback, thisValue?)
```

这样的方法需要以下参数：

+   `callback`是它的第一个参数，一个它调用的函数。根据检查方法的不同，回调返回布尔值或无返回值。它具有以下签名：

    ```js
    function callback(element, index, array)
    ```

`element`是`callback`要处理的数组元素，`index`是元素的索引，`array`是调用`examinationMethod`的数组。

+   `thisValue`允许您配置`callback`内部的`this`的值。

现在是我刚刚描述的检查方法的签名：

`Array.prototype.forEach(callback, thisValue?)`

迭代数组的元素：

```js
var arr = [ 'apple', 'pear', 'orange' ];
arr.forEach(function (elem) {
    console.log(elem);
});
```

`Array.prototype.every(callback, thisValue?)`

如果回调对每个元素返回`true`，则返回`true`。一旦回调返回`false`，迭代就会停止。请注意，不返回值会导致隐式返回`undefined`，`every()`将其解释为`false`。`every()`的工作方式类似于全称量词（“对于所有”）。

这个例子检查数组中的每个数字是否都是偶数：

```js
> function isEven(x) { return x % 2 === 0 }
> [ 2, 4, 6 ].every(isEven)
true
> [ 2, 3, 4 ].every(isEven)
false
```

如果数组为空，则结果为`true`（并且不调用`callback`）：

```js
> [].every(function () { throw new Error() })
true
```

`Array.prototype.some(callback, thisValue?)`

如果回调对至少一个元素返回`true`，则返回`true`。一旦回调返回`true`，迭代就会停止。请注意，不返回值会导致隐式返回`undefined`，`some()`将其解释为`false`。`some()`的工作方式类似于存在量词（“存在”）。

这个例子检查数组中是否有偶数：

```js
> function isEven(x) { return x % 2 === 0 }
> [ 1, 3, 5 ].some(isEven)
false
> [ 1, 2, 3 ].some(isEven)
true
```

如果数组为空，则结果为`false`（并且不调用`callback`）：

```js
> [].some(function () { throw new Error() })
false
```

`forEach()`的一个潜在陷阱是它不支持`break`或类似的东西来提前中止循环。如果你需要这样做，可以使用`some()`：

```js
function breakAtEmptyString(strArr) {
    strArr.some(function (elem) {
        if (elem.length === 0) {
            return true; // break
        }
        console.log(elem);
        // implicit: return undefined (interpreted as false)
    });
}
```

`some()`如果发生了中断，则返回`true`，否则返回`false`。这使您可以根据迭代是否成功完成（这在`for`循环中有点棘手）做出不同的反应。

### 转换方法

转换方法接受一个输入数组并产生一个输出数组，而回调控制输出的产生方式。回调的签名与检查方法相同：

```js
function callback(element, index, array)
```

有两种转换方法：

`Array.prototype.map(callback, thisValue?)`

每个输出数组元素是将`callback`应用于输入元素的结果。例如：

```js
> [ 1, 2, 3 ].map(function (x) { return 2 * x })
[ 2, 4, 6 ]
```

`Array.prototype.filter(callback, thisValue?)`

输出数组仅包含那些`callback`返回`true`的输入元素。例如：

```js
> [ 1, 0, 3, 0 ].filter(function (x) { return x !== 0 })
[ 1, 3 ]
```

### 减少方法

对于减少，回调具有不同的签名：

```js
function callback(previousValue, currentElement, currentIndex, array)
```

参数`previousValue`是回调先前返回的值。当首次调用回调时，有两种可能性（描述适用于`Array.prototype.reduce()`；与`reduceRight()`的差异在括号中提到）：

+   提供了明确的`initialValue`。然后`previousValue`是`initialValue`，`currentElement`是第一个数组元素（`reduceRight`：最后一个数组元素）。

+   没有提供明确的`initialValue`。然后`previousValue`是第一个数组元素，`currentElement`是第二个数组元素（`reduceRight`：最后一个数组元素和倒数第二个数组元素）。

有两种减少的方法：

`Array.prototype.reduce(callback, initialValue?)`

从左到右迭代，并像之前描述的那样调用回调。该方法的结果是回调返回的最后一个值。此示例计算所有数组元素的总和：

```js
function add(prev, cur) {
    return prev + cur;
}
console.log([10, 3, -1].reduce(add)); // 12
```

如果你在一个只有一个元素的数组上调用`reduce`，那么该元素会被返回：

```js
> [7].reduce(add)
7
```

如果你在一个空数组上调用`reduce`，你必须指定`initialValue`，否则你会得到一个异常：

```js
> [].reduce(add)
TypeError: Reduce of empty array with no initial value
> [].reduce(add, 123)
123
```

`Array.prototype.reduceRight(callback, initialValue?)`

与`reduce()`相同，但从右到左迭代。

### 注意

在许多函数式编程语言中，`reduce`被称为`fold`或`foldl`（左折叠），`reduceRight`被称为`foldr`（右折叠）。

看`reduce`方法的另一种方式是它实现了一个 n 元运算符`OP`：

`OP[1≤i≤n]` x[i]

通过一系列二元运算符`op2`的应用：

(...(x[1] `op2` x[2]) `op2` ...) `op2` x[n]

这就是前面代码示例中发生的事情：我们通过 JavaScript 的二进制加法运算符实现了一个数组的 n 元求和运算符。

例如，让我们通过以下函数来检查两个迭代方向：

```js
function printArgs(prev, cur, i) {
    console.log('prev:'+prev+', cur:'+cur+', i:'+i);
    return prev + cur;
}
```

如预期的那样，`reduce()`从左到右迭代：

```js
> ['a', 'b', 'c'].reduce(printArgs)
prev:a, cur:b, i:1
prev:ab, cur:c, i:2
'abc'
> ['a', 'b', 'c'].reduce(printArgs, 'x')
prev:x, cur:a, i:0
prev:xa, cur:b, i:1
prev:xab, cur:c, i:2
'xabc'
```

`reduceRight()`从右到左迭代：

```js
> ['a', 'b', 'c'].reduceRight(printArgs)
prev:c, cur:b, i:1
prev:cb, cur:a, i:0
'cba'
> ['a', 'b', 'c'].reduceRight(printArgs, 'x')
prev:x, cur:c, i:2
prev:xc, cur:b, i:1
prev:xcb, cur:a, i:0
'xcba'
```

## 陷阱：类数组对象

JavaScript 中的一些对象看起来像数组，但它们并不是数组。这通常意味着它们具有索引访问和`length`属性，但没有数组方法。例子包括特殊变量`arguments`，DOM 节点列表和字符串。[类数组对象和通用方法](ch17_split_001.html#array-like_objects "类数组对象和通用方法")提供了处理类数组对象的提示。

## 最佳实践：迭代数组

要迭代一个数组`arr`，你有两个选择：

+   一个简单的`for`循环（参见[for](ch13.html#for_loop "for")）：

    ```js
    for (var i=0; i<arr.length; i++) {
        console.log(arr[i]);
    }
    ```

+   数组迭代方法之一（参见[迭代（非破坏性）](ch18.html#array_iteration_methods "迭代（非破坏性）")）。例如，`forEach()`：

    ```js
    arr.forEach(function (elem) {
        console.log(elem);
    });
    ```

不要使用`for-in`循环（参见[for-in](ch13.html#for-in "for-in")）来迭代数组。它遍历索引，而不是值。在这样做的同时，它包括正常属性的键，包括继承的属性。

