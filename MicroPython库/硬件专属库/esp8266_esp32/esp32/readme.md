# esp32（特定于ESP32的功能）

esp32 模块包含专门用于控制 ESP32 芯片的函数和类。

## 函数

- esp32.`wake_on_touch`(wake)

  配置触摸是否能将设备从睡眠状态唤醒。`wake` 为布尔值。
<br><br>

- esp32.`wake_on_ulp`(wake)

  配置超低功耗协处理器(ULP)是否能将设备从睡眠状态唤醒。`wake` 为布尔值。
<br><br>

- esp32.`wake_on_ext0`(pin, level)

  配置 EXT0 如何将设备从睡眠状态唤醒。`pin` 可以为 None 或有效的 Pin 对象。`level` 为 `esp32.WAKEUP_ALL_LOW` 或 `esp32.WAKEUP_ANY_HIGH`。
<br><br>

- esp32.`wake_on_ext1`(pins, level)

  配置 EXT1 如何将设备从睡眠状态唤醒。`pins` 可以为 None 或有效 Pin 对象的元组/列表。`level` 应为 `esp32.WAKEUP_ALL_LOW` 或 `esp32.WAKEUP_ANY_HIGH`。
<br><br>

- esp32.`gpio_deep_sleep_hold`(enable)

  配置在深度睡眠模式下是否保留非 RTC GPIO 引脚配置。`enable` 为布尔值。
<br><br>

- esp32.`raw_temperature`()

  读取内部温度传感器的原始值，返回一个整数。
<br><br>

- esp32.`idf_heap_info`(capabilities)

  返回关于 ESP-IDF 堆内存区域的信息。其中之一包含 MicroPython 堆，其他用于 ESP-IDF，例如网络缓冲区和其他数据。这些数据有助于了解 ESP-IDF 特别是网络栈可用的内存量。它可能有助于解释 ESP-IDF 操作因分配失败而失败的情况。

  `capabilities` 参数对应于 ESP-IDF 的 MALLOC_CAP_XXX 值，但最有用的两个预定义为 `esp32.HEAP_DATA`（用于数据堆区域）和 `esp32.HEAP_EXEC`（用于本机代码发射器使用的可执行区域）。

  返回值是一个包含 4 元组的列表，每个 4 元组对应一个堆，包含：总字节数、空闲字节数、最大空闲块和随时间观察到的最小空闲量。

  启动后的示例：
  ```
  >>> import esp32; esp32.idf_heap_info(esp32.HEAP_DATA)
  [(240, 0, 0, 0), (7288, 0, 0, 0), (16648, 4, 4, 4), (79912, 35712, 35512, 35108), (15072, 15036, 15036, 15036), (113840, 0, 0, 0)]
  ```

  **注意**: `esp32.HEAP_DATA` 区域中的空闲 IDF 堆内存可自动添加到 MicroPython 堆中，以防止 MicroPython 分配失败。但是，此处返回的信息对于排查 Python 分配失败没有帮助。应使用 `micropython.mem_info()` 和 `gc.mem_free()`。`micropython.mem_info()` 输出中的 "max new split" 值对应于 ESP-IDF 堆中可按需自动添加到 MicroPython 堆的最大空闲块。`gc.mem_free()` 的结果是 `micropython.mem_info()` 打印的当前 "free" 和 "max new split" 值的总和。
<br><br>

- esp32.`idf_task_info`()

  返回关于运行中的 ESP-IDF/FreeRTOS 任务的信息，包括 MicroPython 线程。这些数据有助于深入了解任务花费多少时间运行，或者它们是否被阻塞了很长时间，并确定分配的堆栈是否被充分利用或是否可以减少。

  必须在板配置中设置 `CONFIG_FREERTOS_USE_TRACE_FACILITY=y` 才能使用此方法。此外，建议配置 `CONFIG_FREERTOS_GENERATE_RUN_TIME_STATS=y` 和 `CONFIG_FREERTOS_VTASKLIST_INCLUDE_COREID=y`，以便分别检索总运行时间和每个任务的运行时间以及核心ID。
  
  返回值是一个 2 元组，其中第一个值是总运行时间，第二个是任务列表。每个任务是一个 7 元组，包含：任务ID、名称、当前状态、优先级、运行时间、堆栈高水位标记和运行它的核心ID。当相应的 FreeRTOS 配置选项未启用时，运行时间和核心ID将为None。

  **注意**: 要获得基于此函数的更易于使用的输出，可以使用 utop 库，它实现了类似于 Unix top 命令的实时概览。

## 闪存分区

提供对设备闪存中分区的访问，并包含启用空中(OTA)更新的方法。

- class esp32.`Partition`(id, block_size=4096, / )

  创建表示分区的对象。`id` 可以是字符串，表示要检索的分区标签，或以下常量之一：`BOOT` 或 `RUNNING`。`block_size` 指定单个块的字节大小。
