# Python 3.5

以下是针对 Python 3.5 已最终确定/被接受的 PEP 列表，按其对 MicroPython 的影响进行分组。


| PEP 编号 | 描述  | 状态  |
| --- | --- | --- |
| **语法扩展** | | |
| [PEP 448](https://www.python.org/dev/peps/pep-0448/) | 额外解包泛化 | 部分支持 |
| [PEP 465](https://www.python.org/dev/peps/pep-0465/) | 新的矩阵乘法运算符 | 完全支持 |
| [PEP 492](https://www.python.org/dev/peps/pep-0492/) | 带有 `async` 和 `await` 语法的协程 | 完全支持 |
| **运行时扩展与变更** | | |
| [PEP 461](https://www.python.org/dev/peps/pep-0461/) | 二进制字符串的 % 格式化 | 完全支持 |
| [PEP 475](https://www.python.org/dev/peps/pep-0475/) | 重试因 `EINTR` 失败的系统调用 | 完全支持 |
| [PEP 479](https://www.python.org/dev/peps/pep-0479/) | 更改生成器内部的 `StopIteration` 处理方式 | 完全支持 |
| **标准库变更** | | |
| [PEP 471](https://www.python.org/dev/peps/pep-0471/) | `os.scandir()` |  |
| [PEP 485](https://www.python.org/dev/peps/pep-0485/) | `math.isclose()` — 用于测试近似相等的函数 | 完全支持 |
| **其他变更** | | |
| [PEP 441](https://www.python.org/dev/peps/pep-0441/) | 改进的 Python zip 应用支持 |  |
| [PEP 486](https://www.python.org/dev/peps/pep-0486/) | 让 Python 启动器知晓虚拟环境 | 不相关 |
| [PEP 484](https://www.python.org/dev/peps/pep-0484/) | 类型提示（仅建议性） | 完全支持 |
| [PEP 488](https://www.python.org/dev/peps/pep-0488/) | 取消 PYO 文件 | 不相关 |
| [PEP 489](https://www.python.org/dev/peps/pep-0489/) | 重新设计扩展模块加载方式 |  |

**其他语言变更：**

| 变更 | 状态 |
| - | - |
| 新增了 namereplace 错误处理器。backslashreplace 错误处理器现在可用于解码和转换。||
| 属性文档字符串现在可写。这对 `collections.namedtuple()` 文档字符串尤其有用。||
| 现在支持涉及相对导入的循环导入。||


**新模块：**

- [typing](https://docs.python.org/3/whatsnew/3.5.html#typing)
- [zipzap](https://docs.python.org/3/whatsnew/3.5.html#zipapp)


**内置模块的变更：**

| 变更 | 状态 |
| - | - |
|**[collections](https://docs.python.org/3/whatsnew/3.5.html#collections)**||
| OrderedDict 类现在用 C 实现，速度提升 4 到 100 倍。||
| `OrderedDict.items()`、`OrderedDict.keys()`、`OrderedDict.values()` 视图现在支持 `reversed()` 迭代。||
| deque 类现在定义了 `index()`、`insert()` 和 `copy()` 方法，并支持 + 和 * 运算符。||
| `namedtuple()` 生成的文档字符串现在可以更新。||
| UserString 类现在实现了 `__getnewargs__()`、`__rmod__()`、`casefold()`、`format_map()`、`isprintable()` 和 `maketrans()` 方法，以匹配 str 的相应方法。||
|**[heapq](https://docs.python.org/3/whatsnew/3.5.html#heapq)**||
| `merge()` 中的元素比较现在可以通过新的可选 key 关键字参数传入键函数来自定义，还可以使用新的可选 `reverse` 关键字参数来反转元素比较。||
|**[io](https://docs.python.org/3/whatsnew/3.5.html#io)**||
| 新增了 `BufferedIOBase.readinto1()` 方法，该方法最多使用一次对底层原始流的 `RawIOBase.read()` 或 `RawIOBase.readinto()` 方法的调用。||
|**[json](https://docs.python.org/3/whatsnew/3.5.html#json)**||
| JSON 解码器现在抛出 JSONDecodeError 而非 ValueError，以提供更详细的错误上下文信息。||
|**[math](https://docs.python.org/3/whatsnew/3.5.html#math)**||
| math 模块新增了两个常量：`inf` 和 `nan`。| 完全支持 |
| 新增 `isclose()` 函数，提供一种测试近似相等的方法。||
| 新增 `gcd()` 函数。`fractions.gcd()` 函数现已弃用。||
|**[os](https://docs.python.org/3/whatsnew/3.5.html#os)**||
| 新增了 `scandir()` 函数，返回 DirEntry 对象的迭代器。||
| 在 Linux 3.17 及更高版本上，`urandom()` 函数现在使用 `getrandom()` 系统调用，在 OpenBSD 5.6 及更高版本上使用 `getentropy()`，无需再使用 `/dev/urandom`，避免了因潜在的文件描述符耗尽而导致的失败。||
| 新增 `get_blocking()` 和 `set_blocking()` 函数，允许获取和设置文件描述符的阻塞模式（O_NONBLOCK）。||
| 新增 `os.path.commonpath()` 函数，返回所传入每个路径名的最长公共子路径。||
|**[re](https://docs.python.org/3/whatsnew/3.5.html#re)**||
| 现在允许在 lookbehind 断言中引用和有条件地引用具有固定长度的组。||
| 正则表达式中的捕获组数量不再限制为100个。||
| `sub()` 和 `subn()` 函数现在将未匹配的组替换为空字符串，而非抛出异常。||
| `re.error` 异常新增了 msg、pattern、pos、lineno 和 colno 属性，提供更详细的错误上下文信息。||
|**[socket](https://docs.python.org/3/whatsnew/3.5.html#socket)**||
| 带超时的函数现在使用单调时钟，而非系统时钟。||
| 新增 `socket.sendfile()` 方法，在UNIX上通过使用高性能的 `os.sendfile()` 函数来通过套接字发送文件，使得上传速度比使用普通的 `socket.send()` 快 2 到 3 倍。||
| `socket.sendall()` 方法不再在每次接收或发送字节时重置套接字超时。套接字超时现在是发送所有数据的最大总时长。||
| `socket.listen()` 方法的 backlog 参数现在是可选的。默认情况下，它被设置为 SOMAXCONN 或 128 中的较小值。| 完全支持 |
|**[ssl](https://docs.python.org/3/whatsnew/3.5.html#ssl)**||
| 内存 BIO 支持 ||
| 应用层协议协商支持 ||
| 新增 `SSLSocket.version()` 方法，用于查询实际使用的协议版本。||
| SSLSocket 类现在实现了 `SSLSocket.sendfile()` 方法。||
| 在非阻塞套接字上，如果操作会阻塞，`SSLSocket.send()` 方法现在会抛出 `ssl.SSLWantReadError` 或 `ssl.SSLWantWriteError` 异常。以前，它会返回 0。||
| 根据 RFC 5280，`cert_time_to_seconds()` 函数现在将输入时间解释为 UTC 而非本地时间。此外，返回值始终为 int。||
| 新增 `SSLObject.shared_ciphers()` 和 `SSLSocket.shared_ciphers()` 方法，返回握手期间客户端发送的密码套件列表。||
| SSLSocket 类的 `SSLSocket.do_handshake()`、`SSLSocket.read()`、`SSLSocket.shutdown()` 和 `SSLSocket.write()` 方法不再在每次接收或发送字节时重置套接字超时。||
| `match_hostname()` 函数现在支持 IP 地址的匹配。||
|**[sys](https://docs.python.org/3/whatsnew/3.5.html#sys)**||
| 新增 `set_coroutine_wrapper()` 函数，允许设置一个全局钩子，该钩子会在 async def 函数创建协程对象时被调用。相应的 `get_coroutine_wrapper()` 可用于获取当前设置的包装器。||
| 新增 `is_finalizing()` 函数，可用于检查 Python 解释器是否正在关闭。||
|**[time](https://docs.python.org/3/whatsnew/3.5.html#time)**||
| `monotonic()` 函数现在始终可用。||
