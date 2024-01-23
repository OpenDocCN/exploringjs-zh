# 二十四、异常处理

> 原文：[`exploringjs.com/impatient-js/ch_exception-handling.html`](https://exploringjs.com/impatient-js/ch_exception-handling.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   24.1 动机：抛出和捕获异常

+   24.2 `throw`

    +   24.2.1 我们应该抛出什么值？

+   24.3 `try`语句

    +   24.3.1 `try`块

    +   24.3.2 `catch`子句

    +   24.3.3 `finally`子句

+   24.4 `Error`及其子类

    +   24.4.1 类`Error`

    +   24.4.2 `Error`的内置子类

    +   24.4.3 子类化`Error`

+   24.5 链接错误

    +   24.5.1 为什么我们想要链接错误？

    +   24.5.2 [通过`error.cause` [ES2022]链接错误](ch_exception-handling.html#error.cause)

    +   24.5.3 `.cause`的替代方案：自定义错误类

* * *

本章介绍了 JavaScript 如何处理异常。

![](img/b666ba365e94edaf0ef510fd7e12c7de.png) **为什么 JavaScript 不经常抛出异常？**

直到 ES3，JavaScript 才支持异常。这就解释了为什么它们在语言和标准库中被稀少地使用。

### 24.1 动机：抛出和捕获异常

考虑以下代码。它从文件中读取存储的配置文件到一个具有`Profile`类实例的数组中：

```js
function readProfiles(filePaths) {
 const profiles = [];
 for (const filePath of filePaths) {
 try {
 const profile = readOneProfile(filePath);
 profiles.push(profile);
 } catch (err) { // (A)
 console.log('Error in: '+filePath, err);
 }
 }
}
function readOneProfile(filePath) {
 const profile = new Profile();
 const file = openFile(filePath);
 // ··· (Read the data in `file` into `profile`)
 return profile;
}
function openFile(filePath) {
 if (!fs.existsSync(filePath)) {
 throw new Error('Could not find file '+filePath); // (B)
 }
 // ··· (Open the file whose path is `filePath`)
}
```

让我们来看看 B 行发生了什么：发生了错误，但处理问题的最佳位置不是当前位置，而是 A 行。在那里，我们可以跳过当前文件并继续下一个文件。

因此：

+   在 B 行，我们使用`throw`语句表示出现了问题。

+   在 A 行，我们使用`try-catch`语句来处理问题。

当我们抛出时，以下结构是活动的：

```js
readProfiles(···)
  for (const filePath of filePaths)
    try
      readOneProfile(···)
        openFile(···)
          if (!fs.existsSync(filePath))
            throw
```

`throw`逐个退出嵌套结构，直到遇到`try`语句。执行将继续在该`try`语句的`catch`子句中。

### 24.2 `throw`

这是`throw`语句的语法：

```js
throw «value»;
```

#### 24.2.1 我们应该抛出什么值？

在 JavaScript 中可以抛出任何值。但最好使用`Error`的实例或子类，因为它们支持附加功能，如堆栈跟踪和错误链接（参见§24.4“`Error`及其子类”）。

这给我们留下了以下选项：

+   直接使用`Error`类。这在 JavaScript 中比在更静态的语言中更不受限制，因为我们可以向实例添加自己的属性：

    ```js
    const err = new Error('Could not find the file');
    err.filePath = filePath;
    throw err;
    ```

+   使用 Error 的子类之一。

+   `Error`的子类化（更多细节在后面解释）：

    ```js
    class MyError extends Error {
    }
    function func() {
     throw new MyError('Problem!');
    }
    assert.throws(
     () => func(),
     MyError);
    ```

### 24.3 `try`语句

`try`语句的最大版本如下所示：

```js
try {
 «try_statements»
} catch (error) {
 «catch_statements»
} finally {
 «finally_statements»
}
```

我们可以将这些子句组合如下：

+   `try-catch`

+   `try-finally`

+   `try-catch-finally`

#### 24.3.1 `try`块

`try`块可以被视为语句的主体。这是我们执行常规代码的地方。

#### 24.3.2 `catch`子句

如果异常到达`try`块，则它被分配给`catch`子句的参数，并且在该子句中的代码被执行。接下来，执行通常会在`try`语句之后继续。如果：

+   在`catch`块内部有一个`return`、`break`或`throw`。

+   有一个`finally`子句（在`try`语句结束之前始终执行）。

以下代码演示了在 A 行抛出的值确实在 B 行被捕获。

```js
const errorObject = new Error();
function func() {
 throw errorObject; // (A)
}

try {
 func();
} catch (err) { // (B)
 assert.equal(err, errorObject);
}
```

##### 24.3.2.1 省略`catch`绑定[ES2019]

如果我们对抛出的值不感兴趣，我们可以省略`catch`参数：

```js
try {
 // ···
} catch {
 // ···
}
```

这可能偶尔会有用。例如，Node.js 有 API 函数[`assert.throws(func)`](https://nodejs.org/api/assert.html#assert_assert_throws_fn_error_message)，用于检查`func`内部是否抛出错误。可以这样实现。

```js
function throws(func) {
 try {
 func();
 } catch {
 return; // everything OK
 }
 throw new Error('Function didn’t throw an exception!');
}
```

然而，这个函数的更完整的实现将有一个`catch`参数，并且，例如，检查其类型是否符合预期。

#### 24.3.3 `finally`子句

`finally`子句中的代码总是在`try`语句的末尾执行 - 无论`try`块或`catch`子句中发生了什么。

让我们看一个`finally`的常见用例：我们创建了一个资源，并且希望在完成后始终销毁它，无论在处理它时发生了什么。我们可以这样实现：

```js
const resource = createResource();
try {
 // Work with `resource`. Errors may be thrown.
} finally {
 resource.destroy();
}
```

##### 24.3.3.1 `finally`总是被执行

`finally`子句总是被执行，即使抛出错误（A 行）：

```js
let finallyWasExecuted = false;
assert.throws(
 () => {
 try {
 throw new Error(); // (A)
 } finally {
 finallyWasExecuted = true;
 }
 },
 Error
);
assert.equal(finallyWasExecuted, true);
```

即使有`return`语句（A 行）：

```js
let finallyWasExecuted = false;
function func() {
 try {
 return; // (A)
 } finally {
 finallyWasExecuted = true;
 }
}
func();
assert.equal(finallyWasExecuted, true);
```

### 24.4 `Error`及其子类

`Error`是所有内置错误类的通用超类。

#### 24.4.1 类`Error`

这是`Error`的实例属性和构造函数的样子：

```js
class Error {
 // Instance properties
 message: string;
 cause?: any; // ES2022
 stack: string; // non-standard but widely supported

 constructor(
 message: string = '',
 options?: ErrorOptions // ES2022
 );
}
interface ErrorOptions {
 cause?: any; // ES2022
}
```

构造函数有两个参数：

+   `message`指定了错误消息。

+   `options`在 ECMAScript 2022 中被引入。它包含一个对象，其中一个属性目前受支持：

    +   `.cause`指定了导致当前错误的异常（如果有）。

接下来的子节将更详细地解释实例属性`.message`、`.cause`和`.stack`。

##### 24.4.1.1 `Error.prototype.name`

每个内置错误类`E`都有一个属性`E.prototype.name`：

```js
> Error.prototype.name
'Error'
> RangeError.prototype.name
'RangeError'
```

因此，有两种方法可以获取内置错误对象的类名：

```js
> new RangeError().name
'RangeError'
> new RangeError().constructor.name
'RangeError'
```

##### 24.4.1.2 错误实例属性`.message`

`.message`只包含错误消息：

```js
const err = new Error('Hello!');
assert.equal(String(err), 'Error: Hello!');
assert.equal(err.message, 'Hello!');
```

如果我们省略消息，那么空字符串将被用作默认值（从`Error.prototype.message`继承）：

如果我们省略消息，它就是空字符串：

```js
assert.equal(new Error().message, '');
```

##### 24.4.1.3 错误实例属性`.stack`

实例属性`.stack`不是 ECMAScript 特性，但它被 JavaScript 引擎广泛支持。它通常是一个字符串，但其确切结构没有标准化，并且在引擎之间有所不同。

这是在 JavaScript 引擎 V8 上的样子：

```js
const err = new Error('Hello!');
assert.equal(
err.stack,
`
Error: Hello!
 at file://ch_exception-handling.mjs:1:13
`.trim());
```

##### 24.4.1.4 错误实例属性`.cause` [ES2022]

实例属性`.cause`是通过`new Error()`的第二个参数中的选项对象创建的。它指定了哪个其他错误导致了当前错误。

```js
const err = new Error('msg', {cause: 'the cause'});
assert.equal(err.cause, 'the cause');
```

有关如何使用此属性的信息，请参阅§24.5“链接错误”。

#### 24.4.2 `Error`的内置子类

`Error`有以下子类 - 引用[ECMAScript 规范](https://tc39.github.io/ecma262/#sec-native-error-types-used-in-this-standard)：

+   `AggregateError` [ES2021] 代表一次性表示多个错误。在标准库中，只有`Promise.any()`使用它。

+   `RangeError`指示一个不在可允许值的集合或范围内的值。

+   `ReferenceError`指示检测到无效引用值。

+   `SyntaxError`指示发生了解析错误。

+   `TypeError`用于指示未成功操作，当其他*NativeError*对象都不适合表示失败原因时。

+   `URIError`指示全局 URI 处理函数之一的使用方式与其定义不兼容。

#### 24.4.3 对`Error`进行子类化

自 ECMAScript 2022 以来，`Error`构造函数接受两个参数（参见前面的子节）。因此，在对其进行子类化时，我们有两种选择：要么在子类中省略构造函数，要么像这样调用`super()`：

```js
class MyCustomError extends Error {
 constructor(message, options) {
 super(message, options);
 // ···
 }
}
```

### 24.5 链接错误

#### 24.5.1 为什么我们想要链接错误？

有时，我们捕获在更深层嵌套的函数调用期间抛出的错误，并希望附加更多信息：

```js
function readFiles(filePaths) {
 return filePaths.map(
 (filePath) => {
 try {
 const text = readText(filePath);
 const json = JSON.parse(text);
 return processJson(json);
 } catch (error) {
 // (A)
 }
 });
}
```

`try`子句中的语句可能会引发各种错误。在大多数情况下，错误不会意识到导致它的文件路径。这就是为什么我们想在 A 行附加该信息的原因。

#### 24.5.2 通过`error.cause`链接错误[ES2022]

自 ECMAScript 2022 以来，`new Error()`让我们指定引起错误的原因：

```js
function readFiles(filePaths) {
 return filePaths.map(
 (filePath) => {
 try {
 // ···
 } catch (error) {
 throw new Error(
 `While processing ${filePath}`,
 {cause: error}
 );
 }
 });
}
```

#### 24.5.3 `.cause`的替代方法：自定义错误类

以下自定义错误类支持链接。它与`.cause`向前兼容。

```js
/**
 * An error class that supports error chaining.
 * If there is built-in support for .cause, it uses it.
 * Otherwise, it creates this property itself.
 *
 * @see https://github.com/tc39/proposal-error-cause
 */
class CausedError extends Error {
 constructor(message, options) {
 super(message, options);
 if (
 (isObject(options) && 'cause' in options)
 && !('cause' in this)
 ) {
 // .cause was specified but the superconstructor
 // did not create an instance property.
 const cause = options.cause;
 this.cause = cause;
 if ('stack' in cause) {
 this.stack = this.stack + '\nCAUSE: ' + cause.stack;
 }
 }
 }
}

function isObject(value) {
 return value !== null && typeof value === 'object';
}
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：异常处理**

`exercises/exception-handling/call_function_test.mjs`

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/42)
