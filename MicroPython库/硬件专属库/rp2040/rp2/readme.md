# rp2（特定于 RP2040 的功能）

rp2 模块包含特定于 Raspberry Pi Pico 中使用的 RP2040 的函数和类。

有关更多信息，请参见《RP2040 Python 数据手册》，以及 pico-micropython-examples 中的示例代码。

## 与 PIO 相关的函数

rp2 模块包含用于汇编 PIO 程序的函数。

有关运行 PIO 程序的信息，请参见 `rp2.StateMachine`。

- rp2.`asm_pio`(*, out_init=None, set_init=None, sideset_init=None, side_pindir=False, in_shiftdir=PIO.SHIFT_LEFT, out_shiftdir=PIO.SHIFT_LEFT, autopush=False, autopull=False, push_thresh=32, pull_thresh=32, fifo_join=PIO.JOIN_NONE)

  汇编一个 PIO 程序。

  以下参数控制 GPIO 引脚的初始状态，取值为 `PIO.IN_LOW`、`PIO.IN_HIGH`、`PIO.OUT_LOW` 或 `PIO.OUT_HIGH` 之一。如果程序使用多个引脚，请提供一个元组，例如 `out_init=(PIO.OUT_LOW, PIO.OUT_LOW)`。

  - `out_init`：配置用于 `out()` 指令的引脚。
  - `set_init`：配置用于 `set()` 指令的引脚。最多可配置 5 个。
  - `sideset_init`：配置用于 `.side()` 修饰符的引脚。最多可配置 5 个。
  - `side_pindir`：当设置为 True 时，配置 `.side()` 修饰符用于引脚方向，而不是引脚值（默认值为 False 时）。

  以下参数为默认使用，但可在 `StateMachine.init()` 中重写：
  - `in_shiftdir`：ISR 移位的默认方向，为 `PIO.SHIFT_LEFT` 或 `PIO.SHIFT_RIGHT` 之一。
  - `out_shiftdir`：OSR 移位的默认方向，为 `PIO.SHIFT_LEFT` 或 `PIO.SHIFT_RIGHT` 之一。
  - `push_thresh`：触发自动推送或条件重新推送之前的位阈值。
  - `pull_thresh`：触发自动拉取或条件重新拉取之前的位阈值。

  其余参数如下：
  - `autopush`：配置是否启用自动推送。
  - `autopull`：配置是否启用自动拉取。
  - `fifo_join`：配置是否将 4 字的 TX 和 RX FIFO 合并为一个仅用于单方向的 8 字 FIFO。选项包括 `PIO.JOIN_NONE`、`PIO.JOIN_RX` 和 `PIO.JOIN_TX`。
<br><br>

- rp2.`asm_pio_encode`(instr, sideset_count, sideset_opt=False)

  汇编单条 PIO 指令。通常应使用 `asm_pio()` 代替。

  ```python
  >>> rp2.asm_pio_encode("set(0, 1)", 0)
  57345
  ```
<br>

- rp2.`bootsel_button`()

  临时将 QSPI_SS 引脚变为输入并读取其值，低电平返回 1，高电平返回 0。在带有 BOOTSEL 按钮的典型 RP2040 开发板上，返回值为 1 表示按钮被按下。

  由于此函数会临时禁用对外部闪存的访问，因此它还会临时禁用中断和另一个内核，以防止它们尝试从闪存执行代码。
<br><br>

- class rp2.`PIOASMError`

  如果汇编 PIO 程序时出错，`asm_pio()` 或 `asm_pio_encode()` 会引发此异常。

## PIO 汇编语言指令

PIO 状态机是通过一种自定义汇编语言编程的，该语言包含 9 条核心 PIO 机器指令。在 MicroPython 中，PIO 汇编程序编写为带有 `@rp2.asm_pio()` 装饰器的 Python 函数，并且它们使用 Python 语法。此类程序支持标准的 Python 变量和算术运算，以及以下用于编码 PIO 指令和指导汇编器的自定义函数。有关更多详细信息，请参见《RP2040 数据手册》第 3.4 节。

- `wrap_target`()

  指定程序换行后继续执行的位置。默认情况下，这是 PIO 程序的开头。
<br><br>

