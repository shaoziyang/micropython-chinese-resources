# 术语表

- **baremetal（裸机）**：指没有（完善的）操作系统的系统，例如基于微控制器（MCU）的系统。在裸机系统上运行时，MicroPython 实际上起到小型操作系统的作用，运行用户程序并提供命令解释器（REPL）。
- **buffer protocol（缓冲区协议）**：任何可以自动转换为字节的 Python 对象，如 bytes、bytearray、memoryview 和 str 对象，它们都实现了“缓冲区协议”。
- **board（开发板）**：通常指包含微控制器和支持组件的印刷电路板（PCB）。 MicroPython 固件通常是针对特定开发板提供的，因为固件既包含微控制器特定功能，也包含开发板的功能，如驱动程序或引脚名称。
- **bytecode（字节码）**：通过编译 Python 源代码生成的 Python 程序的紧凑表示形式。这是虚拟机（VM）实际执行的内容。字节码通常在运行时自动生成，对用户不可见。请注意，虽然CPython和微Python都使用字节码，但它们的格式不同。
- **callee-owned tuple（被调用者拥有的元组）**：这是 MicroPython 特有的结构，出于效率考虑，一些内置函数或方法可能会重用同一个底层元组对象来返回数据。这避免了每次调用都必须分配新元组，减少了堆碎片。程序不应持有被调用者拥有的元组的引用，而应仅从中提取数据（或制作副本）。
- **CircuitPython（电路Python）**：由 Adafruit Industries 开发的 MicroPython 变体。
- **CPython**：CPython 是 Python 编程语言的参考实现，也是最著名的一种。然而，它只是众多实现中的一种（包括 Jython、IronPython、PyPy 和 MicroPython）。尽管 MicroPython 的实现与 CPython 有很大差异，但它旨在保持尽可能高的兼容性。
- **cross-compiler（交叉编译器）**：也称为 mpy-cross。该工具在电脑上运行，将包含 MicroPython 代码的 .py 文件转换为包含 MicroPython 字节码的 .mpy 文件。这意味着加载速度更快（开发板无需编译代码），并且在闪存上占用更少空间（字节码的空间效率更高）。
- **driver（驱动程序）**：实现对特定组件（如传感器或显示器）支持的 MicroPython 库。
- **FFI**：Foreign Function Interface（外部函数接口）的缩写。MicroPython Unix 版本使用的一种机制，用于访问操作系统功能。在裸机版本上不可用。
- **filesystem（文件系统）**：大多数 MicroPython 移植版本和开发板提供存储在闪存中的文件系统，用户代码可通过标准 Python 文件 API（如 `open()`）访问。有些开发板还通过USB大容量存储使这个内部文件系统可被主机访问。
- **frozen module（冻结模块）**：经过交叉编译并捆绑到固件镜像中的 Python 模块。这减少了对 RAM 的需求，因为代码直接从闪存执行。
- **Garbage Collector（垃圾回收器）**：在Python（和 MicroPython）中运行的后台进程，用于回收堆中未使用的内存。
- **GPIO**：General-purpose input/output（通用输入/输出）的缩写。控制微控制器上电信号（通常称为“引脚”）的最简单方式。GPIO 通常允许引脚作为输入或输出，并设置或获取其数字值（逻辑“0”或“1”）。MicroPython 通过 `machine.Pin` 和 `machine.Signal` 类抽象 GPIO 访问。
- **GPIO port（GPIO端口）**：一组 GPIO 引脚，通常基于这些引脚的硬件特性（例如，可由同一寄存器控制）。
- **heap（堆）**：MicroPython 存储动态数据的 RAM 区域。它由垃圾回收器自动管理。不同的微控制器和开发板可用于堆的 RAM 数量差异很大，这会影响你的程序的复杂程度。
- **interned string（驻留字符串）**：MicroPython 使用的一种优化，用于提高字符串操作的效率。驻留字符串通过其（唯一的）标识而非地址来引用，因此只需通过其标识符即可快速比较。这也意味着相同的字符串在内存中可以去重。字符串驻留对用户几乎是不可见的。
- **MCU（微控制器）**：微控制器通常比台式机、笔记本电脑或手机的资源少得多，但体积更小、成本更低且功耗更低。MicroPython 设计得小巧且经过优化，足以在普通现代微控制器上运行。
- **micropython-lib（微Python库）**：MicroPython（通常）作为单个可执行文件/二进制文件分发，仅包含少数内置模块。它没有与 CPython 相当的广泛标准库。相反，有一个相关但独立的项目 micropython-lib，它提供了 CPython 标准库中许多模块的实现。

  有些模块是用纯Python实现的，可在所有版本上使用。然而，这些模块中的大多数使用 FFI 来访问操作系统功能，因此只能在 MicroPython Unix端口上使用（对 Windows 的支持有限）。

  与CPython标准库不同，micropython-lib 模块旨在单独安装 —— 可以通过手动复制或使用 `mip`。
