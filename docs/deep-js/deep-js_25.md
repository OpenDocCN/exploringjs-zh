# 十八、元编程与代理

> 原文：[`exploringjs.com/deep-js/ch_proxies.html`](https://exploringjs.com/deep-js/ch_proxies.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   18.1 概述

+   18.2 编程与元编程

    +   18.2.1 元编程的种类

+   18.3 代理解释

    +   18.3.1 一个例子

    +   18.3.2 特定函数陷阱

    +   18.3.3 拦截方法调用

    +   18.3.4 可撤销代理

    +   18.3.5 代理作为原型

    +   18.3.6 转发拦截操作

    +   18.3.7 陷阱：并非所有对象都可以被代理透明包装

+   18.4 代理的用例

    +   18.4.1 跟踪属性访问（`get`，`set`）

    +   18.4.2 关于未知属性的警告（`get`，`set`）

    +   18.4.3 负数组索引（`get`）

    +   18.4.4 数据绑定（`set`）

    +   18.4.5 访问 restful web 服务（方法调用）

    +   18.4.6 可撤销引用

    +   18.4.7 在 JavaScript 中实现 DOM

    +   18.4.8 更多用例

    +   18.4.9 使用代理的库

+   18.5 代理 API 的设计

    +   18.5.1 分层：保持基本级别和元级别分开

    +   18.5.2 虚拟对象与包装器

    +   18.5.3 透明虚拟化和处理程序封装

    +   18.5.4 元对象协议和代理陷阱

    +   18.5.5 强制代理的不变性

+   18.6 常见问题：代理

    +   18.6.1 `enumerate`陷阱在哪里？

+   18.7 参考：代理 API

    +   18.7.1 创建代理

    +   18.7.2 处理程序方法

    +   18.7.3 处理程序方法的不变性

    +   18.7.4 影响原型链的操作

    +   18.7.5 反射

+   18.8 结论

+   18.9 进一步阅读

* * *

### 18.1 概述

代理使我们能够拦截和定制对对象执行的操作（例如获取属性）。它们是一种*元编程*特性。

在以下示例中：

+   `proxy`是一个空对象。

+   通过实现特定方法，`handler`可以拦截对`proxy`执行的操作。

+   如果处理程序不拦截操作，则将其转发到`target`。

我们只拦截一个操作 - `get`（获取属性）：

```js
const logged = [];

const target = {size: 0};
const handler = {
 get(target, propKey, receiver) {
 logged.push('GET ' + propKey);
 return 123;
 }
};
const proxy = new Proxy(target, handler);
```

当我们获取属性`proxy.size`时，处理程序会拦截该操作：

```js
assert.equal(
 proxy.size, 123);

assert.deepEqual(
 logged, ['GET size']);
```

查看完整 API 的参考以获取可以拦截的操作列表。

### 18.2 编程与元编程

在我们深入了解代理是什么以及它们为何有用之前，我们首先需要了解什么是*元编程*。

在编程中，有不同的层次：

+   在*基础级别*（也称为：*应用级别*），代码处理用户输入。

+   在*元级别*，代码处理基础级别的代码。

基础和元级别可以是不同的语言。在下面的元程序中，元编程语言是 JavaScript，基础编程语言是 Java。

```js
const str = 'Hello' + '!'.repeat(3);
console.log('System.out.println("'+str+'")');
```

元编程可以采用不同的形式。在前面的示例中，我们已经将 Java 代码打印到控制台。让我们将 JavaScript 用作元编程语言和基础编程语言。这方面的经典示例是[`eval()`函数](https://exploringjs.com/impatient-js/ch_callables.html#eval)，它允许我们动态评估/编译 JavaScript 代码。在下面的交互中，我们使用它来评估表达式`5 + 2`。

```js
> eval('5 + 2')
7
```

其他 JavaScript 操作可能看起来不像元编程，但实际上是的，如果我们仔细看的话：

```js
// Base level
const obj = {
 hello() {
 console.log('Hello!');
 },
};

// Meta level
for (const key of Object.keys(obj)) {
 console.log(key);
}
```

程序在运行时检查其自身的结构。这看起来不像元编程，因为 JavaScript 中编程构造和数据结构之间的分离是模糊的。所有的`Object.*`方法都可以被视为元编程功能。

#### 18.2.1 元编程的种类

反射元编程意味着程序处理自身。[Kiczales 等人[2]](ch_proxies.html#further-reading-proxies)区分了三种反射元编程：

+   **内省：**我们对程序的结构具有只读访问权限。

+   **自修改：**我们可以改变那个结构。

+   **介入：**我们可以重新定义一些语言操作的语义。

让我们看一些例子。

**示例：内省。**`Object.keys()`执行内省（请参阅上一个示例）。

**示例：自修改。**以下函数`moveProperty`将属性从源移动到目标。它通过使用方括号运算符进行属性访问、赋值运算符和`delete`运算符来进行自修改。（在生产代码中，我们可能会使用属性描述符来完成此任务。）

```js
function moveProperty(source, propertyName, target) {
 target[propertyName] = source[propertyName];
 delete source[propertyName];
}
```

`moveProperty()`的使用方法如下：

```js
const obj1 = { color: 'blue' };
const obj2 = {};

moveProperty(obj1, 'color', obj2);

assert.deepEqual(
 obj1, {});

assert.deepEqual(
 obj2, { color: 'blue' });
```

ECMAScript 5 不支持介入；代理被创建来填补这一空白。

### 18.3 代理解释

代理将介入 JavaScript。它们的工作原理如下。我们可以对对象`obj`执行许多操作，例如：

+   获取对象`obj`的属性`prop`（`obj.prop`）

+   检查对象`obj`是否具有属性`prop`（`'prop' in obj`）

代理是特殊的对象，允许我们定制其中一些操作。代理是由两个参数创建的：

+   `handler`：对于每个操作，都有一个相应的处理程序方法，如果存在，就执行该操作。这样的方法*拦截*了操作（在其传递到目标的途中），并被称为*陷阱*——这个术语是从操作系统的领域借来的。

+   `target`：如果处理程序不拦截操作，那么它将在目标上执行。也就是说，它充当处理程序的后备。在某种程度上，代理包装了目标。

注意：“介入”的动词形式是“介入”。介入是双向的。拦截是单向的。

#### 18.3.1 一个示例

在下面的示例中，处理程序拦截了`get`和`has`操作。

```js
const logged = [];

const target = {};
const handler = {
 /** Intercepts: getting properties */
 get(target, propKey, receiver) {
 logged.push(`GET ${propKey}`);
 return 123;
 },

 /** Intercepts: checking whether properties exist */
 has(target, propKey) {
 logged.push(`HAS ${propKey}`);
 return true;
 }
};
const proxy = new Proxy(target, handler);
```

如果我们获取属性（行 A）或使用`in`运算符（行 B），处理程序将拦截这些操作：

```js
assert.equal(proxy.age, 123); // (A)
assert.equal('hello' in proxy, true); // (B)

assert.deepEqual(
 logged, [
 'GET age',
 'HAS hello',
 ]);
```

处理程序没有实现`set`陷阱（设置属性）。因此，设置`proxy.age`会被转发到`target`，并导致设置`target.age`：

```js
proxy.age = 99;
assert.equal(target.age, 99);
```

#### 18.3.2 特定于函数的陷阱

如果目标是一个函数，可以拦截两个额外的操作：

+   `apply`：进行函数调用。通过以下方式触发：

    +   `proxy(···)`

    +   `proxy.call(···)`

    +   `proxy.apply(···)`

+   `construct`：进行构造函数调用。通过以下方式触发：

    +   `new proxy(···)`

之所以只为函数目标启用这些陷阱的原因很简单：否则，我们将无法转发操作`apply`和`construct`。

#### 18.3.3 拦截方法调用

如果我们想通过代理拦截方法调用，我们面临一个挑战：没有方法调用的陷阱。相反，方法调用被视为两个操作的序列：

+   `get`检索函数

+   一个`apply`来调用那个函数

因此，如果我们想要拦截方法调用，我们需要拦截两个操作：

+   首先，我们拦截`get`并返回一个函数。

+   其次，我们拦截该函数的调用。

以下代码演示了如何实现这一点。

```js
const traced = [];

function traceMethodCalls(obj) {
 const handler = {
 get(target, propKey, receiver) {
 const origMethod = target[propKey];
 return function (...args) { // implicit parameter `this`!
 const result = origMethod.apply(this, args);
 traced.push(propKey + JSON.stringify(args)
 + ' -> ' + JSON.stringify(result));
 return result;
 };
 }
 };
 return new Proxy(obj, handler);
}
```

我们并没有使用代理进行第二次拦截；我们只是将原始方法包装在一个函数中。

让我们使用以下对象来尝试`traceMethodCalls()`：

```js
const obj = {
 multiply(x, y) {
 return x * y;
 },
 squared(x) {
 return this.multiply(x, x);
 },
};

const tracedObj = traceMethodCalls(obj);
assert.equal(
 tracedObj.squared(9), 81);

assert.deepEqual(
 traced, [
 'multiply[9,9] -> 81',
 'squared[9] -> 81',
 ]);
```

甚至在`obj.squared()`内部调用`this.multiply()`也会被追踪！这是因为`this`一直指向代理。

这不是最有效的解决方案。例如，可以缓存方法。此外，代理本身会影响性能。

#### 18.3.4 可撤销的代理

代理可以被*撤销*（关闭）：

```js
const {proxy, revoke} = Proxy.revocable(target, handler);
```

第一次调用函数`revoke`后，我们对`proxy`应用的任何操作都会导致`TypeError`。后续调用`revoke`不会产生进一步的影响。

```js
const target = {}; // Start with an empty object
const handler = {}; // Don’t intercept anything
const {proxy, revoke} = Proxy.revocable(target, handler);

// `proxy` works as if it were the object `target`:
proxy.city = 'Paris';
assert.equal(proxy.city, 'Paris');

revoke();

assert.throws(
 () => proxy.prop,
 /^TypeError: Cannot perform 'get' on a proxy that has been revoked$/
);
```

#### 18.3.5 代理作为原型

代理`proto`可以成为对象`obj`的原型。在`obj`中开始的一些操作可能会在`proto`中继续。其中一个操作是`get`。

```js
const proto = new Proxy({}, {
 get(target, propertyKey, receiver) {
 console.log('GET '+propertyKey);
 return target[propertyKey];
 }
});

const obj = Object.create(proto);
obj.weight;

// Output:
// 'GET weight'
```

在`obj`中找不到`weight`属性，因此搜索会继续在`proto`中，并在那里触发`get`陷阱。还有更多影响原型的操作；它们在本章末尾列出。

#### 18.3.6 转发拦截的操作

处理程序没有实现的操作的陷阱会自动转发到目标。有时，除了转发操作之外，我们还想执行一些任务。例如，拦截和记录所有操作，而不阻止它们到达目标：

```js
const handler = {
 deleteProperty(target, propKey) {
 console.log('DELETE ' + propKey);
 return delete target[propKey];
 },
 has(target, propKey) {
 console.log('HAS ' + propKey);
 return propKey in target;
 },
 // Other traps: similar
}
```

##### 18.3.6.1 改进：使用`Reflect.*`

对于每个陷阱，我们首先记录操作的名称，然后通过手动执行它来转发它。JavaScript 有一个类似模块的对象`Reflect`，可以帮助转发。

对于每个陷阱：

```js
handler.trap(target, arg_1, ···, arg_n)
```

`Reflect`有一个方法：

```js
Reflect.trap(target, arg_1, ···, arg_n)
```

如果我们使用`Reflect`，前面的示例如下。

```js
const handler = {
 deleteProperty(target, propKey) {
 console.log('DELETE ' + propKey);
 return Reflect.deleteProperty(target, propKey);
 },
 has(target, propKey) {
 console.log('HAS ' + propKey);
 return Reflect.has(target, propKey);
 },
 // Other traps: similar
}
```

##### 18.3.6.2 改进：使用代理实现处理程序

现在每个陷阱的作用如此相似，以至于我们可以通过代理来实现处理程序：

```js
const handler = new Proxy({}, {
 get(target, trapName, receiver) {
 // Return the handler method named trapName
 return (...args) => {
 console.log(trapName.toUpperCase() + ' ' + args[1]);
 // Forward the operation
 return ReflecttrapName;
 };
 },
});
```

对于每个陷阱，代理通过`get`操作请求处理程序方法，我们给出一个。也就是说，所有处理程序方法都可以通过单个元方法`get`来实现。代理 API 的目标之一是使这种虚拟化变得简单。

让我们使用基于代理的处理程序：

```js
const target = {};
const proxy = new Proxy(target, handler);

proxy.distance = 450; // set
assert.equal(proxy.distance, 450); // get

// Was `set` operation correctly forwarded to `target`?
assert.equal(
 target.distance, 450);

// Output:
// 'SET distance'
// 'GETOWNPROPERTYDESCRIPTOR distance'
// 'DEFINEPROPERTY distance'
// 'GET distance'
```

#### 18.3.7 陷阱：并非所有对象都可以被代理透明包装

代理对象可以被视为拦截对其目标对象执行的操作 - 代理包装目标。代理的处理程序对象就像代理的观察者或监听器。它通过实现相应的方法（`get`用于读取属性等）指定应拦截哪些操作。如果操作的处理程序方法丢失，则该操作不会被拦截。它会被简单地转发到目标。

因此，如果处理程序是空对象，则代理应该透明地包装目标。然而，这并不总是有效。

##### 18.3.7.1 包装对象会影响`this`

在深入研究之前，让我们快速回顾一下包装目标如何影响`this`：

```js
const target = {
 myMethod() {
 return {
 thisIsTarget: this === target,
 thisIsProxy: this === proxy,
 };
 }
};
const handler = {};
const proxy = new Proxy(target, handler);
```

如果我们直接调用`target.myMethod()`，`this`指向`target`：

```js
assert.deepEqual(
 target.myMethod(), {
 thisIsTarget: true,
 thisIsProxy: false,
 });
```

如果我们通过代理调用该方法，`this`指向`proxy`：

```js
assert.deepEqual(
 proxy.myMethod(), {
 thisIsTarget: false,
 thisIsProxy: true,
 });
```

也就是说，如果代理将方法调用转发到目标，`this`不会改变。因此，如果目标使用`this`（例如，进行方法调用），代理将继续循环。

##### 18.3.7.2 无法透明包装的对象

通常，具有空处理程序的代理会透明地包装目标：我们不会注意到它们的存在，它们也不会改变目标的行为。

然而，如果目标通过代理无法控制的机制与`this`关联信息，我们会遇到问题：事情会失败，因为根据目标是否被包装，关联不同的信息。

例如，以下`Person`类将私有信息存储在 WeakMap`_name`中（有关此技术的更多信息，请参见[*JavaScript for impatient programmers*](https://exploringjs.com/impatient-js/ch_weakmaps.html#private-data-in-weakmaps)）：

```js
const _name = new WeakMap();
class Person {
 constructor(name) {
 _name.set(this, name);
 }
 get name() {
 return _name.get(this);
 }
}
```

`Person`的实例无法被透明地包装：

```js
const jane = new Person('Jane');
assert.equal(jane.name, 'Jane');

const proxy = new Proxy(jane, {});
assert.equal(proxy.name, undefined);
```

`jane.name`与包装的`proxy.name`不同。以下实现没有这个问题：

```js
class Person2 {
 constructor(name) {
 this._name = name;
 }
 get name() {
 return this._name;
 }
}

const jane = new Person2('Jane');
assert.equal(jane.name, 'Jane');

const proxy = new Proxy(jane, {});
assert.equal(proxy.name, 'Jane');
```

##### 18.3.7.3 包装内置构造函数的实例

大多数内置构造函数的实例也使用代理无法拦截的机制。因此，它们也无法被透明地包装。如果我们使用`Date`的一个实例，我们可以看到：

```js
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

assert.throws(
 () => proxy.getFullYear(),
 /^TypeError: this is not a Date object\.$/
);
```

代理不受影响的机制称为*内部槽*。这些槽是与实例关联的类似属性的存储。规范将这些槽处理为具有方括号名称的属性。例如，以下方法是内部的，可以在所有对象`O`上调用：

```js
O.[[GetPrototypeOf]]()
```

与属性不同，访问内部槽不是通过正常的“获取”和“设置”操作完成的。如果通过代理调用`.getFullYear()`，它无法在`this`上找到它需要的内部槽，并通过`TypeError`进行投诉。

对于`Date`方法，[语言规范规定](https://tc39.es/ecma262/#sec-properties-of-the-date-prototype-object)：

> 除非另有规定，下面定义的 Date 原型对象的方法不是通用的，传递给它们的`this`值必须是已初始化为时间值的具有`[[DateValue]]`内部槽的对象。

##### 18.3.7.4 解决方法

作为解决方法，我们可以改变处理程序如何转发方法调用，并有选择地将`this`设置为目标而不是代理：

```js
const handler = {
 get(target, propKey, receiver) {
 if (propKey === 'getFullYear') {
 return target.getFullYear.bind(target);
 }
 return Reflect.get(target, propKey, receiver);
 },
};
const proxy = new Proxy(new Date('2030-12-24'), handler);
assert.equal(proxy.getFullYear(), 2030);
```

这种方法的缺点是，方法在`this`上执行的所有操作都不会通过代理。

##### 18.3.7.5 数组可以被透明地包装

与其他内置对象不同，数组可以被透明地包装：

```js
const p = new Proxy(new Array(), {});

p.push('a');
assert.equal(p.length, 1);

p.length = 0;
assert.equal(p.length, 0);
```

数组可包装的原因是，即使属性访问被定制以使`.length`工作，数组方法不依赖于内部槽 - 它们是通用的。

### 18.4 代理的用例

本节演示了代理可以用于什么。这将使我们有机会看到 API 的实际应用。

#### 18.4.1 跟踪属性访问（`get`，`set`）

假设我们有一个函数`tracePropertyAccesses(obj, propKeys)`，每当`obj`的属性被设置或获取时，它都会记录下来，其键在数组`propKeys`中。在下面的代码中，我们将该函数应用于`Point`类的一个实例：

```js
class Point {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
 toString() {
 return `Point(${this.x}, ${this.y})`;
 }
}

// Trace accesses to properties `x` and `y`
const point = new Point(5, 7);
const tracedPoint = tracePropertyAccesses(point, ['x', 'y']);
```

获取和设置被跟踪对象`p`的属性具有以下效果：

```js
assert.equal(tracedPoint.x, 5);
tracedPoint.x = 21;

// Output:
// 'GET x'
// 'SET x=21'
```

有趣的是，当`Point`访问属性时，跟踪也会起作用，因为此时`this`指的是被跟踪的对象，而不是`Point`的一个实例：

```js
assert.equal(
 tracedPoint.toString(),
 'Point(21, 7)');

// Output:
// 'GET x'
// 'GET y'
```

##### 18.4.1.1 不使用代理实现`tracePropertyAccesses()`

如果没有代理，我们将如下实现`tracePropertyAccesses()`。我们用一个 getter 和一个 setter 替换每个属性来跟踪访问。这些 setter 和 getter 使用额外的对象`propData`来存储属性的数据。请注意，我们正在破坏性地改变原始实现，这意味着我们正在元编程。

```js
function tracePropertyAccesses(obj, propKeys, log=console.log) {
 // Store the property data here
 const propData = Object.create(null);

 // Replace each property with a getter and a setter
 propKeys.forEach(function (propKey) {
 propData[propKey] = obj[propKey];
 Object.defineProperty(obj, propKey, {
 get: function () {
 log('GET '+propKey);
 return propData[propKey];
 },
 set: function (value) {
 log('SET '+propKey+'='+value);
 propData[propKey] = value;
 },
 });
 });
 return obj;
}
```

参数`log`使得对这个函数进行单元测试更容易：

```js
const obj = {};
const logged = [];
tracePropertyAccesses(obj, ['a', 'b'], x => logged.push(x));

obj.a = 1;
assert.equal(obj.a, 1);

obj.c = 3;
assert.equal(obj.c, 3);

assert.deepEqual(
 logged, [
 'SET a=1',
 'GET a',
 ]);
```

##### 18.4.1.2 使用代理实现`tracePropertyAccesses()`

代理给了我们一个更简单的解决方案。我们拦截属性的获取和设置，不需要改变实现。

```js
function tracePropertyAccesses(obj, propKeys, log=console.log) {
 const propKeySet = new Set(propKeys);
 return new Proxy(obj, {
 get(target, propKey, receiver) {
 if (propKeySet.has(propKey)) {
 log('GET '+propKey);
 }
 return Reflect.get(target, propKey, receiver);
 },
 set(target, propKey, value, receiver) {
 if (propKeySet.has(propKey)) {
 log('SET '+propKey+'='+value);
 }
 return Reflect.set(target, propKey, value, receiver);
 },
 });
}
```

#### 18.4.2 关于未知属性的警告（`get`，`set`）

在访问属性方面，JavaScript 非常宽容。例如，如果我们尝试读取一个属性并拼错它的名称，我们不会得到异常 - 我们会得到结果`undefined`。

我们可以使用代理在这种情况下得到一个异常。工作原理如下。我们将代理作为对象的原型。如果在对象中找不到属性，则会触发代理的`get`陷阱：

+   如果在代理之后的原型链中甚至不存在属性，则确实缺少该属性，我们会抛出异常。

+   否则，我们返回继承属性的值。我们通过将`get`操作转发到目标（代理从目标获取其原型）来这样做。

这是这种方法的一个实现：

```js
const propertyCheckerHandler = {
 get(target, propKey, receiver) {
 // Only check string property keys
 if (typeof propKey === 'string' && !(propKey in target)) {
 throw new ReferenceError('Unknown property: ' + propKey);
 }
 return Reflect.get(target, propKey, receiver);
 }
};
const PropertyChecker = new Proxy({}, propertyCheckerHandler);
```

让我们为一个对象使用`PropertyChecker`：

```js
const jane = {
 __proto__: PropertyChecker,
 name: 'Jane',
};

// Own property:
assert.equal(
 jane.name,
 'Jane');

// Typo:
assert.throws(
 () => jane.nmae,
 /^ReferenceError: Unknown property: nmae$/);

// Inherited property:
assert.equal(
 jane.toString(),
 '[object Object]');
```

##### 18.4.2.1 `PropertyChecker`作为一个类

如果我们将`PropertyChecker`转换为构造函数，我们可以通过`extends`在类中使用它。

```js
// We can’t change .prototype of classes, so we are using a function
function PropertyChecker2() {}
PropertyChecker2.prototype = new Proxy({}, propertyCheckerHandler);

class Point extends PropertyChecker2 {
 constructor(x, y) {
 super();
 this.x = x;
 this.y = y;
 }
}

const point = new Point(5, 7);
assert.equal(point.x, 5);
assert.throws(
 () => point.z,
 /^ReferenceError: Unknown property: z/);
```

这是`point`的原型链：

```js
const p = Object.getPrototypeOf.bind(Object);
assert.equal(p(point), Point.prototype);
assert.equal(p(p(point)), PropertyChecker2.prototype);
assert.equal(p(p(p(point))), Object.prototype);
```

##### 18.4.2.2 防止意外创建属性

如果我们担心意外*创建*属性，我们有两个选择：

+   我们可以将代理包装在捕获`set`的对象周围。

+   或者我们可以通过`Object.preventExtensions(obj)`使对象`obj`不可扩展，这意味着 JavaScript 不允许我们向`obj`添加新的（自有）属性。 

#### 18.4.3 负数组索引（`get`）

一些数组方法允许我们通过`-1`引用最后一个元素，通过`-2`引用倒数第二个元素，依此类推。例如：

```js
> ['a', 'b', 'c'].slice(-1)
[ 'c' ]
```

然而，当通过括号运算符（`[]`）访问元素时，这种方法不起作用。但是，我们可以使用代理来添加这种功能。以下函数`createArray()`创建支持负索引的数组。它通过在数组实例周围包装代理来实现。代理拦截了由括号运算符触发的`get`操作。

```js
function createArray(...elements) {
 const handler = {
 get(target, propKey, receiver) {
 if (typeof propKey === 'string') {
 const index = Number(propKey);
 if (index < 0) {
 propKey = String(target.length + index);
 }
 }
 return Reflect.get(target, propKey, receiver);
 }
 };
 // Wrap a proxy around the Array
 return new Proxy(elements, handler);
}
const arr = createArray('a', 'b', 'c');
assert.equal(
 arr[-1], 'c');
assert.equal(
 arr[0], 'a');
assert.equal(
 arr.length, 3);
```

#### 18.4.4 数据绑定（`set`）

数据绑定是关于在对象之间同步数据的。一个常见的用例是基于 MVC（模型视图控制器）模式的小部件：通过数据绑定，*视图*（小部件）会保持最新状态，如果我们改变*模型*（小部件可视化的数据）。

为了实现数据绑定，我们必须观察并对对象所做的更改做出反应。以下代码片段是对如何观察数组的更改进行工作的草图。

```js
function createObservedArray(callback) {
 const array = [];
 return new Proxy(array, {
 set(target, propertyKey, value, receiver) {
 callback(propertyKey, value);
 return Reflect.set(target, propertyKey, value, receiver);
 }
 });
}
const observedArray = createObservedArray(
 (key, value) => console.log(
 `${JSON.stringify(key)} = ${JSON.stringify(value)}`));
observedArray.push('a');

// Output:
// '"0" = "a"'
// '"length" = 1'
```

#### 18.4.5 访问 restful web 服务（方法调用）

代理可以用来创建一个可以调用任意方法的对象。在以下示例中，函数`createWebService()`创建了一个这样的对象`service`。在`service`上调用方法会检索具有相同名称的 web 服务资源的内容。检索是通过 Promise 处理的。

```js
const service = createWebService('http://example.com/data');
// Read JSON data in http://example.com/data/employees
service.employees().then((jsonStr) => {
 const employees = JSON.parse(jsonStr);
 // ···
});
```

以下代码是`createWebService`的一个快速而粗糙的实现，没有代理。我们需要事先知道在`service`上将调用哪些方法。参数`propKeys`提供了这些信息；它保存了一个包含方法名称的数组。

```js
function createWebService(baseUrl, propKeys) {
 const service = {};
 for (const propKey of propKeys) {
 service[propKey] = () => {
 return httpGet(baseUrl + '/' + propKey);
 };
 }
 return service;
}
```

使用代理，`createWebService()`更简单：

```js
function createWebService(baseUrl) {
 return new Proxy({}, {
 get(target, propKey, receiver) {
 // Return the method to be called
 return () => httpGet(baseUrl + '/' + propKey);
 }
 });
}
```

这两种实现都使用以下函数来进行 HTTP GET 请求（其工作原理在[*JavaScript for impatient programmers*](https://exploringjs.com/impatient-js/ch_promises.html#promisifying-xmlhttprequest)中有解释）。

```js
function httpGet(url) {
 return new Promise(
 (resolve, reject) => {
 const xhr = new XMLHttpRequest();
 xhr.onload = () => {
 if (xhr.status === 200) {
 resolve(xhr.responseText); // (A)
 } else {
 // Something went wrong (404, etc.)
 reject(new Error(xhr.statusText)); // (B)
 }
 }
 xhr.onerror = () => {
 reject(new Error('Network error')); // (C)
 };
 xhr.open('GET', url);
 xhr.send();
 });
}
```

#### 18.4.6 可撤销的引用

*可撤销的引用*的工作原理如下：客户端不允许直接访问重要资源（对象），只能通过引用（中间对象，资源的包装器）访问。通常，对引用应用的每个操作都会转发到资源。客户端完成后，通过*撤销*引用来保护资源，关闭它。此后，对引用应用操作会抛出异常，不再转发。

在以下示例中，我们为一个资源创建了一个可撤销的引用。然后，我们通过引用读取了资源的一个属性。这是有效的，因为引用授予了我们访问权限。接下来，我们撤销了引用。现在引用不再让我们读取属性。

```js
const resource = { x: 11, y: 8 };
const {reference, revoke} = createRevocableReference(resource);

// Access granted
assert.equal(reference.x, 11);

revoke();

// Access denied
assert.throws(
 () => reference.x,
 /^TypeError: Cannot perform 'get' on a proxy that has been revoked/
);
```

代理非常适合实现可撤销的引用，因为它们可以拦截和转发操作。这是一个基于代理的`createRevocableReference`的简单实现：

```js
function createRevocableReference(target) {
 let enabled = true;
 return {
 reference: new Proxy(target, {
 get(target, propKey, receiver) {
 if (!enabled) {
 throw new TypeError(
 `Cannot perform 'get' on a proxy that has been revoked`);
 }
 return Reflect.get(target, propKey, receiver);
 },
 has(target, propKey) {
 if (!enabled) {
 throw new TypeError(
 `Cannot perform 'has' on a proxy that has been revoked`);
 }
 return Reflect.has(target, propKey);
 },
 // (Remaining methods omitted)
 }),
 revoke: () => {
 enabled = false;
 },
 };
}
```

通过上一节的代理作为处理程序技术，可以简化代码。这一次，处理程序基本上是`Reflect`对象。因此，`get`陷阱通常返回适当的`Reflect`方法。如果引用已被撤销，则会抛出`TypeError`。

```js
function createRevocableReference(target) {
 let enabled = true;
 const handler = new Proxy({}, {
 get(_handlerTarget, trapName, receiver) {
 if (!enabled) {
 throw new TypeError(
 `Cannot perform '${trapName}' on a proxy`
 + ` that has been revoked`);
 }
 return Reflect[trapName];
 }
 });
 return {
 reference: new Proxy(target, handler),
 revoke: () => {
 enabled = false;
 },
 };
}
```

但是，我们不必自己实现可撤销的引用，因为代理可以被撤销。这一次，撤销发生在代理中，而不是在处理程序中。处理程序所要做的就是将每个操作转发到目标。正如我们已经看到的，如果处理程序没有实现任何陷阱，那么这将自动发生。

```js
function createRevocableReference(target) {
 const handler = {}; // forward everything
 const { proxy, revoke } = Proxy.revocable(target, handler);
 return { reference: proxy, revoke };
}
```

##### 18.4.6.1 膜

膜基于可撤销引用的想法构建：用于安全运行不受信任代码的库在该代码周围包装一个膜，以隔离它并保持系统的其余部分安全。对象在两个方向上通过膜传递：

+   不受信任的代码可能会从外部接收对象（“干燥对象”）。

+   或者它可能将对象（“湿对象”）交给外部。

在这两种情况下，可撤销的引用被包装在对象周围。由包装函数或方法返回的对象也被包装。此外，如果将包装的对象传回膜中，则会被解包。

一旦不受信任的代码完成，所有可撤销的引用都被撤销。因此，外部的代码将不再被执行，它引用的外部对象也将停止工作。Caja 编译器是“用于使第三方 HTML、CSS 和 JavaScript 安全嵌入到您的网站中的工具”。它使用膜来实现这一目标。

#### 18.4.7 在 JavaScript 中实现 DOM

浏览器的文档对象模型（DOM）通常是由 JavaScript 和 C++混合实现的。在纯 JavaScript 中实现它对于以下情况很有用：

+   模拟浏览器环境，例如在 Node.js 中操作 HTML。jsdom 是一个可以实现这一功能的库。

+   加快 DOM 的速度（在 JavaScript 和 C++之间切换需要时间）。

然而，标准的 DOM 可以做一些在 JavaScript 中不容易复制的事情。例如，大多数 DOM 集合都是对 DOM 当前状态的动态更改的实时视图。因此，纯 JavaScript 实现的 DOM 并不是非常高效的。向 JavaScript 添加代理的原因之一是为了实现更高效的 DOM。

#### 18.4.8 更多用例

代理还有更多的用例。例如：

+   远程：本地占位符对象将方法调用转发到远程对象。这个用例类似于 Web 服务的例子。

+   数据库的数据访问对象：读取和写入对象会读取和写入数据库。这个用例类似于 Web 服务的例子。

+   分析：拦截方法调用以跟踪每个方法花费的时间。这个用例类似于跟踪的例子。

#### 18.4.9 使用代理的库

+   Immer（由 Michel Weststrate）有助于非破坏性地更新数据。应用的更改是通过调用方法、设置属性、设置数组元素等来指定的。草案状态是通过代理实现的。

+   MobX 让您观察数据结构（如对象、数组和类实例）的更改。这是通过代理实现的。

+   Alpine.js（由 Caleb Porzio）是一个前端库，通过代理实现数据绑定。

+   on-change（由 Sindre Sorhus）观察对象的更改（通过代理）并报告它们。

+   [*Env utility*（由 Nicholas C. Zakas）](https://github.com/humanwhocodes/env)允许您通过属性访问环境变量，并在它们不存在时抛出异常。这是通过代理实现的。

+   [*LDflex*（由 Ruben Verborgh 和 Ruben Taelman）](https://github.com/LDflex/LDflex)提供了一个用于链接数据（考虑语义网络）的查询语言。流畅的查询 API 是通过代理实现的。

### 18.5 代理 API 的设计

在本节中，我们将更深入地了解代理的工作原理以及为什么它们以这种方式工作。

#### 18.5.1 分层：保持基本级别和元级别分开

Firefox 曾经支持一种有限的元编程形式：如果对象`O`有一个名为`__noSuchMethod__`的方法，那么每当在`O`上调用一个不存在的方法时，它都会被通知。以下代码演示了它是如何工作的：

```js
const calc = {
 __noSuchMethod__: function (methodName, args) {
 switch (methodName) {
 case 'plus':
 return args.reduce((a, b) => a + b);
 case 'times':
 return args.reduce((a, b) => a * b);
 default:
 throw new TypeError('Unsupported: ' + methodName);
 }
 }
};

// All of the following method calls are implemented via
// .__noSuchMethod__().
assert.equal(
 calc.plus(3, 5, 2), 10);
assert.equal(
 calc.times(2, 3, 4), 24);

assert.equal(
 calc.plus('Parts', ' of ', 'a', ' string'),
 'Parts of a string');
```

因此，`__noSuchMethod__`的工作方式类似于代理陷阱。与代理相反，陷阱是我们想要拦截其操作的对象的自有或继承方法。这种方法的问题在于基本级别（普通方法）和元级别（`__noSuchMethod__`）混合在一起。基本级别的代码可能会意外调用或看到元级别的方法，并且可能会意外定义一个元级别的方法。

即使在标准的 ECMAScript 中，基本级别和元级别有时会混合在一起。例如，以下元编程机制可能会失败，因为它们存在于基本级别：

+   `obj.hasOwnProperty(propKey)`: 如果原型链中的属性覆盖了内置实现，则此调用可能会失败。例如，在以下代码中，`obj`会导致失败：

    ```js
    const obj = { hasOwnProperty: null };
    assert.throws(
     () => obj.hasOwnProperty('width'),
     /^TypeError: obj.hasOwnProperty is not a function/
    );
    ```

    这些是调用`.hasOwnProperty()`的安全方式：

    ```js
    assert.equal(
     Object.prototype.hasOwnProperty.call(obj, 'width'), false);

    // Abbreviated version:
    assert.equal(
     {}.hasOwnProperty.call(obj, 'width'), false);
    ```

+   `func.call(···)`, `func.apply(···)`: 对于这两种方法，问题和解决方案与`.hasOwnProperty()`相同。

+   `obj.__proto__`: 在普通对象中，`__proto__`是一个特殊属性，它允许我们获取和设置接收者的原型。因此，当我们将普通对象用作字典时，我们必须[避免将`__proto__`作为属性键](https://exploringjs.com/impatient-js/ch_single-objects.html#the-pitfalls-of-using-an-object-as-a-dictionary)。

到目前为止，应该很明显，使（基本级别）属性键特殊是有问题的。因此，代理是*分层*的：基本级别（代理对象）和元级别（处理程序对象）是分开的。

#### 18.5.2 虚拟对象与包装器

代理有两种角色：

+   作为*包装器*，它们*包装*它们的目标，控制对它们的访问。包装器的示例包括：可撤销资源和通过代理进行跟踪。

+   作为*虚拟对象*，它们只是具有特殊行为的对象，它们的目标并不重要。一个例子是代理，它将方法调用转发到远程对象。

代理 API 的早期设计将代理视为纯粹的虚拟对象。然而，事实证明，即使在这种角色中，目标也是有用的，用于强制执行不变量（稍后解释）并作为处理程序没有实现的陷阱的后备。

#### 18.5.3 透明虚拟化和处理程序封装

代理有两种方式进行屏蔽：

+   无法确定对象是否是代理（*透明虚拟化*）。

+   我们无法通过其代理访问处理程序（*处理程序封装*）。

这两个原则赋予了代理模式相当大的权力，可以模拟其他对象。强制执行*不变量*（稍后解释）的一个原因是为了控制这种权力。

如果我们确实需要一种方法来区分代理和非代理，我们必须自己实现。以下代码是一个模块`lib.mjs`，它导出了两个函数：一个用于创建代理，另一个用于确定对象是否是这些代理之一。

```js
// lib.mjs
const proxies = new WeakSet();

export function createProxy(obj) {
 const handler = {};
 const proxy = new Proxy(obj, handler);
 proxies.add(proxy);
 return proxy;
}

export function isProxy(obj) {
 return proxies.has(obj);
}
```

该模块使用数据结构`WeakSet`来跟踪代理。`WeakSet`非常适合这个目的，因为它不会阻止其元素被垃圾回收。

下一个示例展示了如何使用`lib.mjs`。

```js
// main.mjs
import { createProxy, isProxy } from './lib.mjs';

const proxy = createProxy({});
assert.equal(isProxy(proxy), true);
assert.equal(isProxy({}), false);
```

#### 18.5.4 元对象协议和代理陷阱

在本节中，我们将研究 JavaScript 的内部结构以及选择 Proxy 陷阱集的方式。

在编程语言和 API 设计的上下文中，*协议*是一组接口加上使用它们的规则。ECMAScript 规范描述了如何执行 JavaScript 代码。它包括一个[处理对象的协议](https://tc39.es/ecma262/#sec-ordinary-and-exotic-objects-behaviours)。这个协议在元级别上运行，有时被称为*元对象协议*（MOP）。JavaScript MOP 由所有对象都具有的内部方法组成。 “内部”意味着它们只存在于规范中（JavaScript 引擎可能有也可能没有），并且无法从 JavaScript 访问。内部方法的名称用双方括号写成。

获取属性的内部方法称为[`.[[Get]]()`](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots-get-p-receiver)。如果我们使用双下划线而不是双方括号，这个方法在 JavaScript 中大致实现如下。

```js
// Method definition
__Get__(propKey, receiver) {
 const desc = this.__GetOwnProperty__(propKey);
 if (desc === undefined) {
 const parent = this.__GetPrototypeOf__();
 if (parent === null) return undefined;
 return parent.__Get__(propKey, receiver); // (A)
 }
 if ('value' in desc) {
 return desc.value;
 }
 const getter = desc.get;
 if (getter === undefined) return undefined;
 return getter.__Call__(receiver, []);
}
```

在这段代码中调用的 MOP 方法有：

+   `[[GetOwnProperty]]`（陷阱`getOwnPropertyDescriptor`）

+   `[[GetPrototypeOf]]`（陷阱`getPrototypeOf`）

+   `[[Get]]`（陷阱`get`）

+   `[[Call]]`（陷阱`apply`）

在 A 行中，我们可以看到原型链中的代理是如何找到`get`的，如果在“早期”对象中找不到属性：如果没有键为`propKey`的自有属性，则搜索将继续在`this`的原型`parent`中进行。

**基本与派生操作。**我们可以看到`.[[Get]]()`调用其他 MOP 操作。这样做的操作称为*派生*。不依赖其他操作的操作称为*基本*。

##### 18.5.4.1 代理的元对象协议

[代理的元对象协议](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)与普通对象的不同。对于普通对象，派生操作调用其他操作。对于代理，每个操作（无论是基本还是派生）都会被处理程序方法拦截或转发到目标。

哪些操作应该通过代理进行拦截？

+   一种可能性是只为基本操作提供陷阱。

+   另一种选择是包括一些派生操作。

这样做的好处是可以提高性能并更加方便。例如，如果没有`get`的陷阱，我们将不得不通过`getOwnPropertyDescriptor`来实现其功能。

包括派生陷阱的一个缺点是可能导致代理行为不一致。例如，`get`可能返回与`getOwnPropertyDescriptor`返回的描述符中的值不同的值。

##### 18.5.4.2 选择性拦截：哪些操作应该是可拦截的？

代理的拦截是*选择性*的：我们无法拦截每个语言操作。为什么有些操作被排除在外？让我们看两个原因。

首先，*稳定*操作不太适合拦截。如果一个操作总是对相同的参数产生相同的结果，则该操作是*稳定*的。如果代理可以拦截稳定操作，它可能会变得不稳定，因此不可靠。[严格相等](http://speakingjs.com/es5/ch09.html#_strict_equality)（`===`）就是这样一个稳定操作。它无法被拦截，其结果是通过将代理本身视为另一个对象来计算的。另一种保持稳定性的方法是将操作应用于目标而不是代理。稍后将在我们看如何对代理执行不变性时解释，当`Object.getPrototypeOf()`应用于目标不可扩展的代理时会发生这种情况。

不进行更多操作的拦截的另一个原因是，拦截意味着在通常不可能的情况下执行自定义代码。代码的交错发生越多，理解和调试程序就越困难。它还会对性能产生负面影响。

##### 18.5.4.3 陷阱：`get`与`invoke`

如果我们想通过代理创建虚拟方法，我们必须从`get`陷阱中返回函数。这引发了一个问题：为什么不引入一个额外的陷阱来处理方法调用（例如`invoke`）？这样我们就可以区分：

+   通过`obj.prop`获取属性（陷阱`get`）

+   通过`obj.prop()`调用方法（陷阱`invoke`）

有两个原因不这样做。

首先，并非所有实现都区分`get`和`invoke`。例如，[苹果的 JavaScriptCore 没有](https://mail.mozilla.org/pipermail/es-discuss/2010-May/011062.html)。

其次，提取方法并稍后通过`.call()`或`.apply()`调用它应该与通过分派调用方法具有相同的效果。换句话说，以下两种变体应该等效工作。如果有额外的陷阱`invoke`，那么这种等价性将更难维持。

```js
// Variant 1: call via dynamic dispatch
const result1 = obj.m();

// Variant 2: extract and call directly
const m = obj.m;
const result2 = m.call(obj);
```

###### 18.5.4.3.1 `invoke`的用例

有些事情只有在我们能够区分`get`和`invoke`时才能完成。因此，这些事情在当前的代理 API 中是不可能的。两个例子是：自动绑定和拦截丢失的方法。让我们看看如果代理支持`invoke`，我们将如何实现它们。

**自动绑定。**通过将代理设置为对象`obj`的原型，我们可以自动绑定方法：

+   通过`obj.m`获取方法`m`的值将返回一个`this`绑定到`obj`的函数。

+   `obj.m()`执行方法调用。

自动绑定有助于使用方法作为回调。例如，前面示例中的第 2 个变体变得更简单：

```js
const boundMethod = obj.m;
const result = boundMethod();
```

**拦截丢失的方法。** `invoke`允许代理模拟先前提到的`__noSuchMethod__`机制。代理将再次成为对象`obj`的原型。它会根据未知属性`prop`的访问方式而有不同的反应：

+   如果通过`obj.prop`读取该属性，则不会发生拦截，返回`undefined`。

+   如果我们进行方法调用`obj.prop()`，那么代理会拦截，并且，例如，通知一个回调。

#### 18.5.5 强制执行代理的不变量

在我们讨论不变量是什么以及如何通过代理来强制执行它们之前，让我们回顾一下通过非可扩展性和非可配置性来保护对象的方法。

##### 18.5.5.1 保护对象

保护对象的两种方法：

+   非可扩展性保护对象：如果一个对象是非可扩展的，我们就不能添加属性，也不能改变它的原型。

+   非可配置性保护属性（或者说，它们的属性）：

    +   布尔属性`writable`控制属性的值是否可以更改。

    +   布尔属性`configurable`控制属性的属性是否可以更改。

有关此主题的更多信息，请参见§10“保护对象免受更改”。

##### 18.5.5.2 强制执行不变量

传统上，非可扩展性和非可配置性是：

+   通用：它们适用于所有对象。

+   单调：一旦打开，就不能再关闭。

这些以及其他在语言操作面前保持不变的特征被称为*不变量*。通过代理很容易违反不变量，因为它们不是通过非可扩展性等固有地受限制的。代理 API 通过检查目标对象和处理程序方法的结果来防止这种情况发生。

接下来的两个小节描述了四个不变量。不变量的详尽列表在本章末尾给出。

##### 18.5.5.3 通过目标对象强制执行的两个不变量

以下两个不变性涉及不可扩展性和不可配置性。这些是通过使用目标对象进行记录来强制执行的：处理程序方法返回的结果必须与目标对象大部分同步。

+   不变性：如果`Object.preventExtensions(obj)`返回`true`，则所有未来的调用必须返回`false`，并且`obj`现在必须是不可扩展的。

    +   通过抛出`TypeError`来强制执行代理，如果处理程序返回`true`，但目标对象不可扩展。

+   不变性：一旦对象被设置为不可扩展，`Object.isExtensible(obj)`必须始终返回`false`。

    +   通过抛出`TypeError`来强制执行代理，如果处理程序返回的结果（在强制转换后）与`Object.isExtensible(target)`不同。

##### 18.5.5.4 通过检查返回值强制执行的两个不变性

通过检查返回值强制执行的两个不变性是：

+   不变性：`Object.isExtensible(obj)`必须返回一个布尔值。

    +   通过强制处理程序返回的值转换为布尔值来强制执行代理。

+   不变性：`Object.getOwnPropertyDescriptor(obj, ···)`必须返回一个对象或`undefined`。

    +   通过抛出`TypeError`来强制执行代理，如果处理程序没有返回适当的值。

##### 18.5.5.5 不变性的好处

强制执行不变性具有以下好处：

+   代理与其他对象一样，关于可扩展性和可配置性。因此，保持了普遍性。这是在不阻止代理虚拟（冒充）受保护对象的情况下实现的。

+   受保护的对象不能通过包装代理来误导。误导可能是由错误或恶意代码引起的。

接下来的两节给出了强制执行不变性的示例。

##### 18.5.5.6 示例：不可扩展目标的原型必须被忠实地表示

在响应`getPrototypeOf`陷阱时，如果目标是不可扩展的，代理必须返回目标的原型。

为了演示这个不变性，让我们创建一个处理程序，返回一个与目标原型不同的原型：

```js
const fakeProto = {};
const handler = {
 getPrototypeOf(t) {
 return fakeProto;
 }
};
```

如果目标是可扩展的，则伪造原型可以起作用：

```js
const extensibleTarget = {};
const extProxy = new Proxy(extensibleTarget, handler);

assert.equal(
 Object.getPrototypeOf(extProxy), fakeProto);
```

但是，如果我们为不可扩展的对象伪造原型，就会出现错误。

```js
const nonExtensibleTarget = {};
Object.preventExtensions(nonExtensibleTarget);
const nonExtProxy = new Proxy(nonExtensibleTarget, handler);

assert.throws(
 () => Object.getPrototypeOf(nonExtProxy),
 {
 name: 'TypeError',
 message: "'getPrototypeOf' on proxy: proxy target is"
 + " non-extensible but the trap did not return its"
 + " actual prototype",
 });
```

##### 18.5.5.7 示例：不可写不可配置的目标属性必须被忠实地表示

如果目标具有不可写不可配置的属性，则处理程序必须在`get`陷阱的响应中返回该属性的值。为了演示这个不变性，让我们创建一个总是返回相同值的处理程序。

```js
const handler = {
 get(target, propKey) {
 return 'abc';
 }
};
const target = Object.defineProperties(
 {}, {
 manufacturer: {
 value: 'Iso Autoveicoli',
 writable: true,
 configurable: true
 },
 model: {
 value: 'Isetta',
 writable: false,
 configurable: false
 },
 });
const proxy = new Proxy(target, handler);
```

属性`target.manufacturer`既不可写也不可配置，这意味着处理程序可以假装它有不同的值：

```js
assert.equal(
 proxy.manufacturer, 'abc');
```

但是，属性`target.model`既不可写也不可配置。因此，我们无法伪造它的值：

```js
assert.throws(
 () => proxy.model,
 {
 name: 'TypeError',
 message: "'get' on proxy: property 'model' is a read-only and"
 + " non-configurable data property on the proxy target but"
 + " the proxy did not return its actual value (expected"
 + " 'Isetta' but got 'abc')",
 });
```

### 18.6 常见问题：代理

#### 18.6.1 `enumerate`陷阱在哪里？

ECMAScript 6 最初有一个名为`enumerate`的陷阱，它由`for-in`循环触发。但最近已经删除，以简化代理。`Reflect.enumerate()`也被删除了。([来源：TC39 笔记](https://github.com/tc39/tc39-notes/blob/master/es7/2016-01/2016-01-28.md))

### 18.7 参考：代理 API

本节是代理 API 的快速参考：

+   全局对象`Proxy`

+   全局对象`Reflect`

引用使用以下自定义类型：

```js
type PropertyKey = string | symbol;
```

#### 18.7.1 创建代理

有两种创建代理的方法：

+   `const proxy = new Proxy(target, handler)`

    使用给定的目标和给定的处理程序创建一个新的代理对象。

+   `const {proxy, revoke} = Proxy.revocable(target, handler)`

    创建一个可以通过函数`revoke`撤销的代理。`revoke`可以被多次调用，但只有第一次调用会产生效果并关闭`proxy`。之后，对`proxy`执行的任何操作都会导致抛出`TypeError`。

#### 18.7.2 处理程序方法

本小节解释了处理程序可以实现的陷阱以及触发它们的操作。几个陷阱返回布尔值。对于`has`和`isExtensible`陷阱，布尔值是操作的结果。对于所有其他陷阱，布尔值指示操作是否成功。

所有对象的陷阱：

+   `defineProperty(target, propKey, propDesc): boolean`

    +   `Object.defineProperty(proxy, propKey, propDesc)`

+   `deleteProperty(target, propKey): boolean`

    +   `delete proxy[propKey]`

    +   `delete proxy.someProp`

+   `get(target, propKey, receiver): any`

    +   `receiver[propKey]`

    +   `receiver.someProp`

+   `getOwnPropertyDescriptor(target, propKey): undefined|PropDesc`

    +   `Object.getOwnPropertyDescriptor(proxy, propKey)`

+   `getPrototypeOf(target): null|object`

    +   `Object.getPrototypeOf(proxy)`

+   `has(target, propKey): boolean`

    +   `propKey in proxy`

+   `isExtensible(target): boolean`

    +   `Object.isExtensible(proxy)`

+   `ownKeys(target): Array<PropertyKey>`

    +   `Object.getOwnPropertyPropertyNames(proxy)`（仅使用字符串键）

    +   `Object.getOwnPropertyPropertySymbols(proxy)`（仅使用符号键）

    +   `Object.keys(proxy)`（仅使用可枚举的字符串键；通过`Object.getOwnPropertyDescriptor`检查可枚举性）

+   `preventExtensions(target): boolean`

    +   `Object.preventExtensions(proxy)`

+   `set(target, propKey, value, receiver): boolean`

    +   `receiver[propKey] = value`

    +   `receiver.someProp = value`

+   `setPrototypeOf(target, proto): boolean`

    +   `Object.setPrototypeOf(proxy, proto)`

函数的陷阱（仅当目标是函数时可用）：

+   `apply(target, thisArgument, argumentsList): any`

    +   `proxy.apply(thisArgument, argumentsList)`

    +   `proxy.call(thisArgument, ...argumentsList)`

    +   `proxy(...argumentsList)`

+   `construct(target, argumentsList, newTarget): object`

    +   `new proxy(..argumentsList)`

##### 18.7.2.1 基本操作与派生操作

以下操作是*基本*的，它们不使用其他操作来完成工作：`apply`、`defineProperty`、`deleteProperty`、`getOwnPropertyDescriptor`、`getPrototypeOf`、`isExtensible`、`ownKeys`、`preventExtensions`、`setPrototypeOf`

所有其他操作都是*派生*的，它们可以通过基本操作来实现。例如，`get`可以通过使用`getPrototypeOf`迭代原型链并为每个链成员调用`getOwnPropertyDescriptor`来实现，直到找到自有属性或链结束为止。

#### 18.7.3 处理程序方法的不变量

不变量是处理程序的安全约束。本小节记录了代理 API 强制执行的不变量以及其工作原理。在下面每当我们读到“处理程序必须执行 X”时，这意味着如果处理程序没有执行 X，则会抛出`TypeError`。一些不变量限制返回值，另一些限制参数。陷阱返回值的正确性有两种保证方式：

+   如果期望布尔值，则使用强制转换将非布尔值转换为合法值。

+   在所有其他情况下，非法值会导致`TypeError`。

这是强制执行的不变量的完整列表：

+   `apply(target, thisArgument, argumentsList): any`

    +   不强制执行任何不变量。

    +   仅当目标可调用时才激活。

+   `construct(target, argumentsList, newTarget): object`

    +   处理程序返回的结果必须是对象（而不是`null`或任何其他原始值）。

    +   仅当目标可构造时才激活。

+   `defineProperty(target, propKey, propDesc): boolean`

    +   如果目标不可扩展，则无法添加新属性。

    +   如果`propDesc`将`configurable`属性设置为`false`，则目标必须具有一个不可配置的自有属性，其键为`propKey`。

    +   如果`propDesc`将`configurable`和`writable`属性都设置为`false`，则目标必须具有一个键为`propKey`的自有属性，该属性不可配置且不可写。

    +   如果目标具有键为`propKey`的自有属性，则`propDesc`必须与该属性兼容：如果我们使用描述符重新定义目标属性，则不得抛出异常。

+   `deleteProperty(target, propKey): boolean`

    +   如果：

        +   目标对象具有一个键为`propKey`的不可配置的自有属性。

        +   目标对象是不可扩展的，并且具有一个键为`propKey`的自有属性。

+   `get(target, propKey, receiver): any`

    +   如果目标对象具有一个自有的、不可写的、不可配置的数据属性，其键为`propKey`，则处理程序必须返回该属性的值。

    +   如果目标对象有一个自有的、不可配置的、没有 getter 的访问器属性，那么处理程序必须返回`undefined`。

+   `getOwnPropertyDescriptor(target, propKey): undefined|PropDesc`

    +   处理程序必须返回`undefined`或一个对象。

    +   目标对象的不可配置的自有属性不能被处理程序报告为不存在。

    +   如果目标对象是不可扩展的，则处理程序必须报告目标对象的自有属性存在。

    +   如果处理程序报告一个属性为不可配置，则该属性必须是目标对象的不可配置的自有属性。

    +   如果处理程序报告一个属性为不可配置且不可写，那么该属性必须是目标对象的不可配置不可写的自有属性。

+   `getPrototypeOf(target): null|object`

    +   结果必须是`null`或者一个对象。

    +   如果目标对象不可扩展，则处理程序必须返回目标对象的原型。

+   `has(target, propKey): boolean`

    +   目标对象的不可配置的自有属性不能被处理程序报告为不存在。

    +   如果目标对象是不可扩展的，那么目标对象的自有属性不能被报告为不存在。

+   `isExtensible(target): boolean`

    +   在转换为布尔值后，处理程序返回的值必须与`target.isExtensible()`相同。

+   `ownKeys(target): Array<PropertyKey>`

    +   处理程序必须返回一个对象，该对象被视为类似数组，并转换为数组。

    +   生成的数组不能包含重复条目。

    +   结果的每个元素必须是字符串或符号。

    +   结果必须包含目标对象的所有不可配置的自有属性的键。

    +   如果目标对象不可扩展，则结果必须恰好包含目标对象的自有属性的键（没有其他值）。

+   `preventExtensions(target): boolean`

    +   如果`target.isExtensible()`为`false`，则处理程序只能返回一个真值（表示成功更改）。

+   `set(target, propKey, value, receiver): boolean`

    +   如果目标对象具有一个不可写的、不可配置的数据属性，其键为`propKey`，则处理程序必须返回该属性的值。

    +   如果相应的目标对象属性是一个不可配置的访问器且没有 setter，则无法以任何方式设置该属性。

+   `setPrototypeOf(target, proto): boolean`

    +   如果目标对象不可扩展，则原型不能被更改。这是如何实施的：如果目标对象不可扩展且处理程序返回一个真值（表示成功更改），则`proto`必须与目标对象的原型相同。否则，将抛出`TypeError`。

![](img/290901f5575b7fa8e8b287bbaf550458.png) **ECMAScript 规范中的不变量**

在规范中，不变量在[“代理对象内部方法和内部插槽”](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)部分中列出。

#### 18.7.4 影响原型链的操作

普通对象的以下操作在原型链上执行操作。因此，如果该链中的一个对象是代理，则会触发其陷阱。规范将这些操作实现为内部自有方法（对 JavaScript 代码不可见）。但在本节中，我们假装它们是具有与陷阱相同名称的普通方法。参数`target`成为方法调用的接收者。

+   `target.get(propertyKey, receiver)`

    如果`target`没有具有给定键的自有属性，则在`target`的原型上调用`get`。

+   `target.has(propertyKey)`

    类似于`get`，如果`target`没有具有给定键的自有属性，则在`target`的原型上调用`has`。

+   `target.set(propertyKey, value, receiver)`

    类似于`get`，如果`target`没有具有给定键的自有属性，则在`target`的原型上调用`set`。

所有其他操作只影响自有属性，对原型链没有影响。

![](img/290901f5575b7fa8e8b287bbaf550458.png) **ECMAScript 规范中的内部操作**

在规范中，这些（和其他）操作在“[普通对象内部方法和内部插槽](https://tc39.es/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots)”一节中有描述。

#### 18.7.5 Reflect

全局对象`Reflect`实现了 JavaScript 元对象协议的所有可拦截操作作为方法。这些方法的名称与处理程序方法的名称相同，这有助于从处理程序转发操作到目标，正如我们所见。

+   `Reflect.apply(target, thisArgument, argumentsList): any`

    类似于`Function.prototype.apply()`。

+   `Reflect.construct(target, argumentsList, newTarget=target): object`

    `new`操作符作为一个函数。`target`是要调用的构造函数，可选参数`newTarget`指向启动当前构造函数调用链的构造函数。

+   `Reflect.defineProperty(target, propertyKey, propDesc): boolean`

    类似于`Object.defineProperty()`。

+   `Reflect.deleteProperty(target, propertyKey): boolean`

    `delete`操作符作为一个函数。但它的工作方式略有不同：如果成功删除属性或属性从未存在，则返回`true`。如果属性无法删除且仍然存在，则返回`false`。保护属性免受删除的唯一方法是使它们不可配置。在松散模式下，`delete`操作符返回相同的结果。但在严格模式下，它会抛出`TypeError`而不是返回`false`。

+   `Reflect.get(target, propertyKey, receiver=target): any`

    一个获取属性的函数。可选参数`receiver`指向获取开始的对象。当`get`在原型链中后面达到 getter 时，需要它。然后它为`this`提供值。

+   `Reflect.getOwnPropertyDescriptor(target, propertyKey): undefined|PropDesc`

    与`Object.getOwnPropertyDescriptor()`相同。

+   `Reflect.getPrototypeOf(target): null|object`

    与`Object.getPrototypeOf()`相同。

+   `Reflect.has(target, propertyKey): boolean`

    `in`操作符作为一个函数。

+   `Reflect.isExtensible(target): boolean`

    与`Object.isExtensible()`相同。

+   `Reflect.ownKeys(target): Array<PropertyKey>`

    以数组形式返回所有自有属性键：所有自有可枚举和不可枚举属性的字符串键和符号键。

+   `Reflect.preventExtensions(target): boolean`

    类似于`Object.preventExtensions()`。

+   `Reflect.set(target, propertyKey, value, receiver=target): boolean`

    一个设置属性的函数。

+   `Reflect.setPrototypeOf(target, proto): boolean`

    设置对象原型的新标准方式。目前大多数引擎中有效的非标准方式是设置特殊属性`__proto__`。

几种方法具有布尔结果。对于`.has()`和`.isExtensible()`，它们是操作的结果。对于其余的方法，它们指示操作是否成功。

##### 18.7.5.1 `Reflect`的用例除了转发

除了转发操作，[为什么`Reflect`有用[4]](ch_proxies.html#further-reading-proxies)？

+   不同的返回值：`Reflect`复制了`Object`的以下方法，但其方法返回布尔值，指示操作是否成功（而`Object`方法返回被修改的对象）。

    +   `Object.defineProperty(obj, propKey, propDesc): object`

    +   `Object.preventExtensions(obj): object`

    +   `Object.setPrototypeOf(obj, proto): object`

+   作为函数的运算符：以下`Reflect`方法实现了通过运算符才能实现的功能：

    +   `Reflect.construct(target, argumentsList, newTarget=target): object`

    +   `Reflect.deleteProperty(target, propertyKey): boolean`

    +   `Reflect.get(target, propertyKey, receiver=target): any`

    +   `Reflect.has(target, propertyKey): boolean`

    +   `Reflect.set(target, propertyKey, value, receiver=target): boolean`

+   `apply()`的简短版本：如果我们想完全安全地调用函数的`apply()`方法，我们不能通过动态分发来做到这一点，因为函数可能具有一个具有键`'apply'`的自有属性：

    ```js
    func.apply(thisArg, argArray) // not safe
    Function.prototype.apply.call(func, thisArg, argArray) // safe
    ```

    使用`Reflect.apply()`比安全版本更短：

    ```js
    Reflect.apply(func, thisArg, argArray)
    ```

+   删除属性时不会抛出异常：在严格模式下，如果我们尝试删除一个不可配置的自有属性，`delete`运算符会抛出异常。在这种情况下，`Reflect.deleteProperty()`会返回`false`。

##### 18.7.5.2 `Object.*`与`Reflect.*`

未来，`Object`将承载对普通应用程序有兴趣的操作，而`Reflect`将承载更低级的操作。

### 18.8 结论

这结束了我们对代理 API 的深入研究。需要注意的一点是，代理会减慢代码。如果性能很重要，这可能很重要。

另一方面，性能通常并不是关键，拥有代理赋予我们的元编程能力是很好的。

* * *

**致谢：**

+   Allen Wirfs-Brock 指出了§18.3.7“陷阱：并非所有对象都可以被代理透明地包装”中解释的陷阱。

+   §18.4.3“通过代理使用负数组索引（`get`）”的想法来自[Hemanth.HM](https://twitter.com/gnumanth)的[博客文章](http://h3manth.com/new/blog/2013/negative-array-index-in-javascript/)。

+   André Jaenisch 为使用代理的库列表做出了贡献。

### 18.9 进一步阅读

+   [1] “[关于 ECMAScript 反射 API 设计](http://soft.vub.ac.be/Publications/2012/vub-soft-tr-12-03.pdf)” by Tom Van Cutsem and Mark Miller. Technical report, 2012\. [本章的重要来源。]

+   [2] “[元对象协议的艺术](http://mitpress.mit.edu/books/art-metaobject-protocol)” by Gregor Kiczales, Jim des Rivieres and Daniel G. Bobrow. Book, 1991.

+   [3] “[将元类应用于工作：面向对象编程的新维度](http://www.pearsonhighered.com/educator/product/Putting-Metaclasses-to-Work-A-New-Dimension-in-ObjectOriented-Programming/9780201433050.page)” by Ira R. Forman and Scott H. Danforth. Book, 1999.

+   [4] “[Harmony-reflect: 为什么我应该使用这个库？](https://github.com/tvcutsem/harmony-reflect/wiki)” by Tom Van Cutsem. [解释了为什么`Reflect`很有用。]

[评论](https://github.com/rauschma/deep-js/issues/23)
