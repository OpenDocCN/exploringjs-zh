# 二十五、可调用值

> 原文：[`exploringjs.com/impatient-js/ch_callables.html`](https://exploringjs.com/impatient-js/ch_callables.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   25.1 函数的种类

+   25.2 普通函数

    +   25.2.1 命名函数表达式（高级）

    +   25.2.2 术语：函数定义和函数表达式

    +   25.2.3 函数声明的部分

    +   25.2.4 普通函数扮演的角色

    +   25.2.5 术语：实体 vs. 语法 vs. 角色（高级）

+   25.3 专门的函数

    +   25.3.1 专门的函数仍然是函数

    +   25.3.2 箭头函数

    +   25.3.3 方法、普通函数和箭头函数中的特殊变量`this`

    +   25.3.4 建议：优先使用专门的函数而不是普通函数

+   25.4 总结：可调用值的种类

+   25.5 从函数和方法返回值

+   25.6 参数处理

    +   25.6.1 术语：参数 vs. 参数值

    +   25.6.2 术语：回调

    +   25.6.3 参数过多或不足

    +   25.6.4 参数默认值

    +   25.6.5 剩余参数

    +   25.6.6 命名参数

    +   25.6.7 模拟命名参数

    +   25.6.8 展开（`...`）到函数调用中

+   25.7 函数的方法：`.call()`、`.apply()`、`.bind()`

    +   25.7.1 函数方法`.call()`

    +   25.7.2 函数方法`.apply()`

    +   25.7.3 函数方法`.bind()`

* * *

在本章中，我们将研究可以调用的 JavaScript 值：函数、方法和类。

### 25.1 函数的种类

JavaScript 有两种类别的函数：

+   *普通函数*可以扮演多种角色：

    +   真正的函数

    +   方法

    +   构造函数

+   *专门的函数*只能扮演这些角色中的一个- 例如：

    +   *箭头函数*只能是真正的函数。

    +   *方法*只能是方法。

    +   *类*只能是构造函数。

    ECMAScript 6 中添加了专门的函数。

继续阅读以了解所有这些东西的含义。

### 25.2 普通函数

以下代码展示了创建普通函数的两种方式（大致相同）：

```js
// Function declaration (a statement)
function ordinary1(a, b, c) {
 // ···
}

// const plus anonymous (nameless) function expression
const ordinary2 = function (a, b, c) {
 // ···
};
```

在作用域内，函数声明会提前激活（参见§11.8“声明：作用域和激活”），可以在声明之前调用。这有时很有用。

变量声明，比如`ordinary2`的声明，不会提前激活。

#### 25.2.1 命名函数表达式（高级）

到目前为止，我们只看到了匿名函数表达式-没有名称：

```js
const anonFuncExpr = function (a, b, c) {
 // ···
};
```

但也有*命名函数表达式*：

```js
const namedFuncExpr = function myName(a, b, c) {
 // `myName` is only accessible in here
};
```

`myName`只能在函数体内部访问。函数可以使用它来引用自身（用于自递归等）- 与其分配给哪个变量无关：

```js
const func = function funcExpr() { return funcExpr };
assert.equal(func(), func);

// The name `funcExpr` only exists inside the function body:
assert.throws(() => funcExpr(), ReferenceError);
```

即使它们没有分配给变量，命名函数表达式也有名称（第 A 行）：

```js
function getNameOfCallback(callback) {
 return callback.name;
}

assert.equal(
 getNameOfCallback(function () {}), ''); // anonymous

assert.equal(
 getNameOfCallback(function named() {}), 'named'); // (A)
```

请注意，通过函数声明或变量声明创建的函数始终具有名称：

```js
function funcDecl() {}
assert.equal(
 getNameOfCallback(funcDecl), 'funcDecl');

const funcExpr = function () {};
assert.equal(
 getNameOfCallback(funcExpr), 'funcExpr');
```

函数具有名称的一个好处是，这些名称会出现在 错误堆栈跟踪 中。

#### 25.2.2 术语：函数定义和函数表达式

*函数定义* 是创建函数的语法：

+   函数声明（一个语句）

+   函数表达式

函数声明总是产生普通函数。函数表达式产生普通函数或专门的函数：

+   普通函数表达式（我们已经遇到过的）：

    +   匿名函数表达式

    +   命名函数表达式

+   专门的函数表达式（稍后我们将看到）：

    +   箭头函数（它们总是表达式）

虽然在 JavaScript 中函数声明仍然很受欢迎，但在现代代码中，函数表达式几乎总是箭头函数。

#### 25.2.3 函数声明的各个部分

让我们通过以下示例来检查函数声明的各个部分。大多数术语也适用于函数表达式。

```js
function add(x, y) {
 return x + y;
}
```

+   `add` 是函数声明的 *名称*。

+   `add(x, y)` 是函数声明的 *头部*。

+   `x` 和 `y` 是 *参数*。

+   花括号（`{` 和 `}`）及其之间的所有内容是函数声明的 *主体*。

+   `return` 语句明确地从函数中返回一个值。

##### 25.2.3.1 参数列表中的尾随逗号

JavaScript 一直允许并忽略数组文字中的尾随逗号。自 ES5 以来，它们也允许在对象文字中。自 ES2017 以来，我们可以在参数列表（声明和调用）中添加尾随逗号：

```js
// Declaration
function retrieveData(
 contentText,
 keyword,
 {unique, ignoreCase, pageSize}, // trailing comma
) {
 // ···
}

// Invocation
retrieveData(
 '',
 null,
 {ignoreCase: true, pageSize: 10}, // trailing comma
);
```

#### 25.2.4 普通函数扮演的角色

考虑前一节中的以下函数声明：

```js
function add(x, y) {
 return x + y;
}
```

这个函数声明创建了一个名为 `add` 的普通函数。作为普通函数，`add()` 可以扮演三种角色：

+   真实函数：通过函数调用调用。

    ```js
    assert.equal(add(2, 1), 3);
    ```

+   方法：存储在属性中，通过方法调用调用。

    ```js
    const obj = { addAsMethod: add };
    assert.equal(obj.addAsMethod(2, 4), 6); // (A)
    ```

    在 A 行，`obj` 被称为方法调用的 *接收者*。

+   构造函数：通过 `new` 调用。

    ```js
    const inst = new add();
    assert.equal(inst instanceof add, true);
    ```

    顺便说一下，构造函数（包括类）的名称通常以大写字母开头。

#### 25.2.5 术语：实体 vs. 语法 vs. 角色（高级）

*语法*、*实体* 和 *角色* 的概念之间的区别是微妙的，通常并不重要。但我想让你对此有更敏锐的观察力：

+   *实体* 是 JavaScript 中的一个特性，因为它“存在”于 RAM 中。普通函数是一个实体。

    +   实体包括：普通函数、箭头函数、方法和类。

+   *语法* 是我们用来创建实体的代码。函数声明和匿名函数表达式是语法。它们都创建被称为普通函数的实体。

    +   语法包括：函数声明和匿名函数表达式。产生箭头函数的语法也称为 *箭头函数*。对于方法和类也是如此。

+   *角色* 描述了我们如何使用实体。实体 *普通函数* 可以扮演 *真实函数* 的角色，或者 *方法* 的角色，或者 *类* 的角色。实体 *箭头函数* 也可以扮演 *真实函数* 的角色。

    +   函数的角色是：真实函数、方法和构造函数。

许多其他编程语言只有一个实体扮演 *真实函数* 的角色。然后它们可以将 *函数* 这个名称用于角色和实体。

### 25.3 专门的函数

专门的函数是普通函数的单一版本。它们中的每一个都专门从事一个角色：

+   箭头函数的目的是成为真实函数：

    ```js
    const arrow = () => {
     return 123;
    };
    assert.equal(arrow(), 123);
    ```

+   *方法* 的目的是成为方法：

    ```js
    const obj = {
     myMethod() {
     return 'abc';
     }
    };
    assert.equal(obj.myMethod(), 'abc');
    ```

+   *类* 的目的是成为构造函数：

    ```js
    class MyClass {
     /* ··· */
    }
    const inst = new MyClass();
    ```

除了更好的语法之外，每种专门的函数还支持新功能，使它们在其工作中比普通函数更好。

+   箭头函数很快就会解释。

+   方法在单个对象章节中有解释。

+   类在类章节中有解释。

Tbl. 16 列出了普通和专门函数的功能。

表 16：四种函数的功能。如果单元格值在括号中，那意味着某种限制。特殊变量`this`在§25.3.3 “方法、普通函数和箭头函数中的特殊变量`this`”中有解释。

|  | 函数调用 | 方法调用 | 构造函数调用 |
| --- | --- | --- | --- |
| 普通函数 | (`this === undefined`) | `✔` | `✔` |
| 箭头函数 | `✔` | (词法`this`) | `✘` |
| 方法 | (`this === undefined`) | `✔` | `✘` |
| 类 | `✘` | `✘` | `✔` |

#### 25.3.1 专门函数仍然是函数

重要的是要注意，箭头函数、方法和类仍然被归类为函数：

```js
> (() => {}) instanceof Function
true
> ({ method() {} }.method) instanceof Function
true
> (class SomeClass {}) instanceof Function
true
```

#### 25.3.2 箭头函数

箭头函数被添加到 JavaScript 中有两个原因：

1.  为了提供一种更简洁的创建函数的方式。

1.  它们在方法内部作为真正的函数工作得更好：方法可以通过特殊变量`this`引用接收方法调用的对象。箭头函数可以访问周围方法的`this`，普通函数不能（因为它们有自己的`this`）。

我们首先将研究箭头函数的语法，然后再看各种函数中的`this`是如何工作的。

##### 25.3.2.1 箭头函数的语法

让我们回顾一下匿名函数表达式的语法：

```js
const f = function (x, y, z) { return 123 };
```

箭头函数的（大致）等效形式如下。箭头函数是表达式。

```js
const f = (x, y, z) => { return 123 };
```

这里，箭头函数的体是一个块。但它也可以是一个表达式。下面的箭头函数与前一个完全相同。

```js
const f = (x, y, z) => 123;
```

如果箭头函数只有一个参数，并且该参数是一个标识符（不是解构模式），那么你可以省略参数周围的括号：

```js
const id = x => x;
```

当将箭头函数作为参数传递给其他函数或方法时，这是很方便的：

```js
> [1,2,3].map(x => x+1)
[ 2, 3, 4 ]
```

前面的例子展示了箭头函数的一个好处 - 简洁性。如果我们用函数表达式执行相同的任务，我们的代码会更冗长：

```js
[1,2,3].map(function (x) { return x+1 });
```

##### 25.3.2.2 语法陷阱：从箭头函数返回对象字面量

如果你希望箭头函数的表达式体是一个对象字面量，你必须将字面量放在括号中：

```js
const func1 = () => ({a: 1});
assert.deepEqual(func1(), { a: 1 });
```

如果不这样做，JavaScript 会认为箭头函数有一个块体（不返回任何东西）：

```js
const func2 = () => {a: 1};
assert.deepEqual(func2(), undefined);
```

`{a: 1}`被解释为一个带有标签`a:`和表达式语句`1`的块。没有显式的`return`语句，块体返回`undefined`。

这个陷阱是由语法歧义引起的：对象字面量和代码块具有相同的语法。我们使用括号告诉 JavaScript，体是一个表达式（对象字面量），而不是一个语句（块）。

#### 25.3.3 方法、普通函数和箭头函数中的特殊变量`this`

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png)  **特殊变量`this`是面向对象的特性**

