# 交互式解释器模式（又名REPL）

介绍 MicroPython 交互式解释器模式的一些特点，常用术语为REPL（读取-求值-打印-循环）。

## 自动缩进

当输入以冒号结尾的Python语句（例如 if、for、while）时，提示符会变为三个点（`...`），光标会自动缩进4个空格。按下回车键后，对于常规语句，下一行会保持相同的缩进级别；在合适的情况下，会增加一个缩进级别。按下退格键会取消一个级别的缩进。

如果光标一直回退到起始位置，按下回车键将执行你输入的代码。以下示例展示了输入 for 语句后的显示效果（下划线表示光标最终位置）：

```py
>>> for i in range(30):
...     _
```

如果接着输入 if 语句，会自动增加一个缩进级别：
```py
>>> for i in range(30):
...     if i > 3:
...         _
```

现在输入 break 并按下回车键，再按退格键：
```py
>>> for i in range(30):
...     if i > 3:
...         break
...     _
```

最后输入 print(i)，按下回车键，按退格键，再按回车键：
```py
>>> for i in range(30):
...     if i > 3:
...         break
...     print(i)
...
0
1
2
3
>>>
```

如果前两行全是空格，则不会应用自动缩进。这意味着你可以通过按两次回车键结束复合语句的输入，再按第三次回车键即可完成并执行该语句。


## 自动补全

在REPL中输入命令时，如果当前已输入的内容是某个名称的开头，按下 `Tab` 键会显示可能的补全选项。例如，首先输入 `import machine` 并按下回车键导入 machine 模块。然后输入`m`并按下 Tab 键，它会扩展为 `machine`。输入一个点 `.` 后再次按下 Tab 键，你会看到类似下面的内容：

```py
>>> machine.
__name__  info  unique_id  reset
bootloader  freq  rng  idle
sleep  deepsleep  disable_irq  enable_irq
Pin
```

在存在多个可能的补全选项之前，输入的内容会尽可能地扩展。例如，输入 `machine.Pin.AF3` 并按下 Tab 键，它会扩展为 `machine.Pin.AF3_TIM`。再次按下 Tab 键会显示所有可能的扩展选项：
```py
>>> machine.Pin.AF3_TIM
AF3_TIM10  AF3_TIM11  AF3_TIM8  AF3_TIM9
>>> machine.Pin.AF3_TIM
```


## 中断运行中的程序

可以通过按下 Ctrl+C 来中断运行中的程序。这会触发一个 `KeyboardInterrupt` 异常，若程序没有捕获 `KeyboardInterrupt` 异常，就会返回到 REPL 界面。

例如：
```py
>>> for i in range(1000000):
...     print(i)
...
0
1
2
3
...
6466
6467
6468
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
KeyboardInterrupt:
>>>
```

## 粘贴模式

如果需要将一些代码粘贴到终端窗口，自动缩进功能可能会把格式弄乱。例如，假设有以下 Python 代码：

```py
def foo():
    print('This is a test to show paste mode')
    print('Here is a second line')
foo()
```

若尝试将其粘贴到普通的REPL中，会看到类似下面的结果：

```py
>>> def foo():
...         print('This is a test to show paste mode')
...             print('Here is a second line')
...             foo()
...
Traceback (most recent call last):
  File "<stdin>", line 3
IndentationError: unexpected indent
```

按下 Ctrl+E，将进入粘贴模式，它会关闭自动缩进功能，并将提示符从 `>>>` 改为 `===`。例如：
```py
>>>
paste mode; Ctrl-C to cancel, Ctrl-D to finish
=== def foo():
===     print('This is a test to show paste mode')
===     print('Here is a second line')
=== foo()
===
This is a test to show paste mode
Here is a second line
>>>
```

粘贴模式允许粘贴空行。粘贴的文本会像文件一样被编译。按下Ctrl+D可退出粘贴模式并开始编译。

## 软复位

软复位会重置 Python 解释器，但会尽量不重置与 MicroPython 开发板的连接方式（USB 串口或 Wi-Fi）。

可以在 REPL 中按下 Ctrl+D 执行软复位，也可以在 Python 代码中通过执行以下命令进行软复位：

```py
machine.soft_reset()
```

例如，如果重置了 MicroPython 开发板，然后执行 `dir()` 命令，会看到类似这样的结果：
```py
>>> dir()
['__name__', 'pyb']
```

现在创建一些变量，然后重复`dir()`命令：
```py
>>> i = 1
>>> j = 23
>>> x = 'abc'
>>> dir()
['j', 'x', '__name__', 'pyb', 'i']
>>>
```

