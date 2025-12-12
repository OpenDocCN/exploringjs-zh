# 32 验证外部数据

> 原文：[`exploringjs.com/ts/book/ch_validating-external-data.html`](https://exploringjs.com/ts/book/ch_validating-external-data.html)

(广告，请勿屏蔽。)

1.  32.1 JSON 模式

    1.  32.1.1 一个 JSON 模式示例

1.  32.2 TypeScript 中数据验证的方案

    1.  32.2.1 不使用 JSON 模式的方案

    1.  32.2.2 使用 JSON 模式的方案

    1.  32.2.3 选择库

1.  32.3 示例：通过库 *Zod* 验证数据

    1.  32.3.1 通过 Zod 的构建器 API 定义“模式”

    1.  32.3.2 验证数据

    1.  32.3.3 小贴士：使用 `z.interface()` 和带问号的属性键来指定可选属性（Zod 4）

    1.  32.3.4 数据的外部表示与内部表示

1.  32.4 结论：关于数据验证库的各种思考

*数据验证*意味着确保数据具有所需的结构和内容。

使用 TypeScript，当我们接收到外部数据时，验证变得相关：

+   从 JSON 文件解析的数据

+   来自 Web 服务的接收数据

在这些情况下，我们期望数据符合我们拥有的静态类型，但我们不能确定。这与我们创建的数据形成对比，TypeScript 会持续检查一切是否正确。

本章解释了如何在 TypeScript 中验证外部数据。

### 32.1 JSON 模式

在我们探索 TypeScript 中数据验证的方案之前，我们需要看一下*JSON 模式*，因为其中一些方案基于它。

JSON 模式的理念是将 JSON 数据的*模式*（结构和内容，类似于静态类型）用 JSON 表达。也就是说，元数据以与数据相同的格式表达。

JSON 模式的用例包括：

+   验证 JSON 数据：如果我们有数据模式定义，我们可以使用工具来检查数据是否正确。数据的一个问题也可以自动修复：我们可以指定默认值，用于添加缺失的属性。

+   记录 JSON 数据格式：一方面，核心模式定义可以被视为文档。但 JSON 模式还支持描述、弃用说明、注释、示例等。这些机制被称为*注解*。它们不用于验证，而是用于文档。

+   IDE 对数据编辑的支持：例如，Visual Studio Code 支持 JSON 模式。如果 JSON 文件有模式，我们将获得几个编辑功能：自动完成、错误突出显示等。值得注意的是，VS Code 对 `package.json` 文件的支持完全基于 JSON 模式。

#### 32.1.1 一个 JSON 模式示例

此示例取自 [json-schema.org 网站](https://json-schema.org/learn/miscellaneous-examples.html#describing-geographical-coordinates)：

```ts
{
 "$id": "https://example.com/geographical-location.schema.json",
 "$schema": "http://json-schema.org/draft-07/schema#",
 "title": "Longitude and Latitude Values",
 "description": "A geographical coordinate.",
 "required": [ "latitude", "longitude" ],
 "type": "object",
 "properties": {
 "latitude": {
 "type": "number",
 "minimum": -90,
 "maximum": 90
 },
 "longitude": {
 "type": "number",
 "minimum": -180,
 "maximum": 180
 }
 }
}

```

以下 JSON 数据符合此模式：

```ts
{
 "latitude": 48.858093,
 "longitude": 2.294694
}

```

### 32.2 TypeScript 中数据验证的方案

本节简要概述了在 TypeScript 中验证数据的各种方法。对于每种方法，我列出了一个或多个支持该方法的库。关于库，我不打算全面介绍，因为在这个领域变化很快。

#### 32.2.1 不使用 JSON 模式的方案

验证的一种方法是通过调用构建器方法和函数创建一个模式对象。这样的对象使我们能够：

+   在运行时检查数据。

+   在编译时为通过检查的数据提供适当的类型。

有许多库以这种方式工作。接下来，我们将看看几个例子。

##### 32.2.1.1 Zod

[Zod](https://github.com/vriad/zod)（在本章后面将更深入地演示）使用构建器方法，并且非常受欢迎。

```ts
import { z } from 'zod';

// Define schema
const ProductSchema = z.object({
 name: z.string().min(1), // non-empty
 scores: z.array(
 z.union([ z.number(), z.string() ])
 ),
});

// Derive TypeScript type from schema
type Product = z.infer<typeof ProductSchema>;
type _ = Assert<Equal<
 Product,
 {
 name: string,
 scores: Array<number | string>,
 }
>>;

// Validate data
const product = ProductSchema.parse({
 name: 'Toothpaste',
 scores: [ 5, '*****' ],
});
assertType<Product>(product);

```

##### 32.2.1.2 Valibot

[Valibot](https://github.com/fabian-hiller/valibot) 与 Zod 类似，但使用函数，这有助于从包中排除未使用的代码。

```ts
import * as v from 'valibot';

// Define schema
const ProductSchema = v.object({
 name: v.pipe(v.string(), v.minLength(1)), // non-empty
 scores: v.array(
 v.union([ v.number(), v.string() ])
 ),
});

// Derive TypeScript type from schema
type Product = v.InferOutput<typeof ProductSchema>;
type _ = Assert<Equal<
 Product,
 {
 name: string,
 scores: Array<number | string>,
 }
>>;

// Validate data
const product = v.parse(ProductSchema, {
 name: 'Toothpaste',
 scores: [ 5, '*****' ],
});
assertType<Product>(product);

```

##### 32.2.1.3 ArkType

[ArkType](https://github.com/arktypeio/arktype) 有一种独特的指定类型的方式：它通常使用字符串字面量类型（在编译时通过模板字面量类型解析），而不是函数或方法调用。

```ts
import { type } from 'arktype';

// Define schema
const productSchema = type({
 name: 'string > 0', // non-empty
 scores: '(number | string)[]',
});

// Derive TypeScript type from schema
type Product = typeof productSchema.infer;
type _ = Assert<Equal<
 Product,
 {
 name: string,
 scores: Array<number | string>,
 }
>>;

// Validate data
const product = productSchema({
 name: 'Toothpaste',
 scores: [ 5, '*****' ],
});
if (product instanceof type.errors) {
 throw new TypeError(product.summary);
} else {
 assertType<Product>(product);
}

```

##### 32.2.1.4 标准模式：验证 API 的标准

“[标准模式](https://standardschema.dev)是一个旨在由 JavaScript 和 TypeScript 模式库实现的通用接口。”受 Zod 启发，由许多库支持。

#### 32.2.2 使用 JSON 模式的方案

+   方法：将 TypeScript 类型转换为 JSON 模式。库：

    +   [ts-json-schema-generator](https://github.com/vega/ts-json-schema-generator)

    +   [typescript-json-schema](https://github.com/YousefED/typescript-json-schema)

+   方法：将 JSON 模式转换为 TypeScript 类型。库：

    +   [quicktype](https://github.com/quicktype/quicktype)

    +   [json-schema-to-typescript](https://github.com/bcherny/json-schema-to-typescript)

+   方法：构建器 API 创建 TypeScript 类型和 JSON 模式。库：

    +   [TypeBox](https://github.com/sinclairzx81/typebox)（它还验证 TypeScript 代码中的无类型数据）

+   方法：通过 JSON 模式验证 JSON 数据。此功能对其他方法也很有用。npm 包：

    +   [Ajv JSON 模式验证器](https://github.com/ajv-validator/ajv)

#### 32.2.3 选择库

使用哪种方法和相应的库取决于我们的需求：

+   如果我们从 TypeScript 类型开始，并希望确保数据（来自配置文件等）符合这些类型，那么支持静态类型的构建器 API 是一个好的选择。

+   如果我们的起点是 JSON 模式，那么我们应该考虑支持 JSON 模式的库之一。

### 32.3 示例：通过库 *Zod* 验证数据

#### 32.3.1 通过 Zod 的构建器 API 定义“模式”

Zod 有一个构建器 API，它生成类型和验证函数。该 API 的使用方法如下：

```ts
import * as z from 'zod';

const FileEntryInputSchema = z.union([
 z.string(),
 z.tuple([z.string(), z.string(), z.array(z.string())]),
 z.interface({
 file: z.string(),
 'author?': z.string(),
 'tags?': z.array(z.string()),
 }),
]);

```

注意：`z.interface()` 和带有问号的属性键是 Zod 4 的特性。下面的小节解释了为什么在这里使用它们。

对于较大的模式，将事物分解为多个 `const` 声明是有意义的。

Zod 可以从 `FileEntryInputSchema` 生成静态类型，但我决定（冗余地！）手动维护静态类型 `FileEntryInput`：

```ts
type FileEntryInput =
 | string
 | [ string, string, Array<string> ]
 | { file: string, author?: string, tags?: Array<string> }
 ;

```

为什么会有冗余？

+   它更容易阅读。

+   如果我需要迁移到不同的验证库或方法，这有助于迁移。

我们可以使用静态检查来确保 `FileEntryInputSchema` 和 `FileEntryInput` 保持同步：

```ts
type _ = Assert<Equal<
 z.infer<typeof FileEntryInputSchema>,
 FileEntryInput
>>;

```

泛型类型 `z.infer` 从 Zod 模式推导出类型。

#### 32.3.2 验证数据

模式方法 `.parse()` 检查一个值是否具有正确的结构：

```ts
const fileEntryInput: FileEntryInput = FileEntryInputSchema.parse( // OK
 ['iceland.txt', 'me', ['vacation', 'family']]
);
assert.throws(
 () => FileEntryInputSchema.parse(['iceland.txt', 'me'])
);

```

#### 32.3.3 小贴士：使用 `z.interface()` 和带有问号的属性键来定义可选属性（Zod 4）

它们是 Zod 4 的特性。没有它们，可选属性将具有错误的类型：

```ts
const Schema = z.object({
 file: z.string(),
 author: z.string().optional(),
 tags: z.array(z.string()).optional(),
});
type _ = Assert<Equal<
 z.infer<typeof Schema>,
 {
 file: string,
 author?: string | undefined, // (A)
 tags?: Array<string> | undefined, // (B)
 }
>>;

```

`.optional()` 并不完全符合我们的需求：除了使属性可选外，它还将 `undefined` 添加到其类型中（行 A 和行 B）。

#### 32.3.4 数据的外部表示与内部表示

当处理外部数据时，区分两种类型通常很有用。

一方面，有描述输入数据的类型。其结构优化以便于编写：

```ts
type FileEntryInput =
 | string
 | [ string, string, Array<string> ]
 | { file: string, author?: string, tags?: Array<string> }
 ;

```

另一方面，有在程序中使用的类型。其结构优化以便于在代码中使用：

```ts
type FileEntry = {
 file: string,
 author: null | string,
 tags: Array<string>,
};

```

在我们使用 Zod 确保输入数据符合 `FileEntryInput` 之后，我们可以使用一个转换函数，将数据转换为 `FileEntry` 类型的值。

### 32.4 结论：关于数据验证库的各种思考

我看到两种长期改进数据验证库的方法：

+   TypeScript 中内置对运行时类型表示的支持：这将有助于验证库——至少对于简单的用例。对于高级用例，可能可以利用装饰器。

+   内置支持将类型编译为验证代码。这可能与 Rust 支持的宏类似。

对于具有构建器 API 的库，我会发现将 TypeScript 类型编译为构建器 API 调用的工具很有用（在线和通过 shell 命令）。这将以两种方式帮助：

+   这些工具可以用来探索 API 的工作方式。

+   我们可以选择通过工具生成 API 代码。
