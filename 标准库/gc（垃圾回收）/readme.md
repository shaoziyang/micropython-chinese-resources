# 8. gc（垃圾回收）

gc模块提供了垃圾回收功能，可以回收系统运行中内存产生的垃圾。默认情况下，自动回收功能是使能的。

* gc.`enable`()

  允许自动回收垃圾。
<br><br>

* gc.`disable`()

  禁止自动回收，但可以手动进行回收。
<br><br>

* gc.`collect`()

  回收垃圾。
<br><br>

* gc.`mem_alloc`()

  返回已分配的内存数量。
<br><br>

* gc.`mem_free`()

  返回剩余的内存数量。
<br><br>

* gc.`threshold`()

  设置或返回自动回收门限。
<br>
 
**基本用法**：

```
>>> import gc
>>> gc.enable()
>>> gc.mem_alloc()
4144
>>> gc.mem_free()
98048
>>> gc.collect()
>>> gc.mem_free()
99792
```