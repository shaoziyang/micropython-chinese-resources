# pyboard.py 工具

pyboard.py 是一个可在电脑上运行的独立 Python 工具，它能实现以下功能：

- 在 MicroPython 设备上快速运行 Python 脚本或命令。这在开发 MicroPython 程序时非常有用，可快速测试代码，无需向设备复制文件或从设备复制文件。
- 访问设备上的文件系统。这允许你将代码部署到设备上（即使该开发板不支持 USB MSC）。

尽管软件名称为 pyboard.py，但它适用于所有支持原始 REPL 的 MicroPython（包括 STM32、ESP32、ESP8266、NRF）。

可以从 [GitHub](https://github.com/micropython/micropython/blob/master/tools/pyboard.py) 下载其最新版本。它唯一的依赖是 `pyserial` 库，该库可从 PiPy 或系统包管理器安装。

运行 `pyboard.py --help` 会显示以下输出：

```bash
usage: pyboard [-h] [-d DEVICE] [-b BAUDRATE] [-u USER] [-p PASSWORD]
               [-c COMMAND] [-w WAIT] [--follow | --no-follow] [-f]
               [files [files ...]]

在 pyboard 上运行脚本。

位置参数：
  files                 输入文件

可选参数：
  -h, --help            显示此帮助信息并退出
  -d DEVICE, --device DEVICE
                        pyboard 的串行设备或 IP 地址
  -b BAUDRATE, --baudrate BAUDRATE
                        串行设备的波特率
  -u USER, --user USER  telnet 登录用户名
  -p PASSWORD, --password PASSWORD
                        telnet 登录密码
  -c COMMAND, --command COMMAND
                        作为字符串传入的程序
  -w WAIT, --wait WAIT  等待 USB 连接的开发板可用的秒数
  --follow              运行脚本后跟踪输出（无脚本时为默认值）
  -f, --filesystem      执行文件系统操作：cp local:device | cp :device local | cat path | ls [path] | rm path | mkdir path | rmdir path
```

## 在设备上运行命令

这对于测试简短的代码片段或编写与设备交互的脚本非常有用：

```bash
$ pyboard.py --device /dev/ttyACM0 -c 'print(1+1)'
2
```

如果经常与同一设备交互，可以设置环境变量 `PYBOARD_DEVICE`，以此替代 `--device` 命令行选项。例如，以下命令与前面的示例效果相同：
```bash
$ export PYBOARD_DEVICE=/dev/ttyACM0
$ pyboard.py -c 'print(1+1)'
```

类似地，可以使用 `PYBOARD_BAUDRATE` 环境变量来设置 `--baudrate` 选项的默认值。

## 在设备上运行脚本

如果有一个名为 `app.py` 的脚本，想要在设备上运行，可以使用以下命令：

```bash
$ pyboard.py --device /dev/ttyACM0 app.py
```

注意，这并不会将 app.py 实际复制到设备的文件系统中，而只是将代码加载到 RAM 中并执行。程序产生的任何输出都会被显示出来。

如果 app.py 程序没有结束，你需要停止 `pyboard.py`，例如使用 Ctrl-C 组合键。但此时 app.py 程序仍会在 MicroPython 设备上继续运行。


## 访问文件系统

使用 `-f` 标志，支持以下文件系统操作：

- `cat path`：打印设备上文件的内容。
- `cp src [src...] dest`：在设备和本地之间复制文件。
- `ls [path]`：列出目录内容（默认是当前工作目录）。
- `mkdir path`：创建目录。
- `rm path`：删除文件。
- `rmdir path`：删除目录。
- `touch path`：如果文件不存在则创建文件。

`cp` 命令采用类 SSH 的约定来指代本地文件和远程文件。任何以 `:` 开头的路径都会被解释为设备上的路径，否则为本地路径。因此：

```bash
$ pyboard.py --device /dev/ttyACM0 -f cp main.py :main.py
```

会将电脑当前目录下的 main.py 复制到设备上一个名为 main.py 的文件中。文件名可以省略，例如：

```bash
$ pyboard.py --device /dev/ttyACM0 -f cp main.py :
```

与上面的命令效果相同。

更多示例：
```bash
# 将设备上的 main.py 复制到本地电脑
$ pyboard.py --device /dev/ttyACM0 -f cp :main.py main.py

# 同上，但使用 . 代替
$ pyboard.py --device /dev/ttyACM0 -f cp :main.py .

# 将三个文件复制到设备，并保留它们的文件名
$ pyboard.py --device /dev/ttyACM0 -f cp main.py app.py foo.py :

# 从设备上删除一个文件
$ pyboard.py --device /dev/ttyACM0 -f rm util.py

# 打印设备上一个文件的内容
$ pyboard.py --device /dev/ttyACM0 -f cat boot.py
...boot.py 的内容...
```

## 使用 pyboard 库

还可以将 `pyboard.py` 作为一个库使用，来编写与 MicroPython 开发板交互的脚本。

```python
import pyboard
pyb = pyboard.Pyboard('/dev/ttyACM0', 115200)
pyb.enter_raw_repl()
ret = pyb.exec('print(1+1)')
print(ret)
pyb.exit_raw_repl()
```