- `wrap`()

  指定程序结束并换行的位置。如果不使用此指令，它会自动添加到 PIO 程序的末尾。换行不会消耗任何执行周期。
<br><br>

- `label`(label)

  在当前位置定义一个名为 `label` 的标签。`label` 可以是字符串或整数。
<br><br>

- `word`(instr, label=None)

  在汇编输出中插入一个任意的 16 位字。
  - `instr`：16 位值
  - `label`：如果提供，查找标签并将标签的值与 `instr` 进行逻辑或运算
<br><br>

- `jmp`(…)

  该指令有两种形式：

  - `jmp`(label)
    - `label`：无条件跳转到的标签
  - `jmp`(cond, label)
    - `cond`：要检查的条件，可为以下之一：
      - not_x、not_y：如果寄存器为零则为真
      - x_dec、y_dec：如果寄存器非零则为真，并执行后减操作
      - x_not_y：如果 X 不等于 Y 则为真
      - pin：如果输入引脚被置位则为真
      - not_osre：如果 OSR 非空（尚未达到其阈值）则为真
    - `label`：如果条件为真，则跳转到的标签
<br><br>

- `wait`(polarity, src, index)

  阻塞，等待引脚或 IRQ 线上的高/低电平。
  - `polarity`：0 或 1，等待低电平还是高电平
  - `src`：可为 gpio（绝对引脚）、pin（相对于状态机 in_base 参数的引脚）、irq 之一
  - `index`：0-31，src 的索引
<br><br>

- `in_`(src, bit_count)

  将数据从 src 移入 ISR。
  - `src`：可为 pins、x、y、null、isr、osr 之一
  - `bit_count`：要移入的位数（1-32）
<br><br>

- `out`(dest, bit_count)

  将数据从 OSR 移出到 dest。
  - `dest`：可为 pins、x、y、pindirs、pc、isr、exec 之一
  - `bit_count`：要移出的位数（1-32）
<br><br>

- `push`(…)

  将 ISR 推入 RX FIFO，然后将 ISR 清零。该指令有以下形式：
  - push()
  - push(block)
  - push(noblock)
  - push(iffull)
  - push(iffull, block)
  - push(iffull, noblock)

  如果使用 block，则当 RX FIFO 已满时，该指令会停滞。默认情况下会阻塞。如果使用 iffull，则仅当输入移位计数达到其阈值时才推送。
<br><br>

- `pull`(…)

  从 TX FIFO 拉取数据到 OSR。该指令有以下形式：
  - pull()
  - pull(block)
  - pull(noblock)
  - pull(ifempty)
  - pull(ifempty, block)
  - pull(ifempty, noblock)

  如果使用 block，则当 TX FIFO 为空时，该指令会停滞。默认情况下会阻塞。如果使用 ifempty，则仅当输出移位计数达到其阈值时才拉取。
<br><br>

- `mov`(dest, src)

  将 src 的值移动到 dest 中。
  - dest：可为 pins、x、y、exec、pc、isr、osr 之一
  - src：可为 pins、x、y、null、status、isr、osr 之一；此参数可以选择通过用 invert() 或 reverse() 包装来修改（但不能同时使用两者）
<br><br>

- `irq`(…)

  设置或清除 IRQ 标志。该指令有两种形式：

  - `irq`(index)
    - `index`：0-7，或 rel(0) 到 rel(7)
  - `irq`(mode, index)
    - `mode`：可为 block、clear 之一
    - `index`：0-7，或 rel(0) 到 rel(7)

  如果使用 block，则该指令会停滞，直到标志被另一个实体清除。如果使用 clear，则清除标志而不是设置标志。相对 IRQ 索引通过模 4 加法将状态机 ID 添加到 IRQ 索引中。IRQ 0-3 对处理器可见，4-7 是状态机内部的。
<br><br>

- `set`(dest, data)

  用 data 的值设置 dest。
  - `dest`：pins、x、y、pindirs
  - `data`：值（0-31）
<br><br>

- `nop`()

  这是一条伪指令，汇编为 mov(y, y)，没有副作用。
<br><br>

- `.side`(value)

  这是一个修饰符，可以应用于任何指令，用于控制侧置引脚的值。
  - `value`：要在侧置引脚上输出的值（位）
