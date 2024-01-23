# 十一、属性：赋值 vs. 定义

> 原文：[`exploringjs.com/deep-js/ch_property-assignment-vs-definition.html`](https://exploringjs.com/deep-js/ch_property-assignment-vs-definition.html)

* * *

+   11.1 赋值 vs. 定义

    +   11.1.1 赋值

    +   11.1.2 定义

+   11.2 在理论中的赋值和定义（可选）

    +   11.2.1 分配给属性

    +   11.2.2 定义属性

+   11.3 实践中的定义和赋值

    +   11.3.1 只有定义允许我们创建具有任意属性的属性

    +   11.3.2 赋值操作符不会更改原型中的属性

    +   11.3.3 赋值调用 setter，定义不会

    +   11.3.4 继承的只读属性阻止通过赋值创建自有属性

+   11.4 哪些语言结构使用定义，哪些使用赋值？

    +   11.4.1 对象文字的属性是通过定义添加的

    +   11.4.2 赋值操作符`=`总是使用赋值

    +   11.4.3 公共类字段是通过定义添加的

+   11.5 进一步阅读和本章的来源

* * *

有两种方法可以创建或更改对象`obj`的属性`prop`：

+   *赋值*：`obj.prop = true`

+   *定义*：`Object.defineProperty(obj, '', {value: true})`

本章解释了它们的工作原理。

![](img/6b17215a2abf7f098e996c026fba60fd.png)  **必需知识：属性属性和属性描述符**

对于本章，您应该熟悉属性属性和属性描述符。如果您不熟悉，请查看§9“属性属性：介绍”。

### 11.1 赋值 vs. 定义

#### 11.1.1 赋值

我们使用赋值操作符`=`将值`value`分配给对象`obj`的属性`.prop`：

```js
obj.prop = value
```

这个操作符的工作方式取决于`.prop`的外观：

+   更改属性：如果存在自有数据属性`.prop`，赋值会将其值更改为`value`。

+   调用 setter：如果存在自有或继承的 setter`.prop`，赋值会调用该 setter。

+   创建属性：如果没有自有数据属性`.prop`，也没有自有或继承的 setter，赋值会创建一个新的自有数据属性。

也就是说，赋值的主要目的是进行更改。这就是为什么它支持 setter。

#### 11.1.2 定义

要定义对象`obj`的键`propKey`的属性，我们使用以下方法的操作：

```js
Object.defineProperty(obj, propKey, propDesc)
```

这种方法的工作方式取决于属性的外观：

+   更改属性：如果存在具有键`propKey`的自有属性，则定义将根据属性描述符`propDesc`（如果可能）更改其属性。

+   创建属性：否则，定义将创建一个具有`propDesc`指定属性的自有属性（如果可能）。

也就是说，定义的主要目的是创建一个自有属性（即使存在继承的 setter，它也会忽略）并改变属性的属性。

### 11.2 理论上的赋值和定义（可选）

![](img/c17f3239b9e1f5d335bb0adf8211d9d3.png) **ECMAScript 规范中的属性描述符**

在规范操作中，属性描述符不是 JavaScript 对象，而是[*Records*](https://tc39.es/ecma262/#sec-list-and-record-specification-type)，这是一个规范内部的数据结构，具有*fields*。字段的键用双括号括起来。例如，`Desc.[[Configurable]]`访问`Desc`的`.[[Configurable]]`字段。这些记录在与外部世界交互时会被转换为 JavaScript 对象。

#### 11.2.1 分配属性

通过[ECMAScript 规范中的以下操作](https://tc39.es/ecma262/#sec-ordinarysetwithowndescriptor)来处理属性的赋值工作：

```js
OrdinarySetWithOwnDescriptor(O, P, V, Receiver, ownDesc)
```

这些是参数：

+   `O`是当前正在访问的对象。

+   `P`是我们正在赋值的属性的键。

+   `V`是我们正在赋值的值。

+   `Receiver`是赋值开始的对象。

+   `ownDesc`是`O[P]`的描述符，如果该属性不存在则为`null`。

返回值是一个布尔值，指示操作是否成功。如本章后面所述，严格模式赋值如果`OrdinarySetWithOwnDescriptor()`失败会抛出`TypeError`。

这是算法的高级摘要：

+   它遍历`Receiver`的原型链，直到找到键为`P`的属性。遍历是通过递归调用`OrdinarySetWithOwnDescriptor()`来完成的。在递归过程中，`O`会改变并指向当前正在访问的对象，但`Receiver`保持不变。

+   根据遍历的结果，在`Receiver`（递归开始的地方）中创建一个自有属性，或者发生其他事情。

更详细地说，这个算法的工作方式如下：

+   如果`ownDesc`是`undefined`，那么我们还没有找到一个带有键`P`的属性：

    +   如果`O`有一个原型`parent`，那么我们返回`parent.[[Set]](P, V, Receiver)`。这将继续我们的搜索。该方法调用通常最终会递归调用`OrdinarySetWithOwnDescriptor()`。

    +   否则，我们对`P`的搜索失败了，并将`ownDesc`设置如下：

        ```js
        {
          [[Value]]: undefined, [[Writable]]: true,
          [[Enumerable]]: true, [[Configurable]]: true
        }
        ```

        有了这个`ownDesc`，下一个`if`语句将在`Receiver`中创建一个自有属性。

+   如果`ownDesc`指定了一个数据属性，那么我们已经找到了一个属性：

    +   如果`ownDesc.[[Writable]]`是`false`，则返回`false`。这意味着任何不可写的属性`P`（自有的或继承的）都会阻止赋值。

    +   让`existingDescriptor`为`Receiver.[[GetOwnProperty]](P)`。也就是说，检索赋值开始的属性的描述符。现在我们有：

        +   当前对象`O`和当前属性描述符`ownDesc`。

        +   原始对象`Receiver`和另一方面的原始属性描述符`existingDescriptor`。

    +   如果`existingDescriptor`不是`undefined`：

        +   （如果我们到了这里，那么我们仍然在原型链的开始处 - 只有在`Receiver`没有属性`P`时才会递归。）

        +   以下两个`if`条件永远不应该为`true`，因为`ownDesc`和`existingDesc`应该相等：

            +   如果`existingDescriptor`指定了一个访问器，则返回`false`。

            +   如果`existingDescriptor.[[Writable]]`是`false`，则返回`false`。

        +   返回`Receiver.[[DefineOwnProperty]](P, { [[Value]]: V })`。这个内部方法执行定义，我们用它来改变属性`Receiver[P]`的值。定义算法在下一小节中描述。

    +   否则：

        +   （如果我们到了这里，那么`Receiver`没有一个带有键`P`的自有属性。）

        +   返回 `CreateDataProperty(Receiver, P, V)`。([此操作](https://tc39.es/ecma262/#sec-createdataproperty) 在其第一个参数中创建自有数据属性。)

+   （如果我们到达这里，那么 `ownDesc` 描述的是自有或继承的访问器属性。）

+   让 `setter` 为 `ownDesc.[[Set]]`。

+   如果 `setter` 是 `undefined`，返回 `false`。

+   执行 `Call(setter, Receiver, «V»)`。[`Call()`](https://tc39.es/ecma262/#sec-call) 调用函数对象 `setter`，并将 `this` 设置为 `Receiver`，单个参数 `V`（规范中使用法文引号 `«»` 表示列表）。

+   返回 `true`。

##### 11.2.1.1 从赋值到 `OrdinarySetWithOwnDescriptor()` 的过程是如何进行的？

在不涉及解构的赋值中，涉及以下步骤：

+   在规范中，评估从[赋值表达式的运行时语义部分](https://tc39.es/ecma262/#sec-assignment-operators-runtime-semantics-evaluation)开始。该部分处理为匿名函数提供名称、解构等。

+   如果没有解构模式，则使用 [`PutValue()`](https://tc39.es/ecma262/#sec-putvalue) 进行赋值。

+   对于属性赋值，`PutValue()` 调用内部方法 `.[[Set]]()`。

+   对于普通对象，[`.[[Set]]()`](https://tc39.es/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-set-p-v-receiver) 调用 `OrdinarySet()`（调用 `OrdinarySetWithOwnDescriptor()`）并返回结果。

值得注意的是，在严格模式下，如果 `.[[Set]]()` 的结果为 `false`，`PutValue()` 会抛出 `TypeError`。

#### 11.2.2 定义属性

定义属性的实际工作是通过 ECMAScript 规范中的[以下操作](https://tc39.es/ecma262/#sec-validateandapplypropertydescriptor)处理的：

```js
ValidateAndApplyPropertyDescriptor(O, P, extensible, Desc, current)
```

参数是：

+   我们要定义属性的对象 `O`。这里有一个特殊的仅验证模式，其中 `O` 是 `undefined`。我们在这里忽略了这种模式。

+   我们要定义的属性的属性键 `P`。

+   `extensible` 表示 `O` 是否可扩展。

+   `Desc` 是指定属性所需的属性描述符。

+   如果存在，则 `current` 包含自有属性 `O[P]` 的属性描述符。否则，`current` 是 `undefined`。

操作的结果是一个布尔值，指示操作是否成功。失败可能会产生不同的后果。有些调用者会忽略结果。其他调用者，如 `Object.defineProperty()`，如果结果为 `false`，则会抛出异常。

这是算法的摘要：

+   如果 `current` 是 `undefined`，则属性 `P` 目前不存在，必须创建。

    +   如果 `extensible` 是 `false`，返回 `false` 表示无法添加属性。

    +   否则，检查 `Desc` 并创建数据属性或访问器属性。

    +   返回 `true`。

+   如果 `Desc` 没有任何字段，则返回 `true`，表示操作成功（因为不需要进行任何更改）。

+   如果 `current.[[Configurable]]` 是 `false`：

    +   （`Desc` 不允许改变除 `value` 之外的属性。）

    +   如果 `Desc.[[Configurable]]` 存在，则它必须与 `current.[[Configurable]]` 具有相同的值。如果不是，则返回 `false`。

    +   相同的检查：`Desc.[[Enumerable]]`

+   接下来，我们*验证*属性描述符 `Desc`：`current` 描述的属性是否可以更改为 `Desc` 指定的值？如果不能，返回 `false`。如果可以，继续。

    +   如果描述符是*通用*的（没有特定于数据属性或访问器属性的属性），则验证成功，我们可以继续。

    +   否则，如果一个描述符指定了数据属性，另一个指定了访问器属性：

        +   当前属性必须是可配置的（否则其属性无法按需更改）。如果不是，返回 `false`。

        +   将当前属性从数据属性更改为访问器属性，反之亦然。在这样做时，`.[[Configurable]]`和`.[[Enumerable]]`的值被保留，所有其他属性获得默认值（对象值属性为`undefined`，布尔值属性为`false`）。

    +   否则，如果两个描述符都指定数据属性：

        +   如果`current.[[Configurable]]`和`current.[[Writable]]`都为`false`，则不允许进行任何更改，`Desc`和`current`必须指定相同的属性：

            +   （由于`current.[[Configurable]]`为`false`，`Desc.[[Configurable]]`和`Desc.[[Enumerable]]`已经在先前检查过，并且具有正确的值。）

            +   如果`Desc.[[Writable]]`存在且为`true`，则返回`false`。

            +   如果`Desc.[[Value]]`存在且与`current.[[Value]]`的值不同，则返回`false`。

            +   没有其他事情可做。返回`true`表示算法成功。

            +   （请注意，通常情况下，我们不能更改非可配置属性的任何属性，除了它的值。这个规则的一个例外是，我们总是可以从可写变为不可写。该算法正确处理了这个例外。）

    +   否则（两个描述符都指定访问器属性）：

        +   如果`current.[[Configurable]]`为`false`，则不允许进行任何更改，`Desc`和`current`必须指定相同的属性：

            +   （由于`current.[[Configurable]]`为`false`，`Desc.[[Configurable]]`和`Desc.[[Enumerable]]`已经在先前检查过，并且具有正确的值。）

            +   如果`Desc.[[Set]]`存在，则它必须与`current.[[Set]]`具有相同的值。如果不是，则返回`false`。

            +   相同的检查：`Desc.[[Get]]`

            +   没有其他事情可做。返回`true`表示算法成功。

+   将属性`P`的属性设置为`Desc`指定的值。由于验证，我们可以确保所有更改都是允许的。

+   返回`true`。

### 11.3 定义和实践中的赋值

本节描述了属性定义和赋值的一些后果。

#### 11.3.1 只有定义允许我们创建具有任意属性的属性

如果我们通过赋值创建自有属性，它总是创建属性，其属性`writable`，`enumerable`和`configurable`都为`true`。

```js
const obj = {};
obj.dataProp = 'abc';
assert.deepEqual(
 Object.getOwnPropertyDescriptor(obj, 'dataProp'),
 {
 value: 'abc',
 writable: true,
 enumerable: true,
 configurable: true,
 });
```

因此，如果我们想指定任意属性，我们必须使用定义。

虽然我们可以在对象文字中创建 getter 和 setter，但我们不能通过赋值后来添加它们。在这里，我们也需要定义。

#### 11.3.2 赋值运算符不会更改原型中的属性

让我们考虑以下设置，其中`obj`从`proto`继承了属性`prop`。

```js
const proto = { prop: 'a' };
const obj = Object.create(proto);
```

我们不能通过给`obj.prop`赋值来（破坏性地）更改`proto.prop`。这样做会创建一个新的自有属性：

```js
assert.deepEqual(
 Object.keys(obj), []);

obj.prop = 'b';

// The assignment worked:
assert.equal(obj.prop, 'b');

// But we created an own property and overrode proto.prop,
// we did not change it:
assert.deepEqual(
 Object.keys(obj), ['prop']);
assert.equal(proto.prop, 'a');
```

这种行为的原因如下：原型可以具有其所有后代共享的属性值。如果我们只想在一个后代中更改这样的属性，我们必须通过覆盖来进行非破坏性地更改。然后，更改不会影响其他后代。

#### 11.3.3 赋值调用 setter，定义不会

定义`obj`的`.prop`属性与对其进行赋值有什么区别？

如果我们定义，那么我们的意图是要创建或更改`obj`的自有（非继承的）属性。因此，在以下示例中，定义忽略了`.prop`的继承的 setter：

```js
let setterWasCalled = false;
const proto = {
 get prop() {
 return 'protoGetter';
 },
 set prop(x) {
 setterWasCalled = true;
 },
};
const obj = Object.create(proto);

assert.equal(obj.prop, 'protoGetter');

// Defining obj.prop:
Object.defineProperty(
 obj, 'prop', { value: 'objData' });
assert.equal(setterWasCalled, false);

// We have overridden the getter:
assert.equal(obj.prop, 'objData');
```

相反，如果我们对`.prop`进行赋值，那么我们的意图通常是要更改已经存在的东西，并且这种更改应该由 setter 处理：

```js
let setterWasCalled = false;
const proto = {
 get prop() {
 return 'protoGetter';
 },
 set prop(x) {
 setterWasCalled = true;
 },
};
const obj = Object.create(proto);

assert.equal(obj.prop, 'protoGetter');

// Assigning to obj.prop:
obj.prop = 'objData';
assert.equal(setterWasCalled, true);

// The getter still active:
assert.equal(obj.prop, 'protoGetter');
```

#### 11.3.4 继承的只读属性阻止通过赋值创建自己的属性

如果原型中的`.prop`是只读的会发生什么？

```js
const proto = Object.defineProperty(
 {}, 'prop', {
 value: 'protoValue',
 writable: false,
 });
```

在从`proto`继承只读`.prop`的任何对象中，我们不能使用赋值来创建具有相同键的自有属性，例如：

```js
const obj = Object.create(proto);
assert.throws(
 () => obj.prop = 'objValue',
 /^TypeError: Cannot assign to read only property 'prop'/);
```

为什么我们不能赋值？理由是通过创建自己的属性来覆盖继承的属性可以被视为非破坏性地更改继承的属性。可以说，如果属性是不可写的，我们就不应该能够这样做。

然而，定义`.prop`仍然有效，并允许我们覆盖：

```js
Object.defineProperty(
 obj, 'prop', { value: 'objValue' });
assert.equal(obj.prop, 'objValue');
```

没有 setter 的访问器属性也被认为是只读的：

```js
const proto = {
 get prop() {
 return 'protoValue';
 }
};
const obj = Object.create(proto);
assert.throws(
 () => obj.prop = 'objValue',
 /^TypeError: Cannot set property prop of #<Object> which has only a getter$/);
```

![](img/290901f5575b7fa8e8b287bbaf550458.png) **“覆盖错误”：优缺点**

只读属性在原型链中较早阻止赋值的事实被称为*覆盖错误*：

+   它是在 ECMAScript 5.1 中引入的。

+   一方面，这种行为与原型继承和 setter 的工作方式一致。（因此，可以说这*不*是一个错误。）

+   另一方面，使用这种行为，深冻结全局对象会导致不希望的副作用。

+   曾经有尝试改变这种行为，但那破坏了 Lodash 库并被放弃了（[GitHub 上的拉取请求](https://github.com/tc39/ecma262/pull/1320#issuecomment-443485524)）。

+   背景知识：

    +   [GitHub 上的拉取请求](https://github.com/tc39/ecma262/pull/1307)

    +   [ECMAScript.org 上的维基页面](http://wiki.ecmascript.org/doku.php?id=strawman:fixing_override_mistake) ([存档](https://web.archive.org/web/20141230041441/http://wiki.ecmascript.org/doku.php?id=strawman:fixing_override_mistake))

### 11.4 语言构造使用定义，哪些使用赋值？

在本节中，我们检查语言何时使用定义以及何时使用赋值。我们通过跟踪是否调用继承的 setter 来检测使用的操作。有关更多信息，请参见§11.3.3 “赋值调用 setter，定义不调用”。

#### 11.4.1 对象文字的属性是通过定义添加的

当我们通过对象文字创建属性时，JavaScript 总是使用定义（因此从不调用继承的 setter）：

```js
let lastSetterArgument;
const proto = {
 set prop(x) {
 lastSetterArgument = x;
 },
};
const obj = {
 __proto__: proto,
 prop: 'abc',
};
assert.equal(lastSetterArgument, undefined);
```

#### 11.4.2 赋值运算符`=`总是使用赋值

赋值运算符`=`总是使用赋值来创建或更改属性。

```js
let lastSetterArgument;
const proto = {
 set prop(x) {
 lastSetterArgument = x;
 },
};
const obj = Object.create(proto);

// Normal assignment:
obj.prop = 'abc';
assert.equal(lastSetterArgument, 'abc');

// Assigning via destructuring:
[obj.prop] = ['def'];
assert.equal(lastSetterArgument, 'def');
```

#### 11.4.3 公共类字段是通过定义添加的

遗憾的是，即使公共类字段具有与赋值相同的语法，它们*不*使用赋值来创建属性，它们使用定义（就像对象文字中的属性一样）：

```js
let lastSetterArgument1;
let lastSetterArgument2;
class A {
 set prop1(x) {
 lastSetterArgument1 = x;
 }
 set prop2(x) {
 lastSetterArgument2 = x;
 }
}
class B extends A {
 prop1 = 'one';
 constructor() {
 super();
 this.prop2 = 'two';
 }
}
new B();

// The public class field uses definition:
assert.equal(lastSetterArgument1, undefined);
// Inside the constructor, we trigger assignment:
assert.equal(lastSetterArgument2, 'two');
```

### 11.5 本章的进一步阅读和来源

+   [“原型链”部分](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#prototype-chains) in “JavaScript for impatient programmers”

+   [Allen Wirfs-Brock 发送给 es-discuss 邮件列表的电子邮件](https://mail.mozilla.org/pipermail/es-discuss/2012-July/024227.html)：“赋值和定义之间的区别[…]在 ES 只有数据属性且 ES 代码无法操作属性属性时并不重要。”[这在 ECMAScript 5 中发生了变化。]

[评论](https://github.com/rauschma/deep-js/issues/11)
