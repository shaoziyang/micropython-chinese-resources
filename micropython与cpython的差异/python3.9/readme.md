# Python 3.9

Python 3.9.0（最终版）于2020年10月5日发布。3.9版本的特性在 [PEP 596](https://www.python.org/dev/peps/pep-0596/#features-for-3-9) 中定义，有关变更的详细说明可参见 [Python 3.9 新特性](https://docs.python.org/3/whatsnew/3.9.html)。

**特性**

| PEP编号 | 描述 | 状态 |
| --- | --- | --- |
| [PEP 573](https://www.python.org/dev/peps/pep-0573/) | 从C扩展类型的方法快速访问模块状态 | 不相关 |
| [PEP 584](https://www.python.org/dev/peps/pep-0584/) | 为字典添加联合运算符 | 完全支持 |
| [PEP 585](https://www.python.org/dev/peps/pep-0585/) | 标准集合中的类型提示泛型 |  |
| [PEP 593](https://www.python.org/dev/peps/pep-0593/) | 灵活的函数和变量注解 |  |
| [PEP 602](https://www.python.org/dev/peps/pep-0602/) | CPython 采用年度发布周期（而非之前计划的两个月发布周期） | 不相关 |
| [PEP 614](https://www.python.org/dev/peps/pep-0614/) | 放宽对装饰器的语法限制 |  |
| [PEP 615](https://www.python.org/dev/peps/pep-0615/) | IANA 时区数据库现已纳入标准库的 zoneinfo 模块 |  |
| [PEP 616](https://www.python.org/dev/peps/pep-0616/) | 用于移除前缀和后缀的字符串方法 |  |
| [PEP 617](https://www.python.org/dev/peps/pep-0617/) | CPython现在使用基于 PEG 的新解析器 | 不相关 |


**其他语言变更：**

| 变更 | 状态 |
| - | - |
| `__import__()`现在抛出`ImportError`而非`ValueError`。| 完全支持 |
| Python 现在会获取命令行中指定的脚本文件名的绝对路径（例如：`python3 script.py`）：`__main__`模块的`__file__`属性变为绝对路径，而非相对路径。||
| 默认情况下，为获得最佳性能，`errors`参数仅在首次编码/解码错误时检查，且对于空字符串，`encoding`参数有时会被忽略。||
| 对于所有非零的`n`，`"".replace("", s, n)`现在返回`s`而非空字符串。这与`"".replace("", s)`保持一致。||
| 现在任何有效的表达式都可作为装饰器使用。此前，语法限制更为严格。||
| 现在禁止`aclose()`/`asend()`/`athrow()`的并行运行，且`ag_running`会反映异步生成器的实际运行状态。||
| 在`in`运算符以及`operator`模块的`contains()`、`indexOf()`和`countOf()`函数中，调用`__iter__`方法时出现的意外错误不再被`TypeError`掩盖。||
| 在推导式和生成器表达式的`if`子句中，未加括号的`lambda`表达式不能再作为表达式部分。||


**内置模块的变更：**

| 变更 | 状态 |
| - | - |
| **[asyncio](https://docs.python.org/3/whatsnew/3.9.html#asyncio)** ||
| 由于重大安全问题，`asyncio.loop.create_datagram_endpoint()`的`reuse_address`参数不再受支持。||
| 新增协程`shutdown_default_executor()`，用于调度默认执行器的关闭操作，等待`ThreadPoolExecutor`完成关闭。此外，`asyncio.run()`已更新为使用该新协程。||
| 新增`asyncio.PidfdChildWatcher`，这是一个Linux特有的子进程监视器实现，用于轮询进程文件描述符。||
| 新增协程`asyncio.to_thread()`。||
| 当因超时而取消任务时，`asyncio.wait_for()`现在在超时时间`<= 0`的情况下，也会等待取消完成，就像处理正超时时间时一样。||
| 当对`ssl.SSLSocket`套接字调用不兼容的方法时，`asyncio`现在会抛出`TypeError`。||
| **[gc](https://docs.python.org/3/whatsnew/3.9.html#gc)** ||
| 垃圾回收不会因复活对象而阻塞。||
| 新增函数`gc.is_finalized()`，用于检查对象是否已被垃圾回收器终结。||
| **[math](https://docs.python.org/3/whatsnew/3.9.html#math)** ||
| 扩展`math.gcd()`函数以处理多个参数。此前，它仅支持两个参数。||
| 新增`math.lcm()`：返回指定参数的最小公倍数。||
| 新增`math.nextafter()`：返回`x`向`y`方向的下一个浮点值。||
| 新增`math.ulp()`：返回浮点数的最低有效位的值。||
| **[os](https://docs.python.org/3/whatsnew/3.9.html#os)**||
| 暴露Linux特有的`os.pidfd_open()`和`os.P_PIDFD`。||
| `os.unsetenv()`函数现在在Windows上也可用。|完全支持|
| `os.putenv()`和`os.unsetenv()`函数现在始终可用。|完全支持|
| 新增`os.waitstatus_to_exitcode()`函数：将等待状态转换为退出码。||
| **[random](https://docs.python.org/3/whatsnew/3.9.html#random)** ||
| 新增`random.Random.randbytes`方法：生成随机字节。||
| **[sys](https://docs.python.org/3/whatsnew/3.9.html#sys)** ||
| 新增`sys.platlibdir`属性：特定平台的库目录名称。||
| 此前，`sys.stderr`在非交互模式下是块缓冲的。现在，`stderr`默认始终为行缓冲。||