<br><br>

- `.delay`(value)

  这是一个修饰符，可以应用于任何指令，指定指令执行后延迟的周期数。
  - `value`：延迟的周期数，0-31（如果使用侧置引脚，最大值会减小）
<br><br>

- [value]

  这是一个修饰符，等同于 .delay(value)。


## 类

### class DMA – 访问RP2040的DMA控制器

`DMA` 类提供对 RP2040 直接内存访问（DMA）控制器的访问，能够在内存块和/或IO寄存器之间移动数据。DMA 控制器在总线结构上有独立的读写总线主控制器连接，每个 DMA 通道可以独立地从一个地址读取数据并写入到另一个地址，还可以选择性地递增一个或两个指针，从而允许它在处理器执行其他任务或进入低功耗状态时代表处理器执行传输。RP2040 的 DMA 控制器有 12 个独立的 DMA 通道，可以同时运行。有关 RP2040 DMA系统的完整详细信息，请参阅 RP2040 数据手册的第 2.5 节。

**示例**

DMA控制器最简单的用法是将数据从一个内存块移动到另一个内存块。这可以通过以下代码实现：

```python
a = bytearray(32*1024)
b = bytearray(32*1024)
d = rp2.DMA()
c = d.pack_ctrl() # 只需使用默认控制值。
# 计数在“传输”中，默认是四字节字，因此将长度除以4
d.config(read=a, write=b, count=len(a)//4, ctrl=c, trigger=True)
# 等待完成
while d.active():
    pass
```

请注意，虽然这个示例在等待传输完成时处于空闲循环中，但程序也可以在这段时间内执行一些有用的工作。

DMA控制器的另一个可能更常见的用法是在内存和IO外设之间进行传输。在这种情况下，每次传输时 IO 寄存器的地址不会改变，但内存地址需要递增。还需要控制传输速度，以免在外设尚未准备好接收数据之前写入数据，或在数据未准备好之前读取数据，这可以通过 DMA 通道控制寄存器的 treq_sel 字段来控制。

每个 DMA 通道的控制寄存器的各个字段可以使用 `DMA.pack_ctrl()` 方法打包，并使用 `DMA.unpack_ctrl()` 静态方法解包。将数据从字节数组逐个字节传输到 PIO 状态机的 TX FIFO 的代码如下所示：

```python
# pio_num是正在使用的PIO块的索引，sm_num是该块中的状态机。
# my_state_machine是一个rp2.PIO()实例。
DATA_REQUEST_INDEX = (pio_num << 3) + sm_num

src_data = bytearray(1024)
d = rp2.DMA()

# 传输字节而不是字，不递增写地址并控制传输节奏。
c = d.pack_ctrl(size=0, inc_write=False, treq_sel=DATA_REQUEST_INDEX)

d.config(
    read=src_data,
    write=my_state_machine,
    count=len(src_data),
    ctrl=c,
    trigger=True
)
```

请注意，在这个示例中，为写地址提供的值只是我们要向其发送数据的 PIO 状态机。这是可行的，因为 PIO 状态机实现了缓冲区协议，允许直接访问它们的数据 FIFO 寄存器。

#### 构造函数

- class rp2.`DMA`

  声明对DMA控制器通道的独占使用权。

#### 方法

- DMA.`config`(read=None, write=None, count=None, ctrl=None, trigger=False)

  配置通道的 DMA 寄存器，并可选择启动传输。参数如下：
  - `read`: DMA 控制器将开始读取数据的地址，或提供要读取数据的对象。它可以是整数或任何支持缓冲区协议的对象。
  - `write`: DMA 控制器将开始写入的地址，或要写入数据的对象。它可以是整数或任何支持缓冲区协议的对象。
  - `count`: 此通道停止前将执行的总线传输次数。请注意，这是传输次数，而不是字节数。如果传输宽度为2或4字节，则需要相应地乘以移动的数据总量（以及所需缓冲区的大小）。
  - `ctrl`: DMA 控制寄存器的值。这是一个整数值，通常使用 `DMA.pack_ctrl()` 打包。
  - `trigger`: 可选择立即开始传输。
<br><br>

