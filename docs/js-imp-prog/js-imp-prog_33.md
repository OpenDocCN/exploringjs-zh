# 二十七、模块

> 原文：[`exploringjs.com/impatient-js/ch_modules.html`](https://exploringjs.com/impatient-js/ch_modules.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)



* * *

+   27.1 速查表：模块

    +   27.1.1 导出

    +   27.1.2 导入

+   27.2 JavaScript 源代码格式

    +   27.2.1 内置模块之前的代码是用 ECMAScript 5 编写的

+   27.3 在我们有模块之前，我们有脚本

+   27.4 ES6 之前创建的模块系统

    +   27.4.1 服务器端：CommonJS 模块

    +   27.4.2 客户端：AMD（异步模块定义）模块

    +   27.4.3 JavaScript 模块的特点

+   27.5 ECMAScript 模块

    +   27.5.1 ES 模块：语法、语义、加载器 API

+   27.6 命名导出和导入

    +   27.6.1 命名导出

    +   27.6.2 命名导入

    +   27.6.3 命名空间导入

    +   27.6.4 命名导出风格：内联与子句（高级）

+   27.7 默认导出和导入

    +   27.7.1 默认导出的两种风格

    +   27.7.2 默认导出作为命名导出（高级）

+   27.8 导出和导入的更多细节

    +   27.8.1 导入是对导出的只读视图

    +   27.8.2 ESM 对循环导入的透明支持（高级）

+   27.9 npm 包

    +   27.9.1 包安装在`node_modules/`目录中

    +   27.9.2 npm 为什么可以用于安装前端库？

+   27.10 模块命名

+   27.11 模块规范

    +   27.11.1 模块规范的类别

    +   27.11.2 浏览器中的 ES 模块规范

    +   27.11.3 Node.js 上的 ES 模块规范

+   27.12 [`import.meta` – 当前模块的元数据[ES2020]](ch_modules.html#import.meta)

    +   27.12.1 `import.meta.url`

    +   27.12.2 `import.meta.url`和`URL`类

    +   27.12.3 Node.js 上的`import.meta.url`

+   27.13 [通过`import()` [ES2020]（高级）动态加载模块](ch_modules.html#dynamic-imports)

    +   27.13.1 静态`import`语句的限制

    +   27.13.2 通过`import()`运算符动态导入

    +   27.13.3 `import()`的用例

+   27.14 [模块中的顶层`await` [ES2022]（高级）](ch_modules.html#top-level-await)

    +   27.14.1 顶层`await`的用例

    +   27.14.2 顶层`await`在内部是如何工作的？

    +   27.14.3 顶层`await`的利弊

+   27.15 Polyfills:模拟原生 Web 平台功能（高级）

    +   27.15.1 本节的来源

* * *

### 27.1 速查表：模块

#### 27.1.1 导出

```js
// Named exports
export function f() {}
export const one = 1;
export {foo, b as bar};

// Default exports
export default function f() {} // declaration with optional name
// Replacement for `const` (there must be exactly one value)
export default 123;

// Re-exporting from another module
export {foo, b as bar} from './some-module.mjs';
export * from './some-module.mjs';
export * as ns from './some-module.mjs'; // ES2020
```

#### 27.1.2 导入

```js
// Named imports
import {foo, bar as b} from './some-module.mjs';
// Namespace import
import * as someModule from './some-module.mjs';
// Default import
import someModule from './some-module.mjs';

// Combinations:
import someModule, * as someModule from './some-module.mjs';
import someModule, {foo, bar as b} from './some-module.mjs';

// Empty import (for modules with side effects)
import './some-module.mjs';
```

### 27.2 JavaScript 源代码格式

当前 JavaScript 模块的格局非常多样化：ES6 带来了内置模块，但在它们之前出现的源代码格式仍然存在。了解后者有助于了解前者，因此让我们进行调查。接下来的章节描述了以下传递 JavaScript 源代码的方式：

+   *脚本*是浏览器在全局范围内运行的代码片段。它们是模块的前身。

+   *CommonJS 模块*是一种主要用于服务器的模块格式（例如，通过 Node.js）。

+   *AMD 模块*是一种主要用于浏览器的模块格式。

+   *ECMAScript 模块*是 JavaScript 的内置模块格式。它取代了所有以前的格式。

Tbl. 18 概述了这些代码格式。请注意，对于 CommonJS 模块和 ECMAScript 模块，通常使用两个文件扩展名。哪一个适合取决于我们想要如何使用文件。本章后面会详细介绍。

表 18：传递 JavaScript 源代码的方式。

|  | 运行在 | 加载 | 文件扩展名 |
| --- | --- | --- | --- |
| 脚本 | 浏览器 | 异步 | `.js` |
| CommonJS 模块 | 服务器 | 同步 | `.js .cjs` |
| AMD 模块 | 浏览器 | 异步 | `.js` |
| ECMAScript 模块 | 浏览器和服务器 | 异步 | `.js .mjs` |

#### 27.2.1 在内置模块之前的代码是用 ECMAScript 5 编写的

在我们进入内置模块（在 ES6 中引入）之前，我们将看到的所有代码都将以 ES5 编写。其中包括：

+   ES5 没有`const`和`let`，只有`var`。

+   ES5 没有箭头函数，只有函数表达式。

### 27.3 在我们有模块之前，我们有脚本

最初，浏览器只有*脚本* - 在全局范围内执行的代码片段。例如，考虑一个通过以下 HTML 加载脚本文件的 HTML 文件：

```js
<script src="other-module1.js"></script>
<script src="other-module2.js"></script>
<script src="my-module.js"></script>
```

主文件是`my-module.js`，在那里我们模拟一个模块：

```js
var myModule = (function () { // Open IIFE
 // Imports (via global variables)
 var importedFunc1 = otherModule1.importedFunc1;
 var importedFunc2 = otherModule2.importedFunc2;

 // Body
 function internalFunc() {
 // ···
 }
 function exportedFunc() {
 importedFunc1();
 importedFunc2();
 internalFunc();
 }

 // Exports (assigned to global variable `myModule`)
 return {
 exportedFunc: exportedFunc,
 };
})(); // Close IIFE
```

`myModule`是一个全局变量，它被赋予立即调用函数表达式的结果。函数表达式从第一行开始。它在最后一行被调用。

这种包装代码片段的方式称为*立即调用函数表达式*（IIFE，由 Ben Alman 创造）。我们从 IIFE 中获得了什么？`var`不是块作用域（像`const`和`let`一样），它是函数作用域：为`var`声明的变量创建新作用域的唯一方法是通过函数或方法（对于`const`和`let`，我们可以使用函数、方法或块`{}`）。因此，示例中的 IIFE 隐藏了所有以下变量的全局作用域，并最小化了名称冲突：`importedFunc1`，`importedFunc2`，`internalFunc`，`exportedFunc`。

请注意，我们以特定方式使用 IIFE：最后，我们选择要导出的内容，并通过对象字面量返回它。这被称为*揭示模块模式*（由 Christian Heilmann 创造）。

这种模拟模块的方式有几个问题：

+   脚本文件中的库通过全局变量导出和导入功能，这会导致名称冲突。

+   依赖关系没有明确说明，并且脚本没有内置的方法来加载它所依赖的脚本。因此，网页不仅需要加载页面所需的脚本，还需要加载这些脚本的依赖项，依赖项的依赖项等。而且它必须按正确的顺序这样做！

### 27.4 ES6 之前创建的模块系统

在 ECMAScript 6 之前，JavaScript 没有内置模块。因此，语言的灵活语法被用于在语言内部实现自定义模块系统。其中两种流行的是：

+   CommonJS（针对服务器端）

+   AMD（针对客户端的异步模块定义）

#### 27.4.1 服务器端：CommonJS 模块

模块的原始 CommonJS 标准是为服务器和桌面平台创建的。它是最初的 Node.js 模块系统的基础，在那里取得了巨大的流行。贡献于该流行的是 Node 的 npm 包管理器和工具，使得可以在客户端使用 Node 模块（browserify、webpack 等）。

从现在开始，*CommonJS 模块*指的是这个标准的 Node.js 版本（它还有一些额外的功能）。这是一个 CommonJS 模块的例子：

```js
// Imports
var importedFunc1 = require('./other-module1.js').importedFunc1;
var importedFunc2 = require('./other-module2.js').importedFunc2;

// Body
function internalFunc() {
 // ···
}
function exportedFunc() {
 importedFunc1();
 importedFunc2();
 internalFunc();
}

// Exports
module.exports = {
 exportedFunc: exportedFunc,
};
```

CommonJS 可以被描述如下：

+   设计用于服务器。

+   模块被设计为*同步*加载（导入者等待导入的模块被加载和执行）。

+   紧凑的语法。

#### 27.4.2 客户端：AMD（异步模块定义）模块

AMD 模块格式是为了在浏览器中比 CommonJS 格式更容易使用而创建的。它最流行的实现是[RequireJS](https://requirejs.org)。以下是一个 AMD 模块的例子。

```js
define(['./other-module1.js', './other-module2.js'],
 function (otherModule1, otherModule2) {
 var importedFunc1 = otherModule1.importedFunc1;
 var importedFunc2 = otherModule2.importedFunc2;

 function internalFunc() {
 // ···
 }
 function exportedFunc() {
 importedFunc1();
 importedFunc2();
 internalFunc();
 }

 return {
 exportedFunc: exportedFunc,
 };
 });
```

AMD 可以被描述如下：

+   设计用于浏览器。

+   模块被设计为*异步*加载。这对于浏览器来说是一个至关重要的要求，因为代码不能等待模块下载完成。它必须在模块可用时得到通知。

+   语法稍微复杂一些。

好处是，AMD 模块可以直接执行。相比之下，CommonJS 模块必须在部署之前编译，或者必须生成和动态评估自定义源代码(考虑`eval()`)。这在网络上并不总是被允许。

#### 27.4.3 JavaScript 模块的特点

查看 CommonJS 和 AMD，JavaScript 模块系统之间的相似之处显现出来：

+   每个文件有一个模块。

+   这样的文件基本上是一段被执行的代码：

    +   局部作用域：代码在局部的“模块作用域”中执行。因此，默认情况下，其中声明的所有变量、函数和类都是内部的，而不是全局的。

    +   导出：如果我们希望导出任何声明的实体，必须明确将其标记为导出。

    +   导入：每个模块可以从其他模块导入导出的实体。这些其他模块通过*模块规范符*（通常是路径，偶尔是完整 URL）来标识。

+   模块是*单例*的：即使一个模块被多次导入，它只存在一个“实例”。

+   不使用全局变量。相反，模块规范符充当全局 ID。

### 27.5 ECMAScript 模块

*ECMAScript 模块*（*ES 模块*或*ESM*）是在 ES6 中引入的。它延续了 JavaScript 模块的传统，并具有所有前述的特点。另外：

+   使用 CommonJS，ES 模块共享紧凑的语法和对循环依赖的支持。

+   使用 AMD，ES 模块共享被设计用于异步加载的特点。

ES 模块也有新的好处：

+   语法甚至比 CommonJS 更加紧凑。

+   模块具有*静态*结构（无法在运行时更改）。这有助于静态检查、优化导入的访问、死代码消除等。

+   对循环导入的支持是完全透明的。

这是一个 ES 模块语法的例子：

```js
import {importedFunc1} from './other-module1.mjs';
import {importedFunc2} from './other-module2.mjs';

function internalFunc() {
 ···
}

export function exportedFunc() {
 importedFunc1();
 importedFunc2();
 internalFunc();
}
```

从现在开始，“模块”指的是“ECMAScript 模块”。

#### 27.5.1 ES 模块：语法、语义、加载器 API

ES 模块的完整标准包括以下部分：

1.  语法（代码的编写方式）：什么是模块？如何声明导入和导出？等等。

1.  语义（代码的执行方式）：变量绑定是如何导出的？导入与导出如何连接？等等。

1.  用于配置模块加载的编程加载器 API。

部分 1 和 2 是在 ES6 中引入的。第 3 部分的工作正在进行中。

### 27.6 命名导出和导入

#### 27.6.1 命名导出

每个模块可以有零个或多个*命名导出*。

例如，考虑以下两个文件：

```js
lib/my-math.mjs
main.mjs
```

模块`my-math.mjs`有两个命名导出：`square`和`LIGHTSPEED`。

```js
// Not exported, private to module
function times(a, b) {
 return a * b;
}
export function square(x) {
 return times(x, x);
}
export const LIGHTSPEED = 299792458;
```

要导出某些东西，我们在声明前面放置关键字`export`。未导出的实体对于模块是私有的，无法从外部访问。

#### 27.6.2 命名导入

模块`main.mjs`有一个命名导入，`square`：

```js
import {square} from './lib/my-math.mjs';
assert.equal(square(3), 9);
```

它还可以重命名其导入：

```js
import {square as sq} from './lib/my-math.mjs';
assert.equal(sq(3), 9);
```

##### 27.6.2.1 语法陷阱：命名导入不是解构

命名导入和解构看起来很相似：

```js
import {foo} from './bar.mjs'; // import
const {foo} = require('./bar.mjs'); // destructuring
```

但它们是非常不同的：

+   导入保持与其导出的连接。

+   我们可以在解构模式内再次解构，但是导入语句中的`{}`不能嵌套。

+   重命名的语法不同：

    ```js
    import {foo as f} from './bar.mjs'; // importing
    const {foo: f} = require('./bar.mjs'); // destructuring
    ```

    原理：解构类似于对象文字（包括嵌套），而导入则唤起重命名的想法。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：命名导出**

`exercises/modules/export_named_test.mjs`

#### 27.6.3 命名空间导入

*命名空间导入*是命名导入的一种替代方法。如果我们对模块进行命名空间导入，它将成为一个对象，其属性是命名导出。如果我们使用命名空间导入，`main.mjs`看起来像这样：

```js
import * as myMath from './lib/my-math.mjs';
assert.equal(myMath.square(3), 9);

assert.deepEqual(
 Object.keys(myMath), ['LIGHTSPEED', 'square']);
```

#### 27.6.4 命名导出样式：内联与子句（高级）

到目前为止，我们看到的命名导出样式是*内联*的：我们通过在实体前面加上关键字`export`来导出实体。

但我们也可以使用单独的*导出子句*。例如，这是带有导出子句的`lib/my-math.mjs`的样子：

```js
function times(a, b) {
 return a * b;
}
function square(x) {
 return times(x, x);
}
const LIGHTSPEED = 299792458;

export { square, LIGHTSPEED }; // semicolon!
```

使用导出子句，我们可以在导出之前重命名并在内部使用不同的名称：

```js
function times(a, b) {
 return a * b;
}
function sq(x) {
 return times(x, x);
}
const LS = 299792458;

export {
 sq as square,
 LS as LIGHTSPEED, // trailing comma is optional
};
```

### 27.7 默认导出和导入

每个模块最多可以有一个*默认导出*。这个想法是模块*是*默认导出的值。

![](img/5fad46ca9f1c9224fc57d54750b4f1f4.png)  **避免混合命名导出和默认导出**

模块既可以具有命名导出，也可以具有默认导出，但通常最好每个模块只使用一种导出样式。

作为默认导出的示例，请考虑以下两个文件：

```js
my-func.mjs
main.mjs
```

模块`my-func.mjs`具有默认导出：

```js
const GREETING = 'Hello!';
export default function () {
 return GREETING;
}
```

模块`main.mjs`默认导入导出的函数：

```js
import myFunc from './my-func.mjs';
assert.equal(myFunc(), 'Hello!');
```

注意语法上的区别：命名导入周围的大括号表示我们正在*进入*模块，而默认导入*是*模块。

![](img/4e01a86e5fc2028470dca44da5c2d2aa.png)  **默认导出的用例是什么？**

默认导出最常见的用例是包含单个函数或单个类的模块。

#### 27.7.1 两种默认导出样式

有两种样式可以进行默认导出。

首先，我们可以使用`export default`标记现有声明：

```js
export default function myFunc() {} // no semicolon!
export default class MyClass {} // no semicolon!
```

其次，我们可以直接默认导出值。这种`export default`的样式很像一个声明。

```js
export default myFunc; // defined elsewhere
export default MyClass; // defined previously
export default Math.sqrt(2); // result of invocation is default-exported
export default 'abc' + 'def';
export default { no: false, yes: true };
```

##### 27.7.1.1 为什么有两种默认导出样式？

原因是`export default`不能用于标记`const`：`const`可以定义多个值，但`export default`需要确切的一个值。考虑以下假设的代码：

```js
// Not legal JavaScript!
export default const foo = 1, bar = 2, baz = 3;
```

使用此代码，我们不知道三个值中的哪一个是默认导出。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：默认导出**

`exercises/modules/export_default_test.mjs`

#### 27.7.2 默认导出作为命名导出（高级）

在内部，默认导出只是一个名为`default`的命名导出。例如，考虑具有默认导出的先前模块`my-func.mjs`：

```js
const GREETING = 'Hello!';
export default function () {
 return GREETING;
}
```

以下模块`my-func2.mjs`等效于该模块：

```js
const GREETING = 'Hello!';
function greet() {
 return GREETING;
}

export {
 greet as default,
};
```

对于导入，我们可以使用普通的默认导入：

```js
import myFunc from './my-func2.mjs';
assert.equal(myFunc(), 'Hello!');
```

或者我们可以使用命名导入：

```js
import {default as myFunc} from './my-func2.mjs';
assert.equal(myFunc(), 'Hello!');
```

默认导出也可以通过命名空间导入的属性`.default`来使用：

```js
import * as mf from './my-func2.mjs';
assert.equal(mf.default(), 'Hello!');
```

![](img/4e01a86e5fc2028470dca44da5c2d2aa.png)  **`default`作为变量名是非法的吗？**

`default`不能是变量名，但它可以是导出名称，也可以是属性名称：

```js
const obj = {
 default: 123,
};
assert.equal(obj.default, 123);
```

### 27.8 导出和导入的更多细节

#### 27.8.1 导入是对导出的只读视图

到目前为止，我们已经直触使用了导入和导出，并且一切似乎都按预期工作。但现在是时候更仔细地看一下导入和导出的真正关系了。

考虑以下两个模块：

```js
counter.mjs
main.mjs
```

`counter.mjs`导出一个（可变的！）变量和一个函数：

```js
export let counter = 3;
export function incCounter() {
 counter++;
}
```

`main.mjs`名称导入了两个导出。当我们使用`incCounter()`时，我们发现与`counter`的连接是实时的-我们始终可以访问该变量的实时状态。

```js
import { counter, incCounter } from './counter.mjs';

// The imported value `counter` is live
assert.equal(counter, 3);
incCounter();
assert.equal(counter, 4);
```

请注意，虽然连接是实时的，我们可以读取`counter`，但我们不能更改这个变量（例如，通过`counter++`）。

以这种方式处理导入有两个好处：

+   拆分模块变得更容易，因为以前共享的变量可以成为导出。

+   这种行为对支持透明的循环导入至关重要。继续阅读以获取更多信息。

#### 27.8.2 ESM 对透明支持循环导入（高级）

ESM 支持透明的循环导入。要了解如何实现这一点，考虑以下示例：图 7 显示了一个模块导入其他模块的有向图。在这种情况下，P 导入 M 是循环。

![图 7：模块导入模块的有向图：M 导入 N 和 O，N 导入 P 和 Q，等等。](img/36a86530c92f02bc0545c60194aa676d.png)

图 7：模块导入模块的有向图：M 导入 N 和 O，N 导入 P 和 Q，等等。

解析后，这些模块分两个阶段设置：

+   实例化：访问每个模块并将其导入连接到其导出。在父级实例化之前，必须先实例化其所有子级。

+   评估：执行模块的主体。再次强调，必须在父级之前评估子级。

这种方法正确处理了循环导入，这是由于 ES 模块的两个特性：

+   由于 ES 模块的静态结构，解析后导出已经是已知的。这使得在其子级 M 之前实例化 P 成为可能：P 已经可以查找 M 的导出。

+   当 P 被评估时，M 尚未被评估。但是，P 中的实体可以已经提到了来自 M 的导入。它们只是还不能使用它们，因为导入的值稍后填充。例如，P 中的一个函数可以访问来自 M 的导入。唯一的限制是我们必须等到 M 的评估之后，才能调用该函数。

    导入稍后填充是由它们成为对导出的“实时不可变视图”而启用的。

### 27.9 npm 包

*npm 软件注册表*是分发 JavaScript 库和 Node.js 和 Web 浏览器应用程序的主要方式。它通过*npm 软件包管理器*（简称*npm*）进行管理。软件以所谓的*包*的形式分发。包是一个包含任意文件和顶层描述该包的`package.json`文件的目录。例如，当 npm 在目录`my-package/`中创建一个空包时，我们得到了这个`package.json`：

```js
{
 "name": "my-package",
 "version": "1.0.0",
 "description": "",
 "main": "index.js",
 "scripts": {
 "test": "echo \"Error: no test specified\" && exit 1"
 },
 "keywords": [],
 "author": "",
 "license": "ISC"
}
```

其中一些属性包含简单的元数据：

+   `name`指定了这个包的名称。一旦上传到 npm 注册表，就可以通过`npm install my-package`进行安装。

+   `version`用于版本管理，并遵循[语义化版本](https://semver.org)，有三个数字：

    +   主要版本：当进行不兼容的 API 更改时递增。

    +   次要版本：在向后兼容的方式下添加功能时递增。

    +   补丁版本：在向后兼容的方式下进行更改时递增。

+   `description`，`keywords`，`author`使得更容易找到包。

+   `license`澄清了我们如何使用这个包。

其他属性使高级配置成为可能：

+   `main`：指定“是”包的模块（本章后面会解释）。

+   `scripts`：是我们可以通过`npm run`执行的命令。例如，可以通过`npm run test`执行脚本`test`。

有关`package.json`的更多信息，请参阅[npm 文档](https://docs.npmjs.com/files/package.json)。

#### 27.9.1 包被安装在目录`node_modules/`内

npm 总是将包安装在`node_modules`目录中。通常会有许多这样的目录。npm 使用哪一个取决于当前所在的目录。例如，如果我们在目录`/tmp/a/b/`中，npm 会尝试在当前目录、其父目录、父目录的父目录等中找到`node_modules`。换句话说，它会搜索以下位置的*链*：

+   `/tmp/a/b/node_modules`

+   `/tmp/a/node_modules`

+   `/tmp/node_modules`

当安装一个包`some-pkg`时，npm 会使用最近的`node_modules`。例如，如果我们在`/tmp/a/b/`中，并且该目录中有一个`node_modules`，那么 npm 会将包放在该目录中：

```js
/tmp/a/b/node_modules/some-pkg/
```

在导入模块时，我们可以使用特殊的模块规范告诉 Node.js 我们想要从已安装的包中导入它。这是如何工作的，稍后会有解释。现在，考虑以下示例：

```js
// /home/jane/proj/main.mjs
import * as theModule from 'the-package/the-module.mjs';
```

寻找`the-module.mjs`（Node.js 更喜欢使用文件扩展名`.mjs`来表示 ES 模块），Node.js 会沿着`node_module`链向上查找以下位置：

+   `/home/jane/proj/node_modules/the-package/the-module.mjs`

+   `/home/jane/node_modules/the-package/the-module.mjs`

+   `/home/node_modules/the-package/the-module.mjs`

#### 27.9.2 为什么可以使用 npm 来安装前端库？

在`node_modules`目录中查找已安装的模块只在 Node.js 上受支持。那么为什么我们也可以使用 npm 来为浏览器安装库呢？

这是通过打包工具（例如 webpack）启用的，它会在部署到线上之前编译和优化代码。在这个编译过程中，npm 包中的代码会被调整，以便在浏览器中运行。

### 27.10 命名模块

对于命名模块文件和导入到其中的变量，没有已建立的最佳实践。

在本章中，我使用以下命名风格：

+   模块文件的名称是破折号命名，并以小写字母开头：

    ```js
    ./my-module.mjs
    ./some-func.mjs
    ```

+   命名空间导入的名称是小写和驼峰式命名：

    ```js
    import * as myModule from './my-module.mjs';
    ```

+   默认导入的名称是小写和驼峰式命名：

    ```js
    import someFunc from './some-func.mjs';
    ```

这种风格背后的原因是什么？

+   npm 不允许包名中有大写字母（[来源](https://docs.npmjs.com/files/package.json#name)）。因此，我们避免驼峰命名，以使“本地”文件的名称与 npm 包的名称一致。仅使用小写字母也可以最小化大小写敏感和不敏感文件系统之间的冲突：前者区分具有相同字母但大小写不同的名称的文件；后者则不区分。

+   有明确的规则将破折号命名的文件名转换为驼峰式的 JavaScript 变量名。由于我们命名命名空间导入的方式，这些规则适用于命名空间导入和默认导入。

我也喜欢使用下划线命名的模块文件名，因为我们可以直接使用这些名称进行命名空间导入（无需任何转换）：

```js
import * as my_module from './my_module.mjs';
```

但是这种风格对于默认导入不起作用：我喜欢使用下划线命名空间对象，但对于函数等来说并不是一个好选择。

### 27.11 模块规范

*模块规范*是用于标识模块的字符串。它们在浏览器和 Node.js 中有略微不同的工作方式。在我们看到区别之前，我们需要了解不同类别的模块规范。

#### 27.11.1 模块规范的类别

在 ES 模块中，我们区分以下类别的规范。这些类别起源于 CommonJS 模块。

+   相对路径：以点开头。例如：

    ```js
    './some/other/module.mjs'
    '../../lib/counter.mjs'
    ```

+   绝对路径：以斜杠开头。例如：

    ```js
    '/home/jane/file-tools.mjs'
    ```

+   URL：包括协议（从技术上讲，路径也是 URL）。例如：

    ```js
    'https://example.com/some-module.mjs'
    'file:///home/john/tmp/main.mjs'
    ```

+   裸路径：不以点、斜杠或协议开头，由单个没有扩展名的文件名组成。例如：

    ```js
    'lodash'
    'the-package'
    ```

+   深层导入路径：以裸路径开头，并且至少有一个斜杠。例如：

    ```js
    'the-package/dist/the-module.mjs'
    ```

#### 27.11.2 浏览器中的 ES 模块规范

浏览器处理模块规范如下：

+   相对路径、绝对路径和 URL 都能按预期工作。它们都必须指向真实的文件（与 CommonJS 相反，它允许我们省略文件扩展名等）。

+   模块的文件名扩展名并不重要，只要它们以`text/javascript`的内容类型提供即可。

+   裸路径最终将如何处理尚不清楚。我们可能最终能够通过查找表将它们映射到其他规范。

请注意，打包工具（如 webpack）将模块合并为较少的文件，通常对规范要求不那么严格。这是因为它们在构建/编译时操作（而不是在运行时），并且可以通过遍历文件系统来搜索文件。

#### 27.11.3 Node.js 上的 ES 模块规范

Node.js 处理模块规范如下：

+   相对路径会像在 Web 浏览器中一样解析 - 相对于当前模块的路径。

+   目前不支持绝对路径。作为一种解决方法，我们可以使用以`file:///`开头的 URL。我们可以通过`url.pathToFileURL()`创建这样的 URL。

+   只支持`file:`作为 URL 规范的协议。

+   裸路径被解释为包名，并相对于最近的`node_modules`目录进行解析。应该加载哪个模块是通过查看包的`package.json`的`"main"`属性来确定的（类似于 CommonJS）。

+   深层导入路径也相对于最近的`node_modules`目录进行解析。它们包含文件名，因此始终清楚指的是哪个模块。

除了裸路径之外，所有的规范都必须指向实际的文件。也就是说，ESM 不支持以下 CommonJS 特性：

+   CommonJS 会自动添加丢失的文件扩展名。

+   如果存在带有`"main"`属性的`dir/package.json`，CommonJS 可以导入目录`dir`。

+   如果存在`dir/index.js`模块，CommonJS 可以导入目录`dir`。

所有内置的 Node.js 模块都可以通过裸路径访问，并且具有命名的 ESM 导出 - 例如：

```js
import * as assert from 'assert/strict';
import * as path from 'path';

assert.equal(
 path.join('a/b/c', '../d'), 'a/b/d');
```

##### 27.11.3.1 Node.js 上的文件扩展名

Node.js 支持以下默认的文件扩展名：

+   `.mjs`用于 ES 模块

+   `.cjs`用于 CommonJS 模块

文件扩展名`.js`代表 ESM 或 CommonJS。它是通过“最接近”的`package.json`（在当前目录、父目录等）配置的。使用这种方式的`package.json`与包是独立的。

在`package.json`中，有一个`"type"`属性，它有两个设置：

+   `"commonjs"`（默认）：具有扩展名`.js`或没有扩展名的文件被解释为 CommonJS 模块。

+   `"module"`：具有扩展名`.js`或没有扩展名的文件被解释为 ESM 模块。

##### 27.11.3.2 将非文件源代码解释为 CommonJS 或 ESM

Node.js 执行的并非所有源代码都来自文件。我们也可以通过 stdin、`--eval`和`--print`向其发送代码。命令行选项`--input-type`让我们指定如何解释这样的代码：

+   作为 CommonJS（默认）：`--input-type=commonjs`

+   作为 ESM：`--input-type=module`

### 27.12 `import.meta` - 当前模块的元数据[ES2020]

对象`import.meta`保存了当前模块的元数据。

#### 27.12.1 `import.meta.url`

`import.meta`最重要的属性是`.url`，其中包含当前模块文件的 URL 字符串 - 例如：

```js
'https://example.com/code/main.mjs'
```

#### 27.12.2 `import.meta.url`和类`URL`

在浏览器和 Node.js 中，类`URL`可以通过全局变量访问。我们可以在[Node.js 文档](https://nodejs.org/api/url.html#url_class_url)中查找其完整功能。在使用`import.meta.url`时，它的构造函数特别有用：

```js
new URL(input: string, base?: string|URL)
```

参数`input`包含要解析的 URL。如果提供了第二个参数`base`，它可以是相对的。

换句话说，这个构造函数让我们根据基本 URL 解析相对路径：

```js
> new URL('other.mjs', 'https://example.com/code/main.mjs').href
'https://example.com/code/other.mjs'
> new URL('../other.mjs', 'https://example.com/code/main.mjs').href
'https://example.com/other.mjs'
```

这是我们如何获得一个指向与当前模块相邻的文件`data.txt`的`URL`实例：

```js
const urlOfData = new URL('data.txt', import.meta.url);
```

#### 27.12.3 Node.js 上的`import.meta.url`

在 Node.js 上，`import.meta.url`始终是一个带有`file:` URL 的字符串 - 例如：

```js
'file:///Users/rauschma/my-module.mjs'
```

##### 27.12.3.1 示例：读取模块的同级文件

许多 Node.js 文件系统操作接受路径字符串或`URL`实例。这使我们能够读取当前模块的同级文件`data.txt`：

```js
import * as fs from 'fs';
function readData() {
 // data.txt sits next to current module
 const urlOfData = new URL('data.txt', import.meta.url);
 return fs.readFileSync(urlOfData, {encoding: 'UTF-8'});
}
```

##### 27.12.3.2 模块`fs`和 URL

对于模块`fs`的大多数函数，我们可以通过以下方式引用文件：

+   路径 - 以字符串或`Buffer`实例的形式。

+   URL - 在`URL`实例中（使用协议`file:`）

有关此主题的更多信息，请参阅[Node.js API 文档](https://nodejs.org/api/fs.html#fs_file_paths)。

##### 27.12.3.3 在`file:` URL 和路径之间转换

[Node.js 模块`url`](https://nodejs.org/api/url.html)有两个函数用于在`file:` URL 和路径之间转换：

+   `fileURLToPath(url: URL|string): string`

    将`file:` URL 转换为路径。

+   `pathToFileURL(path: string): URL`

    将路径转换为`file:` URL。

如果我们需要一个可以在本地文件系统中使用的路径，那么`URL`实例的`.pathname`属性并不总是有效：

```js
assert.equal(
 new URL('file:///tmp/with%20space.txt').pathname,
 '/tmp/with%20space.txt');
```

因此，最好使用`fileURLToPath()`：

```js
import * as url from 'url';
assert.equal(
 url.fileURLToPath('file:///tmp/with%20space.txt'),
 '/tmp/with space.txt'); // result on Unix
```

同样，`pathToFileURL()`不仅仅是在绝对路径前面添加`'file://'`。

### 27.13 通过`import()`动态加载模块[ES2020]（高级）

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png) **`import()`操作符使用 Promises**

Promises 是一种处理异步计算结果的技术（即不是立即计算的）。它们在[§40“Promises for asynchronous programming [ES6]”](ch_promises.html)中有解释。在理解它们之前，推迟阅读本节可能是有意义的。

#### 27.13.1 静态`import`语句的限制

到目前为止，导入模块的唯一方法是通过`import`语句。该语句有几个限制：

+   我们必须在模块的顶层使用它。也就是说，我们不能在函数内部或`if`语句内部导入某些东西。

+   模块标识符是固定的。也就是说，我们不能根据条件改变导入的内容。我们也不能动态组装一个标识符。

#### 27.13.2 通过`import()`操作符动态导入

`import()`操作符没有`import`语句的限制。它看起来像这样：

```js
import(moduleSpecifierStr)
.then((namespaceObject) => {
 console.log(namespaceObject.namedExport);
});
```

这个操作符像一个函数一样使用，接收一个带有模块标识符的字符串，并返回一个解析为命名空间对象的 Promise。该对象的属性是导入模块的导出。

通过`await`来使用`import()`更加方便：

```js
const namespaceObject = await import(moduleSpecifierStr);
console.log(namespaceObject.namedExport);
```

请注意，`await`可以在模块的顶层使用（参见下一节）。

让我们看一个使用`import()`的例子。

##### 27.13.2.1 示例：动态加载模块

考虑以下文件：

```js
lib/my-math.mjs
main1.mjs
main2.mjs
```

我们已经看到了模块`my-math.mjs`：

```js
// Not exported, private to module
function times(a, b) {
 return a * b;
}
export function square(x) {
 return times(x, x);
}
export const LIGHTSPEED = 299792458;
```

我们可以使用`import()`按需加载这个模块：

```js
// main1.mjs
const moduleSpecifier = './lib/my-math.mjs';

function mathOnDemand() {
 return import(moduleSpecifier)
 .then(myMath => {
 const result = myMath.LIGHTSPEED;
 assert.equal(result, 299792458);
 return result;
 });
}

mathOnDemand()
.then((result) => {
 assert.equal(result, 299792458);
});
```

这段代码中有两件事是无法通过`import`语句完成的：

+   我们在函数内部导入（而不是在顶层）。

+   模块标识符来自一个变量。

接下来，我们将实现与`main1.mjs`中相同的功能，但通过一个称为*async function*或*async/await*的特性来实现，它为 Promises 提供了更好的语法。

```js
// main2.mjs
const moduleSpecifier = './lib/my-math.mjs';

async function mathOnDemand() {
 const myMath = await import(moduleSpecifier);
 const result = myMath.LIGHTSPEED;
 assert.equal(result, 299792458);
 return result;
}
```

![](img/4e01a86e5fc2028470dca44da5c2d2aa.png) **为什么`import()`是一个操作符而不是一个函数？**

`import()`看起来像一个函数，但不能作为一个函数实现：

+   它需要知道当前模块的 URL 以解析相对模块标识符。

+   如果`import()`是一个函数，我们必须明确地将这些信息传递给它（例如通过参数）。

+   相比之下，操作符是一种核心语言构造，并且隐式访问更多数据，包括当前模块的 URL。

#### 27.13.3 `import()`的用例

##### 27.13.3.1 按需加载代码

Web 应用程序的一些功能在启动时不必存在，可以按需加载。然后`import()`有所帮助，因为我们可以将这样的功能放入模块中 - 例如：

```js
button.addEventListener('click', event => {
 import('./dialogBox.mjs')
 .then(dialogBox => {
 dialogBox.open();
 })
 .catch(error => {
 /* Error handling */
 })
});
```

##### 27.13.3.2 模块的条件加载

我们可能希望根据条件是否为真来加载一个模块。例如，具有 Polyfill 的模块可以在旧平台上提供新功能：

```js
if (isLegacyPlatform()) {
 import('./my-polyfill.mjs')
 .then(···);
}
```

##### 27.13.3.3 计算模块规范

对于国际化等应用程序，如果我们可以动态计算模块规范，这将有所帮助：

```js
import(`messages_${getLocale()}.mjs`)
 .then(···);
```

### 27.14 模块中的顶层`await` [ES2022]（高级）

![](img/ec8e6930fbe484fc519f3aa7b812c3fd.png)  **`await`是异步函数的一个特性**

`await`在§41“异步函数”中有解释。在理解异步函数之前，推迟阅读本节可能是有意义的。

我们可以在模块的顶层使用`await`运算符。如果这样做，模块将变成异步的，并且工作方式会有所不同。幸运的是，我们通常不会作为程序员看到这一点，因为语言会透明地处理它。

#### 27.14.1 顶层`await`的用例

为什么我们希望在模块的顶层使用`await`运算符？它让我们可以使用异步加载的数据初始化模块。接下来的三个小节展示了这种用法的三个例子。

##### 27.14.1.1 动态加载模块

```js
const params = new URLSearchParams(location.search);
const language = params.get('lang');
const messages = await import(`./messages-${language}.mjs`); // (A)

console.log(messages.welcome);
```

在 A 行，我们动态导入一个模块。由于顶层`await`，这几乎和使用普通的静态导入一样方便。

##### 27.14.1.2 在模块加载失败时使用备用

```js
let lodash;
try {
 lodash = await import('https://primary.example.com/lodash');
} catch {
 lodash = await import('https://secondary.example.com/lodash');
}
```

##### 27.14.1.3 使用加载最快的资源

```js
const resource = await Promise.any([
 fetch('http://example.com/first.txt')
 .then(response => response.text()),
 fetch('http://example.com/second.txt')
 .then(response => response.text()),
]);
```

由于`Promise.any()`，变量`resource`是通过任何首次完成的下载进行初始化。

#### 27.14.2 顶层`await`在底层是如何工作的？

考虑以下两个文件。

`first.mjs`：

```js
const response = await fetch('http://example.com/first.txt');
export const first = await response.text();
```

`main.mjs`：

```js
import {first} from './first.mjs';
import {second} from './second.mjs';
assert.equal(first, 'First!');
assert.equal(second, 'Second!');
```

两者大致等同于以下代码：

`first.mjs`：

```js
export let first;
export const promise = (async () => { // (A)
 const response = await fetch('http://example.com/first.txt');
 first = await response.text();
})();
```

`main.mjs`：

```js
import {promise as firstPromise, first} from './first.mjs';
import {promise as secondPromise, second} from './second.mjs';
export const promise = (async () => { // (B)
 await Promise.all([firstPromise, secondPromise]); // (C)
 assert.equal(first, 'First content!');
 assert.equal(second, 'Second content!');
})();
```

如果：

1.  它直接使用顶层`await`（`first.mjs`）。

1.  它导入一个或多个异步模块（`main.mjs`）。

每个异步模块都导出一个 Promise（A 行和 B 行），在其主体执行后实现。在这一点上，安全地访问该模块的导出。

在情况（2）中，导入模块会等待所有导入的异步模块的 Promise 被实现，然后再进入其主体（C 行）。同步模块则像通常一样处理。

等待的拒绝和同步异常的处理方式与异步函数中的处理方式相同。

#### 27.14.3 顶层`await`的利弊

顶层`await`的两个最重要的好处是：

+   它确保模块在完全初始化之前不会访问异步导入。

+   它透明地处理异步性：导入者不需要知道导入的模块是异步的还是同步的。

另一方面，顶层`await`延迟了导入模块的初始化。因此，最好是谨慎使用。需要更长时间的异步任务最好稍后执行。

然而，即使没有顶层`await`的模块也可能阻止导入者（例如在顶层的无限循环中），因此阻塞本身并不是反对它的论点。

### 27.15 Polyfills：模拟原生 Web 平台功能（高级）

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  **后端也有 Polyfills**

本节是关于前端开发和 Web 浏览器，但类似的想法也适用于后端开发。

*Polyfills*有助于解决我们在 JavaScript 中开发 Web 应用程序时面临的冲突：

+   一方面，我们希望使用使应用程序更好和/或开发更容易的现代 Web 平台功能。

+   另一方面，应用程序应该在尽可能多的浏览器上运行。

考虑一个 Web 平台功能 X：

+   X 的*polyfill*是一段代码。如果它在已经内置对 X 的支持的平台上执行，它什么也不做。否则，它会在平台上提供该功能。在后一种情况下，polyfilled 功能（大多数情况下）与本机实现几乎无法区分。为了实现这一点，polyfill 通常会进行全局更改。例如，它可能修改全局数据或配置全局模块加载器。Polyfills 通常打包为模块。

    +   术语[*polyfill*](https://remysharp.com/2010/10/08/what-is-a-polyfill)是由 Remy Sharp 创造的。

+   *推测性 polyfill*是针对提议的 Web 平台功能的 polyfill（尚未标准化）。

    +   替代术语：*prollyfill*

+   X 的*复制品*是一个在本地复制 X 的 API 和功能的库。这样的库独立于 X 的本机（和全局）实现。

    +   *复制品*是本节中引入的一个新术语。替代术语：*ponyfill*

+   还有一个术语*shim*，但它没有一个普遍认可的定义。它通常意思大致相同于*polyfill*。

每次我们的 Web 应用程序启动时，它必须首先执行所有可能不是在所有地方都可用的功能的 polyfills。之后，我们可以确保这些功能在本地是可用的。

#### 27.15.1 本节的来源

+   [“什么是 Polyfill？”](https://remysharp.com/2010/10/08/what-is-a-polyfill) by Remy Sharp

+   *复制品*这个术语的灵感来源：[拉斯维加斯的埃菲尔铁塔](https://en.wikipedia.org/wiki/Paris_Las_Vegas)

+   有用的澄清“polyfill”和相关术语：[“Polyfills and the evolution of the Web”](https://www.w3.org/2001/tag/doc/polyfills/)。由 Andrew Betts 编辑。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

参见测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/20)
