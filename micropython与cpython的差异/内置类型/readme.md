# 内置类型（Builtin types）

## 异常（Exception）

### 在 MicroPython 中，所有异常都具有可读的 `value` 属性和 `errno` 属性，而非仅 `StopIteration`（停止迭代异常）和 `OSError`（操作系统错误异常）才有。

**原因**：MicroPython 为减小代码体积进行了优化。

**解决方法**：仅对 `StopIteration` 异常使用 `value` 属性，对 `OSError` 异常使用 `errno` 属性。不要在其他异常上使用或依赖这两个属性。

示例代码：

```python
e = Exception(1)
print(e.value)
print(e.errno)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`AttributeError`: 'Exception' object has no attribute 'value'|1<br>1|


### 未实现异常链（Exception Chaining）

示例代码：
```python
try:
    raise TypeError
except TypeError:
    raise ValueError
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>` `TypeError`<br><br>During handling of the above exception, another exception occurred:<br><br>Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 11, in `<module>`<br>`ValueError`|Traceback (most recent call last):&nbsp;&nbsp;File "`<stdin>`", line 11, in `<module>`<br>`ValueError`|


### 不支持为内置异常定义用户自定义属性

**原因**：MicroPython 针对内存使用进行了高度优化。

**解决方法**：使用用户自定义的异常子类。

示例代码：
```python
e = Exception()
e.x = 0
print(e.x)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|0|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`AttributeError`: 'Exception' object has no attribute 'x'|


### while 循环条件中的异常可能显示意外的行号

**原因**：条件检查经过优化，会在循环体末尾执行，且报错时会显示该（循环体末尾的）行号。

示例代码：
```python
l = ["-foo", "-bar"]

i = 0
while l[i][0] == "-":
    print("iter")
    i += 1
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|iter<br>iter<br>Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 11, in `<module>`<br>`IndexError`: list index out of range|iter<br>iter<br>Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 13, in `<module>`<br>`IndexError`: list index out of range|


### `Exception.__init__` 方法不存在

**原因**：MicroPython 未完全支持原生类的子类化。

**解决方法**：改用 `super()` 调用：

```python
class A(Exception):
    def __init__(self):
        super().__init__()
```

示例代码：
```python
class A(Exception):
    def __init__(self):
        Exception.__init__(self)

a = A()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
| |Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 18, in `<module>`<br>&nbsp;&nbsp; File "`<stdin>`", line 15, in `__init__`<br>`AttributeError`: type object 'Exception' has no attribute '`__init__`'|


## 字节数组（bytearray）

### 右侧操作数（RHS）不支持的数组切片赋值

示例代码：
```python
b = bytearray(4)
b[0:1] = [1, 2]
print(b)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`bytearray(b'\x01\x02\x00\x00\x00')`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`NotImplementedError`: array/bytes required on right side|


## 字节（bytes）

### bytes 对象支持 `.format()` 方法  

**原因**：MicroPython 致力于成为更规整的实现版本。既然 `str`（字符串）和 `bytes` 都支持 `__mod__()`（即 `%` 运算符），那么为两者都支持 `format()` 方法也符合逻辑。此外，`__mod__` 的支持也可通过编译移除，移除后 `bytes` 格式化就仅剩 `format()` 这一种方式。  

**解决方法**：若需保证与 CPython 的兼容性，不要在 `bytes` 对象上使用 `.format()` 方法。  

示例代码：
```python
print(b"{}".format(1))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`AttributeError`: 'bytes' object has no attribute 'format'|`b'1'`|


### 未实现带关键字参数的 bytes() 函数

**解决方法**：将编码（encoding）作为位置参数传入，例如 `print(bytes('abc', 'utf-8'))`

示例代码：
```python
print(bytes("abc", encoding="utf8"))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`b'abc'`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: keyword argument(s) not implemented - use normal args instead|


### 未实现步长不等于 1 的字节（bytes）切片取值

**原因**：MicroPython 针对内存使用进行了高度优化。

**解决方法**：对于这种极少见的操作，使用显式循环实现。

示例代码：
```python
print(b"123"[0:3:2])
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`b'13'`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: only slices with step=1 (aka None) are supported|


