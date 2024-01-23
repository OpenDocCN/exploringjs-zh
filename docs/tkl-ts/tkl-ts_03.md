# 二、为什么使用 TypeScript？

> 原文：[`exploringjs.com/tackling-ts/ch_why-typescript.html`](https://exploringjs.com/tackling-ts/ch_why-typescript.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   2.1 [使用 TypeScript 的好处]

    +   2.1.1 [更多错误可以被*静态*（不运行代码）检测到]

    +   2.1.2 [记录参数无论如何都是一个好习惯]

    +   2.1.3 [TypeScript 提供了额外的文档层]

    +   2.1.4 [JavaScript 的类型定义改进了自动补全]

    +   2.1.5 [TypeScript 使重构更安全]

    +   2.1.6 [TypeScript 可以将新功能编译成旧代码]

+   2.2 [使用 TypeScript 的缺点]

+   2.3 [TypeScript 的迷思]

    +   2.3.1 [TypeScript 代码比较庞大]

    +   2.3.2 [TypeScript 试图用 C#或 Java 替换 JavaScript]

* * *

如果你已经确定会学习和使用 TypeScript，可以跳过本章。

如果你还不确定 - 这一章是我的推销。

### 2.1 [使用 TypeScript 的好处]

#### 2.1.1 更多错误可以被*静态*（不运行代码）检测到

当你在集成开发环境中编辑 TypeScript 代码时，如果你拼错了名称，错误地调用函数等，你会收到警告。

考虑以下两行代码：

```ts
function func() {}
funcc();
```

对于第二行，我们得到了这个警告：

```ts
Cannot find name 'funcc'. Did you mean 'func'?
```

另一个例子：

```ts
const a = 0;
const b = true;
const result = a + b;
```

这次，最后一行的错误消息是：

```ts
Operator '+' cannot be applied to types 'number' and 'boolean'.
```

#### 2.1.2 记录参数无论如何都是一个好习惯

记录函数和方法的参数是许多人都会做的事情：

```ts
/**
 * @param  {number} num - The number to convert to string
 * @returns {string} `num`, converted to string
 */
function toString(num) {
 return String(num);
}
```

通过`{number}`和`{string}`指定类型并不是必需的，但英文描述中也提到了它们。

如果我们使用 TypeScript 的符号来记录类型，我们会得到这些信息被检查一致性的额外好处：

```ts
function toString(num: number): string {
 return String(num);
}
```

#### 2.1.3 TypeScript 提供了额外的文档层

每当我将 JavaScript 代码迁移到 TypeScript 时，我都会注意到一个有趣的现象：为了找到函数或方法的参数的适当类型，我必须检查它在哪里被调用。这意味着静态类型给了我本地的信息，否则我必须在其他地方查找。

而且我确实发现理解 TypeScript 代码库比理解 JavaScript 代码库更容易：TypeScript 提供了额外的文档层。

这种额外的文档也有助于团队合作，因为清楚了代码的使用方式，而且 TypeScript 经常会在我们做错事情时提醒我们。

#### 2.1.4 JavaScript 的类型定义改进了自动补全

如果 JavaScript 代码有类型定义，那么编辑器可以使用它们来改进自动补全。

除了使用 TypeScript 的语法之外，还可以通过 JSDoc 注释提供所有类型信息 - 就像我们在本章开头所做的那样。在这种情况下，TypeScript 也可以检查代码的一致性并生成类型定义。有关更多信息，请参见 TypeScript 手册中的“类型检查 JavaScript 文件”章节。

#### 2.1.5 TypeScript 使重构更安全

重构是许多集成开发环境提供的自动化代码转换。

重命名方法是重构的一个例子。在纯 JavaScript 中这样做可能会很棘手，因为相同的名称可能指代不同的方法。TypeScript 对方法和类型的连接有更多信息，这使得在那里重命名方法更安全。

#### 2.1.6 TypeScript 可以将新功能编译为旧代码

TypeScript 倾向于快速支持 ECMAScript 阶段 4 的功能（这些功能计划包括在下一个 ECMAScript 版本中）。当我们编译为 JavaScript 时，编译器选项`--target`让我们指定输出与兼容的 ECMAScript 版本。然后，任何不兼容的功能（后来引入的）将被编译为等效的兼容代码。

请注意，对于较旧的 ECMAScript 版本的这种支持并不需要 TypeScript 或静态类型：[JavaScript 编译器 Babel](https://babeljs.io)也可以做到，但它将 JavaScript 编译为 JavaScript。

### 2.2 使用 TypeScript 的缺点

+   这是 JavaScript 之上的一个附加层：更复杂，更多要学习的东西，等等。

+   它在编写代码时引入了一个编译步骤。

+   只有当 npm 软件包具有静态类型定义时才能使用。

    +   如今，许多软件包要么附带类型定义，要么在[DefinitelyTyped](http://definitelytyped.org)上有可用的类型定义。然而，尤其是后者偶尔可能会有些错误，这会导致一些在没有静态类型的情况下不会出现的问题。

+   偶尔很难正确使用静态类型。我的建议是尽可能保持简单 - 例如：不要过度使用泛型和类型变量。

### 2.3 TypeScript 的神话

#### 2.3.1 TypeScript 代码很庞大

TypeScript 代码*可以*非常庞大。但不一定非得如此。例如，由于类型推断，我们通常可以少用一些类型注释：

```ts
function selectionSort(arr: number[]) { // (A)
 for (let i=0; i<arr.length; i++) {
 const minIndex = findMinIndex(arr, i);
 [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]]; // swap
 }
}

function findMinIndex(arr: number[], startIndex: number) { // (B)
 let minValue = arr[startIndex];
 let minIndex = startIndex;
 for (let i=startIndex+1; i < arr.length; i++) {
 const curValue = arr[i];
 if (curValue < minValue) {
 minValue = curValue;
 minIndex = i;
 }
 }
 return minIndex;
}

const arr = [4, 2, 6, 3, 1, 5];
selectionSort(arr);
assert.deepEqual(
 arr, [1, 2, 3, 4, 5, 6]);
```

这个 TypeScript 代码与 JavaScript 代码不同的唯一位置是 A 行和 B 行。

TypeScript 的编写方式有多种风格：

+   在面向对象编程（OOP）风格中使用类和 OOP 模式

+   在函数式编程（FP）风格中使用函数模式

+   在 OOP 和 FP 的混合中

+   等等。

#### 2.3.2 TypeScript 试图用 C#或 Java 替换 JavaScript

最初，TypeScript 确实发明了一些自己的语言构造（例如枚举）。但自 ECMAScript 6 以来，它主要坚持成为 JavaScript 的严格超集。

我的印象是 TypeScript 团队喜欢 JavaScript，并且不想用“更好”的东西（例如 Dart）来替换它。他们确实希望尽可能地使 JavaScript 代码具有静态类型。许多新的 TypeScript 功能都是出于这种愿望驱动的。

[评论](https://github.com/rauschma/tackling-ts/issues/2)
