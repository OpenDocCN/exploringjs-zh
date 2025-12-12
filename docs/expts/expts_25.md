# 20 对象类型交集

> 原文：[`exploringjs.com/ts/book/ch_intersections-object-types.html`](https://exploringjs.com/ts/book/ch_intersections-object-types.html)

(请勿拦截，广告。)

1.  20.1 对象类型交集

    1.  20.1.1 扩展与交集

    1.  20.1.2 示例：`NonNullable` (`T & {}`)

    1.  20.1.3 示例：推断出的交集

    1.  20.1.4 示例：通过交集组合两个对象类型

在本章中，我们探讨在 TypeScript 中对象类型交集的用途。

在本章中，*对象类型*指的是：

+   对象字面量类型

+   接口类型

+   映射类型（例如`Record`）

### 20.1 对象类型交集

两个对象类型的交集具有两者的属性：

```ts
type Obj1 = { prop1: boolean };
type Obj2 = { prop2: number };

const obj: Obj1 & Obj2 = {
 prop1: true,
 prop2: 123,
};

```

#### 20.1.1 扩展与交集

使用接口，我们可以使用`extends`来添加属性：

```ts
interface Person {
 name: string;
}
interface Employe extends Person {
 company: string;
}

```

使用对象类型，我们可以使用交集：

```ts
type Person = {
 name: string,
};

type Employee =
 & Person
 & {
 company: string,
 }
;

```

一个需要注意的问题是，只有`extends`支持重写。更多信息，请参阅$type。

#### 20.1.2 示例：`NonNullable` (`T & {}`)

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
 NonNullable<string>, // (A)
 string
 >>,
];

```

`NonNullable<T>`的结果是一个类型，它是`T`与所有非空值交集的结果。

很有趣的是`string & {}`等于`string`（行 A）。

#### 20.1.3 示例：推断出的交集

以下代码展示了当我们使用内置类型守卫`in`时，`obj`的推断类型是如何变化的（行 A 和行 B）：

```ts
function func(obj: object) {
 if ('prop1' in obj) { // (A)
 assertType<
 object & Record<'prop1', unknown>
 >(obj);
 if ('prop2' in obj) { // (B)
 assertType<
 object & Record<'prop1', unknown> & Record<'prop2', unknown>
 >(obj);
 }
 }
}

```

#### 20.1.4 示例：通过交集组合两个对象类型

在下一个示例中，我们将参数的类型`Obj`与类型`WithKey`组合在一起——通过向参数添加`WithKey`的属性`.key`：

```ts
type WithKey = {
 key: string,
};
function addKey<Obj extends object>(obj: Obj, key: string)
 : Obj & WithKey
{
 const objWithKey = obj as (Obj & WithKey);
 objWithKey.key = key;
 return objWithKey;
}

```

`addKey()`的使用方式如下：

```ts
const paris = {
 city: 'Paris',
};

const parisWithKey = addKey(paris, 'paris');
assertType<
{ 
 city: string,
 key: string,
}
>(parisWithKey);

```
