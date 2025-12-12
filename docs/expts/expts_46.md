# 38 模板字面量类型 (`${U}`)：创建和转换字符串字面量类型

> 原文：[`exploringjs.com/ts/book/ch_template-literal-types.html`](https://exploringjs.com/ts/book/ch_template-literal-types.html)

(请勿屏蔽广告。)

1.  38.1 本章结构

1.  38.2 模板字面量类型的语法和基本使用方法

    1.  38.2.1 语法

    1.  38.2.2 连接操作是分配的

    1.  38.2.3 通过 `infer` 提取子字符串

    1.  38.2.4 通过 `infer` 和 `extends` 解析子字符串

    1.  38.2.5 将原始类型插入到模板字面量中

1.  38.3 字符串操作实用类型

    1.  38.3.1 示例：`isUppercase`

    1.  38.3.2 示例：`ToString`

    1.  38.3.3 示例：修剪字符串字面量类型

1.  38.4 使用元组

    1.  38.4.1 连接字符串元组

    1.  38.4.2 拆分字符串

    1.  38.4.3 示例：通过字符串字面量类型定义字符串字面量联合类型

    1.  38.4.4 将字符串拆分为代码单元

1.  38.5 使用对象

    1.  38.5.1 示例：向属性名添加前缀

1.  38.6 实际示例

    1.  38.6.1 示例：将输出样式化到终端

    1.  38.6.2 示例：属性路径

    1.  38.6.3 示例：更改属性名前缀

    1.  38.6.4 示例：将驼峰式转换为连字符式

    1.  38.6.5 示例：将连字符式转换为驼峰式

1.  38.7 使用模板字面量类型做一些有趣的事情

    1.  38.7.1 Node.js：UUID 类型

    1.  38.7.2 解析 CLI 参数（Stefan Baumgartner）

    1.  38.7.3 `document.querySelector()` 的智能结果类型（Mike Ryan）

    1.  38.7.4 在 Angular 中设置路由（Mike Ryan）

    1.  38.7.5 表达路由提取器（Dan Vanderkam）

    1.  38.7.6 Tailwind 颜色变化（Tomek Sułkowski）

    1.  38.7.7 Arktype：定义类型

1.  38.8 结论和注意事项

1.  38.9 进一步阅读

1.  38.10 本章来源

在本章中，我们将更深入地了解 TypeScript 中的模板字面量类型：虽然它们的语法类似于 JavaScript 的模板字面量，但它们在类型级别上操作。它们的使用案例包括：

+   字符串字面量的静态语法检查

+   转换属性名称的大小写（例如，从连字符大小写到驼峰大小写）

+   简洁地指定大型字符串字面量联合类型

### 38.1 本章的结构

首先，我们将通过小型玩具示例学习模板字面量类型的工作原理。我们将编写类似于 JavaScript 代码的类型级别代码。以下是我们将涵盖的主题：

+   模板字面量类型的语法和基本用法

+   字符串操作实用类型

+   与元组一起工作

+   与对象一起工作

之后，我们将看看人们使用模板字面量类型所做的实际例子和巧妙的事情。

### 38.2 模板字面量类型的语法和基本用法

#### 38.2.1 语法

模板字面量类型的语法受到 JavaScript 模板字面量的启发。它们还允许我们连接值，可选地穿插静态字符串片段：

```ts
type Song<Num extends number, Bev extends string> = // (A)
 `${Num} bottles of ${Bev}`
;
type _ = Assert<Equal<
 Song<99, 'juice'>, // (B)
 '99 bottles of juice'
>>;

```

说明：

+   在行（A）中，我们定义了泛型类型`Song`。它类似于 JavaScript 中的函数，但在类型级别上操作。使用`extends`关键字来指定`Num`和`Bev`必须可以赋值的类型。

+   在行（B）中，我们将`Song`应用于两个参数：一个数字字面量类型和一个字符串字面量类型。

#### 38.2.2 连接是分配的

如果我们将字符串字面量联合类型插入到模板字面量类型中，后者将应用于前者中的每个成员：

```ts
type Modules = 'fs' | 'os' | 'path';
type Prefixed = `node:${Modules}`;
type _ = Assert<Equal<
 Prefixed, 'node:fs' | 'node:os' | 'node:path'
>>;

```

如果我们插入多个字符串字面量联合类型，那么我们将得到笛卡尔积（使用所有可能的组合）：

```ts
type Words = `${ 'd' | 'l' }${ 'i' | 'o' }ve`;
type _ = Assert<Equal<
 Words, 'dive' | 'dove' | 'live' | 'love'
>>;

```

这使我们能够简洁地指定大型联合。我们稍后会看到一个实际例子。

[Anders Hejlsberg 警告](https://github.com/microsoft/TypeScript/pull/40336): “小心联合类型的交叉积分布会迅速升级为非常大且昂贵的类型。还请注意，联合类型限制在少于 100,000 个组成部分，以下将导致错误：”

```ts
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
// @ts-expect-error: Expression produces a union type
// that is too complex to represent.
type Zip = `${Digit}${Digit}${Digit}${Digit}${Digit}`;

```

上面的联合类型恰好有 100,000 个元素。这是另一个超过限制的例子：

```ts
type Hex =
 | '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
 | 'A' | 'B' | 'C' | 'D' | 'E' | 'F'
 ;
// @ts-expect-error: Expression produces a union type
// that is too complex to represent.
type CssColor = `#${Hex}${Hex}${Hex}${Hex}${Hex}${Hex}`;

```

#### 38.2.3 通过`infer`提取子字符串

如果我们在模板字面量内部使用`infer`运算符，我们可以提取字符串的一部分：

```ts
type ParseSemver<Str extends string> =
 Str extends `${infer Major}.${infer Minor}.${infer Patch}`
 ? [ Major, Minor, Patch ]
 : never
;
type _ = Assert<Equal<
 ParseSemver<'1.2.3'>, ['1', '2', '3']
>>;

```

#### 38.2.4 通过`infer`加`extends`解析子字符串

默认情况下，模板字面量内的`infer`提取一个字符串：

```ts
type _ = Assert<Equal<
 '¡Hola!' extends `¡${infer S}!` ? S : never,
 'Hola'
>>;

```

如果我们通过`extends`将`infer`约束到类型，TypeScript 将解析该类型的值——例如：

```ts
type _ = [
 Assert<Equal<
 'true' extends `${infer B extends boolean}` ? B : never,
 true
 >>,
 Assert<Equal<
 '256' extends `${infer N extends number}` ? N : never,
 256
 >>,
];

```

仅对整数进行解析数字：

```ts
type StrToNum<T> =
 T extends `${infer N extends number}` ? N : never
;
type _ = [
 Assert<Equal<
 StrToNum<'123'>, 123
 >>,
 Assert<Equal<
 StrToNum<'-123'>, -123
 >>,
 Assert<Equal<
 StrToNum<'1.0'>, number
 >>,
 Assert<Equal<
 StrToNum<'1e2'>, number
 >>,
 Assert<Equal<
 StrToNum<'abc'>, never
 >>,
];

```

有关使用`StrToNum`的示例，请参阅“提取元组的索引（数字）”（§37.3.2）（ch_computing-with-tuple-types.html#TupleIndices）。

#### 38.2.5 将原始类型插值到模板字面量中

到目前为止，我们已经将字面量类型和联合类型插入到模板字面量中，但我们也可以插入一些原始类型。这使我们能够构造 `string` 的子集：

```ts
type Version = `v${number}.${number}`;

// @ts-expect-error: Type '""' is not assignable to
// type '`v${number}.${number}`'.
const version0: Version = '';

const version1: Version = 'v1.0'; // OK

// @ts-expect-error: Type '"v2.zero"' is not assignable to
// type '`v${number}.${number}`'.
const version2: Version = 'v2.zero';

```

这些是受支持的类型：

```ts
const undefinedValue: `${undefined}` = 'undefined';
const nullValue: `${null}` = 'null';
const booleanValue: `${boolean}` = 'true';
const numberValue: `${number}` = '123';
const bigintValue: `${bigint}` = '123';
const stringValue: `${string}` = 'abc';

// @ts-expect-error: Type 'symbol' is not assignable to type
// 'string | number | bigint | boolean | null | undefined'.
const symbolValue: `${symbol}` = 'symbol';

```

注意，`undefined` 是具有单个值 `undefined` 的类型。`null` 类似。

##### 38.2.5.1 示例: 仅保留包含数字的字符串

可以使用插值类型 `number` 来“过滤”字符串字面量类型的联合：

```ts
type Properties = '0' | '2' | 'length' | 'toString';
type _ = Assert<Equal<
 Properties & `${number}`, // (A)
 '0' | '2'
>>;

```

我们在行 A 中使用交集类型来提取 `Properties` 中那些被转换为字符串的元素。使用此技巧的示例：

+   “移除 `Pick<T, K>` 参数 `K` 的约束”（§36.3.3.2）

+   “提取元组的索引键（字符串）”（§37.3.1）

##### 38.2.5.2 示例: 排除以“a”开头的字符串

模板字面量类型 `` `a${string}` `` 匹配所有以字符“a”开头的字符串字面量类型。因此，我们可以用它来排除这些类型：

```ts
type _ = Assert<Equal<
 Exclude<'apple' | 'apricot' | 'banana', `a${string}`>,
 'banana'
>>;

```

### 38.3 字符串操作实用类型

TypeScript 有四个内置的字符串操作类型（[文档](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#intrinsic-string-manipulation-types)）：

+   `Uppercase<StringLiteralType>`

    ```ts
    type _ = Assert<Equal<
     Uppercase<'hello'>, 'HELLO'
    >>;

    ```

+   `Lowercase<StringLiteralType>`

    ```ts
    type _ = Assert<Equal<
     Lowercase<'HELLO'>, 'hello'
    >>;

    ```

+   `Capitalize<StringLiteralType>`

    ```ts
    type _ = Assert<Equal<
     Capitalize<'hello'>, 'Hello'
    >>;

    ```

+   `Uncapitalize<StringLiteralType>`

    ```ts
    type _ = Assert<Equal<
     Uncapitalize<'HELLO'>, 'hELLO'
    >>;

    ```

TypeScript（截至版本 5.7）使用以下 JavaScript 方法来执行更改——这些方法不是区域感知的（[来源](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#intrinsic-string-manipulation-types) – 请参阅末尾的“技术细节”）：

```ts
str.toUpperCase() // Uppercase
str.toLowerCase() // Lowercase
str.charAt(0).toUpperCase() + str.slice(1) // Capitalize
str.charAt(0).toLowerCase() + str.slice(1) // Uncapitalize

```

#### 38.3.1 示例: `isUppercase`

我们可以使用 `Uppercase` 来定义一个泛型类型 `IsUppercase`：

```ts
type IsUppercase<Str extends string> = Str extends Uppercase<Str>
 ? true
 : false;

type _ = [
 Assert<Equal<
 IsUppercase<'SUNSHINE'>, true
 >>,
 Assert<Equal<
 IsUppercase<'SUNSHINe'>, false
 >>,
];

```

#### 38.3.2 示例: `ToString`

`ToString` 使用正常的模板字面量类型插值将原始字面量类型转换为字符串字面量类型：

```ts
type ToString<
 T extends string | number | bigint | boolean | null | undefined
> = `${T}`
;

type _ = Assert<Equal<
 ToString<'abc' | 123 | -456n | false | null | undefined>,
 'abc' | '123' | '-456' | 'false' | 'null' | 'undefined'
>>;

```

#### 38.3.3 示例: 去除字符串字面量类型的空格

要去除字符串字面量类型的空格，我们递归地从其开始和结束处移除所有空格：

```ts
type TrimStart<Str extends string> =
 Str extends ` ${infer Rest}`
 ? TrimStart<Rest> // (A)
 : Str
type TrimEnd<Str extends string> =
 Str extends `${infer Rest} `
 ? TrimEnd<Rest> // (B)
 : Str
;
type Trim<Str extends string> = TrimStart<TrimEnd<Str>>;

```

```ts
type _ = [
 Assert<Equal<
 TrimStart<'  text  '>,
 'text  '
 >>,
 Assert<Equal<
 TrimEnd<'  text  '>,
 '  text'
 >>,
 Assert<Equal<
 Trim<'  text  '>,
 'text'
 >>,
];

```

注意行 A 和行 B 中的自递归调用。

致谢：此示例受到了 [Mike Ryan 的代码](https://x.com/MikeRyanDev/status/1308472279010025477)的启发。

### 38.4 处理元组

#### 38.4.1 连接字符串元组

```ts
type Join<Strs extends string[], Sep extends string = ','> =
 Strs extends [
 infer First extends string,
 ...infer Rest extends string[]
 ]
 ? Rest['length'] extends 0
 ? First
 : `${First}${Sep}${Join<Rest, Sep>}`
 : ''
;
type _ = Assert<Equal<
 Join<['Hello', 'How', 'Are', 'You'], ' '>,
 'Hello How Are You'
>>;

```

再次强调，我们正在以函数式风格在类型级别上进行编程。

在第二行，我们将元组 `Strs` 分解（考虑解构）：

+   `First` 是第一个元素（一个字符串）。

+   `Rest` 是剩余的元素（一个元组）。

下一步：

+   如果元组 `Rest` 为空，则结果简单地为 `First`。

+   否则，我们将 `First` 与分隔符 `Sep` 和连接的 `Rest` 连接起来。

#### 38.4.2 分割字符串

```ts
type Split<Str extends string, Sep extends string> =
 Str extends `${infer First}${Sep}${infer Rest}`
 ? [First, ...Split<Rest, Sep>] // (A)
 : [Str] // (B)
;

```

```ts
type _ = Assert<Equal<
 Split<'How | are|you', '|'>,
 ['How ', ' are', 'you']
>>;

```

我们在模板字面量中使用 `infer` 来确定分隔符 `Sep` 前的 `First` 前缀，然后递归调用 `Split`（行 A）。

如果没有找到分隔符，则结果是一个包含整个 `Str` 的元组（行 B）。

#### 38.4.3 示例：通过字符串字面量类型定义字符串字面量联合类型

我们可以使用之前定义的 `Split` 和 `Trim` 将字符串字面量类型转换为字符串字面量联合类型：

```ts
type StringToUnion<Str extends string> =
 Trim<TupleToUnion<Split<Str, '|'>>>;
type TupleToUnion<Tup extends readonly unknown[]> = Tup[number]; 

type _ = Assert<Equal<
 StringToUnion<'A | B | C'>,
 'A' | 'C' | 'B'
>>;

```

`TupleToUnion` 将元组 `Tup` 视为一个从数字（索引）到值的映射：`Tup[number]` 表示“给我所有值”（`Tup` 的范围）。

注意 `Trim` 的应用是如何分布并应用于联合的每个成员的。

#### 38.4.4 分割字符串为代码单元

如果我们不使用固定的分隔符进行推断，那么推断出的第一个字符串始终是一个单个代码单元（一个“JavaScript 字符”，而不是由一个或两个代码单元组成的 Unicode 码点）：

```ts
type SplitCodeUnits<Str extends string> =
 Str extends `${infer First}${infer Rest}`
 // `First` is not empty
 ? [First, ...SplitCodeUnits<Rest>]
 // `First` (and therefore `Str`) is empty
 : []
;
type _ = [
 Assert<Equal<
 SplitCodeUnits<'rainbow'>,
 ['r', 'a', 'i', 'n', 'b', 'o', 'w']
 >>,
 Assert<Equal<
 SplitCodeUnits<''>,
 []
 >>,
];

```

### 38.5 与对象一起工作

#### 38.5.1 示例：向属性名添加前缀

下面，我们使用映射类型将对象类型 `Obj` 转换为新的对象类型，其中每个属性键以美元符号 (`$`) 开头：

```ts
type PrependDollarSign<Obj> = {
 [Key in (keyof Obj & string) as `$${Key}`]: Obj[Key]
};

type Items = {
 count: number,
 item: string,
 [Symbol.toStringTag]: string,
};
type _ = Assert<Equal<
 PrependDollarSign<Items>,
 {
 $count: number,
 $item: string,
 // Omitted: [Symbol.toStringTag]
 }
>>;

```

这段代码是如何工作的？我们组装一个输出对象，如下所示：

+   `PrependDollarSign` 的结果通过循环计算。在每次迭代中，循环变量 `Key` 被分配给字符串字面量类型联合中的一个元素。这里有三种描述该联合的方式：

    +   `keyof Obj & string`

    +   `Obj` 的键与所有字符串的交集

    +   `Obj` 的字符串键

+   每次循环迭代向输出对象贡献一个属性：

    +   该属性键通过 `as` 指定：`` `$${Key}` ``

    +   该属性的值位于冒号之后：`Obj[Key]`

### 38.6 实际示例

#### 38.6.1 示例：将输出样式化到终端

在 Node.js 中，我们可以使用 [util.styleText()](https://nodejs.org/api/util.html#utilstyletextformat-text-options) 将格式化文本输出到控制台：

```ts
console.log(
 util.styleText(['bold', 'underline', 'red'], 'Hello!')
);

```

要获取所有可能的样式值列表，请在 Node.js REPL 中评估此表达式：

```ts
Object.keys(util.inspect.colors)

```

下面，我们定义一个 `styleText()` 函数，该函数使用单个字符串来指定多个样式，并静态检查该字符串是否具有正确的格式：

```ts
const styles = [
 'bold',
 'italic',
 'underline',
 'red',
 'green',
 'blue',
 // ...
] as const; // `as const` enables us to derive a type
type StyleUnion = TupleToUnion<typeof styles>;
type TupleToUnion<Tup extends readonly unknown[]> = Tup[number]; 

type StyleTextFormat =
| `${StyleUnion}`
| `${StyleUnion}+${StyleUnion}`
| `${StyleUnion}+${StyleUnion}+${StyleUnion}`
;

function styleText(format: StyleTextFormat, text: string): string {
 return util.styleText(format.split('+'), text);
}

styleText('bold+underline+red', 'Hello!'); // OK
// @ts-expect-error: Argument of type '"bol+underline+red"' is not
// assignable to parameter of type 'StyleTextFormat'.
styleText('bol+underline+red', 'Hello!'); // typo: 'bol'

```

#### 38.6.2 示例：属性路径

以下示例基于 [Anders Hejlsberg](https://github.com/microsoft/TypeScript/pull/40336) 的代码：

```ts
type PropType<T, Path extends string> =
 Path extends keyof T
 // `Path` is already a key of `T`
 ? T[Path]
 // Otherwise: extract first dot-separated key
 : Path extends `${infer First}.${infer Rest}`
 ? First extends keyof T
 // Use key `First` and compute PropType for result
 ? PropType<T[First], Rest>
 : unknown
 : unknown;

function getPropValue
 <T, P extends string>
 (value: T, path: P): PropType<T, P>
{
 // Not implemented yet...
 return null as PropType<T, P>;
}

const obj = { a: { b: ['x', 'y']}} as const;

assertType<
 { readonly b: readonly ['x', 'y'] }
>(getPropValue(obj, 'a'));

assertType<
 readonly ['x', 'y']
>(getPropValue(obj, 'a.b'));

assertType<
 'y'
>(getPropValue(obj, 'a.b.1'));

assertType<
 unknown
>(getPropValue(obj, 'a.b.p'));

function myFunc(str: string) {
 // If the second argument is not a literal,
 // we can’t infer a return type.
 assertType<
 unknown
 >(getPropValue(obj, str));
}

```

#### 38.6.3 示例：更改属性名前缀

这是一个更复杂的早期示例的版本：

+   类型 `JsonLd` 描述了 [JSON-LD 格式的结构化数据](https://developers.google.com/search/docs/appearance/structured-data/article)。

+   我们使用 `PropKeysAtToUnderscore` 将 `JsonLd` 类型转换为不需要引号的键的类型。

```ts
type PropKeysAtToUnderscore<Obj> = {
 [Key in keyof Obj as AtToUnderscore<Key>]: Obj[Key];
};
type AtToUnderscore<Key> =
 // Remove prefix '@', add prefix '_'
 Key extends `@${infer Rest}` ? `_${Rest}` : Key // (A)
;

type JsonLd = {
 '@context': string,
 '@type': string,
 datePublished: string,
};
type _ = Assert<Equal<
 PropKeysAtToUnderscore<JsonLd>,
 {
 _context: string,
 _type: string,
 datePublished: string,
 }
>>;

```

注意，在行 A 中，我们只更改以 at 符号开头的字符串键。其他键（包括符号）不会被更改。

我们可以像这样约束 `Key` 的类型：

```ts
AtToUnderscore<Key extends string>

```

但然后我们无法使用 `AtToUnderscore` 对符号进行操作，必须在传递给这个实用类型之前以某种方式过滤值。

#### 38.6.4 示例：将驼峰形式转换为连字符形式

我们可以使用模板字面量类型将驼峰形式的字符串（JavaScript）转换为连字符形式的字符串（CSS）。

让我们先转换一个字符串为一个字符串元组的元组：

```ts
type SplitCamelCase<
 Str extends string,
 Word extends string = '',
 Words extends string[] = []
> = Str extends `${infer Char}${infer Rest}`
 ? IsUppercase<Char> extends true
 // `Word` is empty if initial `Str` starts with capital letter
 ? SplitCamelCase<Rest, Char, Append<Words, Word>>
 : SplitCamelCase<Rest, `${Word}${Char}`, Words>
 // We have reached the end of `Str`:
 // `Word` is only empty if initial `Str` was empty.
 : [...Words, Word]
;
type IsUppercase<Str extends string> = Str extends Uppercase<Str>
 ? true
 : false
;
// Only append `Str` to `Arr` if `Str` isn’t empty
type Append<Arr extends string[], Str extends string> =
 Str extends ''
 ? Arr
 : [...Arr, Str]
;

type _1 = [
 Assert<Equal<
 SplitCamelCase<'howAreYou'>,
 ['how', 'Are', 'You']
 >>,
 Assert<Equal<
 SplitCamelCase<'PascalCase'>,
 ['Pascal', 'Case']
 >>,
 Assert<Equal<
 SplitCamelCase<'CAPS'>,
 ['C', 'A', 'P', 'S']
 >>,
];

```

要理解它是如何工作的，考虑参数的角色：

+   `Str` 是实际参数，也是唯一一个没有默认值的参数。`SplitCamelCase` 使用递归遍历 `Str` 的字符。

+   `Word`: 当在单词内部时，我们将字符添加到这个参数中。

+   `Words`: 一旦一个单词完成，它就会被添加到这个（最初为空的）元组中。一旦递归完成，`Words` 就包含了 `SplitCamelCase` 的结果，并返回。

剩下的工作涉及将元组元素转换为小写并连接起来，以连字符作为分隔符：

```ts
type ToHyphenCase<Str extends string> =
 HyphenateWords<SplitCamelCase<Str>>
;
type HyphenateWords<Words extends string[]> =
 Words extends [
 infer First extends string,
 ...infer Rest extends string[]
 ]
 ? Rest['length'] extends 0
 ? Lowercase<First>
 : `${Lowercase<First>}-${HyphenateWords<Rest>}`
 : ''
;

type _2 = [
 Assert<Equal<
 ToHyphenCase<'howAreYou'>,
 'how-are-you'
 >>,
 Assert<Equal<
 ToHyphenCase<'PascalCase'>,
 'pascal-case'
 >>,
];

```

#### 38.6.5 示例：将连字符形式转换为驼峰形式

从连字符形式转换为驼峰形式是更容易的，因为我们有连字符作为分隔符。作为一个实用类型，我们使用本章早些时候提到的 `Split`。

```ts
type ToLowerCamelCase<Str extends string> =
 Uncapitalize<ToUpperCamelCase<Str>>
;
// Upper camel case (Pascal case) is easier to compute
type ToUpperCamelCase<Str extends string> =
 camelizeWords<Split<Str, '-'>>
;

type camelizeWords<Words extends string[]> =
 Words extends [
 infer First extends string,
 ...infer Rest extends string[]
 ]
 ? Rest['length'] extends 0
 ? Capitalize<First>
 : `${Capitalize<First>}${camelizeWords<Rest>}`
 : ''
;

type _ = [
 Assert<Equal<
 ToLowerCamelCase<'how-are-you'>,
 'howAreYou'
 >>,
 Assert<Equal<
 ToUpperCamelCase<'how-are-you'>,
 'HowAreYou'
 >>,
];

```

### 38.7 人们使用模板字面量类型所做的事情的巧妙之处

在本节中，我收集了一些人们使用模板字面量类型所做的事情。

#### 38.7.1 Node.js：UUID 的类型

[`@types/node`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/node/crypto.d.ts) 使用以下类型来表示 UUID：

```ts
type UUID = `${string}-${string}-${string}-${string}-${string}`;

```

#### 38.7.2 解析 CLI 参数（Stefan Baumgartner）

给定以下关于 shell 脚本参数的定义（`num` 是一个字符串参数的任意名称——不是一个类型）：

```ts
const opts = program
 .option("-e, --episode <num>", "Download episode No. <num>")
 .option("--keep", "Keeps temporary files")
 .option("--ratio [ratio]", "Either 16:9, or a custom ratio")
 .opts();

```

这个类型可以静态地推导出 `opts`：

```ts
{
 episode: string;
} & {
 keep: boolean;
} & {
 ratio: string | boolean;
}

```

更多信息：[“TypeScript 的收敛点”](https://fettblog.eu/slides/the-typescript-converging-point/) by Stefan Baumgartner

#### 38.7.3 `document.querySelector()` 的智能结果类型（Mike Ryan）

通过在编译时解析 `querySelector()` 的参数，我们可以推导出好的类型：

```ts
const a = querySelector('div.banner > a.call-to-action');
 // HTMLAnchorElement
const b = querySelector('input, div');
 // HTMLInputElement | HTMLDivElement
const c = querySelector('circle[cx="150"]');
 // SVGCircleElement
const d = querySelector('button#buy-now');
 // HTMLButtonElement
const e = querySelector('section p:first-of-type');
 // HTMLParagraphElement

```

更多信息：[推文](https://x.com/MikeRyanDev/status/1308472279010025477) by Mike Ryan.

#### 38.7.4 在 Angular 中输入路由（Mike Ryan）

```ts
const AppRoutes = routes(
 {
 path: '' as const,
 },
 {
 path: 'book/:id' as const,
 children: routes(
 {
 path: 'author/:id' as const,
 },
 ),
 },
);

// `AppRoutes` determines (statically!) what arguments can be
// passed to `buildPath()`:

const buildPath = createPathBuilder(AppRoutes);
buildPath(); // OK
buildPath('book', 12); // OK
buildPath('book', '123', 'author', 976); // OK

buildPath('book', null);
 // Error: argument not assignable to parameter
buildPath('fake', 'route');
 // Error: argument not assignable to parameter

```

更多信息：[推文](https://x.com/MikeRyanDev/status/1308036861747752962) by Mike Ryan.

#### 38.7.5 Express 路由提取器（Dan Vanderkam）

`handleGet()`的第一个参数决定了回调的参数：

```ts
handleGet(
 '/posts/:postId/:commentId',
 ({postId, commentId}) => {
 console.log(postId, commentId);
 }
);

```

更多信息：[丹·范德坎姆的推文](https://x.com/danvdk/status/1301707026507198464)

#### 38.7.6 Tailwind 颜色变化（托梅克·苏尔科夫斯基）

由于模板字面量是分配性的，我们可以非常简洁地定义`TailwindColor`：

```ts
type BaseColor =
 'gray' | 'red' | 'yellow' | 'green' |
 'blue' | 'indigo' | 'purple' | 'pink';
type Variant =
 50 | 100 | 200 | 300 | 400
 500 | 600 | 700 | 800 | 900;
type TailwindColor = `${BaseColor}-${Variant}`;

```

更多信息：[托梅克·苏尔科夫斯基的推文](https://x.com/sulco/status/1332337570563448834)

#### 38.7.7 Arktype：定义类型

在 TypeScript 库 Zod[使用链式方法调用来定义类型](https://zod.dev)的地方，ArkType 库[使用（主要是）通过模板字面量类型解析的字符串字面量](https://github.com/arktypeio/arktype)。我更喜欢 Zod 的方法，但 ArkType 使用字符串字面量所做的事情真是令人惊叹：

```ts
const currentTsSyntax = type({
 keyword: "null",
 stringLiteral: "'TS'",
 numberLiteral: "5",
 bigintLiteral: "5n",
 union: "string|number",
 intersection: "boolean&true",
 array: "Date[]",
 grouping: "(0|1)[]",
 objectLiteral: {
 nested: "string",
 "optional?": "number"
 },
 tuple: ["number", "number"]
});

```

### 38.8 结论和注意事项

人们使用模板字面量类型所做的事情真是令人惊叹：我们现在可以在字符串字面量中静态检查复杂的数据，或者使用它们来推导类型。然而，这样做也有一些注意事项：

+   错误信息通常不是很好。

+   使用模板字面量的类型级代码可能难以理解。

+   这样的代码可能会减慢类型检查的速度——尤其是如果它涉及到递归的话。

### 38.9 进一步阅读

+   模板字面量类型示例列表：[“令人惊叹的模板字面量类型”](https://github.com/ghoullier/awesome-template-literal-types)

### 38.10 本章来源

+   TypeScript 手册中的[“模板字面量类型”](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)章节

+   由安德斯·海尔斯伯格发起的拉取请求[“模板字面量类型和映射类型 `as` 子句”](https://github.com/microsoft/TypeScript/pull/40336)

+   博客文章[“在 TypeScript 中将字符串字面量类型转换为 camelCase”](https://blog.beraliv.dev/2022-07-14-camel-case)，作者：阿列克谢·贝列津

+   博客文章[“我不知道你可以在 TypeScript 中组合模板字面量类型”](https://macarthur.me/posts/template-literal-types/)，作者：亚历克斯·麦克阿瑟