我们在这里快速看一下特殊变量`this`，以便理解为什么箭头函数比普通函数更好。

但这个特性只在面向对象编程中才重要，并且在§28.5 “方法和特殊变量`this`”中有更深入的介绍。因此，如果你还没有完全理解，不要担心。

在方法内部，特殊变量`this`让我们可以访问*接收者* - 接收方法调用的对象：

```js
const obj = {
 myMethod() {
 assert.equal(this, obj);
 }
};
obj.myMethod();
```

普通函数可以是方法，因此也有隐式参数`this`：

```js
const obj = {
 myMethod: function () {
 assert.equal(this, obj);
 }
};
obj.myMethod();
```

当我们将普通函数用作真正的函数时，`this`甚至是一个隐式参数。然后它的值是`undefined`（如果严格模式激活，几乎总是激活的）：

```js
function ordinaryFunc() {
 assert.equal(this, undefined);
}
ordinaryFunc();
```

这意味着普通函数作为真正的函数时，无法访问周围方法的`this`（A 行）。相反，箭头函数没有`this`作为隐式参数。它们将其视为任何其他变量，因此可以访问周围方法的`this`（B 行）：

```js
const jill = {
 name: 'Jill',
 someMethod() {
 function ordinaryFunc() {
 assert.throws(
 () => this.name, // (A)
 /^TypeError: Cannot read properties of undefined \(reading 'name'\)$/);
 }
 ordinaryFunc();

 const arrowFunc = () => {
 assert.equal(this.name, 'Jill'); // (B)
 };
 arrowFunc();
 },
};
jill.someMethod();
```