- DMA.`irq`(handler=None, hard=False)

  返回此 DMA 通道的 IRQ 对象，并可选择配置它。
<br><br>

- DMA.`close`()

  释放对底层 DMA 通道的声明，并释放中断处理程序。执行此操作后，DMA 对象将无法再使用。
<br><br>

- DMA.`pack_ctrl`(default=None, **kwargs)

  将关键字参数中提供的值打包到新控制寄存器值的命名字段中。未提供的任何字段将设置为默认值。默认值要么从提供的默认值中获取，要么如果未提供，则使用适合当前通道的默认值；将其设置为 `DMA.ctrl` 属性的当前值提供了一种轻松覆盖部分字段的方法。

  关键字参数的键可以是DMA.unpack_ctrl()方法返回的任何键。可写值如下：
  - `enable`: bool，设置为启用通道（默认值：True）。
  - `high_pri`: bool，使此通道的总线流量具有高优先级（默认值：False）。
  - `size`: int，传输大小：0=字节，1=半字，2=字（默认值：2）。
  - `inc_read`: bool，每次传输后递增读取地址（默认值：True）。
  - `inc_write`: bool，每次传输后递增写入地址（默认值：True）。
  - `ring_size`: int，如果非零，则当地址递增时，只有一个地址寄存器的底部 `ring_size` 位将发生变化，导致地址在接下来的 `1 << ring_size` 字节边界处回绕。地址回绕由 `ring_sel` 标志控制。零值禁用地址回绕。
  - `ring_sel`: bool，设置为 False 使 `ring_size` 应用于读取地址，或设置为 True 应用于写入地址。
  - `chain_to`: int，此传输完成后要触发的通道号。将此值设置为此 DMA 对象自己的通道号将禁用链接（这是默认值）。
  - `treq_sel`: int，选择传输请求信号。有关详细信息，请参阅 RP2040 数据手册的第2.5.3节。
  - `irq_quiet`: bool，在每次传输结束时不生成中断。相反，当中断触发寄存器写入零值时将生成中断，这将停止一系列链接传输（默认值：True）。
  - `bswap`: bool，如果设置为true，字或半字中的字节将在写入前反转（默认值：True）。
  - `sniff_en`: bool，设置为 True 允许芯片的嗅探硬件访问数据（默认值：False）。
  - `write_err`: bool，将此设置为 True 将清除先前报告的写入错误。
  - `read_err`: bool，将此设置为 True 将清除先前报告的读取错误。

  有关所有这些字段的详细信息，请参阅RP2040数据手册第2.5.7节中CH0_CTRL_TRIG寄存器的描述。
<br><br>

- DMA.`unpack_ctrl`(value)

  将DMA通道控制寄存器的值解包到一个字典中，该字典包含控制寄存器中每个字段的键/值对。`value` 是要解包的 ctrl 寄存器值。

  此方法将返回所有可以传递给 `DMA.pack_ctrl` 的键的值。此外，它还将返回控制寄存器中的只读标志：`busy`，在传输开始时为高，传输结束时为低；以及`ahb_err`，它是`read_err`和`write_err`标志的逻辑或。打包时将忽略这些值，因此解包控制寄存器创建的字典可以直接用作打包的关键字参数。
<br><br>

- DMA.`active`([value])

  获取或设置DMA通道当前是否正在运行。

  ```python
  >>> sm.active()
  0
  >>> sm.active(1)
  >>> while sm.active():
  ...     pass
  ```

#### 属性

- DMA.`read`

  此属性反映下一次总线传输将从其读取的地址。可以用整数或支持缓冲区协议的对象写入，这样做会立即生效。
<br><br>

- DMA.`write`

  此属性反映下一次总线传输将写入的地址。可以用整数或支持缓冲区协议的对象写入，这样做会立即生效。
<br><br>

- DMA.`count`

  读取此属性将返回当前传输序列中剩余的总线传输次数。写入此属性将设置下一个传输序列的总传输次数。
<br><br>

- DMA.`ctrl`

  此属性反映DMA通道控制寄存器。通常使用`DMA.pack_ctrl()`方法打包的整数写入。返回的寄存器值可以使用`DMA.unpack_ctrl()`方法解包。
