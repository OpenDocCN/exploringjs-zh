# 第十四章：异常处理

> 原文：[14. Exception Handling](https://exploringjs.com/es5/ch14.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章描述了 JavaScript 的异常处理工作原理。它从异常处理的一般解释开始。

## 什么是异常处理？

在异常处理中，通常会将紧密耦合的语句分组在一起。如果在执行这些语句时，其中一个导致错误，那么继续执行剩余的语句就没有意义了。相反，您尝试尽可能优雅地从错误中恢复。这在某种程度上类似于事务（但没有原子性）。

让我们来看一下没有异常处理的代码：

```js
function processFiles() {
    var fileNames = collectFileNames();
    var entries = extractAllEntries(fileNames);
    processEntries(entries);
}
function extractAllEntries(fileNames) {
    var allEntries = new Entries();
    fileNames.forEach(function (fileName) {
        var entry = extractOneEntry(fileName);
        allEntries.add(entry);  // (1)
    });
}
function extractOneEntry(fileName) {
    var file = openFile(fileName);  // (2)
    ...
}
...
```

在（2）处的`openFile()`中，对错误做出反应的最佳方法是什么？显然，不应再执行语句（1）。但我们也不想中止`extractAllEntries()`。相反，足够的是跳过当前文件并继续下一个。为此，我们在先前的代码中添加异常处理：

```js
function extractAllEntries(fileNames) {
    var allEntries = new Entries();
    fileNames.forEach(function (fileName) {
        try {
            var entry = extractOneEntry(fileName);
            allEntries.add(entry);
        } catch (exception) {  // (2)
            errorLog.log('Error in '+fileName, exception);
        }
    });
}
function extractOneEntry(fileName) {
    var file = openFile(fileName);
    ...
}
function openFile(fileName) {
    if (!exists(fileName)) {
        throw new Error('Could not find file '+fileName); // (1)
    }
    ...
}
```

异常处理有两个方面：

1.  如果在发生错误的地方无法有意义地处理问题，请抛出异常。

1.  找到可以处理错误的地方：捕获异常。

在（1）处，以下结构是活动的：

```js
    processFile()
        extractAllEntries(...)
            fileNames.forEach(...)
                function (fileName) { ... }
                    try { ... } catch (exception) { ... }
                        extractOneEntry(...)
                            openFile(...)
```

在（1）处的`throw`语句沿着树向上走，并离开所有结构，直到遇到一个活动的`try`语句。然后调用该语句的`catch`块并将异常值传递给它。

## JavaScript 中的异常处理

JavaScript 中的异常处理与大多数编程语言一样：`try`语句将语句分组，并允许您拦截这些语句中的异常。

### throw

`throw`的语法如下：

```js
throw «value»;
```

任何 JavaScript 值都可以被抛出。为了简单起见，许多 JavaScript 程序只抛出字符串：

```js
// Don't do this
if (somethingBadHappened) {
    throw 'Something bad happened';
}
```

不要这样做。JavaScript 有专门的异常对象构造函数（参见[错误构造函数](ch14.html#error_constructors "错误构造函数")）。使用它们或对其进行子类化（参见第二十八章）。它们的优势是 JavaScript 会自动添加堆栈跟踪（在大多数引擎上），并且它们有额外的上下文特定属性的空间。最简单的解决方案是使用内置构造函数`Error()`：

```js
if (somethingBadHappened) {
    throw new Error('Something bad happened');
}
```

### try-catch-finally

`try-catch-finally`的语法如下。`try`是必需的，`catch`和`finally`至少有一个也必须存在：

```js
try {
    «try_statements»
}
⟦catch («exceptionVar») {
   «catch_statements»
}⟧
⟦finally {
   «finally_statements»
}⟧
```

它是如何工作的：

+   `catch`捕获在`try_statements`中抛出的任何异常，无论是直接抛出还是在它们调用的函数中。提示：如果要区分不同类型的异常，可以使用`constructor`属性来切换异常的构造函数（请参阅[构造函数属性的用例](ch17_split_001.html#switch_constructor "构造函数属性的用例")）。

+   `finally`总是被执行，无论`try_statements`中发生了什么（或者它们调用的函数中发生了什么）。用它来进行应该始终执行的清理操作，无论`try_statements`中发生了什么：

    ```js
    var resource = allocateResource();
    try {
        ...
    } finally {
        resource.deallocate();
    }
    ```

如果`try_statements`中有一个`return`，则`try`块会在之后执行（在离开函数或方法之前立即执行；请参阅接下来的示例）。

### 例子

任何值都可以被抛出：

```js
function throwIt(exception) {
    try {
        throw exception;
    } catch (e) {
        console.log('Caught: '+e);
    }
}
```

以下是交互：

```js
> throwIt(3);
Caught: 3
> throwIt('hello');
Caught: hello
> throwIt(new Error('An error happened'));
Caught: Error: An error happened
```

`finally`总是被执行：

```js
function throwsError() {
    throw new Error('Sorry...');
}
function cleansUp() {
    try {
        throwsError();
    } finally {
        console.log('Performing clean-up');
    }
}
```

以下是交互：

```js
> cleansUp();
Performing clean-up
Error: Sorry...
```

`finally`在`return`语句之后执行：

```js
function idLog(x) {
    try {
        console.log(x);
        return 'result';
    } finally {
        console.log("FINALLY");
    }
}
```

以下是交互：

```js
> idLog('arg')
arg
FINALLY
'result'
```

在执行`finally`之前，返回值已排队：

```js
var count = 0;
function countUp() {
    try {
        return count;
    } finally {
        count++;  // (1)
    }
}
```

在执行语句（1）时，`count`的值已经排队返回：

```js
> countUp()
0
> count
1
```

## 错误构造函数

ECMAScript 标准化以下错误构造函数。描述摘自 ECMAScript 5 规范：

+   `Error`是错误的通用构造函数。这里提到的所有其他错误构造函数都是子构造函数。

+   `EvalError`“在本规范中当前未使用。此对象保留用于与本规范先前版本的兼容性。”

+   `RangeError`“表示数字值超出了允许的范围。”例如：

    ```js
    > new Array(-1)
    RangeError: Invalid array length
    ```

+   `ReferenceError`“表示检测到无效引用值。”通常，这是一个未知的变量。例如：

    ```js
    > unknownVariable
    ReferenceError: unknownVariable is not defined
    ```

+   `SyntaxError`“表示发生了解析错误”——例如，通过`eval()`解析代码时：

    ```js
    > eval('3 +')
    SyntaxError: Unexpected end of file
    ```

+   `TypeError`“表示操作数的实际类型与预期类型不同。”例如：

    ```js
    > undefined.foo
    TypeError: Cannot read property 'foo' of undefined
    ```

+   `URIError`“表示以与其定义不兼容的方式使用了全局 URI 处理函数之一。”例如：

    ```js
    > decodeURI('%2')
    URIError: URI malformed
    ```

以下是错误的属性：

`message`

错误消息。

`name`

错误的名称。

`stack`

堆栈跟踪。这是非标准的，但在许多平台上都可用，例如 Chrome，Node.js 和 Firefox。

## 堆栈跟踪

错误的常见来源要么是外部的（错误的输入，丢失的文件等），要么是内部的（程序中的错误）。特别是在后一种情况下，您将收到意外的异常并需要进行调试。通常情况下，您没有运行调试器。对于“手动”调试，有两条信息是有帮助的：

1.  数据：变量具有什么值？

1.  执行：异常发生在哪一行，活动的函数调用是什么？

您可以将第一项（数据）的一些内容放入消息或异常对象的属性中。第二项（执行）在许多 JavaScript 引擎上通过*堆栈跟踪*得到支持，这是在创建异常对象时调用堆栈的快照。以下示例打印堆栈跟踪：

```js
function catchit() {
    try {
        throwit();
    } catch(e) {
        console.log(e.stack); // print stack trace
    }
}
function throwit() {
    throw new Error('');
}
```

以下是交互：

```js
> catchit()
Error
    at throwit (~/examples/throwcatch.js:9:11)
    at catchit (~/examples/throwcatch.js:3:9)
    at repl:1:5
```

## 实现您自己的错误构造函数

如果您想要堆栈跟踪，您需要内置错误构造函数的服务。您可以使用现有构造函数并将自己的数据附加到其中。或者您可以创建一个子构造函数，其实例可以通过`instanceof`与其他错误构造函数的实例区分开来。然而，这样做（对于内置构造函数）是复杂的；请参阅第二十八章以了解如何做到这一点。