在这段代码中，我们可以观察到处理`this`的两种方式：

+   动态`this`：在 A 行，我们尝试从普通函数访问`.someMethod()`的`this`。在那里，它被函数自己的`this`*遮蔽*，这是`undefined`（由函数调用填充）。鉴于普通函数通过（动态）函数或方法调用接收它们的`this`，它们的`this`被称为*动态*。

+   词汇`this`：在 B 行，我们再次尝试访问`.someMethod()`的`this`。这次，我们成功了，因为箭头函数没有自己的`this`。`this`被*词法*解析，就像任何其他变量一样。这就是为什么箭头函数的`this`被称为*词法*。

#### 25.3.4 建议：优先选择专门的函数而不是普通函数

通常，您应该优先选择专门的函数而不是普通函数，特别是类和方法。

当涉及到真正的函数时，箭头函数和普通函数之间的选择并不那么明确：

+   对于匿名内联函数表达式，箭头函数是明显的赢家，因为它们的紧凑语法和它们不具有`this`作为隐式参数：

    ```js
    const twiceOrdinary = [1, 2, 3].map(function (x) {return x * 2});
    const twiceArrow = [1, 2, 3].map(x => x * 2);
    ```

+   对于独立的命名函数声明，箭头函数仍然受益于词法`this`。但是函数声明（产生普通函数）具有良好的语法，早期激活也偶尔有用（参见§11.8“声明：范围和激活”）。如果普通函数的主体中没有出现`this`，那么使用它作为真正的函数就没有任何缺点。在开发过程中，静态检查工具 ESLint 可以通过[内置规则](https://eslint.org/docs/rules/no-invalid-this)警告我们是否错误地这样做。

    ```js
    function timesOrdinary(x, y) {
     return x * y;
    }
    const timesArrow = (x, y) => {
     return x * y;
    };
    ```

### 25.4 总结：可调用值的种类

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png)  **本节涉及即将到来的内容**

