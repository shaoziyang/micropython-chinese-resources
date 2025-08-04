# MicroPython 远程控制：mpremote

`mpremote` 命令行工具提供了一套集成工具，用于通过串行连接远程交互、管理文件系统以及自动化控制 MicroPython 设备。


## 安装方法

要使用`mpremote`，首先通过`pip`安装：

```bash
$ pip install --user mpremote
```

或通过`pipx`安装：
```bash
$ pipx install mpremote
```

最简单的使用方式是直接调用工具而不带任何参数：
```bash
$ mpremote
```

该命令会自动检测并连接第一个可用的 USB 串行设备，并提供一个交互式终端，用于访问 REPL 和程序输出。串行端口以独占模式打开，因此运行第二个（或第三个等）`mpremote` 实例会连接到后续的串行设备（如果有）。

此外，`pipx` 还允许不安装直接运行 `mpremote`：
```bash
$ pipx run mpremote ...args
```


## 命令

`mpremote` 支持在命令行中输入一系列命令，这些命令会在远程 MicroPython 设备上依次执行。

每个命令的格式为 `<命令名称> [--选项] [参数...]`。对于支持多个参数的命令（例如文件列表），参数列表可以用 `+` 来终止。

如果未指定任何命令，默认命令为 `repl`。此外，如果任何命令需要访问设备，且之前未指定连接方式，则会自动添加 `connect auto`（自动连接）。

为了使设备在执行任何操作命令（`repl`除外）时处于已知状态，`mpremote` 在连接后、运行第一个命令前，会停止所有正在运行的程序并对设备执行软复位。可以使用 `resume` 和 `soft-reset` 命令来控制此行为。有关更多详细信息，请参见“自动连接和自动软复位”。

可以指定多个命令，它们会按顺序运行。

### 命令列表

完整的命令列表如下：

- connect
- disconnect
- resume
- soft_reset
- repl
- eval
- exec
- run
- fs
- df
- edit
- mip
- mount
- unmount
- romfs
- rtc
- sleep
- reset
- bootloader

### 命令说明

- `connect`

  连接到指定设备：

  ```bash
  $ mpremote connect <device>
  ```

  `<device>`可以是：
  - `list`：列出可用设备
  - `auto`：连接到第一个可用的USB串行端口
  - `id:<serial>`：通过USB序列号（`connect list`输出的第二列）连接设备
  - `port:<path>`：通过路径（`connect list`输出的第一列）连接设备
  - `rfc2217://<host>:<port>`：通过TCP串行连接（如基于RFC2217的网络串行端口）
  - 任何有效的设备名称/路径

  💡 注意：无需使用`connect`命令，可直接使用预定义的快捷方式（如Linux的`a0`对应`/dev/ttyACM0`，Windows的`c0`对应`COM0`）。
<br><br>

- `disconnect`

  断开当前设备连接：
  
  ```bash
  $ mpremote disconnect
  ```
  
  断开后，自动软复位功能会启用。
<br><br>

- `resume`
  
  保留后续命令的解释器状态：
  
  ```bash
  $ mpremote resume
  ```
  
  此命令禁用自动软复位，适用于无需软复位即可在设备上运行后续命令的场景。
<br><br>

- `soft-reset`
  
  执行设备的软复位：
  
  ```bash
  $ mpremote soft-reset
  ```
  
  软复位会清除 Python 堆并重启解释器，并阻止后续命令触发自动软复位。
<br><br>

- `repl`

  进入已连接设备的REPL：
  
  ```bash
  $ mpremote repl [--options]
  ```
  
  选项：
  - `--escape-non-printable`：将不可打印字节/字符以十六进制代码显示
  - `--capture <file>`：将REPL会话输出捕获到指定文件
  - `--inject-code <string>`：按下Ctrl-J时在REPL注入指定字符（用于自动化常用命令）
  - `--inject-file <file>`：按下Ctrl-K时在REPL注入指定文件内容（用于运行代码）
  
  运行时，可通过 Ctrl-] 或 Ctrl-x 退出。
  
  💡 注意：`repl` 命令本质是作为终端访问设备，若程序正在运行，需先按 Ctrl-C 中断以进入 REPL。使用 `mpremote soft-reset repl` 可获取清除所有程序状态的“干净”REPL。
  注意：此处的 "REPL" 这一名称体现了该命令的常见用途 —— 访问运行在 MicroPython 设备上的“读取-求值-打印循环”（Read Eval Print Loop）。严格来说，`repl` 命令只是作为一个终端（或“串行监视器”）来访问设备。由于该命令不会触发自动复位行为，这意味着如果当前有程序正在运行，你需要先按 Ctrl-C 将其中断才能进入 REPL，进而访问程序状态。也可以使用 `mpremote soft-reset repl` 来获取一个"干净的"REPL，其中所有程序状态都已被清除。
