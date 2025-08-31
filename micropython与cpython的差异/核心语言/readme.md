# 核心语言

## 类

### 未为用户自定义类实现特殊方法 `__del__`

示例代码：

```py
import gc
class Foo:
    def __del__(self):
        print("__del__")
        
f = Foo()
del f

gc.collect()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
| `__del__` | |


### 方法解析顺序（MRO）与 CPython 不兼容

**原因**：采用深度优先但非穷举的方法解析顺序

**解决方法**：避免使用具有多重继承和复杂方法重写的复杂类层次结构。请记住，许多语言根本不支持多重继承。

示例代码：
```py
class Foo:
    def __str__(self):
        return "Foo"

class C(tuple, Foo):
    pass

t = C((1, 2, 3))
print(t)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
| Foo | (1, 2, 3) |


### 私有类成员的名称修饰未实现

**原因**：MicroPython 编译器未对私有类成员实现名称修饰功能。

**解决方法**：通过手动为私有类成员名称添加唯一前缀，避免使用全局名称或与全局名称发生冲突。

示例代码：
```py
def __print_string(string):
    print(string)

class Foo:
    def __init__(self, string):
        self.string = string

    def do_print(self):
        __print_string(self.string)

example_string = "Example String to print."

class_item = Foo(example_string)
print(class_item.string)

class_item.do_print()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Example String to print.<br>Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 26, in \<module\><br>&nbsp;&nbsp;File "\<stdin\>", line 18, in do_print<br>`NameError`: name '_Foo__print_string' is not defined. Did you mean: '__print_string'? |Example String to print.<br>Example String to print.|


### 当继承原生类型时，在 `super().__init__()` 之前如果在 `__init__(self, ...)` 中调用方法会引发 `AttributeError`（如果未启用 `MICROPY_BUILTIN_METHOD_CHECK_SELF_ARG`，则会导致段错误）。

**原因**：MicroPython 中的原生类型没有单独的 `__new__` 和 `__init__` 方法。

**解决方法**：先调用 `super().__init__()`。

示例代码：
```python
class L1(list):
    def __init__(self, a):
        self.append(a)

try:
    L1(1)
    print("OK")
except AttributeError:
    print("AttributeError")

class L2(list):
    def __init__(self, a):
        super().__init__()
        self.append(a)

try:
    L2(1)
    print("OK")
except AttributeError:
    print("AttributeError")
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|OK<br>OK|`AttributeError`<br>OK|


### 从多个类继承时，`super()` 仅调用一个类

**原因**：参见“方法解析顺序（MRO）与 CPython 不兼容”

**解决方法**：参见“方法解析顺序（MRO）与 CPython 不兼容”

示例代码：

```python
class A:
    def __init__(self):
        print("A.__init__")

class B(A):
    def __init__(self):
        print("B.__init__")
        super().__init__()

class C(A):
    def __init__(self):
        print("C.__init__")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D.__init__")
        super().__init__()

D()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|D.`__init__`<br>B.`__init__`<br>C.`__init__`<br>A.`__init__`|D.`__init__`<br>B.`__init__`<br>A.`__init__`|


### 在子类中调用 super() 获取器属性会返回一个属性对象，而非值

示例代码：
```python
class A:
    @property
    def p(self):
        return {"a": 10}

class AA(A):
    @property
    def p(self):
        return super().p

a = AA()
print(a.p)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|{'a': 10}|`<property>`|


## 函数

### 方法的错误消息可能会显示意外的参数数量

**原因**：MicroPython 将“self”算作一个参数。

**解决方法**：理解错误消息时请记住上述信息。

示例代码：
```python
try:
    [].append()
except Exception as e:
    print(e)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|list.append() takes exactly one argument (0 given)|function takes 2 positional arguments but 1 were given|


### 函数对象没有 `__module__` 属性

**原因**：MicroPython 经过优化，以减小代码大小和内存使用量。

**解决方法**：对于非内置模块，使用 `sys.modules[function.__globals__['__name__']]`。

示例代码：

```python
def f():
    pass

print(f.__module__)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`__main__`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 13, in \<module\><br>`AttributeError:`: 'function' object has no attribute `'__module__'`|


### 不支持为函数定义用户自定义属性

**原因**：MicroPython 针对内存使用进行了高度优化。

**解决方法**：使用外部字典，例如 `FUNC_X[f] = 0`。

