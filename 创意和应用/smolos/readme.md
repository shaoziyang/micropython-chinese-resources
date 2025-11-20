# smolOS

smolOS 是一个微型（体积小于 20KB，代码行数少于 500 行）且简单的 🧪 研究型 ⚙️ 操作系统 ⌨️，它使用 🐍 MicroPython 编写，为微控制器提供类似 POSIX 的 📁 环境，方便用户进行体验。它附带了一套 🧰 实用工具和演示程序。

**Shell**：它既不等同于 MS-DOS 系统的 （command.com），也不是类 bash 的命令行工具，而是介于两者之间的存在。我希望所有命令都能做到对用户友好。例如，没有采用 UNIX 系统中的 ls 命令或 DOS 系统中的 dir 命令，而是选择了 list 命令。在这三个命令里，只有最后一个（ list）能准确表达它的功能 —— 列出文件。
**文件**：扁平的层次结构。微控制器充其量只有几兆字节（MB）的数据存储空间，它并非用于存储文档的设备，而是用来存放少量脚本以运行我们项目的地方。设置目录会让事情变得更复杂：用户会搞不清自己当前处于哪个位置，而且输入 “..” 返回上一级的操作向来都显得别扭。这就是为什么（smolOS）没有 mkdir（创建目录）或 cd（切换目录）命令的原因。


系统应该能在任何 MicroPython 支持的板上运行（实测在esp32和stm32上需要进行少量修改），但它仅在 Seeed XIAO RP2040 上进行了测试和开发。整个系统都在一个文件 smolos.py 中。还有一些可选的工具和演示。
- 主系统文件
- 文本编辑器（包含）
- 额外工具和演示（可选）

## 硬件支持
- 任何 MicroPython 开发板
- XIAO RP2040
- NeoPixel 5x5 网格（显示屏）
- 光敏电阻（输入）
- NeoPixel LED（输出）
- 蜂鸣器（声音）

## smolOS 特性
- 将微控制器变成小型工作电脑
- 为乐趣和学习而设计
- 超小且快速
- 易于使用，类似 MS-DOS、类 POSIX 环境
- 可列出和操作文件
- 包含文本编辑器（功能基础但够用）
- 包含基础工具和演示程序（适用于 NeoPixel、蜂鸣器、发光二极管等）
- 基于 MicroPython 构建，代码清晰
- 代码遵循的主要原则是稳定性和简洁性
- 免费且开源 :)

## 附加程序
- `ansi.py` - 显示 ANSI 转义码
- `life.py` - 适用于 smolOS 的《生命游戏》实现（文本版）
- `buzz.py` - 用于 1 位音乐的简单合成器（需要蜂鸣器）
- `bytebeat.py` - 适用于蜂鸣器的 ByteBeat 实现
以下所有程序均适用于 5x5 网格的 NeoPixel BFF：
- `duck.py` - 程序员专用黄色橡皮鸭
- `neolife.py` - 《生命游戏》实现（适用于 NeoPixel 网格）
- `pixel.py` - 用于操作单个发光二极管的工具
- `plasma.py` - 演示场景的等离子效果
- `font.py` - 字体位图（用于滚动显示）
- `scroller.py` - 滚动显示文本

## 系统安装
- 从 [github](https://github.com/w84death/smolOS/%20) 或者 [smolOS](https://smol.p1x.in/os/smolOS-main.zip) 网站下载源码
- 将文件 smolos.py 和 main.py 复制到系统
- 重启系统


## 运行
- 自动运行
	复制 smolOS 中的 smolos.py 和 main.py 文件到系统，或者在 main.py 中设置下面的代码，每次复位/上电后，就会自动进入 smolOS。
- 手动运行
	在 repl 或者程序中，用下面代码运行 smolOS。
	```py
	from smolos import smolOS
    os = smolOS()
    os.boot()
	```

## 使用说明
运行后，将先显示系统信息，然后进入系统提示 smol $:。在提示符后可以输入命令，就像 linux 的 shell 一样。
![](smolos.png)

输入 help，就会显示快速帮助：
![](help.png)

使用 list 命令，显示当前目录下的文件列表，类似 linux 的 ls 命令：
![](list.png)

edit 文件名，编辑一个文件
![](edit.png)

其它命令：
- `.`，重复上一条命令
- `print 文件名`，显示文件内容
- `info 文件名`，显示文件信息
- `remove 文件名`，删除文件
- `turbo`，切换高性能/低性能模式
- `stats`，显示系统信息
- `led 命令` ，led 控制
	- `on`，亮灯
	- `off`，关灯
	- `boot`，led 闪烁 4 次
- `free`，显示剩余内存
- `exe code`，执行 code 中的代码
- `文件名`（不要加 .py 后缀），运行一个外部 python 文件（相当于 execfile('文件名.py')）

## 相关链接
- 网站：https://smol.p1x.in/os 
- 源码：
	- https://smol.p1x.in/os/smolOS-main.zip 
	- https://github.com/w84death/smolOS
- [快速安装指南](https://smol.p1x.in/os/#install)
- [系统使用方法](https://smol.p1x.in/os/#usage)

## 参考文章
- [hackster.io](https://www.hackster.io/news/krzysztof-jankowski-s-micropython-based-smolos-puts-a-tiny-posix-like-environment-on-your-esp8266-0c776559152b)
- [cnx-software.com](https://www.cnx-software.com/2023/07/12/smolos-brings-a-linux-like-command-line-interface-to-esp8266-microcontroller/)
- [lobste.rs](https://lobste.rs/s/ipztxc/smolos_small_os_for_micropython_on)

## 附注
- 目前 smolOS 可以在 rp2040/rp2350 上直接运行，其它芯片上，可能需要适当修改源码，例如 `stats()` 函数中 `machine.freq()` 部分、文件前面的 `CPU_SPEED_SLOW`/`CPU_SPEED_TURBO`/`SYSTEM_LED_PIN` 等定义。
- `stats()` 函数中显示的系统信息，部分也需要在文件前面的 `OS_NAME`/`OS_VARIANT` 等中修改。
- 如果希望为命令添加别名，在 `__init__()` 函数中的 `self.user_commands` 部分，在命令定义后添加命令别名，如：`"list": self.list, "ls": self.list,`。
- 如果希望添加新的命令，在上面 `self.user_commands` 部分中，添加新的命令字符串和对应的函数。




