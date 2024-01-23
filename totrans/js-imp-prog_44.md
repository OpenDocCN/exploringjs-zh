# 37 解构

> 原文：[`exploringjs.com/impatient-js/ch_destructuring.html`](https://exploringjs.com/impatient-js/ch_destructuring.html)

* * *

+   37.1 解构的第一印象

+   37.2 构造 vs. 提取

+   37.3 我们可以在哪里进行解构？

+   37.4 对象解构

    +   37.4.1 属性值简写

    +   37.4.2 剩余属性

    +   37.4.3 语法陷阱：通过对象解构进行赋值

+   37.5 数组解构

    +   37.5.1 数组解构适用于任何可迭代对象

    +   37.5.2 剩余元素

+   37.6 解构的例子

    +   37.6.1 数组解构：交换变量值

    +   37.6.2 数组解构：返回数组的操作

    +   37.6.3 对象解构：多个返回值

+   37.7 如果模式部分没有匹配到任何内容会发生什么？

    +   37.7.1 对象解构和缺失属性

    +   37.7.2 数组解构和缺失元素

+   37.8 哪些值不能被解构？

    +   37.8.1 你不能对`undefined`和`null`进行对象解构

    +   37.8.2 不能对非可迭代值进行数组解构

+   37.9 （高级）](ch_destructuring.html#advanced-4)

+   37.10 默认值

    +   37.10.1 数组解构中的默认值

    +   37.10.2 对象解构中的默认值

+   37.11 参数定义类似于解构

+   37.12 嵌套解构

* * *

### 37.1 解构的第一印象

通过普通赋值，您一次提取一个数据片段，例如：

```js
const arr = ['a', 'b', 'c'];
const x = arr[0]; // extract
const y = arr[1]; // extract
```

通过解构，您可以通过接收数据的位置中的模式同时提取多个数据片段。在前面的代码中，`=`的左侧就是这样一个位置。在下面的代码中，第 A 行的方括号是一个解构模式：

```js
const arr = ['a', 'b', 'c'];
const [x, y] = arr; // (A)
assert.equal(x, 'a');
assert.equal(y, 'b');
```

这段代码与前面的代码相同。

请注意，模式比数据“小”：我们只提取我们需要的部分。

### 37.2 构造 vs. 提取

为了理解解构是什么，考虑 JavaScript 有两种相反的操作：

+   您可以通过设置属性和对象字面量来*构造*复合数据。

+   您可以通过获取属性来*提取*复合数据的数据片段。

构造数据如下所示：

```js
// Constructing: one property at a time
const jane1 = {};
jane1.first = 'Jane';
jane1.last = 'Doe';

// Constructing: multiple properties
const jane2 = {
 first: 'Jane',
 last: 'Doe',
};

assert.deepEqual(jane1, jane2);
```

提取数据如下所示：

```js
const jane = {
 first: 'Jane',
 last: 'Doe',
};

// Extracting: one property at a time
const f1 = jane.first;
const l1 = jane.last;
assert.equal(f1, 'Jane');
assert.equal(l1, 'Doe');

// Extracting: multiple properties (NEW!)
const {first: f2, last: l2} = jane; // (A)
assert.equal(f2, 'Jane');
assert.equal(l2, 'Doe');
```

第 A 行的操作是新的：我们声明了两个变量`f2`和`l2`，并通过*解构*（多值提取）对它们进行初始化。

第 A 行的以下部分是一个*解构模式*：

```js
{first: f2, last: l2}
```

解构模式在语法上类似于用于多值构造的字面量。但它们出现在数据接收的地方（例如，在赋值的左侧），而不是数据创建的地方（例如，在赋值的右侧）。

### 37.3 我们可以在哪里进行解构？

解构模式可以在“数据接收位置”使用，例如：

+   变量声明：

    ```js
    const [a] = ['x'];
    assert.equal(a, 'x');

    let [b] = ['y'];
    assert.equal(b, 'y');
    ```

+   赋值：

    ```js
    let b;
    [b] = ['z'];
    assert.equal(b, 'z');
    ```

+   参数定义：

    ```js
    const f = ([x]) => x;
    assert.equal(f(['a']), 'a');
    ```

请注意，变量声明包括 `for-of` 循环中的 `const` 和 `let` 声明：

```js
const arr = ['a', 'b'];
for (const [index, element] of arr.entries()) {
 console.log(index, element);
}
// Output:
// 0, 'a'
// 1, 'b'
```

在接下来的两节中，我们将深入探讨两种解构：对象解构和数组解构。

### 37.4 对象解构

*对象解构* 允许你通过看起来像对象字面量的模式批量提取属性的值：

```js
const address = {
 street: 'Evergreen Terrace',
 number: '742',
 city: 'Springfield',
 state: 'NT',
 zip: '49007',
};

const { street: s, city: c } = address;
assert.equal(s, 'Evergreen Terrace');
assert.equal(c, 'Springfield');
```

你可以将模式看作是一个透明的图层，你将其放在数据上：模式键`'street'`在数据中有匹配。因此，数据值`'Evergreen Terrace'`被赋给模式变量`s`。

你也可以对原始值进行对象解构：

```js
const {length: len} = 'abc';
assert.equal(len, 3);
```

你也可以对数组进行对象解构：

```js
const {0:x, 2:y} = ['a', 'b', 'c'];
assert.equal(x, 'a');
assert.equal(y, 'c');
```

为什么会这样？数组索引也是属性。

#### 37.4.1 属性值简写

对象字面量支持属性值简写，对象模式也是如此：

```js
const { street, city } = address;
assert.equal(street, 'Evergreen Terrace');
assert.equal(city, 'Springfield');
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：对象解构**

`exercises/destructuring/object_destructuring_exrc.mjs`

#### 37.4.2 剩余属性

在对象字面量中，你可以有展开属性。在对象模式中，你可以有剩余属性（必须放在最后）：

```js
const obj = { a: 1, b: 2, c: 3 };
const { a: propValue, ...remaining } = obj; // (A)

assert.equal(propValue, 1);
assert.deepEqual(remaining, {b:2, c:3});
```

一个剩余属性变量，比如 `remaining`（第 A 行），被赋予一个包含未在模式中提及的所有数据属性的对象。

`remaining` 也可以被视为从 `obj` 中非破坏性地移除属性 `a` 的结果。

#### 37.4.3 语法陷阱：通过对象解构进行赋值

如果我们在赋值中使用对象解构，就会面临由语法歧义引起的陷阱 - 你不能以大括号开始一个语句，因为这样 JavaScript 会认为你正在开始一个代码块：

```js
let prop;
assert.throws(
 () => eval("{prop} = { prop: 'hello' };"),
 {
 name: 'SyntaxError',
 message: "Unexpected token '='",
 });
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **为什么使用 `eval()`?**

`eval()` 延迟解析（因此延迟了 `SyntaxError`）直到 `assert.throws()` 的回调被执行。如果我们不使用它，当这段代码被解析时就会出现错误，`assert.throws()` 甚至都不会被执行。

解决方法是将整个赋值放在括号中：

```js
let prop;
({prop} = { prop: 'hello' });
assert.equal(prop, 'hello');
```

### 37.5 数组解构

*数组解构* 允许你通过看起来像数组字面量的模式批量提取数组元素的值：

```js
const [x, y] = ['a', 'b'];
assert.equal(x, 'a');
assert.equal(y, 'b');
```

你可以通过在数组模式内部提及空位来跳过元素：

```js
const [, x, y] = ['a', 'b', 'c']; // (A)
assert.equal(x, 'b');
assert.equal(y, 'c');
```

在第 A 行的数组模式的第一个元素是一个空位，这就是为什么索引为 0 的数组元素被忽略了。

#### 37.5.1 数组解构适用于任何可迭代对象

数组解构可以应用于任何可迭代的值，而不仅仅是数组：

```js
// Sets are iterable
const mySet = new Set().add('a').add('b').add('c');
const [first, second] = mySet;
assert.equal(first, 'a');
assert.equal(second, 'b');

// Strings are iterable
const [a, b] = 'xyz';
assert.equal(a, 'x');
assert.equal(b, 'y');
```

#### 37.5.2 剩余元素

在数组字面量中，你可以有展开元素。在数组模式中，你可以有剩余元素（必须放在最后）：

```js
const [x, y, ...remaining] = ['a', 'b', 'c', 'd']; // (A)

assert.equal(x, 'a');
assert.equal(y, 'b');
assert.deepEqual(remaining, ['c', 'd']);
```

一个 rest 元素变量，比如 `remaining`（第 A 行），被赋予一个包含尚未提及的解构值的所有元素的数组。

### 37.6 解构的例子

#### 37.6.1 数组解构：交换变量值

你可以使用数组解构来交换两个变量的值，而不需要临时变量：

```js
let x = 'a';
let y = 'b';

[x,y] = [y,x]; // swap

assert.equal(x, 'b');
assert.equal(y, 'a');
```

#### 37.6.2 数组解构：返回数组的操作

当操作返回数组时，数组解构非常有用，例如正则表达式方法`.exec()`就返回数组：

```js
// Skip the element at index 0 (the whole match):
const [, year, month, day] =
 /^([0-9]{4})-([0-9]{2})-([0-9]{2})$/
 .exec('2999-12-31');

assert.equal(year, '2999');
assert.equal(month, '12');
assert.equal(day, '31');
```

#### 37.6.3 对象解构：多个返回值

如果一个函数返回多个值 - 无论是作为数组打包还是作为对象打包 - 解构非常有用。

考虑一个名为`findElement()`的函数，它在数组中查找元素：

```js
findElement(array, (value, index) => «boolean expression»)
```

它的第二个参数是一个接收元素的值和索引并返回一个布尔值指示这是否是调用者正在寻找的元素的函数。

现在我们面临一个两难选择：`findElement()`应该返回它找到的元素的值还是索引？一个解决方案是创建两个单独的函数，但这将导致重复的代码，因为这两个函数将非常相似。

以下实现通过返回一个包含找到的元素的索引和值的对象来避免重复：

```js
function findElement(arr, predicate) {
 for (let index=0; index < arr.length; index++) {
 const value = arr[index];
 if (predicate(value)) {
 // We found something:
 return { value, index };
 }
 }
 // We didn’t find anything:
 return { value: undefined, index: -1 };
}
```

解构帮助我们处理`findElement()`的结果：

```js
const arr = [7, 8, 6];

const {value, index} = findElement(arr, x => x % 2 === 0);
assert.equal(value, 8);
assert.equal(index, 1);
```

由于我们正在处理属性键，因此我们提到`value`和`index`的顺序并不重要：

```js
const {index, value} = findElement(arr, x => x % 2 === 0);
```

关键是，解构还可以很好地为我们服务，即使我们只对两个结果中的一个感兴趣：

```js
const arr = [7, 8, 6];

const {value} = findElement(arr, x => x % 2 === 0);
assert.equal(value, 8);

const {index} = findElement(arr, x => x % 2 === 0);
assert.equal(index, 1);
```

所有这些便利结合在一起，使得处理多个返回值的方式非常灵活。

### 37.7 如果模式部分没有匹配到任何内容会发生什么？

如果模式的一部分没有匹配，会发生什么？与使用非批处理运算符一样：您会得到`undefined`。

#### 37.7.1 对象解构和缺少属性

如果对象模式中的属性在右侧没有匹配项，则会得到`undefined`：

```js
const {prop: p} = {};
assert.equal(p, undefined);
```

#### 37.7.2 数组解构和缺少元素

如果数组模式中的元素在右侧没有匹配项，则会得到`undefined`：

```js
const [x] = [];
assert.equal(x, undefined);
```

### 37.8 哪些值不能被解构？

#### 37.8.1 您不能对`undefined`和`null`进行对象解构

只有当要解构的值是`undefined`或`null`时，对象解构才会失败。也就是说，每当通过点运算符访问属性失败时，解构也会失败。

```js
> const {prop} = undefined
TypeError: Cannot destructure property 'prop' of 'undefined'
as it is undefined.

> const {prop} = null
TypeError: Cannot destructure property 'prop' of 'null'
as it is null.
```

#### 37.8.2 您不能对非可迭代值进行数组解构

数组解构要求被解构的值是可迭代的。因此，您不能对`undefined`和`null`进行数组解构。但您也不能对非可迭代对象进行数组解构：

```js
> const [x] = {}
TypeError: {} is not iterable
```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **Quiz: basic**

请参阅 quiz app。

### 37.9 （高级）

所有剩余的部分都是高级的。

### 37.10 默认值

通常，如果模式没有匹配，相应的变量将被设置为`undefined`：

```js
const {prop: p} = {};
assert.equal(p, undefined);
```

如果要使用不同的值，需要指定*默认值*（通过`=`）：

```js
const {prop: p = 123} = {}; // (A)
assert.equal(p, 123);
```

在 A 行，我们指定了`p`的默认值为`123`。由于我们正在解构的数据中没有名为`prop`的属性，因此使用了默认值。

#### 37.10.1 数组解构中的默认值

在这里，我们有两个默认值分配给变量`x`和`y`，因为被解构的数组中对应的元素不存在。

```js
const [x=1, y=2] = [];

assert.equal(x, 1);
assert.equal(y, 2);
```

数组模式的第一个元素的默认值是`1`；第二个元素的默认值是`2`。

#### 37.10.2 对象解构中的默认值

您还可以为对象解构指定默认值：

```js
const {first: f='', last: l=''} = {};
assert.equal(f, '');
assert.equal(l, '');
```

在被解构的对象中，既没有属性键`first`也没有属性键`last`。因此，使用了默认值。

使用属性值简写，这段代码变得更简单：

```js
const {first='', last=''} = {};
assert.equal(first, '');
assert.equal(last, '');
```

### 37.11 参数定义类似于解构

考虑到我们在本章中学到的内容，参数定义与数组模式（剩余元素，默认值等）有很多共同之处。实际上，以下两个函数声明是等价的：

```js
function f1(«pattern1», «pattern2») {
 // ···
}

function f2(...args) {
 const [«pattern1», «pattern2»] = args;
 // ···
}
```

### 37.12 嵌套解构

到目前为止，我们只在解构模式中将变量用作*赋值目标*（数据接收器）。但是您也可以使用模式作为赋值目标，这使您可以将模式嵌套到任意深度：

```js
const arr = [
 { first: 'Jane', last: 'Bond' },
 { first: 'Lars', last: 'Croft' },
];
const [, {first}] = arr; // (A)
assert.equal(first, 'Lars');
```

在 A 行的数组模式中，索引 1 处有一个嵌套的对象模式。

嵌套模式可能会变得难以理解，因此最好适度使用。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **Quiz: advanced**

请参阅 quiz app。

[Comments](https://github.com/rauschma/impatient-js/issues/25)
