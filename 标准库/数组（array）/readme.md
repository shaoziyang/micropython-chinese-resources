# 数组（array）

MicroPython的array模块是标准python的array模块简化版本，它提供了数组基本的定义方法和使用方法。

* `class` array.**array**(typecode [ , iterable ] )

  创建指定类型的数组，并用给定的iterable参数初始化。如果没有给出参数，将创建一个空数组。  
  
  MicroPython 的 typecode（类型码）：
  
  | 类型码 | C 语言类型 | Python 类型 | 最小大小 |
  | --- | --- | --- | --- |
  | 'b' | 有符号字符 | int | 1   |
  | 'B' | 无符号字符 | int | 1   |
  | 'u' | Py\_UNICODE | Unicode 字符 | 2   |
  | 'h' | 有符号短整型 | int | 2   |
  | 'H' | 无符号短整型 | int | 2   |
  | 'i' | 有符号整数 | int | 2   |
  | 'I' | 无符号整数 | int | 2   |
  | 'l' | 有符号长整型 | int | 4   |
  | 'L' | 无符号长整型 | int | 4   |
  | 'q' | 有符号扩展整型 | int | 8   |
  | 'Q' | 无符号扩展整型 | int | 8   |
  | 'f' | 单精度浮点 | float | 4   |
  | 'd' | 双精度浮点 | float | 8   |
  
  MicroPython的array模块目前提供了下面几个方法：  
<br>

* `append`(val)

  将新数据添加到数组末尾。  
<br>

* `extend`(iterable)

  将一个数组添加到数组末尾。

  例如：
  ```py
  import array
  
  a = array.array('i')
  a.append(1)
  b = array.array('I')
  b = array.array('I', [1, 2, 3])
  b.extend(a)
  ```


* `__getitem__`(index)

  通过 `a[index]`方式读取数组的索引项（其中a是数组）。如果index是整数，则返回值；如果index是切片，则返回数组。负索引从末尾开始计数，如果索引超出范围，则引发IndexError异常。

  **注意**：`__getitem__`不能被直接调用（`a.__getitem__(index)`是错误使用方式），并且不存在于`__dict__`中，但是可以用\[index\]。  
<br>

* `__setitem__`(index, value)

  以`a[index]=value`方式写入数组（其中a是数组）。若index是整数，则value是单个值；若index是切片，则value为数组。负索引从末尾开始计数，如果索引超出范围，则引发`IndexError`异常。

  **注意**：不能直接调用`__setitem__`（`a.__setitem__(index, value)`是错误使用方式），也不存在于`__dict__`中，但是可以用  `a[index] = value`。  
<br>

* `__len__`()

  返回数组中项数。

  **注意**：`__len__`不能被直接调用（`a.__len__()`是错误使用方式），并且该方法不存在于`__dict__`，但可以用 `len(a)`。  
<br>

* `__add__`(other)

  返回一个与other联合的新数组，称为`a + other`（其中a和other都是数组）。

  **注意**：`__add__`不能被直接调用（`a.__add__(other)` 是错误方式），并且不存在于`__dict__`中，但是可以用 `a+other`。
<br><br>

* `__iadd__`(other)

  将数组与other数组连接在一起，即 `a += other`（其中a和other都是数组）。等效于 `extend(other)`。

  **注意**：`__iadd__`不能被直接调用（`a.__iadd__(other)` 是错误方式），并且不存在于`__dict__`，然而可以使用 `a += other`。
<br><br>

* `__repr__`()

  返回数组的字符串表示形式，即 str(a) 或 repr(a)（其中a是数组）。返回字符串"array(<type\>, [<elements\>\])"，其中<type\>是数组的类型代码字母，<elements\>是数组元素的逗号分隔列表。

  **注意**：`__repr__`不能被直接调用（`a.__repr__()`是错误方式），并且不存在于`__dict__`中，但是可以使用 str(a) 和 repr(a)。
