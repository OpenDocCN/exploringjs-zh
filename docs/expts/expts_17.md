# 13 TypeScript 中的类型是什么？两种视角

> 原文：[`exploringjs.com/ts/book/ch_what-is-a-type.html`](https://exploringjs.com/ts/book/ch_what-is-a-type.html)

（请勿阻塞，Ad。）

1.  13.1 每个视角的两个问题

1.  13.2 动态视角：类型是一组值

1.  13.3 静态视角：类型之间的关系

1.  13.4 命名类型系统与结构类型系统

1.  13.5 进一步阅读

TypeScript 中的类型是什么？本章描述了帮助理解它们的两种视角。两者都很有用；它们相互补充。

### 13.1 每个视角的两个问题

以下两个问题是理解类型工作方式的重要问题，需要从两个视角中回答。

1.  `arg` 具有类型 `MyType` 的意义是什么？

    ```ts
    function myFunc(arg: MyType): void {}

    ```

1.  `UnionType` 是如何从 `Type1` 和 `Type2` 派生出来的？

    ```ts
    type UnionType = Type1 | Type2;

    ```

### 13.2 动态视角：类型是一组值

从这个角度来看，我们关注的是值，类型是一组值：

1.  如果给定的值包含在 `MyType` 中，我们可以将其传递给 `myFunc()`。

1.  `UnionType`（一个集合）定义为 `Type1` 和 `Type2` 的集合论并集。

### 13.3 静态视角：类型之间的关系

从这个角度来看：

+   源代码有位置，每个位置都有一个静态类型。在 TypeScript 感知编辑器中，如果我们用鼠标悬停在某个位置上，我们可以看到该位置的静态类型。

+   类型是通过与其他类型的关系来定义的。

最重要的类型关系是*赋值兼容性*：类型为 `Src` 的位置能否被赋值给类型为 `Trg` 的位置？如果答案是肯定的，那么：

+   `Src` 和 `Trg` 是相同类型。

+   `Src` 或 `Trg` 是类型 `any`。

+   `Src` 是一个字符串字面量类型，而 `Trg` 是原始类型 `string`。

+   `Src` 是一个联合类型，`Src` 的每个组成部分都可以赋值给 `Trg`。

+   `Src` 是一个交集类型，且 `Src` 的至少一个组成部分可以赋值给 `Trg`。

+   `Trg` 是一个联合类型，`Src` 可以赋值给 `Trg` 的至少一个组成部分。

+   `Trg` 是一个交集类型，`Src` 可以赋值给 `Trg` 的每个组成部分。

+   等等。

让我们考虑以下问题：

1.  参数 `arg` 具有类型 `MyType` 意味着我们可以传递给 `myFunc()` 的值只能是 `MyType` 可以赋值的类型。

1.  `UnionType` 通过与其他类型的关系来定义。在上面，我们已经看到了联合类型的两个规则。

### 13.4 命名类型系统与结构类型系统

静态类型系统的一个职责是确定两个静态类型是否兼容——例如：

+   实际参数的静态类型 `Src`（例如，通过函数调用提供）

+   对应形式参数的静态类型 `Trg`（例如，作为函数定义的一部分指定）

类型系统需要检查 `Src` 是否可以分配给 `Trg`。这种检查有两种方法（大致上）：

+   在一个 *名义* 或 *指示* 类型系统中，如果两个静态类型具有相同的标识（“名称”），则它们是相等的。`Src` 只能在它们相等或显式指定了它们之间的关系时分配给 `Trg` – 例如，一个继承关系（`extends`）。

    +   具有名义类型系统的语言包括 C++、Java 和 C#。

+   在一个 *结构* 类型系统中，如果 `Trg` 具有可以接收 `Src` 中内容的结构，则类型 `Src` 可以分配给类型 `Trg` – 例如：对于每个字段 `Src.F`，必须存在一个字段 `Trg.F`，使得 `Src.F` 可以分配给 `Trg.F`。

    +   具有结构类型系统的语言包括 TypeScript、Go（接口）和 OCaml（对象）。

以下代码在具有名义类型系统的最后行会产生类型错误，但在 TypeScript 的结构类型系统中是合法的，因为类 `A` 和类 `B` 具有相同的结构：

```ts
class A {
 typeName = 'A';
}
class B {
 typeName = 'B';
}
const someVariable: A = new B();

```

TypeScript 的接口也以结构方式工作 – 它们不需要实现就可以匹配：

```ts
interface HasTypeName {
 typeName: string;
}
const hasTypeName: HasTypeName = new A(); // OK

```

### 13.5 进一步阅读

+   [TypeScript 手册中的“类型兼容性”章节](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)

+   TypeScript 语言规范 1.8：TypeScript 最初有一个正式的语言规范，但在 TypeScript 1.8（2016 年发布）之后被取消。它已被从 TypeScript 的存储库中移除，但可以从旧提交中下载 [PDF 文件](https://github.com/microsoft/TypeScript/blob/3c99d50da5a579d9fa92d02664b1b66d4ff55944/doc/TypeScript%20Language%20Specification%20-%20ARCHIVED.pdf)。特别有用的是“3.11 类型关系”部分。