<br><br>

- `eval`

  计算并打印 Python 表达式的结果：
  
  ```bash
  $ mpremote eval <string>
  ```
  
  示例：`mpremote eval "1 + 2"` 会输出 `3`。
<br><br>

- `exec`

  执行指定的 Python 代码：

  ```bash
  $ mpremote exec <string>
  ```

  默认会显示代码输出直到结束；`--no-follow` 选项可立即返回，让设备在后台运行代码。
<br><br>

- `run`

  运行本地文件系统中的脚本：

  ```bash
  $ mpremote run <file.py>
  ```
  脚本直接在设备的RAM中执行，无需复制到文件系统，适合快速迭代开发。默认显示输出直到结束；`--no-follow`选项可让设备后台运行脚本。
<br><br>

- `fs`

  执行设备的文件系统命令：
  
  ```bash
  $ mpremote fs <sub-command>
  ```
  
  子命令包括：
  - `cat <file..>`：显示设备中一个或多个文件的内容
  - `ls [dirs...]`：列出当前目录或指定目录
  - `cp [-rf] <src...> <dest>`：复制文件（本地路径无前缀，远程路径以`:`开头，如  `cp main.py :main.py`）
  - `rm [-r] <src...>`：删除设备上的文件或文件夹（`-r`递归删除）
  - `mkdir <dirs...>`：在设备上创建目录
  - `rmdir <dirs...>`：删除设备上的目录
  - `touch <file..>`：在设备上创建文件（若不存在）
  - `sha256sum <file..>`：计算设备上文件的SHA256哈希
  - `tree [-vsh] <dirs...>`：以树形结构显示目录（`-s`显示大小，`-h`人类可读格式）
  
  `cp`命令采用一种约定：路径前带冒号`:`表示远程路径，不带冒号则表示本地路径。这一约定基于安全复制协议（SCP）客户端所使用的规范。

  例如，`mpremote fs cp main.py :main.py` 会将当前本地目录下的 `main.py` 复制到远程文件系统，而 `mpremote fs cp :main.py main.py` 则会将设备上的`main.py`复制回当前本地目录。

  `mpremote rm -r` 命令支持相对路径和绝对路径。使用`:`代表当前远程工作目录（cwd），可以从设备的默认路径（如 `/flash`、`/`）中删除目录树。使用 `-v/--verbose` 选项可以查看正在删除的文件。

  示例：
  - `mpremote rm -r :libs` 会从设备中删除 `libs` 目录及其所有子项。
  - `mpremote rm -rv :/sd` 会删除已挂载 SD 卡中的所有文件，并产生一个非阻塞警告。挂载状态会保留。
  - `mpremote rm -rv :/` 会删除设备上的所有文件，包括位于已挂载虚拟文件系统（如`/sd`或`/flash`）中的文件。删除所有文件夹和文件后，该命令还会返回一个错误，以模拟Unix系统中 `rm -rf /` 的行为。

  ⚠️ 警告：对于通过 `mpremote rm -r :` 删除的文件，目前无法恢复，请谨慎使用。

  `tree`命令会打印指定目录的树形结构。使用 `--size/-s` 选项会显示每个文件的大小，使用 `--human/-h` 选项则会采用更易读的格式。注意：只有当设备文件系统报告非零大小时，才会显示目录大小。`-v` 选项可用于在输出中包含串行设备的名称。

  所有其他命令默认假设路径为远程路径，但为清晰起见，也可选择性地添加`:`。

  所有文件系统子命令都接受多个路径参数，因此如果命令序列中还有其他命令，必须使用`+`来终止参数列表，例如：
  
  ```bash
  $ mpremote fs cp main.py :main.py + repl
  ```

  这会将文件复制到设备，然后进入REPL。`+` 的作用是防止 "repl" 被解释为路径。

  `cp` 命令支持 `-r` 选项进行递归复制。默认情况下，如果源文件和目标文件的 SHA256 哈希值匹配，`cp`命令会跳过向远程设备复制文件。若要忽略哈希值强制复制，可使用 `-f` 选项。

  💡 注意：为方便使用，所有文件系统子命令都有对应的常规命令别名，例如，可以使用 `mpremote cp ...`，而不必写成 `mpremote fs cp ...`。