<br><br>

- DMA.`channel`

  DMA通道的通道号。可以在另一个通道的`DMA.pack_ctrl()`的 chain_to 参数中传递此值，以允许DMA链接。
<br><br>

- DMA.`registers`

  此属性是一个类似数组的对象，允许直接访问DMA通道的寄存器。索引是按字而不是按字节，因此寄存器索引是寄存器地址偏移量除以4。有关寄存器详细信息，请参阅RP2040数据手册。


#### 链式和触发寄存器访问

RP2040 微控制器中的 DMA 控制器提供了几项高级功能，允许一个 DMA 通道启动另一个通道上的传输。一种方法是使用控制寄存器中的 `chain_to` 值，另一种方法是写入具有触发效应的 DMA 通道寄存器。结合一个 DMA 通道直接写入另一个通道的 DMA 寄存器的能力，这使得无需 CPU 干预即可执行复杂的事务。

下面是一个使用链式和寄存器触发实现将多个数据块收集到单个目标位置的示例。这些功能的完整详细信息可在 RP2040 数据手册的2.5节中找到，下面的代码是2.5.6.2子节中示例的Python版本。

```python
from rp2 import DMA
from uctypes import addressof
from array import array

def gather_strings(string_list, buf):
    # 我们使用两个DMA通道。第一个通道将收集列表中的长度和源地址发送到第二个通道的寄存器。第二个通道负责实际复制数据。
    gather_dma = DMA()
    buffer_dma = DMA()
    
    # 打包长度/地址对，以便发送到寄存器。
    gather_list = array("I")
    for s in string_list:
        gather_list.append(len(s))
        gather_list.append(addressof(s))
        gather_list.append(0)
        gather_list.append(0)
    
    # 当写入第二个DMA通道的寄存器时，我们需要将写地址按8字节(1<<3字节)边界环绕。
    # 我们写入最后一个寄存器别名中的``TRANS_COUNT``和``READ_ADD_TRIG``寄存器(寄存器14和15)。
    gather_ctrl = gather_dma.pack_ctrl(ring_size=3, ring_sel=True)
    gather_dma.config(
        read=gather_list, write=buffer_dma.registers[14:16],
        count=2, ctrl=gather_ctrl
    )
    
    # 复制数据时，传输大小为单字节，完成后需要链式返回到另一个收集DMA事务的开始。
    buffer_ctrl = buffer_dma.pack_ctrl(size=0, chain_to=gather_dma.channel)
    
    # 读地址和计数的值将由另一个DMA通道设置。
    buffer_dma.config(write=buf, ctrl=buffer_ctrl)
    
    # 启动传输。
    gather_dma.active(1)
    
    # 等待所有寄存器值发送完毕
    end_address = addressof(gather_list) + 4 * len(gather_list)
    while gather_dma.read != end_address:
        pass

input = ["This is ", "a ", "test", " of the scatter", " gather", " process"]
output = bytearray(64)

print(output)
gather_strings(input, output)
print(output)
```

此示例在等待传输完成时处于空闲状态；或者，它可以设置中断处理程序并立即返回。

### class Flash - 访问内置闪存存储

此类提供对 SPI 闪存的访问。

在大多数情况下，要在设备上持久存储数据，您可能希望使用更高级的抽象，例如通过 Python 标准 API 的文件系统，但此接口对于配置自定义文件系统或为您的应用程序实现低级存储系统很有用。

#### 构造函数

- class rp2.`Flash`

  获取用于访问SPI闪存的单例对象。

#### 方法

- Flash.`readblocks`(block_num, buf)
- Flash.`readblocks`(block_num, buf, offset)
- Flash.`writeblocks`(block_num, buf)
- Flash.`writeblocks`(block_num, buf, offset)
- Flash.`ioctl`(cmd, arg)

  这些方法实现了由 `vfs.AbstractBlockDev` 定义的简单和扩展块协议。

### class PIO – 高级 PIO 用法

`PIO` 类可用于访问 RP2040 的 PIO（可编程 I/O）接口实例。

与 PIO 交互的首选方式是使用 `rp2.StateMachine`，PIO 类适用于高级用法。