<br><br>

- classmethod Partition.`find`(type=TYPE_APP, subtype=0xff, label=None, block_size=4096)

  查找由 `type`(类型)、`subtype`(子类型)和 `label` (标签)指定的分区。返回 Partition 对象的列表（可能为空）。注意：`subtype=0xff` 匹配任何子类型，`label=None` 匹配任何标签。

  `block_size` 指定返回对象使用的单个块的字节大小。
<br><br>

- Partition.`info`()

  返回一个 6 元组(type, subtype, addr, size, label, encrypted)。
<br><br>

- Partition.`readblocks`(block_num, buf)
- Partition.`readblocks`(block_num, buf, offset)
- Partition.`writeblocks`(block_num, buf)
- Partition.`writeblocks`(block_num, buf, offset)
- Partition.`ioctl`(cmd, arg)

  这些方法实现了 `vfs.AbstractBlockDev` 定义的简单和扩展块协议。
<br><br>

- Partition.`set_boot`()

  将分区设置为引导分区。

  **注意**: 更改 OTA 引导分区后，在执行硬重置或电源循环之前，不要进入深度睡眠。这确保引导加载程序在引导前验证新映像。
<br><br>

- Partition.`get_next_update`()

  获取此分区之后的下一个更新分区，并返回一个新的Partition对象。典型用法是 `Partition(Partition.RUNNING).get_next_update()`，它返回给定当前运行分区的下一个要更新的分区。
<br><br>

- classmethod Partition.`mark_app_valid_cancel_rollback`()

  表示当前引导被认为是成功的。在新分区的第一次引导时需要调用 `mark_app_valid_cancel_rollback`，以避免在下一次引导时自动回滚。这使用 ESP-IDF 的"应用回滚"功能，带有 "CONFIG_BOOTLOADER_APP_ROLLBACK_ENABLE"，如果在未启用此功能的固件上调用，则会引发 OSError(-261)。可以在每次引导时调用 `mark_app_valid_cancel_rollback`，而在使用 esptool 加载的固件时不需要这样做。

### 常量

- Partition.`BOOT`
- Partition.`RUNNING`

  在 Partition 构造函数中用于获取各种分区：`BOOT` 是下次重置时将引导的分区，`RUNNING` 是当前正在运行的分区。
<br><br>

- Partition.`TYPE_APP`
- Partition.`TYPE_DATA`

  在 `Partition.find` 中用于指定分区类型：`APP` 用于可引导固件分区（通常标记为factory、ota_0、ota_1），`DATA` 用于其他分区，例如 nvs、otadata、phy_init、vfs。
<br><br>

- esp32.`HEAP_DATA`
- esp32.`HEAP_EXEC`

  在 idf_heap_info 中使用。

## RMT

RMT（远程控制）模块是ESP32的专用模块，最初设计用于发送和接收红外遥控信号。但由于其灵活的设计和极高精度（低至12.5纳秒）的脉冲生成能力，它也可用于传输或接收许多其他类型的数字信号：
```python
import esp32
from machine import Pin

r = esp32.RMT(0, pin=Pin(18), clock_div=8)
r  # RMT(channel=0, pin=18, source_freq=80000000, clock_div=8, idle_level=0)

# 为高电平输出添加载波频率
r = esp32.RMT(0, pin=Pin(18), clock_div=8, tx_carrier=(38000, 50, 1))

# 通道分辨率为100纳秒（1/(源频率/时钟分频)）
r.write_pulses((1, 20, 2, 40), 0)  # 发送0持续100纳秒，1持续2000纳秒，0持续200纳秒，1持续4000纳秒
```

RMT 模块的输入时钟为 80MHz（未来可能支持配置输入时钟，但目前为固定值）。`clock_div` 对输入时钟进行分频，从而决定RMT通道的分辨率。`write_pulses` 中指定的数值会乘以分辨率来定义脉冲时长。

`clock_div` 是8位分频器（0-255），每个脉冲可通过将分辨率乘以 15 位数值（1-PULSE_MAX）来定义。共有 8 个通道（0-7），每个通道可设置不同的时钟分频器。

因此，在上述示例中，80MHz 时钟被 8 分频，分辨率为 (1/(80MHz/8))=100纳秒。由于起始电平为 0 且每个数值会触发电平翻转，因此比特流为 0101，对应时长为 [100ns, 2000ns, 100ns, 4000ns]。

更多细节请参考乐鑫科技的ESP-IDF RMT文档。

⚠️ 警告

当前 MicroPython 的 RMT 实现缺少部分功能，尤其是脉冲接收功能。RMT 应被视为测试功能，其接口未来可能变更。


### class

