# 十四、复制类的实例：.clone() vs. 复制构造函数

> 原文：[`exploringjs.com/deep-js/ch_copying-class-instances.html`](https://exploringjs.com/deep-js/ch_copying-class-instances.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


* * *

+   14.1 `.clone()`方法

+   14.2 静态工厂方法

+   14.3 致谢

* * *

在本章中，我们将介绍两种实现类实例复制的技术：

+   `.clone()`方法

+   所谓的*复制构造函数*，即接收当前类的另一个实例并用它来初始化当前实例的构造函数。

### 14.1 `.clone()`方法

这种技术为每个需要被复制的类引入了一个`.clone()`方法。它返回`this`的深复制。下面的例子展示了三个可以被克隆的类。

```js
class Point {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
 clone() {
 return new Point(this.x, this.y);
 }
}
class Color {
 constructor(name) {
 this.name = name;
 }
 clone() {
 return new Color(this.name);
 }
}
class ColorPoint extends Point {
 constructor(x, y, color) {
 super(x, y);
 this.color = color;
 }
 clone() {
 return new ColorPoint(
 this.x, this.y, this.color.clone()); // (A)
 }
}
```

A 行展示了这种技术的一个重要方面：复合实例属性值也必须递归地被克隆。

### 14.2 静态工厂方法

*复制构造函数*是一种使用当前类的另一个实例来设置当前实例的构造函数。复制构造函数在静态语言（如 C++和 Java）中很受欢迎，因为你可以通过*静态重载*提供构造函数的多个版本。在这里，*静态*意味着选择使用哪个版本是在编译时做出的。

在 JavaScript 中，我们必须在运行时做出决定，这导致了不优雅的代码：

```js
class Point {
 constructor(...args) {
 if (args[0] instanceof Point) {
 // Copy constructor
 const [other] = args;
 this.x = other.x;
 this.y = other.y;
 } else {
 const [x, y] = args;
 this.x = x;
 this.y = y;
 }
 }
}
```

这是你如何使用这个类的方式：

```js
const original = new Point(-1, 4);
const copy = new Point(original);
assert.deepEqual(copy, original);
```

*静态工厂方法*是构造函数的一种替代方法，在这种情况下效果更好，因为我们可以直接调用所需的功能。（这里，*静态*意味着这些工厂方法是类方法。）

在下面的例子中，三个类`Point`、`Color`和`ColorPoint`都有一个静态工厂方法`.from()`：

```js
class Point {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
 static from(other) {
 return new Point(other.x, other.y);
 }
}
class Color {
 constructor(name) {
 this.name = name;
 }
 static from(other) {
 return new Color(other.name);
 }
}
class ColorPoint extends Point {
 constructor(x, y, color) {
 super(x, y);
 this.color = color;
 }
 static from(other) {
 return new ColorPoint(
 other.x, other.y, Color.from(other.color)); // (A)
 }
}
```

在 A 行，我们再次进行递归复制。

这就是`ColorPoint.from()`的工作原理：

```js
const original = new ColorPoint(-1, 4, new Color('red'));
const copy = ColorPoint.from(original);
assert.deepEqual(copy, original);
```

### 14.3 致谢

+   [Ron Korvig](https://github.com/ronkorving)提醒我在 JavaScript 中进行深复制时要使用静态工厂方法而不是重载构造函数。

[评论](https://github.com/rauschma/deep-js/issues/14)