## 复数（complex）

### MicroPython 的 `complex()` 函数会接受某些 CPython 会拒绝的非法值

**原因**：MicroPython 针对内存使用进行了高度优化。

**解决方法**：不要将非标准复数字面量用作 `complex()` 函数的参数。

MicroPython 的 `complex()` 函数会接受“实部与虚部之间包含空格但无正负号”的字面量，并将该空格解析为加号（+）。

示例代码：
```python
try:
    print(complex("1 1j"))
except ValueError:
    print("ValueError")
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`ValueError`|`(1+1j)`|


## 字典（dict）

### 字典的键视图（keys view）不具备集合（set）的行为特性。

**原因**：未实现该功能。

**解决方法**：在使用集合操作前，先将键（keys）显式转换为集合（set）类型。

示例代码：
```python
print({1: 2, 3: 4}.keys() & {1})
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`{1}`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`TypeError`: unsupported types for `__and__`: 'dict_view', 'set'|


## 浮点数（float）

### MicroPython 允许在数学运算中对对象进行隐式转换，而 CPython 不允许此操作。

**解决方法**：为确保与 CPython 的兼容性，应使用 `float(obj)` 对对象进行显式包装（即显式转换为浮点数类型）。

示例代码：
```python
class Test:
    def __float__(self):
        return 0.5

print(2.0 * Test())
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 14, in `<module>`<br>`TypeError`: unsupported operand type(s) for *: 'float' and 'Test'|1.0|


## 整数（int）

### `bit_length` 方法不存在。

**原因**：未实现 `bit_length` 方法。

**解决方法**：在 MicroPython 中避免使用该方法。

示例代码：
```python
x = 255
print("{} is {} bits long.".format(x, x.bit_length()))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|255 is 8 bits long.|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`AttributeError`: 'int' object has no attribute 'bit_length'|


### 不支持对整数衍生类型进行整数转换

**解决方法**：除非确有必要，否则避免继承内置类型。建议优先使用“组合优于继承”（https://en.wikipedia.org/wiki/Composition_over_inheritance）的设计原则。

示例代码：
```python
class A(int):
    __add__ = lambda self, other: A(int(self) + other)

a = A(42)
print(a + a)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|84|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 14, in `<module>`<br>&nbsp;&nbsp;File "`<stdin>`", line 10, in \<lambda\><br>`TypeError`: unsupported types for `__radd__`: 'int', 'int'|


### `to_bytes` 方法未实现 `signed` 参数。

**原因**：`int.to_bytes()` 方法未实现关键字参数 `signed`。

当整数为负数时，MicroPython 的行为与 CPython 中 `int.to_bytes(..., signed=True)` 的行为一致；

当整数为非负数时，MicroPython 的行为与 CPython 中 `int.to_bytes(..., signed=False)` 的行为一致。

（两者差异较为细微：在 CPython 中，若使用 `signed=True` 转换正整数，为容纳表示非负的 0 符号位，输出结果的字节长度可能需要多 1 个字节。）  

**解决方法**：当对可能为负数的整数值调用 `to_bytes()` 方法时，需格外注意。  

示例代码：
```python
x = -1
print(x.to_bytes(1, "big"))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 16, in `<module>`<br>`OverflowError`: can't convert negative int to unsigned|`b'\xff'`|


## 列表（list）

### 未实现步长不等于1的列表删除操作

**解决方法**：对于这种少见的操作，使用显式循环实现。

示例代码：
```python
l = [1, 2, 3, 4]
del l[0:4:2]
print(l)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`[2, 4]`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`NotImplementedError`:|


### 未实现右侧为非可迭代对象的列表切片赋值

**原因**：右侧操作数（RHS）被限制为元组（tuple）或列表（list）类型。

**解决方法**：对右侧操作数使用 `list(<可迭代对象>)`，将其转换为列表类型。

示例代码：
```python
l = [10, 20]
l[0:1] = range(4)
print(l)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`[0, 1, 2, 3, 20]`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`TypeError`: object 'range' isn't a tuple or list|


