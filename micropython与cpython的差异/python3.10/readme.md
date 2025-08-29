# Python 3.10

Python 3.10.0（最终版）于2021年10月4日发布。3.10版本的特性在 [PEP 619](https://www.python.org/dev/peps/pep-0619/#features-for-3-10) 中定义，有关变更的详细说明可参见 [Python 3.10 新特性](https://docs.python.org/3/whatsnew/3.10.html)。

**特性**

| PEP编号 | 描述 | 状态 |
| --- | --- | --- |
| **新语法特性** |||
| [PEP 634](https://www.python.org/dev/peps/pep-0634/) | 结构化模式匹配：规范 | |
| [PEP 635](https://www.python.org/dev/peps/pep-0635/) | 结构化模式匹配：动机和基本原理 | |
| [PEP 636](https://www.python.org/dev/peps/pep-0636/) | 结构化模式匹配：教程 | |
| [bpo-12782](https://github.com/python/cpython/issues/56991) | 带括号的上下文管理器现在正式被允许 | |
| **标准库中的新特性** |||
| [PEP 618](https://www.python.org/dev/peps/pep-0618/) | 为zip添加可选的长度检查 | |
| **解释器改进** |||
| [PEP 626](https://www.python.org/dev/peps/pep-0626/) | 用于调试和其他工具的精确行号 ||
| **新的类型特性** |||
| [PEP 604](https://www.python.org/dev/peps/pep-0604/) | 允许将联合类型写为X \| Y | |
| [PEP 613](https://www.python.org/dev/peps/pep-0613/) | 显式类型别名 | |
| [PEP 612](https://www.python.org/dev/peps/pep-0612/) | 参数规范变量 | |
| **重要的弃用、移除或限制** |||
| [PEP 644](https://www.python.org/dev/peps/pep-0644/) | 要求 OpenSSL 1.1.1 或更高版本 | |
| [PEP 632](https://www.python.org/dev/peps/pep-0632/) | 弃用 distutils 模块 | 不相关 |
| [PEP 623](https://www.python.org/dev/peps/pep-0623/) | 弃用并准备移除 PyUnicodeObject 中的 wstr 成员 | 不相关 |
| [PEP 624](https://www.python.org/dev/peps/pep-0624/) | 移除 Py_UNICODE 编码器 API | 不相关 |
| [PEP 597](https://www.python.org/dev/peps/pep-0597/) | 添加可选的 EncodingWarning | |

**其他语言变更：**

| 变更 | 状态 |
| - | - |
| `int` 类型新增 `int.bit_count()` 方法，返回给定整数二进制展开中1的个数，也称为种群计数。||
| [dict.keys()](https://docs.python.org/3.5/library/stdtypes.html#dict.keys)、[dict.values()](https://docs.python.org/3.5/library/stdtypes.html#dict.values) 和 [dict.items()](https://docs.python.org/3.5/library/stdtypes.html#dict.items) 返回的视图现在都有一个 `mapping` 属性，该属性提供一个包装原始字典的 [types.MappingProxyType](https://docs.python.org/3.5/library/types.html#types.MappingProxyType) 对象。||
| [PEP 618](https://peps.python.org/pep-0618/)：`zip()`函数现在有一个可选的 `strict` 标志，用于要求所有可迭代对象具有相同的长度。||
| 接受整数参数的内置函数和扩展函数不再接受 [Decimal](https://docs.python.org/3.5/library/decimal.html#decimal.Decimal)、[Fraction](https://docs.python.org/3.5/library/fractions.html#fractions.Fraction) 以及其他只能通过损失精度转换为整数的对象（例如，具有[\_\_int\_\_()](https://docs.python.org/3.5/reference/datamodel.html#object.__int__)方法但没有[\_\_index\_\_()](https://docs.python.org/3.5/reference/datamodel.html#object.__index__)方法的对象）。||
| 如果`object.__ipow__()`返回`NotImplemented`，该运算符将如预期的那样正确回退到`object.__pow__()`和`object.__rpow__()`。||
| 赋值表达式现在可以在集合字面量和集合推导式中不加括号使用，也可以在序列索引（但不是切片）中使用。||
| 函数有一个新的`__builtins__`属性，用于在执行函数时查找内置符号，而不是查看`__globals__['__builtins__']`。该属性从`__globals__["__builtins__"]`（如果存在）初始化，否则从当前内置函数初始化。||
| 新增了两个内置函数: `aiter()` 和 `anext()`，分别为 `iter()` 和 `next()` 提供异步对应。||
| 静态方法（`@staticmethod`）和类方法（`@classmethod`）现在继承方法属性（`__module__`、`__name__`、`__qualname__`、`__doc__`、`__annotations__`），并具有新的`__wrapped__`属性。此外，静态方法现在可以作为常规函数调用。||
| 对于复杂目标（除 [PEP 526](https://peps.python.org/pep-0526/) 定义的简单名称目标之外的所有目标）的注解，在使用`from __future__ import annotations`时不再产生任何运行时影响。||
| 类和模块对象现在会根据需要延迟创建空的注解字典。为了向后兼容，注解字典存储在对象的`__dict__`中。这改进了处理`__annotations__`的最佳实践。||
| 由于副作用，在`from __future__ import annotations`下，包含`yield`、`yield from`、`await`或命名表达式的注解现在被禁止。||
| 在`from __future__ import annotations`下，未绑定变量、`super()`以及其他可能改变符号表处理的表达式作为注解使用时，现在不会产生效果。||
| `float`类型和 [decimal.Decimal](https://docs.python.org/3.5/library/decimal.html#decimal.Decimal) 类型的NaN值的哈希现在取决于对象标识。以前，它们总是哈希为0，尽管NaN值彼此不相等。这在创建包含多个NaN的字典和集合时，由于过多的哈希冲突，可能导致潜在的二次运行时行为。||
| 删除`__debug__`常量时将引发`SyntaxError`（而非NameError）。||
| `SyntaxError`异常现在具有`end_lineno`和`end_offset`属性。如果未确定，它们将为`None`。||

**内置模块的变更：**

| 变更 | 状态 |
| - | - |
| **[asyncio](https://docs.python.org/3/whatsnew/3.10.html#asyncio)** ||
| 添加缺失的`connect_accepted_socket()`方法。||
| **[array](https://docs.python.org/3/whatsnew/3.10.html#array)** ||
| `array.array`的 [index()](https://docs.python.org/3.5/library/array.html#array.array.index) 方法现在有可选的 *start* 和 *stop* 参数。||
| **[gc](https://docs.python.org/3/whatsnew/3.10.html#gc)** ||
| 为 [gc.get_objects()](https://docs.python.org/3.5/library/gc.html#gc.get_objects)、[gc.get_referrers()](https://docs.python.org/3.5/library/gc.html#gc.get_referrers) 和 [gc.get_referents()](https://docs.python.org/3.5/library/gc.html#gc.get_referents) 添加审计钩子。 ||
| **[hashlib](https://docs.python.org/3/whatsnew/3.10.html#hashlib)** ||
| hashlib模块要求OpenSSL 1.1.1或更高版本。||
| hashlib模块初步支持OpenSSL 3.0.0。||
| [pbkdf2_hmac()](https://docs.python.org/3.5/library/hashlib.html#hashlib.pbkdf2_hmac) 的纯 Python 回退已被弃用。将来，只有当 Python 是使用 OpenSSL 支持构建时，PBKDF2-HMAC 才可用。||
| **[os](https://docs.python.org/3/whatsnew/3.10.html#os)** ||
| 为 VxWorks RTOS 添加 [os.cpu_count()](https://docs.python.org/3.5/library/os.html#os.cpu_count) 支持。 ||
| 添加新函数 `os.eventfd()` 和相关辅助函数，以包装 Linux 上的 eventfd2 系统调用。||
| 添加`os.splice()`，允许在两个文件描述符之间移动数据，而无需在内核地址空间和用户地址空间之间复制，其中一个文件描述符必须指向管道。||
| 为 macOS 添加 `O_EVTONLY`、`O_FSYNC`、`O_SYMLINK` 和 `O_NOFOLLOW_ANY`。||
| **[platform](https://docs.python.org/3/whatsnew/3.10.html#platform)** ||
| 添加 `platform.freedesktop_os_release()`，用于从 [freedesktop.org/os-release](https://www.freedesktop.org/software/systemd/man/os-release.html) 标准文件检索操作系统标识。||
| **[socket](https://docs.python.org/3/whatsnew/3.10.html#socket)** ||
| [socket.timeout](https://docs.python.org/3.5/library/socket.html#socket.timeout) 异常现在是 `TimeoutError` 的别名。||
| 添加使用 `IPPROTO_MPTCP` 创建 MPTCP 套接字的选项。||
| 添加 `IP_RECVTOS` 选项，用于接收服务类型（ToS）或 DSCP/ECN 字段。||
| **[ssl](https://docs.python.org/3/whatsnew/3.10.html#ssl)** ||
| ssl 模块要求 OpenSSL 1.1.1 或更高版本。||
| ssl 模块初步支持 OpenSSL 3.0.0 和新选项 `OP_IGNORE_UNEXPECTED_EOF`。 ||
| 已弃用的函数和已弃用常量的使用现在会导致 [DeprecationWarning](https://docs.python.org/3.5/library/exceptions.html#DeprecationWarning)。[ssl.SSLContext.options](https://docs.python.org/3.5/library/ssl.html#ssl.SSLContext.options) 默认设置了 [OP_NO_SSLv2](https://docs.python.org/3.5/library/ssl.html#ssl.OP_NO_SSLv2) 和 [OP_NO_SSLv3](https://docs.python.org/3.5/library/ssl.html#ssl.OP_NO_SSLv3)，因此不会对再次设置这些标志发出警告。||
| ssl 模块现在有更安全的默认设置。默认情况下，禁用没有前向 secrecy  或SHA-1 MAC 的密码套件。安全级别 2 禁止安全性低于 112 位的弱 RSA、DH 和 ECC 密钥。`SSLContext` 默认为最低协议版本 TLS 1.2。设置基于 Hynek Schlawack 的研究。||
| 已弃用的协议 SSL 3.0、TLS 1.0 和 TLS 1.1 不再被官方支持。Python 不会主动阻止它们。但是，OpenSSL 构建选项、发行版配置、供应商补丁和密码套件可能会阻止成功握手。||
| 为 [ssl.get_server_certificate()](https://docs.python.org/3.5/library/ssl.html#ssl.get_server_certificate) 函数添加 *timeout* 参数。||
| ssl 模块使用堆类型和多阶段初始化。||
| 新增验证标志 VERIFY_X509_PARTIAL_CHAIN。||
| **[sys](https://docs.python.org/3/whatsnew/3.10.html#sys)** ||
| 添加 `sys.orig_argv` 属性：传递给 Python 可执行文件的原始命令行参数列表。||
| 添加 `sys.stdlib_module_names`，包含标准库模块名称列表。||
| **[_thread](https://docs.python.org/3/whatsnew/3.10.html#_thread)** ||
| [_thread.interrupt_main()](https://docs.python.org/3.5/library/_thread.html#_thread.interrupt_main) 现在接受一个可选的要模拟的信号编号（默认仍然是 signal.SIGINT）。||

**注意事项**

对于MicroPython所实现的Python特性，其行为有时与标准Python存在差异。以下各节中列出的操作在 MicroPython 和标准 Python 中会产生不一致的结果。
