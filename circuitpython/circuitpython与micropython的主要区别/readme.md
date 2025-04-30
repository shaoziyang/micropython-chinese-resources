# Circuitpython与micropython的主要区别

虽然 CircuitPython 是从 micropython 中分叉出来的，但是随着各自的不断发展，两者之间的差异也越来越多。它们之间的主要区别是：

**CircuitPython**

- 在所有板上都支持本机USB，无需特殊软件即可编辑文件。
- 所有版本都启用浮点数（即小数）。
- 错误消息被翻译成10多种语言（中文是拼音）。
- 不支持Python中的并发功能（包括中断和线程）。对于需要它的任务（如音频文件播放），本地模块可以实现一些并发性。

**行为**

- 文件的运行顺序以及它们之间共享的状态。CircuitPython的目标是明确每个文件的作用，并使每个文件相互独立。
- boot.py (or settings.py)在USB初始化之前仅运行一次。这为在启动时配置USB而不是修复它奠定了基础。因为此时串口不可用，所以输出被写入文件 boot_out.txt。
- code.py (or main.py)在每次重新加载后运行，直到完成或中断。运行完成后，虚拟机和硬件将重新初始化。这意味着您无法在REPL中读取code.py运行状态。circuittpython的目标是减少对管脚和内存使用的混淆。
- code.py运行完成后可按任意键进入REPL。它不再与REPL共享code.py状态，所以这是一个新的虚拟机。
- 自动加载状态将在整个重新加载过程中保持。
- 添加一个安全模式，在硬崩溃或掉电后不再运行用户代码。在崩溃后通过大容量存储更容易修复导致严重崩溃的代码。修复后通过复位回到正常模式。
- RGB LED指示CircuitPython 状态，通过不同色彩指示电路状态和错误。
- 在文件写入USB大容量存储后重新运行code.py或其他主文件。（使用supervisor.disable_autoreload()禁用)
- 在主代码完成后需要通过按键进入REPL，并禁用自动加载。
- 主程序可以是这几个文件其中之一：code.txt, code.py, main.py, main.txt
- 引导文件可以是这几个文件其中之一：settings.txt, settings.py, boot.py, boot.txt

**API**

- 统一的硬件API。文档在ReadTheDocs上。
- API文档在shared-bindings中和C 文件共享。
- 没有machine API。

**模块**

- 无模块别名。（uos和utime不能分别作为os和time使用。）相反，os、time和random 是与 CPython 兼容的。
- 新的 storage 模块管理文件系统装载。（来自 MicroPython 中 uos 的功能）
- 与CPython对应的模块，如time、os和random，是CPython版本的严格子集。因此，CircuitPython的代码可以在CPython上运行，但反过来则不一定。
- time.monotonic() 函数用于tick 计数器