- **MicroPython port（MicroPython 移植版）**：MicroPython 支持不同的开发板、实时操作系统（RTOS）和操作系统，并且可以相对容易地适配新系统。支持特定系统的 MicroPython 被称为该系统的“移植版”。不同的移植版可能具有很大不同的功能。本文档旨在作为不同移植版通用 API（“MicroPython 核心”）的参考。注意，有些移植版可能会省略了此处描述的某些 API（例如，由于资源限制）。任何此类差异以及超出 MicroPython 核心功能的特定扩展，将在单独的移植版特定文档中描述。
- **MicroPython Unix port（MicroPython Unix 移植版）**：Unix 移植版是主要的 MicroPython 移植版之一。它旨在运行在 POSIX 兼容的操作系统上，如 Linux、MacOS、FreeBSD、Solaris等。它也 是Windows 移植版的基础。Unix 移植版对于快速开发和测试 MicroPython 语言以及与硬件无关的功能非常有用。它也可以像 CPython 的 python 可执行文件一样工作。
- **mip**：MicroPython 的包安装程序（mip —— “mip installs packages”）。它从 micropython-lib、GitHub 或任意 URL 安装 MicroPython 包。mip 可在具备网络功能的开发板上在设备上使用，也可被 mpremote 等工具在内部使用。
- **mpremote**：用于与 MicroPython 设备交互的工具。
- **.mpy file（.mpy文件）**：交叉编译器的输出。由 py 文件的编译形成，包含 MicroPython 字节码而非 Python 源代码。
- **native（原生的）**：通常指“原生代码”，即目标微控制器的机器码（如 ARM Thumb、Xtensa、x86/x64）。`@native` 装饰器可应用于 MicroPython 函数，为该函数生成原生代码而非字节码，这可能会更快但占用更多 RAM。
- **port**：通常是 MicroPython 移植版的缩写，但也可能指 GPIO 端口。
- **.py file（.py文件）**：包含 Python 源代码的文件。
- **REPL**：“Read, Eval, Print, Loop（读取、求值、打印、循环）”的缩写。这是交互式 Python 提示符，用于调试或测试短代码片段很有用。大多数 MicroPython 开发板通过 UART 提供 REPL，通常是通过主机 (PC) 上的 USB 访问。
- **stream（流）**：也称为“类文件对象”。提供对底层数据的顺序读写访问的 Python 对象。流对象实现相应的接口，包括`read()`、`write()`、`readinto()`、`seek()`、`flush()`、`close()`等方法。流是 MicroPython 中的一个重要概念；许多 I/O 对象实现流接口，因此可以在不同上下文中一致且互换地使用。

  有关微Python中流的更多信息，请参见io模块。

- **UART**：“Universal Asynchronous Receiver/Transmitter（通用异步收发传输器）”的缩写。这是一种通过一对引脚（TX和RX）收发数据的外设。许多开发板提供了至少一个通过 USB 作为串行端口供主机 (PC) 访问。
- **upip**：MicroPython 的一个现已过时的包管理器，灵感来自 CPython 的 pip，但体积更小且功能简化。参见其替代品mip。
- **webrepl**：通过浏览器从网络连接到设备上的 REPL（并传输文件）的一种方式。参见 https://micropython.org/webrepl。
