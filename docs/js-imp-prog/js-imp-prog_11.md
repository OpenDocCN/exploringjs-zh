# 九、断言 API

> 原文：[`exploringjs.com/impatient-js/ch_assertion-api.html`](https://exploringjs.com/impatient-js/ch_assertion-api.html)

* * *

+   9.1 软件开发中的断言

+   9.2 本书中如何使用断言

    +   9.2.1 通过断言在代码示例中记录结果

    +   9.2.2 通过断言实现测试驱动的练习

+   9.3 普通比较 vs. 深度比较

+   9.4 快速参考：模块`assert`

    +   9.4.1 普通相等

    +   9.4.2 深度相等

    +   9.4.3 期望异常

    +   9.4.4 另一个工具函数

* * *

### 9.1 软件开发中的断言

在软件开发中，*断言*陈述关于值或代码片段的事实，这些事实必须为真。如果不是，将抛出异常。Node.js 通过其内置模块`assert`支持断言-例如：

```js
import * as assert from 'assert/strict';
assert.equal(3 + 5, 8);
```

此断言陈述了 3 加 5 的预期结果为 8。导入语句使用了`assert`的[推荐的`strict`版本](https://nodejs.org/api/assert.html#assert_strict_mode)。

### 9.2 本书中如何使用断言

在本书中，断言有两种用法：用于记录代码示例中的结果，以及实现测试驱动的练习。

#### 9.2.1 通过断言在代码示例中记录结果

在代码示例中，断言表达了预期的结果。例如，以下函数：

```js
function id(x) {
 return x;
}
```

`id()`返回其参数。我们可以通过断言展示它的作用：

```js
assert.equal(id('abc'), 'abc');
```

在示例中，我通常省略了导入`assert`的语句。

使用断言的动机是：

+   您可以明确指定预期的内容。

+   代码示例可以自动测试，这确保它们确实有效。

#### 9.2.2 通过断言实现测试驱动的练习

本书的练习是通过测试框架 Mocha 进行测试驱动的。测试中的检查是通过`assert`的方法进行的。

以下是这样一个测试的示例：

```js
// For the exercise, you must implement the function hello().
// The test checks if you have done it properly.
test('First exercise', () => {
 assert.equal(hello('world'), 'Hello world!');
 assert.equal(hello('Jane'), 'Hello Jane!');
 assert.equal(hello('John'), 'Hello John!');
 assert.equal(hello(''), 'Hello !');
});
```

有关更多信息，请参阅§10“开始使用测验和练习”。

### 9.3 普通比较 vs. 深度比较

严格的`equal()`使用`===`来比较值。因此，对象只等于自身-即使另一个对象具有相同的内容（因为`===`不比较对象的内容，只比较它们的标识）：

```js
assert.notEqual({foo: 1}, {foo: 1});
```

`deepEqual()`是比较对象的更好选择：

```js
assert.deepEqual({foo: 1}, {foo: 1});
```

这种方法也适用于数组：

```js
assert.notEqual(['a', 'b', 'c'], ['a', 'b', 'c']);
assert.deepEqual(['a', 'b', 'c'], ['a', 'b', 'c']);
```

### 9.4 模块`assert`的快速参考

有关完整文档，请参阅[Node.js 文档](https://nodejs.org/api/assert.html)。

#### 9.4.1 普通相等

+   `function equal(actual: any, expected: any, message?: string): void`

    `actual === expected`必须为`true`。如果不是，则抛出`AssertionError`。

    ```js
    assert.equal(3+3, 6);
    ```

+   `function notEqual(actual: any, expected: any, message?: string): void`

    `actual !== expected`必须为`true`。如果不是，则抛出`AssertionError`。

    ```js
    assert.notEqual(3+3, 22);
    ```

可选的最后一个参数`message`可用于解释所断言的内容。如果断言失败，将使用消息设置抛出的`AssertionError`。

```js
let e;
try {
 const x = 3;
 assert.equal(x, 8, 'x must be equal to 8')
} catch (err) {
 assert.equal(
 String(err),
 'AssertionError [ERR_ASSERTION]: x must be equal to 8');
}
```

#### 9.4.2 深度相等

+   `function deepEqual(actual: any, expected: any, message?: string): void`

    `actual`必须是深度等于`expected`。如果不是，则抛出`AssertionError`。

    ```js
    assert.deepEqual([1,2,3], [1,2,3]);
    assert.deepEqual([], []);

    // To .equal(), an object is only equal to itself:
    assert.notEqual([], []);
    ```

+   `function notDeepEqual(actual: any, expected: any, message?: string): void`

    `actual`必须不是深度等于`expected`。如果是，则抛出`AssertionError`。

    ```js
    assert.notDeepEqual([1,2,3], [1,2]);
    ```

#### 9.4.3 期望异常

如果你想要（或期望）收到一个异常，你需要使用`throws()`：这个函数调用它的第一个参数，函数`block`，只有在它抛出异常时才成功。可以使用额外的参数来指定异常的样子。

+   `function throws(block: Function, message?: string): void`

    ```js
    assert.throws(
     () => {
     null.prop;
     }
    );
    ```

+   `function throws(block: Function, error: Function, message?: string): void`

    ```js
    assert.throws(
     () => {
     null.prop;
     },
     TypeError
    );
    ```

+   `function throws(block: Function, error: RegExp, message?: string): void`

    ```js
    assert.throws(
     () => {
     null.prop;
     },
     /^TypeError: Cannot read properties of null \(reading 'prop'\)$/
    );
    ```

+   `function throws(block: Function, error: Object, message?: string): void`

    ```js
    assert.throws(
     () => {
     null.prop;
     },
     {
     name: 'TypeError',
     message: "Cannot read properties of null (reading 'prop')",
     }
    );
    ```

#### 9.4.4 另一个工具函数

+   `function fail(message: string | Error): never`

    每次调用时都会抛出一个`AssertionError`。这在单元测试中偶尔是有用的。

    ```js
    try {
     functionThatShouldThrow();
     assert.fail();
    } catch (_) {
     // Success
    }
    ```

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

请参阅测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/3)