本节主要作为当前和即将到来的章节的参考。如果您不理解一切，不要担心。

到目前为止，我们看到的所有（真正的）函数和方法都是：

+   单结果

+   同步

后续章节将涵盖其他编程模式：

+   *迭代*将对象视为数据容器（所谓的*可迭代*）并提供了一种标准化的方法来检索其中的内容。如果函数或方法返回可迭代对象，则会返回多个值。

+   *异步编程*处理长时间运行的计算。当计算完成时，会通知您，并且可以在其中间做其他事情。异步交付单个结果的标准模式称为*Promise*。

这些模式可以组合使用-例如，有同步可迭代和异步可迭代。

几种新的函数和方法有助于处理一些模式组合：

+   *异步函数*有助于实现返回 Promise 的函数。还有*异步方法*。

+   *同步生成器函数*有助于实现返回同步可迭代对象的函数。还有*同步生成器方法*。

+   *异步生成器函数*有助于实现返回异步可迭代对象的函数。还有*异步生成器方法*。

这使我们有 4 种（2×2）函数和方法：

+   同步与异步

+   生成器与单结果

Tbl. 17 提供了创建这 4 种函数和方法的语法概述。

表 17：创建函数和方法的语法。最后一列指定实体产生的值数量。

|  |  | **结果** | **#** |
| --- | --- | --- | --- |
| **同步函数** | **同步方法** |  |  |
| `function f() {}` | `{ m() {} }` | 值 | 1 |
| `f = function () {}` |  |  |  |
| `f = () => {}` |  |  |  |
| **同步生成器函数** | **同步生成器方法** |  |  |
| `function* f() {}` | `{ * m() {} }` | 可迭代 | 0+ |
| `f = function* () {}` |  |  |  |
| **异步函数** | **异步方法** |  |  |
| `async function f() {}` | `{ async m() {} }` | Promise | 1 |
| `f = async function () {}` |  |  |  |
| `f = async () => {}` |  |  |  |
| **异步生成器函数** | **异步生成器方法** |  |  |
| `async function* f() {}` | `{ async * m() {} }` | 异步可迭代 | 0+ |
| `f = async function* () {}` |  |  |  |

