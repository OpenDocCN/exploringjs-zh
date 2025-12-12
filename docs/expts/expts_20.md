# 16 TypeScript 中的符号

> 原文：[`exploringjs.com/ts/book/ch_symbols.html`](https://exploringjs.com/ts/book/ch_symbols.html)

(广告，请勿屏蔽。)

1.  16.1 符号的类型

    1.  16.1.1 `symbol` 和 `typeof MY_SYMBOL`

    1.  16.1.2 类型 `unique symbol`

1.  16.2 符号类型的联合

1.  16.3 符号作为特殊值

1.  16.4 符号作为枚举值

1.  16.5 进一步阅读

在本章中，我们探讨 TypeScript 在类型级别上如何处理 JavaScript 符号。

如果你想刷新你对 JavaScript 符号的了解，可以查看“Exploring JavaScript”中的第 [“Symbols”](https://exploringjs.com/js/book/ch_symbols.html) 章。

### 16.1 符号的类型

#### 16.1.1 `symbol` 和 `typeof MY_SYMBOL`

类型推断通常给我们：

+   更宽、更一般的 `let` 类型

+   更窄、更具体的 `const` 类型

例如：

```ts
let value1 = 123;
assertType<number>(value1);

const value2 = 123;
assertType<123>(value2);

```

这也适用于符号：

```ts
let SYM1 = Symbol('SYM1');
assertType<symbol>(SYM1);

const SYM2 = Symbol('SYM2');
assertType<typeof SYM2>(SYM2);

```

`symbol` 是所有符号的类型。`typeof SYM2` 是一个特定符号的类型。我们无法创建另一个与 `typeof SYM2` 匹配的值：

```ts
function f(_sym: typeof SYM2) {}

f(SYM2); // OK
// @ts-expect-error: Argument of type 'symbol' is not assignable to
// parameter of type 'unique symbol'.
f(Symbol('SYM2')); // new, different value!

```

注意错误信息中的类型 `unique symbol`。我们很快就会了解到它是什么。

我们无法创建一个与原始 `SYM2` 相等的新的符号，这是 JavaScript 的一个现象：

```ts
> Symbol() === Symbol()
false

```

在这方面，符号与对象字面量类似：

```ts
> {} === {}
false

```

##### 16.1.1.1 陷阱：赋值扩展符号的类型

如果我们将具有 `typeof SYM` 类型的变量 `SYM` 赋值给另一个变量 `X`，那么后者的类型将扩展到 `symbol` – 即使我们在声明时使用了 `const`。

```ts
const SYM = Symbol('SYM'); // typeof SYM

function getSym(): typeof SYM {
 const X = SYM; // symbol

 // @ts-expect-error: Type 'symbol' is not assignable to
 // type 'unique symbol'.
 return X;
}

```

相关 GitHub 问题：[“unique symbol” 在赋值给 `const` 时丢失，尽管有类型断言](https://github.com/microsoft/TypeScript/issues/55901)

#### 16.1.2 类型 `unique symbol`

类型 `unique symbol` 是 `symbol` 的子类型。这意味着这个特定位置持有具有特定类型的符号。它与 `typeof SOME_SYMBOL` 非常相似，但不命名特定的符号。每个 `unique symbol` 的位置都是不同的，并且与其他所有位置不兼容（类似于 `typeof SOME_SYMBOL`）。

`unique symbol` 可以用于 `const` 变量声明和 `static readonly` 属性。如果我们想在其他地方表达唯一性，我们必须使用 `typeof S` – 就像我们之前做的那样。鉴于 `unique symbol` 实际上是以另一种方式表达 `typeof S`，它并不非常有用

```ts
const SYM: unique symbol = Symbol('SYM');
class MyClass {
 static readonly SYM: unique symbol = Symbol('SYM');
}

```

之前的代码完全等同于：

```ts
const SYM = Symbol('SYM');
class MyClass {
 static readonly SYM = Symbol('SYM');
}

```

我们还可以在另一个位置使用 `unique symbol` – 作为只读属性的类型。这样做是为了声明诸如 `Symbol.iterator`（文件 `lib.es2015.iterable.d.ts`）之类的已知符号的类型：

```ts
interface SymbolConstructor {
 readonly iterator: unique symbol;
}

```

为什么接口的名称是 `SymbolConstructor`？这是因为符号在文件 `lib.es2015.symbol.d.ts` 中的设置方式：

```ts
interface SymbolConstructor {
 readonly prototype: Symbol;
 (description?: string | number): symbol;
 for(key: string): symbol;
 keyFor(sym: symbol): string | undefined;
}

declare var Symbol: SymbolConstructor;

```

##### 16.1.2.1 我们不能创建类型为 `unique symbol` 的属性

考虑以下类型：

```ts
type Obj = {
 readonly sym: unique symbol,
};

```

我们不能创建一个可以分配给 `Obj` 的对象：

```ts
const obj1: Obj = {
 // @ts-expect-error: Type 'symbol' is not assignable to
 // type 'unique symbol'.
 sym: Symbol('SYM'),
};

const SYM: unique symbol = Symbol('SYM');
const obj2: Obj = {
 // @ts-expect-error: Type 'typeof SYM' is not assignable to
 // type 'typeof sym'.
 sym: SYM,
};

```

在前面的子节 `SymbolConstructor.iterator` 并不是为尚未创建的对象设计的。它是为已经存在的单个全局值设计的。

### 16.2 符号类型的联合

我们可以使用符号来定义联合类型：

```ts
const ACTIVE = Symbol('ACTIVE');
const INACTIVE = Symbol('INACTIVE');
type ActSym = typeof ACTIVE | typeof INACTIVE;

const activation1: ActSym = ACTIVE;
const activation2: ActSym = INACTIVE;
// @ts-expect-error: Type 'unique symbol' is not assignable to
// type 'ActSym'.
const activation3: ActSym = Symbol('ACTIVE');

```

我们必须使用 `typeof` 从程序级别到类型级别：

+   程序级别：`ACTIVE`

+   类型级别：`typeof ACTIVE`

基于符号的联合类型与以下字符串联合类型如何比较？

```ts
type ActStr = 'ACTIVE' | 'INACTIVE';

```

为了更容易比较 `ActSym` 和 `ActStr`，让我们以更复杂的方式定义后者（我们通常不会这样做）：

```ts
const ACTIVE = 'ACTIVE';
const INACTIVE = 'INACTIVE';
type ActStr = typeof ACTIVE | typeof INACTIVE;

const activation1: ActStr = ACTIVE;
const activation2: ActStr = INACTIVE;
const activation3: ActStr = 'ACTIVE'; // OK!

```

基于字符串的联合类型的优缺点是什么？

+   优点：无需导入常量，我们可以简单地提及字符串（见上面最后一行）。

+   缺点：联合值不是类型安全的。字符串是通过值（而不是身份）进行比较的，这就是为什么一个值可能会被错误地认为是 `ActStr` 的成员，而实际上它不是。这种错误不会在基于符号的类型联合中发生。

### 16.3 符号作为特殊值

`undefined` 和 `null` 经常被用作与类型实际值不同的特殊“非值”：

```ts
type StreamValue =
 | null // end of file
 | string
;

```

然而，符号可以是一个更好、更易于理解的替代方案：

```ts
const EOF = Symbol('EOF');
type StreamValue =
 | typeof EOF
 | string
;

```

关于这个主题的更多信息，请参阅“Adding special values to types” (§17)。

### 16.4 符号作为枚举值

如果我们想要创建具有新、唯一值的枚举，符号也工作得很好：

```ts
const Active = Symbol('Active');
const Inactive = Symbol('Inactive');

const Activation = {
 __proto__: null,
 Active, // (A)
 Inactive, // (B)
} as const;

type ActivationType = PropertyValues<typeof Activation>;
type _ = Assert<Equal<
 ActivationType, typeof Active | typeof Inactive
>>;

type PropertyValues<Obj> = Obj[Exclude<keyof Obj, '__proto__'>];

```

为什么在第一行和第二行中声明变量 `Active` 和 `Inactive` 是一个中间步骤？为什么不在行 A 和行 B 中创建符号？

如果我们这样做的话：

+   `Activation.Active` 将具有 `symbol` 类型，而不是 `typeof Active` 类型。

+   `Activation.Inactive` 将具有 `symbol` 类型，而不是 `typeof Inactive` 类型。

因此，`ActivationType` 将是 `symbol`。对于更长的解释，请参阅“Symbols as property values” (§26.4.5)。

### 16.5 进一步阅读

+   TypeScript：在 TypeScript 手册中的[“Symbols”](https://www.typescriptlang.org/docs/handbook/symbols.html)章节

+   JavaScript：在“Exploring JavaScript”中的[“Symbols”](https://exploringjs.com/js/book/ch_symbols.html)章节
