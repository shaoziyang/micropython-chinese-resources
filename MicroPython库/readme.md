# MicroPython 库

python 以其丰富而强大的库著称，正是因为有了这些库，我们可以快速实现各种功能，而不需要每次都从最底层开始构建。在 MicroPython 上同样也是如此，它除了包含硬件接口（底层硬件驱动）外，还包含了网络、文件系统、json、压缩、数学计算等许多功能，让硬件开发变得简单和轻松。 MicroPython 的库分为系统库和用户库，系统库集成在 MicroPython 固件中，可以直接使用；用户库是用户自己开发或者第三方的库，需要复制到芯片的文件系统中才能使用。

在早期一段时间中，MicroPython 将内置的库称为微库，因为这些库是应用于微控制器上的，和PC上的标准库略有差异，功能有缩减，因此在库的名称前加上了一个字母'u'，比如 ure、uio、ujson 等。但从 v1.21 版开始，为了加强和 CPython 的兼容性和用户程序的通用性，将命名方式和 CPython 统一，取消了 'u' 命名规则（但为了保持和旧版本兼容现在仍然还可以使用旧的名称），正常情况下应当始终导入非 'u' 版本的库，如 utime 变为 time，uasyncio 变为 asyncio。唯一的例外是 uctypes，因为它与 CPython 
的 ctypes 模块不兼容。

MicroPython 系统库按照功能大致可以分为 MicroPython 标准库、 MicroPython 专用库和硬件专属库。MicroPython 标准库基本上和 CPython 中对应库的功能是兼容的或是其子集，提供通用功能；而 MicroPython 专用库提供了硬件接口、网络驱动、文件系统等与具体控制器相关的功能；而硬件专属库则提供了每种不同微控制器的专用功能，如 STM32 的 stm 库，esp8266 的 esp 库，esp32 控制器的 esp32 库等，它们包含了专属硬件更底层的功能接口，比如 flash 读写、协处理器功能。

MicroPython 专用库中最主要的模块是 machine，它包含了大部分的底层硬件功能接口，包括定时器、RTC、串口、I2C、SPI等，每个功能模块都有一个单独的子类，方便引用。在 STM32 上稍有区别，因为历史原因和硬件架构，目前部分硬件功能接口放在硬件专属库 pyb 中，而不是 machine 中，有的功能虽然在 machine 中也有（比如Timer），但是功能和 pyb 中并完全不相同（以后可能会发生变化）。

虽然不同的控制器包含的功能有一定差异，但是 MicroPython 尽可能的提供了统一的接口方式，因此在编程时移植时，比其它语言更加简单方便。

- [MicroPython 标准库](标准库/readme.md)
- [MicroPython 专用库](专用库/readme.md)
- [MicroPython 硬件专属库](硬件专属库/readme.md)
- [第三方库](第三方库/readme.md)
