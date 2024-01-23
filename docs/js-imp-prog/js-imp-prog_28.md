# 二十三、控制流语句

> 原文：[`exploringjs.com/impatient-js/ch_control-flow.html`](https://exploringjs.com/impatient-js/ch_control-flow.html)

* * *

+   23.1 控制循环：`break` 和 `continue`

    +   23.1.1 `break`

    +   23.1.2 `break` 加标签：离开任何带标签的语句

    +   23.1.3 `continue`

+   23.2 控制流语句的条件

+   23.3 [`if` 语句 [ES1]](ch_control-flow.html#if)

    +   23.3.1 `if` 语句的语法

+   23.4 [`switch` 语句 [ES3]](ch_control-flow.html#switch)

    +   23.4.1 `switch` 语句的第一个例子

    +   23.4.2 不要忘记 `return` 或 `break`！

    +   23.4.3 空的 case 子句

    +   23.4.4 通过 `default` 子句检查非法值

+   23.5 [`while` 循环 [ES1]](ch_control-flow.html#while)

    +   23.5.1 `while` 循环的例子

+   23.6 [`do-while` 循环 [ES3]](ch_control-flow.html#do-while)

+   23.7 [`for` 循环 [ES1]](ch_control-flow.html#for)

    +   23.7.1 `for` 循环的例子

+   23.8 [`for-of` 循环 [ES6]](ch_control-flow.html#for-of)

    +   23.8.1 `const`: `for-of` vs. `for`

    +   23.8.2 遍历可迭代对象

    +   23.8.3 [遍历数组的[index, element]对](ch_control-flow.html#for-of-iterating-index-element)

+   23.9 [`for-await-of` 循环 [ES2018]](ch_control-flow.html#for-await-of-loops-es2018)

+   23.10 [`for-in` 循环（避免使用）[ES1]](ch_control-flow.html#for-in)

+   23.11 循环的建议

* * *

本章涵盖了以下控制流语句：

+   `if` 语句 [ES1]

+   `switch` 语句 [ES3]

+   `while` 循环 [ES1]

+   `do-while` 循环 [ES3]

+   `for` 循环 [ES1]

+   `for-of` 循环 [ES6]

+   `for-await-of` 循环 [ES2018]

+   `for-in` 循环 [ES1]

### 23.1 控制循环：`break` 和 `continue`

两个操作符 `break` 和 `continue` 可以用于控制循环和其他语句，当我们在其中时。

#### 23.1.1 `break`

`break` 有两个版本：一个带操作数，一个不带操作数。后者的版本适用于以下语句：`while`, `do-while`, `for`, `for-of`, `for-await-of`, `for-in` 和 `switch`。它会立即离开当前语句：

```js
for (const x of ['a', 'b', 'c']) {
 console.log(x);
 if (x === 'b') break;
 console.log('---')
}

// Output:
// 'a'
// '---'
// 'b'
```

#### 23.1.2 `break` 加标签：离开任何带标签的语句

带操作数的 `break` 可以在任何地方使用。它的操作数是一个*标签*。标签可以放在任何语句前面，包括块。`break my_label` 会离开标签为 `my_label` 的语句：

```js
my_label: { // label
 if (condition) break my_label; // labeled break
 // ···
}
```

在以下示例中，搜索可以是：

+   失败：循环在没有找到`result`的情况下结束。这直接在循环之后处理（B 行）。

+   成功：在循环中，我们找到了一个`result`。然后我们使用`break`加标签（A 行）来跳过处理失败的代码。

```js
function findSuffix(stringArray, suffix) {
 let result;
 search_block: {
 for (const str of stringArray) {
 if (str.endsWith(suffix)) {
 // Success:
 result = str;
 break search_block; // (A)
 }
 } // for
 // Failure:
 result = '(Untitled)'; // (B)
 } // search_block

 return { suffix, result };
 // Same as: {suffix: suffix, result: result}
}
assert.deepEqual(
 findSuffix(['notes.txt', 'index.html'], '.html'),
 { suffix: '.html', result: 'index.html' }
);
assert.deepEqual(
 findSuffix(['notes.txt', 'index.html'], '.mjs'),
 { suffix: '.mjs', result: '(Untitled)' }
);
```

#### 23.1.3 `continue`

`continue` 只能在 `while`, `do-while`, `for`, `for-of`, `for-await-of`, 和 `for-in` 中使用。它会立即离开当前循环迭代，并继续下一个迭代 - 例如：

```js
const lines = [
 'Normal line',
 '# Comment',
 'Another normal line',
];
for (const line of lines) {
 if (line.startsWith('#')) continue;
 console.log(line);
}
// Output:
// 'Normal line'
// 'Another normal line'
```

### 23.2 控制流语句的条件

`if`, `while` 和 `do-while` 都有原则上是布尔值的条件。但是，条件只需要是*真值*（如果强制转换为布尔值则为`true`）就可以被接受。换句话说，以下两个控制流语句是等价的：

```js
if (value) {}
if (Boolean(value) === true) {}
```

这是所有*假值*的列表：

+   `undefined`, `null`

+   `false`

+   `0`, `NaN`

+   `0n`

+   `''`

所有其他值都是真值。更多信息，请参见§15.2 “Falsy and truthy values”。

### 23.3 `if`语句 [ES1]

这些都是两个简单的`if`语句：一个只有一个“then”分支，另一个既有“then”分支又有“else”分支：

```js
if (cond) {
 // then branch
}

if (cond) {
 // then branch
} else {
 // else branch
}
```

而不是块，`else`也可以跟着另一个`if`语句：

```js
if (cond1) {
 // ···
} else if (cond2) {
 // ···
}

if (cond1) {
 // ···
} else if (cond2) {
 // ···
} else {
 // ···
}
```

您可以使用更多的`else if`继续这个链。

#### 23.3.1 `if`语句的语法

`if`语句的一般语法是：

```js
if (cond) «then_statement»
else «else_statement»
```

到目前为止，`then_statement`一直是一个块，但我们可以使用任何语句。该语句必须以分号结束：

```js
if (true) console.log('Yes'); else console.log('No');
```

这意味着`else if`不是自己的结构；它只是一个`else_statement`是另一个`if`语句的`if`语句。

### 23.4 `switch`语句 [ES3]

`switch`语句的外观如下：

```js
switch («switch_expression») {
  «switch_body»
}
```

`switch`的主体由零个或多个 case 子句组成：

```js
case «case_expression»:
  «statements»
```

并且，可选地，一个默认子句：

```js
default:
  «statements»
```

`switch`的执行方式如下：

+   它评估 switch 表达式。

+   它跳转到第一个 case 子句，其表达式与 switch 表达式的结果相同。

+   否则，如果没有这样的子句，它就会跳转到默认子句。

+   否则，如果没有默认子句，它就什么也不做。

#### 23.4.1 `switch`语句的第一个示例

让我们看一个例子：以下函数将数字从 1-7 转换为工作日的名称。

```js
function dayOfTheWeek(num) {
 switch (num) {
 case 1:
 return 'Monday';
 case 2:
 return 'Tuesday';
 case 3:
 return 'Wednesday';
 case 4:
 return 'Thursday';
 case 5:
 return 'Friday';
 case 6:
 return 'Saturday';
 case 7:
 return 'Sunday';
 }
}
assert.equal(dayOfTheWeek(5), 'Friday');
```

#### 23.4.2 不要忘记`return`或`break`！

在 case 子句的末尾，执行会继续下一个 case 子句，除非我们`return`或`break` - 例如：

```js
function englishToFrench(english) {
 let french;
 switch (english) {
 case 'hello':
 french = 'bonjour';
 case 'goodbye':
 french = 'au revoir';
 }
 return french;
}
// The result should be 'bonjour'!
assert.equal(englishToFrench('hello'), 'au revoir');
```

也就是说，我们的`dayOfTheWeek()`的实现之所以有效，仅仅是因为我们使用了`return`。我们可以通过使用`break`来修复`englishToFrench()`：

```js
function englishToFrench(english) {
 let french;
 switch (english) {
 case 'hello':
 french = 'bonjour';
 break;
 case 'goodbye':
 french = 'au revoir';
 break;
 }
 return french;
}
assert.equal(englishToFrench('hello'), 'bonjour'); // ok
```

#### 23.4.3 空 case 子句

case 子句的语句可以被省略，这实际上给了我们每个 case 子句多个 case 表达式：

```js
function isWeekDay(name) {
 switch (name) {
 case 'Monday':
 case 'Tuesday':
 case 'Wednesday':
 case 'Thursday':
 case 'Friday':
 return true;
 case 'Saturday':
 case 'Sunday':
 return false;
 }
}
assert.equal(isWeekDay('Wednesday'), true);
assert.equal(isWeekDay('Sunday'), false);
```

#### 23.4.4 通过`default`子句检查非法值

如果`switch`表达式没有其他匹配项，就会跳转到`default`子句。这使得它对错误检查很有用：

```js
function isWeekDay(name) {
 switch (name) {
 case 'Monday':
 case 'Tuesday':
 case 'Wednesday':
 case 'Thursday':
 case 'Friday':
 return true;
 case 'Saturday':
 case 'Sunday':
 return false;
 default:
 throw new Error('Illegal value: '+name);
 }
}
assert.throws(
 () => isWeekDay('January'),
 {message: 'Illegal value: January'});
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png) **练习：`switch`**

+   `exercises/control-flow/number_to_month_test.mjs`

+   奖励：`exercises/control-flow/is_object_via_switch_test.mjs`

### 23.5 `while`循环 [ES1]

`while`循环有以下语法：

```js
while («condition») {
  «statements»
}
```

在每次循环迭代之前，`while`评估`condition`：

+   如果结果为假，循环就结束了。

+   如果结果为真，`while`体将再执行一次。

#### 23.5.1 `while`循环的示例

以下代码使用了`while`循环。在每次循环迭代中，它通过`.shift()`移除`arr`的第一个元素并记录它。

```js
const arr = ['a', 'b', 'c'];
while (arr.length > 0) {
 const elem = arr.shift(); // remove first element
 console.log(elem);
}
// Output:
// 'a'
// 'b'
// 'c'
```

如果条件总是评估为`true`，那么`while`就是一个无限循环：

```js
while (true) {
 if (Math.random() === 0) break;
}
```

### 23.6 `do-while`循环 [ES3]

`do-while`循环的工作方式与`while`很像，但它在每次循环迭代之后检查条件，而不是之前。

```js
let input;
do {
 input = prompt('Enter text:');
 console.log(input);
} while (input !== ':q');
```

`do-while`也可以被视为至少运行一次的`while`循环。

[`prompt()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/prompt)是一个全局函数，在 Web 浏览器中可用。它提示用户输入文本并返回它。

### 23.7 `for`循环 [ES1]

`for`循环有以下语法：

```js
for («initialization»; «condition»; «post_iteration») {
  «statements»
}
```

第一行是循环的*head*，控制*body*（循环的其余部分）的执行次数。它有三个部分，每个部分都是可选的：

+   `initialization`：为循环设置变量等。此处通过`let`或`const`声明的变量仅在循环内部存在。

+   `condition`：在每次循环迭代之前检查此条件。如果它是假的，循环就会停止。

+   `post_iteration`：此代码在每次循环迭代之后执行。

因此，`for`循环大致相当于以下`while`循环：

```js
«initialization»
while («condition») {
  «statements»
  «post_iteration»
}
```

#### 23.7.1 `for`循环的示例

例如，这是如何通过`for`循环从零数到两的：

```js
for (let i=0; i<3; i++) {
 console.log(i);
}

// Output:
// 0
// 1
// 2
```

这是如何通过`for`循环记录数组的内容：

```js
const arr = ['a', 'b', 'c'];
for (let i=0; i<arr.length; i++) {
 console.log(arr[i]);
}

// Output:
// 'a'
// 'b'
// 'c'
```

如果我们省略头的所有三个部分，我们就得到了一个无限循环：

```js
for (;;) {
 if (Math.random() === 0) break;
}
```

### 23.8 `for-of`循环 [ES6]

`for-of`循环遍历任何*可迭代* - 支持*迭代协议*的数据容器。每个迭代的值都存储在变量中，如头部中指定的那样：

```js
for («iteration_variable» of «iterable») {
  «statements»
}
```

迭代变量通常是通过变量声明创建的：

```js
const iterable = ['hello', 'world'];
for (const elem of iterable) {
 console.log(elem);
}
// Output:
// 'hello'
// 'world'
```

但我们也可以使用一个（可变的）已经存在的变量：

```js
const iterable = ['hello', 'world'];
let elem;
for (elem of iterable) {
 console.log(elem);
}
```

#### 23.8.1 `const`：`for-of` vs. `for`

请注意，在`for-of`循环中，我们可以使用`const`。迭代变量仍然可以在每次迭代中不同（它只是在迭代期间不能更改）。可以将其视为在新的作用域中每次执行一个新的`const`声明。

相比之下，在`for`循环中，如果它们的值发生变化，我们必须通过`let`或`var`声明变量。

#### 23.8.2 遍历可迭代对象

如前所述，`for-of`适用于任何可迭代对象，而不仅仅是数组 - 例如，与 Set 一起使用：

```js
const set = new Set(['hello', 'world']);
for (const elem of set) {
 console.log(elem);
}
```

#### 23.8.3 遍历数组的[index, element]对

最后，我们还可以使用`for-of`来遍历数组的[index, element]条目：

```js
const arr = ['a', 'b', 'c'];
for (const [index, elem] of arr.entries()) {
 console.log(`${index} -> ${elem}`);
}
// Output:
// '0 -> a'
// '1 -> b'
// '2 -> c'
```

使用`[index, element]`，我们正在使用*解构*来访问数组元素。

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：`for-of`**

`exercises/control-flow/array_to_string_test.mjs`

### 23.9 `for-await-of`循环[ES2018]

`for-await-of`类似于`for-of`，但它适用于异步可迭代对象，而不是同步可迭代对象。它只能在异步函数和异步生成器内部使用。

```js
for await (const item of asyncIterable) {
 // ···
}
```

`for-await-of`在异步迭代章节中有详细描述。

### 23.10 `for-in`循环（避免）[ES1]

`for-in`循环访问对象的所有（自己的和继承的）可枚举属性键。在遍历数组时，这很少是一个好选择：

+   它访问属性键，而不是值。

+   作为属性键，数组元素的索引是字符串，而不是数字（有关数组元素工作原理的更多信息）。

+   它访问所有可枚举的属性键（自己的和继承的），而不仅仅是数组元素的属性键。

以下代码演示了这些要点：

```js
const arr = ['a', 'b', 'c'];
arr.propKey = 'property value';

for (const key in arr) {
 console.log(key);
}

// Output:
// '0'
// '1'
// '2'
// 'propKey'
```

### 23.11 循环建议

+   如果要循环遍历异步可迭代对象（在 ES2018+中），必须使用`for-await-of`。

+   要循环同步可迭代对象（在 ES6+中），必须使用`for-of`。请注意，数组是可迭代对象。

+   要循环遍历 ES5+中的数组，可以使用数组方法`.forEach()`。

+   在 ES5 之前，您可以使用普通的`for`循环来遍历数组。

+   不要使用`for-in`循环遍历数组。

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

请参阅测验应用程序。

[评论](https://github.com/rauschma/impatient-js/issues/16)
