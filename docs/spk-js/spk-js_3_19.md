# 第二十五章：ECMAScript 5 中的新功能

> 原文：[25. New in ECMAScript 5](https://exploringjs.com/es5/ch25.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)


本章列出了仅在 ECMAScript 5 中可用的功能。如果您必须使用旧版 JavaScript 引擎，您应该避免使用这些功能，或者通过库启用其中一些功能（稍后将进行描述）。请注意，通常情况下，本书假定您正在使用完全支持 ECMAScript 5 的现代引擎。

ECMAScript 5 规范包含了对其范围的以下描述：

> ECMAScript 的第五版（作为 ECMA-262 第 5 版发布）
>
> +   对已成为浏览器实现中常见的语言规范的实际解释进行了编码
> 
> +   增加了对自第三版出版以来出现的新功能的支持。这些功能包括
> 
>     +   访问器属性，
> 
>     +   反射创建和检查对象，
> 
>     +   程序控制属性属性，
> 
>     +   附加数组操作函数，
> 
>     +   对 JSON 对象编码格式的支持，以及 x
> 
>     +   提供增强的错误检查和程序安全性的严格模式。

## 新功能

ECMAScript 5 中包含的新功能如下：

严格模式（参见[严格模式](ch07.html#strict_mode "严格模式")）

将以下行放在文件或函数的开头可以打开所谓的*严格模式*，使 JavaScript 成为一个更干净的语言，禁止一些功能，执行更多检查，并抛出更多异常：

```js
'use strict';
```

访问器（参见[访问器（Getter 和 Setter）](ch17_split_000.html#getters_setters "访问器（Getter 和 Setter）")）

Getter 和 setter 允许您通过方法实现属性的获取和设置。例如，以下对象`obj`包含属性`foo`的 getter：

```js
> var obj = { get foo() { return 'abc' } };
> obj.foo
'abc'
```

## 语法更改

ECMAScript 5 包括以下语法更改：

保留字作为属性键

您可以在点运算符之后使用保留字（例如`new`和`function`）并且在对象文字中作为非引用的属性键：

```js
> var obj = { new: 'abc' };
> obj.new
'abc'
```

合法的尾随逗号

对象文字和数组文字中的尾随逗号是合法的。

多行字符串文字

如果通过反斜杠转义行尾，字符串文字可以跨多行。

## 标准库中的新功能

ECMAScript 5 为 JavaScript 的标准库带来了几个新增功能。本节按类别列出了它们。

### 元编程

获取和设置原型（参见[获取和设置原型](ch17_split_000.html#get_set_prototype "获取和设置原型")）：

+   `Object.create()`

+   `Object.getPrototypeOf()`

通过属性描述符管理属性属性（参见[属性描述符](ch17_split_000.html#property_descriptors "属性描述符")）：

+   `Object.defineProperty()`

+   `Object.defineProperties()`

+   `Object.create()`

+   `Object.getOwnPropertyDescriptor()`

列出属性（参见[迭代和属性检测](ch17_split_000.html#iterate_and_detect_properties "迭代和属性检测")）：

+   `Object.keys()`

+   `Object.getOwnPropertyNames()`

保护对象（参见[保护对象](ch17_split_001.html#protecting_objects "保护对象")）：

+   `Object.preventExtensions()`

+   `Object.isExtensible()`

+   `Object.seal()`

+   `Object.isSealed()`

+   `Object.freeze()`

+   `Object.isFrozen()`

新的`Function`方法（参见[Function.prototype.bind(thisValue, arg1?, ..., argN?)](ch17_split_000.html#Function.prototype.bind "Function.prototype.bind(thisValue, arg1?, ..., argN?)")）：

+   `Function.prototype.bind()`

### 新方法

字符串（参见第十二章）：

+   新方法`String.prototype.trim()`

+   通过方括号操作符`[...]`访问字符

新的`Array`方法（参见[Array Prototype Methods](ch18.html#array_prototype_methods "Array Prototype Methods"）：

+   `Array.isArray()`

+   `Array.prototype.every()`

+   `Array.prototype.filter()`

+   `Array.prototype.forEach()`

+   `Array.prototype.indexOf()`

+   `Array.prototype.lastIndexOf()`

+   `Array.prototype.map()`

+   `Array.prototype.reduce()`

+   `Array.prototype.some()`

新的`Date`方法（参见[Date Prototype Methods](ch20.html#date_prototype_methods "Date Prototype Methods")）：

+   `Date.now()`

+   `Date.prototype.toISOString()`

### JSON

对 JSON 的支持（参见第二十二章）：

+   `JSON.parse()`（参见[JSON.parse(text, reviver?)](ch22.html#JSON.parse "JSON.parse(text, reviver?)")）

+   `JSON.stringify()`（参见[JSON.stringify(value, replacer?, space?)](ch22.html#JSON.stringify "JSON.stringify(value, replacer?, space?)")）

+   一些内置对象具有特殊的`toJSON()`方法：

+   `Boolean.prototype.toJSON()`

+   `Number.prototype.toJSON()`

+   `String.prototype.toJSON()`

+   `Date.prototype.toJSON()`

## 与旧版浏览器一起工作的提示

如果您需要与旧版浏览器一起工作，以下资源将非常有用：

+   Juriy Zaytsev（“kangax”）的[兼容性表](http://kangax.github.io/es5-compat-table/)显示了各种浏览器的各个版本支持 ECMAScript 5 的程度。

+   [es5-shim](https://github.com/kriskowal/es5-shim/) 将 ECMAScript 5 的大部分（但不是全部）功能带到只支持 ECMAScript 3 的浏览器中。
