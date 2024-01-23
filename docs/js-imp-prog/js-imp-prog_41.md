# 三十四、WeakMaps (WeakMap) (高级)

> 原文：[`exploringjs.com/impatient-js/ch_weakmaps.html`](https://exploringjs.com/impatient-js/ch_weakmaps.html)

* * *

+   34.1 WeakMaps 是黑盒

+   34.2 WeakMap 的键是*弱引用*

    +   34.2.1 所有 WeakMap 的键必须是对象

    +   34.2.2 用例：将值附加到对象

+   34.3 示例

    +   34.3.1 通过 WeakMaps 缓存计算结果

    +   34.3.2 在 WeakMaps 中保持私有数据

+   34.4 WeakMap API

* * *

WeakMaps 类似于 Maps，但有以下区别：

+   它们是黑盒，只有拥有 WeakMap 和键的人才能访问值。

+   WeakMap 的键是*弱引用*：如果一个对象是 WeakMap 的键，它仍然可以被垃圾回收。这让我们可以使用 WeakMap 来附加数据到对象上。

接下来的两节将更详细地讨论这意味着什么。

### 34.1 WeakMaps 是黑盒

不可能检查 WeakMap 内部的内容：

+   例如，你不能迭代或循环遍历键、值或条目。你也不能计算大小。

+   此外，你也不能清除 WeakMap - 你必须创建一个新的实例。

这些限制实现了一个安全属性。引用[Mark Miller](https://github.com/tc39/tc39-notes/blob/master/meetings/2014-11/nov-19.md#412-should-weakmapweakset-have-a-clear-method-markm)：

> WeakMap/键对值的映射只能被拥有 WeakMap 和键的人观察或影响。使用`clear()`，只拥有 WeakMap 的人将能够影响 WeakMap 和键到值的映射。

### 34.2 WeakMap 的键是*弱引用*

WeakMap 的键被称为*弱引用*：通常，如果一个对象引用另一个对象，那么只要前者存在，后者就不能被垃圾回收。但是对于 WeakMap 来说，情况不同：如果一个对象是键，而且没有其他地方引用它，那么它可以在 WeakMap 仍然存在的情况下被垃圾回收。这也导致相应的条目被移除（但没有办法观察到）。

#### 34.2.1 所有 WeakMap 的键必须是对象

所有 WeakMap 的键必须是对象。如果使用原始值，会报错：

```js
> const wm = new WeakMap();
> wm.set(123, 'test')
TypeError: Invalid value used as weak map key
```

如果使用原始值作为键，WeakMaps 将不再是黑盒。但是鉴于原始值永远不会被垃圾回收，你也不会从弱引用键中获益，可以使用普通的 Map。

#### 34.2.2 用例：将值附加到对象

这是 WeakMaps 的主要用例：你可以使用它们来外部附加值到对象上 - 例如：

```js
const wm = new WeakMap();
{
 const obj = {};
 wm.set(obj, 'attachedValue'); // (A)
}
// (B)
```

在 A 行，我们将一个值附加到`obj`。在 B 行，`obj`可能已经被垃圾回收，尽管`wm`仍然存在。这种将值附加到对象的技术等同于将该对象的属性存储在外部。如果`wm`是一个属性，前面的代码将如下所示：

```js
{
 const obj = {};
 obj.wm = 'attachedValue';
}
```

### 34.3 示例

#### 34.3.1 通过 WeakMaps 缓存计算结果

使用 WeakMaps，你可以将先前计算的结果与对象关联起来，而不必担心内存管理。下面的函数`countOwnKeys()`就是一个例子：它在 WeakMap `cache`中缓存先前的结果。

```js
const cache = new WeakMap();
function countOwnKeys(obj) {
 if (cache.has(obj)) {
 return [cache.get(obj), 'cached'];
 } else {
 const count = Object.keys(obj).length;
 cache.set(obj, count);
 return [count, 'computed'];
 }
}
```

如果我们使用这个函数和一个对象`obj`，你会发现结果只在第一次调用时计算，而第二次调用时使用了缓存值：

```js
> const obj = { foo: 1, bar: 2};
> countOwnKeys(obj)
[2, 'computed']
> countOwnKeys(obj)
[2, 'cached']
```

#### 34.3.2 在 WeakMaps 中保持私有数据

在下面的代码中，WeakMaps `_counter`和`_action`用于存储`Countdown`实例的虚拟属性的值：

```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
 constructor(counter, action) {
 _counter.set(this, counter);
 _action.set(this, action);
 }
 dec() {
 let counter = _counter.get(this);
 counter--;
 _counter.set(this, counter);
 if (counter === 0) {
 _action.get(this)();
 }
 }
}

// The two pseudo-properties are truly private:
assert.deepEqual(
 Object.keys(new Countdown()),
 []);
```

这是`Countdown`的使用方式：

```js
let invoked = false;

const cd = new Countdown(3, () => invoked = true);

cd.dec(); assert.equal(invoked, false);
cd.dec(); assert.equal(invoked, false);
cd.dec(); assert.equal(invoked, true);
```

![](img/90f73f1851c5b1baf43cb746913c09e6.png)  **练习：WeakMaps 用于私有数据**

`exercises/weakmaps/weakmaps_private_data_test.mjs`

### 34.4 WeakMap API

`WeakMap`的构造函数和四个方法与它们的`Map`等价物一样工作： 

+   `new WeakMap<K, V>(entries?: Iterable<[K, V]>)` ^([ES6])

+   `.delete(key: K) : boolean` ^([ES6])

+   `.get(key: K) : V` ^([ES6])

+   `.has(key: K) : boolean` ^([ES6])

+   `.set(key: K, value: V) : this` ^([ES6])

![](img/4ca05ad97a693bee61e4fd6459232e60.png)  **测验**

查看测验应用。

[评论](https://github.com/rauschma/impatient-js/issues/36)
