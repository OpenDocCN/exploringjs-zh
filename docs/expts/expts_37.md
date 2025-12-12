# 30 类型守卫和窄化

> 原文：[`exploringjs.com/ts/book/ch_type-guards.html`](https://exploringjs.com/ts/book/ch_type-guards.html)

（广告，请勿拦截。）

1.  30.1 类型守卫和窄化

    1.  30.1.1 示例：使用 `===` 的类型守卫

    1.  30.1.2 示例：使用 `typeof` 的类型守卫

1.  30.2 我们何时需要窄化类型？

    1.  30.2.1 类型 `unknown` 通常过于通用

1.  30.3 我们在哪里可以使用类型守卫？

    1.  30.3.1 类型守卫也可以影响表达式的部分

1.  30.4 内置的类型守卫有哪些类型？

    1.  30.4.1 `typeof`、`instanceof`、`Array.isArray`

    1.  30.4.2 严格相等（`===`）和严格不等（`!==`）

    1.  30.4.3 真值检查

    1.  30.4.4 通过操作符 `in` 检查不同属性

    1.  30.4.5 检查共享属性值（区分联合）

1.  30.5 通过赋值进行窄化

    1.  30.5.1 使用显式类型窄化存储位置

    1.  30.5.2 没有显式类型的变量窄化

1.  30.6 我们可以在哪些存储位置使用类型窄化？

    1.  30.6.1 窄化点符号名称的类型

    1.  30.6.2 窄化数组元素类型

1.  30.7 用户定义的类型守卫函数

    1.  30.7.1 示例：`isNonNullable()`

    1.  30.7.2 示例：使用用户定义的类型守卫帮助 `.every()`

    1.  30.7.3 基于 `this` 的类型守卫

    1.  30.7.4 推断类型谓词

    1.  30.7.5 高级示例：用户定义的类型守卫 `isTypeOf()`

有时一个值 `nameOpt` 的类型不够具体——例如，类型 `null | string`。在我们对 `nameOpt` 做任何事情之前，我们必须检查它是否为 `null` 或字符串：

+   这样的检查条件称为 *类型守卫*。

+   检查的结果使得类型变得更加具体（更小），变为 `null` 或 `string`。这个过程称为 *窄化*。

在本章中，我们探讨类型守卫和窄化。

### 30.1 类型守卫和窄化

#### 30.1.1 示例：使用 `===` 的类型守卫

这是一个第一个例子：

```ts
function greet(nameOpt: null | string): string {
 if (nameOpt === null) { // (A)
 assertType<null>(nameOpt); // (B)
 nameOpt = 'anonymous'; // (C)
 assertType<string>(nameOpt);
 } else {
 assertType<string>(nameOpt);
 }
 assertType<string>(nameOpt); // (D)
 return `Hello ${nameOpt}!`;
}

```

`nameOpt` 的初始类型 `null | string` 过于通用：

+   在我们能够处理它之前，我们需要对其进行**缩小**——使其更加具体。

+   我们通过行 A 的 `if` 语句实现这一点。其条件被称为**类型守卫**：一个具有布尔结果的表达式，用于检查 `nameOpt` 的类型。

在行 B，我们可以看到，带有类型守卫的 `if` 语句确实缩小了 if-分支内 `nameOpt` 的类型：在其开始时，它是 `null`。我们还可以看到，在 else-分支中，`nameOpt` 的类型是 `string`。

行 C 的赋值也改变了 `nameOpt` 的类型。它并没有缩小其先前的类型，而是缩小了其原始类型。

**控制流分析**。在 `if` 语句的两个分支（行 D）之后，`nameOpt` 的类型是 `string`，因为这是两个分支结束时它的类型。所以 TypeScript 会追踪两个分支中发生的事情，并相应地调整类型。这种追踪被称为**控制流分析**。

**类型守卫的效果是静态和动态的**。有趣的是，类型守卫始终与运行时存在的值相关联。它在类型级别和 JavaScript 级别都有影响。

**缩小产生类型的子集**。在本章中，我们将类型解释为值的集合，如“什么是 TypeScript 中的类型？（两种视角” §13 中所述。缩小使类型变小：我们从一组值到一个适当的子集。

#### 30.1.2 示例：使用 `typeof` 的类型守卫

在以下示例中，我们在类型守卫中使用 `typeof`：

```ts
function getScore(value: number | string): number {
 if (typeof value === 'number') { // (A)
 assertType<number>(value);
 return value;
 }
 if (typeof value === 'string') { // (B)
 assertType<string>(value);
 return value.length;
 }
 throw new Error('Unexpected value: ' + value);
}

assert.equal(
 getScore('*****'), 5
);
assert.equal(
 getScore(3), 3
);

```

在此示例中，有两个类型守卫：一个在行 A，一个在行 B。

### 30.2 何时需要缩小类型？

这些是类型过于通用的例子：

+   可空类型：

    ```ts
    function func1(arg: null | string) {}
    function func2(arg: undefined | string) {}

    ```

+   区分联合:

    ```ts
    type Teacher = { kind: 'Teacher', teacherId: string };
    type Student = { kind: 'Student', studentId: string };
    type Attendee = Teacher | Student;

    function func3(attendee: Attendee) {}

    ```

+   可选参数的类型：

    ```ts
    function func4(arg?: string) {
     assertType<string | undefined>(arg);
    }

    ```

注意，这些类型都是联合类型！

#### 30.2.1 类型 `unknown` 通常过于通用

如果一个值具有 类型 `unknown`，我们几乎无法对其进行操作，必须首先缩小其类型（行 A）：

```ts
function parseStringLiteral(stringLiteral: string): string {
 // We use `unknown` instead of the less-safe `any`
 const result: unknown = JSON.parse(stringLiteral);
 if (typeof result === 'string') { // (A)
 return result;
 }
 throw new Error('Not a string literal: ' + stringLiteral);
}

```

换句话说：类型 `unknown` 太过于通用，我们必须对其进行缩小。从某种意义上讲，`unknown` 也是一种联合类型——所有类型的联合。

### 30.3 在哪里可以使用类型守卫？

在 `if` 语句中：

```ts
function f(value: null | string): void {
 if (value === null) {
 assertType<null>(value);
 } else {
 assertType<string>(value);
 }
}

```

在条件表达式中：

```ts
function f(value: null | string): void {
 const result = value === null
 ? assertType<null>(value)
 : assertType<string>(value)
 ;
}

```

在 `switch` 语句中：

```ts
function f(value: null | string): void {
 switch (typeof value) {
 case 'object':
 assertType<null>(value);
 break;
 case 'string':
 assertType<string>(value);
 break;
 }
}

```

#### 30.3.1 类型守卫也可以影响表达式的部分

如果逻辑与运算符 (`&&`) 的左侧是一个类型守卫，那么它会影响其右侧（可能包含更多 `&&` 的使用）：

```ts
function f(value: null | string): void {
 if (value !== null && value.length > 0) {
 // ...
 }
}

```

`&&` 操作符在其左侧为假值时停止评估。因此，如果我们到达右侧，我们可以确信 `value !== null` 是真的——这就是为什么我们可以访问 `value.length`。

使用逻辑或运算符 (`||`)，我们得到一个相关效果：

```ts
function f(value: null | string): void {
 if (value === null || value.length > 0) {
 // ...
 }
}

```

`||` 操作符在其左侧为真值时停止评估。因此，如果我们到达了右侧，我们可以确定 `value === null` 是假的——这就是为什么我们可以访问 `value.length`。

### 30.4 内置的类型守卫有哪些？

在本节中，我们探讨 TypeScript 的内置类型守卫（评估为 `true` 或 `false` 的表达式）。

#### 30.4.1 `typeof`、`instanceof`、`Array.isArray`

这些是三种常见的内置类型守卫：

```ts
function func(value: Function | Date | Array<number>): void {
 if (typeof value === 'function') {
 assertType<Function>(value);
 }

 if (value instanceof Date) {
 assertType<Date>(value);
 }

 if (Array.isArray(value)) {
 assertType<Array<number>>(value);
 }
}

```

#### 30.4.2 严格相等 (`===`) 和严格不等 (`!==`)

严格相等可以在类型守卫中使用：

```ts
function func(value: unknown): void {
 if (value === 'abc') {
 assertType<"abc">(value);
 }
}

```

如果联合类型的一个元素是单例类型（具有单个值），则我们可以使用 `===` 和 `!==` 进行缩小：

```ts
interface Book {
 title: null | string;
 isbn: string;
}

function getTitle(book: Book): string {
 if (book.title !== null) {
 assertType<string>(book.title);
 return book.title;
 }
 if (book.title === null) {
 assertType<null>(book.title);
 return '(Untitled)';
 }
 throw new Error();
}

```

`null` 是一个单例类型，其唯一成员是值 `null`。

#### 30.4.3 真值检查

##### 30.4.3.1 通过 `if` 进行真值检查

真值检查与相等检查相关。它们也充当类型守卫：

```ts
function f(value: null | string): void {
 if (value) {
 assertType<string>(value);
 } else {
 assertType<null | string>(value); // (A)
 }
}

```

初看，我们可能会认为所有字符串都进入 if 分支，而所有 null 都进入 else 分支。然而，情况并非如此：

+   Then 分支：`value` 既不是 `null` 也不是空字符串。

+   Else 分支：`value` 要么是 `null`，要么是空字符串。这解释了行 A 中的类型。

##### 30.4.3.2 建议：避免真值检查

真值检查有问题，因为它拒绝了非空值，如空字符串、零等。TypeScript 没有类型来表示“非空字符串”、“非零数字”等。

因此，我建议避免使用它——使用它的代码更难理解。相反，我们可以使用 `===` 和 `!==` 进行更明确的检查。使用明确的检查，前面的例子就不再那么令人困惑了：

```ts
function f(value: null | string): void {
 if (value !== null) {
 assertType<string>(value);
 } else {
 assertType<null>(value);
 }
}

```

##### 30.4.3.3 通过逻辑与 (`&&`) 和逻辑或 (`||`) 进行真值检查

逻辑与 (`&&`) 和逻辑或 (`||`) 操作符也可以执行真值检查：

```ts
function f1(value: null | string): void {
 if (value && value.length > 0) {
 // ...
 }
}
function f2(value: null | string): void {
 if (!value || value.length > 0) {
 // ...
 }
}

```

##### 30.4.3.4 `Boolean()` 不能用作类型守卫

`Boolean()` 函数不能用作类型守卫：

```ts
function f(value: null | string): void {
 if (Boolean(value)) {
 assertType<null | string>(value); // (A)
 }
}

```

然而，没有缩小：在行 A 中，`value` 的类型不是 `string`。

从原则上讲，`Boolean()` 可以被转换成一个类型守卫函数，但这会有几个缺点[several downsides](https://github.com/microsoft/TypeScript/issues/50387#issuecomment-2037968462)。

作为 `Boolean()` 的替代，我们可以使用前缀操作符逻辑否定 (`!`) 两次。这确实会缩小范围：

```ts
function f(value: null | string): void {
 if (!!value) {
 assertType<string>(value);
 }
}

```

然而，`Boolean()` 和 `!!` 都是真值检查，因此最好避免使用。

#### 30.4.4 通过操作符 `in` 检查不同的属性

`in` 操作符可以在类型守卫中使用——用于检查不同的属性：

```ts
type FirstOrSecond =
 | {first: string}
 | {second: string}
;
function func(firstOrSecond: FirstOrSecond): void {
 if ('first' in firstOrSecond) {
 assertType<{ first: string }>(firstOrSecond);
 }
}

```

注意以下检查将不会工作：

```ts
function func2(firstOrSecond: FirstOrSecond): void {
 // @ts-expect-error: Property 'first' does not exist on
 // type 'FirstOrSecond'. [...]
 if (firstOrSecond.first !== undefined) {
 // ···
 }
}

```

在这个案例中，问题在于，如果没有缩小范围，我们无法访问类型为 `FirstOrSecond` 的值的 `.first` 属性。

##### 30.4.4.1 运算符 `in` 也会缩小到单个属性

我们也可以使用 `in` 来缩小到单个属性：

```ts
function func(obj: object): void {
 if ('name' in obj) {
 assertType<object & Record<'name', unknown>>(obj);
 const value = obj.name;
 assertType<unknown>(value);
 }
}

```

这种缩小仅在 `in` 操作符的左侧是字符串字面量（而不是，例如，一个变量）时才有效。

#### 30.4.5 检查共享属性（区分联合）的值

在 区分联合 中，联合类型的组件具有（至少）一个共同的属性，其值对于每个组件都不同。这种属性称为 *区分符*。

检查区分符的值是一个类型守卫：

```ts
type Teacher = { kind: 'Teacher', teacherId: string };
type Student = { kind: 'Student', studentId: string };
type Attendee = Teacher | Student;

function getId(attendee: Attendee) {
 switch (attendee.kind) {
 case 'Teacher':
 assertType<{ kind: 'Teacher', teacherId: string }>(attendee);
 return attendee.teacherId;
 case 'Student':
 assertType<{ kind: 'Student', studentId: string }>(attendee);
 return attendee.studentId;
 default:
 throw new Error();
 }
}

```

在前面的例子中，`.kind` 是一个区分符：联合类型 `Attendee` 的每个组件都具有这个属性，并且具有唯一的值。

`if` 语句和等式检查与 `switch` 语句的工作方式类似：

```ts
function getId(attendee: Attendee) {
 if (attendee.kind === 'Teacher') {
 assertType<{ kind: 'Teacher', teacherId: string }>(attendee);
 return attendee.teacherId;
 } else if (attendee.kind === 'Student') {
 assertType<{ kind: 'Student', studentId: string }>(attendee);
 return attendee.studentId;
 } else {
 throw new Error();
 }
}

```

### 30.5 通过赋值进行缩小

#### 30.5.1 使用显式类型缩小存储位置

赋值缩小存储位置（如变量）的类型。如果一个变量 `x` 有显式类型，我们可以将其分配任何该类型的值——这会缩小其类型：

```ts
let x: null | string;
type _1 = Assert<Equal<
 typeof x,
 null | string
>>;

x = 'a';
type _2 = Assert<Equal<
 typeof x,
 string
>>;

x = null;
type _3 = Assert<Equal<
 typeof x,
 null
>>;

// @ts-expect-error: Type 'number' is not assignable to type 'string'.
x = 1;

```

在我们分配一个字符串后，`x` 的类型缩小到 `string`。在分配 `null` 后，`x` 的类型缩小到 `null`。最后，我们可以看到我们必须保持在 `null | string` 类型范围内——否则，我们会得到一个错误。

注意，使用类型守卫时，我们总是缩小当前类型，而使用赋值时，我们缩小原始类型。

#### 30.5.2 没有显式类型的缩小变量

如果 编译器选项 `noImplicitAny` 是激活的（如果 `strict` 是激活的，则它是激活的），那么每个存储位置都必须有一个类型——要么是推断类型，要么是显式定义的类型。唯一的例外是通过 `let`（`var` 的工作方式类似）定义的变量：

```ts
let x; // implicitly has type `any`
type _1 = Assert<Equal<
 typeof x, // (A)
 undefined
>>;
x = 1;
type _2 = Assert<Equal<
 typeof x,
 number
>>;
x = 'a';
type _3 = Assert<Equal<
 typeof x,
 string
>>;
x = true;
type _4 = Assert<Equal<
 typeof x,
 boolean
>>;

```

`x` 初始具有隐式类型 `any`。在行 A 中，我们观察到其类型，这就是为什么它的类型缩小到 `undefined`。我们没有限制可以分配的内容。每次，类型都会相应地缩小。没有限制，因为我们总是缩小 `x` 的原始类型：顶级类型 `any`。

如果我们用一个值初始化一个 `let` 变量，那么它没有隐式类型 `any`：

```ts
let x = 1;
type _ = Assert<Equal<
 typeof x,
 number
>>;
// @ts-expect-error: Type 'string' is not assignable to type 'number'.
x = 'a';

```

### 30.6 我们可以在哪些存储位置使用类型缩小？

#### 30.6.1 缩小点符号名称的类型

我们可以缩小属性的类型（甚至是通过属性名称链访问的嵌套属性）：

```ts
type MyType = {
 prop?: number | string,
};
function func(arg: MyType): void {
 if (typeof arg.prop === 'string') {
 assertType<string>(arg.prop); // (A)

 [].forEach((x) => {
 assertType<string | number | undefined>(arg.prop); // (B)
 });

 assertType<string>(arg.prop);

 arg = {};
 assertType<string | number | undefined>(arg.prop); // (C)
 }
}

```

让我们看看前面代码中的几个位置：

+   行 A：我们通过类型守卫缩小了 `arg.prop` 的类型。

+   行 B：回调可能执行得晚得多（想想异步代码），这就是为什么 TypeScript 在回调内部取消缩小。

+   行 C：前面的赋值也取消了缩小。

#### 30.6.2 缩小数组元素类型

##### 30.6.2.1 数组方法 `.every()` 缩小

我们可以使用方法 `.every()` 来缩小数组元素的类型：

```ts
function f(mixedValues: Array<undefined | null | number>): void {
 if (mixedValues.every(x => x !== undefined && x !== null)) {
 assertType<Array<number>>(mixedValues);

 // @ts-expect-error: Argument of type 'null' is not assignable to
 // parameter of type 'number'.
 mixedValues.push(null); // (A)
 }
}

```

有趣的是，TypeScript 不允许我们在将类型缩小到 `Array<number>` 之后将值 `null` 推送到 `mixedValues`（行 A）。

##### 30.6.2.2 数组方法 `.filter()` 产生具有更窄类型的数组

`.filter()` 产生具有更窄类型的数组（即，它实际上并没有缩小现有类型）：

```ts
const mixedValues = [1, undefined, 2, null];
assertType<(number | null | undefined)[]>(mixedValues);

const numbers = mixedValues.filter(
 x => x !== undefined && x !== null
);
assertType<number[]>(numbers);
assert.deepEqual(
 numbers,
 [1, 2]
);

```

### 30.7 用户定义的类型守卫函数

TypeScript 允许我们定义自己的类型守卫函数——例如：

```ts
function isFunction(value: unknown): value is Function {
 return typeof value === 'function';
}

```

返回类型 `value is Function` 是一个 *类型谓词*。它是 `isFunction()` 类型签名的一部分：

```ts
assertType<(value: unknown) => value is Function>(isFunction);

```

用户定义的类型守卫函数必须始终返回布尔值。如果 `isFunction(x)` 返回 `true`，TypeScript 将实际参数 `x` 的类型缩小到 `Function`：

```ts
function func(arg: unknown): void {
 if (isFunction(arg)) {
 assertType<Function>(arg); // type is narrowed
 }
}

```

注意，TypeScript 不关心我们如何计算用户定义的类型守卫函数的结果。这给了我们在检查方面很大的自由度。例如，我们可以将 `isFunction()` 实现如下：

```ts
function isFunction(value: any): value is Function {
 try {
 value(); // (A)
 return true;
 } catch {
 return false;
 }
}

```

注意，我们必须为参数 `value` 使用类型 `any`，因为类型 `unknown` 不允许我们在行 A 进行函数调用。

![图标“详情”](img/837806d7ec89826c3784b2e685feb762.png) **术语：“类型守卫函数”与“类型守卫”**

自定义类型守卫函数通常被称为自定义类型守卫——即使它们仅在类型守卫表达式中被调用（而不是类型守卫本身）。

#### 30.7.1 例子：`isNonNullable()`

实用类型 `NonNullable<T>` 从联合类型 `T` 中移除 `undefined` 和 `null`。在下一个例子中，我们使用它来定义一个自定义类型守卫：

```ts
function isNonNullable<T>(value: T): value is NonNullable<T> {
 return value !== undefined && value !== null;
}
function f1(arg: null | string) {
 if (isNonNullable(arg)) {
 assertType<string>(arg);
 }
}
function f2(arg: undefined | string) {
 if (isNonNullable(arg)) {
 assertType<string>(arg);
 }
}

```

#### 30.7.2 例子：使用用户定义的类型守卫帮助 `.every()`

让我们看看一个使用 `.every()` 的类型守卫不正常工作的例子。我们将使用以下实用类型，该类型在“用于构造函数的泛型类型：`Class<T>`”（§23.3）中有所解释。

```ts
type Class<T> = abstract new (...args: Array<any>) => T;

```

在行 A 的类型守卫之后，我们本应期望 `arr` 的类型为 `Array<T>`。然而，情况并非如此：

```ts
function f1<T>(arr: Array<unknown>, theClass: Class<T>): void {
 if (arr.every(x => x instanceof theClass)) { // (A)
 assertType<Array<unknown>>(arr)
 }
}

```

我们可以通过将行 A 的检查转换为用户定义的类型守卫 `isInstanceOf()` 来修复这个问题：

```ts
function f2<T>(arr: Array<unknown>, theClass: Class<T>): void {
 if (arr.every(x => isInstanceOf(x, theClass))) {
 assertType<Array<T>>(arr)
 }
}
function isInstanceOf<T>(value: unknown, theClass: Class<T>): value is T  {
 return value instanceof theClass;
}

```

#### 30.7.3 基于 `this` 的类型守卫

在我们可以使用 `this` 类型的位置，我们可以实现使用该类型的类型守卫（行 A 和行 B）

```ts
abstract class Shape {
 isCircle(): this is Circle { // (A)
 return this instanceof Circle;
 }
 isRectangle(): this is Rectangle { // (B)
 return this instanceof Rectangle;
 }
}
class Circle extends Shape {
 center = new Point();
 radius = 0;
}
class Rectangle extends Shape {
 corner1 = new Point();
 corner2 = new Point();
}
class Point {
 x = 0;
 y = 0;
}

```

类型守卫 `.isCircle()` 和 `.isRectangle()` 使我们能够从抽象超类 `Shape` 的类型转到其子类之一 - 例如：

```ts
function f(shape: Shape): void {
 if (shape.isCircle()) {
 assertType<Circle>(shape);
 }
}

```

##### 30.7.3.1 示例：缩小属性的类型

在以下代码中，我们使用基于 `this` 的类型守卫来缩小属性的类型：

```ts
class ValueContainer<T> {
 value: null | T = null;
 hasValue(): this is { value: T } & this {
 return this.value !== null;
 }
}

function f(stringContainer: ValueContainer<string>): void {
 assertType<null | string>(stringContainer.value);
 if (stringContainer.hasValue()) {
 assertType<string>(stringContainer.value);
 }
}

```

如果类型守卫 `.hasValue()` 返回 `true`，我们可以确信 `.value` 不是 `null`。因此，我们可以相应地缩小其类型。`& this` 在这种情况下实际上并不需要，但它确保了如果我们添加更多属性，该方法仍然有效：这意味着我们保留了所有的 `this`，但将其与 `.value` 具有更窄类型的类型相交。

#### 30.7.4 推断出的类型谓词

在某些情况下，TypeScript 可以推断类型谓词 - 例如：

```ts
const isNumber = (x: unknown) => typeof x === 'number';
assertType<
 (x: unknown) => x is number
>(isNumber);

const isNonNullable = <T>(x: T) => x != null;
assertType<
 <T>(x: T) => x is NonNullable<T>
>(isNonNullable);

```

这就是为什么数组方法 `.every()` 和 `.filter()` 可以通过不是类型守卫的回调函数缩小数组。

只有在以下情况下才能推断出类型谓词：

+   没有显式指定返回类型。

+   函数只有一个返回语句，没有隐式返回。

+   函数不会修改其参数。

+   函数返回一个类型守卫表达式。

对于真值检查，没有推断出类型谓词：

```ts
const isTruthy = (x: unknown) => !!x;
assertType<
 (x: unknown) => boolean
>(isTruthy);

```

#### 30.7.5 高级示例：用户定义的类型守卫 `isTypeOf()`

让我们将 JavaScript 操作符 `typeof` 转换为用户定义的类型守卫 `isTypeOf()` - 同时修复以下 `typeof` 错误（`null` 应该产生结果 `'null'`）：

```ts
> typeof null
'object'
> typeof {}
'object'

```

这就是使用 `isTypeOf()` 的样子：

```ts
//===== Using isTypeOf() at the JavaScript level =====
assert.equal(
 isTypeOf('abc', 'string'), true
);
assert.equal(
 isTypeOf(123, 'string'), false
);
// Fix `typeof` bug:
assert.equal(
 isTypeOf(null, 'null'), true
);
assert.equal(
 isTypeOf({}, 'null'), false
);
assert.equal(
 isTypeOf({}, 'object'), true
);
assert.equal(
 isTypeOf(null, 'object'), false
);

//===== Using isTypeOf() at the type level =====
function fn(value: unknown) {
 if (isTypeOf(value, 'string')) {
 assertType<string>(value);
 }

 // Fix `typeof` bug:
 if (isTypeOf(value, 'object')) {
 assertType<object>(value);
 }
 if (isTypeOf(value, 'null')) {
 assertType<null>(value);
 }
}

```

##### 30.7.5.1 通过条件类型实现 `isTypeOf()`

这是首次尝试在 TypeScript 中实现 `typeof`：

```ts
function isTypeOf<
 T extends string
>(
 value: unknown, typeString: T
): value is TypeStringToType<T> {
 switch (typeString) {
 case 'null':
 return value === null;
 case 'object':
 return typeof value === 'object' && value !== null;
 default:
 return typeof value === typeString;
 }
}

type TypeStringToType<S extends string> =
 S extends 'undefined' ? undefined :
 S extends 'null' ? null :
 S extends 'boolean' ? boolean :
 S extends 'number' ? number :
 S extends 'bigint' ? bigint :
 S extends 'string' ? string :
 S extends 'symbol' ? symbol :
 S extends 'object' ? object :
 S extends 'function' ? Function :
 never
 ;

```

泛型类型 `TypeStringToType` 使用 条件类型 将字符串字面量类型（如 `'number'`）转换为类型（如 `number`）。

##### 30.7.5.2 通过类型查找表实现 `isTypeOf()`

以下解决方案与上一个类似，但这次，我们不使用条件类型将字符串字面量类型转换为类型；我们使用对象字面量类型作为查找表：

```ts
function isTypeOf<
 T extends keyof TypeofLookupTable // (A)
>(
 value: unknown, typeString: T
): value is TypeofLookupTable[T] {
 switch (typeString) {
 case 'null':
 return value === null;
 case 'object':
 return typeof value === 'object' && value !== null;
 default:
 return typeof value === typeString;
 }
}

type TypeofLookupTable = {
 'undefined': undefined,
 'null': null,
 'boolean': boolean,
 'number': number,
 'bigint': bigint,
 'string': string,
 'symbol': symbol,
 'object': object,
 'function': Function,
};

```

查找表为我们提供了一个很好的好处：我们可以将 `typeString` 的类型限制为 `TypeofLookupTable` 的键 - 这意味着如果我们输入了拼写错误，我们会得到编译器错误：

```ts
// @ts-expect-error: Argument of type '"nmbr"' is not assignable to
// parameter of type 'keyof TypeofLookupTable'.
isTypeOf(123, 'nmbr')

```

（这种方法受到 [Ran Lottem](https://dev.to/krumpet/generic-type-guard-in-typescript-258l) 的代码的启发。）

##### 30.7.5.3 通过函数重载实现 `isTypeOf()`

另一个选项是使用 函数重载：

```ts
function isTypeOf(value: unknown, typeString: 'undefined'): value is undefined;
function isTypeOf(value: unknown, typeString: 'null'): value is null;
function isTypeOf(value: unknown, typeString: 'boolean'): value is boolean;
function isTypeOf(value: unknown, typeString: 'number'): value is number;
function isTypeOf(value: unknown, typeString: 'bigint'): value is bigint;
function isTypeOf(value: unknown, typeString: 'string'): value is string;
function isTypeOf(value: unknown, typeString: 'symbol'): value is symbol;
function isTypeOf(value: unknown, typeString: 'object'): value is object;
function isTypeOf(value: unknown, typeString: 'function'): value is Function;
function isTypeOf(value: unknown, typeString: string): boolean {
 switch (typeString) {
 case 'null':
 return value === null;
 case 'object':
 return typeof value === 'object' && value !== null;
 default:
 return typeof value === typeString;
 }
}

```

（这种方法是 [Nick Fisher](https://twitter.com/spadgos/status/1266839605883666432) 的想法。）
