# Python 3.6

Python 3.6 beta 1 于 2016 年 9 月 12 日发布，其新特性摘要可参考此处：

| PEP 编号 | 描述 | 状态 |
| --- | --- | --- |
| **新语法特性** | | |
| [PEP 498](https://www.python.org/dev/peps/pep-0498/) | 字面字符串格式化 | 完全支持 |
| [PEP 515](https://www.python.org/dev/peps/pep-0515/) | 数值字面量中的下划线 | 完全支持 |
| [PEP 525](https://www.python.org/dev/peps/pep-0525/) | 异步生成器 |  |
| [PEP 526](https://www.python.org/dev/peps/pep-0526/) | 变量注解语法（暂定） | 完全支持 |
| [PEP 530](https://www.python.org/dev/peps/pep-0530/) | 异步推导式 |  |
| **新内置特性** | | |
| [PEP 468](https://www.python.org/dev/peps/pep-0468/) | 保留函数中关键字参数的顺序 |  |
| [PEP 487](https://www.python.org/dev/peps/pep-0487/) | 更简洁的类创建自定义方式 | 部分支持 |
| [PEP 520](https://www.python.org/dev/peps/pep-0520/) | 保留类属性定义顺序 |  |
| **标准库变更** | | |
| [PEP 495](https://www.python.org/dev/peps/pep-0495/) | 本地时间歧义处理 |  |
| [PEP 506](https://www.python.org/dev/peps/pep-0506/) | 在标准库中添加 secrets 模块 |  |
| [PEP 519](https://www.python.org/dev/peps/pep-0519/) | 添加文件系统路径协议 |  |
| **CPython 内部机制** | | |
| [PEP 509](https://www.python.org/dev/peps/pep-0509/) | 为字典添加私有版本 | 不支持 |
| [PEP 523](https://www.python.org/dev/peps/pep-0523/) | 为 CPython 添加帧评估 API |  |
| **Linux/Windows 变更** | | |
| [PEP 524](https://www.python.org/dev/peps/pep-0524/) | 使 Linux 上的 os.urandom() 阻塞（在系统启动期间） |  |
| [PEP 528](https://www.python.org/dev/peps/pep-0528/) | 将 Windows 控制台编码改为 UTF-8 |  |
| [PEP 529](https://www.python.org/dev/peps/pep-0529/) | 将 Windows 文件系统编码改为 UTF-8 |  |


**其他语言变更：**

| 变更 | 状态 |
| - | - |
| 全局或非局部语句现在必须在同一作用域中受影响名称的首次使用之前出现。此前这只是一个语法警告。||
| 现在可以将特殊方法设为 `None`，以表示相应的操作不可用。例如，如果一个类将 `__iter__()` 设为 `None`，那么该类不可迭代。||
| 重复的长回溯行序列现在会缩写为 *[Previous line repeated {count more times]}*（前面的行重复了 {count} 次）。||
| 当导入无法找到模块时，现在会抛出新的异常 `ModuleNotFoundError`。目前在 try-except 中检查 ImportError 的代码仍然有效。||
| 依赖无参数 super() 的类方法，在类创建期间从元类方法调用时，现在能正确工作。||


**内置模块的变更：**

| 变更 | 状态 |
| - | - |
|**[array](https://docs.python.org/3.6/whatsnew/3.6.html#array)**||
| `array.array` 的迭代器耗尽后，即使被迭代的数组被扩展，迭代器也会保持耗尽状态。||
|**[binascii](https://docs.python.org/3.6/whatsnew/3.6.html#binascii)**||
| `b2a_base64()` 函数现在接受一个可选的 newline 关键字参数，用于控制是否在返回值后附加换行符。| 完全支持 |
|**[cmath](https://docs.python.org/3.6/whatsnew/3.6.html#cmath)**||
| 新增了 `cmath.tau`（τ）常量。||
| 新增常量：`cmath.inf` 和 `cmath.nan`，以匹配 `math.inf` 和 `math.nan`；还有 `cmath.infj` 和 `cmath.nanj`，以匹配复数 repr 所用的格式。||
|**[collections](https://docs.python.org/3.6/whatsnew/3.6.html#collections)**||
| 新增了 `Collection` 抽象基类，用于表示有大小的可迭代容器类。||
| 新增了 `Reversible` 抽象基类，表示同时提供 `__reversed__()` 方法的可迭代类。||
| 新增了 `AsyncGenerator` 抽象基类，表示异步生成器。||
| `namedtuple()` 函数现在接受一个可选的 keyword 参数 module，指定后会用于返回的命名元组类的 `__module__` 属性。||
| `namedtuple()` 的 verbose 和 rename 参数现在仅支持关键字方式传入。||
| 递归的 `collections.deque` 实例现在可以被 pickle 序列化。||
|**[hashlib](https://docs.python.org/3.6/whatsnew/3.6.html#hashlib)**||
| 模块中添加了 BLAKE2 哈希函数。`blake2b()` 和 `blake2s()` 始终可用，并支持 BLAKE2 的全部特性集。||
| 添加了 SHA-3 哈希函数 `sha3_224()`、`sha3_256()`、`sha3_384()`、`sha3_512()`，以及 SHAKE 哈希函数 `shake_128()` 和 `shake_256()`。||
| 在 OpenSSL 1.1.0 及更高版本中，现在可使用基于密码的密钥派生函数 `scrypt()`。||
|**[json](https://docs.python.org/3.6/whatsnew/3.6.html#json)**||
| `json.load()` 和 `json.loads()` 现在支持二进制输入。编码后的 JSON 应使用 UTF-8、UTF-16 或 UTF-32 表示。||
|**[math](https://docs.python.org/3.6/whatsnew/3.6.html#math)**||
| 新增了 `math.tau`（τ）常量。| 完全支持 |
|**[os](https://docs.python.org/3.6/whatsnew/3.6.html#os)**||
| 新增 `close()` 方法，允许显式关闭 `scandir()` 迭代器。`scandir()` 迭代器现在支持上下文管理器协议。||
| 在 Linux 上，`os.urandom()` 现在会阻塞，直到系统 urandom 熵池初始化完成，以提高安全性。||
| Linux 的 `getrandom()` 系统调用（获取随机字节）现在通过新的 `os.getrandom()` 函数暴露。||
|**[re](https://docs.python.org/3.6/whatsnew/3.6.html#re)**||
| 增加了对正则表达式中修饰符范围的支持。例如：`(?i:p)ython` 匹配 "python" 和 "Python"，但不匹配 "PYTHON"；`(?i)g(?-i:v)r` 匹配 "GvR" 和 "gvr"，但不匹配 "GVR"。||
| 匹配对象的组可以通过 `__getitem__` 访问，这与 `group()` 等效。因此 `mo['name']` 现在等效于 `mo.group('name')`。||
| 匹配对象现在支持类索引对象作为组索引。||
|**[socket](https://docs.python.org/3.6/whatsnew/3.6.html#socket)**||
| `ioctl()` 函数现在支持 `SIO_LOOPBACK_FAST_PATH` 控制代码。||
| 现在支持 `getsockopt()` 常量 `SO_DOMAIN`、`SO_PROTOCOL`、`SO_PEERSEC` 和 `SO_PASSSEC`。||
| `setsockopt()` 现在支持 `setsockopt(level, optname, None, optlen: int)` 形式。||
| socket 模块现在支持地址族 `AF_ALG`，以与 Linux 内核加密 API 交互。添加了 `ALG_`、`SOL_ALG` 和 `sendmsg_afalg()`。||
| 新增了 Linux 常量 `TCP_USER_TIMEOUT` 和 `TCP_CONGESTION`。||
|**[ssl](https://docs.python.org/3.6/whatsnew/3.6.html#ssl)**||
| ssl 支持 OpenSSL 1.1.0。建议的最低版本是 1.0.2。||
| 3DES 已从默认密码套件中移除，并添加了 ChaCha20 Poly1305 密码套件。||
| SSLContext 在选项和密码方面有了更好的默认配置。||
| 通过新的 SSLSession 类，可以将 SSL 会话从一个客户端连接复制到另一个。TLS 会话恢复可以加快初始握手速度、减少延迟并提高性能。||
| 新的 `get_ciphers()` 方法可用于获取按密码优先级排序的已启用密码列表。||
| 所有常量和标志都已转换为 IntEnum 和 IntFlags。||
| 为 SSLContext 添加了服务器端和客户端特定的 TLS 协议。||
| 添加了 `SSLContext.post_handshake_auth` 以启用，以及 `ssl.SSLSocket.verify_client_post_handshake()` 以启动 TLS 1.3 握手后认证。||
|**[struct](https://docs.python.org/3.6/whatsnew/3.6.html#struct)**||
| 现在通过 "e" 格式说明符支持 IEEE 754 半精度浮点数。||
|**[sys](https://docs.python.org/3.6/whatsnew/3.6.html#sys)**||
| 新的 `getfilesystemencodeerrors()` 函数返回用于在 Unicode 文件名和字节文件名之间转换的错误模式名称。||
|**[zlib](https://docs.python.org/3.6/whatsnew/3.6.html#zlib)**||
| `compress()` 和 `decompress()` 函数现在接受关键字参数。||
