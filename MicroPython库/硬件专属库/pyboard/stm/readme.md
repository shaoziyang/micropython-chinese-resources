# stm（STM32微控制器专用功能）

stm 模块提供 STM32 微控制器特有的功能，包括直接访问外设寄存器。

## 内存访问

模块提供三个用于原始内存访问的对象：

- stm.`mem8`
- stm.`mem16`
- stm.`mem32`

  读取/写入8/16/32位内存。

  使用下标表示法`[...]`配合目标地址来索引这些对象。这些内存对象可与外设寄存器常量结合使用，用于读写MCU硬件外设的寄存器，以及地址空间的所有其他区域。
<br>

## 外设寄存器常量

该模块定义了从 CMSIS 头文件生成的寄存器常量，可用的常量取决于编译目标的微控制器系列。一些常量示例如：

- stm.`GPIOA`

  GPIOA外设的基地址。
<br><br>

- stm.`GPIOB`
  
  GPIOB外设的基地址。
<br><br>

- stm.`GPIO_BSRR`

  GPIO位设置/复位寄存器的偏移量。
<br><br>

- stm.`GPIO_IDR`

  GPIO输入数据寄存器的偏移量。
<br><br>

- stm.`GPIO_ODR`

  GPIO输出数据寄存器的偏移量。
<br><br>

以某外设命名的常量（如GPIOA）表示该外设的绝对地址；以某外设名称为前缀的常量（如GPIO_BSRR）表示寄存器的相对偏移量。访问外设寄存器需要将外设的绝对基地址与相对寄存器偏移量相加。例如，`GPIOA + GPIO_BSRR` 表示 GPIOA->BSRR 寄存器的完整绝对地址。

使用示例：

```py
# 设置PA2引脚为高电平
stm.mem32[stm.GPIOA + stm.GPIO_BSRR] = 1 << 2

# 读取PA3引脚状态
value = (stm.mem32[stm.GPIOA + stm.GPIO_IDR] >> 3) & 1
```
<br>

## STM32WBxx 微控制器专用函数

这些函数仅用在 STM32WBxx 微控制器，用于与第二个 CPU（RF核心）交互：

- stm.`rfcore_status`()

  以整数形式返回第二个 CPU 的状态（设备信息表的第一个字）。
<br><br>

- stm.`rfcore_fw_version`(id)

  获取运行在第二个CPU上的固件版本。传入 `id=0` 获取 FUS 版本，传入 `id=1` 获取 WS 版本。

  返回包含完整版本号的5元组。
<br><br>

- stm.`rfcore_sys_hci`(ogf, ocf, data, timeout_ms=0)

  在SYS通道上执行HCI命令，执行过程是同步的。
  
  返回包含SYS命令结果的字节对象。
<br>

## STM32WLxx 微控制器专用函数

这些函数仅用在 STM32WLxx 微控制器，用于与集成的 "SUBGHZ" 无线调制解调器外设交互：

- stm.`subghz_cs`(level)

  设置连接到无线外设的内部 SPI 片选(CS)引脚。`level` 参数为低电平有效：真值表示"CS引脚高电平"（信号无效），假值表示"CS引脚低电平"（信号有效）。

  对应此 CS 信号的内部 SPI 总线可通过 `machine.SPI()` 的 `id="SUB-GHZ"` 实例化。
<br><br>

- stm.`subghz_irq`(handler)

  将内部 SUBGHZ 无线中断处理函数设置为指定的函数。该处理函数作为"硬"中断响应无线外设中断。
  
  若将 `handler` 参数设为 `None`，则禁用中断。
  
  注意：由于硬件限制，每次中断触发时，MicroPython 会在调用处理函数前禁用中断。若要接收下一次中断，需再次调用 `subghz_irq()` 设置处理函数（这会重新启用中断）。
<br><br>

- stm.`subghz_is_busy`()

  返回对应无线外设内部"RFBUSYS"信号的布尔值。通过 SPI 向无线设备发送新命令前，应轮询此函数直到返回 `False`，以确认忙信号已释放。
  
