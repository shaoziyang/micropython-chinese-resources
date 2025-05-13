# 标准库

python以其丰富而强大的库著称，正是因为有了这些库，我们可以快速实现各种功能，而不需要每次都从最底层开始构建。在micropythhon上同样也是如此，它除了包含硬件接口（底层硬件驱动）外，还包含了网络、文件系统、json、压缩、数学计算等许多功能，让硬件开发变得简单和轻松。micropython的库分为系统库和用户库，系统库集成在micropython固件中，可以直接使用；用户库是用户自己开发或者第三方的库，需要复制到芯片的文件系统中才能使用。

在早期一段时间中，micropython将内置的库称为微库，因为这些库是应用于微控制器上的，和PC上的标准库略有差异，功能有缩减，因此在库的名称前加上了一个字母'u'，比如ure、uio、ujson等。但从v1.21开始，为了加强和 CPython的兼容性和用户程序的通用性，将命名方式和CPython统一，取消了'u'命名规则（但为了保持和旧版本兼容现在仍然还可以使用旧的名称），正常情况下应当始终导入非'u'版本的库，如utime变为time，uasyncio变为asyncio。唯一的例外是uctypes，因为它与CPython
的ctypes模块不兼容。

micropython系统库按照功能大致可以分为硬件接口库和通用库两大类。通用库基本上和CPython中对应库的功能是兼容或是其子集，而硬件接口库和具体的控制器相关，虽然不同的控制器包含的功能有一定区别，但是micropython尽可能使用统一的接口方式，因此在编程时会变得比较容易移植。

硬件接口库中最主要的库是machine，它包含了大部分的底层硬件功能接口，包括定时器、RTC、串口、I2C、SPI等，每个功能模块都有一个单独的子类，方便引用。在STM32上稍有区别，因为历史原因和硬件架构，目前部分硬件功能接口放在pyb库中，而不是machine中，有的功能虽然在machine中也有（比如Timer），但是功能和pyb中并完全不相同。此外每种控制器还有自己的专有库，如STM32的stm库，esp8266的esp库，esp32控制器的esp32库等，它们包含了硬件更底层的功能接口，比如flash读写。

- [exception（异常）](exception（异常）/readme.md)
- [array（数组）](array（数组）/readme.md)
- [builtins（内置函数）](builtins（内置函数）/readme.md)
- [binascii（binary/ASCII 转换）](binascii/readme.md)
- [cmath（复数运算）](cmath（复数运算）/readme.md)
- [collections（集合和容器类型）](collections/readme.md)
- [errno（系统错误码）](errno（系统错误码）/readme.md)
- [gc（垃圾回收）](gc（垃圾回收）/readme.md)
- [gzip（压缩和解压缩）](gzip（压缩和解压缩）/readme.md)
- [hashlib（哈希算法库）](hashlib（哈希算法库）/readme.md)
- [heapq（堆队列算法）](heapq（堆队列算法）/readme.md)
- [io（输入输出流）](io（输入输出流）/readme.md)
- [json（JSON 编码解码）](json/readme.md)
- [marshal（Python 对象序列化）](marshal/readme.md)
- [math（数学函数）](math（数学函数）/readme.md)
- [os（"操作系统"基础服务）](os/readme.md)
- [platform（硬件平台标识）](platform/readme.md)
- [random（随机数）](random/readme.md)
- [re（简单正则表达式）](re/readme.md)
- [select（数据流等待事件）](select/readme.md)
- [socket](socket/readme.md)
- [ssl（SSL/TLS 模块）](ssl/readme.md)