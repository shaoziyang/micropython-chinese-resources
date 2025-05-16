# heapq（堆队列算法）

该模块实现了[最小堆队列算法](https://en.wikipedia.org/wiki/Heap_%28data_structure%29)。堆队列本质上是一个列表，其元素的存储方式使得列表的第一项总是最小的。

## 函数

* heapq.`heappush`(heap, item)

  将元素推入堆中。
<br><br>

* heapq.`heappop`(heap)

  弹出并返回堆中的第一项元素。如果堆是空的将引起 IndexError 异常。返回的值将是堆中最小的项。
<br><br>

* heapq.`heapify`(x)

  将列表转换成堆。


## 使用方法

heapq的基本使用方法如下：

```py
>>> import heapq
>>> buf=[3, 1, 5]
>>> heapq.heappush(buf, 4)
>>> buf
[3, 1, 5, 4]
>>> heapq.heappop(buf)
3
>>> buf
[1, 4, 5]
>>> heapq.heappop(buf)
1
>>> buf
[4, 5]
>>> heapq.heappop(buf)
4
>>> buf
[5]
>>> heapq.heappop(buf)
5
>>> heapq.heappop(buf)               # heap是空的，将引起异常
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: empty heap
```

