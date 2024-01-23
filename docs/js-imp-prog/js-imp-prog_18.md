# 14 The non-values undefined and null

> 原文：[`exploringjs.com/impatient-js/ch_undefined-null.html`](https://exploringjs.com/impatient-js/ch_undefined-null.html)

* * *

+   14.1 `undefined` vs. `null`

+   14.2 Occurrences of `undefined` and `null`

    +   14.2.1 Occurrences of `undefined`

    +   14.2.2 Occurrences of `null`

+   14.3 Checking for `undefined` or `null`

+   14.4 [The nullish coalescing operator (`??`) for default values [ES2020]](ch_undefined-null.html#nullish-coalescing-operator)

    +   14.4.1 Example: counting matches

    +   14.4.2 Example: specifying a default value for a property

    +   14.4.3 Using destructuring for default values

    +   14.4.4 Legacy approach: using logical Or (`||`) for default values

    +   14.4.5 [空值合并赋值运算符 (`??=`) [ES2021]](ch_undefined-null.html#nullish-coalescing-assignment-operator)

+   14.5 `undefined` and `null` don’t have properties

+   14.6 The history of `undefined` and `null`

* * *

许多编程语言都有一个叫做 `null` 的“非值”。它表示变量当前没有指向对象 - 例如，当它尚未初始化时。

相反，JavaScript 有两个：`undefined` 和 `null`。

### 14.1 `undefined` vs. `null`

这两个值非常相似，经常可以互换使用。它们的区别因此很微妙。语言本身做出以下区分：

+   `undefined` 意味着“未初始化”（例如变量）或“不存在”（例如对象的属性）。

+   `null` 意味着“有意的缺少任何对象值”（引用自[语言规范](https://tc39.github.io/ecma262/#sec-null-value)）。

程序员可能会做出以下区分：

+   `undefined` 是语言使用的非值（当某些东西未初始化时等）。

+   `null` 意味着“明确关闭”。也就是说，它有助于实现一个包括有意义值和代表“没有有意义值”的元值的类型。这种类型在函数式编程中称为[*option type* 或 *maybe type*](https://en.wikipedia.org/wiki/Option_type)。

### 14.2 Occurrences of `undefined` and `null`

下面的小节描述了语言中 `undefined` 和 `null` 出现的地方。我们将遇到几种稍后在本书中更详细解释的机制。

#### 14.2.1 Occurrences of `undefined`

未初始化的变量 `myVar`：

```js
let myVar;
assert.equal(myVar, undefined);
```

参数 `x` 未提供：

```js
function func(x) {
 return x;
}
assert.equal(func(), undefined);
```

属性 `.unknownProp` 丢失：

```js
const obj = {};
assert.equal(obj.unknownProp, undefined);
```

如果我们没有通过 `return` 语句明确指定函数的结果，JavaScript 会为我们返回 `undefined`：

```js
function func() {}
assert.equal(func(), undefined);
```

#### 14.2.2 Occurrences of `null`

对象的原型要么是对象，要么在原型链的末端是 `null`。`Object.prototype` 没有原型：

```js
> Object.getPrototypeOf(Object.prototype)
null
```

如果我们对字符串（例如 `'x'`）进行正则表达式匹配（例如 `/a/`），我们要么得到一个具有匹配数据的对象（如果匹配成功），要么得到 `null`（如果匹配失败）：

```js
> /a/.exec('x')
null
```

JSON 数据格式 不支持 `undefined`，只支持 `null`：

```js
> JSON.stringify({a: undefined, b: null})
'{"b":null}'
```

### 14.3 Checking for `undefined` or `null`

检查任何一个：

```js
if (x === null) ···
if (x === undefined) ···
```

`x` 有值吗？

```js
if (x !== undefined && x !== null) {
 // ···
}
if (x) { // truthy?
 // x is neither: undefined, null, false, 0, NaN, ''
}
```

`x` 是 `undefined` 还是 `null`？

```js
if (x === undefined || x === null) {
 // ···
}
if (!x) { // falsy?
 // x is: undefined, null, false, 0, NaN, ''
}
```

*Truthy* 意味着“如果强制转换为布尔值，则为 `true`”。*Falsy* 意味着“如果强制转换为布尔值，则为 `false`”。这两个概念在§15.2 “Falsy and truthy values”中得到了适当的解释。

### 14.4 The nullish coalescing operator (`??`) for default values [ES2020]

有时我们会收到一个值，只想在它既不是`null`也不是`undefined`时使用它。否则，我们希望使用一个默认值作为后备。我们可以通过*空值合并运算符*(`??`)来实现这一点：

```js
const valueToUse = receivedValue ?? defaultValue;
```

以下两个表达式是等价的：

```js
a ?? b
a !== undefined && a !== null ? a : b
```

#### 14.4.1 示例：计算匹配项

以下代码显示了一个现实世界的例子：

```js
function countMatches(regex, str) {
 const matchResult = str.match(regex); // null or Array
 return (matchResult ?? []).length;
}

assert.equal(
 countMatches(/a/g, 'ababa'), 3);
assert.equal(
 countMatches(/b/g, 'ababa'), 2);
assert.equal(
 countMatches(/x/g, 'ababa'), 0);
```

如果`str`中存在一个或多个`regex`的匹配项，则`.match()`返回一个数组。如果没有匹配项，它不幸地返回`null`（而不是空数组）。我们通过`??`运算符来修复这个问题。

我们也可以使用可选链：

```js
return matchResult?.length ?? 0;
```

#### 14.4.2 示例：为属性指定默认值

```js
function getTitle(fileDesc) {
 return fileDesc.title ?? '(Untitled)';
}

const files = [
 {path: 'index.html', title: 'Home'},
 {path: 'tmp.html'},
];
assert.deepEqual(
 files.map(f => getTitle(f)),
 ['Home', '(Untitled)']);
```

#### 14.4.3 使用解构来设置默认值

在某些情况下，解构也可以用于默认值 - 例如：

```js
function getTitle(fileDesc) {
 const {title = '(Untitled)'} = fileDesc;
 return title;
}
```

#### 14.4.4 传统方法：使用逻辑或(`||`)来设置默认值

在 ECMAScript 2020 之前和空值合并运算符之前，逻辑或被用于默认值。这有一个缺点。

`||`对`undefined`和`null`的工作方式与预期相同：

```js
> undefined || 'default'
'default'
> null || 'default'
'default'
```

但它还返回所有其他假值的默认值 - 例如：

```js
> false || 'default'
'default'
> 0 || 'default'
'default'
> 0n || 'default'
'default'
> '' || 'default'
'default'
```

将其与`??`的工作方式进行比较：

```js
> undefined ?? 'default'
'default'
> null ?? 'default'
'default'

> false ?? 'default'
false
> 0 ?? 'default'
0
> 0n ?? 'default'
0n
> '' ?? 'default'
''
```

#### 14.4.5 空值合并赋值运算符(`??=`) [ES2021]

`??=`是一个逻辑赋值运算符。以下两个表达式大致等价：

```js
a ??= b
a ?? (a = b)
```

这意味着`??=`是短路的：只有在`a`是`undefined`或`null`时才进行赋值。

##### 14.4.5.1 示例：使用`??=`添加丢失的属性

```js
const books = [
 {
 isbn: '123',
 },
 {
 title: 'ECMAScript Language Specification',
 isbn: '456',
 },
];

// Add property .title where it’s missing
for (const book of books) {
 book.title ??= '(Untitled)';
}

assert.deepEqual(
 books,
 [
 {
 isbn: '123',
 title: '(Untitled)',
 },
 {
 title: 'ECMAScript Language Specification',
 isbn: '456',
 },
 ]);
```

### 14.5 `undefined`和`null`没有属性

`undefined`和`null`是 JavaScript 中唯一两个值，如果我们尝试读取属性，会得到异常。为了探索这一现象，让我们使用以下函数，该函数读取（“获取”）属性`.foo`并返回结果。

```js
function getFoo(x) {
 return x.foo;
}
```

如果我们将`getFoo()`应用于各种值，我们可以看到它仅对`undefined`和`null`失败：

```js
> getFoo(undefined)
TypeError: Cannot read properties of undefined (reading 'foo')
> getFoo(null)
TypeError: Cannot read properties of null (reading 'foo')

> getFoo(true)
undefined
> getFoo({})
undefined
```

### 14.6 未定义和 null 的历史

在 Java 中（它启发了 JavaScript 的许多方面），初始化值取决于变量的静态类型：

+   具有对象类型的变量被初始化为`null`。

+   每种原始类型都有自己的初始化值。例如，`int`变量的初始化值为`0`。

在 JavaScript 中，每个变量既可以保存对象值，也可以保存原始值。因此，如果`null`表示“不是对象”，JavaScript 还需要一个初始化值，表示“既不是对象也不是原始值”。该初始化值是`undefined`。

![](img/4ca05ad97a693bee61e4fd6459232e60.png) **测验**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/9)