<br><br>

- `df`

  查询设备的存储空间使用情况：
  
  ```bash
  $ mpremote df
  ```
  
  输出类似Unix的`df`命令，显示文件系统的总大小、已用和可用空间。
<br><br>

- `edit`

  编辑设备上的文件：
  
  ```bash
  $ mpremote edit <files...>
  ```
  
  命令会将设备文件复制到本地临时目录，通过环境变量`$EDITOR`指定的编辑器打开；编辑完成后，更新后的文件会复制回设备。
<br><br>

- `mip`

  使用 `mip` 工具从 `micropython-lib` 或 GitHub 安装包：
  
  ```bash
  $ mpremote mip install <packages...>
  ```
<br>

- `mount`

  将本地目录挂载到远程设备：

  ```bash
  $ mpremote mount [options] <local-dir>
  ```
  
  使得远程设备能够将本地主机目录视为自身的文件系统。这在开发过程中非常实用，可避免在处理文件时需要将其复制到设备上的麻烦。

  设备会安装一个文件系统驱动程序，该驱动程序随后会作为 `/remote` 挂载到设备的虚拟文件系统（VFS）中，它通过与 `mpremote` 的串行连接作为辅助通道来访问文件。设备的当前工作目录（通过`os.chdir`设置）会被切换到 `/remote`，因此在挂载生效期间，导入操作和文件访问都会指向该目录，而非默认的文件系统路径。

  注意：如果 `mount` 命令之后没有其他操作命令，序列末尾会自动添加 `repl` 命令。

  使用过程中，按 Ctrl-D 会像往常一样触发软复位，但挂载会自动重新连接。不过，如果设备启动时运行了 `main.py`，则无法重新挂载。这种情况下，可以使用原始模式软重启：按 Ctrl-A Ctrl-D 进行重启，然后按 Ctrl-B 返回正常的 REPL，此时挂载即可就绪。

  选项包括：
  - `-l, --unsafe-links`：默认情况下，如果设备访问的文件或目录位于已挂载本地目录之外（向上一个或多个目录层级），会引发错误。此选项会禁用对符号链接的该检查，允许设备跟随本地目录之外的符号链接。
<br><br>

- `unmount`

  从远程设备卸载本地目录：

  ```bash
  $ mpremote unmount
  ```

  `mpremote` 终止时会自动卸载，也可在命令序列中用于提前卸载。
<br><br>

- `romfs`

  管理设备上的 ROMFS 分区：

  ```bash
  $ mpremote romfs <sub-command>
  ```

  子命令：
  - `romfs query`：列出所有可用ROMFS分区及大小
  - `romfs [-o <output>] build <source>`：从源目录创建ROMFS镜像（默认输出为`source.romfs`）
  - `romfs [-p <partition>] deploy <source>`：将ROMFS镜像部署到设备（源为目录时自动创建临时镜像）

  `build`（构建）和 `deploy`（部署）子命令均支持`-m/--mpy`选项，在创建 ROMFS 镜像时会自动将 `.py` 文件编译为 `.mpy` 文件。此选项默认启用，但仅在已安装 `mpy_cross` Python包（例如通过 `pip install mpy_cross` 安装）的情况下生效。如果未安装该包，会打印警告信息，且 `.py` 文件将保持原样。使用 `--no-mpy` 选项可禁用`.py`文件的编译功能。
<br><br>

- `rtc`

  设置/获取设备时钟（RTC）：

  ```bash
  $ mpremote rtc  # 查询当前时间（返回datetime元组）
  $ mpremote rtc --set  # 将设备时间设为主机当前时间
  ```
<br>

- `sleep`

  延迟执行下一个命令：

  ```bash
  $ mpremote sleep <seconds>
  ```

  示例：`mpremote sleep 0.5` 暂停0.5秒。这会将命令序列的执行暂停指定的秒数，例如等待设备执行某项操作。
