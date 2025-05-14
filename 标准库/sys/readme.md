# sys（系统特定功能）

`sys` 模块提供了与 Python 解释器和系统环境交互的功能。

## 函数

- sys.`exit`(retval=0, / )

  用指定的退出码中止当前程序，它也会产生一个 `SystemExit` 异常。如果指定了 `retval` 参数，这个参数也会传递到 `SystemExit`。
  
  在除了 Windows 和 Unix 外的嵌入式移植版本上，未处理的 `SystemExit` 当前会导致 MicroPython 进行软复位。
<br><br>

- sys.`atexit`(func)

  注册在终止时调用的 `func`。`func` 必须是不带参数的可调用函数，或者是 `None` 禁用调用。`atexit` 函数将返回此函数设置的上一个值，它的初始值为 `None`。
<br><br>

  **与 CPython 的差异**

  此函数是 MicroPython 的扩展，旨在提供与 CPython 中的 `atexit` 模块类似的功能。
<br><br>

- sys.`print_exception`(exc, file=sys.stdout, / )

  将异常及其追溯信息打印到类似文件的对象 file 中（默认打印到 `sys.stdout`）。。
<br><br>

  **和 CPython 的差异**

  在 MicroPython 中，这个函数是 CPython 中 `traceback` 模块的简化版本。和函数 `traceback.print_exception()` 不同，此函数只获取异常值，而不获取异常类型、异常值和回溯对象。file 参数必须是位置参数，不支持更多参数。兼容 CPython 的 `traceback` 模块放在了 `micropython-lib` 中。
<br><br>

- sys.`settrace`(tracefunc)

  启用对字节码执行的跟踪。详细信息请参阅 [CPython 文档](https://docs.python.org/3/library/sys.html#sys.settrace)。
  
  此函数需要自行定义构建的 MicroPython，因为它通常不存在于预构建的固件中（它会显著影响性能）。相关配置选项为 `MICROPY_PY_SYS_SETTRACE`。
<br><br>

## 常数

- sys.`argv`

  当前程序启动时使用的可变参数列表。
<br><br>

- sys.`byteorder`

  系统的字节序，"little" （小）或 "big"（大）。
<br><br>

- sys.`implementation`

  提供当前 Python 实现相关信息的对象。对于 MicroPython 而言，它具备以下这些属性：
  - `name`：字符串'micropython'
  - `version`：元组(主版本, 次版本, 小版本, 发布级别)，如(1, 21, 0, "")。
  - `_machine`：描述设备的字符串。
  - `_mpy`：mpy 文件格式版本。
  - `_build`：有助于识别 MicroPython 构建配置的字符串。

  这个方法推荐用来区分 MicroPython 和其它的版本的 Python (注意极少数 micropython 移植版不支持这个函数)。
  
  从 1.22.0-preview 版本开始，`implementation.version`中的第四个元素（发布级别）要么是空字符串，要么是"preview"。

  `_build` 是在 1.25.0 版本中添加的，是一组以连字符分隔的元素。未来可能会添加新元素，因此最好使用 `sys.implementation._build.split("-")` 访问此字段。目前使用的元素有：

  - 在unix、webassembly 和 windows 移植版上，第一个元素是变量名，例如 "standard"。
  - 在微控制器版本，第一个元素是板名，第二个元素（如果存在）是板变体，例如 "RPI_PICO2-RISCV"。
<br><br>

  **和 CPython 的差异**

  CPython 包含了更多属性，MicroPython 只实现了基本功能。
<br><br>

- sys.`maxsize`

  在当前平台上，本地整数类型所能容纳的最大值；或者，若 MicroPython 整数类型的表示范围比平台最大值小（例如在不支持长整型的 MicroPython 版本中），则为 MicroPython 整数类型的最大值。
  
  这个属性可以用来检测系统的 "位数" (如 32 位或 64 位)。建议不要直接将此属性与某个值进行比较，而是计算其位数，例如：

  ```py
  bits = 0
  v = sys.maxsize
  while v:
      bits += 1
      v >>= 1

  if bits > 32:
      # 64 位 (或更高) 系统
      ...
  else:
      # 32 位 (或更低) 系统
      # 注意在 32 位系统中，因为前面说明的原因，bits 数值可能小于 32 (如 31)
      # 因此要使用 "> 16", "> 32", "> 64" 这种方法进行比较。
  ```

- sys.`modules`

  已载入模块的字典。在某些移植版中，没有包含这个函数。
<br><br>

- sys.`path`

  系统路径。与 windows 和 Linux 的系列路径类似，可以将用户目录附加到系统路径中，这样可以方便调用不同目录中的模块和程序而无需在程序前加上路径。可以通过append() 函数添加新的路径。

  ```py
  >>> sys.path
  ['', '/sd', '/sd/lib', '/flash', '/flash/lib']
  >>> sys.path.append('/user')
  >>> sys.path
  ['', '/sd', '/sd/lib', '/flash', '/flash/lib', '/user']
  ```

  **与 CPython 的差异**
  
  在 MicroPython 中，".frozen" 条目表明导入操作应在此搜索点查找冻结模块（frozen modules）。若未找到冻结模块，搜索不会尝试查找名为 `.frozen` 的目录，而是直接继续检查 `sys.path` 中的下一个项。
<br><br>

- sys.`platform`

  MicroPython 运行的硬件平台。对于 OS/RTOS 移植，这通常是操作系统的标识符，例如"linux"。对于硬件移植，它通常是硬件或开发板的标识符，例如原始 MicroPython 的 "pyboard" 开发板。因此，它可以用来区分不同开发板。如果需要判断程序是否在 MicroPython 上运行（相对于其他Python实现），请使用 `sys.implementation`。
<br><br>

- sys.`ps1`
- sys.`ps2`

  REPL 使用的提示字符串。默认情况下，标准 Python 提示符为 '>>>' 和 '...'，可以修改为任意字符串，甚至是空字符串或者中文。
<br><br>

- sys.`stderr`

  标准错误输出设备（PyBoard 中默认是 USB 虚拟串口，可设置为用其他串口）。
<br><br>

- sys.`stdin`

  标准输入设备（PyBoard 中默认是 USB 虚拟串口，可设置为其他串口）。
<br><br>

- sys.`stdout`

  标准输出设备（PyBoard 中默认是 USB 虚拟串口，可设置为其他串口）。
<br><br>

- sys.`version`

  Python 语言的版本，字符串格式，如 '3.4.0; MicroPythonv1.22.0-preview.4.g9f835df35 on 2023-10-10'。
<br><br>

- sys.`version_info`

  表示 MicroPython 所兼容的 Python 语言版本的整数元组。如：(3, 4, 0)。
<br><br>

  **与 CPython 的差异**

  只支持前三个版本号（主、次、小），并且只能通过索引引用，不能通过名称引用。