有关汇编 PIO 程序的内容，请参见 `rp2.asm_pio()`。


#### 构造函数

- class rp2.`PIO`(id)

  获取编号为 `id` 的 PIO 实例。RP2040 有两个 PIO 实例，编号为 0 和 1。

  如果提供了其他任何参数，将引发 `ValueError`。


#### 方法

- PIO.`gpio_base`( [ base ] )

  查询并可选地设置此 PIO 实例当前的 GPIO 基准。

  如果提供了参数，则该参数必须是一个引脚（或对应引脚编号的整数），且仅限于 GPIO0 或 GPIO16。GPIO 基准将被设置为该引脚。设置 GPIO 基准必须在添加任何程序或创建状态机之前完成。

  返回当前的 GPIO 基准引脚。
<br><br>

- PIO.`add_program`(program)

  将 `program` 添加到此 PIO 实例的指令存储器中。

  每个 PIO 实例上可用于程序的存储器空间是有限的。如果 PIO 的程序存储器中没有足够的剩余空间，此方法将引发 `OSError`(ENOMEM)。
<br><br>

- PIO.`remove_program`( [ program ] )

  从此 PIO 实例的指令存储器中移除 `program`。

  如果未提供 `program`，则移除所有程序。

  移除已被移除的程序不会报错。
<br><br>

- PIO.`state_machine`(id [ , program, ... ] )

  获取编号为 `id` 的状态机。在 RP2040 上，每个 PIO 实例有四个状态机，编号为 0 到 3。

  可选地用 `program` 对其进行初始化：参见 `StateMachine.init`。

  ```py
  >>> rp2.PIO(1).state_machine(3)
  StateMachine(7)
  ```

- PIO.`irq`(handler=None, trigger=IRQ_SM0 | IRQ_SM1 | IRQ_SM2 | IRQ_SM3, hard=False)

  返回此 PIO 实例的 IRQ 对象。

  MicroPython 仅使用每个 PIO 实例上的 IRQ 0。IRQ 1 不可用。

  可选对其进行配置。


#### 常量
- PIO.`IN_LOW`
- PIO.`IN_HIGH`
- PIO.`OUT_LOW`
- PIO.`OUT_HIGH`

  这些常量用于 `asm_pio` 的 `out_init`、`set_init` 和 `sideset_init` 参数。
<br><br>

- PIO.`SHIFT_LEFT`
- PIO.`SHIFT_RIGHT`

  这些常量用于 `asm_pio` 或 `StateMachine.init` 的 `in_shiftdir` 和 `out_shiftdir` 参数。
<br><br>

- PIO.`JOIN_NONE`
- PIO.`JOIN_TX`
- PIO.`JOIN_RX`

  这些常量用于 `asm_pio` 的 `fifo_join` 参数。
<br><br>

- PIO.`IRQ_SM0`
- PIO.`IRQ_SM1`
- PIO.`IRQ_SM2`
- PIO.`IRQ_SM3`

  这些常量用于 `PIO.irq` 的 `trigger` 参数。
  
### class StateMachine — 访问 RP2040 的可编程 I/O 接口  

状态机类（StateMachine）用于访问 RP2040 的 PIO（即可编程输入/输出）接口。关于 PIO 程序的汇编方法，请参考 `rp2.asm_pio()`。  


#### 构造函数  

- class rp2.`StateMachine`(id[, program, ... ])

  获取编号为 `id` 的状态机。RP2040 有两个完全相同的 PIO 实例，每个实例包含 4 个状态机，因此总共有 8 个状态机，编号为 0 到 7。

  可选参数：可通过给定的程序 `program` 对其进行初始化（详见 `StateMachine.init`）。  

#### 方法