<br><br>

- `reset`

  硬复位设备：

  ```bash
  $ mpremote reset
  ```

  等价于`machine.reset()`。
<br><br>

- `bootloader`

  让设备进入引导加载模式 (bootloader)：

  ```bash
  $ mpremote bootloader
  ```

  引导加载程序因硬件和开发板而异（如 stm32 的 DFU、rp2040/Pico 的 UF2）。


## 自动连接和软复位

若未显式指定连接命令，`mpremote` 会在工具启动和结束时自动连接/断开第一个可用 USB 串行设备。

连接设备后，`mpremote` 会在执行 `mount`、`eval`、`exec`、`run`、`fs` 等命令时自动软复位（首次执行时），确保代码在干净环境中运行。`disconnect` 后会重新启用自动软复位。

`resume` 命令可禁用自动软复位，`soft-reset` 可显式触发软复位。


## 快捷方式

可通过配置文件 `.config/mpremote/config.py` 定义快捷方式，文件中需定义字典 `commands`，键为快捷方式名称，值为命令字符串或列表。

内置快捷方式：

- `devs`：等价于 `connect list`
- `a0/a1/a2/a3`：对应 `connect /dev/ttyACMn`（Linux）
- `u0/u1/u2/u3`：对应 `connect /dev/ttyUSBn`（Linux）
- `c0/c1/c2/c3`：对应 `connect COMn`（Windows）
- `cat`/`edit`/`ls`等：等价于 `fs <sub-command>`

可以在用户配置文件 (`.config/mpremote/config.py`) 中定义额外的快捷方式。此文件应定义一个名为`commands`的字典。该字典的键为快捷方式，值可以是字符串或字符串列表：

```python
"c33": "connect id:334D335C3138",
```

命令 `c33` 会被替换为 `connect id:334D335C3138`。

```python
"test": ["mount", ".", "exec", "import test"],
```

命令`test`会被替换为 `mount . exec "import test"`。

快捷方式还可以接受参数。例如：

```python
"multiply x=4 y=7": "eval x*y",
```

运行`mpremote multiply 3 7`会在设备上设置`x`和`y`为变量，然后计算表达式`x*y`的值。

`config.py`的示例可能如下所示：

```python
commands = {
    "c33": "connect id:334D335C3138",  # 通过ID连接到特定设备。
    "bl": "bootloader",  # bootloader的较短别名。
    "double x=4": "eval x*2",  # x是参数，默认值为4
    "wl_scan": ["exec", """
import network
wl = network.WLAN()
wl.active(1)
for ap in wl.scan():
    print(ap)
""",],  # 打印附近的WiFi网络。
    "wl_ipconfig": [
        "exec",
        "import network; sta_if = network.WLAN(network.WLAN.IF_STA); print(sta_if.ipconfig('addr4'))",
    ],  # 打印station接口的IP地址。
    "test": ["mount", ".", "exec", "import test"],  # 挂载当前目录并运行test.py。
    "demo": ["run", "path/to/demo.py"],  # 在设备上执行demo.py。
}
```


## 示例

- 连接第一个设备并进入 REPL：

  ```bash
  mpremote
  ```
<br>

- 连接 `/dev/ttyACM1`（Linux）并进入 REPL：

  ```bash
  mpremote a1
  ```
<br>

- 连接 `COM1`（Windows）并进入 REPL：

  ```bash
  mpremote c1
  ```
<br>

- 连接到指定设备并进入 REPL：

  ```bash
  mpremote connect /dev/ttyUSB0
  ```
<br>

- 连接到 `/dev/ttyACM1`，并进入运行命令 `ls`：

  ```bash
  mpremote a1 ls
  ```

  它等效于命令 `mpremote connect /dev/ttyACM1 fs ls`。
<br><br>

- 执行Python命令查看内存信息：

  ```bash
  mpremote exec "import micropython; micropython.mem_info()"
  ```
<br>

- 依次计算每个表达式并打印结果。

  ```bash
  mpremote eval 1/2 eval 3/4
  ```  
<br>

- 在`/dev/ttyACM0`设备上计算`1/2`，然后在`/dev/ttyACM1`设备上计算`3/4`，并分别打印结果。

  ```bash
  mpremote a0 eval 1/2 a1 eval 3/4
  ```
<br>