### 25.5 从函数和方法返回值

（本节提到的所有内容都适用于函数和方法。）

`return`语句明确地从函数中返回一个值：

```js
function func() {
 return 123;
}
assert.equal(func(), 123);
```

另一个例子：

```js
function boolToYesNo(bool) {
 if (bool) {
 return 'Yes';
 } else {
 return 'No';
 }
}
assert.equal(boolToYesNo(true), 'Yes');
assert.equal(boolToYesNo(false), 'No');
```

如果在函数末尾没有显式返回任何内容，JavaScript 会为你返回`undefined`：

```js
function noReturn() {
 // No explicit return
}
assert.equal(noReturn(), undefined);
```

### 25.6 参数处理

再次强调，本节中只提到了函数，但所有内容也适用于方法。

#### 25.6.1 术语：参数 vs. 参数

术语*参数*和*参数*基本上是指同一件事。如果你愿意，你可以做出以下区分：

+   *参数*是函数定义的一部分。它们也被称为*形式参数*和*形式参数*。

+   *参数*是函数调用的一部分。它们也被称为*实际参数*和*实际参数*。

#### 25.6.2 术语：回调

*回调*或*回调函数*是作为函数或方法调用的参数的函数。

以下是一个回调函数的例子：

```js
const myArray = ['a', 'b'];
const callback = (x) => console.log(x);
myArray.forEach(callback);

// Output:
// 'a'
// 'b'
```

#### 25.6.3 参数过多或不足

如果函数调用提供的参数数量与函数定义所期望的参数数量不同，JavaScript 不会报错：

+   额外的参数被忽略。

+   缺少的参数被设置为`undefined`。

例如：

```js
function foo(x, y) {
 return [x, y];
}

// Too many arguments:
assert.deepEqual(foo('a', 'b', 'c'), ['a', 'b']);

// The expected number of arguments:
assert.deepEqual(foo('a', 'b'), ['a', 'b']);

// Not enough arguments:
assert.deepEqual(foo('a'), ['a', undefined]);
```

#### 25.6.4 参数默认值

参数默认值指定如果未提供参数时要使用的值，例如：

```js
function f(x, y=0) {
 return [x, y];
}

assert.deepEqual(f(1), [1, 0]);
assert.deepEqual(f(), [undefined, 0]);
```

`undefined`也会触发默认值：

```js
assert.deepEqual(
 f(undefined, undefined),
 [undefined, 0]);
```

#### 25.6.5 Rest 参数

通过在标识符前加上三个点(`...`)来声明 rest 参数。在函数或方法调用期间，它接收一个包含所有剩余参数的数组。如果末尾没有额外的参数，它就是一个空数组，例如：

```js
function f(x, ...y) {
 return [x, y];
}
assert.deepEqual(
 f('a', 'b', 'c'), ['a', ['b', 'c']]
);
assert.deepEqual(
 f(), [undefined, []]
);
```

有两个与我们如何使用 rest 参数相关的限制：

+   我们不能在一个函数定义中使用多个 rest 参数。

    ```js
    assert.throws(
     () => eval('function f(...x, ...y) {}'),
     /^SyntaxError: Rest parameter must be last formal parameter$/
    );
    ```

