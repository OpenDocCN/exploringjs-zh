# 二十九、ES6 中的类

> 原文：[`exploringjs.com/impatient-js/ch_classes.html`](https://exploringjs.com/impatient-js/ch_classes.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   29.1 速查表：类

+   29.2 类的基本要点

    +   29.2.1 一个人的类

    +   29.2.2 类表达式

    +   29.2.3 instanceof 运算符

    +   29.2.4 公共槽（属性）vs. 私有槽

    +   29.2.5 [更详细的私有槽[ES2022]（高级）](ch_classes.html#private-slots-in-more-detail-es2022-advanced)

    +   29.2.6 JavaScript 中类的优缺点

    +   29.2.7 使用类的技巧

+   29.3 类的内部

    +   29.3.1 类实际上是两个连接的对象

    +   29.3.2 类设置其实例的原型链

    +   29.3.3 `.__proto__` vs. `.prototype`

    +   29.3.4 `Person.prototype.constructor`（高级）

    +   29.3.5 分派 vs. 直接方法调用（高级）

    +   29.3.6 类从普通函数演变而来（高级）

+   29.4 类的原型成员

    +   29.4.1 公共原型方法和访问器

    +   29.4.2 [私有方法和访问器[ES2022]](ch_classes.html#private-methods-accessors)

+   29.5 [类的实例成员[ES2022]](ch_classes.html#instance-members-of-classes-es2022)

    +   29.5.1 实例公共字段

    +   29.5.2 实例私有字段

    +   29.5.3 ES2022 之前的实例私有数据（高级）

    +   29.5.4 通过 WeakMaps 模拟受保护的可见性和友元可见性（高级）

+   29.6 类的静态成员

    +   29.6.1 静态公共方法和访问器

    +   29.6.2 ES2022 中的静态公共字段

    +   29.6.3 ES2022 中的静态私有方法、访问器和字段

    +   29.6.4 [类中的静态初始化块[ES2022]](ch_classes.html#class-static-initialization-blocks)

    +   29.6.5 陷阱：使用`this`访问静态私有字段

    +   29.6.6 所有成员（静态的、原型的、实例的）都可以访问所有私有成员

    +   29.6.7 ES2022 之前的静态私有方法和数据

    +   29.6.8 静态工厂方法

+   29.7 子类化

    +   29.7.1 子类化的内部（高级）

    +   29.7.2 `instanceof`和子类化（高级）

    +   29.7.3 并非所有对象都是`Object`的实例（高级）

    +   29.7.4 内置对象的原型链（高级）

    +   29.7.5 混入类（高级）

+   29.8 `Object.prototype`的方法和访问器（高级）

    +   29.8.1 安全使用 `Object.prototype` 方法

    +   29.8.2 `Object.prototype.toString()`

    +   29.8.3 `Object.prototype.toLocaleString()`

    +   29.8.4 `Object.prototype.valueOf()`

    +   29.8.5 `Object.prototype.isPrototypeOf()`

    +   29.8.6 `Object.prototype.propertyIsEnumerable()`

    +   29.8.7 `Object.prototype.__proto__`（访问器）

    +   29.8.8 `Object.prototype.hasOwnProperty()`

+   29.9 常见问题：类

    +   29.9.1 为什么本书中称其为“实例私有字段”，而不是“私有实例字段”？](ch_classes.html#why-are-they-called-instance-private-fields-in-this-book-and-not-private-instance-fields)

    +   29.9.2 为什么标识符前缀是 `#`？为什么不通过 `private` 声明私有字段？](ch_classes.html#why-the-identifier-prefix-why-not-declare-private-fields-via-private)

* * *

在本书中，JavaScript 的面向对象编程（OOP）分为四个步骤介绍。本章涵盖了第 3 步和第 4 步，上一章涵盖了第 1 步和第 2 步。这些步骤如下（图 12）：

1.  **单个对象（上一章）：** JavaScript 的基本 OOP 构建块 *对象* 在孤立状态下是如何工作的？

1.  **原型链（上一章）：** 每个对象都有零个或多个 *原型对象* 的链。原型是 JavaScript 的核心继承机制。

1.  **类（本章）：** JavaScript 的 *类* 是对象的工厂。类与其实例之间的关系基于原型继承（第 2 步）。

1.  **子类化（本章）：** *子类* 与 *超类* 之间的关系也是基于原型继承的。

![图 12：本书以四个步骤介绍 JavaScript 中的面向对象编程。](img/1cbbf21bc1415ec3aaec2ea35fe128e8.png)

图 12：本书以四个步骤介绍 JavaScript 中的面向对象编程。

### 29.1 备忘单：类

超类：

```js
class Person {
 #firstName; // (A)
 constructor(firstName) {
 this.#firstName = firstName; // (B)
 }
 describe() {
 return `Person named ${this.#firstName}`;
 }
 static extractNames(persons) {
 return persons.map(person => person.#firstName);
 }
}
const tarzan = new Person('Tarzan');
assert.equal(
 tarzan.describe(),
 'Person named Tarzan'
);
assert.deepEqual(
 Person.extractNames([tarzan, new Person('Cheeta')]),
 ['Tarzan', 'Cheeta']
);
```

子类：

```js
class Employee extends Person {
 constructor(firstName, title) {
 super(firstName);
 this.title = title; // (C)
 }
 describe() {
 return super.describe() +
 ` (${this.title})`;
 }
}

const jane = new Employee('Jane', 'CTO');
assert.equal(
 jane.title,
 'CTO'
);
assert.equal(
 jane.describe(),
 'Person named Jane (CTO)'
);
```

注意：

+   `.#firstName` 是一个 *私有字段*，必须在初始化之前（行 B）进行声明（行 A）。

    +   私有字段只能在其所在的类内部访问。甚至子类也无法访问它。

+   `.title` 是一个属性，可以在没有先前声明的情况下进行初始化（行 C）。JavaScript 相对经常地将实例数据公开（与例如 Java 相反，后者更喜欢隐藏它）。

### 29.2 课程的要点

类基本上是一种紧凑的语法，用于设置原型链（在上一章中有解释）。在底层，JavaScript 的类是非常规的。但是当我们使用它们时，我们很少看到这一点。它们通常应该对已经使用过其他面向对象编程语言的人来说是熟悉的。

请注意，我们不需要类来创建对象。我们也可以通过对象字面量来创建对象。这就是为什么 JavaScript 不需要单例模式，而类的使用比许多其他具有类的语言中少。

#### 29.2.1 人的类

我们之前使用了 `jane` 和 `tarzan`，它们是代表人的单个对象。让我们使用 *类声明* 来实现这样的对象的工厂：

```js
class Person {
 #firstName; // (A)
 constructor(firstName) {
 this.#firstName = firstName; // (B)
 }
 describe() {
 return `Person named ${this.#firstName}`;
 }
 static extractNames(persons) {
 return persons.map(person => person.#firstName);
 }
}
```

现在可以通过 `new Person()` 创建 `jane` 和 `tarzan`：

```js
const jane = new Person('Jane');
const tarzan = new Person('Tarzan');
```

让我们来看看`Person`类的内部是什么。

+   `.constructor()` 是一个特殊的方法，在创建新实例之后调用。在其中，`this` 指的是该实例。

+   [ES2022] `.#firstName`是一个*实例私有字段*：这样的字段存储在实例中。它们的访问方式类似于属性，但它们的名称是分开的-它们总是以井号符号（`#`）开头。并且它们对类外部是不可见的：

    ```js
    assert.deepEqual(
     Reflect.ownKeys(jane),
     []
    );
    ```

    在我们可以在构造函数中初始化`.#firstName`（B 行）之前，我们需要在类主体中提到它来声明它（A 行）。

+   `.describe()`是一个方法。如果我们通过`obj.describe()`调用它，那么`this`在`.describe()`的主体内指的是`obj`。

    ```js
    assert.equal(
     jane.describe(), 'Person named Jane'
    );
    assert.equal(
     tarzan.describe(), 'Person named Tarzan'
    );
    ```

+   `.extractName()`是一个*静态*方法。“静态”意味着它属于类，而不属于实例：

    ```js
    assert.deepEqual(
     Person.extractNames([jane, tarzan]),
     ['Jane', 'Tarzan']
    );
    ```

我们还可以在构造函数中创建实例属性（公共字段）：

```js
class Container {
 constructor(value) {
 this.value = value;
 }
}
const abcContainer = new Container('abc');
assert.equal(
 abcContainer.value, 'abc'
);
```

与实例私有字段相反，实例属性不必在类主体中声明。

#### 29.2.2 类表达式

有两种*类定义*（定义类的方式）：

+   *类声明*，我们在上一节中看到的。

+   *类表达式*，我们将在下面看到。

类表达式可以是匿名的也可以是命名的：

```js
// Anonymous class expression
const Person = class { ··· };

// Named class expression
const Person = class MyClass { ··· };
```

命名类表达式的名称类似于命名函数表达式的名称：它只能在类主体内部访问，并且保持不变，无论该类分配给什么。

#### 29.2.3 `instanceof`运算符

`instanceof`运算符告诉我们一个值是否是给定类的实例：

```js
> new Person('Jane') instanceof Person
true
> {} instanceof Person
false
> {} instanceof Object
true
> [] instanceof Array
true
```

我们将在之后更详细地探讨`instanceof`运算符（在我们看完子类化之后）。

#### 29.2.4 公共槽（属性）vs. 私有槽

在 JavaScript 语言中，对象可以有两种“槽”。

+   *公共槽*（也称为*属性*）。例如，方法是公共槽。

+   *私有槽*[ES2022]。例如，私有字段是私有槽。

这些是我们需要了解有关属性和私有槽的最重要规则：

+   在类中，我们可以使用字段、方法、getter 和 setter 的公共和私有版本。它们都是对象中的槽。它们放置在哪些对象取决于是否使用关键字`static`和其他因素。

+   具有相同键的 getter 和 setter 创建一个单一的*访问器*槽。访问器也可以只有 getter 或只有 setter。

+   属性和私有槽非常不同-例如：

    +   它们被分开存储。

    +   它们的键是不同的。私有槽的键甚至不能直接访问（参见本章后面的§29.2.5.2“每个私有槽都有一个唯一的键（*私有名称*）”）。

    +   属性是从原型继承的，私有槽不是。

    +   私有槽只能通过类创建。

！[](../Images/b666ba365e94edaf0ef510fd7e12c7de.png) **有关属性和私有槽的更多信息**

本章不涵盖所有属性和私有槽的细节（只涵盖基本内容）。如果您想深入了解，可以在这里进行。

+   [§28.8.1“属性属性和属性描述符[ES5]”](ch_objects.html#property-attributes-property-descriptors)

+   ECMAScript 语言规范中的“对象内部方法和内部槽”一节解释了私有槽的工作原理。搜索“`[[PrivateElements]]`”。

以下类演示了两种槽。它的每个实例都有一个私有字段和一个属性：

```js
class MyClass {
 #instancePrivateField = 1;
 instanceProperty = 2;
 getInstanceValues() {
 return [
 this.#instancePrivateField,
 this.instanceProperty,
 ];
 }
}
const inst = new MyClass();
assert.deepEqual(
 inst.getInstanceValues(), [1, 2]
);
```

如预期的那样，在`MyClass`之外，我们只能看到属性：

```js
assert.deepEqual(
 Reflect.ownKeys(inst),
 ['instanceProperty']
);
```

接下来，我们将看一些私有槽的细节。

#### 29.2.5 更详细的私有槽[ES2022]（高级）

##### 29.2.5.1 私有槽不能在子类中访问

私有槽确实只能在其类的主体内部访问。我们甚至不能从子类访问它：

```js
class SuperClass {
 #superProp = 'superProp';
}
class SubClass extends SuperClass {
 getSuperProp() {
 return this.#superProp;
 }
}
// SyntaxError: Private field '#superProp'
// must be declared in an enclosing class
```

通过`extends`进行子类化在本章后面有解释。如何解决这个限制在§29.5.4 “通过 WeakMaps 模拟受保护的可见性和友元可见性”中有解释。

##### 29.2.5.2 每个私有槽都有一个唯一的键（*私有名称*）

私有槽具有类似于 symbols 的唯一键。考虑之前的以下类：

```js
class MyClass {
 #instancePrivateField = 1;
 instanceProperty = 2;
 getInstanceValues() {
 return [
 this.#instancePrivateField,
 this.instanceProperty,
 ];
 }
}
```

在内部，`MyClass`的私有字段大致处理如下：

```js
let MyClass;
{ // Scope of the body of the class
 const instancePrivateFieldKey = Symbol();
 MyClass = class {
 // Very loose approximation of how this
 // works in the language specification
 __PrivateElements__ = new Map([
 [instancePrivateFieldKey, 1],
 ]);
 instanceProperty = 2;
 getInstanceValues() {
 return [
 this.__PrivateElements__.get(instancePrivateFieldKey),
 this.instanceProperty,
 ];
 }
 }
}
```

`instancePrivateFieldKey`的值称为*私有名称*。我们不能直接在 JavaScript 中使用私有名称，我们只能间接使用它们，通过私有字段、私有方法和私有访问器的固定标识符。公共槽的固定标识符（如`getInstanceValues`）被解释为字符串键，私有槽的固定标识符（如`#instancePrivateField`）引用私有名称（类似于变量名称引用值）。

##### 29.2.5.3 相同的私有标识符在不同类中引用不同的私有名称

因为私有槽的标识符不被用作键，所以在不同类中使用相同的标识符会产生不同的槽（A 行和 C 行）：

```js
class Color {
 #name; // (A)
 constructor(name) {
 this.#name = name; // (B)
 }
 static getName(obj) {
 return obj.#name;
 }
}
class Person {
 #name; // (C)
 constructor(name) {
 this.#name = name;
 }
}

assert.equal(
 Color.getName(new Color('green')), 'green'
);

// We can’t access the private slot #name of a Person in line B:
assert.throws(
 () => Color.getName(new Person('Jane')),
 {
 name: 'TypeError',
 message: 'Cannot read private member #name from'
 + ' an object whose class did not declare it',
 }
);
```

##### 29.2.5.4 私有字段的名称永远不会冲突

即使子类使用相同的名称作为私有字段，这两个名称也永远不会冲突，因为它们引用私有名称（始终是唯一的）。在下面的示例中，`SuperClass`中的`.#privateField`与`SubClass`中的`.#privateField`不冲突，即使两个槽都直接存储在`inst`中：

```js
class SuperClass {
 #privateField = 'super';
 getSuperPrivateField() {
 return this.#privateField;
 }
}
class SubClass extends SuperClass {
 #privateField = 'sub';
 getSubPrivateField() {
 return this.#privateField;
 }
}
const inst = new SubClass();
assert.equal(
 inst.getSuperPrivateField(), 'super'
);
assert.equal(
 inst.getSubPrivateField(), 'sub'
);
```

通过`extends`进行子类化在本章后面有解释。

##### 29.2.5.5 使用`in`来检查对象是否具有给定的私有槽

`in`运算符可用于检查私有槽是否存在（A 行）：

```js
class Color {
 #name;
 constructor(name) {
 this.#name = name;
 }
 static check(obj) {
 return #name in obj; // (A)
 }
}
```

让我们看看`in`应用于私有槽的更多示例。

**私有方法。**以下代码显示私有方法在实例中创建私有槽：

```js
class C1 {
 #priv() {}
 static check(obj) {
 return #priv in obj;
 }
}
assert.equal(C1.check(new C1()), true);
```

**静态私有字段。**我们也可以使用`in`来检查静态私有字段：

```js
class C2 {
 static #priv = 1;
 static check(obj) {
 return #priv in obj;
 }
}
assert.equal(C2.check(C2), true);
assert.equal(C2.check(new C2()), false);
```

**静态私有方法。**我们也可以检查静态私有方法的槽：

```js
class C3 {
 static #priv() {}
 static check(obj) {
 return #priv in obj;
 }
}
assert.equal(C3.check(C3), true);
```

**在不同类中使用相同的私有标识符。**在下一个示例中，两个类`Color`和`Person`都有一个标识符为`#name`的槽。`in`运算符可以正确区分它们：

```js
class Color {
 #name;
 constructor(name) {
 this.#name = name;
 }
 static check(obj) {
 return #name in obj;
 }
}
class Person {
 #name;
 constructor(name) {
 this.#name = name;
 }
 static check(obj) {
 return #name in obj;
 }
}

// Detecting Color’s #name
assert.equal(
 Color.check(new Color()), true
);
assert.equal(
 Color.check(new Person()), false
);

// Detecting Person’s #name
assert.equal(
 Person.check(new Person()), true
);
assert.equal(
 Person.check(new Color()), false
);
```

#### 29.2.6 类在 JavaScript 中的优缺点

我建议使用类的原因如下：

+   类是对象创建和继承的常见标准，现在在许多库和框架中得到了广泛支持。与以前几乎每个框架都有自己的继承库相比，这是一个改进。

+   它们有助于 IDE 和类型检查器的工作，并在那里启用新功能。

+   如果您来自另一种语言到 JavaScript，并且习惯于类，那么您可以更快地开始。

+   JavaScript 引擎对它们进行了优化。也就是说，使用类的代码几乎总是比使用自定义继承库的代码更快。

+   我们可以对内置构造函数进行子类化，如`Error`。

这并不意味着类是完美的：

+   存在过度继承的风险。

+   在类中放入过多功能存在风险（其中一些功能通常最好放在函数中）。

+   类看起来对于来自其他语言的程序员来说很熟悉，但它们的工作方式和使用方式不同（见下一小节）。因此，存在这样的风险，即这些程序员编写的代码感觉不像 JavaScript。

+   类在表面上的工作方式与实际工作方式非常不同。换句话说，语法和语义之间存在断裂。两个例子是：

    +   类`C`内部的方法定义会在对象`C.prototype`中创建一个方法。

    +   类是函数。

    断开连接的动机是向后兼容性。幸运的是，这种断开在实践中几乎没有问题；如果我们按照类所假装的那样做，通常都没问题。

这是对类的第一次看法。我们很快会探索更多功能。

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：编写一个类**

`exercises/classes/point_class_test.mjs`

#### 29.2.7 使用类的提示

+   谨慎使用继承 - 它往往会使代码变得更加复杂，并将相关功能分散到多个位置。

+   通常最好使用外部函数和变量而不是静态成员。我们甚至可以通过不导出它们来将它们私有化到一个模块中。这个规则的两个重要例外是：

    +   需要访问私有插槽的操作

    +   静态工厂方法

+   只将核心功能放在原型方法中。其他功能最好通过函数实现 - 尤其是涉及多个类的实例的算法。

### 29.3 类的内部

#### 29.3.1 类实际上是两个连接的对象

在底层，一个类变成了两个连接的对象。让我们重新审视类`Person`，看看它是如何工作的：

```js
class Person {
 #firstName;
 constructor(firstName) {
 this.#firstName = firstName;
 }
 describe() {
 return `Person named ${this.#firstName}`;
 }
 static extractNames(persons) {
 return persons.map(person => person.#firstName);
 }
}
```

类创建的第一个对象存储在`Person`中。它有四个属性：

```js
assert.deepEqual(
 Reflect.ownKeys(Person),
 ['length', 'name', 'prototype', 'extractNames']
);

// The number of parameters of the constructor
assert.equal(
 Person.length, 1
);

// The name of the class
assert.equal(
 Person.name, 'Person'
);
```

剩下的两个属性是：

+   `Person.extractNames`是我们已经看到在操作中的静态方法。

+   `Person.prototype`指向由类定义创建的第二个对象。

这是`Person.prototype`的内容：

```js
assert.deepEqual(
 Reflect.ownKeys(Person.prototype),
 ['constructor', 'describe']
);
```

有两个属性：

+   `Person.prototype.constructor`指向构造函数。

+   `Person.prototype.describe`是我们已经使用过的方法。

#### 29.3.2 类设置其实例的原型链

对象`Person.prototype`是所有实例的原型：

```js
const jane = new Person('Jane');
assert.equal(
 Object.getPrototypeOf(jane), Person.prototype
);

const tarzan = new Person('Tarzan');
assert.equal(
 Object.getPrototypeOf(tarzan), Person.prototype
);
```

这解释了实例如何获得它们的方法：它们从对象`Person.prototype`继承。

图 13 可视化了一切是如何连接的。

![图 13：类 Person 具有属性.prototype，指向所有 Person 实例的原型对象。对象 jane 和 tarzan 是这样的实例。](img/c08fdf88503c70cdc845305d06d0c55a.png)

图 13：类`Person`具有属性`.prototype`，指向所有`Person`实例的原型对象。对象`jane`和`tarzan`是这样的实例。

#### 29.3.3 `.__proto__` vs. `.prototype`

很容易混淆`.__proto__`和`.prototype`。希望图 13 能清楚地说明它们的区别：

+   `.__proto__`是类`Object`的访问器，它让我们获取和设置其实例的原型。

+   `.prototype`是一个像其他任何属性一样的普通属性。它之所以特殊，只是因为`new`运算符使用它的值作为实例的原型。它的名称并不理想。一个不同的名称，比如`.instancePrototype`，更合适。

#### 29.3.4 `Person.prototype.constructor`（高级）

图 13 中有一个细节我们还没有看过：`Person.prototype.constructor`指回`Person`：

```js
> Person.prototype.constructor === Person
true
```

这个设置存在是因为向后兼容性。但它有两个额外的好处。

首先，每个类的实例都继承属性`.constructor`。因此，给定一个实例，我们可以通过它创建“相似”的对象：

```js
const jane = new Person('Jane');

const cheeta = new jane.constructor('Cheeta');
// cheeta is also an instance of Person
assert.equal(cheeta instanceof Person, true);
```

其次，我们可以获取创建给定实例的类的名称：

```js
const tarzan = new Person('Tarzan');
assert.equal(tarzan.constructor.name, 'Person');
```

#### 29.3.5 分派 vs. 直接方法调用（高级）

在本小节中，我们学习了两种不同的调用方法：

+   分派方法调用

+   直接方法调用

理解它们两个将为我们提供重要的方法工作见解。

我们还需要本章后面的第二种方法：它将允许我们从`Object.prototype`中借用有用的方法。

##### 29.3.5.1 分派方法调用

让我们来看看类的方法调用是如何工作的。我们重新访问之前的`jane`：

```js
class Person {
 #firstName;
 constructor(firstName) {
 this.#firstName = firstName;
 }
 describe() {
 return 'Person named '+this.#firstName;
 }
}
const jane = new Person('Jane');
```

图 14 有一个带有`jane`的原型链的图表。

![图 14：jane 的原型链以 jane 开始，然后继续到 Person.prototype。](img/e78cd85588d30424a17c527acfb55b69.png)

图 14：`jane`的原型链以`jane`开始，然后继续到`Person.prototype`。

普通方法调用是*分发*的-方法调用

```js
jane.describe()
```

发生在两个步骤中：

+   分发：JavaScript 遍历原型链，从`jane`开始查找具有键`'describe'`的自有属性的第一个对象：它首先查看`jane`，并没有找到自有属性`.describe`。它继续查看`jane`的原型，`Person.prototype`，并找到了一个自有属性`describe`，返回其值。

    ```js
    const func = jane.describe;
    ```

+   调用：方法调用一个值与函数调用一个值不同，它不仅调用括号前面的内容和括号内的参数，还将`this`设置为方法调用的接收者（在本例中为`jane`）：

    ```js
    func.call(jane);
    ```

这种动态查找方法并调用它的方式称为*动态分发*。

##### 29.3.5.2 直接方法调用

我们还可以*直接*进行方法调用，而无需分发：

```js
Person.prototype.describe.call(jane)
```

这一次，我们直接通过`Person.prototype.describe`指向方法，而不是在原型链中搜索它。我们还通过`.call()`不同地指定了`this`。

`this` 总是指向实例

无论实例的原型链中的方法位于何处，`this` 总是指向实例（原型链的开头）。这使得`.describe()`能够在示例中访问`.#firstName`。

直接方法调用何时有用？每当我们想要从其他地方借用一个给定对象没有的方法时，例如：

```js
const obj = Object.create(null);

// `obj` is not an instance of Object and doesn’t inherit
// its prototype method .toString()
assert.throws(
 () => obj.toString(),
 /^TypeError: obj.toString is not a function$/
);
assert.equal(
 Object.prototype.toString.call(obj),
 '[object Object]'
);
```

#### 29.3.6 类从普通函数演变而来（高级）

在 ECMAScript 6 之前，JavaScript 没有类。而是使用普通函数作为*构造函数*：

```js
function StringBuilderConstr(initialString) {
 this.string = initialString;
}
StringBuilderConstr.prototype.add = function (str) {
 this.string += str;
 return this;
};

const sb = new StringBuilderConstr('¡');
sb.add('Hola').add('!');
assert.equal(
 sb.string, '¡Hola!'
);
```

类提供了更好的语法来实现这一点：

```js
class StringBuilderClass {
 constructor(initialString) {
 this.string = initialString;
 }
 add(str) {
 this.string += str;
 return this;
 }
}
const sb = new StringBuilderClass('¡');
sb.add('Hola').add('!');
assert.equal(
 sb.string, '¡Hola!'
);
```

使用构造函数进行子类化特别棘手。类还提供了超出更方便的语法之外的好处：

+   内置构造函数（如`Error`）可以被子类化。

+   我们可以通过`super`访问被覆盖的属性。

+   类不能被函数调用。

+   方法不能被`new`调用，也没有`.prototype`属性。

+   支持私有实例数据。

+   还有更多。

类与构造函数非常兼容，甚至可以扩展它们：

```js
function SuperConstructor() {}
class SubClass extends SuperConstructor {}

assert.equal(
 new SubClass() instanceof SuperConstructor, true
);
```

`extends`和子类化在本章后面有解释。

##### 29.3.6.1 类就是构造函数

这给我们带来了一个有趣的见解。一方面，`StringBuilderClass`通过`StringBuilderClass.prototype.constructor`引用其构造函数。

另一方面，类*就是*构造函数（一个函数）：

```js
> StringBuilderClass.prototype.constructor === StringBuilderClass
true
> typeof StringBuilderClass
'function'
```

![](img/b666ba365e94edaf0ef510fd7e12c7de.png)  构造函数（函数）与类

由于它们的相似性，我可以互换使用术语*构造函数（函数）*和*类*。

### 29.4 类的原型成员

#### 29.4.1 公共原型方法和访问器

以下类声明体中的所有成员都创建了`PublicProtoClass.prototype`的属性。

```js
class PublicProtoClass {
 constructor(args) {
 // (Do something with `args` here.)
 }
 publicProtoMethod() {
 return 'publicProtoMethod';
 }
 get publicProtoAccessor() {
 return 'publicProtoGetter';
 }
 set publicProtoAccessor(value) {
 assert.equal(value, 'publicProtoSetter');
 }
}

assert.deepEqual(
 Reflect.ownKeys(PublicProtoClass.prototype),
 ['constructor', 'publicProtoMethod', 'publicProtoAccessor']
);

const inst = new PublicProtoClass('arg1', 'arg2');
assert.equal(
 inst.publicProtoMethod(), 'publicProtoMethod'
);
assert.equal(
 inst.publicProtoAccessor, 'publicProtoGetter'
);
inst.publicProtoAccessor = 'publicProtoSetter';
```

##### 29.4.1.1 各种公共原型方法和访问器（高级）

```js
const accessorKey = Symbol('accessorKey');
const syncMethodKey = Symbol('syncMethodKey');
const syncGenMethodKey = Symbol('syncGenMethodKey');
const asyncMethodKey = Symbol('asyncMethodKey');
const asyncGenMethodKey = Symbol('asyncGenMethodKey');

class PublicProtoClass2 {
 // Identifier keys
 get accessor() {}
 set accessor(value) {}
 syncMethod() {}
 * syncGeneratorMethod() {}
 async asyncMethod() {}
 async * asyncGeneratorMethod() {}

 // Quoted keys
 get 'an accessor'() {}
 set 'an accessor'(value) {}
 'sync method'() {}
 * 'sync generator method'() {}
 async 'async method'() {}
 async * 'async generator method'() {}

 // Computed keys
 get [accessorKey]() {}
 set accessorKey {}
 [syncMethodKey]() {}
 * [syncGenMethodKey]() {}
 async [asyncMethodKey]() {}
 async * [asyncGenMethodKey]() {}
}

// Quoted and computed keys are accessed via square brackets:
const inst = new PublicProtoClass2();
inst['sync method']();
inst[syncMethodKey]();
```

引用和计算键也可以在对象字面量中使用：

+   §28.7.1 “对象字面量中的引用键”

+   §28.7.2 “对象字面量中的计算键”

有关访问器（通过 getter 和/或 setter 定义）、生成器、异步方法和异步生成器方法的更多信息：

+   §28.3.6 “对象字面量：访问器”

+   §38 “同步生成器”

+   §41 “异步函数”

+   §42.2 “异步生成器”

#### 29.4.2 私有方法和访问器 [ES2022]

私有方法（和访问器）是原型成员和实例成员的有趣混合体。

一方面，私有方法存储在实例的槽中（A 行）：

```js
class MyClass {
 #privateMethod() {}
 static check() {
 const inst = new MyClass();
 assert.equal(
 #privateMethod in inst, true // (A)
 );
 assert.equal(
 #privateMethod in MyClass.prototype, false
 );
 assert.equal(
 #privateMethod in MyClass, false
 );
 }
}
MyClass.check();
```

为什么它们不存储在`.prototype`对象中？私有槽不会被继承，只有属性会被继承。

另一方面，私有方法在实例之间是共享的，就像原型公共方法一样：

```js
class MyClass {
 #privateMethod() {}
 static check() {
 const inst1 = new MyClass();
 const inst2 = new MyClass();
 assert.equal(
 inst1.#privateMethod,
 inst2.#privateMethod
 );
 }
}
```

由于它们的语法与原型公共方法相似，因此它们在这里进行了介绍。

以下代码演示了私有方法和访问器的工作原理：

```js
class PrivateMethodClass {
 #privateMethod() {
 return 'privateMethod';
 }
 get #privateAccessor() {
 return 'privateGetter';
 }
 set #privateAccessor(value) {
 assert.equal(value, 'privateSetter');
 }
 callPrivateMembers() {
 assert.equal(this.#privateMethod(), 'privateMethod');
 assert.equal(this.#privateAccessor, 'privateGetter');
 this.#privateAccessor = 'privateSetter';
 }
}
assert.deepEqual(
 Reflect.ownKeys(new PrivateMethodClass()), []
);
```

##### 29.4.2.1 各种私有方法和访问器（高级）

使用私有槽时，键始终是标识符：

```js
class PrivateMethodClass2 {
 get #accessor() {}
 set #accessor(value) {}
 #syncMethod() {}
 * #syncGeneratorMethod() {}
 async #asyncMethod() {}
 async * #asyncGeneratorMethod() {}
}
```

更多关于访问器（通过 getter 和/或 setter 定义）、生成器、异步方法和异步生成器方法的信息：

+   §28.3.6 “对象字面量：访问器”

+   §38 “同步生成器”

+   §41 “异步函数”

+   §42.2 “异步生成器”

### 29.5 实例类成员 [ES2022]

#### 29.5.1 实例公共字段

以下类的实例具有两个实例属性（在 A 行和 B 行创建）：

```js
class InstPublicClass {
 // Instance public field
 instancePublicField = 0; // (A)

 constructor(value) {
 // We don’t need to mention .property elsewhere!
 this.property = value; // (B)
 }
}

const inst = new InstPublicClass('constrArg');
assert.deepEqual(
 Reflect.ownKeys(inst),
 ['instancePublicField', 'property']
);
assert.equal(
 inst.instancePublicField, 0
);
assert.equal(
 inst.property, 'constrArg'
);
```

如果我们在构造函数中创建了一个实例属性（B 行），我们就不需要在其他地方“声明”它。正如我们已经看到的，对于实例私有字段来说是不同的。

请注意，实例属性在 JavaScript 中相对常见；比如在 Java 中，大多数实例状态都是私有的。

##### 29.5.1.1 具有带引号和计算键的实例公共字段（高级）

```js
const computedFieldKey = Symbol('computedFieldKey');
class InstPublicClass2 {
 'quoted field key' = 1;
 [computedFieldKey] = 2;
}
const inst = new InstPublicClass2();
assert.equal(inst['quoted field key'], 1);
assert.equal(inst[computedFieldKey], 2);
```

##### 29.5.1.2 实例公共字段中`this`的值是什么？（高级）

在实例公共字段的初始化程序中，`this`指的是新创建的实例：

```js
class MyClass {
 instancePublicField = this;
}
const inst = new MyClass();
assert.equal(
 inst.instancePublicField, inst
);
```

##### 29.5.1.3 实例公共字段何时执行？（高级）

实例公共字段的执行大致遵循这两条规则：

+   在基类（没有超类的类）中，实例公共字段在构造函数之前立即执行。

+   在派生类（具有超类的类）中：

    +   当`super()`被调用时，超类设置其实例槽。

    +   实例公共字段在`super()`之后立即执行。

以下示例演示了这些规则：

```js
class SuperClass {
 superProp = console.log('superProp');
 constructor() {
 console.log('super-constructor');
 }
}
class SubClass extends SuperClass {
 subProp = console.log('subProp');
 constructor() {
 console.log('BEFORE super()');
 super();
 console.log('AFTER super()');
 }
}
new SubClass();

// Output:
// 'BEFORE super()'
// 'superProp'
// 'super-constructor'
// 'subProp'
// 'AFTER super()'
```

`extends`和子类化在本章后面有解释。

#### 29.5.2 实例私有字段

以下类包含两个实例私有字段（A 行和 B 行）：

```js
class InstPrivateClass {
 #privateField1 = 'private field 1'; // (A)
 #privateField2; // (B) required!
 constructor(value) {
 this.#privateField2 = value; // (C)
 }
 /**
 * Private fields are not accessible outside the class body.
 */
 checkPrivateValues() {
 assert.equal(
 this.#privateField1, 'private field 1'
 );
 assert.equal(
 this.#privateField2, 'constructor argument'
 );
 }
}

const inst = new InstPrivateClass('constructor argument');
 inst.checkPrivateValues();

// No instance properties were created
assert.deepEqual(
 Reflect.ownKeys(inst),
 []
);
```

请注意，如果我们在类主体中声明了`.#privateField2`，那么我们只能在 C 行中使用它。

#### 29.5.3 ES2022 之前的私有实例数据（高级）

在本节中，我们将介绍两种保持实例数据私有的技术。因为它们不依赖于类，所以我们也可以用它们来处理其他方式创建的对象，比如通过对象字面量。

##### 29.5.3.1 ES6 之前：通过命名约定实现私有成员

第一种技术是通过在属性名称前加下划线使其私有化。这并不以任何方式保护属性；它只是向外界发出信号：“你不需要知道这个属性。”

在以下代码中，属性`._counter`和`._action`是私有的。

```js
class Countdown {
 constructor(counter, action) {
 this._counter = counter;
 this._action = action;
 }
 dec() {
 this._counter--;
 if (this._counter === 0) {
 this._action();
 }
 }
}

// The two properties aren’t really private:
assert.deepEqual(
 Object.keys(new Countdown()),
 ['_counter', '_action']);
```

使用这种技术，我们不会得到任何保护，私有名称可能会冲突。但好处是，它很容易使用。

私有方法的工作方式类似：它们是以下划线开头的普通方法。

##### 29.5.3.2 ES6 及以后：通过 WeakMaps 实现私有实例数据

我们也可以通过 WeakMaps 管理私有实例数据：

```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
 constructor(counter, action) {
 _counter.set(this, counter);
 _action.set(this, action);
 }
 dec() {
 let counter = _counter.get(this);
 counter--;
 _counter.set(this, counter);
 if (counter === 0) {
 _action.get(this)();
 }
 }
}

// The two pseudo-properties are truly private:
assert.deepEqual(
 Object.keys(new Countdown()),
 []);
```

关于它的工作原理的详细解释在 WeakMaps 章节中有说明。

这种技术为我们提供了相当大的保护，防止外部访问和名称冲突。但它也更复杂一些。

通过控制谁可以访问 `_superProp` 来控制伪属性 `_superProp` 的可见性——例如：如果变量存在于模块内并且未导出，则模块内的所有人都可以访问它，模块外的任何人都无法访问它。换句话说：在这种情况下，隐私的范围不是类，而是模块。不过，我们可以缩小范围：

```js
let Countdown;
{ // class scope
 const _counter = new WeakMap();
 const _action = new WeakMap();

 Countdown = class {
 // ···
 }
}
```

这种技术实际上不支持私有方法。但是，具有对 `_superProp` 的访问权限的模块局部函数是下一个最好的选择：

```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
 constructor(counter, action) {
 _counter.set(this, counter);
 _action.set(this, action);
 }
 dec() {
 privateDec(this);
 }
}

function privateDec(_this) { // (A)
 let counter = _counter.get(_this);
 counter--;
 _counter.set(_this, counter);
 if (counter === 0) {
 _action.get(_this)();
 }
}
```

注意，`this` 变成了显式函数参数 `_this`（A 行）。

#### 29.5.4 通过 WeakMaps 模拟受保护的可见性和友元可见性（高级）

如前所述，实例私有字段只在其类内部可见，甚至在子类中也不可见。因此，没有内置的方法可以获取：

+   受保护的可见性：一个类及其所有子类可以访问一个实例数据。

+   友元可见性：一个类及其“友元”（指定的函数、对象或类）可以访问一个实例数据。

在前一小节中，我们通过 WeakMaps 模拟了“模块可见性”（模块内的所有人都可以访问一个实例数据）。因此：

+   如果我们将一个类及其子类放入同一个模块中，我们就会得到受保护的可见性。

+   如果我们将一个类及其友元放入同一个模块中，我们就会得到友元可见性。

下一个示例演示了受保护的可见性：

```js
const _superProp = new WeakMap();
class SuperClass {
 constructor() {
 _superProp.set(this, 'superProp');
 }
}
class SubClass extends SuperClass {
 getSuperProp() {
 return _superProp.get(this);
 }
}
assert.equal(
 new SubClass().getSuperProp(),
 'superProp'
);
```

通过 `extends` 进行子类化 在本章后面有解释。

### 29.6 类的静态成员

#### 29.6.1 静态公共方法和访问器

在以下类声明的主体中，所有成员都创建所谓的 *静态* 属性——`StaticClass` 本身的属性。

```js
class StaticPublicMethodsClass {
 static staticMethod() {
 return 'staticMethod';
 }
 static get staticAccessor() {
 return 'staticGetter';
 }
 static set staticAccessor(value) {
 assert.equal(value, 'staticSetter');
 }
}
assert.equal(
 StaticPublicMethodsClass.staticMethod(), 'staticMethod'
);
assert.equal(
 StaticPublicMethodsClass.staticAccessor, 'staticGetter'
);
StaticPublicMethodsClass.staticAccessor = 'staticSetter';
```

##### 29.6.1.1 所有种类的静态公共方法和访问器（高级）

```js
const accessorKey = Symbol('accessorKey');
const syncMethodKey = Symbol('syncMethodKey');
const syncGenMethodKey = Symbol('syncGenMethodKey');
const asyncMethodKey = Symbol('asyncMethodKey');
const asyncGenMethodKey = Symbol('asyncGenMethodKey');

class StaticPublicMethodsClass2 {
 // Identifier keys
 static get accessor() {}
 static set accessor(value) {}
 static syncMethod() {}
 static * syncGeneratorMethod() {}
 static async asyncMethod() {}
 static async * asyncGeneratorMethod() {}

 // Quoted keys
 static get 'an accessor'() {}
 static set 'an accessor'(value) {}
 static 'sync method'() {}
 static * 'sync generator method'() {}
 static async 'async method'() {}
 static async * 'async generator method'() {}

 // Computed keys
 static get [accessorKey]() {}
 static set accessorKey {}
 static [syncMethodKey]() {}
 static * [syncGenMethodKey]() {}
 static async [asyncMethodKey]() {}
 static async * [asyncGenMethodKey]() {}
}

// Quoted and computed keys are accessed via square brackets:
StaticPublicMethodsClass2['sync method']();
StaticPublicMethodsClass2[syncMethodKey]();
```

引号和计算键也可以在对象字面量中使用：

+   §28.7.1 “对象字面量中的引号键”

+   §28.7.2 “对象字面量中的计算键”

有关访问器（通过 getter 和/或 setter 定义）、生成器、异步方法和异步生成器方法的更多信息：

+   §28.3.6 “对象字面量：访问器”

+   §38 “同步生成器”

+   §41 “异步函数”

+   §42.2 “异步生成器”

#### 29.6.2 静态公共字段 [ES2022]

以下代码演示了静态公共字段。`StaticPublicFieldClass` 有三个静态公共字段：

```js
const computedFieldKey = Symbol('computedFieldKey');
class StaticPublicFieldClass {
 static identifierFieldKey = 1;
 static 'quoted field key' = 2;
 static [computedFieldKey] = 3;
}

assert.deepEqual(
 Reflect.ownKeys(StaticPublicFieldClass),
 [
 'length', // number of constructor parameters
 'name', // 'StaticPublicFieldClass'
 'prototype',
 'identifierFieldKey',
 'quoted field key',
 computedFieldKey,
 ],
);

assert.equal(StaticPublicFieldClass.identifierFieldKey, 1);
assert.equal(StaticPublicFieldClass['quoted field key'], 2);
assert.equal(StaticPublicFieldClass[computedFieldKey], 3);
```

#### 29.6.3 静态私有方法、访问器和字段 [ES2022]

以下类有两个静态私有槽（A 行和 B 行）：

```js
class StaticPrivateClass {
 // Declare and initialize
 static #staticPrivateField = 'hello'; // (A)
 static #twice() { // (B)
 const str = StaticPrivateClass.#staticPrivateField;
 return str + ' ' + str;
 }
 static getResultOfTwice() {
 return StaticPrivateClass.#twice();
 }
}

assert.deepEqual(
 Reflect.ownKeys(StaticPrivateClass),
 [
 'length', // number of constructor parameters
 'name', // 'StaticPublicFieldClass'
 'prototype',
 'getResultOfTwice',
 ],
);

assert.equal(
 StaticPrivateClass.getResultOfTwice(),
 'hello hello'
);
```

这是所有种类的静态私有槽的完整列表：

```js
class MyClass {
 static #staticPrivateMethod() {}
 static * #staticPrivateGeneratorMethod() {}

 static async #staticPrivateAsyncMethod() {}
 static async * #staticPrivateAsyncGeneratorMethod() {}

 static get #staticPrivateAccessor() {}
 static set #staticPrivateAccessor(value) {}
}
```

#### 29.6.4 类中的静态初始化块 [ES2022]

通过类设置实例数据，我们有两种构造：

+   *字段*，用于创建并可选地初始化实例数据

+   *构造函数*，每次创建新实例时执行的代码块

对于静态数据，我们有：

+   *静态字段*

+   *静态块* 在类创建时执行

以下代码演示了静态块（A 行）：

```js
class Translator {
 static translations = {
 yes: 'ja',
 no: 'nein',
 maybe: 'vielleicht',
 };
 static englishWords = [];
 static germanWords = [];
 static { // (A)
 for (const [english, german] of Object.entries(this.translations)) {
 this.englishWords.push(english);
 this.germanWords.push(german);
 }
 }
}
```

我们也可以在类后执行静态块中的代码（在顶层）。然而，使用静态块有两个好处：

+   所有与类相关的代码都在类内部。

+   静态块中的代码可以访问私有槽。

##### 29.6.4.1 静态初始化块的规则

静态初始化块的工作规则相对简单：

+   每个类可以有多个静态块。

+   静态块的执行与静态字段初始化的执行交错进行。

+   超类的静态成员在子类的静态成员之前执行。

以下代码演示了这些规则：

```js
class SuperClass {
 static superField1 = console.log('superField1');
 static {
 assert.equal(this, SuperClass);
 console.log('static block 1 SuperClass');
 }
 static superField2 = console.log('superField2');
 static {
 console.log('static block 2 SuperClass');
 }
}

class SubClass extends SuperClass {
 static subField1 = console.log('subField1');
 static {
 assert.equal(this, SubClass);
 console.log('static block 1 SubClass');
 }
 static subField2 = console.log('subField2');
 static {
 console.log('static block 2 SubClass');
 }
}

// Output:
// 'superField1'
// 'static block 1 SuperClass'
// 'superField2'
// 'static block 2 SuperClass'
// 'subField1'
// 'static block 1 SubClass'
// 'subField2'
// 'static block 2 SubClass'
```

通过 `extends` 进行子类化 在本章后面有解释。

#### 29.6.5 陷阱：使用 `this` 访问静态私有字段

在静态公共成员中，我们可以通过`this`访问静态公共插槽。遗憾的是，我们不应该用它来访问静态私有插槽。

##### 29.6.5.1 `this`和静态公共字段

考虑以下代码：

```js
class SuperClass {
 static publicData = 1;

 static getPublicViaThis() {
 return this.publicData;
 }
}
class SubClass extends SuperClass {
}
```

通过`extends`进行子类化在本章后面有解释。

静态公共字段是属性。如果我们调用该方法

```js
assert.equal(SuperClass.getPublicViaThis(), 1);
```

然后`this`指向`SuperClass`，一切都按预期工作。我们还可以通过子类调用`.getPublicViaThis()`：

```js
assert.equal(SubClass.getPublicViaThis(), 1);
```

`SubClass`从其原型`SuperClass`继承了`.getPublicViaThis()`。`this`指向`SubClass`，事情继续进行，因为`SubClass`还继承了`.publicData`属性。

顺便说一句，如果我们在`getPublicViaThis()`中为`this.publicData`赋值，并通过`SubClass.getPublicViaThis()`调用它，那么我们将创建一个新的`SubClass`自己的属性，它（非破坏性地）覆盖了从`SuperClass`继承的属性。

##### 29.6.5.2 `this`和静态私有字段

考虑以下代码：

```js
class SuperClass {
 static #privateData = 2;
 static getPrivateDataViaThis() {
 return this.#privateData;
 }
 static getPrivateDataViaClassName() {
 return SuperClass.#privateData;
 }
}
class SubClass extends SuperClass {
}
```

通过`SuperClass`调用`.getPrivateDataViaThis()`是有效的，因为`this`指向`SuperClass`：

```js
assert.equal(SuperClass.getPrivateDataViaThis(), 2);
```

然而，通过`SubClass`调用`.getPrivateDataViaThis()`是无效的，因为此时`this`指向`SubClass`，而`SubClass`没有静态私有字段`.#privateData`（原型链中的私有插槽不会被继承）：

```js
assert.throws(
 () => SubClass.getPrivateDataViaThis(),
 {
 name: 'TypeError',
 message: 'Cannot read private member #privateData from'
 + ' an object whose class did not declare it',
 }
);
```

解决方法是直接访问`.#privateData`，通过`SuperClass`：

```js
assert.equal(SubClass.getPrivateDataViaClassName(), 2);
```

使用静态私有方法时，我们面临相同的问题。

#### 29.6.6 所有成员（静态、原型、实例）都可以访问所有私有成员

类中的每个成员都可以访问该类中的所有其他成员-包括公共成员和私有成员：

```js
class DemoClass {
 static #staticPrivateField = 1;
 #instPrivField = 2;

 static staticMethod(inst) {
 // A static method can access static private fields
 // and instance private fields
 assert.equal(DemoClass.#staticPrivateField, 1);
 assert.equal(inst.#instPrivField, 2);
 }

 protoMethod() {
 // A prototype method can access instance private fields
 // and static private fields
 assert.equal(this.#instPrivField, 2);
 assert.equal(DemoClass.#staticPrivateField, 1);
 }
}
```

相反，外部任何人都无法访问私有成员：

```js
// Accessing private fields outside their classes triggers
// syntax errors (before the code is even executed).
assert.throws(
 () => eval('DemoClass.#staticPrivateField'),
 {
 name: 'SyntaxError',
 message: "Private field '#staticPrivateField' must"
 + " be declared in an enclosing class",
 }
);
// Accessing private fields outside their classes triggers
// syntax errors (before the code is even executed).
assert.throws(
 () => eval('new DemoClass().#instPrivField'),
 {
 name: 'SyntaxError',
 message: "Private field '#instPrivField' must"
 + " be declared in an enclosing class",
 }
);
```

#### 29.6.7 ES2022 之前的静态私有方法和数据

以下代码仅在 ES2022 中有效-由于每一行中都有一个井号（`#`）：

```js
class StaticClass {
 static #secret = 'Rumpelstiltskin';
 static #getSecretInParens() {
 return `(${StaticClass.#secret})`;
 }
 static callStaticPrivateMethod() {
 return StaticClass.#getSecretInParens();
 }
}
```

由于私有插槽只存在于每个类中一次，我们可以将`#secret`和`#getSecretInParens`移到围绕类的范围，并使用模块将它们隐藏在模块外的世界中。

```js
const secret = 'Rumpelstiltskin';
function getSecretInParens() {
 return `(${secret})`;
}

// Only the class is accessible outside the module
export class StaticClass {
 static callStaticPrivateMethod() {
 return getSecretInParens();
 }
}
```

#### 29.6.8 静态工厂方法

有时类可以以多种方式实例化。然后我们可以实现“静态工厂方法”，比如`Point.fromPolar()`：

```js
class Point {
 static fromPolar(radius, angle) {
 const x = radius * Math.cos(angle);
 const y = radius * Math.sin(angle);
 return new Point(x, y);
 }
 constructor(x=0, y=0) {
 this.x = x;
 this.y = y;
 }
}

assert.deepEqual(
 Point.fromPolar(13, 0.39479111969976155),
 new Point(12, 5)
);
```

我喜欢静态工厂方法的描述性：`fromPolar`描述了如何创建实例。JavaScript 的标准库也有这样的工厂方法-例如：

+   `Array.from()`

+   `Object.create()`

我更喜欢要么没有静态工厂方法，要么*只有*静态工厂方法。在后一种情况下需要考虑的事项：

+   一个工厂方法可能会直接调用构造函数（但具有描述性的名称）。

+   我们需要找到一种方法来防止构造函数被外部调用。

在下面的代码中，我们使用一个秘密令牌（A 行）来防止构造函数被外部模块调用。

```js
// Only accessible inside the current module
const secretToken = Symbol('secretToken'); // (A)

export class Point {
 static create(x=0, y=0) {
 return new Point(secretToken, x, y);
 }
 static fromPolar(radius, angle) {
 const x = radius * Math.cos(angle);
 const y = radius * Math.sin(angle);
 return new Point(secretToken, x, y);
 }
 constructor(token, x, y) {
 if (token !== secretToken) {
 throw new TypeError('Must use static factory method');
 }
 this.x = x;
 this.y = y;
 }
}
Point.create(3, 4); // OK
assert.throws(
 () => new Point(3, 4),
 TypeError
);
```

### 29.7 子类化

类也可以扩展现有的类。例如，以下类`Employee`扩展了`Person`：

```js
class Person {
 #firstName;
 constructor(firstName) {
 this.#firstName = firstName;
 }
 describe() {
 return `Person named ${this.#firstName}`;
 }
 static extractNames(persons) {
 return persons.map(person => person.#firstName);
 }
}

class Employee extends Person {
 constructor(firstName, title) {
 super(firstName);
 this.title = title;
 }
 describe() {
 return super.describe() +
 ` (${this.title})`;
 }
}

const jane = new Employee('Jane', 'CTO');
assert.equal(
 jane.title,
 'CTO'
);
assert.equal(
 jane.describe(),
 'Person named Jane (CTO)'
);
```

与扩展相关的术语：

+   “扩展”的另一个词是“子类化”。

+   `Person`是`Employee`的超类。

+   `Employee`是`Person`的子类。

+   “基类”是一个没有超类的类。

+   “派生类”是一个有超类的类。

在派生类的`.constructor()`中，我们必须在访问`this`之前通过`super()`调用超级构造函数。为什么？

让我们考虑一系列的类：

+   基类`A`

+   类`B`扩展了`A`。

+   类`C`扩展了`B`。

如果我们调用`new C()`，`C`的构造函数会调用`B`的构造函数，后者会调用`A`的构造函数。实例总是在基类中创建，然后在子类的构造函数添加插槽之前。因此，在调用`super()`之前实例不存在，我们无法通过`this`访问它。

请注意，静态公共插槽是继承的。例如，`Employee`继承了静态方法`.extractNames()`：

```js
> 'extractNames' in Employee
true
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：子类化**

`exercises/classes/color_point_class_test.mjs`

#### 29.7.1 子类的内部（高级）

![图 15：这些是类 Person 及其子类 Employee 的对象。左列是关于类的，右列是关于 Employee 实例 jane 及其原型链的。](img/2f7f76cfc1a6bab2586a13f8db175cd7.png)

图 15：这些是类`Person`及其子类`Employee`的对象。左列是关于类的，右列是关于`Employee`实例`jane`及其原型链的。

前一节中的类`Person`和`Employee`由几个对象组成（图 15）。理解这些对象如何相关的一个关键见解是，有两个原型链：

+   实例的原型链，位于右侧。

+   类的原型链，位于左侧。

##### 29.7.1.1 实例原型链（右列）

实例的原型链以`jane`开始，然后是`Employee.prototype`和`Person.prototype`。原则上，原型链在这一点上结束，但我们得到了另一个对象：`Object.prototype`。这个原型为几乎所有对象提供服务，这就是为什么它也被包括在这里：

```js
> Object.getPrototypeOf(Person.prototype) === Object.prototype
true
```

##### 29.7.1.2 类原型链（左列）

在类原型链中，`Employee`排在第一位，`Person`排在其后。之后，链条继续包括`Function.prototype`，这是因为`Person`是一个函数，函数需要`Function.prototype`的服务。

```js
> Object.getPrototypeOf(Person) === Function.prototype
true
```

#### 29.7.2 `instanceof`和子类（高级）

我们还没有学习`instanceof`的真正工作原理。`instanceof`如何确定值`x`是否是类`C`的实例（它可以是`C`的直接实例或`C`的子类的直接实例）？它检查`C.prototype`是否在`x`的原型链中。也就是说，以下两个表达式是等价的：

```js
x instanceof C
C.prototype.isPrototypeOf(x)
```

如果我们回到图 15，我们可以确认原型链确实给我们带来了以下正确的答案：

```js
> jane instanceof Employee
true
> jane instanceof Person
true
> jane instanceof Object
true
```

请注意，如果`instanceof`的自身操作数是原始值，`instanceof`总是返回`false`：

```js
> 'abc' instanceof String
false
> 123 instanceof Number
false
```

#### 29.7.3 并非所有对象都是`Object`的实例（高级）

只有对象（非原始值）是`Object`的实例，如果`Object.prototype`在它们的原型链中(见上一小节)。几乎所有对象都是`Object`的实例——例如：

```js
assert.equal(
 {a: 1} instanceof Object, true
);
assert.equal(
 ['a'] instanceof Object, true
);
assert.equal(
 /abc/g instanceof Object, true
);
assert.equal(
 new Map() instanceof Object, true
);

class C {}
assert.equal(
 new C() instanceof Object, true
);
```

在下一个例子中，`obj1`和`obj2`都是对象（行 A 和行 C），但它们不是`Object`的实例（行 B 和行 D）：`Object.prototype`不在它们的原型链中，因为它们没有任何原型。

```js
const obj1 = {__proto__: null};
assert.equal(
 typeof obj1, 'object' // (A)
);
assert.equal(
 obj1 instanceof Object, false // (B)
);

const obj2 = Object.create(null);
assert.equal(
 typeof obj2, 'object' // (C)
);
assert.equal(
 obj2 instanceof Object, false // (D)
);
```

`Object.prototype`是结束大多数原型链的对象。它的原型是`null`，这意味着它也不是`Object`的实例：

```js
> typeof Object.prototype
'object'
> Object.getPrototypeOf(Object.prototype)
null
> Object.prototype instanceof Object
false
```

#### 29.7.4 内置对象的原型链（高级）

接下来，我们将利用我们对子类的了解来理解一些内置对象的原型链。以下工具函数`p()`将帮助我们进行探索。

```js
const p = Object.getPrototypeOf.bind(Object);
```

我们提取了`Object`的方法`.getPrototypeOf()`并将其赋值给`p`。

##### 29.7.4.1 `{}`的原型链

让我们从检查普通对象开始：

```js
> p({}) === Object.prototype
true
> p(p({})) === null
true
```

![图 16：通过对象字面量创建的对象的原型链以该对象开始，继续包括 Object.prototype，最后为 null。](img/e6bb1e3280584f9f2a385f54baab341c.png)

图 16：通过对象字面量创建的对象的原型链以该对象开始，继续包括`Object.prototype`，最后为`null`。

图 16 显示了这个原型链的图表。我们可以看到`{}`确实是`Object`的一个实例——`Object.prototype`在它的原型链中。

##### 29.7.4.2 `[]`的原型链

数组的原型链是什么样的？

```js
> p([]) === Array.prototype
true
> p(p([])) === Object.prototype
true
> p(p(p([]))) === null
true
```

![图 17：数组的原型链包括这些成员：数组实例，Array.prototype，Object.prototype，null。](img/b5d54f0b878da14e43c748ac3f704302.png)

图 17：Array 的原型链有这些成员：Array 实例，`Array.prototype`，`Object.prototype`，`null`。

这个原型链（在图 17 中可视化）告诉我们，Array 对象是`Array`和`Object`的实例。

##### 29.7.4.3 `function () {}`的原型链

最后，普通函数的原型链告诉我们，所有函数都是对象：

```js
> p(function () {}) === Function.prototype
true
> p(p(function () {})) === Object.prototype
true
```

##### 29.7.4.4 内置类的原型链

基类的原型是`Function.prototype`，这意味着它是一个函数（`Function`的实例）：

```js
class A {}
assert.equal(
 Object.getPrototypeOf(A),
 Function.prototype
);

assert.equal(
 Object.getPrototypeOf(class {}),
 Function.prototype
);
```

派生类的原型是它的超类：

```js
class B extends A {}
assert.equal(
 Object.getPrototypeOf(B),
 A
);

assert.equal(
 Object.getPrototypeOf(class extends Object {}),
 Object
);
```

有趣的是，`Object`、`Array`和`Function`都是基类：

```js
> Object.getPrototypeOf(Object) === Function.prototype
true
> Object.getPrototypeOf(Array) === Function.prototype
true
> Object.getPrototypeOf(Function) === Function.prototype
true
```

然而，正如我们所见，即使基类的实例也在它们的原型链中有`Object.prototype`，因为它提供了所有对象都需要的服务。

为什么`Array`和`Function`是基类？

基类是实例实际创建的地方。`Array`和`Function`都需要创建自己的实例，因为它们有所谓的“内部插槽”，这些插槽不能后来添加到由`Object`创建的实例中。

#### 29.7.5 mixin 类（高级）

JavaScript 的类系统只支持*单一继承*。也就是说，每个类最多只能有一个超类。绕过这个限制的一种方法是通过一种称为*mixin 类*（简称：*mixin*）的技术。

思路是这样的：假设我们想让一个类`C`继承自两个超类`S1`和`S2`。这将是*多重继承*，JavaScript 不支持。

我们的解决方法是将`S1`和`S2`转换为*mixins*，子类的工厂：

```js
const S1 = (Sup) => class extends Sup { /*···*/ };
const S2 = (Sup) => class extends Sup { /*···*/ };
```

这两个函数中的每一个都返回一个扩展给定超类`Sup`的类。我们创建类`C`如下：

```js
class C extends S2(S1(Object)) {
 /*···*/
}
```

我们现在有一个类`C`，它扩展了由`S2()`返回的类，该类又扩展了由`S1()`返回的类，该类又扩展了`Object`。

##### 29.7.5.1 品牌管理的 mixin 示例

我们实现了一个 mixin`Branded`，它具有设置和获取对象品牌的辅助方法：

```js
const Named = (Sup) => class extends Sup {
 name = '(Unnamed)';
 toString() {
 const className = this.constructor.name;
 return `${className} named ${this.name}`;
 }
};
```

我们使用这个 mixin 来实现一个具有名称的类`City`：

```js
class City extends Named(Object) {
 constructor(name) {
 super();
 this.name = name;
 }
}
```

以下代码确认 mixin 起作用：

```js
const paris = new City('Paris');
assert.equal(
 paris.name, 'Paris'
);
assert.equal(
 paris.toString(), 'City named Paris'
);
```

##### 29.7.5.2 mixin 的好处

Mixin 使我们摆脱了单一继承的限制：

+   相同的类可以扩展一个超类和零个或多个 mixins。

+   相同的 mixin 可以被多个类使用。

### 29.8 `Object.prototype`的方法和访问器（高级）

正如我们在§29.7.3 “并非所有对象都是`Object`的实例”中所看到的，几乎所有对象都是`Object`的实例。这个类提供了几个有用的方法和一个访问器给它的实例：

+   配置如何将对象转换为原始值（例如通过`+`运算符）：以下方法有默认实现，但通常在子类或实例中被覆盖。

    +   `.toString()`：配置如何将对象转换为字符串。

    +   `.toLocaleString()`：`.toString()`的一个版本，可以通过参数（语言、地区等）以各种方式配置。

    +   `.valueOf()`：配置如何将对象转换为非字符串原始值（通常是数字）。

+   有用的方法（带有陷阱-见下一小节）：

    +   `.isPrototypeOf()`：接收者是否在给定对象的原型链中？

    +   `.propertyIsEnumerable()`：接收者是否具有具有给定键的可枚举自有属性？

+   避免使用这些特性（有更好的替代方法）：

    +   `.__proto__`：获取和设置接收者的原型。

        +   不建议使用这个访问器。替代方法：

            +   `Object.getPrototypeOf()`

            +   `Object.setPrototypeOf()`

    +   `.hasOwnProperty()`: 接收者是否具有给定键的自有属性？

        +   不建议使用这个方法。ES2022 及以后的替代方法：`Object.hasOwn()`。

在我们更详细地了解这些特性之前，我们将了解一个重要的陷阱（以及如何解决它）：我们不能在所有对象上使用 `Object.prototype` 的特性。

#### 29.8.1 安全地使用 `Object.prototype` 的方法

在任意对象上调用 `Object.prototype` 的方法并不总是有效。为了说明这一点，我们使用了 `Object.prototype.hasOwnProperty` 方法，它返回 `true` 如果对象具有给定键的自有属性：

```js
> {ownProp: true}.hasOwnProperty('ownProp')
true
> {ownProp: true}.hasOwnProperty('abc')
false
```

在任意对象上调用 `.hasOwnProperty()` 可能会失败。一方面，如果对象不是 `Object` 的实例，则此方法不可用（参见§29.7.3 “并非所有对象都是 `Object` 的实例”）：

```js
const obj = Object.create(null);
assert.equal(obj instanceof Object, false);
assert.throws(
 () => obj.hasOwnProperty('prop'),
 {
 name: 'TypeError',
 message: 'obj.hasOwnProperty is not a function',
 }
);
```

另一方面，如果对象用自有属性覆盖了 `.hasOwnProperty()`，则我们不能使用它（行 A）：

```js
const obj = {
 hasOwnProperty: 'yes' // (A)
};
assert.throws(
 () => obj.hasOwnProperty('prop'),
 {
 name: 'TypeError',
 message: 'obj.hasOwnProperty is not a function',
 }
);
```

然而，有一种安全的方法来使用 `.hasOwnProperty()`：

```js
function hasOwnProp(obj, propName) {
 return Object.prototype.hasOwnProperty.call(obj, propName); // (A)
}
assert.equal(
 hasOwnProp(Object.create(null), 'prop'), false
);
assert.equal(
 hasOwnProp({hasOwnProperty: 'yes'}, 'prop'), false
);
assert.equal(
 hasOwnProp({hasOwnProperty: 'yes'}, 'hasOwnProperty'), true
);
```

行 A 中的方法调用在§29.3.5 “分派 vs. 直接方法调用”中有解释。

我们还可以使用`.bind()`来实现 `hasOwnProp()`：

```js
const hasOwnProp = Object.prototype.hasOwnProperty.call
 .bind(Object.prototype.hasOwnProperty);
```

这是如何工作的？当我们像前面的示例中的行 A 那样调用 `.call()` 时，它确切地执行了 `hasOwnProp()` 应该执行的操作，包括避免陷阱。然而，如果我们想要函数调用它，我们不能简单地提取它，我们还必须确保它的 `this` 始终具有正确的值。这就是 `.bind()` 的作用。

![](img/4e01a86e5fc2028470dca44da5c2d2aa.png) **通过动态分派使用 `Object.prototype` 方法永远不可以吗？**

在某些情况下，我们可以懒惰地像普通方法一样调用 `Object.prototype` 的方法（不需要 `.call()` 或 `.bind()`）：如果我们知道接收者并且它们是固定布局的对象。

另一方面，如果我们不知道它们的接收者和/或它们是字典对象，那么我们需要采取预防措施。

#### 29.8.2 `Object.prototype.toString()`

通过重写 `.toString()`（在子类或实例中），我们可以配置对象如何转换为字符串：

```js
> String({toString() { return 'Hello!' }})
'Hello!'
> String({})
'[object Object]'
```

将对象转换为字符串最好使用 `String()`，因为它也适用于 `undefined` 和 `null`：

```js
> undefined.toString()
TypeError: Cannot read properties of undefined (reading 'toString')
> null.toString()
TypeError: Cannot read properties of null (reading 'toString')
> String(undefined)
'undefined'
> String(null)
'null'
```

#### 29.8.3 `Object.prototype.toLocaleString()`

`.toLocaleString()` 是 `.toString()` 的一个版本，可以通过区域设置和通常的附加选项进行配置。任何类或实例都可以实现这个方法。在标准库中，以下类可以实现：

+   `Array.prototype.toLocaleString()`

+   `Number.prototype.toLocaleString()`

+   `Date.prototype.toLocaleString()`

+   `TypedArray.prototype.toLocaleString()`

+   `BigInt.prototype.toLocaleString()`

例如，这是十进制小数如何根据区域设置（'fr' 是法语，'en' 是英语）而被转换为不同的字符串的示例：

```js
> 123.45.toLocaleString('fr')
'123,45'
> 123.45.toLocaleString('en')
'123.45'
```

#### 29.8.4 `Object.prototype.valueOf()`

通过重写 `.valueOf()`（在子类或实例中），我们可以配置对象如何转换为非字符串值（通常是数字）：

```js
> Number({valueOf() { return 123 }})
123
> Number({})
NaN
```

#### 29.8.5 `Object.prototype.isPrototypeOf()`

`proto.isPrototypeOf(obj)` 如果 `proto` 在 `obj` 的原型链中则返回 `true`，否则返回 `false`。

```js
const a = {};
const b = {__proto__: a};
const c = {__proto__: b};

assert.equal(a.isPrototypeOf(b), true);
assert.equal(a.isPrototypeOf(c), true);

assert.equal(a.isPrototypeOf(a), false);
assert.equal(c.isPrototypeOf(a), false);
```

这是如何安全使用这个方法的（详情见§29.8.1 “安全使用 `Object.prototype` 方法”）：

```js
const obj = {
 // Overrides Object.prototype.isPrototypeOf
 isPrototypeOf: true,
};
// Doesn’t work in this case:
assert.throws(
 () => obj.isPrototypeOf(Object.prototype),
 {
 name: 'TypeError',
 message: 'obj.isPrototypeOf is not a function',
 }
);
// Safe way of using .isPrototypeOf():
assert.equal(
 Object.prototype.isPrototypeOf.call(obj, Object.prototype), false
);
```

#### 29.8.6 `Object.prototype.propertyIsEnumerable()`

`obj.propertyIsEnumerable(propKey)` 如果 `obj` 具有一个自有可枚举属性，其键为 `propKey` 则返回 `true`，否则返回 `false`。

```js
const proto = {
 enumerableProtoProp: true,
};
const obj = {
 __proto__: proto,
 enumerableObjProp: true,
 nonEnumObjProp: true,
};
Object.defineProperty(
 obj, 'nonEnumObjProp',
 {
 enumerable: false,
 }
);

assert.equal(
 obj.propertyIsEnumerable('enumerableProtoProp'),
 false // not an own property
);
assert.equal(
 obj.propertyIsEnumerable('enumerableObjProp'),
 true
);
assert.equal(
 obj.propertyIsEnumerable('nonEnumObjProp'),
 false // not enumerable
);
assert.equal(
 obj.propertyIsEnumerable('unknownProp'),
 false // not a property
);
```

这是如何安全使用这个方法的（详情见§29.8.1 “安全使用 `Object.prototype` 方法”）：

```js
const obj = {
 // Overrides Object.prototype.propertyIsEnumerable
 propertyIsEnumerable: true,
 enumerableProp: 'yes',
};
// Doesn’t work in this case:
assert.throws(
 () => obj.propertyIsEnumerable('enumerableProp'),
 {
 name: 'TypeError',
 message: 'obj.propertyIsEnumerable is not a function',
 }
);
// Safe way of using .propertyIsEnumerable():
assert.equal(
 Object.prototype.propertyIsEnumerable.call(obj, 'enumerableProp'),
 true
);
```

另一个安全的替代方法是使用属性描述符：

```js
assert.deepEqual(
 Object.getOwnPropertyDescriptor(obj, 'enumerableProp'),
 {
 value: 'yes',
 writable: true,
 enumerable: true,
 configurable: true,
 }
);
```

#### 29.8.7 `Object.prototype.__proto__`（访问器）

属性 `__proto__` 存在两个版本：

+   所有`Object`的实例都有的访问器。

+   对象文字的一个属性，它设置了由它们创建的对象的原型。

我建议避免前一种特性：

+   如§29.8.1 “Using `Object.prototype` methods safely”中所解释的，它并不适用于所有对象。

+   ECMAScript 规范已经将其弃用，并称其为“可选的”和“遗留的”(https://tc39.es/ecma262/#sec-object.prototype.__proto__)。

相比之下，对象文字中的`__proto__`总是有效的，而且没有被弃用。

如果你对访问器`__proto__`的工作原理感兴趣，请继续阅读。

`__proto__`是`Object.prototype`的一个访问器，它被所有`Object`的实例继承。通过类实现它会像这样：

```js
class Object {
 get __proto__() {
 return Object.getPrototypeOf(this);
 }
 set __proto__(other) {
 Object.setPrototypeOf(this, other);
 }
 // ···
}
```

由于`__proto__`是从`Object.prototype`继承的，我们可以通过创建一个不在其原型链中具有`Object.prototype`的对象来移除这个特性（参见§29.7.3 “Not all objects are instances of `Object`”）：

```js
> '__proto__' in {}
true
> '__proto__' in Object.create(null)
false
```

#### 29.8.8 `Object.prototype.hasOwnProperty()`

![](img/0ac255e56dc93a43365d8502301c8688.png)  **Better alternative to `.hasOwnProperty()`: `Object.hasOwn()` [ES2022]**

参见[§28.9.4 “`Object.hasOwn()`: Is a given property own (non-inherited)? [ES2022]”](ch_objects.html#Object.hasOwn)。

`obj.hasOwnProperty(propKey)`如果`obj`有一个自有的（非继承的）属性，其键是`propKey`，则返回`true`，否则返回`false`。

```js
const obj = { ownProp: true };
assert.equal(
 obj.hasOwnProperty('ownProp'), true // own
);
assert.equal(
 'toString' in obj, true // inherited
);
assert.equal(
 obj.hasOwnProperty('toString'), false
);
```

这是如何安全使用这个方法的（详情见§29.8.1 “Using `Object.prototype` methods safely”）：

```js
const obj = {
 // Overrides Object.prototype.hasOwnProperty
 hasOwnProperty: true,
};
// Doesn’t work in this case:
assert.throws(
 () => obj.hasOwnProperty('anyPropKey'),
 {
 name: 'TypeError',
 message: 'obj.hasOwnProperty is not a function',
 }
);
// Safe way of using .hasOwnProperty():
assert.equal(
 Object.prototype.hasOwnProperty.call(obj, 'anyPropKey'), false
);
```

### 29.9 常见问题：类

#### 29.9.1 为什么在本书中称它们为“实例私有字段”，而不是“私有实例字段”？

这样做是为了突出不同的属性（公共槽）和私有槽：通过改变形容词的顺序，“public”和“field”以及“private”和“field”这些词总是一起提到。

#### 29.9.2 为什么标识符前缀`#`？为什么不通过`private`声明私有字段？

私有字段是否可以通过`private`声明并使用普通标识符？让我们来看看如果可能的话会发生什么：

```js
class MyClass {
 private value; // (A)
 compare(other) {
 return this.value === other.value;
 }
}
```

每当`other.value`这样的表达式出现在`MyClass`的主体中时，JavaScript 都必须决定：

+   `.value`是一个属性吗？

+   `.value`是一个私有字段吗？

在编译时，JavaScript 不知道 A 行的声明是否适用于`other`（因为它是`MyClass`的一个实例）或者不适用。这留下了两种选择来做决定：

1.  `.value`总是被解释为私有字段。

1.  JavaScript 在运行时决定：

    +   如果`other`是`MyClass`的一个实例，那么`.value`会被解释为私有字段。

    +   否则`.value`被解释为属性。

这两个选项都有缺点：

+   使用选项（1），我们不能再将`.value`作为属性使用——对于任何对象都是如此。

+   选项（2）会对性能产生负面影响。

这就是为什么引入了名字前缀`#`。现在决定很容易：如果我们使用`#`，我们想要访问一个私有字段。如果不使用，我们想要访问一个属性。

`private`适用于静态类型语言（如 TypeScript），因为它们在编译时知道`other`是否是`MyClass`的一个实例，然后可以将`.value`视为私有或公共的。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **Quiz**

参见 quiz app。

[评论](https://github.com/rauschma/impatient-js/issues/19)
