# 第二十八章：内置子类

> 原文：[28. Subclassing Built-ins](https://exploringjs.com/es5/ch28.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


JavaScript 的内置构造函数很难进行子类化。本章解释了原因并提出了解决方案。

## 术语

我们使用短语*subclass a built-in*并避免术语*extend*，因为它在 JavaScript 中被使用：

子类化内置`A`

创建给定内置构造函数`A`的子构造函数`B`。`B`的实例也是`A`的实例。

扩展对象`obj`

将一个对象的属性复制到另一个对象。Underscore.js [使用这个术语](http://underscorejs.org/#extend)，延续了 Prototype 框架建立的传统。

子类化内置有两个障碍：具有内部属性的实例和无法作为函数调用的构造函数。

## 障碍 1：具有内部属性的实例

大多数内置构造函数都有所谓的*内部属性*的实例（参见[Kinds of Properties](ch17_split_000.html#kinds_of_properties "Kinds of Properties")），它们的名称用双方括号写成，像这样：`[[PrimitiveValue]]`。内部属性由 JavaScript 引擎管理，通常在 JavaScript 中无法直接访问。JavaScript 中的正常子类化技术是使用`this`的子构造函数调用超级构造函数（参见[Layer 4: Inheritance Between Constructors](ch17_split_001.html#constructor_inheritance "Layer 4: Inheritance Between Constructors")）：

```js
function Super(x, y) {
    this.x = x;  // (1)
    this.y = y;  // (1)
}
function Sub(x, y, z) {
    // Add superproperties to subinstance
    Super.call(this, x, y);  // (2)
    // Add subproperty
    this.z = z;
}
```

大多数内置忽略作为`this`传入的子实例（2），这是下一节描述的一个障碍。此外，向现有实例添加内部属性（1）通常是不可能的，因为它们往往会从根本上改变实例的性质。因此，不能使用（2）处的调用来添加内部属性。以下构造函数具有具有内部属性的实例：

包装构造函数

`Boolean`，`Number`和`String`的实例包装原始值。它们都有内部属性`[[PrimitiveValue]]`，其值由`valueOf()`返回；`String`还有两个额外的实例属性：

+   `Boolean`：内部实例属性`[[PrimitiveValue]]`。

+   `Number`：内部实例属性`[[PrimitiveValue]]`。

+   `String`：内部实例属性`[[PrimitiveValue]]`，自定义内部实例方法`[[GetOwnProperty]]`，普通实例属性`length`。`[[GetOwnProperty]]`使得可以通过使用数组索引时从包装字符串中读取字符来进行索引访问。

`Array`

自定义的内部实例方法`[[DefineOwnProperty]]`拦截正在设置的属性。它确保`length`属性正常工作，通过在添加数组元素时保持`length`的最新状态，并在`length`变小时删除多余的元素。

`Date`

内部实例属性`[[PrimitiveValue]]`存储由日期实例表示的时间（自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数）。

`Function`

内部实例属性`[[Call]]`（实例被调用时要执行的代码）和可能还有其他属性。

`RegExp`

内部实例属性`[[Match]]`，以及两个非内部实例属性。来自 ECMAScript 规范：

> `[[Match]]`内部属性的值是`RegExp`对象的模式的实现相关表示。

唯一没有内部属性的内置构造函数是`Error`和`Object`。

### 解决障碍 1

`MyArray`是`Array`的子类。它有一个 getter `size`，返回数组中的实际元素，忽略了空洞（其中`length`考虑了空洞）。实现`MyArray`的技巧是创建一个数组实例，并将其方法复制到其中：²⁰

```js
function MyArray(/*arguments*/) {
    var arr = [];
    // Don’t use Array constructor to set up elements (doesn’t always work)
    Array.prototype.push.apply(arr, arguments);  // (1)
    copyOwnPropertiesFrom(arr, MyArray.methods);
    return arr;
}
MyArray.methods = {
    get size() {
        var size = 0;
        for (var i=0; i < this.length; i++) {
            if (i in this) size++;
        }
        return size;
    }
}
```

此代码使用辅助函数`copyOwnPropertiesFrom()`，该函数在[Copying an Object](ch17_split_000.html#code_copyOwnPropertiesFrom "Copying an Object")中显示和解释。

我们在第（1）行不调用`Array`构造函数，因为有一个怪癖：如果它以一个数字作为单个参数调用，那么这个数字不会成为一个元素，而是确定一个空数组的长度（参见[Initializing an array with elements (avoid!)](ch18.html#avoid_array_constructor "Initializing an array with elements (avoid!)")）。

以下是交互：

```js
> var a = new MyArray('a', 'b')
> a.length = 4;
> a.length
4
> a.size
2
```

### 注意事项

将方法复制到实例会导致冗余，如果可以使用原型，则可以避免这种情况。此外，`MyArray`创建的对象不是其实例：

```js
> a instanceof MyArray
false
> a instanceof Array
true
```

## 障碍 2：无法将构造函数作为函数调用

即使`Error`和子类没有具有内部属性的实例，您仍然无法轻松地对其进行子类化，因为子类化的标准模式不起作用（与之前重复）：

```js
function Super(x, y) {
    this.x = x;
    this.y = y;
}
function Sub(x, y, z) {
    // Add superproperties to subinstance
    Super.call(this, x, y);  // (1)
    // Add subproperty
    this.z = z;
}
```

问题在于`Error`总是产生一个新实例，即使作为函数调用（1）；也就是说，它忽略了通过`call()`传递给它的`this`参数：

```js
> var e = {};
> Object.getOwnPropertyNames(Error.call(e)) // new instance
[ 'stack', 'arguments', 'type' ]
> Object.getOwnPropertyNames(e) // unchanged
[]
```

在前面的交互中，`Error`返回了一个具有自己属性的实例，但它是一个新实例，而不是`e`。如果`Error`将自己的属性添加到`this`（在前面的情况下是`e`），那么子类化模式将起作用。

### 解决障碍 2

在子构造函数内部，创建一个新的超级实例，并将其自己的属性复制到子实例中：

```js
function MyError() {
    // Use Error as a function
    var superInstance = Error.apply(null, arguments);
    copyOwnPropertiesFrom(this, superInstance);
}
MyError.prototype = Object.create(Error.prototype);
MyError.prototype.constructor = MyError;
```

辅助函数`copyOwnPropertiesFrom()`在[Copying an Object](ch17_split_000.html#code_copyOwnPropertiesFrom "Copying an Object")中显示。尝试`MyError`：

```js
try {
    throw new MyError('Something happened');
} catch (e) {
    console.log('Properties: '+Object.getOwnPropertyNames(e));
}
```

这是在 Node.js 上的输出：

```js
Properties: stack,arguments,message,type
```

`instanceof`关系应该是正常的：

```js
> new MyError() instanceof Error
true
> new MyError() instanceof MyError
true
```

## 另一种解决方案：委托

委托是子类化的一个非常干净的替代方法。例如，要创建自己的数组构造函数，您可以在属性中保留一个数组：

```js
function MyArray(/*arguments*/) {
    this.array = [];
    Array.prototype.push.apply(this.array, arguments);
}
Object.defineProperties(MyArray.prototype, {
    size: {
        get: function () {
            var size = 0;
            for (var i=0; i < this.array.length; i++) {
                if (i in this.array) size++;
            }
            return size;
        }
    },
    length: {
        get: function () {
            return this.array.length;
        },
        set: function (value) {
            return this.array.length = value;
        }
    }
});
```

显而易见的限制是，您无法通过方括号访问`MyArray`的元素；您必须使用方法来这样做：

```js
MyArray.prototype.get = function (index) {
    return this.array[index];
}
MyArray.prototype.set = function (index, value) {
    return this.array[index] = value;
}
```

可以通过以下元编程的方法传递`Array.prototype`的普通方法：

```js
[ 'toString', 'push', 'pop' ].forEach(function (key) {
    MyArray.prototype[key] = function () {
        return Array.prototype[key].apply(this.array, arguments);
    }
});
```

我们通过在存储在`MyArray`实例中的数组`this.array`上调用它们，从`Array`方法派生`MyArray`方法。

使用`MyArray`：

```js
> var a = new MyArray('a', 'b');
> a.length = 4;
> a.push('c')
5
> a.length
5
> a.size
3
> a.set(0, 'x');
> a.toString()
'x,b,,,c'
```

* * *

²⁰受[Ben Nadel](http://bit.ly/1oOERFo)的一篇博文的启发。