- esp32.`RMT`(channel, *, pin=None, clock_div=8, idle_level=False, tx_carrier=None)

  该类用于访问8个RMT通道中的一个：
  - `channel`：指定配置的RMT通道（0-7）
  - `pin`：配置绑定到RMT通道的引脚
  - `clock_div`：8 位时钟分频器（默认 8），对 80MHz 源时钟分频以设置通道分辨率
  - `idle_level`：指定无传输时的输出电平（布尔值，True为高电平，False为低电平）
  - `tx_carrier`：启用载波传输时需传入三元组（载波频率，占空比0-100，载波应用的输出电平）

- RMT.`source_freq`()

  返回源时钟频率（当前固定为80MHz）。
<br><br>

- RMT.`clock_div`()

  返回时钟分频值，通道分辨率为 `1/(源频率/clock_div)`。
<br><br>

- RMT.`wait_done`(*, timeout=0)

  若通道空闲返回 True，若正在传输脉冲序列返回 False。可通过 `timeout` 参数阻塞等待传输完成（单位：毫秒）。
<br><br>

- RMT.`loop`(enable_loop)

  配置通道循环传输：`enable_loop=True` 时，下次调用 `write_pulses` 将启用循环；若当前正在循环传输时设置为 False，将完成当前循环后停止。
<br><br>

- RMT.`write_pulses`(duration, data=True)

  开始传输脉冲序列，支持三种模式：
  - **模式1**：`duration` 为时长列表/元组，`data` 为初始输出电平（每个时长后电平翻转）
  - **模式2**：`duration` 为固定时长，`data` 为输出电平列表/元组
  - **模式3**：`duration` 和 `data` 为等长列表/元组，分别指定每个脉冲的时长和电平
  
  时长单位为通道分辨率（1 到 `PULSE_MAX`），输出电平为布尔值（True=高电平）。若此前有传输未完成，该方法将阻塞直至完成。启用循环时，序列将无限重复，且新序列需等待当前循环结束后开始。硬件不支持超过126个脉冲的循环序列。
<br><br>

- static RMT.`bitstream_channel`([value])

  选择 `machine.bitstream` 使用的 RMT 通道：
  
  `value` 可以是 None 或有效通道号。None 代表禁用 RMT，使用 `machine.bitstream` 位操作实现。默认使用编号最大的通道，无参数事返回当前通道号

### 常量

- RMT.`PULSE_MAX`

  脉冲时长可设置的最大整数值。


## Ultra-Low-Power co-processor（超低功耗协处理器）

该类用于访问 ESP32、ESP32-S2 和 ESP32-S3 芯片上的超低功耗（ULP）协处理器。  

⚠️ 警告

此类不提供对 ESP32-S2 和 ESP32-S3 芯片上 RISCV ULP 协处理器的访问权限。  


- class esp32.`ULP`

  该类用于访问超低功耗协处理器。
<br><br>

- ULP.`set_wakeup_period`(period_index, period_us)

  设置唤醒周期。
<br><br>

- ULP.`load_binary`(load_addr, program_binary)

  将程序二进制数据 `program_binary` 加载到ULP的指定地址 `load_addr`。
<br><br>

- ULP.`run`(entry_point)

  从指定入口点`entry_point`启动ULP运行。


### 常量

- esp32.`WAKEUP_ALL_LOW`
- esp32.`WAKEUP_ANY_HIGH`

  用于选择引脚的唤醒电平。
<br>

## Non-Volatile Storage（非易失性存储）

该类用于访问由 ESP-IDF 管理的非易失性存储（NVS）。NVS 被划分为多个命名空间，每个命名空间包含类型化的键值对。键为字符串，值可以是各种整数类型、字符串或二进制数据块。当前驱动仅支持32位有符号整数和二进制块。

⚠️ 警告

对NVS的修改需要通过调用 `commit` 方法提交到闪存。未调用 `commit` 将导致修改在下次重置时丢失。

- class esp32.`NVS`(namespace)

  创建一个用于访问指定命名空间的对象（若命名空间不存在则自动创建）。
<br><br>

- NVS.`set_i32`(key, value)

  为指定键设置 32 位有符号整数值。请记住调用 `commit`！
<br><br>

- NVS.`get_i32`(key)

  返回指定键的有符号整数值。若键不存在或类型不同，将引发 OSError。
<br><br>

- NVS.`set_blob`(key, value)

  为指定键设置二进制数据块值。传入的 `value` 必须支持缓冲区协议（如bytes、bytearray、str）。注意：ESP-IDF 区分二进制块和字符串，即使传入字符串，此方法也始终写入二进制块。
  
  请记住调用 `commit`！
<br><br>

- NVS.`get_blob`(key, buffer)

  将指定键的二进制块值读取到 `buffer`（必须为 bytearray）中，返回实际读取的长度。若键不存在、类型不同或缓冲区过小，将引发 OSError。
<br><br>

- NVS.`erase_key`(key)

  删除键值对。
<br><br>

- NVS.`commit`()

  将 `set_xxx` 方法所做的修改提交到闪存。
