# 9 属性属性：介绍

> 原文：[`exploringjs.com/deep-js/ch_property-attributes-intro.html`](https://exploringjs.com/deep-js/ch_property-attributes-intro.html)

* * *

+   9.1 对象的结构

    +   9.1.1 内部插槽

    +   9.1.2 属性键

    +   9.1.3 属性属性

+   9.2 属性描述符

+   9.3 检索属性的描述符

    +   9.3.1 `Object.getOwnPropertyDescriptor()`: 检索单个属性的描述符

    +   9.3.2 `Object.getOwnPropertyDescriptors()`: 检索对象所有属性的描述符

+   9.4 通过描述符定义属性

    +   9.4.1 `Object.defineProperty()`: 通过描述符定义单个属性

    +   9.4.2 `Object.defineProperties()`: 通过描述符定义多个属性

+   9.5 `Object.create()`: 通过描述符创建对象

+   9.6 `Object.getOwnPropertyDescriptors()`的用例

    +   9.6.1 用例：将属性复制到对象中

    +   9.6.2 `Object.getOwnPropertyDescriptors()`的用例：克隆对象

+   9.7 省略描述符属性

    +   9.7.1 在创建属性时省略描述符属性

    +   9.7.2 在更改属性时省略描述符属性

+   9.8 内置构造使用什么属性属性？

    +   9.8.1 通过赋值创建的自有属性

    +   9.8.2 通过对象字面量创建的自有属性

    +   9.8.3 数组的自有属性`.length`

    +   9.8.4 内置类的原型属性

    +   9.8.5 用户定义类的原型属性和实例属性

+   9.9 API：属性描述符

+   9.10 进一步阅读

* * *

在本章中，我们更详细地了解了 ECMAScript 规范如何看待 JavaScript 对象。特别是，在规范中，属性不是原子的，而是由多个*属性*（类似记录中的字段）组成。甚至数据属性的值也存储在属性中！

### 9.1 对象的结构

[在 ECMAScript 规范中](https://tc39.es/ecma262/#sec-object-type)，对象由以下组成：

+   *内部槽*是存储位置，无法从 JavaScript 访问，只能从规范中的操作访问。

+   *属性*的集合。每个属性将*键*与*属性*（类似于记录中的字段）关联起来。

#### 9.1.1 内部槽

[规范](https://tc39.es/ecma262/#sec-object-internal-methods-and-internal-slots)描述了内部槽如下。我添加了项目符号并强调了一部分：

+   内部槽对应于与对象关联并由各种 ECMAScript 规范算法使用的内部状态。

+   内部槽不是对象属性，也不会被继承。

+   根据特定的内部槽规范，这种状态可能包括：

    +   任何 ECMAScript 语言类型的值或

    +   特定 ECMAScript 规范类型值。

+   除非另有明确规定，否则内部槽将作为创建对象的过程的一部分分配，并且可能不会动态添加到对象中。

+   除非另有规定，内部槽的初始值为`undefined`。

+   本规范中的各种算法创建具有内部槽的对象。然而，**ECMAScript 语言没有直接的方法将内部槽与对象关联起来**。

+   在本规范中，内部方法和内部槽使用双方括号`[[ ]]`括起来的名称进行标识。

有两种类型的内部槽：

+   用于操作对象的方法槽（获取属性，设置属性等）。

+   存储值的数据槽。

普通对象具有以下数据槽：

+   `.[[Prototype]]: null | object`

    +   存储对象的原型。

    +   可以通过`Object.getPrototypeOf()`和`Object.setPrototypeOf()`间接访问。

+   `.[[Extensible]]: boolean`

    +   指示是否可以向对象添加属性。

    +   可以通过`Object.preventExtensions()`设置为`false`。

+   `.[[PrivateFieldValues]]: EntryList`

    +   用于管理[私有类字段](https://2ality.com/2019/07/private-class-fields.html)。

#### 9.1.2 属性键

属性的键可以是：

+   一个字符串

+   一个符号

#### 9.1.3 属性属性

有两种属性，它们的特征是它们的属性：

+   *数据属性*存储数据。它的属性`value`保存任何 JavaScript 值。

+   *访问器属性*由 getter 函数和/或 setter 函数组成。前者存储在属性`get`中，后者存储在属性`set`中。

此外，两种属性都具有的属性。以下表列出了所有属性及其默认值。

| 属性类型 | 属性名称和类型 | 默认值 |
| --- | --- | --- |
| 数据属性 | `value: any` | `undefined` |
|  | `writable: boolean` | `false` |
| 访问器属性 | `get: (this: any) => any` | `undefined` |
|  | `set: (this: any, v: any) => void` | `undefined` |
| 所有属性 | `configurable: boolean` | `false` |
|  | `enumerable: boolean` | `false` |

我们已经遇到了`value`，`get`和`set`属性。其他属性的工作方式如下：

+   `writable`确定数据属性的值是否可以更改。

+   `configurable`确定属性的属性是否可以更改。如果为`false`，则：

    +   我们不能删除属性。

    +   我们不能将属性从数据属性更改为访问器属性，反之亦然。

    +   我们不能更改除`value`之外的任何属性。

    +   但是，允许进行一次属性更改：我们可以将`writable`从`true`更改为`false`。这种异常背后的原因是[历史](https://stackoverflow.com/questions/9829817/why-can-i-set-enumerability-and-writability-of-unconfigurable-property-descrip/9843191#9843191)：数组的`.length`属性一直是可写的且不可配置的。允许更改其`writable`属性使我们能够冻结数组。

+   `enumerable`会影响一些操作（例如`Object.keys()`）。如果它为`false`，那么这些操作会忽略该属性。大多数属性是可枚举的（例如通过赋值或对象文字创建的属性），这就是为什么你很少会在实践中注意到这个属性。如果你仍然对它的工作原理感兴趣，请参见[§12“属性的可枚举性”]。

##### 9.1.3.1 陷阱：继承的不可写属性阻止通过赋值创建自有属性

如果继承的属性是不可写的，我们就不能使用赋值来创建具有相同键的自有属性：

```js
const proto = {
 prop: 1,
};
// Make proto.prop non-writable:
Object.defineProperty(
 proto, 'prop', {writable: false});

const obj = Object.create(proto);

assert.throws(
 () => obj.prop = 2,
 /^TypeError: Cannot assign to read only property 'prop'/);
```

有关更多信息，请参见[§11.3.4“继承的只读属性阻止通过赋值创建自有属性”]。

### 9.2 属性描述符

*属性描述符*将属性的属性编码为 JavaScript 对象。它们的 TypeScript 接口如下所示。

```js
interface DataPropertyDescriptor {
 value?: any;
 writable?: boolean;
 configurable?: boolean;
 enumerable?: boolean;
}
interface AccessorPropertyDescriptor {
 get?: (this: any) => any;
 set?: (this: any, v: any) => void;
 configurable?: boolean;
 enumerable?: boolean;
}
type PropertyDescriptor = DataPropertyDescriptor | AccessorPropertyDescriptor;
```

问号表示所有属性都是可选的。[§9.7“省略描述符属性”]描述了如果省略它们会发生什么。

### 9.3 检索属性的描述符

#### 9.3.1 `Object.getOwnPropertyDescriptor()`: 检索单个属性的描述符

考虑以下对象：

```js
const legoBrick = {
 kind: 'Plate 1x3',
 color: 'yellow',
 get description() {
 return `${this.kind} (${this.color})`;
 },
};
```

让我们首先获取数据属性`.color`的描述符：

```js
assert.deepEqual(
 Object.getOwnPropertyDescriptor(legoBrick, 'color'),
 {
 value: 'yellow',
 writable: true,
 enumerable: true,
 configurable: true,
 });
```

访问器属性`.description`的描述符如下所示：

```js
const desc = Object.getOwnPropertyDescriptor.bind(Object);
assert.deepEqual(
 Object.getOwnPropertyDescriptor(legoBrick, 'description'),
 {
 get: desc(legoBrick, 'description').get, // (A)
 set: undefined,
 enumerable: true,
 configurable: true
 });
```

在 A 行使用实用函数`desc()`可以确保`.deepEqual()`起作用。

#### 9.3.2 `Object.getOwnPropertyDescriptors()`: 检索对象的所有属性的描述符

```js
const legoBrick = {
 kind: 'Plate 1x3',
 color: 'yellow',
 get description() {
 return `${this.kind} (${this.color})`;
 },
};

const desc = Object.getOwnPropertyDescriptor.bind(Object);
assert.deepEqual(
 Object.getOwnPropertyDescriptors(legoBrick),
 {
 kind: {
 value: 'Plate 1x3',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 color: {
 value: 'yellow',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 description: {
 get: desc(legoBrick, 'description').get, // (A)
 set: undefined,
 enumerable: true,
 configurable: true,
 },
 });
```

在 A 行使用辅助函数`desc()`可以确保`.deepEqual()`起作用。

### 9.4 通过描述符定义属性

如果我们通过属性描述符`propDesc`定义具有键`k`的属性，那么发生的情况取决于：

+   如果没有键为`k`的属性，则会创建一个具有`propDesc`指定的属性的新自有属性。

+   如果存在键为`k`的属性，定义会更改属性的属性，使其与`propDesc`匹配。

#### 9.4.1 `Object.defineProperty()`: 通过描述符定义单个属性

首先，让我们通过描述符创建一个新属性：

```js
const car = {};

Object.defineProperty(car, 'color', {
 value: 'blue',
 writable: true,
 enumerable: true,
 configurable: true,
});

assert.deepEqual(
 car,
 {
 color: 'blue',
 });
```

接下来，我们通过描述符改变属性的种类；我们将数据属性转换为 getter：

```js
const car = {
 color: 'blue',
};

let readCount = 0;
Object.defineProperty(car, 'color', {
 get() {
 readCount++;
 return 'red';
 },
});

assert.equal(car.color, 'red');
assert.equal(readCount, 1);
```

最后，我们通过描述符改变数据属性的值：

```js
const car = {
 color: 'blue',
};

// Use the same attributes as assignment:
Object.defineProperty(
 car, 'color', {
 value: 'green',
 writable: true,
 enumerable: true,
 configurable: true,
 });

assert.deepEqual(
 car,
 {
 color: 'green',
 });
```

我们使用了与赋值相同的属性属性。

#### 9.4.2 `Object.defineProperties()`: 通过描述符定义多个属性

`Object.defineProperties()`是`Object.defineProperty()`的多属性版本：

```js
const legoBrick1 = {};
Object.defineProperties(
 legoBrick1,
 {
 kind: {
 value: 'Plate 1x3',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 color: {
 value: 'yellow',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 description: {
 get: function () {
 return `${this.kind} (${this.color})`;
 },
 enumerable: true,
 configurable: true,
 },
 });

assert.deepEqual(
 legoBrick1,
 {
 kind: 'Plate 1x3',
 color: 'yellow',
 get description() {
 return `${this.kind} (${this.color})`;
 },
 });
```

### 9.5 `Object.create()`: 通过描述符创建对象

`Object.create()`创建一个新对象。它的第一个参数指定该对象的原型。它的可选第二个参数指定该对象的属性的描述符。在下一个示例中，我们创建了与上一个示例中相同的对象。

```js
const legoBrick2 = Object.create(
 Object.prototype,
 {
 kind: {
 value: 'Plate 1x3',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 color: {
 value: 'yellow',
 writable: true,
 enumerable: true,
 configurable: true,
 },
 description: {
 get: function () {
 return `${this.kind} (${this.color})`;
 },
 enumerable: true,
 configurable: true,
 },
 });

// Did we really create the same object?
assert.deepEqual(legoBrick1, legoBrick2); // Yes!
```

### 9.6 `Object.getOwnPropertyDescriptors()`的用例

`Object.getOwnPropertyDescriptors()`在与`Object.defineProperties()`或`Object.create()`结合使用时，可以帮助我们处理两种用例。

#### 9.6.1 用例：将属性复制到对象中

自 ES6 以来，JavaScript 已经有了一个用于复制属性的工具方法：`Object.assign()`。但是，该方法使用简单的获取和设置操作来复制键为`key`的属性：

```js
target[key] = source[key];
```

这意味着只有在以下情况下才会创建属性的忠实副本：

+   它的属性`writable`为`true`，它的属性`enumerable`为`true`（因为这就是赋值创建属性的方式）。

+   它是一个数据属性。

以下示例说明了这个限制。对象`source`具有一个键为`data`的 setter。

```js
const source = {
 set data(value) {
 this._data = value;
 }
};

// Property `data` exists because there is only a setter
// but has the value `undefined`.
assert.equal('data' in source, true);
assert.equal(source.data, undefined);
```

如果我们使用`Object.assign()`来复制属性`data`，那么访问器属性`data`将被转换为数据属性：

```js
const target1 = {};
Object.assign(target1, source);

assert.deepEqual(
 Object.getOwnPropertyDescriptor(target1, 'data'),
 {
 value: undefined,
 writable: true,
 enumerable: true,
 configurable: true,
 });

// For comparison, the original:
const desc = Object.getOwnPropertyDescriptor.bind(Object);
assert.deepEqual(
 Object.getOwnPropertyDescriptor(source, 'data'),
 {
 get: undefined,
 set: desc(source, 'data').set,
 enumerable: true,
 configurable: true,
 });
```

幸运的是，使用`Object.getOwnPropertyDescriptors()`与`Object.defineProperties()`一起确实可以复制属性`data`：

```js
const target2 = {};
Object.defineProperties(
 target2, Object.getOwnPropertyDescriptors(source));

assert.deepEqual(
 Object.getOwnPropertyDescriptor(target2, 'data'),
 {
 get: undefined,
 set: desc(source, 'data').set,
 enumerable: true,
 configurable: true,
 });
```

##### 9.6.1.1 陷阱：复制使用`super`的方法

使用`super`的方法与其*home object*（存储在其中的对象）紧密相关。目前没有办法将这样的方法复制或移动到不同的对象。

#### 9.6.2 `Object.getOwnPropertyDescriptors()`的用例：克隆对象

浅克隆类似于复制属性，这就是为什么在这里使用`Object.getOwnPropertyDescriptors()`也是一个不错的选择。

要创建克隆，我们使用`Object.create()`：

```js
const original = {
 set data(value) {
 this._data = value;
 }
};

const clone = Object.create(
 Object.getPrototypeOf(original),
 Object.getOwnPropertyDescriptors(original));

assert.deepEqual(original, clone);
```

有关此主题的更多信息，请参阅§6“复制对象和数组”。

### 9.7 省略描述符属性

描述符的所有属性都是可选的。省略属性时会发生什么取决于操作。

#### 9.7.1 创建属性时省略描述符属性

当我们通过描述符创建新属性时，省略属性意味着使用它们的默认值：

```js
const car = {};
Object.defineProperty(
 car, 'color', {
 value: 'red',
 });
assert.deepEqual(
 Object.getOwnPropertyDescriptor(car, 'color'),
 {
 value: 'red',
 writable: false,
 enumerable: false,
 configurable: false,
 });
```

#### 9.7.2 更改属性时省略描述符属性

如果我们改变一个现有属性，那么省略描述符属性意味着相应的属性不受影响：

```js
const car = {
 color: 'yellow',
};
assert.deepEqual(
 Object.getOwnPropertyDescriptor(car, 'color'),
 {
 value: 'yellow',
 writable: true,
 enumerable: true,
 configurable: true,
 });
Object.defineProperty(
 car, 'color', {
 value: 'pink',
 });
assert.deepEqual(
 Object.getOwnPropertyDescriptor(car, 'color'),
 {
 value: 'pink',
 writable: true,
 enumerable: true,
 configurable: true,
 });
```

### 内置构造使用什么属性属性？

属性属性的一般规则（少数例外）是：

+   原型链开头的对象属性通常是可写的，可枚举的和可配置的。

+   如枚举性章节中所述，大多数继承属性都是不可枚举的，以隐藏它们，使它们不被`for-in`循环等遗留构造所发现。继承属性通常是可写的和可配置的。

#### 9.8.1 通过赋值创建的自有属性

```js
const obj = {};
obj.prop = 3;

assert.deepEqual(
 Object.getOwnPropertyDescriptors(obj),
 {
 prop: {
 value: 3,
 writable: true,
 enumerable: true,
 configurable: true,
 }
 });
```

#### 9.8.2 通过对象字面量创建的自有属性

```js
const obj = { prop: 'yes' };

assert.deepEqual(
 Object.getOwnPropertyDescriptors(obj),
 {
 prop: {
 value: 'yes',
 writable: true,
 enumerable: true,
 configurable: true
 }
 });
```

#### 9.8.3 数组的自有属性`.length`

数组的自有属性`.length`是不可枚举的，因此它不会被`Object.assign()`、扩展和类似操作复制。它也是不可配置的：

```js
> Object.getOwnPropertyDescriptor([], 'length')
{ value: 0, writable: true, enumerable: false, configurable: false }
> Object.getOwnPropertyDescriptor('abc', 'length')
{ value: 3, writable: false, enumerable: false, configurable: false }
```

`.length`是一个特殊的数据属性，它受其他自有属性（特别是索引属性）的影响。

#### 9.8.4 内置类的原型属性

```js
assert.deepEqual(
 Object.getOwnPropertyDescriptor(Array.prototype, 'map'),
 {
 value: Array.prototype.map,
 writable: true,
 enumerable: false,
 configurable: true
 });
```

#### 9.8.5 用户定义类的原型属性和实例属性

```js
class DataContainer {
 accessCount = 0;
 constructor(data) {
 this.data = data;
 }
 getData() {
 this.accessCount++;
 return this.data;
 }
}
assert.deepEqual(
 Object.getOwnPropertyDescriptors(DataContainer.prototype),
 {
 constructor: {
 value: DataContainer,
 writable: true,
 enumerable: false,
 configurable: true,
 },
 getData: {
 value: DataContainer.prototype.getData,
 writable: true,
 enumerable: false,
 configurable: true,
 }
 });
```

请注意，`DataContainer`实例的所有自有属性都是可写的，可枚举的和可配置的：

```js
const dc = new DataContainer('abc')
assert.deepEqual(
 Object.getOwnPropertyDescriptors(dc),
 {
 accessCount: {
 value: 0,
 writable: true,
 enumerable: true,
 configurable: true,
 },
 data: {
 value: 'abc',
 writable: true,
 enumerable: true,
 configurable: true,
 }
 });
```

### 9.9 API：属性描述符

以下工具方法使用属性描述符：

+   `Object.defineProperty(obj: object, key: string|symbol, propDesc: PropertyDescriptor): object` ^([ES5])

    在`obj`上创建或更改一个属性，其键为`key`，属性通过`propDesc`指定。返回修改后的对象。

    ```js
    const obj = {};
    const result = Object.defineProperty(
     obj, 'happy', {
     value: 'yes',
     writable: true,
     enumerable: true,
     configurable: true,
     });

    // obj was returned and modified:
    assert.equal(result, obj);
    assert.deepEqual(obj, {
     happy: 'yes',
    });
    ```

+   `Object.defineProperties(obj: object, properties: {[k: string|symbol]: PropertyDescriptor}): object` ^([ES5])

    `Object.defineProperty()`的批量版本。对象`properties`的每个属性`p`指定要添加到`obj`的一个属性：`p`的键指定属性的键，`p`的值是一个描述符，指定属性的属性。

    ```js
    const address1 = Object.defineProperties({}, {
     street: { value: 'Evergreen Terrace', enumerable: true },
     number: { value: 742, enumerable: true },
    });
    ```

+   `Object.create(proto: null|object, properties?: {[k: string|symbol]: PropertyDescriptor}): object` ^([ES5])

    首先创建一个原型为`proto`的对象。然后，如果提供了可选参数`properties`，则以与`Object.defineProperties()`相同的方式向其添加属性。最后返回结果。例如，以下代码片段产生与上一个片段相同的结果：

    ```js
    const address2 = Object.create(Object.prototype, {
     street: { value: 'Evergreen Terrace', enumerable: true },
     number: { value: 742, enumerable: true },
    });
    assert.deepEqual(address1, address2);
    ```

+   `Object.getOwnPropertyDescriptor(obj: object, key: string|symbol): undefined|PropertyDescriptor` ^([ES5])

    返回`obj`的自有（非继承）属性的描述符，其键为`key`。如果没有这样的属性，则返回`undefined`。

    ```js
    assert.deepEqual(
     Object.getOwnPropertyDescriptor(Object.prototype, 'toString'),
     {
     value: {}.toString,
     writable: true,
     enumerable: false,
     configurable: true,
     });
    assert.equal(
     Object.getOwnPropertyDescriptor({}, 'toString'),
     undefined);
    ```

+   `Object.getOwnPropertyDescriptors(obj: object): {[k: string|symbol]: PropertyDescriptor}` ^([ES2017])

    返回一个对象，其中`obj`的每个属性键`'k'`都映射到`obj.k`的属性描述符。结果可以用作`Object.defineProperties()`和`Object.create()`的输入。

    ```js
    const propertyKey = Symbol('propertyKey');
    const obj = {
     [propertyKey]: 'abc',
     get count() { return 123 },
    };

    const desc = Object.getOwnPropertyDescriptor.bind(Object);
    assert.deepEqual(
     Object.getOwnPropertyDescriptors(obj),
     {
     [propertyKey]: {
     value: 'abc',
     writable: true,
     enumerable: true,
     configurable: true
     },
     count: {
     get: desc(obj, 'count').get, // (A)
     set: undefined,
     enumerable: true,
     configurable: true
     }
     });
    ```

    在 A 行使用`desc()`是一个解决方法，使`.deepEqual()`起作用。

### 9.10 进一步阅读

接下来的三章将更详细地介绍属性特性：

+   §10 “保护对象免受更改”

+   §11 “属性：赋值 vs. 定义”

+   §12 “属性的可枚举性”

[评论](https://github.com/rauschma/deep-js/issues/9)
