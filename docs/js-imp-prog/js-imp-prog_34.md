# 二十八、对象

> 原文：[`exploringjs.com/impatient-js/ch_objects.html`](https://exploringjs.com/impatient-js/ch_objects.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)



* * *

+   28.1 速查表：对象

    +   28.1.1 单个对象

    +   28.1.2 原型链

+   28.2 什么是对象？

    +   28.2.1 使用对象的两种方式

+   28.3 固定布局对象

    +   28.3.1 对象文字：属性

    +   28.3.2 对象文字：属性值简写

    +   28.3.3 获取属性

    +   28.3.4 设置属性

    +   28.3.5 对象文字：方法

    +   28.3.6 对象文字：访问器

+   28.4 [扩展到对象文字(`...`)[ES2018]](ch_objects.html#spreading-into-object-literals)

    +   28.4.1 扩展的用例：复制对象

    +   28.4.2 扩展的用例：缺失属性的默认值

    +   28.4.3 扩展的用例：非破坏性地更改属性

    +   28.4.4 [“破坏性扩展”：`Object.assign()`[ES6]](ch_objects.html#destructive-spreading-object.assign-es6)

+   28.5 方法和特殊变量`this`

    +   28.5.1 方法是其值为函数的属性

    +   28.5.2 特殊变量`this`

    +   28.5.3 方法和`.call()`

    +   28.5.4 方法和`.bind()`

    +   28.5.5 `this`陷阱：提取方法

    +   28.5.6 `this`陷阱：意外遮蔽`this`

    +   28.5.7 `this`在各种上下文中的值（高级）

+   28.6 [可选链接用于属性获取和方法调用[ES2020]（高级）](ch_objects.html#optional-chaining)

    +   28.6.1 示例：可选的固定属性获取

    +   28.6.2 更详细的操作符（高级）

    +   28.6.3 可选属性获取的短路

    +   28.6.4 可选链接：缺点和替代方案

    +   28.6.5 常见问题

+   28.7 字典对象（高级）

    +   28.7.1 对象文字中的引用键

    +   28.7.2 对象文字中的计算键

    +   28.7.3 `in`运算符：是否存在具有给定键的属性？

    +   28.7.4 删除属性

    +   28.7.5 可枚举性

    +   28.7.6 通过`Object.keys()`等列出属性键

    +   28.7.7 通过`Object.values()`列出属性值

    +   28.7.8 [通过`Object.entries()`列出属性条目[ES2017]](ch_objects.html#Object.entries)

    +   28.7.9 属性被确定性地列出

    +   28.7.10 [通过`Object.fromEntries()`组装对象[ES2019]](ch_objects.html#Object.fromEntries)

    +   28.7.11 使用对象作为字典的陷阱

+   28.8 属性特性和冻结对象（高级）

    +   28.8.1 [属性特性和属性描述符[ES5]](ch_objects.html#property-attributes-property-descriptors)

    +   28.8.2 [冻结对象[ES5]](ch_objects.html#freezing-objects-es5)

+   28.9 原型链

    +   28.9.1 JavaScript 的操作：所有属性 vs. 自有属性

    +   28.9.2 陷阱：原型链中只有第一个成员被改变

    +   28.9.3 使用原型的技巧（高级）

    +   28.9.4 [`Object.hasOwn()`: 给定属性是否为自有（非继承）？[ES2022]](ch_objects.html#Object.hasOwn)

    +   28.9.5 通过原型共享数据

+   28.10 FAQ：对象

    +   28.10.1 为什么对象保留属性的插入顺序？

* * *

在本书中，JavaScript 的面向对象编程（OOP）风格分四步介绍。本章涵盖了第 1 步和第 2 步；下一章涵盖了第 3 步和第 4 步。这些步骤是（图 8）：

1.  **单个对象（本章）：** *对象*，JavaScript 的基本 OOP 构建块，在孤立状态下如何工作？

1.  **原型链（本章）：** 每个对象都有零个或多个*原型对象*链。原型是 JavaScript 的核心继承机制。

1.  **类（下一章）：** JavaScript 的*类*是对象的工厂。类与其实例之间的关系基于原型继承（第 2 步）。

1.  **子类（下一章）：** *子类* 与 *超类* 之间的关系也是基于原型继承的。

![图 8：本书分四步介绍 JavaScript 中的面向对象编程。](img/77a1456d5bcb274b676345e01127a1b5.png)

图 8：本书分四步介绍 JavaScript 中的面向对象编程。

### 28.1 对象速查表

#### 28.1.1 单个对象

通过*对象字面量*创建对象（以大括号开始和结束）：

```js
const myObject = { // object literal
 myProperty: 1,
 myMethod() {
 return 2;
 }, // comma!
 get myAccessor() {
 return this.myProperty;
 }, // comma!
 set myAccessor(value) {
 this.myProperty = value;
 }, // last comma is optional
};

assert.equal(
 myObject.myProperty, 1
);
assert.equal(
 myObject.myMethod(), 2
);
assert.equal(
 myObject.myAccessor, 1
);
myObject.myAccessor = 3;
assert.equal(
 myObject.myProperty, 3
);
```

能够直接创建对象（无需类）是 JavaScript 的一个亮点。

扩展为对象：

```js
const original = {
 a: 1,
 b: {
 c: 3,
 },
};

// Spreading (...) copies one object “into” another one:
const modifiedCopy = {
 ...original, // spreading
 d: 4,
};

assert.deepEqual(
 modifiedCopy,
 {
 a: 1,
 b: {
 c: 3,
 },
 d: 4,
 }
);

// Caveat: spreading copies shallowly (property values are shared)
modifiedCopy.a = 5; // does not affect `original`
modifiedCopy.b.c = 6; // affects `original`
assert.deepEqual(
 original,
 {
 a: 1, // unchanged
 b: {
 c: 6, // changed
 },
 },
);
```

我们还可以使用扩展来制作对象的未修改（浅层）副本：

```js
const exactCopy = {...obj};
```

#### 28.1.2 原型链

原型是 JavaScript 的基本继承机制。甚至类也是基于它构建的。每个对象的原型都是`null`或一个对象。后者的对象也可以有原型，依此类推。通常，我们得到原型的*链*。

原型的管理方式如下：

```js
// `obj1` has no prototype (its prototype is `null`)
const obj1 = Object.create(null); // (A)
assert.equal(
 Object.getPrototypeOf(obj1), null // (B)
);

// `obj2` has the prototype `proto`
const proto = {
 protoProp: 'protoProp',
};
const obj2 = {
 __proto__: proto, // (C)
 objProp: 'objProp',
}
assert.equal(
 Object.getPrototypeOf(obj2), proto
);
```

注：

+   在创建对象时设置对象的原型：A 行，C 行

+   检索对象的原型：B 行

每个对象都继承其原型的所有属性：

```js
// `obj2` inherits .protoProp from `proto`
assert.equal(
 obj2.protoProp, 'protoProp'
);
assert.deepEqual(
 Reflect.ownKeys(obj2),
 ['objProp'] // own properties of `obj2`
);
```

对象的非继承属性称为其*自有*属性。

原型最重要的用例是多个对象可以通过从共同原型继承方法来共享它们。

### 28.2 什么是对象？

JavaScript 中的对象：

+   对象是一组槽（键值条目）。

+   公共槽称为*属性*：

    +   属性键只能是字符串或符号。

+   私有槽只能通过类创建，并在§29.2.4“公共槽（属性）vs. 私有槽”中进行了解。

#### 28.2.1 使用对象的两种方式

在 JavaScript 中有两种使用对象的方式：

+   固定布局对象：以这种方式使用，对象就像数据库中的记录一样工作。它们具有固定数量的属性，其键在开发时已知。它们的值通常具有不同的类型。

    ```js
    const fixedLayoutObject = {
     product: 'carrot',
     quantity: 4,
    };
    ```

+   字典对象：以这种方式使用，对象就像查找表或映射一样。它们具有可变数量的属性，其键在开发时未知。它们的所有值都具有相同的类型。

    ```js
    const dictionaryObject = {
     ['one']: 1,
     ['two']: 2,
    };
    ```

请注意，这两种方式也可以混合使用：有些对象既是固定布局对象，又是字典对象。

使用对象的方式会影响它们在本章中的解释：

+   首先，我们将探索固定布局对象。即使属性键在底层是字符串或符号，它们对我们来说将显示为固定标识符。

+   稍后，我们将探索字典对象。请注意，Maps 通常比对象更好地充当字典。但是，我们将遇到的一些操作对于固定布局对象也很有用。

### 28.3 固定布局对象

让我们首先探索*固定布局对象*。

#### 28.3.1 对象文字：属性

*对象文字*是创建固定布局对象的一种方式。它们是 JavaScript 的一个突出特点：我们可以直接创建对象-无需类！这是一个例子：

```js
const jane = {
 first: 'Jane',
 last: 'Doe', // optional trailing comma
};
```

在示例中，我们通过对象文字创建了一个对象，它以大括号`{}`开头和结尾。在其中，我们定义了两个*属性*（键值条目）：

+   第一个属性的键是`first`，值为`'Jane'`。

+   第二个属性的键是`last`，值为`'Doe'`。

自 ES5 以来，对象文字中允许使用尾随逗号。

我们将在以后看到指定属性键的其他方法，但是使用这种指定方式，它们必须遵循 JavaScript 变量名称的规则。例如，我们可以使用`first_name`作为属性键，但不能使用`first-name`）。但是，保留字是允许的。

```js
const obj = {
 if: true,
 const: true,
};
```

为了检查各种操作对对象的影响，我们将在本章的这一部分偶尔使用`Object.keys()`。它列出属性键：

```js
> Object.keys({a:1, b:2})
[ 'a', 'b' ]
```

#### 28.3.2 对象文字：属性值简写

每当属性的值是通过与键同名的变量定义的时候，我们可以省略键。

```js
function createPoint(x, y) {
 return {x, y}; // Same as: {x: x, y: y}
}
assert.deepEqual(
 createPoint(9, 2),
 { x: 9, y: 2 }
);
```

#### 28.3.3 获取属性

这是我们*获取*（读取）属性的方式（A 行）：

```js
const jane = {
 first: 'Jane',
 last: 'Doe',
};

// Get property .first
assert.equal(jane.first, 'Jane'); // (A)
```

获取未知属性会产生`undefined`：

```js
assert.equal(jane.unknownProperty, undefined);
```

#### 28.3.4 设置属性

这是我们*设置*（写入）属性的方式（A 行）：

```js
const obj = {
 prop: 1,
};
assert.equal(obj.prop, 1);
obj.prop = 2; // (A)
assert.equal(obj.prop, 2);
```

我们刚刚通过设置更改了现有属性。如果我们设置一个未知的属性，我们将创建一个新条目：

```js
const obj = {}; // empty object
assert.deepEqual(
 Object.keys(obj), []);

obj.unknownProperty = 'abc';
assert.deepEqual(
 Object.keys(obj), ['unknownProperty']);
```

#### 28.3.5 对象文字：方法

以下代码显示了如何通过对象文字创建方法`.says()`：

```js
const jane = {
 first: 'Jane', // value property
 says(text) {   // method
 return `${this.first} says “${text}”`; // (A)
 }, // comma as separator (optional at end)
};
assert.equal(jane.says('hello'), 'Jane says “hello”');
```

在方法调用`jane.says('hello')`期间，`jane`被称为方法调用的*接收者*，并分配给特殊变量`this`（有关`this`的更多信息，请参见§28.5“方法和特殊变量`this`”）。这使得方法`.says()`能够访问 A 行中的兄弟属性`.first`。

#### 28.3.6 对象文字：访问器

通过对象文字内部的语法定义*访问器*，看起来像方法：*getter*和/或*setter*（即，每个访问器都有一个或两个）。

调用访问器看起来像访问值属性：

+   读取属性会调用 getter。

+   写入属性会调用 setter。

##### 28.3.6.1 获取器

通过在方法定义前加上修饰符`get`来创建 getter：

```js
const jane = {
 first: 'Jane',
 last: 'Doe',
 get full() {
 return `${this.first}  ${this.last}`;
 },
};

assert.equal(jane.full, 'Jane Doe');
jane.first = 'John';
assert.equal(jane.full, 'John Doe');
```

##### 28.3.6.2 设置器

通过在方法定义前加上修饰符`set`来创建 setter：

```js
const jane = {
 first: 'Jane',
 last: 'Doe',
 set full(fullName) {
 const parts = fullName.split(' ');
 this.first = parts[0];
 this.last = parts[1];
 },
};

jane.full = 'Richard Roe';
assert.equal(jane.first, 'Richard');
assert.equal(jane.last, 'Roe');
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：通过对象文字创建对象**

`exercises/objects/color_point_object_test.mjs`

### 28.4 扩展到对象文字（`...`）[ES2018]

在对象文字中，*扩展属性*将另一个对象的属性添加到当前对象中：

```js
> const obj = {one: 1, two: 2};
> {...obj, three: 3}
{ one: 1, two: 2, three: 3 }
```

```js
const obj1 = {one: 1, two: 2};
const obj2 = {three: 3};
assert.deepEqual(
 {...obj1, ...obj2, four: 4},
 {one: 1, two: 2, three: 3,  four: 4}
);
```

如果属性键冲突，最后提到的属性将“获胜”：

```js
> const obj = {one: 1, two: 2, three: 3};
> {...obj, one: true}
{ one: true, two: 2, three: 3 }
> {one: true, ...obj}
{ one: 1, two: 2, three: 3 }
```

所有值都是可扩展的，甚至`undefined`和`null`：

```js
> {...undefined}
{}
> {...null}
{}
> {...123}
{}
> {...'abc'}
{ '0': 'a', '1': 'b', '2': 'c' }
> {...['a', 'b']}
{ '0': 'a', '1': 'b' }
```

字符串和数组的属性`.length`被隐藏在这种操作中（它不是*可枚举*的；有关更多信息，请参见[§28.8.1“属性属性和属性描述符[ES5]”](ch_objects.html#property-attributes-property-descriptors)）。

扩展包括其键为符号的属性（这些符号被`Object.keys()`，`Object.values()`和`Object.entries()`忽略）：

```js
const symbolKey = Symbol('symbolKey');
const obj = {
 stringKey: 1,
 [symbolKey]: 2,
};
assert.deepEqual(
 {...obj, anotherStringKey: 3},
 {
 stringKey: 1,
 [symbolKey]: 2,
 anotherStringKey: 3,
 }
);
```

#### 28.4.1 扩展的用例：复制对象

我们可以使用扩展来创建对象`original`的副本：

```js
const copy = {...original};
```

注意 - 复制是*浅层*的：`copy`是一个全新的对象，其中包含`original`的所有属性（键值条目）的副本。但是，如果属性值是对象，则这些对象本身不会被复制；它们在`original`和`copy`之间共享。让我们看一个例子：

```js
const original = { a: 1, b: {prop: true} };
const copy = {...original};
```

`copy`的第一级确实是一个副本：如果我们更改该级别的任何属性，它不会影响原始对象：

```js
copy.a = 2;
assert.deepEqual(
 original, { a: 1, b: {prop: true} }); // no change
```

但是，深层次的内容不会被复制。例如，`.b`的值在原始对象和副本之间共享。在副本中更改`.b`也会在原始对象中更改它。

```js
copy.b.prop = false;
assert.deepEqual(
 original, { a: 1, b: {prop: false} });
```

![](img/0ac255e56dc93a43365d8502301c8688.png) **JavaScript 没有内置支持深复制**

对象的*深复制*（其中所有级别都被复制）通常很难以通用方式实现。因此，JavaScript 目前没有内置操作。如果我们需要这样的操作，我们必须自己实现它。

#### 28.4.2 扩展的用例：缺少属性的默认值

如果我们的代码的输入之一是具有数据的对象，则可以通过指定默认值使属性变为可选，如果缺少这些属性，则使用这些默认值。其中一种方法是通过包含默认值的对象。在下面的示例中，该对象是`DEFAULTS`：

```js
const DEFAULTS = {alpha: 'a', beta: 'b'};
const providedData = {alpha: 1};

const allData = {...DEFAULTS, ...providedData};
assert.deepEqual(allData, {alpha: 1, beta: 'b'});
```

结果对象`allData`是通过复制`DEFAULTS`并用`providedData`的属性覆盖其属性而创建的。

但是我们不需要对象来指定默认值；我们也可以在对象文字中单独指定它们：

```js
const providedData = {alpha: 1};

const allData = {alpha: 'a', beta: 'b', ...providedData};
assert.deepEqual(allData, {alpha: 1, beta: 'b'});
```

#### 28.4.3 扩展的用例：非破坏性地更改属性

到目前为止，我们已经遇到了一种更改对象的属性`.alpha`的方法：我们*设置*它（A 行）并改变对象。也就是说，这种更改属性的方式是破坏性的。

```js
const obj = {alpha: 'a', beta: 'b'};
obj.alpha = 1; // (A)
assert.deepEqual(obj, {alpha: 1, beta: 'b'});
```

通过扩展，我们可以非破坏性地更改`.alpha` - 我们复制了`obj`的副本，其中`.alpha`具有不同的值：

```js
const obj = {alpha: 'a', beta: 'b'};
const updatedObj = {...obj, alpha: 1};
assert.deepEqual(updatedObj, {alpha: 1, beta: 'b'});
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：通过扩展（固定键）非破坏性地更新属性**

`exercises/objects/update_name_test.mjs`

#### 28.4.4 “破坏性扩展”：`Object.assign()` [ES6]

`Object.assign()`是一个工具方法：

```js
Object.assign(target, source_1, source_2, ···)
```

此表达式将`source_1`的所有属性分配给`target`，然后将`source_2`的所有属性等。最后，它返回`target` - 例如：

```js
const target = { a: 1 };

const result = Object.assign(
 target,
 {b: 2},
 {c: 3, b: true});

assert.deepEqual(
 result, { a: 1, b: true, c: 3 });
// target was modified and returned:
assert.equal(result, target);
```

`Object.assign()`的用例与扩展属性的用例类似。在某种程度上，它是破坏性地扩展。

### 28.5 方法和特殊变量`this`

#### 28.5.1 方法是其值为函数的属性

让我们重新访问用于介绍方法的示例：

```js
const jane = {
 first: 'Jane',
 says(text) {
 return `${this.first} says “${text}”`;
 },
};
```

有些令人惊讶的是，方法就是函数：

```js
assert.equal(typeof jane.says, 'function');
```

为什么？我们在可调用值章节中学到，普通函数扮演了几种角色。*方法*是其中之一。因此，在内部，`jane`大致如下。

```js
const jane = {
 first: 'Jane',
 says: function (text) {
 return `${this.first} says “${text}”`;
 },
};
```

#### 28.5.2 特殊变量`this`

考虑以下代码：

```js
const obj = {
 someMethod(x, y) {
 assert.equal(this, obj); // (A)
 assert.equal(x, 'a');
 assert.equal(y, 'b');
 }
};
obj.someMethod('a', 'b'); // (B)
```

在 B 行，`obj`是方法调用的*接收者*。它通过一个隐式（隐藏）参数传递给存储在`obj.someMethod`中的函数，其名称为`this`（A 行）。

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **如何理解`this`**

理解`this`的最佳方法是将其视为普通函数（因此也是方法）的隐式参数。

#### 28.5.3 方法和`.call()`

方法是函数，函数本身也有方法。其中之一是`.call()`。让我们看一个例子来了解这个方法是如何工作的。

在上一节中，有这种方法调用：

```js
obj.someMethod('a', 'b')
```

这种调用等同于：

```js
obj.someMethod.call(obj, 'a', 'b');
```

这也等同于：

```js
const func = obj.someMethod;
func.call(obj, 'a', 'b');
```

`.call()`使通常隐含的参数`this`变得显式：当通过`.call()`调用函数时，第一个参数是`this`，然后是常规（显式）函数参数。

顺便说一句，这意味着实际上有两个不同的点运算符：

1.  一个用于访问属性：`obj.prop`

1.  另一个用于调用方法：`obj.prop()`

它们的不同之处在于（2）不仅仅是（1）后面跟着函数调用运算符`()`。相反，（2）还提供了`this`的值。

#### 28.5.4 方法和`.bind()`

`.bind()`是函数对象的另一个方法。在下面的代码中，我们使用`.bind()`将方法`.says()`转换为独立的函数`func()`：

```js
const jane = {
 first: 'Jane',
 says(text) {
 return `${this.first} says “${text}”`; // (A)
 },
};

const func = jane.says.bind(jane, 'hello');
assert.equal(func(), 'Jane says “hello”');
```

通过`.bind()`将`this`设置为`jane`在这里至关重要。否则，`func()`将无法正常工作，因为`this`在 A 行中使用。在下一节中，我们将探讨为什么会这样。

#### 28.5.5 `this`陷阱：提取方法

我们现在对函数和方法有了相当多的了解，并且准备好看看涉及方法和`this`的最大陷阱：如果我们不小心，从对象中提取的方法进行函数调用可能会失败。

在以下示例中，当我们提取方法`jane.says()`，将其存储在变量`func`中，并调用`func`时，我们失败了。

```js
const jane = {
 first: 'Jane',
 says(text) {
 return `${this.first} says “${text}”`;
 },
};
const func = jane.says; // extract the method
assert.throws(
 () => func('hello'), // (A)
 {
 name: 'TypeError',
 message: "Cannot read properties of undefined (reading 'first')",
 });
```

在 A 行，我们正在进行普通函数调用。在普通函数调用中，`this`是`undefined`（如果严格模式激活，几乎总是激活的）。因此，A 行等价于：

```js
assert.throws(
 () => jane.says.call(undefined, 'hello'), // `this` is undefined!
 {
 name: 'TypeError',
 message: "Cannot read properties of undefined (reading 'first')",
 }
);
```

我们如何解决这个问题？我们需要使用`.bind()`来提取方法`.says()`：

```js
const func2 = jane.says.bind(jane);
assert.equal(func2('hello'), 'Jane says “hello”');
```

`.bind()`确保我们调用`func()`时`this`始终是`jane`。

我们还可以使用箭头函数来提取方法：

```js
const func3 = text => jane.says(text);
assert.equal(func3('hello'), 'Jane says “hello”');
```

##### 28.5.5.1 示例：提取一个方法

以下是我们在实际网页开发中可能看到的代码的简化版本：

```js
class ClickHandler {
 constructor(id, elem) {
 this.id = id;
 elem.addEventListener('click', this.handleClick); // (A)
 }
 handleClick(event) {
 alert('Clicked ' + this.id);
 }
}
```

在 A 行，我们没有正确提取方法`.handleClick()`。相反，我们应该这样做：

```js
const listener = this.handleClick.bind(this);
elem.addEventListener('click', listener);

// Later, possibly:
elem.removeEventListener('click', listener);
```

每次调用`.bind()`都会创建一个新函数。这就是为什么如果我们想要稍后删除它，就需要将结果存储在某个地方。

##### 28.5.5.2 如何避免提取方法的陷阱

遗憾的是，没有简单的方法可以避免提取方法的陷阱：每当我们提取一个方法时，都必须小心并正确地处理它 - 例如，通过绑定`this`或使用箭头函数。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：提取一个方法**

`exercises/objects/method_extraction_exrc.mjs`

#### 28.5.6 `this`陷阱：意外遮蔽`this`

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **意外遮蔽`this`只是普通函数的问题**

箭头函数不会遮蔽`this`。

考虑以下问题：当我们在普通函数内部时，我们无法访问周围范围的`this`，因为普通函数有它自己的`this`。换句话说，内部作用域中的变量隐藏了外部作用域中的变量。这就是所谓的*遮蔽*。以下代码是一个例子：

```js
const prefixer = {
 prefix: '==> ',
 prefixStringArray(stringArray) {
 return stringArray.map(
 function (x) {
 return this.prefix + x; // (A)
 });
 },
};
assert.throws(
 () => prefixer.prefixStringArray(['a', 'b']),
 {
 name: 'TypeError',
 message: "Cannot read properties of undefined (reading 'prefix')",
 }
);
```

在 A 行，我们想要访问`.prefixStringArray()`的`this`。但我们不能，因为周围的普通函数有它自己的`this`，*遮蔽*了（并阻止访问）方法的`this`。前者的`this`的值是`undefined`，因为回调函数被函数调用。这解释了错误消息。

修复这个问题的最简单方法是使用箭头函数，它没有自己的`this`，因此不会遮蔽任何东西：

```js
const prefixer = {
 prefix: '==> ',
 prefixStringArray(stringArray) {
 return stringArray.map(
 (x) => {
 return this.prefix + x;
 });
 },
};
assert.deepEqual(
 prefixer.prefixStringArray(['a', 'b']),
 ['==> a', '==> b']);
```

我们也可以将`this`存储在不同的变量中（A 行），这样它就不会被遮蔽：

```js
prefixStringArray(stringArray) {
 const that = this; // (A)
 return stringArray.map(
 function (x) {
 return that.prefix + x;
 });
},
```

另一个选择是通过`.bind()`（第 A 行）为回调函数指定一个固定的`this`：

```js
prefixStringArray(stringArray) {
 return stringArray.map(
 function (x) {
 return this.prefix + x;
 }.bind(this)); // (A)
},
```

最后，`.map()`让我们指定一个值作为`this`（A 行），在调用回调函数时使用：

```js
prefixStringArray(stringArray) {
 return stringArray.map(
 function (x) {
 return this.prefix + x;
 },
 this); // (A)
},
```

##### 28.5.6.1 避免意外遮蔽`this`的陷阱

如果我们遵循§25.3.4“建议：优先使用专门的函数而不是普通函数”中的建议，我们可以避免意外遮蔽`this`的陷阱。这是一个总结：

+   使用箭头函数作为匿名内联函数。它们没有`this`作为隐式参数，也不会遮蔽它。

+   对于命名的独立函数声明，我们可以使用箭头函数或函数声明。如果我们选择后者，就必须确保它们的主体中没有提到`this`。

#### 28.5.7 各种上下文中`this`的值（高级）

在各种上下文中`this`的值是多少？

在可调用实体内，`this`的值取决于可调用实体的调用方式和可调用实体的类型：

+   函数调用：

    +   普通函数：`this === undefined`（在严格模式下）

    +   箭头函数：`this`与周围作用域相同（词法`this`）

+   方法调用：`this`是调用的接收者

+   `new`：`this`指的是新创建的实例

我们还可以在所有常见的顶层作用域中访问`this`：

+   `<script>`元素：`this === globalThis`

+   ECMAScript 模块：`this === undefined`

+   CommonJS 模块：`this === module.exports`

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **提示：假装顶层作用域中不存在`this`**

我喜欢这样做是因为顶层的`this`很令人困惑，而且对于它的（少数）用例有更好的替代方案。

### 28.6 可选链用于属性获取和方法调用[ES2020]（高级）

存在以下种类的可选链操作：

```js
obj?.prop     // optional fixed property getting
obj?.[«expr»] // optional dynamic property getting
func?.(«arg0», «arg1») // optional function or method call
```

大致的想法是：

+   如果问号前的值既不是`undefined`也不是`null`，那么执行问号后的操作。

+   否则，返回`undefined`。

这三种语法的每一种都会在后面更详细地介绍。以下是一些最初的例子：

```js
> null?.prop
undefined
> {prop: 1}?.prop
1

> null?.(123)
undefined
> String?.(123)
'123'
```

#### 28.6.1 示例：可选的固定属性获取

考虑以下数据：

```js
const persons = [
 {
 surname: 'Zoe',
 address: {
 street: {
 name: 'Sesame Street',
 number: '123',
 },
 },
 },
 {
 surname: 'Mariner',
 },
 {
 surname: 'Carmen',
 address: {
 },
 },
];
```

我们可以使用可选链来安全地提取街道名称：

```js
const streetNames = persons.map(
 p => p.address?.street?.name);
assert.deepEqual(
 streetNames, ['Sesame Street', undefined, undefined]
);
```

##### 28.6.1.1 通过 nullish coalescing 处理默认值

nullish coalescing operator 允许我们使用默认值`'(no name)'`而不是`undefined`：

```js
const streetNames = persons.map(
 p => p.address?.street?.name ?? '(no name)');
assert.deepEqual(
 streetNames, ['Sesame Street', '(no name)', '(no name)']
);
```

#### 28.6.2 更详细的操作符（高级）

##### 28.6.2.1 可选的固定属性获取

以下两个表达式是等价的：

```js
o?.prop
(o !== undefined && o !== null) ? o.prop : undefined
```

例子：

```js
assert.equal(undefined?.prop, undefined);
assert.equal(null?.prop,      undefined);
assert.equal({prop:1}?.prop,  1);
```

##### 28.6.2.2 可选的动态属性获取

以下两个表达式是等价的：

```js
o?.[«expr»]
(o !== undefined && o !== null) ? o[«expr»] : undefined
```

例子：

```js
const key = 'prop';
assert.equal(undefined?.[key], undefined);
assert.equal(null?.[key], undefined);
assert.equal({prop:1}?.[key], 1);
```

##### 28.6.2.3 可选的函数或方法调用

以下两个表达式是等价的：

```js
f?.(arg0, arg1)
(f !== undefined && f !== null) ? f(arg0, arg1) : undefined
```

例子：

```js
assert.equal(undefined?.(123), undefined);
assert.equal(null?.(123), undefined);
assert.equal(String?.(123), '123');
```

请注意，如果可选链的左侧不可调用，则此运算符会产生错误：

```js
assert.throws(
 () => true?.(123),
 TypeError);
```

为什么？这个想法是，该运算符只容忍有意的遗漏。一个不可调用的值（除了`undefined`和`null`之外）可能是一个错误，应该报告而不是绕过。

#### 28.6.3 可选属性获取的短路

在一系列属性获取和方法调用中，一旦第一个可选运算符在其左侧遇到`undefined`或`null`，评估就会停止：

```js
function invokeM(value) {
 return value?.a.b.m(); // (A)
}

const obj = {
 a: {
 b: {
 m() { return 'result' }
 }
 }
};
assert.equal(
 invokeM(obj), 'result'
);
assert.equal(
 invokeM(undefined), undefined // (B)
);
```

在 B 行中考虑`invokeM(undefined)`：`undefined?.a`是`undefined`。因此我们期望 A 行中的`.b`失败。但实际上并不是：`?.`运算符遇到值`undefined`，整个表达式的评估立即返回`undefined`。

这种行为不同于普通运算符，JavaScript 总是在评估运算符之前评估所有操作数。这被称为*短路*。其他短路运算符包括：

+   `(a && b)`: 只有在`a`为真时才评估`b`。

+   `(a || b)`: 只有在`a`为假时才评估`b`。

+   `(c ? t : e)`: 如果`c`为真，则评估`t`。否则，评估`e`。

#### 28.6.4 可选链：缺点和替代方案

可选链也有缺点：

+   深度嵌套的结构更难管理。例如，如果有许多属性名称序列，重构会更加困难：每个序列都会强制多个对象的结构。

+   在访问数据时如此宽容会隐藏问题，这些问题将在后来显现，并且更难以调试。例如，早期出现的可选属性名称序列中的拼写错误会产生比正常拼写错误更多的负面影响。

可选链的另一种方法是在一个位置提取信息一次：

+   我们可以编写一个辅助函数来提取数据。

+   或者我们可以编写一个函数，其输入是深度嵌套的数据，输出是更简单、规范化的数据。

通过任一方法，都可以进行检查并在出现问题时提前失败。

进一步阅读：

+   “[过度防御性编程](https://medium.com/@vcarl/overly-defensive-programming-e7a1b3d234c2)” 由 [Carl Vitullo](https://twitter.com/vcarl_)

+   [Twitter 上的讨论](https://twitter.com/housecor/status/1088419498846244864) 由 [Cory House](https://twitter.com/housecor)

#### 28.6.5 经常问的问题

##### 28.6.5.1 可选链操作符（`?.`）的好记忆法是什么？

您是否偶尔不确定可选链操作符是以点号（`.?`）还是问号（`?.`）开始的？那么这个记忆法可能会帮助您：

+   IF（`?`）左侧不是 nullish

+   THEN（`.`）访问属性。

##### 28.6.5.2 为什么 `o?.[x]` 和 `f?.()` 中有点号？

以下两个可选操作符的语法并不理想：

```js
obj?.[«expr»]          // better: obj?[«expr»]
func?.(«arg0», «arg1») // better: func?(«arg0», «arg1»)
```

不幸的是，不够优雅的语法是必要的，因为区分理想的语法（第一个表达式）和条件运算符（第二个表达式）太复杂了：

```js
obj?['a', 'b', 'c'].map(x => x+x)
obj ? ['a', 'b', 'c'].map(x => x+x) : []
```

##### 28.6.5.3 为什么 `null?.prop` 的计算结果是 `undefined` 而不是 `null`？

操作符 `?.` 主要关注其右侧：属性 `.prop` 存在吗？如果不存在，就提前停止。因此，保留左侧的信息很少有用。然而，只有一个“提前终止”值确实简化了事情。

### 28.7 字典对象（高级）

对象最适合作为固定布局的对象。但在 ES6 之前，JavaScript 没有字典的数据结构（ES6 带来了 Maps）。因此，对象必须被用作字典，这带来了一个重要的限制：字典键必须是字符串（ES6 也引入了符号）。

首先，我们看一下与字典相关的对象的特性，但也适用于固定布局的对象。本节以实际使用对象作为字典的提示结束。（提示：如果可能的话，最好使用 Maps。）

#### 28.7.1 对象字面量中的引用键

到目前为止，我们一直使用固定布局的对象。属性键是固定的标记，必须是有效的标识符，并在内部变为字符串：

```js
const obj = {
 mustBeAnIdentifier: 123,
};

// Get property
assert.equal(obj.mustBeAnIdentifier, 123);

// Set property
obj.mustBeAnIdentifier = 'abc';
assert.equal(obj.mustBeAnIdentifier, 'abc');
```

作为下一步，我们将超越属性键的这种限制：在本小节中，我们将使用任意固定字符串作为键。在下一小节中，我们将动态计算键。

两种语法使我们能够使用任意字符串作为属性键。

首先，在通过对象字面量创建属性键时，我们可以引用属性键（使用单引号或双引号）：

```js
const obj = {
 'Can be any string!': 123,
};
```

其次，在获取或设置属性时，我们可以使用带有字符串的方括号：

```js
// Get property
assert.equal(obj['Can be any string!'], 123);

// Set property
obj['Can be any string!'] = 'abc';
assert.equal(obj['Can be any string!'], 'abc');
```

我们也可以使用这些语法来定义方法：

```js
const obj = {
 'A nice method'() {
 return 'Yes!';
 },
};

assert.equal(obj['A nice method'](), 'Yes!');
```

#### 28.7.2 对象字面量中的计算键

在上一小节中，属性键是通过对象字面量中的固定字符串指定的。在本节中，我们将学习如何动态计算属性键。这使我们能够使用任意字符串或符号。

对象字面量中动态计算的属性键的语法受到动态访问属性的启发。也就是说，我们可以使用方括号来包装表达式：

```js
const obj = {
 ['Hello world!']: true,
 ['p'+'r'+'o'+'p']: 123,
 [Symbol.toStringTag]: 'Goodbye', // (A)
};

assert.equal(obj['Hello world!'], true);
assert.equal(obj.prop, 123);
assert.equal(obj[Symbol.toStringTag], 'Goodbye');
```

计算键的主要用例是将符号作为属性键（A 行）。

请注意，用于获取和设置属性的方括号操作符可以使用任意表达式：

```js
assert.equal(obj['p'+'r'+'o'+'p'], 123);
assert.equal(obj['==> prop'.slice(4)], 123);
```

方法也可以有计算属性键：

```js
const methodKey = Symbol();
const obj = {
 [methodKey]() {
 return 'Yes!';
 },
};

assert.equal(obj[methodKey](), 'Yes!');
```

在本章的其余部分，我们将主要再次使用固定属性键（因为它们在语法上更方便）。但所有特性也适用于任意字符串和符号。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：通过展开（计算键）非破坏性地更新属性**

`exercises/objects/update_property_test.mjs`

#### 28.7.3 `in`运算符：是否存在具有给定键的属性？

`in`运算符检查对象是否具有具有给定键的属性：

```js
const obj = {
 alpha: 'abc',
 beta: false,
};

assert.equal('alpha' in obj, true);
assert.equal('beta' in obj, true);
assert.equal('unknownKey' in obj, false);
```

##### 28.7.3.1 通过真值检查属性是否存在

我们也可以使用真值检查来确定属性是否存在：

```js
assert.equal(
 obj.alpha ? 'exists' : 'does not exist',
 'exists');
assert.equal(
 obj.unknownKey ? 'exists' : 'does not exist',
 'does not exist');
```

前面的检查有效是因为`obj.alpha`是真值，并且读取一个不存在的属性会返回`undefined`（假值）。

然而，有一个重要的警告：如果属性存在但具有假值（`undefined`、`null`、`false`、`0`、`""`等），真值检查会失败：

```js
assert.equal(
 obj.beta ? 'exists' : 'does not exist',
 'does not exist'); // should be: 'exists'
```

#### 28.7.4 删除属性

我们可以通过`delete`运算符删除属性：

```js
const obj = {
 myProp: 123,
};

assert.deepEqual(Object.keys(obj), ['myProp']);
delete obj.myProp;
assert.deepEqual(Object.keys(obj), []);
```

#### 28.7.5 可枚举性

*可枚举性*是属性的*属性*。一些操作会忽略不可枚举的属性，例如`Object.keys()`和属性展开时。默认情况下，大多数属性是可枚举的。下一个例子展示了如何改变它以及它如何影响属性展开。

```js
const enumerableSymbolKey = Symbol('enumerableSymbolKey');
const nonEnumSymbolKey = Symbol('nonEnumSymbolKey');

// We create enumerable properties via an object literal
const obj = {
 enumerableStringKey: 1,
 [enumerableSymbolKey]: 2,
}

// For non-enumerable properties, we need a more powerful tool
Object.defineProperties(obj, {
 nonEnumStringKey: {
 value: 3,
 enumerable: false,
 },
 [nonEnumSymbolKey]: {
 value: 4,
 enumerable: false,
 },
});

// Non-enumerable properties are ignored by spreading:
assert.deepEqual(
 {...obj},
 {
 enumerableStringKey: 1,
 [enumerableSymbolKey]: 2,
 }
);
```

`Object.defineProperties()`在本章后面有解释。下一小节展示了这些操作如何受到可枚举性的影响：

#### 28.7.6 通过`Object.keys()`等列出属性键

表 19：列出*自有*（非继承的）属性键的标准库方法。它们都返回包含字符串和/或符号的数组。

|  | 可枚举 | 非可枚举 | 字符串 | 符号 |
| --- | --- | --- | --- | --- |
| `Object.keys()` | `✔` |  | `✔` |  |
| `Object.getOwnPropertyNames()` | `✔` | `✔` | `✔` |  |
| `Object.getOwnPropertySymbols()` | `✔` | `✔` |  | `✔` |
| `Reflect.ownKeys()` | `✔` | `✔` | `✔` | `✔` |

tbl. 19 中的每个方法都返回一个参数的自有属性键的数组。在方法的名称中，我们可以看到以下区别：

+   *属性键*可以是字符串或符号。

+   *属性名*是其值为字符串的属性键。

+   *属性符号*是其值为符号的属性键。

为了演示这四个操作，我们重新访问上一小节的例子：

```js
const enumerableSymbolKey = Symbol('enumerableSymbolKey');
const nonEnumSymbolKey = Symbol('nonEnumSymbolKey');

const obj = {
 enumerableStringKey: 1,
 [enumerableSymbolKey]: 2,
}
Object.defineProperties(obj, {
 nonEnumStringKey: {
 value: 3,
 enumerable: false,
 },
 [nonEnumSymbolKey]: {
 value: 4,
 enumerable: false,
 },
});

assert.deepEqual(
 Object.keys(obj),
 ['enumerableStringKey']
);
assert.deepEqual(
 Object.getOwnPropertyNames(obj),
 ['enumerableStringKey', 'nonEnumStringKey']
);
assert.deepEqual(
 Object.getOwnPropertySymbols(obj),
 [enumerableSymbolKey, nonEnumSymbolKey]
);
assert.deepEqual(
 Reflect.ownKeys(obj),
 [
 'enumerableStringKey', 'nonEnumStringKey',
 enumerableSymbolKey, nonEnumSymbolKey,
 ]
);
```

#### 28.7.7 通过`Object.values()`列出属性值

`Object.values()`列出对象的所有可枚举的字符串键属性的值：

```js
const firstName = Symbol('firstName');
const obj = {
 [firstName]: 'Jane',
 lastName: 'Doe',
};
assert.deepEqual(
 Object.values(obj),
 ['Doe']);
```

#### 28.7.8 通过`Object.entries()`列出属性条目[ES2017]

`Object.entries()`列出所有可枚举的字符串键属性作为键值对。每对被编码为一个两元素数组：

```js
const firstName = Symbol('firstName');
const obj = {
 [firstName]: 'Jane',
 lastName: 'Doe',
};
assert.deepEqual(
 Object.entries(obj),
 [
 ['lastName', 'Doe'],
 ]);
```

##### 28.7.8.1 `Object.entries()`的简单实现

下面的函数是`Object.entries()`的简化版本：

```js
function entries(obj) {
 return Object.keys(obj)
 .map(key => [key, obj[key]]);
}
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：`Object.entries()`**

`exercises/objects/find_key_test.mjs`

#### 28.7.9 属性被有确定性地列出

对象的自有（非继承的）属性总是按照以下顺序列出：

1.  具有包含整数索引的字符串键的属性（包括数组索引）：

    按升序排列的数字顺序

1.  剩余的具有字符串键的属性：

    按照它们被添加的顺序

1.  具有符号键的属性：

    按照它们被添加的顺序

下面的例子演示了如何根据这些规则对属性键进行排序：

```js
> Object.keys({b:0,a:0, 10:0,2:0})
[ '2', '10', 'b', 'a' ]
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **属性的顺序**

[ECMAScript 规范](https://tc39.github.io/ecma262/#sec-ordinaryownpropertykeys)更详细地描述了属性的排序方式。

#### 28.7.10 通过`Object.fromEntries()`组装对象[ES2019]

给定一个[key, value]对的可迭代对象，`Object.fromEntries()`创建一个对象：

```js
const symbolKey = Symbol('symbolKey');
assert.deepEqual(
 Object.fromEntries(
 [
 ['stringKey', 1],
 [symbolKey, 2],
 ]
 ),
 {
 stringKey: 1,
 [symbolKey]: 2,
 }
);
```

`Object.fromEntries()`执行与`Object.entries()`相反的操作。然而，`Object.entries()`忽略了符号键属性，而`Object.fromEntries()`则不会（见上一个例子）。

为了展示两者，我们将使用它们来实现库[Underscore](https://underscorejs.org)中的两个工具函数在接下来的子子部分中。

##### 28.7.10.1 示例：`pick()`

[Underscore 函数`pick()`](https://underscorejs.org/#pick)具有以下签名：

```js
pick(object, ...keys)
```

它返回一个`object`的副本，其中只有那些键在尾随参数中提到的属性：

```js
const address = {
 street: 'Evergreen Terrace',
 number: '742',
 city: 'Springfield',
 state: 'NT',
 zip: '49007',
};
assert.deepEqual(
 pick(address, 'street', 'number'),
 {
 street: 'Evergreen Terrace',
 number: '742',
 }
);
```

我们可以实现`pick()`如下：

```js
function pick(object, ...keys) {
 const filteredEntries = Object.entries(object)
 .filter(([key, _value]) => keys.includes(key));
 return Object.fromEntries(filteredEntries);
}
```

##### 28.7.10.2 示例：`invert()`

[Underscore 函数`invert()`](https://underscorejs.org/#invert)具有以下签名：

```js
invert(object)
```

它返回一个`object`的副本，其中所有属性的键和值被交换：

```js
assert.deepEqual(
 invert({a: 1, b: 2, c: 3}),
 {1: 'a', 2: 'b', 3: 'c'}
);
```

我们可以这样实现`invert()`：

```js
function invert(object) {
 const reversedEntries = Object.entries(object)
 .map(([key, value]) => [value, key]);
 return Object.fromEntries(reversedEntries);
}
```

##### 28.7.10.3 `Object.fromEntries()`的简单实现

以下函数是`Object.fromEntries()`的简化版本：

```js
function fromEntries(iterable) {
 const result = {};
 for (const [key, value] of iterable) {
 let coercedKey;
 if (typeof key === 'string' || typeof key === 'symbol') {
 coercedKey = key;
 } else {
 coercedKey = String(key);
 }
 result[coercedKey] = value;
 }
 return result;
}
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：使用`Object.entries()`和`Object.fromEntries()`**

`exercises/objects/omit_properties_test.mjs`

#### 28.7.11 使用对象作为字典的陷阱

如果我们使用普通对象（通过对象文字创建）作为字典，我们必须注意两个陷阱。

第一个陷阱是`in`运算符也会找到继承的属性：

```js
const dict = {};
assert.equal('toString' in dict, true);
```

我们希望`dict`被视为空，但`in`运算符检测到它从其原型`Object.prototype`继承的属性。

第二个陷阱是我们不能使用属性键`__proto__`，因为它具有特殊的功能（它设置对象的原型）：

```js
const dict = {};

dict['__proto__'] = 123;
// No property was added to dict:
assert.deepEqual(Object.keys(dict), []);
```

##### 28.7.11.1 安全地使用对象作为字典

那么我们如何避免这两个陷阱呢？

+   如果可以的话，我们使用 Maps。它们是字典的最佳解决方案。

+   如果我们不能，我们使用一个对象作为字典的库，以防止我们犯错。

+   如果不可能或不希望这样做，我们使用一个没有原型的对象。

以下代码演示了如何使用没有原型的对象作为字典：

```js
const dict = Object.create(null); // prototype is `null`

assert.equal('toString' in dict, false); // (A)

dict['__proto__'] = 123;
assert.deepEqual(Object.keys(dict), ['__proto__']);
```

我们避免了两个陷阱：

+   首先，没有原型的属性不继承任何属性（行 A）。

+   其次，在现代 JavaScript 中，`__proto__`是通过`Object.prototype`实现的。这意味着如果原型链中没有`Object.prototype`，它就会被关闭。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：使用对象作为字典**

`exercises/objects/simple_dict_test.mjs`

### 28.8 属性属性和冻结对象（高级）

#### 28.8.1 属性属性和属性描述符[ES5]

就像对象由属性组成一样，属性由*属性*组成。属性的值只是几个属性中的一个。其他包括：

+   `writable`：是否可以更改属性的值？

+   `enumerable`：属性是否被`Object.keys()`，扩展等考虑？

当我们使用一个操作来处理属性属性时，属性是通过*属性描述符*指定的：每个属性代表一个属性的对象。例如，这是我们如何读取属性`obj.myProp`的属性：

```js
const obj = { myProp: 123 };
assert.deepEqual(
 Object.getOwnPropertyDescriptor(obj, 'myProp'),
 {
 value: 123,
 writable: true,
 enumerable: true,
 configurable: true,
 });
```

这就是我们如何更改`obj.myProp`的属性：

```js
assert.deepEqual(Object.keys(obj), ['myProp']);

// Hide property `myProp` from Object.keys()
// by making it non-enumerable
Object.defineProperty(obj, 'myProp', {
 enumerable: false,
});

assert.deepEqual(Object.keys(obj), []);
```

**进一步阅读：**

+   可枚举性在本章的前面有更详细的介绍。

+   有关属性属性和属性描述符的更多信息，请参阅[*深入 JavaScript*](https://exploringjs.com/deep-js/ch_property-attributes-intro.html)。

#### 28.8.2 冻结对象[ES5]

`Object.freeze(obj)`使`obj`完全不可变：我们不能更改属性，添加属性，或更改其原型 - 例如：

```js
const frozen = Object.freeze({ x: 2, y: 5 });
assert.throws(
 () => { frozen.x = 7 },
 {
 name: 'TypeError',
 message: /^Cannot assign to read only property 'x'/,
 });
```

在幕后，`Object.freeze()`改变属性（例如，使它们不可写）和对象（例如，使它们*不可扩展*，意味着不能再添加属性）的属性。

有一个警告：`Object.freeze(obj)`只能浅冻结。也就是说，只有`obj`的属性被冻结，而不是存储在属性中的对象。

![](img/aec4653f22c8cf0e517ff5024759dfe1.png) **更多信息**

有关冻结和其他锁定对象的方法的更多信息，请参阅[*深入 JavaScript*](https://exploringjs.com/deep-js/ch_protecting-objects.html)。

### 28.9 原型链

原型是 JavaScript 的唯一继承机制：每个对象都有一个原型，要么是`null`，要么是一个对象。在后一种情况下，对象继承原型的所有属性。

在对象文字中，我们可以通过特殊属性`__proto__`设置原型：

```js
const proto = {
 protoProp: 'a',
};
const obj = {
 __proto__: proto,
 objProp: 'b',
};

// obj inherits .protoProp:
assert.equal(obj.protoProp, 'a');
assert.equal('protoProp' in obj, true);
```

鉴于原型对象本身可以有一个原型，我们得到了一系列对象-所谓的*原型链*。继承给我们一种印象，即我们正在处理单个对象，但实际上我们正在处理对象链。

图 9 显示了`obj`的原型链是什么样子。

![图 9：obj 开始了一个包含 proto 和其他对象的对象链。](img/3dab4bfabd32003cdb2479f38899b9ab.png)

图 9：`obj`开始了一个包含`proto`和其他对象的对象链。

非继承属性称为*自有属性*。`obj`有一个自有属性`.objProp`。

#### 28.9.1 JavaScript 的操作：所有属性与自有属性

一些操作考虑所有属性（自有和继承的）-例如，获取属性：

```js
> const obj = { one: 1 };
> typeof obj.one // own
'number'
> typeof obj.toString // inherited
'function'
```

其他操作只考虑自有属性-例如，`Object.keys()`：

```js
> Object.keys(obj)
[ 'one' ]
```

继续阅读另一个操作，该操作也只考虑自有属性：设置属性。

#### 28.9.2 陷阱：只有原型链的第一个成员被改变

给定一个具有原型对象链的对象`obj`，设置`obj`的自有属性只改变`obj`是有意义的。但是，通过`obj`设置继承属性也只会改变`obj`。它在`obj`中创建一个新的自有属性，覆盖了继承属性。让我们看看如何在以下对象中工作：

```js
const proto = {
 protoProp: 'a',
};
const obj = {
 __proto__: proto,
 objProp: 'b',
};
```

在下一个代码片段中，我们设置了继承属性`obj.protoProp`（A 行）。这通过创建自有属性“改变”了它：当读取`obj.protoProp`时，首先找到自有属性，其值*覆盖*了继承属性的值。

```js
// In the beginning, obj has one own property
assert.deepEqual(Object.keys(obj), ['objProp']);

obj.protoProp = 'x'; // (A)

// We created a new own property:
assert.deepEqual(Object.keys(obj), ['objProp', 'protoProp']);

// The inherited property itself is unchanged:
assert.equal(proto.protoProp, 'a');

// The own property overrides the inherited property:
assert.equal(obj.protoProp, 'x');
```

`obj`的原型链如图所示。10。

![图 10：obj 的自有属性.protoProp 覆盖了从 proto 继承的属性。](img/488c29553b4afa118ff5630e10e3fee4.png)

图 10：`obj`的自有属性`.protoProp`覆盖了从`proto`继承的属性。

#### 28.9.3 处理原型的提示（高级）

##### 28.9.3.1 获取和设置原型

`__proto__`的建议：

+   不要将`__proto__`用作伪属性（`Object`的所有实例的 setter）：

    +   它不能与所有对象一起使用（例如不是`Object`的实例的对象）。

    +   语言规范已经将其弃用。

    有关此功能的更多信息，请参阅§29.8.7“`Object.prototype.__proto__`（访问器）”。

+   在对象文字中使用`__proto__`设置原型是不同的：这是对象文字的一个特性，没有陷阱。

获取和设置原型的推荐方法是：

+   获取对象的原型：

    ```js
    Object.getPrototypeOf(obj: Object) : Object
    ```

+   在创建对象时，设置对象的原型的最佳时间是。我们可以通过对象文字中的`__proto__`或通过以下方式来实现：

    ```js
    Object.create(proto: Object) : Object
    ```

    如果必须，我们可以使用`Object.setPrototypeOf()`来更改现有对象的原型。但这可能会对性能产生负面影响。

这就是这些特性的使用方式：

```js
const proto1 = {};
const proto2a = {};
const proto2b = {};

const obj1 = {
 __proto__: proto1,
 a: 1,
 b: 2,
};
assert.equal(Object.getPrototypeOf(obj1), proto1);

const obj2 = Object.create(
 proto2a,
 {
 a: {
 value: 1,
 writable: true,
 enumerable: true,
 configurable: true,
 },
 b: {
 value: 2,
 writable: true,
 enumerable: true,
 configurable: true,
 }, 
 }
);
assert.equal(Object.getPrototypeOf(obj2), proto2a);

Object.setPrototypeOf(obj2, proto2b);
assert.equal(Object.getPrototypeOf(obj2), proto2b);
```

##### 28.9.3.2 检查一个对象是否在另一个对象的原型链中

到目前为止，“`proto`是`obj`的原型”总是意味着“`proto`是`obj`的*直接*原型”。但它也可以更松散地使用，并意味着`proto`在`obj`的原型链中。可以通过`.isPrototypeOf()`检查这种更松散的关系：

例如：

```js
const a = {};
const b = {__proto__: a};
const c = {__proto__: b};

assert.equal(a.isPrototypeOf(b), true);
assert.equal(a.isPrototypeOf(c), true);

assert.equal(c.isPrototypeOf(a), false);
assert.equal(a.isPrototypeOf(a), false);
```

有关此方法的更多信息，请参见§29.8.5 “`Object.prototype.isPrototypeOf()`”。

#### 28.9.4 `Object.hasOwn()`: 给定属性是否是自有的（非继承的）？[ES2022]

`in`运算符（A 行）检查对象是否具有给定属性。相反，`Object.hasOwn()`（B 行和 C 行）检查属性是否是自有的。

```js
const proto = {
 protoProp: 'protoProp',
};
const obj = {
 __proto__: proto,
 objProp: 'objProp',
}
assert.equal('protoProp' in obj, true); // (A)
assert.equal(Object.hasOwn(obj, 'protoProp'), false); // (B)
assert.equal(Object.hasOwn(proto, 'protoProp'), true); // (C)
```

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png) **ES2022 之前的替代方法：`.hasOwnProperty()`**

在 ES2022 之前，我们可以使用另一个特性：§29.8.8 “`Object.prototype.hasOwnProperty()`”。这个特性有陷阱，但引用的部分解释了如何解决它们。

#### 28.9.5 通过原型共享数据

考虑以下代码：

```js
const jane = {
 firstName: 'Jane',
 describe() {
 return 'Person named '+this.firstName;
 },
};
const tarzan = {
 firstName: 'Tarzan',
 describe() {
 return 'Person named '+this.firstName;
 },
};

assert.equal(jane.describe(), 'Person named Jane');
assert.equal(tarzan.describe(), 'Person named Tarzan');
```

我们有两个非常相似的对象。两者都有两个名为`.firstName`和`.describe`的属性。另外，方法`.describe()`是相同的。我们如何避免重复这个方法？

我们可以将它移到一个对象`PersonProto`中，并使该对象成为`jane`和`tarzan`的原型：

```js
const PersonProto = {
 describe() {
 return 'Person named ' + this.firstName;
 },
};
const jane = {
 __proto__: PersonProto,
 firstName: 'Jane',
};
const tarzan = {
 __proto__: PersonProto,
 firstName: 'Tarzan',
};
```

原型的名称反映了`jane`和`tarzan`都是人。

![图 11：对象 jane 和 tarzan 共享方法.describe()，通过它们的共同原型 PersonProto。](img/0cd9c5f607e5b90dabb4d4f8d5d26d93.png)

图 11：对象`jane`和`tarzan`共享方法`.describe()`，通过它们的共同原型`PersonProto`。

图 11 说明了这三个对象是如何连接的：底部的对象现在包含了特定于`jane`和`tarzan`的属性。顶部的对象包含了它们之间共享的属性。

当我们调用方法`jane.describe()`时，`this`指向该方法调用的接收者`jane`（在图表的左下角）。这就是为什么该方法仍然有效。`tarzan.describe()`也是类似的。

```js
assert.equal(jane.describe(), 'Person named Jane');
assert.equal(tarzan.describe(), 'Person named Tarzan');
```

展望下一章关于类的章节 - 这是类在内部组织的方式：

+   所有实例都共享一个带有方法的共同原型。

+   特定于实例的数据存储在每个实例的自有属性中。

§29.3 “类的内部”更详细地解释了这一点。

### 28.10 常见问题：对象

#### 28.10.1 为什么对象保留属性的插入顺序？

原则上，对象是无序的。排序属性的主要原因是列出条目、键或值的操作是确定性的。这有助于例如测试。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

参见测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/18)
