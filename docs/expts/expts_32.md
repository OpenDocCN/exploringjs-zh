# 26 枚举和枚举模式

> 原文：[`exploringjs.com/ts/book/ch_enums.html`](https://exploringjs.com/ts/book/ch_enums.html)

(广告，请勿遮挡。)

1.  26.1 理解枚举

    1.  26.1.1 用例 1：具有原始值的常量命名空间

    1.  26.1.2 用例 2：具有唯一值的自定义类型

    1.  26.1.3 用例 3：具有对象值的常量命名空间

1.  26.2 TypeScript 枚举

    1.  26.2.1 数字枚举

    1.  26.2.2 字符串枚举

    1.  26.2.3 异构枚举

    1.  26.2.4 省略初始化器

    1.  26.2.5 引用枚举成员名称

    1.  26.2.6 枚举成员名称的大小写

    1.  26.2.7 枚举定义了一个值和一个类型

    1.  26.2.8 枚举类型接受的值

    1.  26.2.9 运行时枚举

    1.  26.2.10 使用枚举处理枚举用例

1.  26.3 TypeScript 常量枚举

    1.  26.3.1 运行时常量枚举

    1.  26.3.2 常量枚举的缺点

1.  26.4 枚举模式：枚举对象

    1.  26.4.1 提高对象字面量方式的方法

    1.  26.4.2 创建枚举对象的辅助函数

    1.  26.4.3 使用枚举对象处理枚举用例

    1.  26.4.4 在类型级别使用枚举对象的成员

    1.  26.4.5 作为属性值的符号

    1.  26.4.6 用例：枚举作为具有对象值的常量命名空间

1.  26.5 枚举模式：枚举类

1.  26.6 枚举模式：字符串字面量联合

    1.  26.6.1 实现字符串字面量联合

1.  26.7 更多我们可以用枚举做的事情

    1.  26.7.1 枚举值作为 Maps 中的键

    1.  26.7.2 枚举成员的键（字符串）和它们的值之间的映射

    1.  26.7.3 迭代枚举成员

    1.  26.7.4 指定位向量

    1.  26.7.5 通过 Zod 对枚举进行类型验证

1.  26.8 建议

TypeScript 内置了对 *枚举* 的支持，枚举基本上是常量的命名空间。在本章中，我们探讨它们是如何工作的，我们可以使用哪些模式，以及如何在这之间进行选择。

### 26.1 理解枚举的意义

对于 TypeScript，在非类型级别上只使用 JavaScript 已经变得很有吸引力，因为这使得编译过程更容易理解且更快（归功于一种称为 *类型剥离* 的技术）。这种编码风格可以通过编译器选项 `erasableSyntaxOnly`（ch_tsconfig-json.html#erasableSyntaxOnly) 来强制执行。

*枚举* 是少数几个非类型特性之一，它不是 JavaScript。这就是为什么，在检查了它们的工作原理之后，我们还将探讨 JavaScript 的替代方案：我们可以用来替代枚举的模式。为了找到它们，我们将关注三个用例。

#### 26.1.1 使用用例 1：具有原始值的常量命名空间

这个用例是关于分组相关常量——想想查找表——例如：

```ts
const Color_Red = '#FF0000';
const Color_Green = '#00FF00';
const Color_Blue = '#0000FF';

```

我们希望为这些常量创建一个命名空间，并通过 `Color.Red` 等方式访问它们。

#### 26.1.2 使用用例 2：具有唯一值的自定义类型

有时，我们想要定义一个具有有限值集的自定义类型——例如，用于表达程序状态：

```ts
const Status_Pending = 'Pending';
const Status_Ongoing = 'Ongoing';
const Status_Finished = 'Finished';
type Status =
 | typeof Status_Pending
 | typeof Status_Ongoing
 | typeof Status_Finished
;

```

对于这个用例，我们希望能够 检查完备性：当我们处理情况（例如，通过 `switch`）时，TypeScript 应在类型检查期间警告我们是否忘记了某个情况：

```ts
function describeStatus(status: Status): string {
 switch (status) {
 case Status_Pending:
 return 'Not yet';
 case Status_Ongoing:
 return 'Working on it...';
 case Status_Finished:
 return 'We are done';
 default:
 throw new UnexpectedValueError(status);
 }
}

```

这是 `UnexpectedValueError` 的简单版本：

```ts
class UnexpectedValueError extends Error {
 constructor(value: never) {
 // Only solution that can stringify undefined, null, symbols, and
 // objects without prototypes
 super('Unexpected value: ' + {}.toString.call(value));
 }
}

```

它是如何工作的？如果变量没有更多的值可以具有，TypeScript 会推断出类型 `never`。因此，这是 `status` 在 `switch` 检查了它所有可能的值之后的类型。构造函数 `UnexpectedValueError` 如果其参数 *没有* 类型 `never`，则会产生类型错误。有关更多信息和一个更复杂的 `UnexpectedValueError` 版本，请参阅 “`never` 的用例：编译时的完备性检查”（§15.4）。

#### 26.1.3 使用用例 3：具有对象值的常量命名空间

这是一个此类常量的例子：

```ts
const TextStyle_Bold = {
 key: 'Bold',
 html: 'b',
 latex: 'textbf',
};
const TextStyle_Italics = {
 key: 'Italics',
 html: 'i',
 latex: 'textit',
};

```

有趣的是，这个用例扩展了用例 1 和用例 2：

+   一方面，我们可以查找更多的值。

+   另一方面，我们可以为常量定义一个类型，然后获取具有更多信息的类型成员：

    ```ts
    type TextStyle = typeof TextStyle_Bold | typeof TextStyle_Italics;

    ```

TypeScript 枚举不支持这种用例。但有一些枚举模式可以做到。

### 26.2 TypeScript 枚举

#### 26.2.1 数字枚举

这是一个数字枚举：

```ts
enum NoYes {
 No = 0,
 Yes = 1, // trailing comma
}

assert.equal(NoYes.No, 0);
assert.equal(NoYes.Yes, 1);

```

这个枚举看起来和功能与对象字面量类似：

+   它有 *成员* `No` 和 `Yes`。

+   每个成员都有一个名称，并且（在这种情况下）通过一个 *初始化器* 显式地分配了一个值：一个等号后跟一个值。我们通过点操作符访问这些值：

    ```ts
    assert.equal(NoYes.No, 0);
    assert.equal(NoYes.Yes, 1);

    ```

+   在对象字面量中，允许并忽略尾随逗号。

这是 `NoYes` 的使用方式：

```ts
function toGerman(value: NoYes) {
 switch (value) {
 case NoYes.No:
 return 'Nein';
 case NoYes.Yes:
 return 'Ja';
 }
}
assert.equal(toGerman(NoYes.No), 'Nein');
assert.equal(toGerman(NoYes.Yes), 'Ja');

```

#### 26.2.2 字符串枚举

我们也可以使用字符串作为枚举成员值：

```ts
enum NoYes {
 No = 'No',
 Yes = 'Yes',
}

assert.equal(NoYes.No, 'No');
assert.equal(NoYes.Yes, 'Yes');

```

#### 26.2.3 异构枚举

最后一种枚举称为 *异构枚举*。异构枚举的成员值是数字和字符串的混合：

```ts
enum Enum {
 One = 'One',
 Two = 'Two',
 Three = 3,
 Four = 4,
}
assert.deepEqual(
 [Enum.One, Enum.Two, Enum.Three, Enum.Four],
 ['One', 'Two', 3, 4]
);

```

异构枚举不常用，因为它们的应用很少。

然而，TypeScript 只支持数字和字符串作为枚举成员值。其他值，如符号，是不允许的。

#### 26.2.4 省略初始化器

我们也可以完全省略初始化器——在这种情况下，TypeScript 会自动分配数字：

```ts
enum NoYes {
 No,
 Yes,
}
assert.equal(NoYes.No, 0);
assert.equal(NoYes.Yes, 1);

```

也可能只省略一些初始化器。然而，我的建议是避免这样做，要么一个都不省略，要么全部省略。我们不会深入探讨部分省略的确切工作方式，但这是一个快速示例：

```ts
enum Omissions {
 A,
 B,
 C = 'C',
 D = 'D',
 E = 8,
 // We can only omit the initializer
 // at the beginning or after a number
 F,
}
assert.equal(Omissions.A, 0);
assert.equal(Omissions.B, 1);
assert.equal(Omissions.C, 'C');
assert.equal(Omissions.D, 'D');
assert.equal(Omissions.E, 8);
assert.equal(Omissions.F, 9);

```

#### 26.2.5 引用枚举成员名称

与 JavaScript 对象类似，我们可以引用枚举成员的名称：

```ts
enum HttpRequestField {
 'Accept',
 'Accept-Charset',
 'Accept-Datetime',
 'Accept-Encoding',
 'Accept-Language',
}
assert.equal(HttpRequestField['Accept-Charset'], 1);

```

没有方法可以计算枚举成员的名称。对象字面量支持通过方括号计算属性键。

#### 26.2.6 枚举成员名称的大小写

在枚举或其他地方命名常量有几个先例：

+   传统上，JavaScript 使用全大写名称，这是它从 Java 和 C 继承的约定：

    +   `Number.MAX_VALUE`

    +   `Math.SQRT2`

+   已知符号是驼峰式命名，以小写字母开头，因为它们与属性名称相关：

    +   `Symbol.asyncIterator`

+   TypeScript 手册使用以大写字母开头的驼峰式命名。这是 TypeScript 的标准样式，我们也在 `NoYes` 枚举中使用了它。

#### 26.2.7 枚举定义了一个值和一个类型

考虑以下枚举：

```ts
enum ShapeKind { Circle, Rectangle }

```

##### 26.2.7.1 枚举 `ShapeKind` 作为值

一方面，`ShapeKind` 是一个值：

```ts
const value = ShapeKind.Circle;
assert.equal(value, 0);

```

##### 26.2.7.2 枚举 `ShapeKind` 作为类型

另一方面，`ShapeKind` 也是一个类型——其结构与它的值相当不同：

```ts
type _ = [
 // Type `ShapeKind` is assignable to and from
 // a union of number literal types
 Assert<Assignable<
 0 | 1, ShapeKind
 >>,
 Assert<Assignable<
 ShapeKind, 0 | 1
 >>,
 // Accordingly, the keys of type `ShapeKind` are
 // equal to the keys of type `number`
 Assert<Equal<
 keyof ShapeKind,
 keyof number
 >>,
];

```

换句话说：尽管枚举的值类似于对象，但其类型与对象类型（其键是属性键的类型等）相当不同。

##### 26.2.7.3 `ShapeKind` 作为枚举成员类型的命名空间

尽管 `ShapeKind` 基本上是一个联合类型，但它还额外是一个其成员类型的命名空间——例如，类型 `ShapeKind.Circle` 可以赋值给和来自字面量数字类型 `0`：

```ts
type _ = [
 Assert<Assignable<
 0, ShapeKind.Circle
 >>,
 Assert<Assignable<
 ShapeKind.Circle, 0
 >>,
];

```

这意味着我们可以在类型级别使用枚举成员类型——例如，用于参数：

```ts
function describeCircle(circle: ShapeKind.Circle): string {
 // ···
}

```

或者用于区分联合中的判别符：

```ts
type Shape =
 | {
 kind: ShapeKind.Circle,
 center: Point,
 }
 | {
 kind: ShapeKind.Rectangle,
 corner1: Point,
 corner2: Point,
 }
;
type Point = {
 x: number,
 y: number,
}

```

#### 26.2.8 枚举类型接受的值

##### 26.2.8.1 数字枚举类型接受的值

如果一个参数具有数字枚举类型，它接受枚举成员值和数字：

```ts
enum Fruit { Apple, Strawberry }
function check(_animal: Fruit) {}

check(Fruit.Apple);
check(0);

```

然而，我们只能使用枚举成员的值：

```ts
// @ts-expect-error: Argument of type '2' is not assignable to parameter of type 'Fruit'.
check(2);

```

##### 26.2.8.2 字符串枚举类型接受的值

如果一个参数具有字符串枚举类型，成员值被认为是唯一的——它不接受字符串：

```ts
enum Fruit { Apple='Apple', Strawberry='Strawberry' }
function check(_animal: Fruit) {}

check(Fruit.Apple);
// @ts-expect-error: Argument of type '"Apple"' is not assignable to parameter of type 'Fruit'.
check('Apple');

```

#### 26.2.9 运行时的枚举

TypeScript 将枚举编译为 JavaScript 对象。

##### 26.2.9.1 在运行时对枚举进行编号

例如，考虑以下枚举：

```ts
enum Animal { Dog, Cat }

```

TypeScript 将此枚举编译为：

```ts
var Animal;
(function (Animal) {
 Animal[Animal["Dog"] = 0] = "Dog";
 Animal[Animal["Cat"] = 1] = "Cat";
})(Animal || (Animal = {}));

```

在此代码中，执行以下赋值：

```ts
Animal["Dog"] = 0;
Animal["Cat"] = 1;

Animal[0] = "Dog";
Animal[1] = "Cat";

```

有两组赋值：

+   映射：前两个赋值将枚举成员名称映射到值。

+   反向映射：最后两个赋值将值映射到名称。

TypeScript 允许我们使用反向映射来查找枚举键，给定枚举值：

```ts
enum Animal { Dog, Cat }
assert.equal(
 Animal[0], 'Dog'
);

```

##### 26.2.9.2 运行时的字符串枚举

基于字符串的枚举在运行时具有更简单的表示形式。

考虑以下枚举。

```ts
enum Animal {
 Dog = 'DOG!',
 Cat = 'CAT!',
}

```

它被编译成以下 JavaScript 代码：

```ts
var Animal;
(function (Animal) {
 Animal["Dog"] = "DOG!";
 Animal["Cat"] = "CAT!";
})(Animal || (Animal = {}));

```

TypeScript 不支持基于字符串的枚举的反向映射。

#### 26.2.10 使用枚举处理枚举用例

让我们回到本章开头提到的用例。

##### 26.2.10.1 用例：具有原始值的常量的命名空间

我们可以使用枚举作为具有原始值的常量的命名空间：

```ts
enum Color {
 Red = '#FF0000',
 Green = '#00FF00',
 Blue = '#0000FF',
}

```

枚举适用于此用例——但它们的值只能是数字或字符串。

##### 26.2.10.2 用例：具有唯一值的自定义类型

以下枚举定义了一个具有唯一值的自定义类型：

```ts
enum Status {
 Pending = 'Pending',
 Ongoing = 'Ongoing',
 Finished = 'Finished',
}

```

通过`=`显式指定字符串值的优点：我们获得更多的类型安全，并且不会意外地使用与枚举值相等的字符串，而期望`Status`。

注意，我们不需要显式定义类型——`Status`已经是一个类型。

支持枚举的完备性检查：

```ts
function describeStatus(status: Status): string {
 switch (status) {
 case Status.Pending:
 return 'Not yet';
 case Status.Ongoing:
 return 'Working on it...';
 case Status.Finished:
 return 'We are done';
 default:
 throw new UnexpectedValueError(status);
 }
}
assert.equal(
 describeStatus(Status.Pending),
 'Not yet'
);

```

### 26.3 TypeScript const 枚举

如果枚举以关键字`const`开头，则它在运行时没有表示。相反，直接使用其成员的值。

#### 26.3.1 运行时的 const 枚举

考虑以下 const 枚举：

```ts
const enum Vegetable {
 Carrot = 'Carrot',
 Onion = 'Onion',
}

function toGerman(vegetable: Vegetable) {
 switch (vegetable) {
 case Vegetable.Carrot:
 return 'Karotte';
 case Vegetable.Onion:
 return 'Zwiebel';
 }
}

```

如果我们编译它，const 枚举在运行时不表示。只有其成员的值保留：

```ts
function toGerman(vegetable) {
 switch (vegetable) {
 case "Carrot" /* Vegetable.Carrot */:
 return 'Karotte';
 case "Onion" /* Vegetable.Onion */:
 return 'Zwiebel';
 }
}

```

将其与正常枚举`Vegetable`的编译输出进行比较：

```ts
var Vegetable;
(function (Vegetable) {
 Vegetable["Carrot"] = "Carrot";
 Vegetable["Onion"] = "Onion";
})(Vegetable || (Vegetable = {}));
function toGerman(vegetable) {
 switch (vegetable) {
 case Vegetable.Carrot:
 return 'Karotte';
 case Vegetable.Onion:
 return 'Zwiebel';
 }
}

```

#### 26.3.2 const 枚举的缺点

对于大多数项目，最好避免使用 const 枚举：

+   与正常枚举类似，如果编译器选项``erasableSyntaxOnly`处于活动状态，则不允许使用它们。

+   如果一个库包导出一个 const 枚举，客户端将无法使用编译器选项``isolatedModules`。

+   编译时的枚举值可以与运行时的枚举值不同——这可能导致意外的错误。

[TypeScript 手册](https://www.typescriptlang.org/docs/handbook/enums.html#const-enum-pitfalls)更详细地描述了这些和其他陷阱。

### 26.4 枚举模式：枚举对象

是时候看看我们可以用代替枚举的纯 JavaScript 模式了。一个常见的模式是通过对象字面量定义“枚举对象”：

```ts
const Tree = {
 Maple: 'MAPLE',
 Oak: 'OAK',
};

```

#### 26.4.1 改进对象字面量的方法

我们刚刚看到了枚举对象的基本形式。我们可以通过几种方式改进这个模式：

1.  通过`{} as const`创建常量对象

1.  通过`Object.freeze({})`创建冻结对象

1.  通过`{__proto__: null}`设置`null`原型

#1 是必须的，如果我们想从一个枚举对象中推导出类型。其他两个改进是可选的。它们会产生更好的结果，但也会增加视觉上的杂乱。

##### 26.4.1.1 `as const`对象字面量：为枚举值和键推导类型

如果我们将`as const`应用于对象字面量，我们将在类型级别获得更具体的属性值：

```ts
const Tree = {
 Maple: 'MAPLE',
 Oak: 'OAK',
};
type _1 = Assert<Equal<
 typeof Tree,
 {
 Maple: string,
 Oak: string,
 }
>>;

const TreeAsConst = {
 Maple: 'MAPLE',
 Oak: 'OAK',
} as const;
type _2 = Assert<Equal<
 typeof TreeAsConst,
 {
 readonly Maple: 'MAPLE',
 readonly Oak: 'OAK',
 }
>>;

```

**为属性值推导类型。**这有什么用？它使我们能够为`Tree`的属性值创建类型：

```ts
type _ = [
 Assert<Equal<
 ValueOf<typeof Tree>,
 string
 >>,
 Assert<Equal<
 ValueOf<typeof TreeAsConst>,
 'MAPLE' | 'OAK'
 >>,
];

```

能够为枚举对象值创建类型将帮助我们处理“具有唯一值的自定义类型”用例。

辅助类型`ValueOf`看起来是这样的：

```ts
type ValueOf<Obj> = Obj[keyof Obj];

```

[*索引访问类型* `Obj[K]`](ch_computing-with-types-overview.html#indexed-access-types) 包含所有键在`K`中的属性的值。

**为属性键推导类型。**我们还可以为`Tree`的键推导类型。但为此，我们不需要`as const`：

```ts
type _3 = Assert<Equal<
 keyof (typeof Tree),
 'Maple' | 'Oak'
>>;
type _4 = Assert<Equal<
 keyof (typeof TreeAsConst),
 'Maple' | 'Oak'
>>;

```

##### 26.4.1.2 冻结对象字面量：运行时无修改

在 JavaScript 级别上，我们可以冻结对象：

```ts
const TreeFrozen = Object.freeze({
 Maple: 'MAPLE',
 Oak: 'OAK',
});

```

这有助于我们在运行时，因为`TreeFrozen`不能被改变：

```ts
assert.throws(
 () => TreeFrozen.newProp = true,
 /^TypeError: Cannot add property newProp, object is not extensible$/
);
assert.throws(
 () => TreeFrozen.Maple = 'Ash',
 /^TypeError: Cannot assign to read only property 'Maple'/
);

```

在类型级别上，`Object.freeze()`与`as const`具有相同的效果：

```ts
const frozenObject = Object.freeze({
 prop: 123
});
type _ = Assert<Equal<
 typeof frozenObject,
 {
 readonly prop: 123,
 }
>>;

```

##### 26.4.1.3 具有`null`原型的对象字面量：无继承属性

我们可以使用伪属性键`__proto__`将`constants`的原型设置为`null`。这是一个好习惯，因为这样我们就不必处理继承属性：

考虑以下两个版本的`Tree`：

```ts
const Tree = {
 Maple: 'MAPLE',
 Oak: 'OAK',
};
const TreeProtoNull = {
 __proto__: null,
 Maple: 'MAPLE',
 Oak: 'OAK',
};

```

我们不能使用`in`运算符来检查枚举`Tree`是否具有给定的键，因为它还考虑了继承属性——这些属性不是枚举成员：

```ts
assert.equal(
 'toString' in Tree,
 true
);
assert.equal(
 'toString' in TreeProtoNull,
 false
);

```

我们还可以读取继承属性，这看起来就像枚举`Tree`有比实际更多的成员：

```ts
assert.equal(
 typeof Tree.toString,
 'function'
);
assert.equal(
 TreeProtoNull.toString,
 undefined
);

```

然而，`Object.keys()`和`Object.values()`是安全的，因为它们只考虑自己的（非继承的）属性：

```ts
assert.deepEqual(
 Object.keys(Tree),
 ['Maple', 'Oak']
);
assert.deepEqual(
 Object.keys(TreeProtoNull),
 ['Maple', 'Oak']
);

assert.deepEqual(
 Object.values(Tree),
 ['MAPLE', 'OAK']
);
assert.deepEqual(
 Object.values(TreeProtoNull),
 ['MAPLE', 'OAK']
);

```

##### 26.4.1.4 `__proto__`不是已经被弃用了吗？

注意，`__proto__`也存在于`Object.prototype`中，作为一个 getter 和 setter。这个特性已被弃用，转而使用`Object.getPrototypeOf()`和`Object.setPrototypeOf()`。然而，这与在对象字面量中使用此名称是不同的——它没有被弃用。

更多信息，请参阅“探索 JavaScript”的以下部分：

+   [“使用对象作为字典的陷阱”](https://exploringjs.com/js/book/ch_objects.html#the-pitfalls-of-using-an-object-as-a-dictionary)

+   [“与原型一起工作的技巧”](https://exploringjs.com/js/book/ch_objects.html#tips-for-working-with-prototypes)

+   [“`Object.prototype.__proto__` (访问器)”](https://exploringjs.com/js/book/ch_classes.html#Object.prototype.__proto__)

##### 26.4.1.5 `{__proto__}`的注意事项：类型级别的额外属性

如果一个对象字面量使用`.__proto__`，那么 TypeScript 会在类型级别包含该属性：

```ts
const TreeProtoNull = {
 __proto__: null,
 Maple: 'MAPLE',
 Oak: 'OAK',
} as const;
type _ = Assert<Equal<
 typeof TreeProtoNull,
 {
 readonly __proto__: null;
 readonly Maple: 'MAPLE';
 readonly Oak: 'OAK';
 }
>>;

```

我希望情况不是这样——鉴于`.__proto__`不是一个真正的属性（[相关 GitHub 问题](https://github.com/microsoft/TypeScript/issues/38385)）。

作为结果，我们必须手动排除`TreeProtoNull`通过`ValueOf`（行 A）推导类型时的键`'__proto__'`。

```ts
type TreeProtoNullType = ValueOf<typeof TreeProtoNull>;
// 123 | null

type _Y = [
 Assert<Equal<
 ValueOf<typeof TreeProtoNull>,
 'MAPLE' | 'OAK' | null
 >>,
 Assert<Equal<
 ValueOf<Omit<typeof TreeProtoNull, '__proto__'>>, // (A)
 'MAPLE' | 'OAK'
 >>,
];

```

另一种解决方案是创建一个辅助函数来创建枚举，该函数在运行时将原型设置为`null`，但在类型级别上不暴露`.__proto__`。我们将在下一节中这样做。

#### 26.4.2 创建枚举对象的辅助函数

这就是我们使用前一小节中提到的所有改进后的枚举对象模式的样子：

```ts
const Tree = Object.freeze({
 __proto__: null,
 Maple: 'MAPLE',
 Oak: 'OAK',
});

```

注意，我们不需要`as const`，因为`Object.freeze()`具有相同的效果。

一个辅助函数可以使这段代码稍微不那么冗长。结果有一个具有特定属性值类型的类型，我们可以从中推导出`Tree`的类型：

```ts
const Tree = createEnum({
 Maple: 'MAPLE',
 Oak: 'OAK',
});
type _ = Assert<Equal<
 typeof Tree,
 {
 readonly Maple: 'MAPLE';
 readonly Oak: 'OAK';
 }
>>;

```

这就是辅助函数：

```ts
/**
 * Returns an enum object. Adds the following improvements:
 * - Sets the prototype to `null`.
 * - Freezes the object.
 * - The result has the same type as if `as const` had been applied.
 */
function createEnum<
 // The two type variables are necessary so that the result has specific
 // property value types.
 T extends { [idx: string]: V },
 V extends
 | undefined | null | boolean | number | bigint | string | symbol 
 | object
>(enumObj: T): Readonly<T> {
 // Copying `enumObj` is better for performance than Object.setPrototypeOf()
 return Object.freeze({
 __proto__: null,
 ...enumObj,
 });
}

```

#### 26.4.3 使用枚举对象处理枚举用例

##### 26.4.3.1 用例：具有原始值常量的命名空间

对于这个用例，一个对象字面量是一个非常好的替代方案：

```ts
const Color = {
 Red: '#FF0000',
 Green: '#00FF00',
 Blue: '#0000FF',
};

```

我们可以使用`null`原型和冻结，但在这个情况下它们不是必需的。

##### 26.4.3.2 用例：具有唯一值的自定义类型

让我们使用对象字面量来定义枚举的值部分（我们将在下一部分讨论类型部分）：

```ts
const Status = {
 Pending: 'Pending',
 Ongoing: 'Ongoing',
 Finished: 'Finished',
} as const; // (A)

```

多亏了行 A 中的`as const`，我们可以从`Status`推导出一个类型：

```ts
type StatusType = ValueOf<typeof Status>;
type _ = Assert<Equal<
 StatusType, 'Pending' | 'Ongoing' | 'Finished'
>>;

```

实用类型`ValueOf`是在之前定义的。

为什么这个类型叫`StatusType`而不是`Status`？由于 TypeScript 中值和类型的命名空间是分开的，我们确实可以使用相同的名称。然而，我在使用 Visual Studio Code 重命名值和类型时遇到了问题：你不能同时进行这两项操作，VSC 会因此困惑，因为导入`Status`会同时导入值和类型。

使用名称`StatusType`而不是`TStatus`的好处是，前者会在`Status`的自动完成中显示。

##### 26.4.3.3 穷举性检查

TypeScript 支持对字面量类型联合的穷举性检查。这正是`StatusType`所做的事情。因此，我们可以使用与枚举相同的模式：

```ts
function describeStatus(status: StatusType): string { // (A)
 switch (status) {
 case Status.Pending:
 return 'Not yet';
 case Status.Ongoing:
 return 'Working on it...';
 case Status.Finished:
 return 'We are done';
 default:
 throw new UnexpectedValueError(status);
 }
}
assert.equal(
 describeStatus(Status.Pending),
 'Not yet'
);

```

注意，在行 A 中，`status` 的类型是 `StatusType`，而不是 `Status`（它是一个值）。

#### 26.4.4 在类型级别使用枚举对象的成员

枚举定义了一个值、一个类型以及成员的类型。对于枚举对象，我们必须手动创建后两者。我们已经从一个枚举对象中推导出了一个类型。那么成员的类型呢？考虑以下枚举对象：

```ts
const ShapeKind = {
 Circle: 0,
 Rectangle: 1,
} as const;

```

成员 `ShapeKind.Circle` 和 `ShapeKind.Rectangle` 只作为值存在，而不是作为类型：

```ts
// @ts-expect-error: Cannot find namespace 'ShapeKind'.
type _ = ShapeKind.Circle;

```

因此，如果我们想在类型级别使用这些值，我们必须对它们应用 `typeof`（行 A 和行 B）：

```ts
type Shape =
 | {
 key: typeof ShapeKind.Circle, // (A)
 center: Point,
 }
 | {
 key: typeof ShapeKind.Rectangle, // (B)
 corner1: Point,
 corner2: Point,
 }
;
type Point = {
 x: number,
 y: number,
}

```

#### 26.4.5 符号作为属性值

使用字符串作为枚举对象的属性值的一个缺点是它们不是唯一的：派生类型接受属性值和通过字符串字面量创建的字符串（如果它们相等的话）。如果我们使用符号，我们可以获得更多的类型安全性：

```ts
const Pending = Symbol('Pending');
const Ongoing = Symbol('Ongoing');
const Finished = Symbol('Finished');

const Status = {
 Pending,
 Ongoing,
 Finished,
} as const;

assertType<typeof Pending>(Status.Pending);

type StatusType = ValueOf<typeof Status>;
type _ = Assert<Equal<
 StatusType,
 typeof Pending | typeof Ongoing | typeof Finished
>>;

```

实用类型 `ValueOf` 之前已经定义过 此处。

这似乎过于复杂：为什么在使用之前先声明符号的变量？为什么不直接在对象字面量中创建符号？唉，这是 `as const` 对象中符号的当前限制——它们不会产生唯一类型（[相关 GitHub 问题](https://github.com/microsoft/TypeScript/issues/54100)）：

```ts
const Status = {
 Pending: Symbol('Pending'),
 Ongoing: Symbol('Ongoing'),
 Finished: Symbol('Finished'),
} as const;

// Alas, the type of Status.Pending is not `typeof Pending`
assertType<symbol>(Status.Pending);

// The derived type is `symbol`, not a union type
type StatusType = ValueOf<typeof Status>;
type _ = Assert<Equal<
 StatusType,
 symbol
>>;

```

#### 26.4.6 使用枚举作为具有对象值的常量的命名空间

有时，有一个类似于枚举的结构来查找更丰富的数据是有用的——存储在对象中。这是枚举无法做到的，但通过枚举对象是可能的：

```ts
// This type is optional: It constrains the property values
// of `TextStyle` but has no other use.
type TextStyleProp = {
 key: string,
 html: string,
 latex: string,
};
const TextStyle = {
 Bold: {
 key: 'Bold',
 html: 'b',
 latex: 'textbf',
 },
 Italics: {
 key: 'Italics',
 html: 'i',
 latex: 'textit',
 },
} as const satisfies Record<string, TextStyleProp>;

type TextStyleType = ValueOf<typeof TextStyle>;
type ValueOf<Obj> = Obj[keyof Obj];

```

因为 `TextStyle` 的每个属性值都有具有唯一值的 `.key` 属性，所以 `TextStyleType` 是一个 区分联合。

##### 26.4.6.1 完备性检查

由于 `TextStyleType` 是一个区分联合，我们可以进行完备性检查：

```ts
function describeTextStyle(textStyle: TextStyleType): string {
 switch (textStyle.key) {
 case TextStyle.Bold.key:
 return 'Bold text';
 case TextStyle.Italics.key:
 return 'Text in italics';
 default:
 throw new UnexpectedValueError(textStyle); // No `.key`!
 }
}

```

在 `default` 情况下，在我们检查了 `textStyle.key` 可以有的所有值之后，`textStyle` 本身具有 `never` 类型。

### 26.5 枚举模式：枚举类

我们还可以使用类作为枚举——这是从 Java 借用的一个模式：

```ts
class TextStyle {
 static Bold = new TextStyle({
 html: 'b',
 latex: 'textbf',
 });
 static Italics = new TextStyle({
 html: 'i',
 latex: 'textit',
 });
 html: string;
 latex: string;
 constructor(props: TextStyleProps) {
 this.html = props.html;
 this.latex = props.latex;
 }
 wrapHtml(html: string): string {
 return `<${this.html}>${html}</${this.html}>`;
 }
}
type TextStyleProps = {
 html: string,
 latex: string,
};
assert.equal(
 TextStyle.Bold.wrapHtml('Hello!'),
 '<b>Hello!</b>'
);

```

我们可以创建一个具有 `TextStyle` 静态属性的类型：

```ts
type TextStyleKeys = EnumKeys<typeof TextStyle>;
type _1 = Assert<Equal<
 TextStyleKeys, 'Bold' | 'Italics'
>>;

type EnumKeys<T> = Exclude<keyof T, 'prototype'>;

// Why exclude 'prototype'?
type _2 = Assert<Equal<
 keyof typeof TextStyle,
 'prototype' | 'Bold' | 'Italics'
>>;

```

枚举类的优点之一是我们可以使用方法为枚举值添加行为。缺点是没有任何简单的方法来进行完备性检查：对于具有对象属性值的 `as const` 枚举对象，我们可以创建一个关联的区分联合类型——在那里我们可以检查完备性。然而，类 `TextStyle` 的每个静态属性只是具有 `TextStyle` 类型（`TextStyle` 实例的类型）并且这阻止了我们创建区分联合。

`Object.keys()`和`Object.values()`会忽略`TextStyle`的非可枚举属性，例如`.prototype` – 这就是为什么我们可以使用它们来枚举键和值 – 例如：

```ts
assert.deepEqual(
 // TextStyle.prototype is non-enumerable
 Object.keys(TextStyle),
 ['Bold', 'Italics']
);

```

### 26.6 枚举模式：字符串字面量联合

当涉及到定义具有固定成员集的类型时，字符串字面量联合是枚举的一个有趣替代品：

```ts
type Activation = 'Active' | 'Inactive';

```

这个模式的优缺点是什么？

优点：

+   这是一个简洁且简单的解决方案。

+   它支持详尽性检查。

+   在 Visual Studio Code 中重命名成员效果相当好。

缺点：

+   类型成员不是唯一的。我们可以通过使用符号来改变这一点，但这样我们会失去许多字符串字面量联合类型的便利性 – 例如，我们不得不导入值。

+   我们不能为联合的元素使用 JSDoc 注释。这意味着我们也不能`@deprecate`它们。

#### 26.6.1 将字符串字面量联合具体化

*具体化*意味着在对象级别（想想 JavaScript 值）为在元级别（想想 TypeScript 类型）存在的实体创建一个实体。如果我们想迭代它们的元素（这些元素在运行时不存在），我们就需要这样做。其他枚举模式默认支持这一点。

##### 26.6.1.1 从集合推导字符串字面量联合

我们可以使用集合来具体化字符串字面量联合类型：

```ts
const activation = new Set([
 'Active',
 'Inactive',
] as const);
assertType<Set<'Active' | 'Inactive'>>(activation);

// @ts-expect-error: Argument of type '"abc"' is not assignable to
// parameter of type '"Active" | "Inactive"'.
activation.has('abc');
 // Auto-completion works for arguments of .has(), .delete() etc.

// Let’s turn the Set into a string literal union
type Activation = SetElementType<typeof activation>;
type _ = Assert<Equal<
 Activation, 'Active' | 'Inactive'
>>;

type SetElementType<S extends Set<any>> =
 S extends Set<infer Elem> ? Elem : never;

```

##### 26.6.1.2 从数组推导字符串字面量联合

当涉及到将字符串字面量联合具体化时，集合通常是最佳选择，因为我们可以在运行时检查给定的字符串是否是成员。然而，我们也可以使用数组来做到这一点：

```ts
const activation = [
 'Active',
 'Inactive',
] as const;
type Activation = (typeof activation)[number];
type _ = Assert<Equal<
 Activation,
 'Active' | 'Inactive'
>>;

```

### 26.7 我们可以用枚举做更多的事情

在本节中，我们探讨我们可以用枚举做更多的事情。我们将主要使用枚举对象，但偶尔也会提到枚举和其他枚举模式。

#### 26.7.1 枚举值作为 Map 中的键

有时我们希望使用枚举值来查找其他值。让我们探索以下枚举对象是如何工作的：

```ts
const Pending = Symbol('Pending');
const Ongoing = Symbol('Ongoing');
const Finished = Symbol('Finished');
const Status = {
 Pending,
 Ongoing,
 Finished,
} as const;
type StatusType = ValueOf<typeof Status>;

```

实用类型`ValueOf`是在之前定义的。

以下地图使用`Status`的值作为键：

```ts
const statusPairs = [
 [Status.Pending, 'not yet'],
 [Status.Ongoing, 'working on it'],
 [Status.Finished, 'finished'],
] as const;

type StatusMapKey = (typeof statusPairs)[number][0];
const statusMap = new Map<StatusMapKey, string>(statusPairs);
assertType<
 Map<
 typeof Pending | typeof Ongoing | typeof Finished,
 string
 >
>(statusMap);

```

如果你想知道为什么我们没有直接使用`statusPairs`的值作为`new Map()`的参数并省略类型参数：TypeScript 无法推断键为符号时的类型参数，并报告编译时错误。使用字符串时，代码会更简单：

```ts
const statusMap2 = new Map([ // no type parameters!
 ['Pending', 'not yet'],
 ['Ongoing', 'working on it'],
 ['Finished', 'finished'],
] as const);
assertType<
 Map<
 'Pending' | 'Ongoing' | 'Finished',
 'not yet' | 'working on it' | 'finished'
 >
>(statusMap2);

```

作为最后一步，手动检查我们是否使用了所有枚举值：

```ts
type _ = Assert<Equal<
 MapKey<typeof statusMap>, StatusType // (A)
>>;
type MapKey<M extends Map<any, any>> =
 M extends Map<infer K, any> ? K : never;

```

在行 A 中，我们从`statusMap`中提取键的类型，并要求它等于`StatusType`。

##### 26.7.1.1 通过`Record`进行详尽性检查

如果我们使用`Record`，TypeScript 可以检查联合类型是否被详尽使用。

```ts
Record<UnionType, T>

```

然而，这种联合类型只能包含 `string`、`number` 或 `symbol` 的子类型元素。这意味着 `Record` 对于字符串字面量类型的联合工作得很好：

```ts
type Status = 'Pending' | 'Ongoing' | 'Finished';
const statusMap = {
 'Pending': 'not yet',
 'Ongoing': 'working on it',
 // @ts-expect-error: Type '{ Pending: string; Ongoing: string; }' does
 // not satisfy the expected type 'Record<Status, string>'. Property
 // 'Finished' is missing in type '{ Pending: string; Ongoing: string; }'
 // but required in type 'Record<Status, string>'.
} satisfies Record<Status, string>;

```

#### 26.7.2 枚举成员键（字符串）与值之间的映射

有时将枚举键映射到值或反之亦然很有用。这种用法的一个重要用例是从 JSON 中反序列化符号或对象的枚举并将其序列化为 JSON。

```ts
const Pending = Symbol('Pending');
const Ongoing = Symbol('Ongoing');
const Finished = Symbol('Finished');
const Status = {
 Pending,
 Ongoing,
 Finished,
} as const;

```

##### 26.7.2.1 从枚举键到枚举值

将枚举成员的键映射到其值的一个用例是解析 JSON 数据：

```ts
function parseEnumKey<
 E extends Record<string, unknown>
>(enumObject: E, enumKey: string): E[keyof E] {
 if (!Object.hasOwn(enumObject, enumKey)) {
 throw new TypeError('Unknown key: ' + {}.toString.call(enumKey));
 }
 return enumObject[enumKey] as any;
}
assert.equal(
 parseEnumKey(Status, 'Ongoing'),
 Status.Ongoing
);

```

我们可以使 `enumKey` 的类型和返回类型都更具体（见下一小节），但那时我们再也不能解析 `string` 类型的值了。

##### 26.7.2.2 从枚举值到枚举键

将枚举成员的值映射到其键的一个用例是创建 JSON 数据：

```ts
function stringifyEnumValue<
 E extends object,
 V extends E[keyof E]
>(enumObject: E, enumValue: V): keyof E {
 for (const [key, value] of Object.entries(enumObject)) {
 if (enumValue === value) {
 return key as any;
 }
 }
 throw new TypeError('Unknown value: ' + {}.toString.call(enumValue));
}
assert.equal(
 stringifyEnumValue(Status, Status.Ongoing),
 'Ongoing'
);

```

#### 26.7.3 遍历枚举成员

遍历枚举成员的一个用例是创建一个列出枚举中收集的选项的用户界面。哪些枚举模式支持迭代？

+   我们只能遍历字符串枚举——因为具有数字成员的枚举具有反向映射。

+   对于枚举对象 `enumObj`，我们可以使用 `Object.keys(enumObj)` 和 `Object.values(enumObj)`。

+   对于枚举类 `enumClass`，我们可以使用 `Object.keys(enumClass)` 和 `Object.values(enumClass)`。枚举类的自有属性 `enumClass.prototype` 不可枚举，因此这两种方法都会忽略它。

+   如果我们要遍历字符串字面量联合的值，我们必须通过 Set 或 Array 来具体化它。

#### 26.7.4 指定位向量

枚举的一个用例是指定位向量（多个独立的位标志）。

##### 26.7.4.1 通过位掩码指定位向量

指定位向量的传统方式是通过位掩码：

```ts
const Permission = {
 Read:    1 << 2, // bit 2
 Write:   1 << 1, // bit 1
 Execute: 1 << 0, // bit 0
}
function setPermission(filePath: string, permission: number): void {
 // ···
}
setPermission(
 'read-and-write.txt',
 Permission.Read | Permission.Write
);

```

关于此类位操作的更多信息，请参阅“探索 JavaScript”中的[“位运算符”](https://exploringjs.com/js/book/ch_numbers.html#bitwise-operators)部分。

##### 26.7.4.2 通过 Set 指定位向量

指定位向量的另一种选项是通过 Set。我通常更喜欢这个选项。

```ts
const Read = Symbol('Read');
const Write = Symbol('Write');
const Execute = Symbol('Execute');
const Permission = {
 Read, Write, Execute
} as const;
type PermissionType = (typeof Permission)[keyof typeof Permission];

function setPermission(filePath: string, permission: Set<PermissionType>): void {
 // ···
}
setPermission(
 'read-and-write.txt',
 new Set([Permission.Read, Permission.Write])
);

```

#### 26.7.5 通过 Zod 对枚举进行类型验证

[Zod](https://zod.dev/) 是一个用于 *验证* 解析数据的工具（通常是 JSON）——检查运行时数据是否与预期的类型匹配。这在使用无类型数据时增加了类型安全性。

Zod 也支持枚举，但建议使用字符串字面量联合。它们必须通过字符串数组定义，因为 Zod 需要它在运行时可以使用的数据。

```ts
import { z } from 'zod';

const ActivationSchema = z.enum(['Active', 'Inactive']);

// Derive a type from the schema
type Activation = z.infer<typeof ActivationSchema>;
type _ = Assert<Equal<
 Activation,
 'Active' | 'Inactive'
>>;

// Use the schema to “parse” data at runtime
// (check that it has the correct type)
const activation = ActivationSchema.parse('Inactive');
assertType<Activation>(activation);
assert.throws(
 () => ActivationSchema.parse('HELLO')
);

```

### 26.8 建议

我们应该如何在枚举和各种枚举模式之间进行选择？

让我们来看看用例：

+   带有原始值的常量的命名空间：

    +   对于这个用例，枚举对象是最好的选择。

    +   只要原始值是字符串或数字，我们也可以使用枚举。

+   具有唯一值的自定义类型：

    +   我喜欢为这种用途使用带有符号的枚举对象。然而，它们稍微有点冗长。

    +   另一个轻量级的替代方案是字符串字面量联合。

    +   枚举也是一个选项，但应该是字符串枚举，因为它们更安全，并且记录它们的值（例如，用于调试）效果更好。

+   带有对象值的常量命名空间：

    +   如果你想进行详尽的检查，请使用枚举对象。

    +   如果你希望枚举值具有方法，请使用枚举类。

    +   我们不能为这个用例使用枚举。

各种考虑因素：

+   如本章开头所述，使用类型剥离和 `erasableSyntaxOnly`，我们无法使用枚举。此外，枚举被转换成的代码有点难以阅读——特别是如果枚举有数字成员的话。

+   使用字符串字面量联合，无法为成员提供 JSDoc 注释——这可以通过 `@deprecated` 实现弃用。

+   如果枚举值是字符串，比较可能会更慢。然而，这只有在你的代码进行许多比较时才重要。

    +   （我听说有一个 TypeScript 项目，从字符串切换到其他类型显著提高了性能。遗憾的是，我再也找不到那个链接了——欢迎提供提示！）

+   带有字符串、符号和对象的枚举比带有数字的枚举更容易处理——因为当你记录枚举值时，你会看到名称。