### 未实现步长不等于1的列表赋值操作

**解决方法**：对于这种少见的操作，使用显式循环实现。

示例代码：
```python
l = [1, 2, 3, 4]
l[0:4:2] = [5, 6]
print(l)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`[5, 2, 6, 4]`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 9, in `<module>`<br>`NotImplementedError`:|


## 内存视图（memoryview）

### 若内存视图（memoryview）的目标对象被调整大小，该内存视图可能会失效

**原因**：CPython 会阻止存在 `memoryview` 引用的 `bytearray` 或 `io.BytesIO` 对象调整大小；而在 MicroPython 中，需要开发者手动确保当有内存视图引用某对象时，该对象不会被调整大小。

在最糟糕的情况下，调整作为内存视图目标对象的大小，可能导致该内存视图（或多个内存视图）引用已释放的无效内存（即“释放后使用”漏洞），进而破坏 MicroPython 运行时环境。

**解决方法**：不要调整任何已被分配内存视图的 `bytearray` 或 `io.BytesIO` 对象的大小。

示例代码：
```python
b = bytearray(b"abcdefg")
m = memoryview(b)
b.extend(b"hijklmnop")
print(b, bytes(m))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 12, in `<module>`<br>`BufferError`: Existing exports of data: object cannot be re-sized|`bytearray(b'abcdefghijklmnop') b'abcdefg'`|


## 字符串（str）

### 与 CPython 不同，MicroPython 允许在任意基数（radix）下使用 "," 分组选项

**原因**：为减小代码体积，MicroPython 不会对这种组合抛出错误

**解决方法**：若需保证与 CPython 的兼容性，不要使用 `{:,b}` 这类格式字符串（注：示例中 `{:,b}` 表示带“,”分组的二进制格式，同理 `{:,x}` 为十六进制、`{:,o}` 为八进制）。

示例代码：
```python
try:
    print("{:,b}".format(99))
except ValueError:
    print("ValueError")
try:
    print("{:,x}".format(99))
except ValueError:
    print("ValueError")
try:
    print("{:,o}".format(99))
except ValueError:
    print("ValueError")
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|ValueError<br>ValueError<br>ValueError|110,0011<br>63<br>143|


### 未实现属性/下标（Attributes/subscr）功能

示例代码：
```python
print("{a[0]}".format(a=[1, 2]))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|1|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: attributes not supported|


### 未实现带关键字参数的 str(…) 函数

**解决方法**：直接传入编码格式。例如 `print(str('abc', 'utf-8'))`

示例代码：
```python
print(str(b"abc", encoding="utf8"))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|abc|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: keyword argument(s) not implemented - use normal args instead|


### 未实现 `str.ljust()` 和 `str.rjust()` 方法

**原因**：MicroPython 针对内存使用进行了高度优化，且存在简单的替代方案。

**解决方法**：不要使用 `s.ljust(10)`，而是使用 `"% -10s" % s`；不要使用 `s.rjust(10)`，而是使用 `"%10s" % s`。此外，也可分别使用格式化字符串 `"{:<10}".format(s)` 或 `"{:>10}".format(s)`。

示例代码：
```python
print("abc".ljust(10))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|abc|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`AttributeError`: 'str' object has no attribute 'ljust'|


### 未实现以 `None` 作为第一个参数的 `rsplit` 方法（如 `str.rsplit(None, n)`）

示例代码：
```python
print("a a a".rsplit(None, 1))
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`['a a', 'a']`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: rsplit(None,n)|


### 未实现步长不等于 1 的字符串下标切片（Subscript）

示例代码：
```python
print("abcdefghi"[0:9:2])
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|acegi|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: only slices with step=1 (aka None) are supported|


## 元组（tuple）

### 未实现步长不等于 1 的元组切片取值

示例代码：
```python
print((1, 2, 3, 4)[0:4:2])
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`(1, 3)`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "`<stdin>`", line 8, in `<module>`<br>`NotImplementedError`: only slices with step=1 (aka None) are supported|

