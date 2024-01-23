# 12 属性的可枚举性

> 原文：[`exploringjs.com/deep-js/ch_enumerability.html`](https://exploringjs.com/deep-js/ch_enumerability.html)

* * *

+   12.1 可枚举性如何影响属性迭代构造

    +   12.1.1 只考虑可枚举属性的操作

    +   12.1.2 同时考虑可枚举和不可枚举属性的操作

    +   12.1.3 内省操作的命名规则

+   12.2 预定义和创建属性的可枚举性

+   12.3 可枚举性的用例

    +   12.3.1 用例：隐藏属性不被`for-in`循环处理

    +   12.3.2 用例：标记不需要被复制的属性

    +   12.3.3 将属性标记为私有

    +   12.3.4 隐藏自有属性不被`JSON.stringify()`处理

+   12.4 结论

* * *

可枚举性是对象属性的*属性*。在本章中，我们将更仔细地看看它是如何使用的，以及它如何影响`Object.keys()`和`Object.assign()`等操作。

![](img/6b17215a2abf7f098e996c026fba60fd.png) **必需知识：属性特性**

在本章中，您应该熟悉属性特性。如果不熟悉，请查看§9“属性特性：介绍”。

### 12.1 可枚举性如何影响属性迭代构造

为了演示各种操作受可枚举性的影响，我们使用以下对象`obj`，其原型是`proto`。

```js
const protoEnumSymbolKey = Symbol('protoEnumSymbolKey');
const protoNonEnumSymbolKey = Symbol('protoNonEnumSymbolKey');
const proto = Object.defineProperties({}, {
 protoEnumStringKey: {
 value: 'protoEnumStringKeyValue',
 enumerable: true,
 },
 [protoEnumSymbolKey]: {
 value: 'protoEnumSymbolKeyValue',
 enumerable: true,
 },
 protoNonEnumStringKey: {
 value: 'protoNonEnumStringKeyValue',
 enumerable: false,
 },
 [protoNonEnumSymbolKey]: {
 value: 'protoNonEnumSymbolKeyValue',
 enumerable: false,
 },
});

const objEnumSymbolKey = Symbol('objEnumSymbolKey');
const objNonEnumSymbolKey = Symbol('objNonEnumSymbolKey');
const obj = Object.create(proto, {
 objEnumStringKey: {
 value: 'objEnumStringKeyValue',
 enumerable: true,
 },
 [objEnumSymbolKey]: {
 value: 'objEnumSymbolKeyValue',
 enumerable: true,
 },
 objNonEnumStringKey: {
 value: 'objNonEnumStringKeyValue',
 enumerable: false,
 },
 [objNonEnumSymbolKey]: {
 value: 'objNonEnumSymbolKeyValue',
 enumerable: false,
 },
});
```

#### 12.1.1 只考虑可枚举属性的操作

表 2：忽略不可枚举属性的操作。

| 操作 |  | 字符串键 | 符号键 | 继承 |
| --- | --- | --- | --- | --- |
| `Object.keys()` | ES5 | `✔` | `✘` | `✘` |
| `Object.values()` | ES2017 | `✔` | `✘` | `✘` |
| `Object.entries()` | ES2017 | `✔` | `✘` | `✘` |
| 扩展`{...x}` ^([ES2018]) | `✔` | `✔` | `✘` |
| `Object.assign()` | ES6 | `✔` | `✔` | `✘` |
| `JSON.stringify()` | ES5 | `✔` | `✘` | `✘` |
| `for-in` | ES1 | `✔` | `✘` | `✔` |

以下操作（在 tbl. 2 中总结）只考虑可枚举属性：

+   `Object.keys()` ^([ES5]) 返回可枚举自有字符串键属性的键。

    ```js
    > Object.keys(obj)
    [ 'objEnumStringKey' ]
    ```

+   `Object.values()` ^([ES2017]) 返回可枚举自有字符串键属性的值。

    ```js
    > Object.values(obj)
    [ 'objEnumStringKeyValue' ]
    ```

+   `Object.entries()` ^([ES2017]) 返回可枚举自有字符串键属性的键值对。（请注意，`Object.fromEntries()`接受符号作为键，但只创建可枚举属性。）

    ```js
    > Object.entries(obj)
    [ [ 'objEnumStringKey', 'objEnumStringKeyValue' ] ]
    ```

+   扩展到对象字面量 ^([ES2018]) 只考虑自有可枚举属性（带有字符串键或符号键）。

    ```js
    > const copy = {...obj};
    > Reflect.ownKeys(copy)
    [ 'objEnumStringKey', objEnumSymbolKey ]
    ```

+   `Object.assign()` ^([ES6]) 只会复制可枚举的自有属性（带有字符串键或符号键）。

    ```js
    > const copy = Object.assign({}, obj);
    > Reflect.ownKeys(copy)
    [ 'objEnumStringKey', objEnumSymbolKey ]
    ```

+   `JSON.stringify()` ^([ES5]) 只会将可枚举的自有属性与字符串键字符串化。

    ```js
    > JSON.stringify(obj)
    '{"objEnumStringKey":"objEnumStringKeyValue"}'
    ```

+   `for-in` 循环 ^([ES1]) 遍历自有和继承的可枚举字符串键属性。

    ```js
    const propKeys = [];
    for (const propKey in obj) {
     propKeys.push(propKey);
    }
    assert.deepEqual(
     propKeys, ['objEnumStringKey', 'protoEnumStringKey']);
    ```

`for-in` 是唯一一个内置操作，其中可枚举性对继承属性很重要。所有其他操作只适用于自有属性。

#### 12.1.2 同时考虑可枚举和不可枚举属性的操作

表 3：同时考虑可枚举和不可枚举属性的操作。

| 操作 |  | 字符串键 | 符号键 | 继承 |
| --- | --- | --- | --- | --- |
| `Object.getOwnPropertyNames()` | ES5 | `✔` | `✘` | `✘` |
| `Object.getOwnPropertySymbols()` | ES6 | `✘` | `✔` | `✘` |
| `Reflect.ownKeys()` | ES6 | `✔` | `✔` | `✘` |
| `Object.getOwnPropertyDescriptors()` | ES2017 | `✔` | `✔` | `✘` |

以下操作（在 tbl. 3 中总结）考虑了可枚举和不可枚举的属性：

+   `Object.getOwnPropertyNames()` ^([ES5]) 列出所有自有字符串键属性的键。

    ```js
    > Object.getOwnPropertyNames(obj)
    [ 'objEnumStringKey', 'objNonEnumStringKey' ]
    ```

+   `Object.getOwnPropertySymbols()` ^([ES6]) 列出所有自有符号键属性的键。

    ```js
    > Object.getOwnPropertySymbols(obj)
    [ objEnumSymbolKey, objNonEnumSymbolKey ]
    ```

+   `Reflect.ownKeys()` ^([ES6]) 列出所有自有属性的键。

    ```js
    > Reflect.ownKeys(obj)
    [
     'objEnumStringKey',
     'objNonEnumStringKey',
     objEnumSymbolKey,
     objNonEnumSymbolKey
    ]
    ```

+   `Object.getOwnPropertyDescriptors()` ^([ES2017]) 列出所有自有属性的属性描述符。

    ```js
    > Object.getOwnPropertyDescriptors(obj)
    {
     objEnumStringKey: {
     value: 'objEnumStringKeyValue',
     writable: false,
     enumerable: true,
     configurable: false
     },
     objNonEnumStringKey: {
     value: 'objNonEnumStringKeyValue',
     writable: false,
     enumerable: false,
     configurable: false
     },
     [objEnumSymbolKey]: {
     value: 'objEnumSymbolKeyValue',
     writable: false,
     enumerable: true,
     configurable: false
     },
     [objNonEnumSymbolKey]: {
     value: 'objNonEnumSymbolKeyValue',
     writable: false,
     enumerable: false,
     configurable: false
     }
    }
    ```

#### 12.1.3 内省操作的命名规则

*内省*使程序能够在运行时检查值的结构。这是*元编程*：普通编程是关于编写程序；元编程是关于检查和/或更改程序。

在 JavaScript 中，常见的内省操作具有简短的名称，而很少使用的操作具有较长的名称。忽略不可枚举的属性是规范，这就是为什么执行这种操作的名称很短，而不执行这种操作的名称很长的原因：

+   `Object.keys()` 忽略不可枚举的属性。

+   `Object.getOwnPropertyNames()` 列出所有自有属性的字符串键。

然而，`Reflect` 方法（如 `Reflect.ownKeys()`）违反了这一规则，因为 `Reflect` 提供了更多与代理相关的“元”操作。

此外，自 ES6 以来，还有以下区别：

+   *属性键*要么是字符串，要么是符号。

+   *属性名称*是字符串键的属性键。

+   *属性符号*是符号键的属性键。

因此，`Object.keys()` 的更好名称现在将是 `Object.names()`。

### 12.2 预定义和创建属性的可枚举性

在本节中，我们将像这样缩写 `Object.getOwnPropertyDescriptor()`：

```js
const desc = Object.getOwnPropertyDescriptor.bind(Object);
```

大多数数据属性都是使用以下属性创建的：

```js
{
 writable: true,
 enumerable: false,
 configurable: true,
}
```

这包括：

+   赋值

+   对象字面量

+   公共类字段

+   `Object.fromEntries()`

最重要的不可枚举属性是：

+   内置类的原型属性

    ```js
    > desc(Object.prototype, 'toString').enumerable
    false
    ```

+   通过用户定义的类创建的原型属性

    ```js
    > desc(class {foo() {}}.prototype, 'foo').enumerable
    false
    ```

+   数组的属性`.length`：

    ```js
    > Object.getOwnPropertyDescriptor([], 'length')
    { value: 0, writable: true, enumerable: false, configurable: false }
    ```

+   字符串的属性`.length`（注意原始值的所有属性都是只读的）：

    ```js
    > Object.getOwnPropertyDescriptor('', 'length')
    { value: 0, writable: false, enumerable: false, configurable: false }
    ```

接下来我们将看一下可枚举性的使用案例，这将告诉我们为什么有些属性是可枚举的，而其他的不是。

### 12.3 可枚举性的使用案例

可枚举性是一个不一致的特性。它确实有用例，但总是有某种注意事项。在本节中，我们将看看使用案例和注意事项。

#### 12.3.1 使用案例：隐藏`for-in`循环中的属性

`for-in`循环遍历对象的*所有*可枚举的字符串键属性，包括自有的和继承的。因此，属性`enumerable`用于隐藏不应遍历的属性。这是在 ECMAScript 1 中引入可枚举性的原因。

一般来说，最好避免使用`for-in`。接下来的两个小节将解释为什么。以下函数将帮助我们演示`for-in`的工作原理。

```js
function listPropertiesViaForIn(obj) {
 const result = [];
 for (const key in obj) {
 result.push(key);
 }
 return result;
}
```

##### 12.3.1.1 使用`for-in`遍历对象的注意事项

`for-in`遍历所有属性，包括继承的属性：

```js
const proto = {enumerableProtoProp: 1};
const obj = {
 __proto__: proto,
 enumerableObjProp: 2,
};
assert.deepEqual(
 listPropertiesViaForIn(obj),
 ['enumerableObjProp', 'enumerableProtoProp']);
```

对于普通的普通对象，`for-in`不会看到继承的方法，比如`Object.prototype.toString()`，因为它们都是不可枚举的：

```js
const obj = {};
assert.deepEqual(
 listPropertiesViaForIn(obj),
 []);
```

在用户定义的类中，所有继承的属性也是不可枚举的，因此被忽略：

```js
class Person {
 constructor(first, last) {
 this.first = first;
 this.last = last;
 }
 getName() {
 return this.first + ' ' + this.last;
 }
}
const jane = new Person('Jane', 'Doe');
assert.deepEqual(
 listPropertiesViaForIn(jane),
 ['first', 'last']);
```

**结论：** 在对象中，`for-in`考虑了继承属性，我们通常希望忽略这些属性。因此最好将`for-of`循环与`Object.keys()`、`Object.entries()`等结合使用。

##### 12.3.1.2 使用`for-in`遍历数组的注意事项

数组和字符串中的自有属性`.length`是不可枚举的，因此被`for-in`忽略：

```js
> listPropertiesViaForIn(['a', 'b'])
[ '0', '1' ]
> listPropertiesViaForIn('ab')
[ '0', '1' ]
```

然而，通常不安全使用`for-in`来遍历数组的索引，因为它考虑了既继承的又是非索引的自有属性。以下示例演示了如果数组具有自有的非索引属性会发生什么：

```js
const arr1 = ['a', 'b'];
assert.deepEqual(
 listPropertiesViaForIn(arr1),
 ['0', '1']);

const arr2 = ['a', 'b'];
arr2.nonIndexProp = 'yes';
assert.deepEqual(
 listPropertiesViaForIn(arr2),
 ['0', '1', 'nonIndexProp']);
```

**结论：**不应该使用`for-in`来遍历数组的索引，因为它考虑索引属性和非索引属性：

+   如果您对数组的键感兴趣，请使用数组方法`.keys()`：

    ```js
    > [...['a', 'b', 'c'].keys()]
    [ 0, 1, 2 ]
    ```

+   如果要遍历数组的元素，请使用`for-of`循环，这样还可以与其他可迭代的数据结构一起使用。

#### 12.3.2 用例：标记不要复制的属性

通过使属性不可枚举，我们可以将它们从某些复制操作中隐藏起来。让我们首先检查两个历史复制操作，然后再转向更现代的复制操作。

##### 12.3.2.1 历史复制操作：Prototype 的`Object.extend()`

[Prototype](https://en.wikipedia.org/wiki/Prototype_JavaScript_Framework)是一个 JavaScript 框架，由 Sam Stephenson 于 2005 年 2 月创建，作为 Ruby on Rails 中 Ajax 支持的基础的一部分。

[Prototype 的`Object.extend(destination, source)`](http://api.prototypejs.org/language/Object/extend/)将`source`的所有可枚举的自有和继承属性复制到`destination`的自有属性中。[它的实现方式](https://github.com/prototypejs/prototype/blob/5fddd3e/src/prototype/lang/object.js#L88)如下：

```js
function extend(destination, source) {
 for (var property in source)
 destination[property] = source[property];
 return destination;
}
```

如果我们使用`Object.extend()`与一个对象，我们可以看到它将继承属性复制到自有属性中，并忽略不可枚举的属性（它还忽略符号键属性）。所有这些都是由`for-in`的工作方式决定的。

```js
const proto = Object.defineProperties({}, {
 enumProtoProp: {
 value: 1,
 enumerable: true,
 },
 nonEnumProtoProp: {
 value: 2,
 enumerable: false,
 },
});
const obj = Object.create(proto, {
 enumObjProp: {
 value: 3,
 enumerable: true,
 },
 nonEnumObjProp: {
 value: 4,
 enumerable: false,
 },
});

assert.deepEqual(
 extend({}, obj),
 {enumObjProp: 3, enumProtoProp: 1});
```

##### 12.3.2.2 历史复制操作：jQuery 的`$.extend()`

[jQuery 的`$.extend(target, source1, source2, ···)`](https://api.jquery.com/jquery.extend/)类似于`Object.extend()`：

+   它将`source1`的所有可枚举的自有和继承属性复制到`target`的自有属性中。

+   然后它对`source2`执行相同的操作。

+   等等。

##### 12.3.2.3 可枚举性驱动复制的缺点

基于可枚举性进行复制有几个缺点：

+   虽然可枚举性对于隐藏继承属性很有用，但主要是以这种方式使用，因为我们通常只想将自有属性复制到自有属性中。忽略继承属性可以更好地实现相同的效果。

+   要复制哪些属性通常取决于手头的任务；为所有用例提供单个标志很少有意义。更好的选择是提供一个带有*谓词*（返回布尔值的回调）的复制操作，告诉它何时忽略属性。

+   可枚举性在复制时方便地隐藏了数组的自有属性`.length`。但这是一个极为罕见的特例：一个既影响兄弟属性又受其影响的魔术属性。如果我们自己实现这种魔术，我们将使用（继承的）getter 和/或 setter，而不是（自有的）数据属性。

##### 12.3.2.4 `Object.assign()` ^([ES5])

在 ES6 中，[`Object.assign(target, source_1, source_2, ···)`](https://exploringjs.com/impatient-js/ch_single-objects.html#object.assign)可以用于将源合并到目标中。源的所有自有可枚举属性都会被考虑（无论是字符串键还是符号键）。`Object.assign()`使用“get”操作从源中读取值，并使用“set”操作将值写入目标。

关于可枚举性，`Object.assign()`延续了`Object.extend()`和`$.extend()`的传统。[引用 Yehuda Katz](https://mail.mozilla.org/pipermail/es-discuss/2012-October/025934.html)：

> `Object.assign`将为所有已经在流通中的`extend()`API 铺平道路。我们认为在这些情况下不复制可枚举方法的先例足以证明`Object.assign`具有这种行为的足够理由。

换句话说：`Object.assign()`是为了从`$.extend()`（以及类似的方法）升级而创建的。它的方法比`$.extend`更清晰，因为它忽略了继承的属性。

##### 12.3.2.5 非可枚举性在复制时有用的罕见例子

非可枚举性有助的情况很少。一个罕见的例子是[库`fs-extra`最近遇到的问题](https://github.com/jprichardson/node-fs-extra/issues/577)：

+   内置的 Node.js 模块`fs`有一个包含基于 Promise 的`fs` API 版本的对象的属性`.promises`。在问题出现时，读取`.promise`会导致以下警告被记录到控制台：

    ```js
    ExperimentalWarning: The fs.promises API is experimental
    ```

+   除了提供自己的功能外，`fs-extra`还将`fs`中的所有内容重新导出。对于 CommonJS 模块，这意味着将`fs`的所有属性复制到`fs-extra`的`module.exports`中（[通过`Object.assign()`](https://github.com/jprichardson/node-fs-extra/blob/master/lib/index.js)）。当`fs-extra`这样做时，就会触发警告。这很令人困惑，因为每次加载`fs-extra`时都会发生这种情况。

+   [一个快速的修复](https://github.com/nodejs/node/pull/20504)是将属性`fs.promises`设为不可枚举。之后，`fs-extra`就忽略了它。

#### 12.3.3 将属性标记为私有

如果我们使一个属性不可枚举，它就不能被`Object.keys()`、`for-in`循环等看到了。在这些机制方面，该属性是私有的。

然而，这种方法存在几个问题：

+   在复制对象时，我们通常希望复制私有属性。这与使不应该被复制的属性不可枚举相冲突（见上一节）。

+   属性并不是真正私有的。获取、设置和其他几种机制对可枚举和不可枚举的属性没有区别。

+   在处理代码时，无论是作为源代码还是交互式地，我们无法立即看到属性是否可枚举。命名约定（例如给属性名加上下划线前缀）更容易发现。

+   我们不能使用可枚举性来区分公共和私有方法，因为原型中的方法默认是不可枚举的。

#### 12.3.4 隐藏自有属性不被`JSON.stringify()`处理

`JSON.stringify()`在输出中不包括非可枚举的属性。因此，我们可以使用可枚举性来确定应该导出到 JSON 的自有属性。这个用例类似于前面的一个，将属性标记为私有。但也不同，因为这更多关于导出，并且应用了略微不同的考虑。例如：一个对象能否完全从 JSON 中重建？

作为可枚举性的替代方案，对象可以实现方法`.toJSON()`，并且`JSON.stringify()`会将该方法返回的内容字符串化，而不是对象本身。下一个例子演示了这是如何工作的。

```js
class Point {
 static fromJSON(json) {
 return new Point(json[0], json[1]);
 }
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
 toJSON() {
 return [this.x, this.y];
 }
}
assert.equal(
 JSON.stringify(new Point(8, -3)),
 '[8,-3]'
);
```

我觉得`toJSON()`比可枚举性更清晰。它还给了我们更多关于存储格式应该是什么样的自由。

### 12.4 结论

我们已经看到几乎所有非可枚举性的应用都是现在有其他更好的解决方案的变通方法。

对于我们自己的代码，我们通常可以假装可枚举性不存在：

+   通过对象字面量和赋值创建属性总是创建可枚举的属性。

+   通过类创建的原型属性总是不可枚举的。

也就是说，我们自动遵循最佳实践。

[评论](https://github.com/rauschma/deep-js/issues/12)
