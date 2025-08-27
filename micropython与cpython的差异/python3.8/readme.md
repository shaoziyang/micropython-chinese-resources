# Python 3.8

Python 3.8.0（最终版）于2019年10月14日发布。3.8版本的特性在 [PEP 569](https://www.python.org/dev/peps/pep-0569/#id9) 中定义，有关变更的详细说明可参见 [Python 3.8新特性](https://docs.python.org/3/whatsnew/3.8.html)。

| PEP 编号 | 描述 | 状态 |
| --- | --- | --- |
| [PEP 570](https://www.python.org/dev/peps/pep-0570/) | 仅限位置参数 |  |
| [PEP 572](https://www.python.org/dev/peps/pep-0572/) | 赋值表达式 | 完全支持 |
| [PEP 574](https://www.python.org/dev/peps/pep-0574/) | 带有带外数据的Pickle协议5 |  |
| [PEP 578](https://www.python.org/dev/peps/pep-0578/) | 运行时审计钩子 |  |
| [PEP 587](https://www.python.org/dev/peps/pep-0587/) | Python 初始化配置 |  |
| [PEP 590](https://www.python.org/dev/peps/pep-0590/) | Vectorcall：CPython的快速调用协议 |  |
| 其他 | f字符串支持=用于自文档化表达式和调试 | 完全支持 |


**其他语言变更：**

| 变更 | 状态 |
| - | - |
| 由于实现问题，`finally`子句中使用`continue`语句曾是非法的。在Python 3.8中，这一限制被取消。 | 完全支持 |
| `bool`、`int` 和 `fractions.Fraction` 类型现在拥有 `as_integer_ratio()` 方法，类似 `float` 和 `decimal.Decimal` 中的该方法。 | |
| `int`、`float` 和 `complex` 的构造函数现在会使用 `__index__()` 特殊方法（如果该方法存在，且对应的 `__int__()`、`__float__()` 或 `__complex__()` 方法不存在）。| |
| 正则表达式中新增了对 `N{name}` 转义的支持。| |
| 现在可使用 `reversed()` 按反向插入顺序迭代字典和字典视图。| |
| 函数调用中关键字名称的语法受到进一步限制。特别是不再允许 `f((keyword)=arg)` 这种形式。 | |
| `yield` 和 `return` 语句中的通用可迭代解包不再要求使用括号包裹。| |
| 当代码中漏掉逗号（如`[(10, 20) (30, 40)]`）时，编译器会显示`SyntaxWarning`并给出有益的建议。| |
| `datetime.date` 或 `datetime.datetime` 的子类与 `datetime.timedelta` 对象之间的算术运算，现在返回子类的实例，而非基类的实例。| |
| 当 Python 解释器被 Ctrl-C（SIGINT） 中断，且产生的 `KeyboardInterrupt` 异常未被捕获时，Python 进程现在会通过 SIGINT 信号或正确的退出码退出，以便调用进程能够检测到它是因 Ctrl-C 而终止的。| |
| 某些高级编程风格需要更新现有函数的 `types.CodeType` 对象。| |
| 对于整数，`pow()` 函数的三参数形式现在允许在底数与模数互质的情况下，指数为负数。| |
| 字典推导式已与字典字面量同步，即先计算键，再计算值。| |
| `object.__reduce__()` 方法现在可以返回长度为2到6的元组。| |


**内置模块的变更：**
| 变更 | 状态 |
| - | - |
| **[asyncio](https://docs.python.org/3/whatsnew/3.8.html#asyncio)** | |
| `asyncio.run()` 已从临时API升级为稳定API。| 完全支持 |
| 运行 `python -m asyncio` 会启动原生异步REPL。| |
| 异常 `asyncio.CancelledError` 现在继承自 `BaseException` 而非 `Exception`，且不再继承自 `concurrent.futures.CancelledError`。| 完全支持 |
| 新增 `asyncio.Task.get_coro()`，用于获取 `asyncio.Task` 中包装的协程。| |
| 现在可以为asyncio任务命名，可通过向`asyncio.create_task()`或事件循环的`create_task()`方法传递`name`关键字参数，或调用任务对象的`set_name()`方法来实现。| |
| `asyncio.loop.create_connection()`新增了对“Happy Eyeballs”算法的支持。为指定该行为，新增了两个参数：`happy_eyeballs_delay`和`interleave`。| |
| **[gc](https://docs.python.org/3/whatsnew/3.8.html#gc)** | |
| `get_objects()`现在可以接收一个可选的`generation`参数，用于指定要从中获取对象的代。（注意，尽管`gc`是内置模块，但MicroPython未实现`get_objects()`） | |
| **[math](https://docs.python.org/3/whatsnew/3.8.html#math)** | |
| 新增`math.dist()`函数，用于计算两点之间的欧几里得距离。 | |
| 扩展了`math.hypot()`函数，使其能够处理多维情况。 | |
| 新增`math.prod()`函数，作为与`sum()`类似的函数，返回“起始”值（默认值为1）与可迭代数字序列的乘积。 | |
| 新增两个组合函数`math.perm()`和`math.comb()`。 | |
| 新增`math.isqrt()`函数，用于计算精确的整数平方根，无需转换为浮点数。 | |
| `math.factorial()`函数不再接受非类整数参数。 | 完全支持 |
| **[sys](https://docs.python.org/3/whatsnew/3.8.html#sys)** | |
| 新增`sys.unraisablehook()`函数，可通过重写该函数来控制“不可捕获的异常”的处理方式。 | |
