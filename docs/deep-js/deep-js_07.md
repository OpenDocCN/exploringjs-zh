# 五、全局变量的详细了解

> 原文：[`exploringjs.com/deep-js/ch_global-scope.html`](https://exploringjs.com/deep-js/ch_global-scope.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   5.1 作用域

+   5.2 词法环境

+   5.3 全局对象

+   5.4 在浏览器中，`globalThis` 不直接指向全局对象

+   5.5 全局环境

    +   5.5.1 脚本作用域和模块作用域

    +   5.5.2 创建变量：声明记录 vs. 对象记录

    +   5.5.3 获取或设置变量

    +   5.5.4 全局 ECMAScript 变量和全局宿主变量

+   5.6 结论：为什么 JavaScript 同时具有普通全局变量和全局对象？

+   5.7 本章的进一步阅读和来源

* * *

在本章中，我们将详细了解 JavaScript 的全局变量是如何工作的。有几个有趣的现象起到了作用：脚本的作用域，所谓的*全局对象*等等。

### 5.1 作用域

变量的*词法作用域*（简称：*作用域*）是程序中可以访问它的区域。JavaScript 的作用域是*静态*的（在运行时不会改变），并且可以嵌套 - 例如：

```js
function func() { // (A)
 const aVariable = 1;
 if (true) { // (B)
 const anotherVariable = 2;
 }
}
```

`if` 语句引入的作用域（B 行）嵌套在函数 `func()` 的作用域（A 行）内。

作用域 S 的最内层包围作用域称为 S 的*外部作用域*。在例子中，`func` 是 `if` 的外部作用域。

### 5.2 词法环境

在 JavaScript 语言规范中，作用域通过*词法环境*“实现”。它们由两个组件组成：

+   将变量名映射到变量值的*环境记录*（类似于字典）。这是作用域变量的实际存储空间。记录中的名称-值条目称为*绑定*。

+   作用域的*外部环境*的引用 - 外部作用域的环境。

因此，嵌套作用域的树由外部环境引用链接的环境树表示。

### 5.3 全局对象

全局对象是一个对象，其属性成为全局变量。（我们很快将会详细讨论它如何适配环境树。）可以通过以下全局变量访问它：

+   在所有平台上都可用：[`globalThis`](https://exploringjs.com/impatient-js/ch_variables-assignment.html#globalThis)。其名称基于它在全局作用域中与 `this` 具有相同的值。

+   全局对象的其他变量在所有平台上都不可用：

    +   `window` 是指向全局对象的经典方式。它在正常的浏览器代码中有效，但在*Web Workers*（与正常浏览器进程并发运行的进程）和 Node.js 中无效。

    +   `self` 在浏览器中随处可用（包括在 Web Workers 中）。但它不受 Node.js 支持。

    +   `global` 仅在 Node.js 上可用。

### 5.4 在浏览器中，`globalThis` 不直接指向全局对象

在浏览器中，`globalThis` 不直接指向全局对象，而是有一个间接引用。例如，考虑网页上的 iframe：

+   每当 iframe 的 `src` 改变时，它都会获得一个新的全局对象。

+   然而，`globalThis` 总是具有相同的值。可以从 iframe 外部检查该值，如下所示（灵感来自[`globalThis`提案中的一个例子](https://github.com/tc39/proposal-global#html-and-the-windowproxy)）。

文件 `parent.html`：

```js
<iframe src="iframe.html?first"></iframe>
<script>
 const iframe = document.querySelector('iframe');
 const icw = iframe.contentWindow; // `globalThis` of iframe

 iframe.onload = () => {
 // Access properties of global object of iframe
 const firstGlobalThis = icw.globalThis;
 const firstArray = icw.Array;
 console.log(icw.iframeName); // 'first'

 iframe.onload = () => {
 const secondGlobalThis = icw.globalThis;
 const secondArray = icw.Array;

 // The global object is different
 console.log(icw.iframeName); // 'second'
 console.log(secondArray === firstArray); // false

 // But globalThis is still the same
 console.log(firstGlobalThis === secondGlobalThis); // true
 };
 iframe.src = 'iframe.html?second';
 };
</script>
```

文件 `iframe.html`：

```js
<script>
 globalThis.iframeName = location.search.slice(1);
</script>
```

浏览器如何确保在这种情况下`globalThis`不会改变？它们内部区分了两个对象：

+   [`Window`](https://html.spec.whatwg.org/multipage/window-object.html#the-window-object)是全局对象。它会在位置改变时改变。

+   [`WindowProxy`](https://html.spec.whatwg.org/multipage/window-object.html#the-windowproxy-exotic-object)是一个对象，它将所有访问都转发到当前的`Window`。这个对象永远不会改变。

在浏览器中，`globalThis` 指的是 `WindowProxy`；在其他地方，它直接指向全局对象。

### 5.5 全局环境

全局范围是“最外层”的范围 - 它没有外部范围。它的环境是*全局环境*。每个环境都通过一系列由外部环境引用链接的环境链与全局环境连接。全局环境的外部环境引用是`null`。

全局环境记录使用两个环境记录来管理其变量：

+   *对象环境记录*具有与普通环境记录相同的接口，但它将其绑定存储在 JavaScript 对象中。在这种情况下，对象就是全局对象。

+   一个普通（*声明性*）环境记录，它有自己的绑定存储。

哪个环境记录将在何时使用将很快解释。

#### 5.5.1 脚本范围和模块范围

在 JavaScript 中，我们只在脚本的顶层处于全局范围。相反，每个模块都有自己的作用域，它是脚本范围的子作用域。

如果我们忽略了变量绑定如何添加到全局环境的相对复杂的规则，那么全局范围和模块范围的工作就好像它们是嵌套的代码块一样：

```js
{ // Global scope (scope of *all* scripts)

 // (Global variables)

 { // Scope of module 1
 ···
 }
 { // Scope of module 2
 ···
 }
 // (More module scopes)
}
```

#### 5.5.2 创建变量：声明性记录 vs. 对象记录

为了创建一个真正全局的变量，我们必须处于全局范围内 - 这只有在脚本的顶层才是这样：

+   顶层的`const`、`let`和`class`在声明性环境记录中创建绑定。

+   顶层的`var`和函数声明在对象环境记录中创建绑定。

```js
<script>
 const one = 1;
 var two = 2;
</script>
<script>
 // All scripts share the same top-level scope:
 console.log(one); // 1
 console.log(two); // 2

 // Not all declarations create properties of the global object:
 console.log(globalThis.one); // undefined
 console.log(globalThis.two); // 2
</script>
```

#### 5.5.3 获取或设置变量

当我们获取或设置一个变量，并且两个环境记录都有该变量的绑定时，那么声明性记录会胜出：

```js
<script>
 let myGlobalVariable = 1; // declarative environment record
 globalThis.myGlobalVariable = 2; // object environment record

 console.log(myGlobalVariable); // 1 (declarative record wins)
 console.log(globalThis.myGlobalVariable); // 2
</script>
```

#### 5.5.4 全局 ECMAScript 变量和全局宿主变量

除了通过`var`和函数声明创建的变量之外，全局对象还包含以下属性：

+   所有 ECMAScript 的内置全局变量

+   宿主平台（浏览器、Node.js 等）的所有内置全局变量

使用 `const` 或 `let` 可以保证全局变量声明不会影响（或受到影响）ECMAScript 和宿主平台的内置全局变量。

例如，浏览器有[全局变量`.location`](https://developer.mozilla.org/en-US/docs/Web/API/Window/location)：

```js
// Changes the location of the current document:
var location = 'https://example.com';

// Shadows window.location, doesn’t change it:
let location = 'https://example.com';
```

如果变量已经存在（比如在这种情况下的 `location`），那么带有初始化器的 `var` 声明就像是一个赋值。这就是为什么我们在这个例子中遇到麻烦的原因。

请注意，这只是在全局范围中的一个问题。在模块中，我们永远不会处于全局范围（除非我们使用`eval()`或类似的东西）。

图 10 总结了本节中我们所学到的一切。

![](img/55ca076d7f5298721a42eabbf8a45ec7.png)

图 10：全局范围的环境通过一个*全局环境记录*来管理其绑定，该记录又基于两个环境记录：一个*对象环境记录*，其绑定存储在全局对象中，以及一个*声明性环境记录*，它使用内部存储来存储其绑定。因此，全局变量可以通过向全局对象添加属性或通过各种声明来创建。全局对象初始化为 ECMAScript 和宿主平台的内置全局变量。每个 ECMAScript 模块都有自己的环境，其外部环境是全局环境。

### 5.6 结论：为什么 JavaScript 既有普通的全局变量又有全局对象？

全局对象通常被认为是一个错误。因此，新的构造，如`const`、`let`和类，在脚本范围内创建普通的全局变量。

值得庆幸的是，现代 JavaScript 中编写的大多数代码都存在于[ECMAScript 模块和 CommonJS 模块](https://exploringjs.com/impatient-js/ch_modules.html)中。每个模块都有自己的作用域，这就是为什么控制全局变量的规则很少适用于基于模块的代码。

### 5.7 进一步阅读和本章的来源

ECMAScript 规范中的环境和全局对象：

+   [“词法环境”部分](https://tc39.es/ecma262/#sec-lexical-environments)提供了环境的概述。

+   [“全局环境记录”部分](https://tc39.es/ecma262/#sec-global-environment-records)涵盖了全局环境。

+   [“ECMAScript 标准内置对象”部分](https://tc39.es/ecma262/#sec-ecmascript-standard-built-in-objects)描述了 ECMAScript 如何管理其内置对象（其中包括全局对象）。

`globalThis`：

+   2ality 发布了“ES 功能：`globalThis`”的帖子

+   各种访问全局`this`值的方式：Mathias Bynens 的[“一个可怕的`globalThis`多填充在通用 JavaScript 中”](https://mathiasbynens.be/notes/globalthis)

浏览器中的全局对象：

+   关于浏览器中发生的事情的背景：Anne van Kesteren 的[“定义 WindowProxy、Window 和 Location 对象”](https://blog.whatwg.org/windowproxy-window-and-location)

+   非常技术性：WHATWG HTML 标准中的[“领域、设置对象和全局对象”部分](https://html.spec.whatwg.org/multipage/webappapis.html#realms-settings-objects-global-objects)

+   在 ECMAScript 规范中，我们可以看到 Web 浏览器如何自定义全局`this`：[“InitializeHostDefinedRealm()”部分](https://tc39.es/ecma262/#sec-initializehostdefinedrealm)

[评论](https://github.com/rauschma/deep-js/issues/4)
