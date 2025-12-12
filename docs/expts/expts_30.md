# 25   只读可访问性 (readonly 等)

> 原文：[`exploringjs.com/ts/book/ch_readonly.html`](https://exploringjs.com/ts/book/ch_readonly.html)

(请勿阻止，广告。)

1.  25.1   `const` 变量声明：只有绑定是不可变的

1.  25.2   只读对象属性

    1.  25.2.1   初始化后无变化

    1.  25.2.2   `readonly` 不影响可赋值性

    1.  25.2.3   只读索引签名

    1.  25.2.4   工具类型 `Readonly<T>`

1.  25.3   类属性

1.  25.4   只读数组

1.  25.5   只读元组

1.  25.6   `ReadonlySet` 和 `ReadonlyMap`

1.  25.7   Const 断言 (`as const`)

1.  25.8   使用建议

    1.  25.8.1   如果你想要接受只读元组，你需要 `ReadonlyArray`

    1.  25.8.2   使用类型 `ReadonlyArray` 的缺点

1.  25.9   进一步阅读

    1.  25.9.1   本章来源

在本章中，我们探讨如何在 TypeScript 中使事物变为“只读”状态——主要通过关键字 `readonly`。

### 25.1   `const` 变量声明：只有绑定是不可变的

在 JavaScript 中，如果一个变量通过 `const` 声明，绑定变为不可变，但绑定的值不是：

```ts
const obj = {prop: 'yes'};

// We can’t assign a different value:
assert.throws(
 () => obj = {},
 /^TypeError: Assignment to constant variable./
);

// But we can modify the assigned value:
obj.prop = 'no'; // OK

```

TypeScript 的只读可访问性与之类似。然而，它仅在编译时进行检查；它不会以任何方式影响输出的 JavaScript。

### 25.2   只读对象属性

我们可以使用关键字 `readonly` 使对象属性不可变：

```ts
type ReadonlyProp = {
 readonly prop: { str: string },
};
const obj: ReadonlyProp = {
 prop: { str: 'a' },
};

```

将属性变为不可变具有以下后果：

```ts
// The property is immutable:
// @ts-expect-error: Cannot assign to 'prop' because it is
// a read-only property.
obj.prop = { str: 'x' };

// But not the property value:
obj.prop.str += 'b';

```

#### 25.2.1   初始化后无变化

如果一个属性 `.count` 是只读的，我们可以通过对象字面量来初始化它，但不能之后更改它。如果我们想更改其值，我们必须创建一个新的对象（行 A）：

```ts
type Counter = {
 readonly count: number,
};

function createCounter(): Counter {
 return { count: 0 };
}
function toIncremented(counter: Counter): Counter {
 return { // (A)
 count: counter.count + 1,
 };
}

```

`toIncremented()` 的这个变异版本会产生编译时错误：

```ts
function increment(counter: Counter): void {
 // @ts-expect-error: Cannot assign to 'count' because it is
 // a read-only property.
 counter.count++;
}

```

#### 25.2.2   `readonly` 不影响可赋值性

比较令人惊讶的是，`readonly` 不影响可赋值性：

```ts
type Obj = { prop: number };
type ReadonlyObj = { readonly prop: number };

function func(_obj: Obj) { }
function readonlyFunc(_readonlyObj: ReadonlyObj) { }

const obj: Obj = { prop: 123 };
func(obj);
readonlyFunc(obj);

const readonlyObj: ReadonlyObj = { prop: 123 };
func(readonlyObj);
readonlyFunc(readonlyObj);

```

然而，我们可以通过类型相等性来检测它：

```ts
type _ = Assert<Not<Equal<
 { readonly prop: number },
 { prop: number }
>>>;

```

已经有一个针对编译器选项 `--enforceReadonly` 的 [pull request](https://github.com/microsoft/TypeScript/pull/58296)，这将改变此行为。

#### 25.2.3   只读索引签名

除了属性外，索引签名也可以通过 `readonly` 进行修改。以下是一个描述类似数组的内置类型：

```ts
interface ArrayLike<T> {
 readonly length: number;
 readonly [n: number]: T; // (A)
}

```

行 A 是一个只读索引签名。`ArrayLike`使用的一个例子：`Array.from()`参数的类型。

```ts
interface ArrayConstructor {
 from<T>(iterable: Iterable<T> | ArrayLike<T>): T[];
 // ···
}

```

如果索引签名是只读的，我们就不能使用索引访问来更改值：

```ts
const arrayLike: ArrayLike<string> = {
 length: 2,
 0: 'a',
 1: 'b',
};
assert.deepEqual(
 Array.from(arrayLike), ['a', 'b']
);
assert.equal(
 // Reading is allowed:
 arrayLike[0], 'a'
);
// Writing is not allowed:
// @ts-expect-error: Index signature in type 'ArrayLike<string>'
// only permits reading.
arrayLike[0] = 'x';

```

#### 25.2.4 实用类型`Readonly<T>`

实用类型`Readonly<T>`使`T`的所有属性变为只读：

```ts
type Point = {
 x: number,
 y: number,
 dist(): number,
};
type ReadonlyPoint = Readonly<Point>;

type _ = Assert<Equal<
 ReadonlyPoint,
 {
 readonly x: number,
 readonly y: number,
 readonly dist: () => number,
 }
>>;

```

### 25.3 类属性

类也可以有只读属性。这些属性必须直接或在构造函数中初始化，之后不能更改。这就是为什么可变增加`.incMut()`不起作用的原因：

```ts
class Counter {
 readonly count: number;
 constructor(count: number) {
 this.count = count;
 }
 inc(): Counter {
 return new Counter(this.count + 1);
 }
 incMut(): void {
 // @ts-expect-error: Cannot assign to 'count' because
 // it is a read-only property.
 this.count++;
 }
}

```

### 25.4 只读数组

有两种方式可以将字符串数组等声明为只读：

```ts
ReadonlyArray<string>
readonly string[]

```

`ReadonlyArray`仅是一个类型：与`Array`不同，它在运行时不存在。这就是我们可以使用此类型的方式：

```ts
const arr: ReadonlyArray<string> = ['a', 'b'];

// @ts-expect-error: Index signature in type 'readonly string[]'
// only permits reading.
arr[0] = 'x';

// @ts-expect-error: Cannot assign to 'length' because it is
// a read-only property.
arr.length = 1;

// @ts-expect-error: Property 'push' does not exist on
// type 'readonly string[]'.
arr.push('x');

```

我们创建一个普通的数组，并给它类型`ReadonlyArray<string>`。这就是在类型级别使数组变为只读的方法。

在最后一行，我们可以看到类型`ReadonlyArray`不仅使属性和索引签名`readonly`，它还缺少可变方法：

```ts
interface ReadonlyArray<T> {
 readonly length: number;
 readonly [n: number]: T;

 // Included: non-destructive methods such as .map(), .filter(), etc.
 // Excluded: destructive methods such as .push(), .sort(), etc.
 // ···
}

```

如果我们想在运行时创建只读的数组，我们可以使用以下方法：

```ts
class ImmutableArray<T> {
 #arr: Array<T>;
 constructor(arr: Array<T>) {
 this.#arr = arr;
 }
 get length(): number {
 return this.#arr.length;
 }
 at(index: number): T | undefined {
 return this.#arr.at(index);
 }
 map<U>(
 callbackfn: (value: T, index: number, array: readonly T[]) => U,
 thisArg?: any
 ): U[] {
 return this.#arr.map(callbackfn, thisArg);
 }
 // (Many omitted methods)
}

```

我们没有实现`ReadonlyArray<T>`，因为我们不通过方括号提供索引访问，只通过`.at()`。前者可以通过[代理](https://exploringjs.com/deep-js/ch_proxies.html)实现，但这会导致代码不够优雅且性能较差。

### 25.5 只读元组

在普通元组中，我们可以为元素分配不同的值，但不能改变长度：

```ts
const tuple: [string, number] = ['a', 1];
tuple[0] = 'x'; // OK

tuple.length = 2; // OK
// @ts-expect-error: Type '1' is not assignable to type '2'.
tuple.length = 1;
// The type of `.length` is 2 (not `number`)
type _ = Assert<Equal<
 (typeof tuple)['length'], 2
>>;

// Interestingly, `.push()` is allowed:
tuple.push('x'); // OK

```

如果元组是只读的，我们就不能为元素或`.length`分配不同的值：

```ts
const tuple: readonly [string, number] = ['a', 1];

// @ts-expect-error: Cannot assign to '0' because it is
// a read-only property.
tuple[0] = 'x';

// @ts-expect-error: Cannot assign to 'length' because it is
// a read-only property.
tuple.length = 2;

// @ts-expect-error: Property 'push' does not exist on
// type 'readonly [string, number]'.
tuple.push('x');

```

我们设置只读元组一次（在开始时）并且之后不能更改。只读元组的类型是`ReadonlyArray`的子类型：

```ts
type _ = Assert<Extends<
 typeof tuple, ReadonlyArray<string | number>
>>;

```

### 25.6 `ReadonlySet`和`ReadonlyMap`

与类型`ReadonlyArray`是`Array`的只读版本类似，也存在`Set`和`Map`的只读版本：

```ts
// Not included here: methods defined in lib.es2015.iterable.d.ts
// such as: .keys() and .[Symbol.iterator]()

interface ReadonlySet<T> {
 forEach(
 callbackfn: (value: T, value2: T, set: ReadonlySet<T>) => void,
 thisArg?: any
 ): void;
 has(value: T): boolean;
 readonly size: number;
}
interface ReadonlyMap<K, V> {
 forEach(
 callbackfn: (value: V, key: K, map: ReadonlyMap<K, V>) => void,
 thisArg?: any
 ): void;
 get(key: K): V | undefined;
 has(key: K): boolean;
 readonly size: number;
}

```

这是我们如何使用`ReadonlySet`的方式：

```ts
const COLORS: ReadonlySet<string> = new Set(['red', 'green']);

```

这是一个包装类，它使集合在运行时变为只读：

```ts
class ImmutableSet<T> implements ReadonlySet<T> {
 #set: Set<T>;
 constructor(set: Set<T>) {
 this.#set = set;
 }
 get size(): number {
 return this.#set.size;
 }
 forEach(
 callbackfn: (value: T, value2: T, set: ReadonlySet<T>) => void,
 thisArg?: any
 ): void {
 return this.#set.forEach(callbackfn, thisArg);
 }
 // Etc.
}

```

### 25.7 常量断言（`as const`）

`const`断言`as const`是对值的注释，它仅影响它们的类型。它可以应用于：

+   枚举成员的引用

+   布尔字面量

+   字符串字面量

+   对象字面量

+   数组字面量

它有两个效果：

+   非原始类型的值变为只读：

    +   对象：所有属性变为只读

    +   数组：变为只读元组

+   推断的类型变得更窄——例如`'abc'`（不是`string`）

这就是对象受到影响的方式——注意`readonly`和更窄的类型（`123`与`number`相比）：

```ts
const obj = { prop: 123 };
type _1 = Assert<Equal<
 typeof obj, { prop: number }
>>;

const constObj = { prop: 123 } as const;
type _2 = Assert<Equal<
 typeof constObj, { readonly prop: 123 }
>>;

```

这就是数组受到影响的方式——注意`readonly`和更窄的类型（`'a'`和`'b'`与`string`相比）：

```ts
const arr = ['a', 'b'];
type _1 = Assert<Equal<
 typeof arr, string[]
>>;

const constTuple = ['a', 'b'] as const;
type _2 = Assert<Equal<
 typeof constTuple, readonly ["a", "b"]
>>;

```

由于原始值已经是不可变的，`as const`只会导致推断出更窄的类型：

```ts
let str1 = 'abc';
type _1 = Assert<Equal<
 typeof str1, string
>>;

let str2 = 'abc' as const;
type _2 = Assert<Equal<
 typeof str2, 'abc'
>>;

```

从`let`到`const`的转换也会缩小类型：

```ts
const str3 = 'abc';
type _3 = Assert<Equal<
 typeof str3, 'abc'
>>;

```

### 25.8 使用建议

#### 25.8.1 如果你想要接受只读元组，则需要 `ReadonlyArray`

尽管 `readonly` 不会影响可赋值性，但只读元组是 `ReadonlyArray` 的子类型，因此与 `Array` 不兼容，因为后者类型具有前者没有的方法。让我们看看这对函数和泛型类型意味着什么。

##### 25.8.1.1 函数

以下函数 `sum()` 不能应用于只读元组：

```ts
function sum(numbers: Array<number>): number {
 return numbers.reduce((acc, x) => acc + x, 0);
}

sum([1, 2, 3]); // OK

const readonlyTuple = [1, 2, 3] as const;
// @ts-expect-error: Argument of type 'readonly [1, 2, 3]'
// is not assignable to parameter of type 'number[]'.
sum(readonlyTuple);

```

```ts
function sum(numbers: ReadonlyArray<number>): number {
 return numbers.reduce((acc, x) => acc + x, 0);
}

const readonlyTuple = [1, 2, 3] as const;
sum(readonlyTuple); // OK

```

##### 25.8.1.2 泛型类型

如果类型 `T` 被限制为普通数组类型，那么它就不匹配 `as const` 文字的类型：

```ts
type Wrap<T extends Array<unknown>> = Promise<T>;
const arr = ['a', 'b'] as const;
// @ts-expect-error: Type 'readonly ["a", "b"]' does not satisfy
// the constraint 'unknown[]'.
type _ = Wrap<typeof arr>;

```

我们可以通过切换到 `ReadonlyArray` 来改变这一点：

```ts
type Wrap<T extends ReadonlyArray<unknown>> = Promise<T>;
const arr = ['a', 'b'] as const;
type Result = Wrap<typeof arr>;
type _ = Assert<Equal<
 Result, Promise<readonly ["a", "b"]>
>>;

```

#### 25.8.2 使用类型 `ReadonlyArray` 的缺点

使用类型 `ReadonlyArray` 有一个缺点：你不能将数据传递到不使用该类型的位置（其中有很多）——例如：

```ts
function appFunc(arr: ReadonlyArray<string>): void {
 // @ts-expect-error: Argument of type 'readonly string[]'
 // is not assignable to parameter of type 'string[]'.
 libFunc(arr);
}

function libFunc(arr: Array<string>): void {}

```

### 25.9 进一步阅读

+   “探索 JavaScript”中的章节 [“保护对象不被更改”](https://exploringjs.com/js/book/ch_objects.html#protecting-objects)

#### 25.9.1 本章的来源

本章节的以下部分来自官方 TypeScript 手册：

+   [`readonly` 和 `const`](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#readonly-and-const)

+   [实用类型 > `Readonly<Type>`](https://www.typescriptlang.org/docs/handbook/utility-types.html#readonlytype)

+   [类成员 > 字段 > `readonly`](https://www.typescriptlang.org/docs/handbook/2/classes.html#readonly)

+   [对象类型 > 属性修饰符 > `readonly` 属性](https://www.typescriptlang.org/docs/handbook/2/objects.html#readonly-properties)

+   [`ReadonlyArray` 类型](https://www.typescriptlang.org/docs/handbook/2/objects.html#the-readonlyarray-type)

+   [`readonly` 元组类型](https://www.typescriptlang.org/docs/handbook/2/objects.html#readonly-tuple-types)
