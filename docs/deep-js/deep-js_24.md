# 十七、通过实现 Promise 来探索 Promise

> 原文：[`exploringjs.com/deep-js/ch_implementing-promises.html`](https://exploringjs.com/deep-js/ch_implementing-promises.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   17.1 复习：Promise 的状态

+   17.2 版本 1：独立的 Promise

    +   17.2.1 方法`.then()`

    +   17.2.2 方法`.resolve()`

+   17.3 版本 2：链接`.then()`调用

+   17.4 便捷方法`.catch()`

+   17.5 省略反应

+   17.6 实现

+   17.7 版本 3：扁平化从`.then()`回调返回的 Promise

    +   17.7.1 从`.then()`的回调中返回 Promise

    +   17.7.2 扁平化使 Promise 状态更加复杂

    +   17.7.3 实现 Promise 扁平化

+   17.8 版本 4：在反应回调中抛出异常

+   17.9 版本 5：揭示构造函数模式

* * *

![](img/0c2cffc9b09a6e4a1ac19d36b7230eed.png)  **所需知识：Promise**

在本章中，您应该对 Promise 有一定了解，但这里也复习了许多相关知识。如果需要，您可以阅读[“JavaScript for impatient programmers”中关于 Promise 的章节](https://exploringjs.com/impatient-js/ch_promises.html)。

在这一章中，我们将从不同的角度来接触 Promise：我们将创建一个简单的实现。这种不同的角度曾经帮助我很大地理解 Promise。

Promise 的实现是`ToyPromise`类。为了更容易理解，它并不完全匹配 API。但它足够接近，仍然能让我们深入了解 Promise 的工作原理。

![](img/0c2cffc9b09a6e4a1ac19d36b7230eed.png)  **带有代码的存储库**

`ToyPromise`可以在 GitHub 上找到，存储在[`toy-promise`](https://github.com/rauschma/toy-promise)存储库中。

### 17.1 复习：Promise 的状态

![](img/2464420838a24a1e997c946490e804d6.png)

图 11：Promise 的状态（简化版本）：Promise 最初是 pending 状态。如果我们解决它，它就会变成 fulfilled。如果我们拒绝它，它就会变成 rejected。

我们从一个简化版本开始解释 Promise 状态的工作方式（图 11）：

+   Promise 最初是*pending*状态。

+   如果一个 Promise 被值`v` *resolved*，它就会变成*fulfilled*（稍后，我们将看到解决也可以拒绝）。`v`现在是 Promise 的*fulfillment value*。

+   如果一个 Promise 被错误`e` *rejected*，它就会变成*rejected*。`e`现在是 Promise 的*rejection value*。

### 17.2 版本 1：独立的 Promise

我们的第一个实现是一个独立的 Promise，具有最小的功能：

+   我们可以创建一个 Promise。

+   我们可以解决或拒绝一个 Promise，而且只能做一次。

+   我们可以通过`.then()`注册*reactions*（回调）。注册必须独立于 Promise 是否已经解决或未解决而做正确的事情。

+   `.then()`目前不支持链接，它不返回任何东西。

`ToyPromise1`是一个具有三个原型方法的类：

+   `ToyPromise1.prototype.resolve(value)`

+   `ToyPromise1.prototype.reject(reason)`

+   `ToyPromise1.prototype.then(onFulfilled, onRejected)`

也就是说，`resolve`和`reject`是方法（而不是传递给构造函数回调参数的函数）。

这是第一个实现的用法：

```js
// .resolve() before .then()
const tp1 = new ToyPromise1();
tp1.resolve('abc');
tp1.then((value) => {
 assert.equal(value, 'abc');
});
```

```js
// .then() before .resolve()
const tp2 = new ToyPromise1();
tp2.then((value) => {
 assert.equal(value, 'def');
});
tp2.resolve('def');
```

图 12 说明了我们的第一个`ToyPromise`是如何工作的。

![](img/6b17215a2abf7f098e996c026fba60fd.png) **Promises 中的数据流图是可选的**

图表的动机是为 Promises 的工作原理提供一个视觉解释。但它们是可选的。如果你觉得它们令人困惑，你可以忽略它们，专注于代码。

![](img/e65f0282853cd39c5f190034058283ac.png)

图 12：`ToyPromise1`：如果 Promise 被解决，提供的值将传递给*满足反应*（`.then()`的第一个参数）。如果 Promise 被拒绝，提供的值将传递给*拒绝反应*（`.then()`的第二个参数）。

#### 17.2.1 方法`.then()`

让我们先来看一下`.then()`。它必须处理两种情况：

+   如果 Promise 仍处于挂起状态，则会排队调用`onFulfilled`和`onRejected`。它们将在 Promise 解决时使用。

+   如果 Promise 已经被满足或拒绝，`onFulfilled`或`onRejected`可以立即被调用。

```js
then(onFulfilled, onRejected) {
 const fulfillmentTask = () => {
 if (typeof onFulfilled === 'function') {
 onFulfilled(this._promiseResult);
 }
 };
 const rejectionTask = () => {
 if (typeof onRejected === 'function') {
 onRejected(this._promiseResult);
 }
 };
 switch (this._promiseState) {
 case 'pending':
 this._fulfillmentTasks.push(fulfillmentTask);
 this._rejectionTasks.push(rejectionTask);
 break;
 case 'fulfilled':
 addToTaskQueue(fulfillmentTask);
 break;
 case 'rejected':
 addToTaskQueue(rejectionTask);
 break;
 default:
 throw new Error();
 }
}
```

上一个代码片段使用以下辅助函数：

```js
function addToTaskQueue(task) {
 setTimeout(task, 0);
}
```

Promise 必须始终异步解决。这就是为什么我们不直接执行任务，而是将它们添加到事件循环的任务队列中（浏览器、Node.js 等）。请注意，真正的 Promise API 不使用普通任务（如`setTimeout()`），它使用[*微任务*](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)，它们与当前普通任务紧密耦合，并且总是直接执行。

#### 17.2.2 方法`.resolve()`

`.resolve()`的工作方式如下：如果 Promise 已经解决，它什么也不做（确保 Promise 只能解决一次）。否则，Promise 的状态将更改为`'fulfilled'`，结果将缓存在`this.promiseResult`中。接下来，将调用到目前为止已排队的所有满足反应。

```js
resolve(value) {
 if (this._promiseState !== 'pending') return this;
 this._promiseState = 'fulfilled';
 this._promiseResult = value;
 this._clearAndEnqueueTasks(this._fulfillmentTasks);
 return this; // enable chaining
}
```

```js
_clearAndEnqueueTasks(tasks) {
 this._fulfillmentTasks = undefined;
 this._rejectionTasks = undefined;
 tasks.map(addToTaskQueue);
}
```

`reject()`类似于`resolve()`。

### 17.3 版本 2：链接`.then()`调用

![](img/b17ddb8b605b284cdb68bb5baf7692ee.png)

图 13：`ToyPromise2`链接`.then()`调用：`.then()`现在返回一个 Promise，该 Promise 由满足反应或拒绝反应返回的任何值解决。

我们实现的下一个特性是链接（图 13）：我们从满足反应或拒绝反应中返回的值可以由后续的`.then()`调用中的满足反应处理。（在下一个版本中，由于特殊支持返回 Promises，链接将变得更加有用。）

在下面的示例中：

+   第一个`.then()`：我们在满足反应中返回一个值。

+   第二个`.then()`：我们通过满足反应接收该值。

```js
new ToyPromise2()
 .resolve('result1')
 .then(x => {
 assert.equal(x, 'result1');
 return 'result2';
 })
 .then(x => {
 assert.equal(x, 'result2');
 });
```

在下面的示例中：

+   第一个`.then()`：我们在拒绝反应中返回一个值。

+   第二个`.then()`：我们通过满足反应接收该值。

```js
new ToyPromise2()
 .reject('error1')
 .then(null,
 x => {
 assert.equal(x, 'error1');
 return 'result2';
 })
 .then(x => {
 assert.equal(x, 'result2');
 });
```

### 17.4 便利方法`.catch()`

新版本引入了一个方便的方法`.catch()`，使得只提供拒绝反应更容易。请注意，只提供满足反应已经很容易 - 我们只需省略`.then()`的第二个参数（参见上一个示例）。

如果我们使用它，上一个示例看起来更好（A 行）：

```js
new ToyPromise2()
 .reject('error1')
 .catch(x => { // (A)
 assert.equal(x, 'error1');
 return 'result2';
 })
 .then(x => {
 assert.equal(x, 'result2');
 });
```

以下两个方法调用是等效的：

```js
.catch(rejectionReaction)
.then(null, rejectionReaction)
```

这就是`.catch()`的实现方式：

```js
catch(onRejected) { // [new]
 return this.then(null, onRejected);
}
```

### 17.5 省略反应

新版本还会在我们省略满足反应时转发满足，并在我们省略拒绝反应时转发拒绝。这有什么用呢？

以下示例演示了如何传递拒绝：

```js
someAsyncFunction()
 .then(fulfillmentReaction1)
 .then(fulfillmentReaction2)
 .catch(rejectionReaction);
```

`rejectionReaction`现在可以处理`someAsyncFunction()`、`fulfillmentReaction1`和`fulfillmentReaction2`的拒绝。

以下示例演示了如何传递满足：

```js
someAsyncFunction()
 .catch(rejectionReaction)
 .then(fulfillmentReaction);
```

如果`someAsyncFunction()`拒绝了它的 Promise，`rejectionReaction`可以修复任何问题并返回一个完成值，然后由`fulfillmentReaction`处理。

如果`someAsyncFunction()`实现了它的 Promise，`fulfillmentReaction`也可以处理它，因为`.catch()`被跳过了。

### 17.6 实现

所有这些是如何在底层处理的？

+   `.then()`返回一个 Promise，该 Promise 解析为`onFulfilled`或`onRejected`返回的内容。

+   如果`onFulfilled`或`onRejected`丢失，无论它们将接收到什么都会传递给由`.then()`返回的 Promise。

只有`.then()`发生了变化：

```js
then(onFulfilled, onRejected) {
 const resultPromise = new ToyPromise2(); // [new]

 const fulfillmentTask = () => {
 if (typeof onFulfilled === 'function') {
 const returned = onFulfilled(this._promiseResult);
 resultPromise.resolve(returned); // [new]
 } else { // [new]
 // `onFulfilled` is missing
 // => we must pass on the fulfillment value
 resultPromise.resolve(this._promiseResult);
 } 
 };

 const rejectionTask = () => {
 if (typeof onRejected === 'function') {
 const returned = onRejected(this._promiseResult);
 resultPromise.resolve(returned); // [new]
 } else { // [new]
 // `onRejected` is missing
 // => we must pass on the rejection value
 resultPromise.reject(this._promiseResult);
 }
 };

 ···

 return resultPromise; // [new]
}
```

`.then()`创建并返回一个新的 Promise（方法的第一行和最后一行）。另外：

+   `fulfillmentTask`的工作方式不同。现在完成后会发生什么：

    +   如果提供了`onFullfilled`，则调用它并使用其结果来解析`resultPromise`。

    +   如果`onFulfilled`丢失，我们使用当前 Promise 的完成值来解析`resultPromise`。

+   `rejectionTask`的工作方式不同。这是拒绝后现在发生的事情：

    +   如果提供了`onRejected`，则调用它并使用其结果来*解析*`resultPromise`。请注意，`resultPromise`不会被拒绝：我们假设`onRejected()`修复了任何问题。

    +   如果`onRejected`丢失，我们使用当前 Promise 的拒绝值来拒绝`resultPromise`。

### 17.7 版本 3：扁平化从`.then()`回调返回的 Promises

#### 17.7.1 从`.then()`回调返回 Promises

Promise 扁平化主要是为了使链接更加方便：如果我们想要将一个值从一个`.then()`回调传递到下一个回调，我们在前者中返回它。之后，`.then()`将其放入它已经返回的 Promise 中。

如果我们从`.then()`回调返回一个 Promise，这种方法就变得不方便。例如，基于 Promise 的函数的结果（A 行）：

```js
asyncFunc1()
.then((result1) => {
 assert.equal(result1, 'Result of asyncFunc1()');
 return asyncFunc2(); // (A)
})
.then((result2Promise) => {
 result2Promise
 .then((result2) => { // (B)
 assert.equal(
 result2, 'Result of asyncFunc2()');
 });
});
```

这一次，将 A 行返回的值放入由`.then()`返回的 Promise 中，迫使我们在 B 行解开该 Promise。如果能够让 A 行返回的 Promise 替换由`.then()`返回的 Promise，那将是很好的。如何确切地做到这一点并不立即清楚，但如果成功，我们可以像这样编写我们的代码：

```js
asyncFunc1()
.then((result1) => {
 assert.equal(result1, 'Result of asyncFunc1()');
 return asyncFunc2(); // (A)
})
.then((result2) => {
 // result2 is the fulfillment value, not the Promise
 assert.equal(
 result2, 'Result of asyncFunc2()');
});
```

在 A 行，我们返回了一个 Promise。由于 Promise 扁平化，`result2`是该 Promise 的完成值，而不是 Promise 本身。

#### 17.7.2 扁平化使 Promise 状态变得更加复杂

![](img/290901f5575b7fa8e8b287bbaf550458.png)  **在 ECMAScript 规范中扁平化 Promises**

在 ECMAScript 规范中，扁平化 Promises 的细节在[“Promise Objects”部分](https://tc39.es/ecma262/#sec-promise-objects)中有描述。

Promise API 如何处理扁平化？

如果 Promise P 用 Promise Q 解析，那么 P 不会包装 Q，P“变成”Q：P 的状态和解决值现在总是与 Q 的相同。这有助于我们理解`.then()`，因为`.then()`将其返回的 Promise 解析为其回调之一返回的值。

P 如何变成 Q？通过*锁定*Q：P 变得外部无法解决，Q 的解决会触发 P 的解决。锁定是一个额外的不可见的 Promise 状态，使状态变得更加复杂。

Promise API 还有一个额外的特性：Q 不必是一个 Promise，只需是一个所谓的*thenable*。thenable 是一个带有方法`.then()`的对象。增加这种灵活性的原因是为了使不同的 Promise 实现能够一起工作（当 Promise 首次添加到语言中时很重要）。

图[14]（#fig:promise-states-all）可视化了新的状态。

![](img/0742c20efb6f5057f0bd62c47d52311d.png)

图 14：Promise 的所有状态：Promise 扁平化引入了不可见的伪状态“锁定”。如果 Promise P 用 thenable Q 解析，那么 P 的状态和解决值总是与 Q 相同。

请注意，*解决*的概念也变得更加复杂。现在解决一个 Promise 只意味着它不能直接被解决：

+   解决可能会拒绝一个 Promise：我们可以用一个被拒绝的 Promise 来解决一个 Promise。

+   解决甚至可能不会解决一个 Promise：我们可以用另一个始终处于挂起状态的 Promise 来解决一个 Promise。

ECMAScript 规范是这样规定的：“一个未解决的 Promise 始终处于挂起状态。已解决的 Promise 可能是挂起的、已实现的或已拒绝的。”

#### 17.7.3 实现 Promise 扁平化

图 15 显示了`ToyPromise3`如何处理扁平化。

![](img/9199e8ef16c2591c4b1c8fde86d52e89.png)

图 15：`ToyPromise3`扁平化已解决的 Promises：如果第一个 Promise 以一个 thenable `x1`解决，它将锁定在`x1`上，并以`x1`的解决值解决。如果第一个 Promise 以一个非 thenable 值解决，一切都与以前一样。

我们通过这个函数检测 thenables：

```js
function isThenable(value) { // [new]
 return typeof value === 'object' && value !== null
 && typeof value.then === 'function';
}
```

为了实现锁定，我们引入了一个新的布尔标志`._alreadyResolved`。将其设置为`true`会停用`.resolve()`和`.reject()`，例如：

```js
resolve(value) { // [new]
 if (this._alreadyResolved) return this;
 this._alreadyResolved = true;

 if (isThenable(value)) {
 // Forward fulfillments and rejections from `value` to `this`.
 // The callbacks are always executed asynchronously
 value.then(
 (result) => this._doFulfill(result),
 (error) => this._doReject(error));
 } else {
 this._doFulfill(value);
 }

 return this; // enable chaining
}
```

如果`value`是一个 thenable，那么我们将当前 Promise 锁定在它上面：

+   如果`value`以结果实现，当前 Promise 也将以该结果实现。

+   如果`value`以错误拒绝，当前 Promise 也将以该错误拒绝。

通过私有方法`._doFulfill()`和`._doReject()`执行结算，以绕过`._alreadyResolved`的保护。

`._doFulfill()`相对简单：

```js
_doFulfill(value) { // [new]
 assert.ok(!isThenable(value));
 this._promiseState = 'fulfilled';
 this._promiseResult = value;
 this._clearAndEnqueueTasks(this._fulfillmentTasks);
}
```

这里没有显示`.reject()`。它唯一的新功能是它现在也遵守`._alreadyResolved`。

### 17.8 版本 4：反应回调中抛出的异常

![](img/6a4077d1031b0a2125df45147ca425b2.png)

图 16：`ToyPromise4`将 Promise 反应中的异常转换为`.then()`返回的 Promise 的拒绝。

作为我们的最终特性，我们希望我们的 Promises 能够将用户代码中的异常作为拒绝处理（图 16）。在本章中，“用户代码”指的是`.then()`的两个回调参数。

```js
new ToyPromise4()
 .resolve('a')
 .then((value) => {
 assert.equal(value, 'a');
 throw 'b'; // triggers a rejection
 })
 .catch((error) => {
 assert.equal(error, 'b');
 })
```

`.then()`现在通过辅助方法`._runReactionSafely()`安全地运行 Promise 反应`onFulfilled`和`onRejected`，例如：

```js
 const fulfillmentTask = () => {
 if (typeof onFulfilled === 'function') {
 this._runReactionSafely(resultPromise, onFulfilled); // [new]
 } else {
 // `onFulfilled` is missing
 // => we must pass on the fulfillment value
 resultPromise.resolve(this._promiseResult);
 } 
 };
```

`._runReactionSafely()`的实现如下：

```js
_runReactionSafely(resultPromise, reaction) { // [new]
 try {
 const returned = reaction(this._promiseResult);
 resultPromise.resolve(returned);
 } catch (e) {
 resultPromise.reject(e);
 }
}
```

### 17.9 版本 5：揭示构造函数模式

我们跳过了最后一步：如果我们想要将`ToyPromise`转换为一个实际的 Promise 实现，我们仍然需要实现[揭示构造函数模式](https://blog.domenic.me/the-revealing-constructor-pattern/)：JavaScript Promises 不是通过方法而是通过函数来解决和拒绝的，这些函数被传递给*执行程序*，构造函数的回调参数。

```js
const promise = new Promise(
 (resolve, reject) => { // executor
 // ···
 });
```

如果执行程序抛出异常，则`promise`将被拒绝。

[Comments](https://github.com/rauschma/deep-js/issues/18)
