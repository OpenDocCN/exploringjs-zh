# 18 对象类型化

> 原文：[`exploringjs.com/ts/book/ch_typing-objects.html`](https://exploringjs.com/ts/book/ch_typing-objects.html)

(广告，请勿屏蔽。)

1.  18.1 对象类型

    1.  18.1.1 使用对象的方式

    1.  18.1.2 对象类型在 TypeScript 中以结构化方式工作

1.  18.2 对象字面量类型的成员

    1.  18.2.1 方法签名

    1.  18.2.2 对象类型成员的键

    1.  18.2.3 对象类型成员的修饰符

1.  18.3 额外的属性检查：何时允许额外的属性？

    1.  18.3.1 为什么对象字面量中禁止额外的属性？

    1.  18.3.2 如果物体来自其他地方，为什么允许额外的属性？

    1.  18.3.3 空对象字面量类型允许额外的属性

    1.  18.3.4 仅匹配没有属性的物体

    1.  18.3.5 在对象字面量中允许额外的属性

1.  18.4 对象类型和继承属性

    1.  18.4.1 TypeScript 不区分自有属性和继承属性

    1.  18.4.2 对象字面量类型描述`Object`的实例

1.  18.5 接口与对象字面量类型

    1.  18.5.1 对象字面量类型可以内联

    1.  18.5.2 同名接口合并

    1.  18.5.3 映射类型看起来像对象字面量类型

    1.  18.5.4 只有接口支持多态的`this`类型

    1.  18.5.5 只有接口支持`extends` – 但类型交集（`&`）类似

1.  18.6 通过`never`禁止属性

    1.  18.6.1 禁止带字符串键的属性

    1.  18.6.2 禁止索引属性（带数字键）

1.  18.7 索引签名：对象作为字典

    1.  18.7.1 类型化索引签名键

    1.  18.7.2 字符串键与数字键

    1.  18.7.3 索引签名与属性签名和方法签名

1.  18.8 `Record<K, V>`用于字典对象

    1.  18.8.1 索引签名不允许键联合

    1.  18.8.2 `Record` 强制键联合的完备性

    1.  18.8.3 `Record`：防止键联合的完备性检查

1.  18.9 `object` vs `Object` vs. `{}`

    1.  18.9.1 纯 JavaScript：对象与 `Object` 实例

    1.  18.9.2 TypeScript 中的 `Object`（大写“O”）：`Object` 类的实例

    1.  18.9.3 类型 `{}` 基本上意味着：非空值

    1.  18.9.4 各种对象的推断类型

1.  18.10 总结：`object` vs `Object` vs. `{}` vs. `Record`

1.  18.11 本章的来源

在本章中，我们将探讨在 TypeScript 中如何静态地对对象和属性进行类型化。

### 18.1 对象类型

#### 18.1.1 使用对象的方式

在 JavaScript 中，使用对象有两种方式：

+   固定布局对象：以这种方式使用，对象就像数据库中的记录。它具有固定数量的属性，其键在开发时已知。它们的值通常具有不同的类型。

    ```ts
    const fixedLayoutObject: FixedLayoutObjectType = {
     product: 'carrot',
     quantity: 4,
    };

    ```

+   字典对象：以这种方式使用，对象就像一个查找表或映射。它具有可变数量的属性，其键在开发时未知。所有这些值都具有相同的类型。

    ```ts
    const dictionaryObject: DictionaryObjectType = {
     ['one']: 1,
     ['two']: 2,
    };

    ```

注意，这两种方式也可以混合使用：一些对象既是固定布局对象也是字典对象。

这两种对象类型最常见的类型化方式是：

```ts
type FixedLayoutObjectType = {
 product: string,
 quantity: number,
};
type DictionaryObjectType = Record<string, number>;

```

+   `FixedLayoutObjectType` 是一个对象字面量类型。属性之间的分隔符可以是逗号（`,`）或分号（`;`）。我更喜欢前者，因为这是 JavaScript 对象字面量所使用的。

+   `DictionaryObjectType` 使用实用类型 `Record` 定义字典对象的类型，其键为字符串，值为数字。

在回到字典对象类型之前，我们将更详细地查看固定布局对象类型。

#### 18.1.2 对象类型在 TypeScript 中的结构工作方式

在 TypeScript 中，对象类型以结构方式工作：它们匹配具有其结构的所有值。因此，可以在给定值之后定义一个类型，并且仍然匹配它 - 例如：

```ts
const myPoint = {x: 1, y: 2};

function logPoint(point: {x: number, y: number}): void {
 console.log(point);
}

logPoint(myPoint); // Works!

```

有关此主题的更多信息，请参阅“命名类型系统与结构类型系统”（§13.4）。

### 18.2 对象字面量类型的成员

对象字面量类型体内的结构称为它们的 *成员*。这些是最常见的成员：

```ts
type ExampleObjectType = {
 // Property signature
 myProperty: boolean,

 // Method signature
 myMethod(str: string): number,

 // Index signature
 [key: string]: any,

 // Call signature
 (num: number): string,

 // Construct signature
 new(str: string): ExampleInstanceType, 
};

type ExampleInstanceType = {};

```

让我们更详细地看看这些成员：

+   属性签名定义属性，应该是自解释的：

    ```ts
    myProperty: boolean;

    ```

+   方法签名定义方法，将在下一小节中描述。

    ```ts
    myMethod(str: string): number;

    ```

    注意：参数的名称（在这种情况下：`str`）有助于说明事物的工作方式，但没有其他目的。

+   索引签名用于描述用作字典的数组或对象。它们将在本章后面描述。

    ```ts
    [key: string]: any;

    ```

    注意：名称 `key` 仅用于文档目的。

+   调用签名使对象字面量类型能够描述函数。参见“具有调用签名的接口”（§27.2.2）。

    ```ts
    (num: number): string;

    ```

+   构造签名使对象字面量类型能够描述类和构造函数。参见“具有构造签名的对象类型字面量”（§23.2.3）。

    ```ts
    new(str: string): ExampleInstanceType; 

    ```

#### 18.2.1 方法签名

就 TypeScript 的类型系统而言，方法定义和值是函数的属性是等效的：

```ts
type HasMethodDef = {
 simpleMethod(flag: boolean): void,
};
type HasFuncProp = {
 simpleMethod: (flag: boolean) => void,
};
type _ = Assert<Equal<
 HasMethodDef,
 HasFuncProp
>>;

const objWithMethod = {
 simpleMethod(flag: boolean): void {},
};
assertType<HasMethodDef>(objWithMethod);
assertType<HasFuncProp>(objWithMethod);

const objWithOrdinaryFunction: HasMethodDef = {
 simpleMethod: function (flag: boolean): void {},
};
assertType<HasMethodDef>(objWithOrdinaryFunction);
assertType<HasFuncProp>(objWithOrdinaryFunction);

const objWithArrowFunction: HasMethodDef = {
 simpleMethod: (flag: boolean): void => {},
};
assertType<HasMethodDef>(objWithArrowFunction);
assertType<HasFuncProp>(objWithArrowFunction);

```

我的建议是使用最能表达属性设置方式的语法。

#### 18.2.2 对象类型成员的键

##### 18.2.2.1 引号键

就像在 JavaScript 中一样，属性键可以是引号的：

```ts
type Obj = { 'hello everyone!': string };

```

##### 18.2.2.2 未引用的数字作为键

在实践中，这很少很重要，但作为旁白：就像在 JavaScript 中一样，我们可以使用未引用的数字作为键。与 JavaScript 不同，这些键被认为是数字字面量类型：

```ts
type _ = Assert<Equal<
 keyof {0: 'a', 1: 'b'},
 0 | 1
>>;

```

为了比较，这是 JavaScript 的工作方式：

```ts
assert.deepEqual(
 Object.keys({0: 'a', 1: 'b'}),
 [ '0', '1' ]
);

```

更多信息请参阅 $type。

##### 18.2.2.3 计算属性键

[计算属性键](https://exploringjs.com/js/book/ch_objects.html#object-literals-computed-keys)是 JavaScript 的一个特性。在类型级别上也有类似的功能：

```ts
type ExampleObjectType = {
 // Property signature with computed key
 [Symbol.toStringTag]: string,

 // Method signature with computed key
 [Symbol.iterator](): IteratorObject<string>,
};

```

意想不到的是，计算属性键是值，而不是类型。TypeScript 内部应用 `typeof` 来创建类型：

```ts
type _ = Assert<Equal<
 { ['hello']: string },
 { hello: string }
>>;

```

计算属性键允许什么样的值？它的类型必须是：

+   一个字符串字面量类型，例如 `'abc'`

+   一个数字字面量类型，例如 `123`

+   一个独特的符号类型

+   `any`

#### 18.2.3 对象类型成员的修饰符

##### 18.2.3.1 可选属性

如果我们在属性名称后加上问号 (`?`)，则该属性是可选的。相同的语法也用于标记函数、方法和构造函数的参数为可选。在以下示例中，属性 `.middle` 是可选的：

```ts
type Name = {
 first: string;
 middle?: string;
 last: string;
};

```

因此，省略该属性（行 A）是可以的：

```ts
const john: Name = {first: 'Doe', last: 'Doe'}; // (A)
const jane: Name = {first: 'Jane', middle: 'Cecily', last: 'Doe'};

```

###### 18.2.3.1.1 可选与 `undefined | string` 使用 `exactOptionalPropertyTypes`

在这本书中，所有代码都使用编译器设置 `exactOptionalPropertyTypes`。使用此设置，可选属性与类型为 `undefined | string` 的属性之间的区别是直观的：

```ts
type Obj = {
 prop1?: string;
 prop2: undefined | string; 
};

const obj1: Obj = { prop1: 'a', prop2: 'b' };

// .prop1 can be omitted; .prop2 can be `undefined`
const obj2: Obj = { prop2: undefined };

// @ts-expect-error: Type '{ prop1: undefined; prop2: string; }' is not
// assignable to type 'Obj' with 'exactOptionalPropertyTypes: true'.
// Consider adding 'undefined' to the types of the target's properties.
// Types of property 'prop1' are incompatible. Type 'undefined' is not
// assignable to type 'string'.
const obj3: Obj = { prop1: undefined, prop2: 'b' };

// @ts-expect-error: Property 'prop2' is missing in type '{ prop1: string;
// }' but required in type 'Obj'.
const obj4: Obj = { prop1: 'a' };

```

当我们想要明确省略时，类型如 `undefined | string` 和 `null | string` 是有用的。当人们看到这样的明确省略的属性时，他们会知道它存在，但已被关闭。

###### 18.2.3.1.2 可选与 `undefined | string` 无 `exactOptionalPropertyTypes`

如果 `exactOptionalPropertyTypes` 为 `false`，则有一件事会改变：`.prop1` 也可以是 `undefined`：

```ts
type Obj = {
 prop1?: string;
 prop2: undefined | string; 
};

const obj1: Obj = { prop1: undefined, prop2: undefined };

```

##### 18.2.3.2 只读属性

在以下示例中，属性 `.prop` 是只读的：

```ts
type MyObj = {
 readonly prop: number;
};

```

作为结果，我们可以读取它，但不能更改它：

```ts
const obj: MyObj = {
 prop: 1,
};

console.log(obj.prop); // OK

// @ts-expect-error: Cannot assign to 'prop' because it is a read-only
// property.
obj.prop = 2;

```

### 18.3 多余的属性检查：何时允许额外的属性？

例如，考虑以下对象字面量类型：

```ts
type Point = {
 x: number,
 y: number,
};

```

有两种方式（以及其他方式）可以解释这种对象字面量类型：

+   严格解释：它可以描述所有具有**恰好**指定类型的 `.x` 和 `.y` 属性的对象。换句话说：这些对象不得有**多余的属性**（超过所需属性）。

+   宽泛解释：它可以描述所有至少具有 `.x` 和 `.y` 属性的对象。换句话说：允许有多余的属性。

TypeScript 使用两种解释。为了探索它是如何工作的，我们将使用以下函数：

```ts
function computeDistance(point: Point) { /*...*/ }

```

默认情况下，允许有多余的属性 `.z`：

```ts
const obj = { x: 1, y: 2, z: 3 };
computeDistance(obj); // OK

```

然而，如果我们直接使用对象字面量，则不允许有多余的属性：

```ts
// @ts-expect-error: Object literal may only specify known properties, and
// 'z' does not exist in type 'Point'.
computeDistance({ x: 1, y: 2, z: 3 }); // error

computeDistance({x: 1, y: 2}); // OK

```

#### 18.3.1 为什么对象字面量中禁止多余的属性？

为什么对象字面量有更严格的规则？它们可以提供对属性键中错误的保护。我们将使用以下对象字面量类型来演示这意味着什么。

```ts
type Person = {
 first: string,
 middle?: string,
 last: string,
};
function computeFullName(person: Person) { /*...*/ }

```

属性 `.middle` 是可选的，可以省略。对于 TypeScript 来说，名称的拼写错误看起来就像省略了它并提供了一个多余的属性。然而，它仍然会捕捉到这个错误，因为在这种情况下不允许有多余的属性：

```ts
// @ts-expect-error: Object literal may only specify known properties, but
// 'mdidle' does not exist in type 'Person'. Did you mean to write
// 'middle'?
computeFullName({first: 'Jane', mdidle: 'Cecily', last: 'Doe'});

```

#### 18.3.2 为什么如果对象来自其他地方，则允许多余的属性？

理念是，如果一个对象来自其他地方，我们可以假设它已经被审查过，并且不会有任何错误。然后我们可以不那么小心。

如果拼写错误不是问题，我们的目标应该是最大化灵活性。考虑以下函数：

```ts
type HasYear = {
 year: number,
};

function getAge(obj: HasYear) {
 const yearNow = new Date().getFullYear();
 return yearNow - obj.year;
}

```

如果不允许传递给 `getAge()` 的值有多余属性，这个函数的实用性将非常有限。

#### 18.3.3 空对象字面量类型允许多余的属性

如果对象字面量类型为空，则始终允许有多余的属性：

```ts
type Empty = {};
type OneProp = {
 myProp: number,
};

// @ts-expect-error: Object literal may only specify known properties, and
// 'anotherProp' does not exist in type 'OneProp'.
const a: OneProp = { myProp: 1, anotherProp: 2 };
const b: Empty = { myProp: 1, anotherProp: 2 }; // OK

```

#### 18.3.4 仅匹配没有属性的对象

如果我们想强制一个对象没有任何属性，可以使用以下技巧（来源：[Geoff Goodman](https://twitter.com/filearts/status/1222502898552180737)）：

```ts
type WithoutProperties = {
 [key: string]: never,
};

// @ts-expect-error: Type 'number' is not assignable to type 'never'.
const a: WithoutProperties = { prop: 1 };
const b: WithoutProperties = {}; // OK

```

#### 18.3.5 对象字面量中允许多余的属性

如果我们想在对象字面量中允许多余的属性，可以考虑以下示例，类型 `Point` 和函数 `computeDistance1()`：

```ts
type Point = {
 x: number,
 y: number,
};

function computeDistance1(point: Point) { /*...*/ }

// @ts-expect-error: Object literal may only specify known properties, and
// 'z' does not exist in type 'Point'.
computeDistance1({ x: 1, y: 2, z: 3 });

```

一种选择是将对象字面量赋值给一个中间变量：

```ts
const obj = { x: 1, y: 2, z: 3 };
computeDistance1(obj);

```

第二种选择是使用类型断言：

```ts
computeDistance1({ x: 1, y: 2, z: 3 } as Point); // OK

```

第三种选择是将 `computeDistance1()` 重写为使用类型参数：

```ts
function computeDistance2<P extends Point>(point: P) { /*...*/ }
computeDistance2({ x: 1, y: 2, z: 3 }); // OK

```

第四种选择是扩展 `Point` 以允许多余的属性：

```ts
type PointEtc = Point & {
 [key: string]: any;
};
function computeDistance3(point: PointEtc) { /*...*/ }

computeDistance3({ x: 1, y: 2, z: 3 }); // OK

```

我们使用交集类型（`&` 操作符）来定义 `PointEtc`。更多信息，请参阅“对象类型的交集”（§20.1）。

我们将继续讨论两个示例，其中 TypeScript 不允许多余的属性是一个问题。

##### 18.3.5.1 允许多余的属性：示例 `Incrementor` 工厂

在这个例子中，我们实现了一个类型为 `Incrementor` 的对象工厂，并希望返回一个子类型，但 TypeScript 不允许额外的属性 `.counter`：

```ts
type Incrementor = {
 inc(): number,
};
function createIncrementor(): Incrementor {
 return {
 // @ts-expect-error: Object literal may only specify known properties, and
 // 'counter' does not exist in type 'Incrementor'.
 counter: 0,
 inc() {
 // @ts-expect-error: Property 'counter' does not exist on type
 // 'Incrementor'.
 return this.counter++;
 },
 };
}

```

可惜，即使使用类型断言，仍然存在一个类型错误：

```ts
function createIncrementor2(): Incrementor {
 return {
 counter: 0,
 inc() {
 // @ts-expect-error: Property 'counter' does not exist on type
 // 'Incrementor'.
 return this.counter++;
 },
 } as Incrementor;
}

```

使用 `as any` 可以解决问题，但返回的对象类型将是 `any`，例如在 `.inc()` 中，TypeScript 不会检查 `this` 的属性是否真的存在。

一个合适的解决方案是为 `Incrementor` 添加索引签名。或者——特别是如果不可能的话——引入一个中间变量：

```ts
function createIncrementor3(): Incrementor {
 const incrementor = {
 counter: 0,
 inc() {
 return this.counter++;
 },
 };
 return incrementor;
}

```

##### 18.3.5.2 允许多余的属性：示例 `.dateStr`

以下比较函数可以用于对具有属性 `.dateStr` 的对象进行排序：

```ts
function compareDateStrings(
 a: {dateStr: string}, b: {dateStr: string}) {
 if (a.dateStr < b.dateStr) {
 return +1;
 } else if (a.dateStr > b.dateStr) {
 return -1;
 } else {
 return 0;
 }
 }

```

例如，在单元测试中，我们可能希望直接使用对象字面量调用此函数。TypeScript 不允许这样做，我们需要使用一种解决方案。

### 18.4 对象类型和继承属性

#### 18.4.1 TypeScript 不区分自有属性和继承属性

TypeScript 不区分自有属性和继承属性。它们都被简单地视为属性。

```ts
type MyType = {
 toString(): string, // inherited property
 prop: number, // own property
};
const obj: MyType = { // OK
 prop: 123,
};

```

`obj` 从 `Object.prototype` 继承了 `.toString()` 方法。

这种方法的缺点是，JavaScript 中的一些现象无法通过 TypeScript 的类型系统来描述。优点是类型系统更简单。

#### 18.4.2 对象字面量类型描述 `Object` 的实例

所有对象字面量类型都描述了 `Object` 的实例，并继承了 `Object.prototype` 的属性。在以下示例中，类型为 `{}` 的参数 `x` 与返回类型 `Object` 兼容：

```ts
function f1(x: {}): Object {
 return x;
}

```

类似地，`{}` 有一个 `.toString()` 方法：

```ts
function f2(x: {}): { toString(): string } {
 return x;
}

```

### 18.5 接口与对象字面量类型

由于历史原因，对象类型可以以两种方式定义：

```ts
// Object literal type
type ObjType1 = {
 a: boolean,
 b: number,
 c: string,
};

// Interface
interface ObjType2 {
 a: boolean;
 b: number;
 c: string;
}

```

+   在这两种情况下，都可以使用分号或逗号作为分隔符。我更喜欢逗号用于对象字面量类型，分号用于接口，因为这反映了 JavaScript 的样子（对象字面量和类）。

+   允许并可选地使用尾随分隔符。

现在定义对象类型的方式在某种程度上是等价的。我们将深入了解（轻微的）差异。

#### 18.5.1 对象字面量类型可以被内联

对象字面量类型可以被内联，而接口不能：

```ts
// The object literal type is inlined
// (mentioned inside the parameter definition)
function f1(x: {prop: number}) {}

// We can’t mention the interface inside the parameter definition.
// We can only define it externally and refer to it.
function f2(x: ObjectInterface) {} 
interface ObjectInterface {
 prop: number;
}

```

#### 18.5.2 同名接口将被合并

具有重复名称的类型别名是非法的：

```ts
// @ts-expect-error: Duplicate identifier 'PersonAlias'.
type PersonAlias = {first: string};
// @ts-expect-error: Duplicate identifier 'PersonAlias'.
type PersonAlias = {last: string};

```

相反，具有重复名称的接口将被合并：

```ts
interface PersonInterface {
 first: string;
}
interface PersonInterface {
 last: string;
}
const jane: PersonInterface = {
 first: 'Jane',
 last: 'Doe',
};

```

这被称为[声明合并](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)，可以用来组合来自多个源的类型 - 例如，只要`Array.fromAsync()`是一个新方法，它就不是核心库声明文件的一部分，而是通过`lib.esnext.array.d.ts`提供的 - 它将其作为增量添加到`ArrayConstructor`（`Array`作为类值的类型）：

```ts
interface ArrayConstructor {
 fromAsync<T>(···): Promise<T[]>;
}

```

#### 18.5.3 映射类型看起来像对象字面量类型

映射类型（行 A）看起来像对象字面量类型：

```ts
type Point = {
 x: number,
 y: number,
};

type PointCopy1 = {
 [Key in keyof Point]: Point[Key] // (A)
};

```

作为选项，我们可以用分号结束行 A。然而，逗号是不允许的。

更多关于这个主题的信息，请参阅“映射类型`{[K in U]: X}`”（§36）[“Mapped types `{[K in U]: X}`” (§36)](ch_mapped-types.html#ch_mapped-types)。

#### 18.5.4 只有接口支持多态`this`类型

多态`this`类型只能用于接口：

```ts
interface AddsStrings {
 add(str: string): this;
};

class StringBuilder implements AddsStrings {
 result = '';
 add(str: string): this {
 this.result += str;
 return this;
 }
}

```

#### 18.5.5 只有接口支持`extends` - 但类型交集（`&`）类似

接口`B`可以扩展另一个接口`A`，然后被解释为`A`的增量：

```ts
interface A {
 propA: number;
}
interface B extends A {
 propB: number;
}
type _ = Assert<Equal<
 B,
 {
 propA: number,
 propB: number,
 }
>>;

```

对象字面量类型不支持`extend`，但交集类型`&`有类似的效果：

```ts
type A = {
 propA: number,
};
type B = {
 propB: number,
} & A;
type _ = Assert<Equal<
 B,
 {
 propA: number,
 propB: number,
 }
>>;

```

对象类型的交集在另一章中描述得更详细 another chapter。在这里，我们将探讨它们如何与`extends`精确地不同。

##### 18.5.5.1 冲突

如果扩展接口和扩展接口之间存在冲突，那么这就是一个错误：

```ts
interface A {
 prop: string;
}
// @ts-expect-error: Interface 'B' incorrectly extends interface 'A'.
// Types of property 'prop' are incompatible.
interface B extends A {
 prop: number;
}

```

相比之下，交集类型不会对冲突提出异议，但它们可能在某些位置导致`never`：

```ts
type A = {
 prop: string,
};
type B = {
 prop: number,
} & A;
type _ = Assert<Equal<
 B,
 {
 prop: number & string, // never
 }
>>;

```

##### 18.5.5.2 只有接口支持覆盖

覆盖一个方法意味着用一个*兼容*的方法替换超类中的方法 - 大概：

+   覆盖方法可以返回更具体的价值 - 例如，期望`Object`的覆盖方法调用者不会介意覆盖方法返回一个`RegExp`。

+   重写方法可以期望更不具体的参数 – 例如，重写方法的调用者传递一个类型为`string`的参数时，如果重写方法接受`string | number`，则不会介意。

```ts
interface A {
 m(x: string): Object;
}
interface B extends A {
 m(x: string | number): RegExp;
}

type _ = Assert<Equal<
 B,
 {
 m(x: string | number): RegExp,
 }
>>;

function f(x: B) {
 assertType<RegExp>(x.m('abc'));
}

```

我们可以看到，重写方法“获胜”并完全替换了`B`中的被重写方法。相比之下，交集类型中同时存在这两种方法：

```ts
type A = {
 m(x: string): Object,
};
type B = {
 m(x: string | number): RegExp,
};

type _ = [
 Assert<Equal<
 A & B,
 {
 m: ((x: string) => Object) & ((x: string | number) => RegExp),
 }
 >>,
 Assert<Equal<
 B & A,
 {
 m: ((x: string | number) => RegExp) & ((x: string) => Object),
 }
 >>,
];

function f1(x: A & B) {
 assertType<Object>(x.m('abc')); // (A)
}
function f2(x: B & A) {
 assertType<RegExp>(x.m('abc')); // (B)
}

```

当涉及到返回类型（行 A 和行 B）时，交集中的较早成员获胜。这就是为什么`B & A`（`B1`）更类似于`B extends A`，尽管`A & B`（`B2`）看起来更漂亮：

```ts
type B1 = {
 prop: number,
} & A;
type B2 = A & {
 prop: number,
};

```

##### 18.5.5.3　`extends`或`&` – 哪个类型使用？

哪个类型取决于上下文。如果涉及继承，则接口和`extends`通常是更好的选择，因为它们支持重写。

![图标“外部”](img/c1f1e62c75044317fd354db81eaefed3.png) **本节来源**

+   [GitHub 问题“TypeScript：类型与接口”](https://github.com/peerigon/eslint-config-peerigon/issues/64) 由 [Johannes Ewald](https://github.com/jhnns) 提出

### 18.6　通过`never`禁止属性

由于没有其他类型可以赋值给`never`，我们可以用它来禁止属性。

#### 18.6.1　禁止带有字符串键的属性

类型`EmptyObject`禁止字符串键：

```ts
type EmptyObject = Record<string, never>;

// @ts-expect-error: Type 'number' is not assignable to type 'never'.
const obj1: EmptyObject = { prop: 123 };
const obj2: EmptyObject = {}; // OK

```

相比之下，类型`{}`可以从所有对象赋值，并且不是空对象类型：

```ts
const obj3: {} = { prop: 123 };

```

#### 18.6.2　禁止索引属性（带数字键）

类型`NoIndices`禁止数字键但允许字符串键`'prop'`：

```ts
type NoIndices = Record<number, never> & { prop?: boolean };

//===== Objects =====
const obj1: NoIndices = {}; // OK
const obj2: NoIndices = { prop: true }; // OK
// @ts-expect-error: Type 'string' is not assignable to type 'never'.
const obj3: NoIndices = { 0: 'a' }; // OK

//===== Arrays =====
const arr1: NoIndices = []; // OK
// @ts-expect-error: Type 'string' is not assignable to type 'never'.
const arr2: NoIndices = ['a'];

```

### 18.7　索引签名：对象作为字典

到目前为止，我们只使用了固定布局对象的类型。我们如何表达一个对象将被用作字典的事实？例如：在以下代码片段中`TranslationDict`应该是什么？

```ts
function translate(dict: TranslationDict, english: string): string {
 const translation = dict[english];
 if (translation === undefined) {
 throw new Error();
 }
 return translation;
}

```

一种选择是使用索引签名（行 A）来表示`TranslationDict`是为将字符串键映射到字符串值的对象而设计的（另一种选择是`Record` – 我们将在稍后讨论）：

```ts
type TranslationDict = {
 [key: string]: string, // (A)
};
const dict = {
 'yes': 'sí',
 'no': 'no',
 'maybe': 'tal vez',
};
assert.equal(
 translate(dict, 'maybe'),
 'tal vez');

```

名称`key`不重要 – 它可以是任何标识符，并且被忽略（但不能省略）。

#### 18.7.1　类型索引签名键

索引签名表示一个无限集合的属性；只允许以下类型：

+   `string`

+   `number`

+   `symbol`

+   模板字符串字面量与无限原语类型 – 例如：`` `${bigint}` ``

+   之前任何类型的联合

特别不允许的是：

+   单个字符串字面量类型 – 例如：`'a'`、`1`、`false`

+   字符串字面量类型的联合 – 例如：

    +   `'a' | 'b'`

    +   `1 | 2`

    +   `boolean`（它是`false | true`）

+   模板字符串字面量与之前类型之一 – 例如：`` `${boolean}` ``

+   类型：`never`、`any`、`unknown`

如果需要更多功能，请考虑使用映射类型。

这些是索引签名的示例：

```ts
type IndexSignature1 = {
 [key: string]: boolean,
};
// Template string literal with infinite primitive type
type IndexSignature2 = {
 [key: `${bigint}`]: string,
};
// Union of previous types
type IndexSignature3 = {
 [key: string | `${bigint}`]: string,
};

```

#### 18.7.2 字符串键与数字键

就像在纯 JavaScript 中一样，TypeScript 的数字属性键是字符串属性键的子集 ([参见“探索 JavaScript”](https://exploringjs.com/js/book/ch_arrays.html#array-indices))。因此，如果我们同时有一个字符串索引签名和一个数字索引签名，前者的属性类型必须是后者的超类型。以下示例之所以有效，是因为 `Object` 是 `RegExp` 的超类型（`RegExp` 可以赋值给 `Object`）：

```ts
type StringAndNumberKeys = {
 [key: string]: Object,
 [key: number]: RegExp,
};

```

以下代码演示了使用字符串和数字作为属性键的效果：

```ts
function f(x: StringAndNumberKeys) {
 return {
 str: x['abc'],
 num: x[123],
 };
}
assertType<
 (x: StringAndNumberKeys) => {
 str: Object | undefined,
 num: RegExp | undefined,
 }
>(f);

```

#### 18.7.3 索引签名与属性签名和方法签名

如果一个对象字面量类型中既有索引签名又有属性和/或方法签名，那么索引属性值的类型也必须是属性值和/或方法类型的超类型。

```ts
type T1 = {
 [key: string]: boolean,

 // @ts-expect-error: Property 'myProp' of type 'number' is not assignable
 // to 'string' index type 'boolean'.
 myProp: number,

 // @ts-expect-error: Property 'myMethod' of type '() => string' is not
 // assignable to 'string' index type 'boolean'.
 myMethod(): string,
};

```

相比之下，以下两个对象字面量类型不会产生错误：

```ts
type T2 = {
 [key: string]: number,
 myProp: number,
};

type T3 = {
 [key: string]: () => string,
 myMethod(): string,
}

```

### 18.8 `Record<K, V>` 用于字典对象

内置的泛型实用工具类型 `Record<K, V>` 是用于键类型为 `K` 且值类型为 `V` 的字典对象：

```ts
const dict: Record<string, number> = {
 one: 1,
 two: 2,
 three: 3,
};

```

如果你好奇 `Record` 是如何定义的：“`Record` 是一个映射类型” (§36.6)。这种知识可以帮助你记住它如何处理有限和无限键类型。

`Record` 支持字面类型联合作为键类型；索引签名不支持。更多内容将在下文中介绍。

#### 18.8.1 索引签名不允许键联合

索引签名的键类型必须是无限的：

```ts
type Key = 'A' | 'B' | 'C';

// @ts-expect-error: An index signature parameter type cannot be a literal
// type or generic type. Consider using a mapped object type instead.
const dict: {[key: Key]: true} = {
 A: true,
 C: true,
};

```

#### 18.8.2 `Record` 强制执行键联合的详尽性

当 `Record` 的键类型是字面类型联合时，它强制执行详尽性：

```ts
type T = 'A' | 'B' | 'C';

// @ts-expect-error: Property 'C' is missing in type '{ A: true; B: true; }'
// but required in type 'Record<T, true>'.
const nonExhaustiveKeys: Record<T, true> = {
 A: true,
 B: true,
};

const exhaustiveKeys: Record<T, true> = {
 A: true,
 B: true,
 C: true,
};

```

错误的键也会产生错误：

```ts
const wrongKey: Record<T, true> = {
 A: true,
 B: true,
 // @ts-expect-error: Object literal may only specify known properties,
 // and 'D' does not exist in type 'Record<T, true>'.
 D: true,
};

```

#### 18.8.3 `Record`：防止对键联合进行详尽性检查

如果我们想防止对类型为联合的键进行详尽性检查，则可以使用实用工具类型 `Partial`（它使所有属性都成为可选的）。然后我们可以省略一些属性，但错误的键仍然会产生错误：

```ts
type T = 'A' | 'B' | 'C';
const nonExhaustiveKeys: Partial<Record<T, true>> = {
 A: true,
};
const wrongKey: Partial<Record<T, true>> = {
 // @ts-expect-error: Object literal may only specify known properties,
 // and 'D' does not exist in type 'Partial<Record<T, true>>'.
 D: true,
};

```

### 18.9 `object` 与 `Object` 与 `{}`

这些是三种类似的对象通用类型：

+   小写“o”的 `object` 是所有非原始值的类型。它与 JavaScript 操作符 `typeof` 返回的值 `'object'` 稍有相关。

    ```ts
    const obj1: object = {};
    const obj2: object = [];
    // @ts-expect-error: Type 'number' is not assignable to type 'object'.
    const obj3: object = 123;

    ```

+   大写“O”的 `Object` 是类 `Object` 实例的类型：

    ```ts
    const obj1: Object = new Object();

    ```

    但它也接受原始值（除了 `undefined` 和 `null`）：

    ```ts
    const obj2: Object = 123;

    ```

    注意，非空原始值通过其包装类型继承 `Object.prototype` 的方法。

+   `{}` 接受所有非空值。它与 `Object` 的唯一区别在于它不介意属性与 `Object.prototype` 属性冲突：

    ```ts
    const obj1: {} = { toString: true }; // OK
    const obj2: Object = {
     // @ts-expect-error: Type 'boolean' is not assignable to
     // type '() => string'.
     toString: true,
    };

    ```

    因此，类型 `{}` 基本上意味着：“值不能为 null”。

![图标“提示”](img/0873709827ba4924e4afbb757e47a4df.png) **这些类型并不常用**

由于我们使用这些类型时无法访问任何属性，因此它们并不常用。如果一个值确实有那种类型，我们通常在对其进行任何操作之前通过 类型守卫 来缩小其类型。

#### 18.9.1 纯 JavaScript：对象与 `Object` 类的实例

在纯 JavaScript 中，有一个重要的区别。

一方面，大多数对象都是 `Object` 的实例。

```ts
> const obj1 = {};
> obj1 instanceof Object
true

```

这意味着：

+   `Object.prototype` 在它们的原型链中（这就是 `instanceof` 检查的内容）：

    ```ts
    > Object.prototype.isPrototypeOf(obj1)
    true

    ```

+   它们继承其属性。

    ```ts
    > obj1.toString === Object.prototype.toString
    true

    ```

另一方面，我们也可以创建没有 `Object.prototype` 在其原型链中的对象。例如，以下对象没有任何原型：

```ts
> const obj2 = Object.create(null);
> Object.getPrototypeOf(obj2)
null

```

`obj2` 是一个不是 `Object` 类实例的对象：

```ts
> typeof obj2
'object'
> obj2 instanceof Object
false

```

#### 18.9.2 TypeScript 中的 `Object`（大写“O”）：`Object` 类的实例

回想一下，每个类 `C` 创建两个实体：

+   一个构造函数 `C`。

+   一个描述构造函数实例的对象类型 `C`。

同样，对于 `Object` 类也有两种对象类型：

+   类型 `Object` 指定了 `Object` 类实例的属性，包括从 `Object.prototype` 继承的属性。

+   类型 `ObjectConstructor` 指定了 `Object` 类（具有属性的对象）的属性。

这些是类型：

```ts
interface Object { // (A)
 constructor: Function;
 toString(): string;
 toLocaleString(): string;
 /** Returns the primitive value of the specified object. */
 valueOf(): Object; // (B)
 hasOwnProperty(v: PropertyKey): boolean;
 isPrototypeOf(v: Object): boolean;
 propertyIsEnumerable(v: PropertyKey): boolean;
}

interface ObjectConstructor {
 /** Invocation via `new` */
 new(value?: any): Object;
 /** Invocation via function calls */
 (value?: any): any;

 readonly prototype: Object; // (C)

 getPrototypeOf(o: any): any;
 // ···
}
declare var Object: ObjectConstructor; // (D)

```

观察：

+   我们有一个名为 `Object` 的变量（行 D）和一个名为 `Object` 的类型（行 A）。

+   `Object.prototype` 也有类型 `Object`（行 C）。鉴于任何 `Object` 的实例都继承其所有属性，这是有道理的。

+   很有趣的是，在行 B 中，`.valueOf()` 的返回类型是 `Object`，并且应该返回原始值。

#### 18.9.3 类型 `{}` 基本上意味着：非空值

`{}` 接受所有除了 `undefined` 和 `null` 以外的值：

```ts
const v1: {} = 123;
const v2: {} = 123;
const v3: {} = {};
const v4: {} = { prop: true };

// @ts-expect-error: Type 'undefined' is not assignable to type '{}'.
const v5: {} = undefined;
// @ts-expect-error: Type 'null' is not assignable to type '{}'.
const v6: {} = null;

```

辅助类型 `NonNullable` 使用 `{}`:

```ts
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T & {};

type _ = [
 Assert<Equal<
 NonNullable<undefined | string>,
 string
 >>,
 Assert<Equal<
 NonNullable<null | string>,
 string
 >>,
 Assert<Equal<
 NonNullable<string>,
 string
 >>,
];

```

`NonNullable<T>` 的结果是 `T` 和所有非空值交集的类型。

#### 18.9.4 各种对象的推断类型

这些是 TypeScript 为通过各种方式创建的对象推断的类型：

```ts
const obj1 = new Object();
assertType<Object>(obj1);

const obj2 = Object.create(null);
assertType<any>(obj2);

const obj3 = {};
assertType<{}>(obj3);

const obj4 = {prop: 123};
assertType<{prop: number}>(obj4);

const obj5 = Reflect.getPrototypeOf({});
assertType<object | null>(obj5);

```

原则上，`Object.create()` 的返回类型可以是 `object` 或计算出的类型。然而，出于历史原因，它是 `any`。这允许我们添加和更改结果的属性。

### 18.10 总结：`object` vs `Object` vs. `{}` vs. `Record`

下表比较了对象的四种类型：

|  | `object` | `Object` | `{}` | `Record` |
| --- | --- | --- | --- | --- |
| 接受 `undefined` 或 `null` | ✘ | ✘ | ✘ | ✘ |
| 接受原始值 | ✘ | ✔ | ✔ | ✘ |
| 有 `.toString()` | ✔ | ✔ | ✔ | N/A |
| 值可能与 `Object` 冲突 | ✔ | ✘ | ✔ | N/A |

最后两行表格对于 `Record` 来说实际上没有太多意义——这就是为什么它的单元格中有一个“N/A”。

**接受 `undefined` 或 `null`：**

```ts
type _ = [
 Assert<Not<Assignable<
 object, undefined
 >>>,
 Assert<Not<Assignable<
 Object, undefined
 >>>,
 Assert<Not<Assignable<
 {}, undefined
 >>>,
 Assert<Not<Assignable<
 Record<keyof any, any>, undefined
 >>>,
];

```

**接受原始值：**

```ts
type _ = [
 Assert<Not<Assignable<
 object, 123
 >>>,
 Assert<Assignable<
 Object, 123
 >>,
 Assert<Assignable<
 {}, 123
 >>,
 Assert<Not<Assignable<
 Record<keyof any, any>, 123
 >>>,
];

```

**具有 `.toString()` 方法：**

```ts
type _ = [
 Assert<Assignable<
 { toString(): string }, object
 >>,
 Assert<Assignable<
 { toString(): string }, Object
 >>,
 Assert<Assignable<
 { toString(): string }, {}
 >>,
];

```

**值可能与 `Object` 冲突：**

```ts
type _ = [
 Assert<Assignable<
 object, { toString(): number }
 >>,
 Assert<Not<Assignable<
 Object, { toString(): number }
 >>>,
 Assert<Assignable<
 {}, { toString(): number }
 >>,
];

```

### 18.11 本章来源

+   [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/basic-types.html)