示例代码：
```python
def f():
    pass

f.x = 0
print(f.x)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|0|Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 13, in \<module\><br>`AttributeError`: 'function' object has no attribute 'x'|


## 生成器

### 在未运行至完成的生成器中，上下文管理器的 `__exit__()` 方法不会被调用

示例代码：
```python
class foo(object):
    def __enter__(self):
        print("Enter")

    def __exit__(self, *args):
        print("Exit")

def bar(x):
    with foo():
        while True:
            x += 1
            yield x

def func():
    g = bar(0)
    for _ in range(3):
        print(next(g))

func()
```


| CPython 输出：| MicroPython 输出：|
| - | - |
|Enter<br>1<br>2<br>3<br>Exit|Enter<br>1<br>2<br>3|


## 运行时

### 局部变量不包含在 `locals()` 的结果中

**原因**：MicroPython 不维护符号化的局部环境，它被优化为一个槽数组。因此，无法通过名称访问局部变量。

示例代码：
```python
def test():
    val = 2
    print(locals())

test()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|{'val': 2}|{'test': <function test at 0x73a861806260>, `'__name__'`: `'__main__'`, `'__file__'`: '\<stdin\>'}|


### 在 `eval()` 函数中运行的代码无法访问局部变量

**原因**：MicroPython 不维护符号化的局部环境，它被优化为一个槽数组。因此，无法通过名称访问局部变量。实际上，MicroPython 中的 `eval(expr)` 等价于 `eval(expr, globals(), globals())`。

示例代码：
```python
val = 1

def test():
    val = 2
    print(val)
    eval("print(val)")

test()
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|2<br>2|2<br>1|


## f-strings

### 若相邻字面量包含花括号，f 字符串不支持与这些相邻字面量进行拼接

**原因**：MicroPython 针对代码空间进行了优化。

**解决方法**：当相邻字面量并非均为 f 字符串时，在字面量字符串之间使用 `+` 运算符进行拼接

示例代码：

```python
x, y = 1, 2
print("aa" f"{x}")    # 正常运行
print(f"{x}" "ab")    # 正常运行
print("a{}a" f"{x}")  # 运行失败
print(f"{x}" "a{}b")  # 运行失败
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|aa1<br>1ab<br>a{}a1<br>1a{}b|aa1<br>1ab<br>Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 12, in \<module\><br>`IndexError`: tuple index out of range|


### f 字符串不支持需要通过解析来处理不平衡嵌套花括号和方括号的表达式

**原因**：MicroPython 针对代码空间进行了优化。

**解决方法**：在 f字符串内部的表达式中，始终使用成对（平衡）的花括号和方括号

示例代码：
```python
print(f"{'hello { world'}")
print(f"{'hello ] world'}")
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|hello { world<br>hello ] world|Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 9<br>`SyntaxError`: invalid syntax|


### f 字符串不支持 !a 转换

**原因**：MicroPython 未实现 `ascii()` 函数

**解决方法**：无

示例代码：
```python
f"{'unicode text'!a}"
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|`"'unicode text'"`|Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 8<br>`SyntaxError`: invalid syntax|


## 导入（import）

### 在 MicroPython 中，包的 `__path__` 属性类型与 CPython 不同（MicroPython 中为单个字符串，而非字符串列表）。

**原因**：MicroPython 不支持跨文件系统拆分的命名空间包；此外，MicroPython 的导入系统经过高度优化，以实现最小化的内存占用。

**解决方法**：导入处理的细节本质上依赖于具体实现，在可移植应用中不要依赖此类细节。

示例代码：
```python
import modules

print(modules.__path__)
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|['/home/micropython/micropython-autodocs/tests/cpydiff/modules']|../tests/cpydiff/modules|


### MicroPython 不支持跨文件系统拆分的命名空间包

**原因**：MicroPython 的导入系统经过高度优化，旨在实现简洁性、最小化内存占用以及最低化文件系统搜索开销。

**解决方法**：不要将属于同一命名空间包的模块安装在不同目录中。对于 MicroPython，建议模块搜索路径最多包含 3 个组成部分：当前应用程序路径、用户专属路径（可写入）、系统级路径（不可写入）。

示例代码：
```python
import sys

sys.path.append(sys.path[1] + "/modules")
sys.path.append(sys.path[1] + "/modules2")

import subpkg.foo
import subpkg.bar

print("Two modules of a split namespace package imported")
```

| CPython 输出：| MicroPython 输出：|
| - | - |
|Two modules of a split namespace package imported|Traceback (most recent call last):<br>&nbsp;&nbsp;File "\<stdin\>", line 14, in \<module\><br>`ImportError`: no module named 'subpkg.bar'|

