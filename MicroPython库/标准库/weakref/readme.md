# weakref（Python对象生命周期管理）

weakref 模块实现了对应 CPython 模块的部分功能，具体如下所述。如需更多信息，请参阅参见原始CPython文档：[weakref](https://docs.python.org/3.5/library/weakref.html#module-weakref)。

weakref 模块支持创建 Python 对象的弱引用。弱引用是一种无法追踪的对象引用堆分配的 Python 对象，因此即使弱引用指向该对象，垃圾回收器仍可回收对它。

可以注册Python回调函数，以便在对象被垃圾回收器回收时调用。这提供了一种安全的对象不再需要时的清理方式。

**可用性**：weakref 模块要求在编译时启用 `MICROPY_PY_WEAKREF`。该功能已在 Unix 覆盖变体和 WebAssembly PyScript 变体中启用。

## 引用对象
ref对象是创建弱引用的最简单方式。

- class weakref.`ref`(object, [callback, ] / )

  返回给定对象的弱引用。
  如果给出了回调并且不是`None`，那么当垃圾回收器回收对象并且弱引用对象仍然存活时，将调用 *callback*。回调函数将传递弱引用对象作为其参数。
	
- ref.`__call__`()

  如果弱引用对象仍然存在，则调用该对象将返回其引用的对象。否则，返回 `None`。

## finalize 对象

finalize 对象是 ref 对象的扩展版本，使用起来更方便，并允许对回调进行更多控制。

- class weakref.`finalize`(object, callback, / , \*args, \*\*kwargs)

  返回对给定对象的弱引用。与 *weakref.ref* 对象不同，finalize 对象在内部保留，在对象被收集之前不会被收集。

  finalize 对象从活动状态开始。当调用 finalize 对象时，无论是显式调用还是收集对象时，它都会转换为死状态。如果调用 `finalize.detach()` 方法，它也会转换为dead。
	
  当垃圾回收器回收对象（或用户代码显式调用finalize对象）并且finalize对象仍处于活动状态时，将调用 *callback*。回调函数将传递以下参数：`callback(*args, **kwargs)`。

- finalize.`__call__`()

  如果 finalize 对象是活跃的，它会转换到死亡状态并返回 `callback(*args,**kwargs)` 的值。否则将返回 None。

- finalize.`alive`()

  只读布尔属性，指示 finalizer 是否处于活动状态。

- finalize.`peek`()

  如果 finalize 对象是活跃的，则返回`(object, callback, args, kwargs)`。否则返回 None。

- finalize.`detach`()

  如果 finalize 对象是活跃的，它会转换到死亡状态并返回 `(object, callback, args, kwargs)`。否则将返回 None。

