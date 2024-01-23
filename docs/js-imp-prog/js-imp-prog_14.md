# 十一、变量和赋值

> 原文：[`exploringjs.com/impatient-js/ch_variables-assignment.html`](https://exploringjs.com/impatient-js/ch_variables-assignment.html)

* * *

+   11.1 `let`

+   11.2 `const`

    +   11.2.1 `const`和不可变性

    +   11.2.2 `const`和循环

+   11.3 在`const`和`let`之间做出决定

+   11.4 变量的作用域

    +   11.4.1 变量屏蔽

+   11.5 (高级)

+   11.6 术语：静态 vs. 动态

    +   11.6.1 静态现象：变量的作用域

    +   11.6.2 动态现象：函数调用

+   11.7 全局变量和全局对象

    +   11.7.1 [`globalThis` [ES2020]](ch_variables-assignment.html#globalThis)

+   11.8 声明：作用域和激活

    +   11.8.1 `const`和`let`：时间死区

    +   11.8.2 函数声明和早期激活

    +   11.8.3 类声明不会早期激活

    +   11.8.4 `var`：变量提升（部分早期激活）

+   11.9 闭包

    +   11.9.1 绑定变量 vs. 自由变量

    +   11.9.2 什么是闭包？

    +   11.9.3 示例：增量器的工厂

    +   11.9.4 闭包的用例

* * *

这些是 JavaScript 声明变量的主要方式：

+   `let`声明可变变量。

+   `const`声明*常量*（不可变变量）。

在 ES6 之前，还有`var`。但它有一些怪癖，所以最好在现代 JavaScript 中避免使用它。您可以在[*Speaking JavaScript*](http://speakingjs.com/es5/ch16.html)中了解更多信息。

### 11.1 `let`

通过`let`声明的变量是可变的。

```js
let i;
i = 0;
i = i + 1;
assert.equal(i, 1);
```

您也可以同时声明和赋值：

```js
let i = 0;
```

### 11.2 `const`

通过`const`声明的变量是不可变的。您必须立即初始化：

```js
const i = 0; // must initialize

assert.throws(
 () => { i = i + 1 },
 {
 name: 'TypeError',
 message: 'Assignment to constant variable.',
 }
);
```

#### 11.2.1 `const`和不可变性

在 JavaScript 中，`const`只表示*绑定*（变量名和变量值之间的关联）是不可变的。值本身可能是可变的，就像下面的示例中的`obj`一样。

```js
const obj = { prop: 0 };

// Allowed: changing properties of `obj`
obj.prop = obj.prop + 1;
assert.equal(obj.prop, 1);

// Not allowed: assigning to `obj`
assert.throws(
 () => { obj = {} },
 {
 name: 'TypeError',
 message: 'Assignment to constant variable.',
 }
);
```

#### 11.2.2 `const`和循环

您可以在`for-of`循环中使用`const`，其中为每次迭代创建一个新的绑定：

```js
const arr = ['hello', 'world'];
for (const elem of arr) {
 console.log(elem);
}
// Output:
// 'hello'
// 'world'
```

在普通的`for`循环中，您必须使用`let`：

```js
const arr = ['hello', 'world'];
for (let i=0; i<arr.length; i++) {
 const elem = arr[i];
 console.log(elem);
}
```

### 11.3 在`const`和`let`之间做出决定

我建议以下规则来决定使用`const`还是`let`：

+   `const`表示不可变的绑定，变量永远不会改变其值。最好使用它。

+   `let`表示变量的值会改变。只有在无法使用`const`时才使用它。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：`const`**

`exercises/variables-assignment/const_exrc.mjs`

### 11.4 变量的作用域

变量的*作用域*是程序中可以访问它的区域。考虑以下代码。

```js
{ // // Scope A. Accessible: x
 const x = 0;
 assert.equal(x, 0);
 { // Scope B. Accessible: x, y
 const y = 1;
 assert.equal(x, 0);
 assert.equal(y, 1);
 { // Scope C. Accessible: x, y, z
 const z = 2;
 assert.equal(x, 0);
 assert.equal(y, 1);
 assert.equal(z, 2);
 }
 }
}
// Outside. Not accessible: x, y, z
assert.throws(
 () => console.log(x),
 {
 name: 'ReferenceError',
 message: 'x is not defined',
 }
);
```

+   作用域 A 是`x`的*(直接)作用域*。

+   作用域 B 和 C 是作用域 A 的*内部作用域*。

+   作用域 A 是作用域 B 和作用域 C 的*外部作用域*。

每个变量在其直接作用域以及所有嵌套在该作用域内的作用域中都是可访问的。

通过`const`和`let`声明的变量被称为*块作用域*，因为它们的作用域始终是最内部的周围块。

#### 11.4.1 遮蔽变量

你不能在同一级别两次声明相同的变量：

```js
assert.throws(
 () => {
 eval('let x = 1; let x = 2;');
 },
 {
 name: 'SyntaxError',
 message: "Identifier 'x' has already been declared",
 });
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **为什么使用`eval()`?**

`eval()` 延迟解析（因此延迟了`SyntaxError`），直到执行`assert.throws()`的回调。如果我们不使用它，当这段代码被解析时，我们已经会得到一个错误，`assert.throws()`甚至不会被执行。

但是，你可以嵌套一个块并使用与块外部相同的变量名`x`：

```js
const x = 1;
assert.equal(x, 1);
{
 const x = 2;
 assert.equal(x, 2);
}
assert.equal(x, 1);
```

在块内部，内部的 `x` 是唯一可访问的具有该名称的变量。内部的 `x` 被称为*遮蔽*外部的 `x`。一旦离开块，你可以再次访问旧值。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验：基础**

参见测验应用。

### 11.5（高级）

其余部分都是高级的。

### 11.6 术语：静态 vs. 动态

这两个形容词描述了编程语言中的现象：

+   *静态*意味着某物与源代码相关，并且可以在不执行代码的情况下确定。

+   *动态*意味着在运行时。

让我们来看看这两个术语的例子。

#### 11.6.1 静态现象：变量的作用域

变量作用域是一个静态现象。考虑以下代码：

```js
function f() {
 const x = 3;
 // ···
}
```

`x`是*静态*（或*词法*）*作用域*。也就是说，它的作用域是固定的，在运行时不会改变。

变量作用域形成静态树（通过静态嵌套）。

#### 11.6.2 动态现象：函数调用

函数调用是一个动态现象。考虑以下代码：

```js
function g(x) {}
function h(y) {
 if (Math.random()) g(y); // (A)
}
```

在 A 行的函数调用是否发生，只能在运行时决定。

函数调用形成一个动态树（通过动态调用）。

### 11.7 全局变量和全局对象

JavaScript 的变量作用域是嵌套的。它们形成一个树：

+   最外层的作用域是树的根。

+   直接包含在该作用域中的作用域是根的子级。

+   等等。

根也被称为*全局作用域*。在 Web 浏览器中，直接处于该作用域的唯一位置是在脚本的顶层。全局作用域的变量称为*全局变量*，并且可以在任何地方访问。有两种全局变量：

+   *全局声明变量*是普通变量。

    +   它们只能在脚本的顶层通过`const`、`let`和类声明创建。

+   *全局对象变量*存储在所谓的*全局对象*的属性中。

    +   它们是在脚本的顶层通过`var`和函数声明创建的。

    +   全局对象可以通过全局变量`globalThis`访问。它可以用于创建、读取和删除全局对象变量。

    +   除此之外，全局对象变量的工作方式与普通变量相同。

以下 HTML 片段演示了`globalThis`和两种全局变量。

```js
<script>
 const declarativeVariable = 'd';
 var objectVariable = 'o';
</script>
<script>
 // All scripts share the same top-level scope:
 console.log(declarativeVariable); // 'd'
 console.log(objectVariable); // 'o'

 // Not all declarations create properties of the global object:
 console.log(globalThis.declarativeVariable); // undefined
 console.log(globalThis.objectVariable); // 'o'
</script>
```

每个 ECMAScript 模块都有自己的作用域。因此，在模块的顶层存在的变量并不是全局的。图 5 说明了各种作用域之间的关系。

![图 5：全局作用域是 JavaScript 的最外层作用域。它有两种变量：对象变量（通过全局对象管理）和普通的声明变量。每个 ECMAScript 模块都有自己的作用域，它包含在全局作用域中。](img/50731076c488c6bf43423b1518edad6b.png)

图 5：全局作用域是 JavaScript 的最外层作用域。它有两种变量：*对象变量*（通过*全局对象*管理）和普通的*声明变量*。每个 ECMAScript 模块都有自己的作用域，它包含在全局作用域中。

#### 11.7.1 `globalThis` [ES2020]

全局变量`globalThis`是访问全局对象的新标准方式。它的名称来自于它在全局作用域中与`this`具有相同的值。

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **`globalThis`并不总是直接指向全局对象**

例如，在浏览器中，[存在一个间接访问](https://exploringjs.com/deep-js/ch_global-scope.html#window-proxy)。这种间接通常是不明显的，但它存在并且可以被观察到。

##### 11.7.1.1 `globalThis`的替代方案

访问全局对象的旧方法取决于平台：

+   全局变量`window`：是指全局对象的经典方式。但在 Node.js 和 Web Workers 中不起作用。

+   全局变量`self`：在 Web Workers 和通常的浏览器中可用。但 Node.js 不支持。

+   全局变量`global`：仅在 Node.js 中可用。

##### 11.7.1.2 `globalThis`的用例

全局对象现在被认为是 JavaScript 无法摆脱的错误，因为向后兼容性。它对性能产生负面影响，通常令人困惑。

ECMAScript 6 引入了一些功能，使得更容易避免全局对象-例如：

+   `const`、`let`和类声明在全局作用域中使用时不会创建全局对象属性。

+   每个 ECMAScript 模块都有自己的局部作用域。

最好通过变量访问全局对象变量，而不是通过`globalThis`的属性。前者在所有 JavaScript 平台上始终以相同的方式工作。

网络教程偶尔通过`window.globVar`访问全局变量`globVar`。但前缀“`window.`”是不必要的，我建议省略它：

```js
window.encodeURIComponent(str); // no
encodeURIComponent(str); // yes
```

因此，`globalThis`的用例相对较少-例如：

+   *Polyfills*为旧的 JavaScript 引擎添加新功能。

+   特性检测，以找出 JavaScript 引擎支持的功能。

### 11.8 声明：作用域和激活

这些是声明的两个关键方面：

+   作用域：声明的实体可以在哪里看到？这是一个静态特征。

+   激活：何时可以访问实体？这是一个动态特征。一些实体可以在我们进入它们的作用域时立即访问。对于其他实体，我们必须等到执行到达它们的声明时才能访问。

Tbl. 1 总结了各种声明如何处理这些方面。

表 1：声明的方面。“重复”描述了声明是否可以在相同的名称下两次使用（每个作用域）。“全局属性”描述了当在脚本的全局作用域中执行时，声明是否向全局对象添加属性。*TDZ*表示*暂时死区*（稍后会解释）。(*) 函数声明通常是块作用域的，但在松散模式中是函数作用域的。

|  | 作用域 | 激活 | 重复 | 全局属性 |
| --- | --- | --- | --- | --- |
| `const` | 块 | 声明（TDZ） | `✘` | `✘` |
| `let` | 块 | 声明（TDZ） | `✘` | `✘` |
| `function` | 块 (*) | 开始 | `✔` | `✔` |
| `class` | 块 | 声明（TDZ） | `✘` | `✘` |
| `import` | 模块 | 与导出相同 | `✘` | `✘` |
| `var` | 函数 | 开始，部分 | `✔` | `✔` |

`import`在§27.5“ECMAScript 模块”中有描述。以下各节更详细地描述了其他构造。

#### 11.8.1 `const`和`let`：暂时死区

对于 JavaScript，TC39 需要决定在直接作用域中访问常量时会发生什么：

```js
{
 console.log(x); // What happens here?
 const x;
}
```

一些可能的方法是：

1.  名称在当前作用域周围的作用域中解析。

1.  你会得到`undefined`。

1.  有一个错误。

第一种方法被拒绝了，因为语言中没有这种方法的先例。因此，对 JavaScript 程序员来说不直观。

第二种方法被拒绝了，因为`x`在其声明之前和之后将不是一个常量-它将具有不同的值。

`let` 使用与 `const` 相同的方法 3，因此两者的工作方式类似，很容易在它们之间切换。

进入变量作用域并执行其声明之间的时间称为*时间死区*（TDZ）：

+   在此期间，该变量被视为未初始化（就好像那是它的特殊值）。

+   如果访问未初始化的变量，将会得到 `ReferenceError`。

+   一旦到达变量声明，变量就会被设置为初始化程序的值（通过赋值符号指定），或者 `undefined` - 如果没有初始化程序。

以下代码说明了时间死区：

```js
if (true) { // entering scope of `tmp`, TDZ starts
 // `tmp` is uninitialized:
 assert.throws(() => (tmp = 'abc'), ReferenceError);
 assert.throws(() => console.log(tmp), ReferenceError);

 let tmp; // TDZ ends
 assert.equal(tmp, undefined);
}
```

下一个示例显示了时间死区确实是*时间*相关的：

```js
if (true) { // entering scope of `myVar`, TDZ starts
 const func = () => {
 console.log(myVar); // executed later
 };

 // We are within the TDZ:
 // Accessing `myVar` causes `ReferenceError`

 let myVar = 3; // TDZ ends
 func(); // OK, called outside TDZ
}
```

即使 `func()` 位于 `myVar` 的声明之前并使用该变量，我们也可以调用 `func()`。但是我们必须等到 `myVar` 的时间死区结束。

#### 11.8.2 函数声明和提前激活

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png) **有关函数的更多信息**

在本节中，我们正在使用函数 - 在我们有机会正确学习它们之前。希望一切仍然有意义。每当不明白时，请参阅§25“可调用值”。

函数声明总是在进入其作用域时执行，无论它在作用域内的位置如何。这使您可以在声明之前调用函数 `foo()`：

```js
assert.equal(foo(), 123); // OK
function foo() { return 123; }
```

`foo()` 的提前激活意味着前面的代码等同于：

```js
function foo() { return 123; }
assert.equal(foo(), 123);
```

如果您通过 `const` 或 `let` 声明函数，则不会提前激活。在以下示例中，只能在声明后使用 `bar()`。

```js
assert.throws(
 () => bar(), // before declaration
 ReferenceError);

const bar = () => { return 123; };

assert.equal(bar(), 123); // after declaration 
```

##### 11.8.2.1 提前调用而不提前激活

即使函数 `g()` 没有提前激活，也可以通过前面的函数 `f()`（在相同的作用域中）调用它，如果我们遵守以下规则：`f()` 必须在 `g()` 的声明之后调用。

```js
const f = () => g();
const g = () => 123;

// We call f() after g() was declared:
assert.equal(f(), 123);
```

模块的函数通常在其完整体执行后调用。因此，在模块中，您很少需要担心函数的顺序。

最后，请注意，提前激活会自动遵守上述规则：进入作用域时，所有函数声明都会首先执行，然后再进行任何调用。

##### 11.8.2.2 提前激活的一个陷阱

如果您依赖提前激活来调用函数而不是声明它，那么您需要小心，不要访问未提前激活的数据。

```js
funcDecl();

const MY_STR = 'abc';
function funcDecl() {
 assert.throws(
 () => MY_STR,
 ReferenceError);
}
```

如果您在声明 `MY_STR` 之后调用 `funcDecl()`，问题就会消失。

##### 11.8.2.3 提前激活的利弊

我们已经看到提前激活有一个陷阱，并且您可以在不使用它的情况下获得大部分好处。因此，最好避免提前激活。但我对此并不感到强烈，并且如前所述，通常使用函数声明，因为我喜欢它们的语法。

#### 11.8.3 类声明不会提前激活

尽管它们在某些方面类似于函数声明，类声明 不会提前激活：

```js
assert.throws(
 () => new MyClass(),
 ReferenceError);

class MyClass {}

assert.equal(new MyClass() instanceof MyClass, true);
```

为什么？考虑以下类声明：

```js
class MyClass extends Object {}
```

`extends` 的操作数是一个表达式。因此，您可以做这样的事情：

```js
const identity = x => x;
class MyClass extends identity(Object) {}
```

对这样的表达式进行评估必须在提到它的位置进行。其他任何操作都会令人困惑。这就解释了为什么类声明不会提前激活。

#### 11.8.4 `var`：提升（部分提前激活）

`var` 是一种旧的声明变量的方式，早于 `const` 和 `let`（现在更受青睐）。考虑以下 `var` 声明。

```js
var x = 123;
```

此声明有两个部分：

+   声明 `var x`：`var` 声明的变量的作用域是最内层的周围函数，而不是最内层的周围块，就像大多数其他声明一样。这样的变量在其作用域开始时已经激活并初始化为 `undefined`。

+   赋值 `x = 123`：赋值总是在原地执行。

以下代码演示了`var`的效果：

```js
function f() {
 // Partial early activation:
 assert.equal(x, undefined);
 if (true) {
 var x = 123;
 // The assignment is executed in place:
 assert.equal(x, 123);
 }
 // Scope is function, not block:
 assert.equal(x, 123);
}
```

### 11.9 闭包

在我们探讨闭包之前，我们需要了解绑定变量和自由变量。

#### 11.9.1 绑定变量 vs. 自由变量

每个作用域中都有一组被提及的变量。在这些变量中，我们区分： 

+   *绑定变量*是在作用域内声明的。它们是参数和局部变量。

+   *自由变量*是在外部声明的。它们也被称为*非局部变量*。

考虑以下代码：

```js
function func(x) {
 const y = 123;
 console.log(z);
}
```

在`func()`的主体中，`x`和`y`是绑定变量。`z`是自由变量。

#### 11.9.2 闭包是什么？

那么闭包是什么呢？

> *闭包*是一个函数加上与其“诞生地”存在的变量的连接。

保持这种连接的意义是什么？它为函数的自由变量提供了值-例如：

```js
function funcFactory(value) {
 return () => {
 return value;
 };
}

const func = funcFactory('abc');
assert.equal(func(), 'abc'); // (A)
```

`funcFactory`返回一个分配给`func`的闭包。因为`func`与其诞生地的变量有连接，所以当它在 A 行被调用时，它仍然可以访问自由变量`value`（尽管它“逃离”了它的作用域）。

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png)  **JavaScript 中的所有函数都是闭包**

JavaScript 中支持静态作用域。因此，每个函数都是一个闭包。

#### 11.9.3 示例：增量器工厂

以下函数返回*增量器*（我刚刚编造的一个名字）。增量器是一个内部存储数字的函数。当它被调用时，它通过将参数添加到数字中来更新该数字，并返回新值。

```js
function createInc(startValue) {
 return (step) => { // (A)
 startValue += step;
 return startValue;
 };
}
const inc = createInc(5);
assert.equal(inc(2), 7);
```

我们可以看到，在 A 行创建的函数保留了其内部数字在自由变量`startValue`中。这一次，我们不仅仅是从诞生作用域中读取，我们使用它来存储我们改变的数据，并且这些数据在函数调用之间保持不变。

我们可以通过局部变量在诞生作用域中创建更多的存储槽：

```js
function createInc(startValue) {
 let index = -1;
 return (step) => {
 startValue += step;
 index++;
 return [index, startValue];
 };
}
const inc = createInc(5);
assert.deepEqual(inc(2), [0, 7]);
assert.deepEqual(inc(2), [1, 9]);
assert.deepEqual(inc(2), [2, 11]);
```

#### 11.9.4 闭包的用例

闭包有什么好处？

+   首先，它们只是静态作用域的一种实现。因此，它们为回调提供上下文数据。

+   它们也可以被函数用来存储在函数调用之间持续存在的状态。`createInc()`就是一个例子。

+   它们可以为对象提供私有数据（通过字面量或类生成）。关于这是如何工作的细节在[*探索 ES6*](https://exploringjs.com/es6/ch_classes.html#_private-data-via-constructor-environments)中有解释。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验：高级**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/6)
