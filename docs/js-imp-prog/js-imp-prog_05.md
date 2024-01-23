# 四、新的 JavaScript 功能

> 原文：[`exploringjs.com/impatient-js/ch_new-javascript-features.html`](https://exploringjs.com/impatient-js/ch_new-javascript-features.html)

* * *

+   4.1 ECMAScript 2022 中的新功能

+   4.2 ECMAScript 2021 中的新功能

+   4.3 ECMAScript 2020 中的新功能

+   4.4 ECMAScript 2019 中的新功能

+   4.5 ECMAScript 2018 中的新功能

+   4.6 ECMAScript 2017 中的新功能

+   4.7 ECMAScript 2016 中的新功能

+   4.8 本章的来源

* * *

本章以逆序列出了 ES2016–ES2022 中的新功能。它在 ES2015（ES6）之后开始，因为该版本具有太多功能无法在此列出。

### 4.1 ECMAScript 2022 中的新功能

ES2022 可能会在 2022 年 6 月成为标准。以下提案已经达到了第 4 阶段，并计划成为该标准的一部分：

+   类的新成员：

    +   现在可以通过以下方式创建属性（公共槽）：

        +   实例公共字段

        +   静态公共字段

    +   私有槽是新的，可以通过以下方式创建：

        +   私有字段（实例私有字段和静态私有字段）

        +   私有方法和访问器（非静态和静态）

    +   静态初始化块

+   私有槽检查（“私有字段的人性化品牌检查”）：以下表达式检查`obj`是否具有私有槽`#privateSlot`：

    ```js
    #privateSlot in obj
    ```

+   模块中的顶层`await`：我们现在可以在模块的顶层使用`await`，不再需要进入异步函数或方法。

+   `error.cause`：`Error`及其子类现在让我们指定导致当前错误的错误：

    ```js
    new Error('Something went wrong', {cause: otherError})
    ```

+   索引值的方法`.at()`（ch_arrays.html#Array.prototype.at）允许我们在给定索引处读取元素（类似于括号运算符`[]`），并支持负索引（与括号运算符不同）。

    ```js
    > ['a', 'b', 'c'].at(0)
    'a'
    > ['a', 'b', 'c'].at(-1)
    'c'
    ```

    以下“可索引”类型具有方法`.at()`：

    +   `string`

    +   `Array`

    +   所有 Typed Array 类：`Uint8Array`等。

+   RegExp 匹配索引：如果我们向正则表达式添加一个标志，使用它会产生记录每个组捕获的开始和结束索引的匹配对象。

+   `Object.hasOwn(obj, propKey)`提供了一种安全的方式来检查对象`obj`是否具有键`propKey`的自有属性。与`Object.prototype.hasOwnProperty`相比，它适用于所有对象。

![](img/0ac255e56dc93a43365d8502301c8688.png) **ES2022 可能还会添加更多功能**

如果发生这种情况，本书将相应地进行更新。

### 4.2 ECMAScript 2021 中的新功能

以下功能是在 ECMAScript 2021 中添加的：

+   `String.prototype.replaceAll()`允许我们替换正则表达式或字符串的所有匹配项（`.replace()`只替换字符串的第一个出现）：

    ```js
    > 'abbbaab'.replaceAll('b', 'x')
    'axxxaax'
    ```

+   `Promise.any()`和`AggregateError`：`Promise.any()`返回一个 Promise，一旦可迭代的 Promise 中的第一个 Promise 被满足，它就会被满足。如果只有拒绝，它们将被放入一个`AggregateError`，成为拒绝值。

    当我们只对多个已完成的 Promise 中的第一个感兴趣时，我们使用`Promise.any()`。

+   逻辑赋值运算符：

    ```js
    a ||= b
    a &&= b
    a ??= b
    ```

+   在以下情况下使用下划线(`_`)作为分隔符：

    +   数字字面量: `123_456.789_012`

    +   大整数字面量: `6_000_000_000_000_000_000_000_000n`

+   WeakRefs: 这个特性超出了本书的范围。有关更多信息，请参阅[其提案](https://github.com/tc39/proposal-weakrefs)。

### 4.3 ECMAScript 2020 中的新功能

以下功能是在 ECMAScript 2020 中添加的：

+   新模块功能：

    +   通过`import()`动态导入：普通的`import`语句是静态的：我们只能在模块的顶层使用它，其模块说明符是一个固定的字符串。`import()`改变了这一点。它可以在任何地方使用（包括条件语句），并且我们可以计算其参数。

    +   `import.meta`包含当前模块的元数据。它的第一个广泛支持的属性是`import.meta.url`，其中包含当前模块文件的 URL 字符串。

    +   命名空间重新导出: 以下表达式将模块`'mod'`的所有导出导入到命名空间对象`ns`中，并导出该对象。

        ```js
        export * as ns from 'mod';
        ```

+   可选链式调用用于属性访问和方法调用。一个可选链式调用的例子是：

    ```js
    value.?prop
    ```

    如果`value`是`undefined`或`null`，则该表达式的值为`undefined`。否则，它的值为`value.prop`。这个特性在属性链中的一些属性可能缺失时特别有用。

+   空值合并运算符(`??`):

    ```js
    value ?? defaultValue
    ```

    如果`value`是`undefined`或`null`，则该表达式为`defaultValue`，否则为`value`。这个运算符让我们在某些东西缺失时使用默认值。

    以前在这种情况下使用逻辑或运算符(`||`)，但它在这里有缺点，因为每当左侧为假值时它就返回默认值（这并不总是正确的）。

+   大整数 - 任意精度整数: 大整数是一种新的原始类型。它支持可以任意大的整数（存储空间会根据需要增长）。

+   `String.prototype.matchAll()`: 如果标志`/g`未设置，则此方法会抛出异常，并返回给定字符串的所有匹配对象的可迭代对象。

+   `Promise.allSettled()`接收一个 Promise 的可迭代对象。它返回一个 Promise，一旦所有输入的 Promise 都被解决，就会被满足。满足值是一个数组，每个输入 Promise 对应一个对象，要么是：

    +   `{ status: 'fulfilled', value: «fulfillment value» }`

    +   `{ status: 'rejected', reason: «rejection value» }`

+   `globalThis`提供了一种访问全局对象的方式，可以在浏览器和 Node.js、Deno 等服务器端平台上使用。

+   `for-in`机制：这个特性超出了本书的范围。有关更多信息，请参阅[其提案](https://github.com/tc39/proposal-for-in-order)。

### 4.4 ECMAScript 2019 中的新功能

以下功能是在 ECMAScript 2019 中添加的：

+   数组方法`.flatMap()`类似于`.map()`，但让回调函数返回零个或多个值的数组，而不是单个值。然后返回的数组被连接在一起，成为`.flatMap()`的结果。使用案例包括：

    +   同时进行过滤和映射

    +   将单个输入值映射到多个输出值

+   数组方法`.flat()`将嵌套的数组转换为扁平数组。可选地，我们可以告诉它在哪个嵌套深度停止扁平化。

+   `Object.fromEntries()`从*entries*的可迭代对象中创建一个对象。每个条目都是一个包含属性键和属性值的两元素数组。

+   字符串方法：`.trimStart()`和`.trimEnd()`的工作方式类似于`.trim()`，但仅删除字符串的开头或结尾处的空格。

+   可选的`catch`绑定：如果我们不使用`catch`子句的参数，现在可以省略它。

+   `Symbol.prototype.description`是用于读取符号描述的 getter。以前，描述包含在`.toString()`的结果中，但无法单独访问。

这些新的 ES2019 功能超出了本书的范围：

+   JSON 超集：参见[2ality 博客文章](https://2ality.com/2019/01/json-superset.html)。

+   格式良好的`JSON.stringify()`：参见[2ality 博客文章](https://2ality.com/2019/01/well-formed-stringify.html)。

+   `Function.prototype.toString()`修订：参见[2ality 博客文章](https://2ality.com/2016/08/function-prototype-tostring.html)。

### 4.5 ECMAScript 2018 中的新功能

以下功能是在 ECMAScript 2018 中添加的：

+   异步迭代是同步迭代的异步版本。它基于 Promises：

    +   使用同步可迭代对象，我们可以立即访问每个项目。使用异步可迭代对象，我们必须在访问项目之前进行`await`。

    +   使用同步可迭代对象，我们使用`for-of`循环。使用异步可迭代对象，我们使用`for-await-of`循环。

+   扩展到对象文字中：通过在对象文字中使用扩展（`...`），我们可以将另一个对象的属性复制到当前对象中。一个用例是创建对象`obj`的浅拷贝：

    ```js
    const shallowCopy = {...obj};
    ```

+   剩余属性（解构）：在对象解构值时，我们现在可以使用剩余语法（`...`）来获取对象中所有未提及的属性。

    ```js
    const {a, ...remaining} = {a: 1, b: 2, c: 3};
    assert.deepEqual(remaining, {b: 2, c: 3});
    ```

+   `Promise.prototype.finally()`与 try-catch-finally 语句的`finally`子句相关联，类似于 Promise 方法`.then()`与`try`子句相关联，`.catch()`与`catch`子句相关联。

    换句话说：`.finally()`的回调无论 Promise 是被兑现还是被拒绝都会执行。

+   新的正则表达式功能：

    +   `RegExp`命名捕获组：除了通过编号访问组外，我们现在可以对它们进行命名并通过名称访问它们：

        ```js
        const matchObj = '---756---'.match(/(?<digits>[0-9]+)/)
        assert.equal(matchObj.groups.digits, '756');
        ```

    +   `RegExp`后瞻断言补充了前瞻断言：

        +   正向后瞻：`(?<=X)`匹配当前位置是否由`'X'`前导。

        +   负向后瞻：`(?<!X)`匹配当前位置是否不是由`'(?<!X)'`前导。

    +   正则表达式的`s`（`dotAll`）标志。如果此标志激活，则点匹配行终止符（默认情况下不匹配）。

    +   `RegExp` Unicode 属性转义在匹配一组 Unicode 代码点时给我们更多的权力，例如：

        ```js
        > /^\p{Lowercase_Letter}+$/u.test('aüπ')
        true
        > /^\p{White_Space}+$/u.test('\n \t')
        true
        > /^\p{Script=Greek}+$/u.test('ΩΔΨ')
        true
        ```

+   模板文字修订允许在标记模板中使用带反斜杠的文本，这在字符串文字中是非法的，例如：

    ```js
    windowsPath`C:\uuu\xxx\111`
    latex`\unicode`
    ```

### 4.6 ECMAScript 2017 中的新功能

以下功能是在 ECMAScript 2017 中添加的：

+   异步函数（`async/await`）让我们使用看起来同步的语法来编写异步代码。

+   `Object.values()`返回一个包含给定对象的所有可枚举字符串键属性的值的数组。

+   `Object.entries()`返回一个包含给定对象的所有可枚举字符串键属性的键值对的数组。每对都被编码为一个两元素数组。

+   字符串填充：字符串方法 `.padStart()` 和 `.padEnd()` 插入填充文本，直到接收者足够长：

    ```js
    > '7'.padStart(3, '0')
    '007'
    > 'yes'.padEnd(6, '!')
    'yes!!!'
    ```

+   函数参数列表和调用中的尾随逗号（ch_callables.html#trailing-commas-parameters）：自 ES3 起，数组文字中允许尾随逗号，自 ES5 起，对象文字中也允许。现在也允许在函数调用和方法调用中使用。

+   以下两个特性超出了本书的范围：

    +   `Object.getOwnPropertyDescriptors()`（参见 [“深入理解 JavaScript”](https://exploringjs.com/deep-js/ch_property-attributes-intro.html#Object.getOwnPropertyDescriptors)）

    +   共享内存和原子操作（参见 [提案](https://github.com/tc39/ecmascript_sharedmem)）

### 4.7 ECMAScript 2016 新特性

以下特性是在 ECMAScript 2016 中添加的：

+   `Array.prototype.includes()` 检查数组是否包含给定值。

+   指数运算符 (`**`):

    ```js
    > 4 ** 2
    16
    ```

### 4.8 本章来源

ECMAScript 特性列表来自 [TC39 关于已完成提案的页面](https://github.com/tc39/proposals/blob/master/finished-proposals.md)。

[评论](https://github.com/rauschma/impatient-js/issues/52)
