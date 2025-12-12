# 19 对象类型联合

> 原文：[`exploringjs.com/ts/book/ch_unions-object-types.html`](https://exploringjs.com/ts/book/ch_unions-object-types.html)

（广告，请勿拦截。）

1.  19.1 从对象类型联合到区分联合

    1.  19.1.1 示例：对象的联合

    1.  19.1.2 `FileEntry`作为区分联合

    1.  19.1.3 区分联合与代数数据类型的关系

    1.  19.1.4 新`FileEnty`的`readFile()`

    1.  19.1.5 区分联合的优缺点

1.  19.2 从区分联合推导类型

    1.  19.2.1 提取区分值（类型标签）

    1.  19.2.2 区分联合元素的映射

    1.  19.2.3 从区分联合中提取子类型

1.  19.3 类层次结构与区分联合

    1.  19.3.1 语法树的类层次结构

    1.  19.3.2 语法树的区分联合

    1.  19.3.3 比较类和区分联合

1.  19.4 通过类定义区分联合

在本章中，我们探讨了在 TypeScript 中对象类型联合的用途。

在本章中，*对象类型*指的是：

+   对象字面量类型

+   接口类型

+   映射类型（如`Record`）

### 19.1 从对象类型联合到区分联合

当一个类型有多个表示形式时，对象类型的联合通常是一个不错的选择——例如，一个可以是`Triangle`、`Rectangle`或`Circle`的`Shape`类型：

```ts
type Shape = Triangle | Rectangle | Circle;

type Triangle = {
 corner1: Point,
 corner2: Point,
 corner3: Point,
};
type Rectangle = {
 corner1: Point,
 corner2: Point,
};
type Circle = {
 center: Point,
 radius: number,
};

type Point = {
 x: number,
 y: number,
};

```

#### 19.1.1 示例：对象的联合

以下类型定义了一个简单的虚拟文件系统：

```ts
type VirtualFileSystem = Map<string, FileEntry>;

type FileEntry = FileEntryData | FileEntryGenerator | FileEntryFile;
type FileEntryData = {
 data: string,
};
type FileEntryGenerator = {
 generator: (path: string) => string,
};
type FileEntryFile = {
 path: string,
};

```

一个用于`VirtualFileSystem`的`readFile()`函数将如下工作（行 A 和行 B）：

```ts
const vfs: VirtualFileSystem = new Map([
 [ '/tmp/file.txt',
 { data: 'Hello!' }
 ],
 [ '/tmp/echo.txt',
 { generator: (path: string) => path }
 ],
]);
assert.equal(
 readFile(vfs, '/tmp/file.txt'), // (A)
 'Hello!'
);
assert.equal(
 readFile(vfs, '/tmp/echo.txt'), // (B)
 '/tmp/echo.txt'
);

```

这是对`readFile()`的一个实现：

```ts
import * as fs from 'node:fs';
function readFile(vfs: VirtualFileSystem, path: string): string {
 const fileEntry = vfs.get(path);
 if (fileEntry === undefined) {
 throw new Error('Unknown path: ' + JSON.stringify(path));
 }
 if ('data' in fileEntry) { // (A)
 return fileEntry.data;
 } else if ('generator' in fileEntry) { // (B)
 return fileEntry.generator(path);
 } else if ('path' in fileEntry) { // (C)
 return fs.readFileSync(fileEntry.path, 'utf-8');
 } else {
 throw new UnexpectedValueError(fileEntry); // (D)
 }
}

```

初始时，`fileEntry`的类型是`FileEntry`，因此：

```ts
FileEntryData | FileEntryGenerator | FileEntryFile

```

在我们可以访问属性之前，我们必须将类型缩小到这个联合类型的一个元素。TypeScript 允许我们通过`in`运算符（行 A，行 B，行 C）来做这件事。

此外，我们通过检查`fileEntry`是否可以赋值给类型`never`来静态地检查我们是否覆盖了所有可能的案例。这是通过以下类来完成的：

```ts
class UnexpectedValueError extends Error {
 constructor(_value: never) {
 super();
 }
}

```

更多关于这种技术以及`UnexpectedValueError`的更长和更好的实现信息，请参阅“`never`的使用场景：编译时的完备性检查”（§15.4）“Use case for `never`: exhaustiveness checks at compile time” (§15.4)。

#### 19.1.2 `FileEntry`作为区分联合

*判别式联合*是具有一个共同属性的对象类型的联合，该属性的值指示联合元素的类型。让我们将 `FileEntry` 转换为判别式联合：

```ts
type FileEntry =
 | {
 kind: 'FileEntryData',
 data: string,
 }
 | {
 kind: 'FileEntryGenerator',
 generator: (path: string) => string,
 }
 | {
 kind: 'FileEntryFile',
 path: string,
 }
 ;
type VirtualFileSystem = Map<string, FileEntry>;

```

具有类型信息的判别式联合的属性称为*判别器*或*类型标签*。`FileEntry` 的判别器是 `.kind`。其他常见名称包括 `.tag`、`.key` 和 `.type`。

一方面，`FileEntry` 现在更加冗长。另一方面，判别式给我们带来了几个好处——正如我们很快就会看到的。

#### 19.1.3 判别式联合与代数数据类型相关

作为旁注，[判别式联合](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions)与函数式编程语言中的[代数数据类型](https://wiki.haskell.org/Algebraic_data_type)相关。这是 `FileEntry` 在 Haskell 中作为代数数据类型的样子（如果 TypeScript 联合元素有更多属性，我们可能会在 Haskell 中使用记录）。

```ts
data FileEntry = FileEntryData String
 | FileEntryGenerator (String -> String)
 | FileEntryFile String

```

#### 19.1.4 `readFile()` for the new `FileEntry`

让我们调整 `readFile()` 以适应新的 `FileEnty` 形状：

```ts
function readFile(vfs: VirtualFileSystem, path: string): string {
 const fileEntry = vfs.get(path);
 if (fileEntry === undefined) {
 throw new Error('Unknown path: ' + JSON.stringify(path));
 }
 switch (fileEntry.kind) {
 case 'FileEntryData':
 return fileEntry.data;
 case 'FileEntryGenerator':
 return fileEntry.generator(path);
 case 'FileEntryFile':
 return fs.readFileSync(fileEntry.path, 'utf-8');
 default:
 throw new UnexpectedValueError(fileEntry);
 }
}

```

这使我们来到了判别式联合的第一个优点：我们可以使用 `switch` 语句。而且，立即很明显 `.kind` 区分了类型联合元素——我们不必寻找仅对元素唯一的属性名称。

注意，缩窄与之前一样工作：一旦我们检查了 `.kind`，我们就可以访问所有相关属性。

#### 19.1.5 判别式联合的优缺点

+   缺点：区分对象类型的联合使其更加冗长。

+   优点：我们可以通过 `switch` 语句处理情况。

+   优点：立即很明显哪个属性区分了联合的元素。

##### 19.1.5.1 优点：内联联合类型元素带有描述

另一个好处是，如果联合元素是内联的（而不是通过具有名称的类型在外部定义），我们仍然可以看到每个元素的作用：

```ts
type Shape =
| {
 tag: 'Triangle',
 corner1: Point,
 corner2: Point,
 corner3: Point,
}
| {
 tag: 'Rectangle',
 corner1: Point,
 corner2: Point,
}
| {
 tag: 'Circle',
 center: Point,
 radius: number,
}
;

```

##### 19.1.5.2 优点：联合元素不需要具有唯一的属性

即使联合元素的所有正常属性都相同，判别式联合也可以工作：

```ts
type Temperature =
 | {
 type: 'TemperatureCelsius',
 value: number,
 }
 | {
 type: 'TemperatureFahrenheit',
 value: number,
 }
;

```

##### 19.1.5.3 对象类型联合的通用优点：描述性

以下类型定义简洁；但你能否知道它是如何工作的？

```ts
type OutputPathDef =
 | null // same as input path
 | '' // stem of output path
 | string // output path with different extension

```

如果我们使用判别式联合，代码将变得更加自我描述：

```ts
type OutputPathDef =
 | { key: 'sameAsInputPath' }
 | { key: 'inputPathStem' }
 | { key: 'inputPathStemPlusExt', ext: string }
 ;

```

这是一个使用 `OutputPathDef` 的函数：

```ts
import * as path from 'node:path';
function deriveOutputPath(def: OutputPathDef, inputPath: string): string {
 if (def.key === 'sameAsInputPath') {
 return inputPath;
 }
 const parsed = path.parse(inputPath);
 const stem = path.join(parsed.dir, parsed.name);
 switch (def.key) {
 case 'inputPathStem':
 return stem;
 case 'inputPathStemPlusExt':
 return stem + def.ext;
 }
}
const zip = { key: 'inputPathStemPlusExt', ext: '.zip' } as const;
assert.equal(
 deriveOutputPath(zip, '/tmp/my-dir'),
 '/tmp/my-dir.zip'
);

```

### 19.2 从判别式联合推导类型

在本节中，我们探讨如何从判别式联合推导类型。作为一个例子，我们处理以下判别式联合：

```ts
type Content =
 | {
 kind: 'text',
 charCount: number,
 }
 | {
 kind: 'image',
 width: number,
 height: number,
 }
 | {
 kind: 'video',
 width: number,
 height: number,
 runningTimeInSeconds: number,
 }
;

```

#### 19.2.1 提取判别式的值（类型标签）

要提取区分值的值，我们可以使用 索引访问类型 (`T[K]`)：

```ts
type GetKind<T extends {kind: string}> =
 T['kind'];

type ContentKind = GetKind<Content>;

type _ = Assert<Equal<
 ContentKind,
 'text' | 'image' | 'video'
>>;

```

由于索引访问类型在联合上是 分配的，`T['kind']` 被应用于 `Content` 的每个元素，结果是字符串字面量类型的联合。

#### 19.2.2 为区分联合的元素创建映射

如果我们使用前一小节中的类型 `ContentKind`，我们可以为 `Content` 的元素定义一个详尽的映射：

```ts
const DESCRIPTIONS_FULL: Record<ContentKind, string> = {
 text: 'plain text',
 image: 'an image',
 video: 'a video',
} as const;

```

如果映射不需要是详尽的，我们可以使用实用类型 `Partial`：

```ts
const DESCRIPTIONS_PARTIAL: Partial<Record<ContentKind, string>> = {
 text: 'plain text',
} as const;

```

#### 19.2.3 从区分联合中提取子类型

有时，我们不需要区分联合的所有内容。我们可以编写自己的实用类型来提取 `Content` 的子类型：

```ts
type ExtractSubtype<
 Union extends {kind: string},
 SubKinds extends GetKind<Union> // (A)
> =
 Union extends {kind: SubKinds} ? Union : never // (B)
;

```

我们使用 条件类型 来遍历联合类型 `U`：

+   行 B：如果一个联合元素的属性 `.kind` 的类型可以被分配给 `SubKinds`，那么我们保留该元素。如果不能，则省略它（通过返回 `never`）。

+   行 A 中的 `extends` 确保我们在提取时不会出错：我们的区分值 `SubKinds` 必须是 `GetKind<Union>` 的子集（参见前面的子节）。

让我们使用 `ExtractSubtype`：

```ts
type _ = Assert<Equal<
 ExtractSubtype<Content, 'text' | 'image'>,
 | {
 kind: 'text',
 charCount: number,
 }
 | {
 kind: 'image',
 width: number,
 height: number,
 }
>>;

```

作为我们自己的 `ExtractSubtype` 的替代，我们也可以使用内置实用类型 `Extract`：

```ts
type _ = Assert<Equal<
 Extract<Content, {kind: 'text' | 'image'}>,
 | {
 kind: 'text',
 charCount: number,
 }
 | {
 kind: 'image',
 width: number,
 height: number,
 }
>>;

```

`Extract` 返回联合 `Content` 中所有可以分配给以下类型的元素：

```ts
{kind: 'text' | 'image'}

```

### 19.3 类层次结构与区分联合的比较

要比较类层次结构与区分联合，我们使用两者来定义表示表达式（如）的语法树：

```ts
1 + 2 + 3

```

语法树可以是：

+   一个数值

+   两个语法树的加法

#### 19.3.1 语法树的类层次结构

以下代码使用一个抽象类和两个子类来表示语法树：

```ts
abstract class SyntaxTree {
 abstract evaluate(): number;
}

class NumberValue extends SyntaxTree {
 numberValue: number;
 constructor(numberValue: number) {
 super();
 this.numberValue = numberValue;
 }
 evaluate(): number {
 return this.numberValue;
 }
}
class Addition extends SyntaxTree {
 operand1: SyntaxTree;
 operand2: SyntaxTree;
 constructor(operand1: SyntaxTree, operand2: SyntaxTree) {
 super();
 this.operand1 = operand1;
 this.operand2 = operand2;
 }
 evaluate(): number {
 return this.operand1.evaluate() + this.operand2.evaluate();
 }
}

```

操作 `evaluate` 在相应的类中处理两个情况：“数值”和“加法”——通过多态。这里就是它的作用：

```ts
const syntaxTree = new Addition(
 new NumberValue(1),
 new Addition(
 new NumberValue(2),
 new NumberValue(3),
 ),
);
assert.equal(
 syntaxTree.evaluate(), 6
);

```

#### 19.3.2 关于语法树的区分联合

以下代码使用具有两个元素的区分联合来表示语法树：

```ts
type SyntaxTree =
 | {
 kind: 'NumberValue';
 numberValue: number;
 }
 | {
 kind: 'Addition';
 operand1: SyntaxTree;
 operand2: SyntaxTree; 
 }
;

function evaluate(syntaxTree: SyntaxTree): number {
 switch(syntaxTree.kind) {
 case 'NumberValue':
 return syntaxTree.numberValue;
 case 'Addition':
 return (
 evaluate(syntaxTree.operand1) +
 evaluate(syntaxTree.operand2)
 );
 default:
 throw new UnexpectedValueError(syntaxTree);
 }
}

```

操作 `evaluate` 在单个位置处理两个情况：“数值”和“加法”，通过 `switch`。这里就是它的作用：

```ts
const syntaxTree: SyntaxTree = {
 kind: 'Addition',
 operand1: {
 kind: 'NumberValue',
 numberValue: 1,
 },
 operand2: {
 kind: 'Addition',
 operand1: {
 kind: 'NumberValue',
 numberValue: 2,
 },
 operand2: {
 kind: 'NumberValue',
 numberValue: 3,
 },
 }
};
assert.equal(
 evaluate(syntaxTree), 6
);

```

我们不需要在行 A 中的类型注解，但它有助于确保数据具有正确的结构。如果我们在这里不这样做，我们会在以后发现问题。

#### 19.3.3 比较类和区分联合

使用类时，我们通过 `instanceof` 检查实例的类型。对于区分联合，我们使用区分值来这样做。从某种意义上说，它们是运行时类型信息。

每种方法都擅长一种类型的可扩展性：

+   使用类时，如果我们想添加一个新操作，我们必须修改每个类。然而，添加新类型不需要对现有代码进行任何更改。

+   使用区分联合时，如果我们想添加一个新类型，我们必须修改每个函数。相比之下，添加新操作很简单。

### 19.4 定义通过类实现的区分联合

也可以通过类来定义区分联合——例如：

```ts
type Color = Black | White;

abstract class AbstractColor {}
class Black extends AbstractColor {
 readonly kind = 'Black';
}
class White extends AbstractColor {
 readonly kind = 'White';
}

function colorToRgb(color: Color): string {
 switch (color.kind) {
 case 'Black':
 return '#000000';
 case 'White':
 return '#FFFFFF';
 }
}

```

我们为什么要这样做呢？我们可以为联合的元素定义和继承方法。

抽象类 `AbstractColor` 只在我们要在联合类之间共享方法时才需要。