+   rest 参数必须始终位于最后。因此，我们无法像这样访问最后一个参数：

    ```js
    assert.throws(
     () => eval('function f(...restParams, lastParam) {}'),
     /^SyntaxError: Rest parameter must be last formal parameter$/
    );
    ```

##### 25.6.5.1 通过 rest 参数强制使用一定数量的参数

你可以使用 rest 参数来强制使用一定数量的参数。例如，考虑以下函数：

```js
function createPoint(x, y) {
 return {x, y};
 // same as {x: x, y: y}
}
```

这是我们如何强制调用者始终提供两个参数的方法：

```js
function createPoint(...args) {
 if (args.length !== 2) {
 throw new Error('Please provide exactly 2 arguments!');
 }
 const [x, y] = args; // (A)
 return {x, y};
}
```

在 A 行，我们通过*解构*访问`args`的元素。

#### 25.6.6 命名参数

当有人调用一个函数时，调用者提供的参数被分配给被调用者接收的参数。执行映射的两种常见方式是：

1.  位置参数：如果参数位置相同，则将参数分配给参数。只有位置参数的函数调用如下所示。

    ```js
    selectEntries(3, 20, 2)
    ```

1.  命名参数：如果参数名称相同，则将参数分配给参数。JavaScript 没有命名参数，但你可以模拟它们。例如，这是一个只有（模拟）命名参数的函数调用：

    ```js
    selectEntries({start: 3, end: 20, step: 2})
    ```

命名参数有几个好处：

+   它们会导致更加自解释的代码，因为每个参数都有一个描述性标签。只需比较`selectEntries()`的两个版本：第二个版本更容易看出发生了什么。

+   参数的顺序不重要（只要名称正确）。

+   处理多个可选参数比较方便：调用者可以轻松地提供所有可选参数的任意子集，并且不必知道省略的参数（对于位置参数，您必须填写前面的可选参数，使用`undefined`）。

#### 25.6.7 模拟命名参数

JavaScript 没有真正的命名参数。模拟它们的官方方式是通过对象字面量：

```js
function selectEntries({start=0, end=-1, step=1}) {
 return {start, end, step};
}
```

这个函数使用*解构*来访问其单个参数的属性。它使用的模式是以下模式的缩写：

```js
{start: start=0, end: end=-1, step: step=1}
```

这种解构模式适用于空对象字面量：

```js
> selectEntries({})
{ start: 0, end: -1, step: 1 }
```

但如果你在没有任何参数的情况下调用函数，它就不起作用：

```js
> selectEntries()
TypeError: Cannot read properties of undefined (reading 'start')
```

您可以通过为整个模式提供默认值来解决这个问题。这个默认值的工作方式与更简单的参数定义的默认值相同：如果参数缺失，则使用默认值。

```js
function selectEntries({start=0, end=-1, step=1} = {}) {
 return {start, end, step};
}
assert.deepEqual(
 selectEntries(),
 { start: 0, end: -1, step: 1 });
```

#### 25.6.8 扩展(`...`)到函数调用

如果在函数调用的参数前面加上三个点(`...`)，那么就会*扩展*它。这意味着参数必须是可迭代对象，并且迭代的值都成为参数。换句话说，一个参数被扩展为多个参数 - 例如：

```js
function func(x, y) {
 console.log(x);
 console.log(y);
}
const someIterable = ['a', 'b'];
func(...someIterable);
 // same as func('a', 'b')

// Output:
// 'a'
// 'b'
```

扩展和剩余参数使用相同的语法(`...`)，但它们具有相反的目的：

+   剩余参数用于定义函数或方法时。它们将参数收集到数组中。

+   在调用函数或方法时使用扩展参数。它们将可迭代对象转换为参数。

##### 25.6.8.1 示例：扩展到`Math.max()`

`Math.max()`返回其零个或多个参数中的最大值。遗憾的是，它不能用于数组，但扩展给了我们一条出路：

```js
> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
> Math.max(-1, ...[-5, 11], 3)
11
```