- StateMachine.`init`(program, freq=-1, *, in_base=None, out_base=None, set_base=None, jmp_pin=None, sideset_base=None, in_shiftdir=None, out_shiftdir=None, push_thresh=None, pull_thresh=None)

  配置状态机实例以运行给定的程序。

  程序会被添加到该 PIO 实例的指令存储器中。如果指令存储器中已包含该程序，则会复用其偏移量以节省指令存储空间。  

  - `freq`：状态机的运行频率（单位为 Hz）。默认值为系统时钟频率。时钟分频器的计算方式为 "系统时钟频率 / freq"，因此可能存在轻微的舍入误差。最小可能的时钟分频为系统时钟的 1/65536，因此在默认系统时钟频率（125MHz）下，`freq` 的最小值为 1908。若要以更低频率运行状态机，需通过 `machine.freq()` 降低系统时钟速度。
  - `in_base`：用于 `in()` 指令的第一个引脚。
  - `out_base`：用于 `out()` 指令的第一个引脚。
  - `set_base`：用于 `set()` 指令的第一个引脚。
  - `jmp_pin`：用于 `jmp(pin, ...)` 指令的第一个引脚。
  - `sideset_base`：用于边置位（side-setting）的第一个引脚。
  - `in_shiftdir`：输入移位寄存器（ISR）的移位方向，可选 `PIO.SHIFT_LEFT`（左移）或 `PIO.SHIFT_RIGHT`（右移）。
  - `out_shiftdir`：输出移位寄存器（OSR）的移位方向，可选 `PIO.SHIFT_LEFT`（左移）或 `PIO.SHIFT_RIGHT`（右移）。
  - `push_thresh`：触发自动推送（auto-push）或条件重推送（conditional re-pushing）的比特阈值。
  - `pull_thresh`：触发自动拉取（auto-pull）或条件重拉取（conditional re-pulling）的比特阈值。
<br><br>

- StateMachine.`active`([value ])

  获取或设置状态机是否运行。
  ```python
  >>> sm.active()
  True
  >>> sm.active(0)
  False
  ```
<br>

- StateMachine.`restart`()

  重启状态机并跳转到程序开头。

  该方法通过 RP2040 的 `SM_RESTART` 寄存器清除状态机的内部状态，包括：
  - 输入和输出移位计数器
  - 输入移位寄存器（ISR）的内容
  - 延迟计数器
  - 等待中断（IRQ）的状态
  - 通过 `StateMachine.exec()` 运行的停滞指令
<br><br>

- StateMachine.`exec`(instr)

  执行单条 PIO 指令。

  如果 `instr` 是字符串，则使用 `asm_pio_encode` 对该字符串中的指令进行编码。
  ```python
  >>> sm.exec("set(0, 1)")
  ```

  如果 `instr` 是整数，则将其视为已编码的 PIO 机器码指令并执行。
  ```python
  >>> sm.exec(rp2.asm_pio_encode("out(y, 8)", 0))
  ```
<br>

- StateMachine.`get`(buf=None, shift=0)

  从状态机的接收 FIFO（RX FIFO）中读取一个字（word）。
  
  如果 FIFO 为空，该方法会阻塞直至数据到达（即状态机推送一个字）。
  
  返回值会先右移 `shift` 位，即最终返回 `word >> shift`。
<br><br>

- StateMachine.`put`(value, shift=0)

  向状态机的发送 FIFO（TX FIFO）中推送字。

  `value` 可以是整数、`B`/`H`/`I` 类型的数组或字节数组（bytearray）。

  该方法会阻塞直至所有字写入 FIFO。如果 FIFO 已满或在写入过程中变满，方法会阻塞直至状态机拉取足够的字以完成写入。

  每个字会先左移 `shift` 位，即状态机接收的是 `word << shift`。
<br><br>

- StateMachine.`rx_fifo`()

  返回状态机接收 FIFO（RX FIFO）中的字数。返回 0 表示 FIFO 为空。

  在调用 `StateMachine.get()` 前，可用于检查是否有等待读取的数据。
<br><br>

- StateMachine.`tx_fifo`()

  返回状态机发送 FIFO（TX FIFO）中的字数。返回 0 表示 FIFO 为空。

  在调用 `StateMachine.put()` 前，可用于检查是否有空间推送新的字。
<br><br>

- StateMachine.`irq`(handler=None, trigger=0 | 1, hard=False)

  返回给定状态机的中断（IRQ）对象。

  可选参数：可对其进行配置。


#### 缓冲区协议  

状态机类支持缓冲区协议（buffer protocol），允许直接访问每个状态机的发送和接收 FIFO。这主要是为了使状态机对象能直接作为 `rp2.DMA()` 通道配置时的读/写参数。

