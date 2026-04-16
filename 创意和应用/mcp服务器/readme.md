# MicroPython的MCP服务器

近年来，将LLM与各种工具和系统连接的趋势日益增长。 如果你能好好操作，像微控制器这样的物理设备也会很有趣。 因此，这次我们实现了一个MCP服务器，可以直接通过LLM控制 MicroPython 主板执行代码。

## 能做些什么

通过该MCP服务器，LLM检查MicroPython板上的读写文件、代码执行和串行输出。 关键是你不仅可以继续写代码，还可以通过触摸机器并检查它。

特别有趣的是，可以积累专属硬件的信息。 诸如哪个GPIO连接到什么，温度和湿度传感器的地址是什么，这些信息可以被组织和写入对话中，内容也可以从下一个任务中推断。

## 它是如何运作的？

机制很简单：MCP服务器运行在主机PC上，并通过USB串口与MicroPython板通信。 MCP服务器以工具形式展示各种操作，LLM调用这些工具来操作MicroPython板。

MicroPython 端不需要特殊的常驻程序，基本上基于对 REPL 和文件系统的操作。 这使得现有MicroPython板块的实现变得容易。 我觉得运行MicroPython的主板，比如ESP32和Raspberry Pi Pico，基本能用。

## 相关链接

- [原文（日语）](https://www.switch-science.com/blogs/magazine/mcp-micropython-bridge)
- [mcp-micropython-bridge](https://github.com/SWITCHSCIENCE/mcp-micropython-bridge)