##### 25.6.8.2 示例：扩展到`Array.prototype.push()`

同样，数组方法`.push()`会将其零个或多个参数破坏性地添加到数组的末尾。JavaScript 没有一种方法可以将一个数组破坏性地附加到另一个数组上。我们再次通过扩展来解决这个问题：

```js
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
assert.deepEqual(arr1, ['a', 'b', 'c', 'd']);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：参数处理**

+   位置参数：`exercises/callables/positional_parameters_test.mjs`

+   命名参数：`exercises/callables/named_parameters_test.mjs`

### 25.7 函数的方法：`.call()`，`.apply()`，`.bind()`

函数是对象，有方法。在本节中，我们将介绍其中三种方法：`.call()`，`.apply()`和`.bind()`。

#### 25.7.1 函数方法`.call()`

每个函数`someFunc`都有以下方法：

```js
someFunc.call(thisValue, arg1, arg2, arg3);
```

这种方法调用大致相当于以下函数调用：

```js
someFunc(arg1, arg2, arg3);
```

然而，使用`.call()`，我们也可以为隐式参数`this`指定一个值。换句话说：`.call()`使隐式参数`this`变为显式参数。

以下代码演示了`.call()`的使用：

```js
function func(x, y) {
 return [this, x, y];
}

assert.deepEqual(
 func.call('hello', 'a', 'b'),
 ['hello', 'a', 'b']);
```

正如我们之前所看到的，如果我们调用一个普通函数，它的`this`是`undefined`：

```js
assert.deepEqual(
 func('a', 'b'),
 [undefined, 'a', 'b']);
```

因此，前面的函数调用等价于：

```js
assert.deepEqual(
 func.call(undefined, 'a', 'b'),
 [undefined, 'a', 'b']);
```

在箭头函数中，通过`.call()`（或其他方式）提供的`this`值会被忽略。

#### 25.7.2 函数方法`.apply()`

每个函数`someFunc`都有以下方法：

```js
someFunc.apply(thisValue, [arg1, arg2, arg3]);
```

这种方法调用大致相当于以下函数调用（使用扩展）：

```js
someFunc(...[arg1, arg2, arg3]);
```

然而，使用`.apply()`，我们也可以为隐式参数`this`指定一个值。

以下代码演示了`.apply()`的使用：

```js
function func(x, y) {
 return [this, x, y];
}

const args = ['a', 'b'];
assert.deepEqual(
 func.apply('hello', args),
 ['hello', 'a', 'b']);
```

#### 25.7.3 函数方法`.bind()`

`.bind()`是函数对象的另一种方法。该方法的调用方式如下：

```js
const boundFunc = someFunc.bind(thisValue, arg1, arg2);
```

`.bind()`返回一个新的函数`boundFunc()`。调用该函数会将`this`设置为`thisValue`并传入这些参数：`arg1`，`arg2`，以及`boundFunc()`的参数。

也就是说，以下两个函数调用是等价的：

```js
boundFunc('a', 'b')
someFunc.call(thisValue, arg1, arg2, 'a', 'b')
```

##### 25.7.3.1 `.bind()`的替代方法

另一种预填充`this`和参数的方法是通过箭头函数：

```js
const boundFunc2 = (...args) =>
 someFunc.call(thisValue, arg1, arg2, ...args);
```

##### 25.7.3.2 `.bind()`的实现

考虑到前一节，`.bind()`可以实现为一个真实函数，如下所示：

```js
function bind(func, thisValue, ...boundArgs) {
 return (...args) =>
 func.call(thisValue, ...boundArgs, ...args);
}
```

##### 25.7.3.3 示例：绑定一个真实函数

对于真实函数使用`.bind()`有点不直观，因为我们必须为`this`提供一个值。鉴于在函数调用期间它是`undefined`，通常将其设置为`undefined`或`null`。

在下面的示例中，我们通过将`add()`的第一个参数绑定到`8`来创建一个只有一个参数的函数`add8()`。

```js
function add(x, y) {
 return x + y;
}

const add8 = add.bind(undefined, 8);
assert.equal(add8(1), 9);
```

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/17)