现在如果按下 Ctrl+D，再重复`dir()`命令，会发现之前创建的变量已经不存在了：
```py
MPY: sync filesystems
MPY: soft reboot
MicroPython v1.5-51-g6f70283-dirty on 2015-10-30; PYBv1.0 with STM32F405RG
Type "help()" for more information.
>>> dir()
['__name__', 'pyb']
>>>
```


## 特殊变量 _（下划线）

在使用 REPL 时，可能会进行计算并查看结果。MicroPython 会将上一条语句的结果存储在变量 `_`（下划线）中。因此，你可以使用这个下划线将结果保存到一个变量中。例如：
```py
>>> 1 + 2 + 3 + 4 + 5
15
>>> x = _
>>> x
15
>>>
```


## 原始模式和原始粘贴模式

原始模式（也称为原始 REPL）通常不是供人直接使用的。它专为程序使用而设计，其行为本质上类似于关闭回显的粘贴模式，并具有可选的流控制功能。

通过 Ctrl+A 进入原始模式。然后发送 Python 代码，接着按下 Ctrl+D。设备会以 “OK” 确认收到 Ctrl+D，随后 Python 代码将被编译并执行。任何输出（或错误）都会被发送回来。按下 Ctrl+B 将退出原始模式，返回到常规（也称为友好）REPL。

原始粘贴模式是原始 REPL 中的一种附加模式，它包含流控制功能，并在接收代码时进行编译。这使得向设备高速传输代码时更加稳定，而且在接收时使用的 RAM 更少，因为它不需要像标准原始模式那样在编译前存储代码的完整副本。

原始粘贴模式使用以下协议：
1. 像往常一样通过 Ctrl+A 进入原始REPL。
2. 发送3个字节：`b"\x05A\x01"`（即 Ctrl+E，然后是 “A”，然后是Ctrl+A）。
3. 读取2个字节以确定设备是否进入了原始粘贴模式：
   - 如果结果是`b"R\x00"`，则设备理解该命令但不支持原始粘贴。
   - 如果结果是`b"R\x01"`，则设备支持原始粘贴并已进入该模式。
   - 否则，结果应为`b"ra"`，且设备不支持原始粘贴，此时应读取并丢弃字符串`b"w REPL; CTRL-B to exit\r\n>"`。
4. 如果设备处于原始粘贴模式，则继续；否则回退到标准原始模式。
5. 读取2个字节，这是流控制窗口大小增量（以字节为单位），存储为16位无符号小端整数。剩余窗口大小变量的初始值应设为该数值。
6. 向设备写入代码：
   - 当还有字节要发送时，写入最多不超过剩余窗口大小的字节，并将剩余窗口大小减去已写入的字节数。
   - 如果剩余窗口大小为0，或者有等待读取的字节，则读取1个字节。如果该字节是`b"\x01"`，则将剩余窗口大小增加步骤5中的窗口大小增量。如果该字节是`b"\x04"`，则设备希望结束数据接收，应向设备写入`b"\x04"`，且之后不再发送任何代码。（注意：如果有来自设备的字节等待读取，不必立即读取并处理，只要剩余窗口大小大于0，设备就会继续接收传入的字节。）
7. 当所有代码都已写入设备后，写入`b"\x04"`以表示数据结束。
8. 从设备读取数据，直到收到`b"\x04"`。此时设备已接收并编译了所有发送的代码，并正在执行。
9. 设备会输出执行代码所产生的所有字符。当（如果）代码执行完毕，会输出b"\x04"，随后是任何未捕获的异常，之后再次输出b"\x04"。然后，设备会返回到标准的原始 REPL，并输出b">"。

例如，在常规（友好）REPL的新行开始，如果发送：
```py
b"\x01\x05A\x01print(123)\x04"
```

那么设备会返回类似这样的内容：
```py
b"\r\nraw REPL; CTRL-B to exit\r\n>R\x01\x80\x00\x01\x04123\r\n\x04\x04>"
```

按时间分解如下：
```
# 步骤1：进入原始REPL
发送：b"\x01"
接收：b"\r\nraw REPL; CTRL-B to exit\r\n>"

# 步骤2-5：进入原始粘贴模式
发送：b"\x05A\x01"
接收：b"R\x01\x80\x00\x01"

# 步骤6-8：写入代码
发送：b"print(123)\x04"
接收：b"\x04"

# 步骤9：代码执行并读取结果
接收：b"123\r\n\x04\x04>"
```

在这种情况下，流控制窗口大小增量为128，开始时立即有两个窗口的数据可用，一个来自初始窗口大小增量值，另一个来自发送的显式`b"\x01"`值。这意味着开始时最多可以写入256字节，之后再等待或检查更多传入的流控制字符。

`tools/pyboard.py` 软件使用原始 REPL（包括原始粘贴模式），在支持 MicroPython 的开发板上执行 Python 代码。

