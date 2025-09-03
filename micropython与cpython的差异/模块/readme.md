# 模块（Modules）

## 仅位置参数（Positional-only Parameters）

为节省代码体积，CPython 中许多支持关键字参数的函数，在 MicroPython 中仅支持位置参数。

MicroPython 采用与 CPython 相同的方式标记仅位置参数：通过插入 `/` 来标记位置参数的结束位置。任何签名以 `/` 结尾的函数，都仅接受位置参数。有关更多详细信息，请参阅 **[PEP 570](https://peps.python.org/pep-0570/)**（Python 增强提案 570，用于规范仅位置参数的语法）。

### 示例（Example）

例如，在 CPython 3.4 中，`socket.socket` 构造函数的签名如下：
```python
socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

然而，MicroPython 文档中记录的该函数签名为：
```python
socket(af=AF_INET, type=SOCK_STREAM, proto=IPPROTO_TCP, /)
```

参数列表末尾的 `/` 表明，在 MicroPython 中这些参数均为**仅位置参数**。以下代码可在 CPython 中正常运行，但在大多数 MicroPython 移植版中无法运行：
```python
import socket
s = socket.socket(type=socket.SOCK_DGRAM)
```

MicroPython 会抛出如下异常：
```python
TypeError: function doesn't take keyword arguments  # 函数不接受关键字参数
```

以下代码可在 CPython 和 MicroPython 中均正常运行（通过位置参数传参）：
```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  # 按位置传入 family 和 type 参数
```


## 数组（array）

### 不支持不同类型码（typecode）之间的比较

**原因**：为控制代码体积。

**解决方法**：比较各个元素。

示例代码：
```python
import array
array.array("b", [1, 2]) == array.array("i", [1, 2])
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|False|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 10, in `<module>`<br>`NotImplementedError`:|


### 未实现溢出检查

**原因**：为减小代码体积、缩短执行时间，MicroPython 采用隐式截断（处理超出类型范围的值）。

**解决方法**：若需保证与 CPython 的兼容性，需显式对值进行掩码（mask）处理（即通过位运算限制值的范围）。

示例代码：
```python
import array
a = array.array("b", [257])
print(a)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 10, in `<module>`<br>`OverflowError`: signed char is greater than maximum|`array('b', [1])`|


### 未实现查找整数（的功能）

示例代码：
```python
import array
print(1 in array.array("B", b"12"))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|False|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 11, in `<module>`<br>`TypeError`: 'array' object doesn't support item deletion|


### 未实现数组删除（功能）

示例代码：
```python
import array
a = array.array("b", (1, 2, 3))
del a[1]
print(a)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`array('b', [1, 3])`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 10, in `<module>`<br>`NotImplementedError`:|


### 未实现步长不等于 1 的数组下标切片（功能）

示例代码：
```python
import array
a = array.array("b", (1, 2, 3))
print(a[3:2:2])
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`array('b')`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 11, in `<module>`<br>`NotImplementedError`: only slices with step=1 (aka None) are supported|

