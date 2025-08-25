# Python 3.7

**新特性：**

| PEP 编号 | 描述 | 状态 |
| --- | --- | --- |
| [PEP 538](https://www.python.org/dev/peps/pep-0538/) | 将遗留的 C 区域设置强制转换为基于 UTF-8 的区域设置 |  |
| [PEP 539](https://www.python.org/dev/peps/pep-0539/) | CPython 中线程本地存储的新 C 语言 API |  |
| [PEP 540](https://www.python.org/dev/peps/pep-0540/) | UTF-8 模式 |  |
| [PEP 552](https://www.python.org/dev/peps/pep-0552/) | 确定性 pyc 文件 |  |
| [PEP 553](https://www.python.org/dev/peps/pep-0553/) | 内置 breakpoint() 函数 |  |
| [PEP 557](https://www.python.org/dev/peps/pep-0557/) | 数据类 |  |
| [PEP 560](https://www.python.org/dev/peps/pep-0560/) | 对 typing 模块和泛型类型的核心支持 |  |
| [PEP 562](https://www.python.org/dev/peps/pep-0562/) | 模块的 `__getattr__` 和 `__dir__` 方法 | 部分支持 |
| [PEP 563](https://www.python.org/dev/peps/pep-0563/) | 注解的延迟求值 |  |
| [PEP 564](https://www.python.org/dev/peps/pep-0564/) | 具有纳秒分辨率的时间函数 | 部分支持 |
| [PEP 565](https://www.python.org/dev/peps/pep-0565/) | 在 `__main__` 中显示 DeprecationWarning 警告 |  |
| [PEP 567](https://www.python.org/dev/peps/pep-0567/) | 上下文变量 |  |


**其他语言变更：**

| 变更 | 状态 |
| - | - |
| `async` 和 `await` 现在是保留关键字。| 完全支持 |
| 字典对象必须保留插入顺序。||
| 现在可以向函数传递超过 255 个参数；函数现在可以拥有超过 255 个参数。||
| `bytes.fromhex()` 和 `bytearray.fromhex()` 现在会忽略所有 ASCII 空白字符，而不仅仅是空格。||
| `str`、`bytes` 和 `bytearray` 新增了对 `isascii()` 方法的支持，该方法可用于测试字符串或字节是否仅包含 ASCII 字符。||
| 当 `from ... import ...` 导入失败时，`ImportError` 现在会显示模块名称和模块的 `__file__` 路径。||
| 现在支持涉及绝对导入且将子模块绑定到某个名称的循环导入。||
| `object.__format__(x, '')` 现在等效于 `str(x)`，而不是 `format(str(self), '')`。||
| 为了更好地支持动态创建堆栈跟踪，现在可以从 Python 代码实例化 `types.TracebackType`，并且 tracebacks 上的 `tb_next` 属性现在可写。||
| 使用 `-m` 开关时，`sys.path[0]` 现在会立即扩展为完整的起始目录路径，而不是保留为空目录（这允许在导入发生时从当前工作目录导入）。||
| 新的 `-X importtime` 选项或 `PYTHONPROFILEIMPORTTIME` 环境变量可用于显示每个模块导入的时间。||


**内置模块的变更：**

| 变更 | 状态 |
| - | - |
|**[asyncio](https://docs.python.org/3/whatsnew/3.7.html#asyncio)**||
| 变更太多，难以一一列举。||
|**[gc](https://docs.python.org/3/whatsnew/3.7.html#gc)**||
| 新特性包括 `gc.freeze()`、`gc.unfreeze()`、`gc.get_freeze_count()`。||
|**[math](https://docs.python.org/3/whatsnew/3.7.html#math)**||
| 新增 `math.remainder()` 函数，用于实现 IEEE 754 风格的余数计算。||
|**[re](https://docs.python.org/3/whatsnew/3.7.html#re)**||
| 多项整理特性，包括更好地支持按空字符串拆分，以及对已编译表达式和匹配对象的复制支持。||
|**[sys](https://docs.python.org/3/whatsnew/3.7.html#sys)**||
| 新增 `sys.breakpointhook()` 函数。新增 `sys.get(/set)_coroutine_origin_tracking_depth()` 函数。||
|**[time](https://docs.python.org/3/whatsnew/3.7.html#time)**||
| 主要是为了支持 PEP564 中的纳秒分辨率而进行的更新，详见上文。||
