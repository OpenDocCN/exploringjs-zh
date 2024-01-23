# 十三、实例化类的技术

> 原文：[`exploringjs.com/deep-js/ch_creating-class-instances.html`](https://exploringjs.com/deep-js/ch_creating-class-instances.html)

* * *

+   13.1 问题：异步初始化属性

+   13.2 解决方案：基于 Promise 的构造函数

    +   13.2.1 使用立即调用的异步箭头函数

+   13.3 解决方案：静态工厂方法

    +   13.3.1 改进：通过秘密令牌私有构造函数

    +   13.3.2 改进：构造函数抛出，工厂方法借用类原型

    +   13.3.3 改进：实例默认处于非活动状态，由工厂方法激活

    +   13.3.4 变体：单独的工厂函数

+   13.4 基于 Promise 的构造函数的子类化（可选）

+   13.5 结论

+   13.6 进一步阅读

* * *

在本章中，我们将研究创建类实例的几种方法：构造函数，工厂函数等。我们通过多次解决一个具体的问题来做到这一点。本章的重点是类，因此忽略了类的替代方案。

### 13.1 问题：异步初始化属性

以下容器类应该异步接收其属性`.data`的内容。这是我们的第一次尝试：

```js
class DataContainer {
 #data; // (A)
 constructor() {
 Promise.resolve('downloaded')
 .then(data => this.#data = data); // (B)
 }
 getData() {
 return 'DATA: '+this.#data; // (C)
 }
}
```

此代码的关键问题：属性`.data`最初为`undefined`。

```js
const dc = new DataContainer();
assert.equal(dc.getData(), 'DATA: undefined');
setTimeout(() => assert.equal(
 dc.getData(), 'DATA: downloaded'), 0);
```

在 A 行，我们声明了[私有字段](https://2ality.com/2019/07/private-class-fields.html) `.#data`，我们在 B 行和 C 行中使用。

`DataContainer`构造函数内部的 Promise 是异步解决的，这就是为什么我们只有在完成当前任务并启动新任务（通过`setTimeout()`）后才能看到`.data`的最终值。换句话说，当我们第一次看到它时，`DataContainer`实例尚未完全初始化。

### 13.2 解决方案：基于 Promise 的构造函数

如果我们延迟访问`DataContainer`实例直到完全初始化，会怎么样？我们可以通过从构造函数返回一个 Promise 来实现这一点。默认情况下，构造函数返回其所属类的新实例。如果我们明确返回一个对象，我们可以覆盖它：

```js
class DataContainer {
 #data;
 constructor() {
 return Promise.resolve('downloaded')
 .then(data => {
 this.#data = data;
 return this; // (A)
 });
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}
new DataContainer()
 .then(dc => assert.equal( // (B)
 dc.getData(), 'DATA: downloaded'));
```

现在我们必须等到可以访问我们的实例（B 行）。在数据“下载”后（A 行）将其传递给我们。此代码中可能出现两种错误：

+   下载可能失败并产生拒绝。

+   在第一个`.then()`回调的主体中可能会抛出异常。

在任何一种情况下，错误都会成为从构造函数返回的 Promise 的拒绝。

利弊：

+   这种方法的好处是，只有在完全初始化后才能访问实例。没有其他方法可以创建`DataContainer`的实例。

+   一个缺点是构造函数返回一个 Promise 而不是一个实例。 

#### 13.2.1 使用立即调用的异步箭头函数

不是直接使用 Promise API 来创建从构造函数返回的 Promise，我们也可以使用一个异步箭头函数，[我们立即调用](https://exploringjs.com/impatient-js/ch_async-functions.html#immediately-invoked-async-arrow-functions)：

```js
constructor() {
 return (async () => {
 this.#data = await Promise.resolve('downloaded');
 return this;
 })();
}
```

### 13.3 解决方案：静态工厂方法

类`C`的*静态工厂方法*创建`C`的实例，是使用`new C()`的替代方法。JavaScript 中静态工厂方法的常见名称：

+   `.create()`: 创建一个新实例。示例：`Object.create()`

+   `.from()`: 创建一个基于不同对象的新实例，通过复制和/或转换它。示例：`Array.from()`

+   `.of()`: 通过使用参数指定的值创建一个新实例。示例：`Array.of()`

在下面的示例中，`DataContainer.create()`是一个静态工厂方法。它返回`DataContainer`的实例的 Promise：

```js
class DataContainer {
 #data;
 static async create() {
 const data = await Promise.resolve('downloaded');
 return new this(data);
 }
 constructor(data) {
 this.#data = data;
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}
DataContainer.create()
 .then(dc => assert.equal(
 dc.getData(), 'DATA: downloaded'));
```

这次，所有异步功能都包含在`.create()`中，这使得类的其余部分完全同步，因此更简单。

优缺点：

+   这种方法的一个好处是构造函数变得简单。

+   这种方法的一个缺点是现在可能会创建不正确设置的实例，通过`new DataContainer()`。

#### 13.3.1 改进：通过秘密令牌的私有构造函数

如果我们想要确保实例始终正确设置，我们必须确保只有`DataContainer.create()`可以调用`DataContainer`的构造函数。我们可以通过一个秘密令牌来实现：

```js
const secretToken = Symbol('secretToken');
class DataContainer {
 #data;
 static async create() {
 const data = await Promise.resolve('downloaded');
 return new this(secretToken, data);
 }
 constructor(token, data) {
 if (token !== secretToken) {
 throw new Error('Constructor is private');
 }
 this.#data = data;
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}
DataContainer.create()
 .then(dc => assert.equal(
 dc.getData(), 'DATA: downloaded'));
```

如果`secretToken`和`DataContainer`位于同一个模块中，并且只有后者被导出，那么外部方无法访问`secretToken`，因此无法创建`DataContainer`的实例。

优缺点：

+   优点：安全且直观。

+   缺点：稍微冗长。

#### 13.3.2 改进：构造函数抛出，工厂方法借用类原型

我们解决方案的以下变体禁用了`DataContainer`的构造函数，并使用一个技巧以另一种方式创建它的实例（行 A）：

```js
class DataContainer {
 static async create() {
 const data = await Promise.resolve('downloaded');
 return Object.create(this.prototype)._init(data); // (A)
 }
 constructor() {
 throw new Error('Constructor is private');
 }
 _init(data) {
 this._data = data;
 return this;
 }
 getData() {
 return 'DATA: '+this._data;
 }
}
DataContainer.create()
 .then(dc => {
 assert.equal(dc instanceof DataContainer, true); // (B)
 assert.equal(
 dc.getData(), 'DATA: downloaded');
 });
```

在内部，`DataContainer`的实例是其原型为`DataContainer.prototype`的任何对象。这就是为什么我们可以通过`Object.create()`（行 A）创建实例，也是为什么`instanceof`在行 B 中起作用的原因。

优缺点：

+   优点：优雅；`instanceof`有效。

+   缺点：

    +   无法完全阻止创建实例。不过公平地说，通过`Object.create()`的解决方案也可以用于我们之前的解决方案。

    +   我们不能在`DataContainer`中使用[私有字段](https://2ality.com/2019/07/private-class-fields.html)和[私有方法](https://2ality.com/2019/07/private-methods-accessors-in-classes.html)，因为这些只对通过构造函数创建的实例正确设置。

#### 13.3.3 改进：实例默认处于非活动状态，由工厂方法激活

另一种更冗长的变体是，默认情况下，实例通过标志`.#active`关闭。将它们打开的初始化方法`.#init()`不能从外部访问，但`Data.container()`可以调用它：

```js
class DataContainer {
 #data;
 static async create() {
 const data = await Promise.resolve('downloaded');
 return new this().#init(data);
 }

 #active = false;
 constructor() {
 }
 #init(data) {
 this.#active = true;
 this.#data = data;
 return this;
 }
 getData() {
 this.#check();
 return 'DATA: '+this.#data;
 }
 #check() {
 if (!this.#active) {
 throw new Error('Not created by factory');
 }
 }
}
DataContainer.create()
 .then(dc => assert.equal(
 dc.getData(), 'DATA: downloaded'));
```

标志`.#active`通过私有方法`.#check()`强制执行，必须在每个方法的开始处调用它。

这种解决方案的主要缺点是冗长。还有一个风险，就是在每个方法中忘记调用`.#check()`。

#### 13.3.4 变体：单独的工厂函数

为了完整起见，我将展示另一种变体：不使用静态方法作为工厂，您也可以使用一个单独的独立函数。

```js
const secretToken = Symbol('secretToken');
class DataContainer {
 #data;
 constructor(token, data) {
 if (token !== secretToken) {
 throw new Error('Constructor is private');
 }
 this.#data = data;
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}

async function createDataContainer() {
 const data = await Promise.resolve('downloaded');
 return new DataContainer(secretToken, data);
}

createDataContainer()
 .then(dc => assert.equal(
 dc.getData(), 'DATA: downloaded'));
```

独立函数作为工厂偶尔是有用的，但在这种情况下，我更喜欢静态方法：

+   独立函数无法访问`DataContainer`的私有成员。

+   我更喜欢`DataContainer.create()`的方式。

### 13.4 以 Promise 为基础的构造函数进行子类化（可选）

一般来说，子类化是一种需要谨慎使用的东西。

使用单独的工厂函数，相对容易扩展`DataContainer`。

然而，使用基于 Promise 的构造函数扩展类会导致严重的限制。在下面的示例中，我们对`DataContainer`进行子类化。子类`SubDataContainer`有自己的私有字段`.#moreData`，它通过连接到其超类的构造函数返回的 Promise 来异步初始化。

```js
class DataContainer {
 #data;
 constructor() {
 return Promise.resolve('downloaded')
 .then(data => {
 this.#data = data;
 return this; // (A)
 });
 }
 getData() {
 return 'DATA: '+this.#data;
 }
}

class SubDataContainer extends DataContainer {
 #moreData;
 constructor() {
 super();
 const promise = this;
 return promise
 .then(_this => {
 return Promise.resolve('more')
 .then(moreData => {
 _this.#moreData = moreData;
 return _this;
 });
 });
 }
 getData() {
 return super.getData() + ', ' + this.#moreData;
 }
}
```

哎呀，我们无法实例化这个类：

```js
assert.rejects(
 () => new SubDataContainer(),
 {
 name: 'TypeError',
 message: 'Cannot write private member #moreData ' +
 'to an object whose class did not declare it',
 }
);
```

为什么会失败？构造函数总是将其私有字段添加到其 `this` 中。然而，在这里，子构造函数中的 `this` 是由超级构造函数返回的 Promise（而不是通过 Promise 交付的 `SubDataContainer` 实例）。

然而，如果 `SubDataContainer` 没有任何私有字段，这种方法仍然有效。

### 13.5 结论

在本章研究的场景中，我更喜欢使用基于 Promise 的构造函数或静态工厂方法加上通过秘密令牌的私有构造函数。

然而，这里介绍的其他技术在其他场景中仍然可能有用。

### 13.6 进一步阅读

+   异步编程：

    +   《JavaScript 程序员的急切指南》中的“用于异步编程的 Promise”章节

    +   《JavaScript 程序员的急切指南》中的“异步函数”章节

    +   《JavaScript 程序员的急切指南》中的“立即调用的异步箭头函数”部分

+   面向对象编程：

    +   《JavaScript 程序员的急切指南》中的“原型链和类”章节

    +   “JavaScript 类中的私有字段的 ES 提案”博文

    +   “JavaScript 类中的私有方法和访问器的 ES 提案”博文

评论
