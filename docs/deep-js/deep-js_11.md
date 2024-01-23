# 八、共享的可变状态的问题及如何避免它们

> 原文：[`exploringjs.com/deep-js/ch_shared-mutable-state.html`](https://exploringjs.com/deep-js/ch_shared-mutable-state.html)

* * *

+   8.1 什么是共享的可变状态，为什么有问题？

+   8.2 通过复制数据来避免共享

    +   8.2.1 复制如何帮助处理共享的可变状态？

+   8.3 通过非破坏性更新避免突变

    +   8.3.1 非破坏性更新如何帮助处理共享的可变状态？

+   8.4 通过使数据不可变来防止突变

    +   8.4.1 不可变性如何帮助处理共享的可变状态？

+   8.5 用于避免共享可变状态的库

    +   8.5.1 Immutable.js

    +   8.5.2 Immer

* * *

本章回答以下问题：

+   什么是共享的可变状态？

+   为什么这是有问题的？

+   如何避免它的问题？

### 8.1 什么是共享的可变状态，为什么有问题？

共享的可变状态工作如下：

+   如果两个或更多参与方可以更改相同的数据（变量、对象等）。

+   以及它们的生命周期是否重叠。

+   那么就有一种风险，即一个参与方的修改会阻止其他参与方正确工作。

请注意，此定义适用于函数调用、协作式多任务处理（例如 JavaScript 中的异步函数）等。在每种情况下风险是相似的。

以下代码是一个例子。这个例子并不现实，但它演示了风险并且容易理解：

```js
function logElements(arr) {
 while (arr.length > 0) {
 console.log(arr.shift());
 }
}

function main() {
 const arr = ['banana', 'orange', 'apple'];

 console.log('Before sorting:');
 logElements(arr);

 arr.sort(); // changes arr

 console.log('After sorting:');
 logElements(arr); // (A)
}
main();

// Output:
// 'Before sorting:'
// 'banana'
// 'orange'
// 'apple'
// 'After sorting:'
```

在这种情况下，有两个独立的参与方：

+   函数 `main()` 想要在对数组进行排序之前和之后记录日志。

+   函数 `logElements()` 记录其参数 `arr` 的元素，但在这样做时会将它们删除。

`logElements()` 打破了 `main()` 并导致它在 A 行记录一个空数组。

在本章的其余部分，我们将看看避免共享可变状态问题的三种方法：

+   通过复制数据来避免共享

+   通过非破坏性更新避免突变

+   通过使数据不可变来防止突变

特别是，我们将回到刚刚看到的例子并加以修复。

### 8.2 通过复制数据来避免共享

复制数据是避免共享数据的一种方式。

！[](../Images/6b17215a2abf7f098e996c026fba60fd.png)  **背景**

有关在 JavaScript 中复制数据的背景，请参考本书中以下两章：

+   §6 “复制对象和数组”

+   §14 “复制类的实例：`.clone()` vs. 复制构造函数”

#### 8.2.1 复制如何帮助处理共享的可变状态？

只要我们只是从共享状态中*读取*，我们就没有任何问题。在*修改*之前，我们需要通过复制来“取消共享”它（尽可能深入）。

*防御性复制* 是一种技术，总是在可能出现问题时进行复制。其目标是保持当前实体（函数、类等）的安全：

+   输入：复制（可能）共享的数据传递给我们，让我们可以使用该数据而不受外部实体的干扰。

+   输出：在向外部参与方暴露内部数据之前复制内部数据，意味着该参与方无法干扰我们的内部活动。

请注意，这些措施保护我们免受其他参与方的伤害，但它们也保护其他参与方免受我们的伤害。

接下来的章节将说明两种防御性复制的方式。

##### 8.2.1.1 复制共享输入

请记住，在本章开头的激励示例中，我们因为`logElements()`修改了它的参数`arr`而陷入了麻烦：

```js
function logElements(arr) {
 while (arr.length > 0) {
 console.log(arr.shift());
 }
}
```

让我们在这个函数中添加防御性复制：

```js
function logElements(arr) {
 arr = [...arr]; // defensive copy
 while (arr.length > 0) {
 console.log(arr.shift());
 }
}
```

现在，如果在`main()`中调用`logElements()`，它就不会再引起问题：

```js
function main() {
 const arr = ['banana', 'orange', 'apple'];

 console.log('Before sorting:');
 logElements(arr);

 arr.sort(); // changes arr

 console.log('After sorting:');
 logElements(arr); // (A)
}
main();

// Output:
// 'Before sorting:'
// 'banana'
// 'orange'
// 'apple'
// 'After sorting:'
// 'apple'
// 'banana'
// 'orange'
```

##### 8.2.1.2 复制暴露的内部数据

让我们从一个不复制暴露的内部数据的`StringBuilder`类开始（A 行）：

```js
class StringBuilder {
 _data = [];
 add(str) {
 this._data.push(str);
 }
 getParts() {
 // We expose internals without copying them:
 return this._data; // (A)
 }
 toString() {
 return this._data.join('');
 }
}
```

只要不使用`.getParts()`，一切都运行良好：

```js
const sb1 = new StringBuilder();
sb1.add('Hello');
sb1.add(' world!');
assert.equal(sb1.toString(), 'Hello world!');
```

然而，如果`.getParts()`的结果发生了变化（A 行），那么`StringBuilder`将无法正常工作：

```js
const sb2 = new StringBuilder();
sb2.add('Hello');
sb2.add(' world!');
sb2.getParts().length = 0; // (A)
assert.equal(sb2.toString(), ''); // not OK
```

解决方案是在暴露之前（A 行）防御性地复制内部的`._data`：

```js
class StringBuilder {
 this._data = [];
 add(str) {
 this._data.push(str);
 }
 getParts() {
 // Copy defensively
 return [...this._data]; // (A)
 }
 toString() {
 return this._data.join('');
 }
}
```

现在更改`.getParts()`的结果不再干扰`sb`的操作：

```js
const sb = new StringBuilder();
sb.add('Hello');
sb.add(' world!');
sb.getParts().length = 0;
assert.equal(sb.toString(), 'Hello world!'); // OK
```

### 8.3 通过非破坏性更新避免突变

如果我们只进行非破坏性地更新数据，就可以避免突变。

![](img/6b17215a2abf7f098e996c026fba60fd.png)  **背景**

有关更新数据的更多信息，请参见§7 “破坏性和非破坏性更新数据”。

#### 8.3.1 非破坏性更新如何帮助共享可变状态？

通过非破坏性更新，共享数据变得不成问题，因为我们从不改变共享的数据。（这仅在所有访问数据的人都这样做时才有效！）

有趣的是，复制数据变得非常简单：

```js
const original = {city: 'Berlin', country: 'Germany'};
const copy = original;
```

这是有效的，因为我们只进行非破坏性的更改，因此需要按需复制数据。

### 8.4 通过使数据不可变来防止突变

我们可以通过使数据不可变来防止共享数据的突变。

![](img/6b17215a2abf7f098e996c026fba60fd.png)  **背景**

有关如何使 JavaScript 中的数据不可变的背景，请参阅本书中的以下两章：

+   §10 “保护对象免受更改”

+   §15 “不可变集合的包装器”

#### 8.4.1 不可变性如何帮助共享可变状态？

如果数据是不可变的，它可以在没有任何风险的情况下共享。特别是，没有必要进行防御性复制。

![](img/c17f3239b9e1f5d335bb0adf8211d9d3.png)  **非破坏性更新是不可变数据的重要补充**

如果我们将这两者结合起来，不可变数据变得几乎和可变数据一样多样化，但没有相关的风险。

### 8.5 避免共享可变状态的库

JavaScript 有几个可用的库支持不可变数据和非破坏性更新。其中两个流行的是：

+   [Immutable.js](https://github.com/immutable-js/immutable-js)为列表、堆栈、集合、映射等提供不可变的数据结构。

+   [Immer](https://immerjs.github.io/immer/) 还支持对普通对象、数组、集合和映射进行不可变性和非破坏性更新。也就是说，不需要新的数据结构。

这些库将在接下来的两个部分中更详细地描述。

#### 8.5.1 Immutable.js

在其存储库中，[Immutable.js 库](https://github.com/immutable-js/immutable-js)被描述为：

> JavaScript 的不可变持久数据集，可以提高效率和简单性。

Immutable.js 提供不可变的数据结构，如：

+   `List`

+   `Stack`

+   `Set`（与 JavaScript 内置的`Set`不同）

+   `Map`（与 JavaScript 内置的`Map`不同）

+   等等。

在以下示例中，我们使用不可变的`Map`：

```js
import {Map} from 'immutable/dist/immutable.es.js';
const map0 = Map([
 [false, 'no'],
 [true, 'yes'],
]);

// We create a modified version of map0:
const map1 = map0.set(true, 'maybe');

// The modified version is different from the original:
assert.ok(map1 !== map0);
assert.equal(map1.equals(map0), false); // (A)

// We undo the change we just made:
const map2 = map1.set(true, 'yes');

// map2 is a different object than map0,
// but it has the same content
assert.ok(map2 !== map0);
assert.equal(map2.equals(map0), true); // (B)
```

注：

+   与修改接收器不同，“破坏性”操作（如`.set()`）返回修改后的副本。

+   要检查两个数据结构是否具有相同的内容，我们使用内置的`.equals()`方法（A 行和 B 行）。

#### 8.5.2 Immer

在其存储库中，[Immer 库](https://immerjs.github.io/immer/)被描述为：

> 通过对当前状态进行突变来创建下一个不可变状态。

Immer 有助于对（可能嵌套的）普通对象、数组、集合和映射进行非破坏性更新。也就是说，没有涉及自定义数据结构。

使用 Immer 看起来像这样：

```js
import {produce} from 'immer/dist/immer.module.js';

const people = [
 {name: 'Jane', work: {employer: 'Acme'}},
];

const modifiedPeople = produce(people, (draft) => {
 draft[0].work.employer = 'Cyberdyne';
 draft.push({name: 'John', work: {employer: 'Spectre'}});
});

assert.deepEqual(modifiedPeople, [
 {name: 'Jane', work: {employer: 'Cyberdyne'}},
 {name: 'John', work: {employer: 'Spectre'}},
]);
assert.deepEqual(people, [
 {name: 'Jane', work: {employer: 'Acme'}},
]);
```

原始数据存储在`people`中。`produce()`为我们提供了一个变量`draft`。我们假装这个变量是`people`，并使用通常会进行破坏性更改的操作。Immer 拦截这些操作。它不会改变`draft`，而是以非破坏性的方式改变`people`。结果被引用为`modifiedPeople`。作为奖励，它是深度不可变的。

`assert.deepEqual()`之所以有效是因为 Immer 返回普通对象和数组。

[评论](https://github.com/rauschma/deep-js/issues/8)