- 连接设备但不触发软复位，执行 `print_state_info()` 函数（例如，用于查看当前程序状态信息），然后触发软复位。

  ```bash
  mpremote resume exec "print_state_info()" soft-reset
  ```
<br>

- 对设备执行硬复位，等待500毫秒直至设备可用，然后进入引导加载程序。

  ```
  mpremote reset sleep 0.5 bootloader`  
  ```
<br>

- 更新设备上的 `utils/driver.py` 文件，然后在设备上执行本地的 `test.py` 脚本。`test.py` 不会复制到设备的文件系统，而是从 RAM 中直接运行。

  ```bash
  mpremote cp utils/driver.py :utils/driver.py + run test.py
  ```
<br>

- 更新设备上的 `utils/driver.py` 文件，然后在设备上执行 `app.py`。 

  这是一种常见的开发流程：更新单个文件后重新启动程序。在这种情况下，设备上的 `main.py` 通常也会执行 `import app`。

  ```bash
  mpremote cp utils/driver.py :utils/driver.py + exec "import app"
  ```
<br>

- 更新设备上的 `utils/driver.py` 文件，然后触发软复位以重启程序，再通过 `repl` 命令监控输出。

  ```bash
  mpremote cp utils/driver.py :utils/driver.py + soft-reset repl
  ```
<br>

- 与上述命令类似，但首先更新整个 `utils` 目录。
  
  ```bash
  mpremote cp -r utils/ :utils/ + soft-reset repl
  ```
<br>

- 将当前本地目录挂载到设备的 `/remote` 路径，并启动一个以 `/remote` 为工作目录的REPL会话。

  ```bash
  mpremote mount .
  ```
<br>

- 挂载当前本地目录后，从挂载目录中执行`demo.py`。

  ```bash
  mpremote mount . exec "import demo"
  ```
<br>

- 将本地 `app` 目录挂载为设备的 `/remote` 后，从主机当前目录执行本地的 `test.py`，且不将其复制到设备文件系统。

  ```bash
  mpremote mount app run test.py
  ```
<br>

- 挂载当前本地目录后，每次按下 Ctrl-J 时，都会从挂载目录执行 `demo.py`。在按下 Ctrl-J 重新导入 `demo.py` 之前，需要先按下 Ctrl-D 重置解释器状态（这会保留挂载状态）。

  ```bash
  mpremote mount . repl --inject-code "import demo"
  ```
<br>

- 与上述命令类似，但每次按下 Ctrl-K 时，会在 REPL 中执行本地文件 `demo.py` 的内容。同样，需先按下Ctrl-D重置解释器状态。

  ```bash
  mpremote mount app repl --inject-file demo.py
  ```
<br>

- 显示设备上 `boot.py` 文件的内容。

  ```bash
  mpremote cat boot.py
  ```
<br>

- 使用本地的 `$EDITOR` 编辑器编辑设备上的 `utils/driver.py` 文件。

  ```bash
  mpremote edit utils/driver.py
  ```
<br>

- 将设备上的 `main.py` 复制到本地目录。

  ```bash
  mpremote cp :main.py .
  ```
<br>

- 将本地目录的 `main.py` 复制到设备。

  ```bash
  mpremote cp main.py :
  ```  
<br>

- 将设备上的 `a.py` 复制为设备上的 `b.py`。

  ```bash
  mpremote cp :a.py :b.py
  ```
<br>

- 将本地目录 `dir` 递归复制到远程设备。

  ```bash
  mpremote cp -r dir/ :
  ```
<br>

- 将本地目录的 `a.py` 和 `b.py` 复制到设备，然后运行 `repl` 命令。

  ```bash
  mpremote cp a.py b.py : + repl
  ```
<br>

- 从 `micropython-lib` 向设备安装 `aioble` 包。

  ```bash
  mpremote mip install aioble
  ```
<br>

- 从 GitHub 上 `org/repo` 的指定分支向设备安装包。

  ```bash
  mpremote mip install github:org/repo@branch
  ```
<br>

- 从 GitLab 上 `org/repo` 的指定分支向设备安装包。

  ```bash
  mpremote mip install gitlab:org/repo@branch
  ```
<br>

- 从 `micropython-lib` 向设备的 `/flash/third-party` 目录安装 `functools` 包。

  ```bash
  mpremote mip install --target /flash/third-party functools
  ```
