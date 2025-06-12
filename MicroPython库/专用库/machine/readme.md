# machine（与硬件相关的功能）

machine 模块包含与硬件相关的特定功能。该模块中的大多数功能允许直接且无限制地访问和控制系统上的硬件模块（如 CPU、定时器、总线等）。如果使用不当，可能导致开发板功能异常、死机、崩溃，在极端情况下甚至会造成硬件损坏。

关于 machine 模块的函数和类方法所使用的回调函数的说明：所有这些回调函数均应视为在中断里执行。对于 ID≥0 的物理设备和 ID 为负数（如 - 1）的 “虚拟” 设备（这些 “虚拟” 设备实际上仍是真实硬件和真实硬件中断之上的轻量级接口）来说，这一点均成立。

## 内存访问

该模块提供了三个用于原始内存访问的对象。

- machine.`mem8`
- machine.`mem16`
- machine.`mem32`

  读写 8/16/32 位内存。

使用下标表示法`[...]`并传入目标地址来访问这些对象。请注意，无论访问的内存大小如何，地址均指字节地址。

**示例用法**（寄存器是特定于STM32微控制器）：

```python
import machine
from micropython import const

GPIOA = const(0x48000000)
GPIO_BSRR = const(0x18)
GPIO_IDR = const(0x10)

# 将PA2引脚置高
machine.mem32[GPIOA + GPIO_BSRR] = 1 << 2

# 读取PA3引脚值
value = (machine.mem32[GPIOA + GPIO_IDR] >> 3) & 1
```

## 复位相关函数

- machine.`reset`()

  以类似于按下外部复位按钮的方式对设备进行硬复位。
<br><br>

- machine.`soft_reset`()

  对解释器执行软复位，删除所有 Python 对象并重置 Python 堆。
<br><br>

- machine.`reset_cause`()

  获取复位原因。可能的返回值见相关常量定义。
<br><br>

- machine.`bootloader`( [ value ] )

  复位设备并进入其引导加载程序。此功能通常用于使设备进入可编程升级新固件的状态。

  某些硬件支持传入可选的 `value` 参数，该参数可控制进入哪种引导加载程序、向其传递什么内容或其他功能。


## 中断相关函数

以下函数可用于控制中断。某些系统需要中断才能正常运行，因此长时间禁用中断可能会影响核心功能，例如看门狗定时器可能会意外触发。中断只能在最短的时间内禁用，然后恢复到之前的状态。例如：

```python
import machine

# 禁用中断
state = machine.disable_irq()

# 在这里执行少量时间关键的工作
# 启用中断
machine.enable_irq(state)
```

- machine.`disable_irq`()

  禁用中断请求。返回先前的 IRQ 状态，该状态应被视为不透明值。此返回值应传递给 `enable_irq()` 函数，以将中断恢复到调用 `disable_irq()` 之前的原始状态。
<br><br>

- machine.`enable_irq`(state)

  重新启用中断请求。`state` 参数应为最近一次调用 `disable_irq()` 函数时返回的值。


## 电源相关函数
- machine.`freq`([ hz ])

  返回 CPU 频率（以赫兹为单位）。

  在某些移植版本上，也可以通过传入 `hz` 参数来设置 CPU 频率。
<br><br>

- machine.`idle`()

  停止向CPU提供时钟，可在短时间或长时间内降低功耗。外设继续工作，一旦触发任何中断，或在CPU暂停后最多1毫秒内恢复执行。

  建议在持续检查外部变化（即轮询）的紧密循环中调用此函数。这将在不显著影响性能的情况下降低功耗。
<br><br>

- machine.`sleep`()

  **注意**: 此函数已弃用，请改用不带参数的 `lightsleep()`。
<br><br>

- machine.`lightsleep`([ time_ms ])
- machine.`deepsleep`([ time_ms ])

  停止执行以尝试进入低功耗状态。

  如果指定了 `time_ms` 参数，将睡眠指定的时间（毫秒）。否则，将无限期持续睡眠。

  无论是否设置超时，如果有需要处理的事件，执行可能随时恢复。此类事件或唤醒源应在睡眠前配置，如引脚变化或RTC超时。

  `lightsleep` 和 `deepsleep` 的具体行为和节能能力高度依赖于底层硬件，但一般特性如下：
  - 浅睡眠（lightsleep）完全保留 RAM 和状态。唤醒后，执行从请求的睡眠位置继续，所有子系统均可正常运行。
  - 深度睡眠（deepsleep）可能不保留 RAM 或系统的任何其他状态（例如外设或网络接口）。唤醒后，执行从主脚本重新开始，类似于硬复位或上电复位。`reset_cause()` 函数将返回 `machine.DEEPSLEEP`，可用于区分深度睡眠唤醒与其他复位。
<br><br>

- machine.`wake_reason`()

  获取唤醒原因。可能的返回值见常量部分。


## 其它函数

注：部分函数的可用性依赖于 MicroPython 的硬件和版本。

- machine.`unique_id`()

  返回一个字节字符串，代表开发板/SoC（片上系统）的唯一标识符。如果底层硬件支持，不同开发板/SoC 的标识符会不同。其长度因硬件而异（因此如果需要短 ID，可使用完整值的子字符串）。在某些 MicroPython 移植版中（如 esp32），该 ID 对应网络 MAC 地址。
<br><br>

- machine.`time_pulse_us`(pin, pulse_level, timeout_us=1000000, / )

  测量给定引脚上的脉冲时间，并以微秒为单位返回脉冲持续时间。`pulse_level` 参数为 `0`（测量低电平脉冲）或 `1`（测量高电平脉冲）。
  如果引脚的当前输入状态与 `pulse_level` 不同，函数会先等待引脚输入变为 `pulse_level`，然后测量引脚保持为 `pulse_level` 的持续时间。如果引脚和  `pulse_level` 状态相同，则直接开始计时。
  如果在等待条件时超时，函数返回 `-2`；如果在主测量阶段超时，返回 `-1`。两种情况的超时时间均由 `timeout_us`（微秒）指定。
<br><br>

- machine.`bitstream`(pin, encoding, timing, data, / )

  通过指定 `pin` （引脚）的脉冲传输数据。`encoding` 参数指定位的编码方式，`timing` 是特定于编码的时序规范。

  支持的编码方式：
  - `0` 表示"高低脉冲宽度调制"。此模式从最高有效位开始，以定时脉冲传输 `0` 和 `1` 位。`timing` 必须是一个代表纳秒的四元组，格式为 `(high_time_0, low_time_0, high_time_1, low_time_1)`。例如，`(400, 850, 800, 450)` 是 800kHz 下 WS2812 RGB LED 的时序规范。

  时序的精度因硬件而异：
  - 在 48MHz 的 Cortex M0 上，精度最差为 ±120ns；
  - 在更快的 MCU（如 ESP8266、ESP32、STM32、Pyboard）上，精度接近 ±30ns。

  **注意**：控制 WS2812/NeoPixel 灯带时，建议使用 `neopixel` 模块中的高级 API。
<br><br>

- machine.`rng`()

  返回一个 24 位软件生成的随机数。


## 常数

- machine.`IDLE`
- machine.`SLEEP`
- machine.`DEEPSLEEP`

  IRQ 唤醒值。
<br><br>

- machine.`PWRON_RESET`
- machine.`HARD_RESET`
- machine.`WDT_RESET`
- machine.`DEEPSLEEP_RESET`
- machine.`SOFT_RESET`

  复位原因。
<br><br>

- machine.`WLAN_WAKE`
- machine.`PIN_WAKE`
- machine.`RTC_WAKE`

  唤醒原因。



## 类

### class Pin – 控制 I/O 引脚

Pin 对象用于控制 I/O 引脚（也称为 GPIO —— 通用输入/输出）。Pin 对象通常与物理引脚相关联，这些引脚可驱动输出电压或读取输入电压。Pin 类包含设置引脚模式（输入、输出等）的方法，以及获取和设置数字逻辑电平的方法。如需对引脚进行模拟控制，请参见ADC类。

通过唯一标识特定 I/O 引脚的标识符来构造 Pin 对象。标识符的允许形式和其映射的物理引脚因端口而异，可能为整数、字符串或包含端口和引脚号的元组。

**使用示例：**
```python
from machine import Pin

# 在引脚 #0 上创建输出引脚
p0 = Pin(0, Pin.OUT)

# 将电平设置为低电平再设置为高电平
p0.value(0)
p0.value(1)

# 配置引脚 #2 上为带上拉电阻的输入
p2 = Pin(2, Pin.IN, Pin.PULL_UP)

# 读取并打印引脚值
print(p2.value())

# 将引脚 #0 重新配置为带下拉电阻的输入模式
p0.init(p0.IN, p0.PULL_DOWN)

# 配置中断回调函数
p0.irq(lambda p:print(p))
```

#### 构造函数

- class machine.`Pin`(id, mode=-1, pull=-1, *, value=None, drive=0, alt=-1)

  访问与给定 `id` 关联的引脚外设（GPIO 引脚）。若构造函数中提供其他参数，则用于初始化引脚，未指定的设置将保持先前状态。

  参数说明：
  - `id`（必填）：可为任意对象，可能的类型包括：
    - `int`（内部引脚标识符）
    - `str`（引脚名称）
    - `tuple`（[端口, 引脚]对）
  - `mode`：指定引脚模式，可选值包括：
    - `Pin.IN`：引脚配置为输入模式，作为输出时处于高阻抗状态。
    - `Pin.OUT`：引脚配置为（普通）输出模式。
    - `Pin.OPEN_DRAIN`：引脚配置为开漏输出模式。开漏输出的工作方式为：输出值为 `0` 时引脚为低电平；输出值为 `1` 时引脚处于高阻抗状态。并非所有硬件都支持此模式，或仅部分引脚支持此功能。
    - `Pin.ALT`：引脚配置为复用功能模式（具体功能与硬件相关）。配置为此模式的引脚不适用其它 Pin 方法（`Pin.init()`除外），调用其他方法可能导致未定义行为或硬件特定结果。并非所有硬件都支持此模式。
    - `Pin.ALT_OPEN_DRAIN`：与 `Pin.ALT` 类似，但引脚配置为开漏模式。并非所有硬件都支持此模式。
    - `Pin.ANALOG`：引脚配置为模拟输入模式（参见ADC类）。
  - `pull`：指定引脚是否连接（弱）上拉/下拉电阻，可选值包括：
    - `None`：无上下拉电阻。
    - `Pin.PULL_UP`：使能上拉电阻。
    - `Pin.PULL_DOWN`：使能下拉电阻。
  - `value`：仅在 `Pin.OUT` 和 `Pin.OPEN_DRAIN` 模式下有效，用于指定初始输出电平（若未指定，则引脚外设状态保持不变）。
  - `drive`：指定引脚的输出驱动能力，可选值如 `Pin.DRIVE_0`、`Pin.DRIVE_1` 等（驱动强度递增）。实际驱动电流因硬件而异，并非所有硬件都支持此参数。
  - `alt`：为引脚指定复用功能（具体取值因硬件而异），仅在 `Pin.ALT` 和 `Pin.ALT_OPEN_DRAIN` 模式下有效。当引脚支持多个复用功能时需使用此参数；如果仅支持一个复用功能，则无需指定。并非所有硬件都支持此参数。

   Pin 类允许为特定引脚设置复用功能，但不支持对此类引脚执行进一步操作。配置为复用功能模式的引脚通常不作为 GPIO 使用，而是由其他硬件外设驱动。此类引脚仅支持通过调用构造函数或 `Pin.init()` 方法重新初始化。若将复用功能模式的引脚重新初始化为 `Pin.IN`、`Pin.OUT` 或 `Pin.OPEN_DRAIN` 模式，复用功能将被移除。
 

#### 方法

- Pin.`init`(mode=-1, pull=-1, *, value=None, drive=0, alt=-1)

  使用给定参数重新初始化引脚。只有那些被指定的参数才会被设置。引脚外设的其余状态将保持不变。有关参数的详细信息，请参阅构造函数文档。

  返回 `None`。
<br><br>

- Pin.`value`([ x ])

  设置或读取引脚的值。

  如果不带参数，则读取引脚的数字逻辑电平，返回 0 或 1 对应于低电压和高电压信号。此方法的行为取决于引脚的模式：
  - `Pin.IN` - 返回当前引脚的实际输入值。
  - `Pin.OUT` - 该方法的行为和返回值未定义。
  - `Pin.OPEN_DRAIN` - 如果引脚处于'0' 状态，则该方法的行为和返回值未定义；否则，如果引脚处于'1'状态，该方法返回当前引脚的实际输入值。

  如果提供参数 `x`，则设置引脚的数字逻辑电平。参数 `x` 可以是任何可转换为布尔值的内容。如果它转换为 `True`，则引脚设置为 '1' 状态，否则设置为 '0' 状态。此方法的行为取决于引脚的模式：
  - `Pin.IN` - 参数存储在引脚的输出缓冲区中。引脚状态不变，仍保持高阻抗状态。一旦引脚更改为 `Pin.OUT` 或 `Pin.OPEN_DRAIN` 模式，存储的参数将在引脚上变为有效。
  - `Pin.OUT` - 输出缓冲区立即设置为给定值。
  - `Pin.OPEN_DRAIN` - 如果值为 '0'，则引脚设置为低电压状态。否则，引脚设置为高阻抗状态。

  设置输出状态时，此方法返回 `None`。
<br><br>

- Pin.`__call__`([ x ])

  Pin 对象是可调用的。调用方法提供了一种(快速)便捷方式来设置和读取引脚。它等同于 `Pin.value([x])`。
<br><br>

- Pin.`on`()

  将引脚设置为高电平。
<br><br>

- Pin.`off`()

  将引脚设置为低电平。
<br><br>

- Pin.`irq`(handler=None, trigger=Pin.IRQ_FALLING | Pin.IRQ_RISING, *, priority=1, wake=None, hard=False)

  配置中断处理程序，当引脚的触发源激活时调用。如果引脚模式是 `Pin.IN`，则触发源是引脚上的外部值；如果引脚模式是 `Pin.OUT`，则触发源是引脚的输出缓冲区；如果引脚模式是 `Pin.OPEN_DRAIN`，则触发源是 '0' 状态的输出缓冲区和 '1' 状态的外部引脚值。

  参数如下：
  - `handler` 是一个可选函数，当中断触发时调用。处理程序必须接受一个参数，即 Pin 实例。
  - `trigger` 配置可以生成中断的事件。可能的值是：
    - `Pin.IRQ_FALLING` 下降沿中断。
    - `Pin.IRQ_RISING` 上升沿中断。
    - `Pin.IRQ_LOW_LEVEL` 低电平中断。
    - `Pin.IRQ_HIGH_LEVEL` 高电平中断。

    这些值可以用 OR 运算符组合在一起，以在多个事件上触发。

  - `priority` 设置中断的优先级级别。它可以接受的值是特定于硬件的，但较大的值始终代表较高的优先级。
  - `wake` 选择此中断可以唤醒系统的电源模式。它可以是 `machine.IDLE`、`machine.SLEEP` 或 `machine.DEEPSLEEP`。这些值也可以用 OR 运算符组合在一起，使引脚在多种电源模式下生成中断。
  - `hard` 如果为 `True`，则使用硬件中断。这减少了引脚变化和调用处理程序之间的延迟。硬件中断处理程序无法分配内存，并非所有移植版本都支持此参数。
  
  此方法返回一个回调对象。

以下方法不是核心 Pin API 的一部分，仅在某些硬件上实现。

- Pin.`low`()
  
  将引脚设置为低电平。
<br><br>

- Pin.`high`()

  将引脚设置为高电平。
<br><br>

- Pin.`mode`([ mode ])

  获取或设置引脚模式。有关mode参数的详细信息，请参阅构造函数部分。
<br><br>

- Pin.`pull`([ pull ])

  获取或设置引脚的上拉/下拉状态。有关pull参数的详细信息，请参阅构造函数部分。
<br><br>

- Pin.`drive`([ drive ])

  获取或设置引脚的驱动强度。有关drive参数的详细信息，请参阅构造函数部分。
<br><br>

- Pin.`toggle`()

  将引脚输出从 "0" 切换到 "1"，或从 "1" 切换到 "0"。

#### 常数

下面的常数用于配置 pin 对象。注意并非所有的移植版本都可以使用全部的常数。

- Pin.`IN`
- Pin.`OUT`
- Pin.`OPEN_DRAIN`
- Pin.`ALT`
- Pin.`ALT_OPEN_DRAIN`
- Pin.`ANALOG`

  选择引脚模式。
<br><br>

- Pin.`PULL_UP`
- Pin.`PULL_DOWN`
- Pin.`PULL_HOLD`

  选择上拉/下拉电阻。`None` 代表无上拉或下拉。
<br><br>

- Pin.`DRIVE_0`
- Pin.`DRIVE_1`
- Pin.`DRIVE_2`

  选择引脚驱动强度。可能会定义其他驱动常量，数字越大对应驱动强度越强。
<br><br>

- Pin.`IRQ_FALLING`
- Pin.`IRQ_RISING`
- Pin.`IRQ_LOW_LEVEL`
- Pin.`IRQ_HIGH_LEVEL`

  选择 IRQ 触发类型。


### class Signal – 控制和感知外部 I/O 设备

Signal 类是 Pin 类的简单扩展。与只能处于"绝对" 0 和 1 状态的 Pin 不同，Signal 可以处于"有效"（开启）或"无效"（关闭）状态，同时可以选择是否取反（低电平有效）。换句话说，它在 Pin 功能的基础上增加了逻辑反相支持。虽然这看似是一个简单的扩展，但它以可移植方式支持不同开发板的各种简单数字功能，而这也 MicroPython 的主要目标之一。无论用户使用的是高电平有效还是低电平有效的 LED，是常开还是常闭的继电器 —— 你都可以开发一个独立的外观简洁的应用程序，使其与所有这些设备兼容，并在应用程序的配置文件中用几行代码处理硬件配置差异。

**示例：**
```python
from machine import Pin, Signal

# 假设有一个高电平有效的LED连接在引脚0上
led1_pin = Pin(0, Pin.OUT)
# ...以及一个低电平有效的LED连接在引脚1上
led2_pin = Pin(1, Pin.OUT)

# 现在如果使用Pin类点亮这两个LED，需要给它们设置不同的值
led1_pin.value(1)
led2_pin.value(0)

# Signal类允许抽象掉高电平/低电平有效的差异
led1 = Signal(led1_pin, invert=False)
led2 = Signal(led2_pin, invert=True)

# 现在点亮它们的代码看起来是一样的
led1.value(1)
led2.value(1)

# 更简洁的写法：
led1.on()
led2.on()
```

Signal vs Pin 的使用指南：
- **使用 Signal**：如果你想控制简单的开关设备（包括软件PWM！），如 LED、多段显示器、继电器、蜂鸣器，或读取简单的二进制传感器，如常开或常闭按钮、上拉或下拉的开关、干簧管、湿度/火焰探测器等。总之，如果你有一个需要 GPIO 访问的实际物理设备/传感器，你可能应该使用 Signal。
- **使用 Pin**：如果你要实现一个更底层的协议或总线来与更复杂的设备通信。

Pin 和 Signal 的区分源于上述用例和 MicroPython 的架构：Pin 提供最低的开销，这在实现位操作协议时可能很重要。而 Signal 在 Pin 的基础上增加了额外的灵活性，代价是轻微的开销（比你在 Python 中手动处理高电平有效与低电平有效的差异要小得多！）。此外，Pin 是一个需要为每个支持的开发板实现的低级对象，而 Signal 是一个高级对象，一旦 Pin 实现了，Signal 就可以随时使用。

再次强调，提供这个类是为了让开发者不必处理诸如低电平有效与高电平有效信号之类的乏味差异，并允许其他用户共享和使用你的应用程序，而不会因为他们的 LED 或继电器的接线方式略有不同而无法使用你的应用程序而感到沮丧。

#### 构造函数

- class machine.`Signal`(pin_obj, invert=False)
- class machine.`Signal`(pin_arguments..., *, invert=False)

  创建一个Signal对象。有两种创建方式：
  - 通过包装现有的 Pin 对象，这是一种适用于任何开发板的通用方法。
  - 直接将所需的 Pin 参数传递给 Signal 构造函数，无需创建中间的 Pin 对象。许多（但不是全部）开发板支持这种方式。

  **参数：**
  - `pin_obj`：已有的 Pin 对象。
  - `pin_arguments`：与可以传递给 Pin 构造函数的参数相同。
  - `invert`：如果为 `True`，信号将被取反（低电平有效）。

#### 方法

- Signal.`value`([ x ])

  此方法允许根据是否提供参数 x 来设置和读取信号的值。
  - 如果省略参数，则此方法读取信号电平，1 表示信号有效（激活），0 表示信号无效。
  - 如果提供参数，则此方法设置信号电平。参数 `x` 可以是任何可转换为布尔值的内容。如果转换为 `True`，则信号为有效状态，否则为无效状态。

  信号有效状态与底层引脚上实际逻辑电平之间的对应关系取决于信号是否被取反（低电平有效）。对于非取反信号，有效状态对应逻辑 1，无效对应逻辑 0。对于取反/低电平有效信号，有效状态对应逻辑 0，无效对应逻辑 1。
<br><br>

- Signal.`on`()

  激活信号。
<br><br>

- Signal.`off`()

  关闭信号。


### class ADC – 模数转换

ADC类为模数转换功能提供接口，可对连续电压进行采样并将其转换为离散值。  

如需对 ADC 采样进行额外控制，请参见 `machine.ADCBlock`。  

**示例用法：**  
```python  
from machine import ADC  

adc = ADC(pin)       # 创建使用 pin 的ADC对象  
val = adc.read_u16() # 读取0-65535范围内的原始模拟值  
val = adc.read_uv()  # 读取以微伏为单位的模拟值  
```  

#### 构造函数  

- class machine.`ADC`(id, *, sample_ns, atten)  

  访问关联到 `id` 标识的 ADC。此 `id` 可以是整数（通常指定通道号）、Pin 对象或底层机器支持的其他值。

  如果提供额外的关键字参数，将配置 ADC 的各个方面。若未提供，这些设置将采用之前的值或默认值。相关设置包括：  
  - `sample_ns`：采样时间（纳秒）。  
  - `atten`：指定输入衰减。  

#### 方法

- ADC.`init`(*, sample_ns, atten)

  将给定设置应用于ADC。仅会更改指定的参数（请参见上面 ADC 构造函数）。
<br><br>

- ADC.`block`()

  返回与此 ADC 对象关联的 `ADCBlock` 实例。

  此方法仅在支持 ADCBlock 类时存在。
<br><br>

- ADC.`read_u16`()

  获取模拟读数并返回 0-65535 范围内的整数。返回值表示 ADC 获取的原始读数，其缩放方式为最小值对应 0，最大值对应 65535。
<br><br>

- ADC.`read_uv`()

  获取模拟读数并返回以微伏为单位的整数值。该值是否经过校准以及校准方式取决于具体硬件。


### class ADCBlock – 控制 ADC 外设

ADCBlock 类提供对 ADC 外设的访问，该外设包含多个可用于采样模拟值的通道。它允许对执行实际采样的 `machine.ADC` 对象的配置进行更精细的控制

此类并非始终可用。

**示例用法：**  
```python  
from machine import ADCBlock

block = ADCBlock(id, bits=12)   # 创建一个12位分辨率的ADCBlock  
adc = block.connect(4, pin)     # 将通道4连接到给定引脚  
val = adc.read_uv()             # 读取模拟值  
```  

#### 构造函数

- class machine.`ADCBlock`(id, *, bits)
  
  访问由 `id` 标识的 ADC 外设，id 可以是整数或字符串。  

  如果提供 `bits` 参数，则设置转换过程的分辨率（以位为单位）。若未指定，则使用之前的或默认的分辨率。


#### 方法  

- ADCBlock.`init`(*, bits)

  配置 ADC 外设。`bits` 参数将设置转换过程的分辨率。
<br><br>

- ADCBlock.`connect`(channel, *, ...)
- ADCBlock.`connect`(source, *, ...)
- ADCBlock.`connect`(channel, source, *, ...)

  连接 ADC 外设上的通道准备采样，并返回表示该连接的 ADC 对象。

  `channel` 参数必须为整数，`source` 参数必须为可连接用于采样的对象（例如 Pin）。

  如果仅提供 `channel`，则配置该通道用于采样。

  如果仅提供 `source`，则将该对象连接到默认通道并准备采样。

  如果同时提供 `channel` 和 `source`，则将它们连接在一起并准备采样。

  任何额外的关键字参数将通过 `init` 方法用于配置返回的 ADC 对象。



### class PWM – 脉冲宽度调制

提供脉冲宽度调制输出功能。  

**示例用法：**  
```python  
from machine import PWM  
pwm = PWM(pin, freq=50, duty_u16=8192) # 在引脚上创建PWM对象，
                                       # 设置频率50Hz、占空比12.5%  
pwm.duty_u16(32768)                    # 将占空比设置为50%

# 用周期200微秒、脉冲宽度5微秒重新初始化  
pwm.init(freq=5000, duty_ns=5000) 

pwm.duty_ns(3000)                      # 将脉冲宽度设置为3微秒  

pwm.deinit()                           # 关闭PWM输出  
```

#### 构造函数  

- class machine.`PWM`(dest, *, freq, duty_u16, duty_ns, invert=False)

  使用以下参数构造并返回新的PWM对象：  
  - `dest`：PWM输出的目标实体，通常为 `machine.Pin` 对象，某些移植版可能允许其他值（如整数）。
  - `freq`：整数，表示 PWM 的频率（单位：Hz）。
  - `duty_u16`：占空比，以 `duty_u16 / 65535` 的比例表示。
  - `duty_ns`：脉冲宽度（单位：纳秒）。
  - `invert`：若为`True`，则反转输出电平。

  **注意**：若多个PWM对象共享同一底层PWM发生器（取决于硬件），设置 `freq` 可能会影响其他对象。`duty_u16` 和 `duty_ns`一次只能指定其中一个。`invert` 仅在 esp32、mimxrt、nrf、rp2、samd 和 zephyr 等硬件可用。

#### 方法

- PWM.`init`(*, freq, duty_u16, duty_ns)

  修改 PWM 对象的设置，参数详情见构造函数说明。
<br><br>

- PWM.`deinit`()

  关闭 PWM 输出。
<br><br>

- PWM.`freq`([ value ])

  获取或设置 PWM 输出的频率：
  - 无参数时，返回当前频率（单位：Hz）。  
  - 有参数时，将频率设置为指定值（单位：Hz）。若频率超出有效范围，可能抛出`ValueError`。  
<br><br>

- PWM.`duty_u16`([ value ])

  获取或设置 PWM 输出的占空比，以 0 到 65535 之间的无符号 16 位数值表示：  
  - 无参数时，返回当前占空比数值。  
  - 有参数时，按 `value / 65535` 的比例设置占空比。  
<br><br>

- PWM.`duty_ns`([ value ])

  获取或设置 PWM 输出的脉冲宽度（单位：纳秒）：
  - 无参数时，返回当前脉冲宽度数值。  
  - 有参数时，将脉冲宽度设置为指定值。
<br><br>

#### PWM 的限制

- **频率精度限制**：由于硬件的离散特性，并非所有频率都能绝对精确生成。通常，PWM频率通过将某个整数基频除以整数分频器得到。例如，基频为80MHz时，若需要300kHz的PWM频率，分频器需为非整数（80000000/300000≈266.67），取整后实际频率会有误差（如299625.5Hz或300751.9Hz）。  
  部分硬件（如RP2040）使用分数分频器，可在高频段通过在两个相邻值间切换脉冲宽度来提高频率精度，但会牺牲频谱纯度。
- **占空比精度限制**：占空比同样受离散特性影响，无法绝对精确。在大多数硬件平台上，占空比会在下一个频率周期生效，因此测量前需等待超过"1/频率"的时间。
- **频率与分辨率的关联性**：频率和占空比分辨率通常相互制约。频率越高，可用的占空比分辨率越低，反之亦然。例如，300kHz的PWM频率可能仅支持8位占空比分辨率（而非预期的16位），此时 `duty_u16` 的低8位无效。  
  ```python  
  # 以下两种设置会生成相同的50%占空比  
  pwm = PWM(Pin(13), freq=300_000, duty_u16=65536//2)  
  pwm = PWM(Pin(13), freq=300_000, duty_u16=65536//2 + 255)  
  ```

### class UART - 双工串行通信总线

UART 实现了标准的 UART/USART 双工串行通信协议。在物理层面，它由两条线路组成：RX（接收）和 TX（发送）。通信单位是字符（不要与字符串中的字符混淆），其宽度可以是 8 位或 9 位。

可以使用以下方式创建和初始化 UART 对象：

```py
from machine import UART

uart = UART(1, 9600)                          # 使用给定波特率初始化
uart.init(9600, bits=8, parity=None, stop=1)  # 使用给定参数初始化
```

不同开发板支持的参数有所不同：

- Pyboard：位数可以是 7、8 或 9。停止位可以是 1 或 2。当 `parity=None` 时，只支持 8 位和 9 位；启用校验位时，只支持 7 位和 8 位。
- WiPy/CC3200：位数可以是 5、6、7、8。停止位可以是 1 或 2。

UART 对象的行为类似于流对象，可以使用准的流方法进行读写操作：

```py
uart.read(10)      # 读取10个字符，返回一个bytes对象
uart.read()        # 读取所有可用字符
uart.readline()    # 读取一行
uart.readinto(buf) # 读取并存储到给定的缓冲区中
uart.write('abc')  # 写入3个字符
```

#### 构造函数

- class machine.`UART`(id, ...)

  构造具有给定 `id` 的 UART 对象。  


#### 方法

- UART.`init`(baudrate=9600, bits=8, parity=None, stop=1, *, ...)

  使用给定参数初始化 UART 总线：
  - `baudrate`：波特率（时钟速率）。
  - `bits`：每个字符的位数，可以是 7、8 或 9。
  - `parity`：校验位，可以是 None（无校验）、0（偶校验）或 1（奇校验）。
  - `stop`：停止位的数量，可以是 1 或 2。

  不同移植版可能支持的其他关键字参数：
  - `tx`：指定要使用的 TX 引脚。
  - `rx`：指定要使用的 RX 引脚。
  - `rts`：指定用于硬件接收流控制的 RTS（输出）引脚。
  - `cts`：指定用于硬件发送流控制的 CTS（输入）引脚。
  - `txbuf`：指定 TX 缓冲区的字符长度。
  - `rxbuf`：指定 RX 缓冲区的字符长度。
  - `timeout`：指定等待第一个字符的时间（以毫秒为单位）。
  - `timeout_char`：指定等待字符之间的时间（以毫秒为单位）。
  - `invert`：指定要反转的信号：
      - `0`：不反转（两条信号的空闲状态均为逻辑高）。
      - `UART.INV_TX`：反转 TX 信号（TX 信号的空闲状态现在为逻辑低）。
      - `UART.INV_RX`：反转 RX 信号（RX 信号的空闲状态现在为逻辑低）。
      - `UART.INV_TX | UART.INV_RX`： 同时反转 TX 和 RX。
  - `flow`：指定要使用的硬件流控制信号。该值是一个位掩码：
      - `0`：忽略硬件流控制信号。
      - `UART.RTS`：通过使用 RTS 输出引脚来启用接收流控制，以指示接收 FIFO 是否有足够空间接收更多数据。
      - `UART.CTS`：当 CTS 输入引脚信号表明接收器缓冲区空间不足时，通过暂停传输来启用发送流控制。
      - `UART.RTS | UART.CTS`：同时启用两者，实现完整的硬件流控制。

  WiPy 仅支持以下仅限关键字参数：
    - `pins`：一个包含 4 个或 2 个元素的列表，指示 TX、RX、RTS 和 CTS 引脚（按此顺序）。如果希望 UART 以有限功能运行，则任何引脚都可以为 `None`。如果给出了 RTS 引脚，则必须同时给出 RX 引脚。CTS 引脚同理。如果未给出引脚，则采用默认的 TX 和 RX 引脚集，并禁用硬件流控制。如果 `pins` 为 `None`，则不会进行引脚分配。  
<br>

  **注意**：可以在同一对象上多次调用 `init()` 以动态重新配置 UART。这允许使用单个 UART 外设为连接到不同 GPIO 引脚的不同设备提供服务。在这种情况下，一次只能服务一个设备。另外，不要调用 `deinit()`，因为这将导致无法再次调用 `init()`。
<br><br>

- UART.`deinit`()

  关闭 UART 总线。

  **注意**：在调用 `deinit()` 之后，将无法在该对象上调用 `init()`。在这种情况下，需要创建一个新实例。
<br><br>

- UART.`any`()

  返回一个整数，表示无需阻塞即可读取的字符数量。如果没有可读取数据，则返回 0；如果有数据，则返回一个正数。即使有多个字符可供读取，该方法也可能返回 1。

  如需更复杂的可用数据查询，请使用 `select.poll`：
  ```python
  poll = select.poll()
  poll.register(uart, select.POLLIN)
  poll.poll(timeout)
  ```
<br><br>

- UART.`read`([ nbytes ])

  读取数据。如果指定了 `nbytes`，则最多读取指定字节；否则，读取尽可能多的数据。如果达到超时时间，可能会提前返回。超时时间可在构造函数中配置。

  返回值：包含读取字节的 bytes 对象。超时返回 `None`。
<br><br>

- UART.`readinto`(buf [, nbytes ])

  将数据读取到 `buf`。如果指定了 `nbytes`，则最多读取指定字节。否则，最多读取 `len(buf)` 字节。如果达到超时时间，可能会提前返回。超时时间可在构造函数中配置。

  返回值：读取并存储到 `buf` 中的字节数，超时返回 `None`。
<br><br>

- UART.`readline`()

  读取一行，以换行符结尾。如果达到超时时间，可能会提前返回。超时时间可在构造函数中配置。

  返回值：读取的行，超时返回 `None`。
<br><br>

- UART.`write`(buf)

  将字节缓冲区写入总线。

  返回值：写入的字节数，超时返回 `None`。
<br><br>

- UART.`sendbreak`()

  在总线上发送中断条件。这会使总线保持低电平的时间比正常传输一个字符所需的时间更长。
<br><br>

- UART.`flush`()

  等待直到所有数据都已发送。如果超时，将引发异常。超时持续时间取决于 `tx` 缓冲区大小和波特率。除非启用流控制，否则不应发生超时。

  **注意**：对于 esp8266 和 nrf，在发送最后一个字节时会返回。因此如有需要，必须在调用脚本中添加一个字符的等待时间。

  可用性：rp2、esp32、esp8266、mimxrt、cc3200、stm32、nrf、renesas-ra。
<br><br>

- UART.`txdone`()

  指示所有数据是否已发送或没有数据传输正在进行。在这种情况下，返回 `True`。如果正在进行数据传输，则返回 `False`。

  **注意**：对于 esp8266 和 nrf，即使传输的最后一个字节仍在发送，调用也可能返回 `True`。如有需要，必须在调用脚本中添加一个字符的等待时间。

  可用性：rp2、esp32、esp8266、mimxrt、cc3200、stm32、nrf、renesas-ra。
<br><br>

- UART.`irq`(handler=None, trigger=0, hard=False)

  配置在 UART 事件发生时调用的中断处理程序。

  参数：
  - `handler`：可选函数，在中断事件触发时调用。处理程序必须接受一个参数，即 UART 实例。
  - `trigger`：配置可生成中断的事件。可能的值是以下一个或多个的掩码：
    - `UART.IRQ_RXIDLE`：在接收至少一个字符后，RX线路变为空闲时中断。
    - `UART.IRQ_RX`：每个接收字符后中断。
    - `UART.IRQ_TXIDLE`：在消息的最后一个字符发送后或发送时中断。
    - `UART.IRQ_BREAK`：在RX检测到中断状态时中断。
  - `hard`：如果为 `True`，则使用硬件中断。这减少了引脚变化与调用处理程序之间的延迟。硬件中断处理程序可能无法分配内存。

  返回一个 irq 对象。

  由于硬件限制，并非所有版本都支持所有触发事件。

  触发事件的可用性

  | Port / Trigger | IRQ_RXIDLE | IRQ_RX | IRQ_TXIDLE | IRQ_BREAK |
  |----------------|------------|--------|------------|-----------|
  | CC3200         |            | yes    |            |           |
  | ESP32          | yes        | yes    |            | yes       |
  | MIMXRT         | yes        |        | yes        |           |
  | NRF            |            | yes    | yes        |           |
  | RENESAS-RA     | yes        | yes    |            |           |
  | RP2            | yes        |        | yes        | yes       |
  | SAMD           | yes        | yes    | yes        |           |
  | STM32          | yes        | yes    |            |           

  **注意**：
  - ESP32 不支持 `hard=True` 选项。
  - rp2 的 `UART.IRQ_TXIDLE` 仅在消息长度超过 5 个字符时触发，并且在仍有 5 个字符要发送时触发。
  - rp2 的 `UART.IRQ_BREAK` 需要接收有效字符才能再次触发。
  - SAMD 的 `UART.IRQ_TXIDLE` 在发送最后一个字符时触发。
  - 在 STM32F4xx MCU 上，使用触发 `UART.IRQ_RXIDLE` 时，处理程序将在第一个字符后调用一次，然后在消息结束且信号空闲时调用一次。

  可用性：cc3200、esp32、mimxrt、nrf、renesas-ra、rp2、samd、stm32。

#### 常量

- UART.`RTS`
- UART.`CTS`

  流控制选项。

  可用性：esp32、mimxrt、renesas-ra、rp2、stm32。
<br><br>

- UART.`IRQ_RXIDLE`
- UART.`IRQ_RX`
- UART.`IRQ_TXIDLE`
- UART.`IRQ_BREAK`

  IRQ触发源。

  可用性：renesas-ra、stm32、esp32、rp2040、mimxrt、samd、cc3200。
  

### class SPI - 串行外设接口总线协议（控制器端）

SPI 是一种由控制器驱动的同步串行协议。在物理层面，总线由 3 个信号线组成：SCK（时钟）、MOSI（主出从入）、MISO（主入从出）。多个设备可共享同一总线，每个设备需单独配备第 4 个信号 CS（片选），用于在总线上选择特定通信设备。CS  信号的控制需通过用户代码实现（使用 `machine.Pin` 类）。

通过 `machine.SPI` 和 `machine.SoftSPI` 类可分别实现硬件和软件 SPI。硬件 SPI 利用系统底层硬件实现读写操作，通常高效而快速，但可能有引脚使用限制；软件 SPI 通过位操作实现，可在任意引脚运行，但效率较低。这两个类方法相同，主要区别在于构造方式。

使用示例：

```py
from machine import SPI, Pin  
spi = SPI(0, baudrate=400000)       # 创建 SPI 外设 0，频率400kHz
                                    # 根据具体场景可能需要额外参数配置总线特性和/或引脚
cs = Pin(4, mode=Pin.OUT, value=1)  # 在引脚 4 创建片选信号  

try:  
    cs(0)                           # 选中外设
    spi.write(b"12345678")          # 写入8字节，忽略接收数据  
finally:  
    cs(1)                           # 取消片选

try:  
    cs(0)                           # 选中外设  
    rxdata = spi.read(8, 0x42)      # 读取8字节，同时每个字节写入0x42  
finally:  
    cs(1)                           # 取消片选

rxdata = bytearray(8)  
try:  
    cs(0)                           # 选中外设  
    spi.readinto(rxdata, 0x42)      # 读取8字节到缓冲区，同时每个字节写入0x42  
finally:  
    cs(1)                           # 取消片选

txdata = b"12345678"  
rxdata = bytearray(len(txdata))  
try:  
    cs(0)                           # 选中外设  
    spi.write_readinto(txdata, rxdata)  # 同步读写字节  
finally:  
    cs(1)                           # 取消片选
```

#### 构造函数

- class machine.`SPI`(id, ...)

  在指定总线 `id` 上构造 SPI 对象。`id` 值取决于具体硬件，通常用 0、1 等标识硬件 SPI 模块。若未提供额外参数，SPI 对象仅创建未初始化（保留总线最后一次初始化的配置）；若提供参数，则直接初始化总线（参数说明见 `init` 方法）。

- class machine.`SoftSPI`(baudrate=500000, *, polarity=0, phase=0, bits=8, firstbit=MSB, sck=None, mosi=None, miso=None)

  构造新的软件 SPI 对象，需至少指定 `sck`、`mosi`、`miso` 引脚参数以初始化总线，其他参数说明见 `SPI.init` 方法。

#### 方法

- SPI.`init`(baudrate=1000000, *, polarity=0, phase=0, bits=8, firstbit=SPI.MSB, sck=None, mosi=None, miso=None, pins=(SCK, MOSI, MISO))

  使用以下参数初始化 SPI 总线：

  - `baudrate`：SCK 时钟速率。
  - `polarity`：时钟空闲电平（0 或 1）。
  - `phase`：采样时钟边沿（0 = 第一个边沿，1 = 第二个边沿）。
  - `bits`：每次传输的位宽（所有硬件都支持 8 位）。
  - `firstbit`：传输顺序（`SPI.MSB` = 高位先行，`SPI.LSB` = 低位先行）。
  - `sck/mosi/miso`：总线信号引脚（硬件 SPI 的引脚通常由 `id` 指定固定的 GPIO，某些情况下硬件可以指定 2-3 个第二功能引脚，软件 SPI （`id`=-1）支持任意引脚分配。
  - `pins`：WiPy 中不支持单独指定 `sck/mosi/miso`，需通过 pins 元组传入。

  注意：硬件 SPI 的实际时钟频率可能低于请求的波特率（取决于具体硬件），可通过打印 SPI 对象查看实际速率。
<br><br>

- SPI.`deinit`()

  关闭 SPI 总线。
<br><br>

- SPI.`read`(nbytes, write=0x00)

  读取 `nbytes` 字节，同时写入 `write` 指定的单字节。返回包含读取数据的 bytes 对象。
<br><br>

- SPI.readinto(buf, write=0x00)

  将数据读取到缓冲区 `buf`，同时写入 `write` 指定的单字节。返回 `None`（WiPy 中返回读取字节数）。
<br><br>

- SPI.`write`(buf)

  写入缓冲区 `buf` 中的数据。返回 `None`（WiPy 中返回写入字节数）。
<br><br>
  
- SPI.`write_readinto`(write_buf, read_buf)

  从 `write_buf` 写入数据，同时读取到 `read_buf`。两个缓冲区可相同也可不同，但长度需一致，。返回 `None`（WiPy 中返回写入字节数）。


#### 常量

- SPI.`CONTROLLER`

  用于将 SPI 总线初始化为控制器模式（仅 WiPy 使用）。
<br><br>

- SPI.`MSB`
- SoftSPI.`MSB`

  设置传输顺序为高位先行。
<br><br>

- SPI.`LSB`
- SoftSPI.`LSB`

  设置传输顺序为低位先行。


### class I2C - 两线串行协议

I2C 是一种用于设备间通信的两线协议。在物理层面，它由两根信号线组成：SCL（时钟线）和 SDA（数据线）。

I2C 对象依附于特定总线创建。创建时可以初始化，也可以稍后再初始化。打印I2C对象，可获取其配置信息。

通过 `machine.I2C` 和 `machine.SoftI2C` 类可以实现硬件和软件两种 I2C 方式。硬件 I2C 利用系统的底层硬件支持来进行读写操作，通常高效且速度快，但在可用引脚方面有限制。软件 I2C 通过位操作来实现，能在任意引脚上使用，效率稍低。这两个类具有相同的使用方法，主要差异在于构造方式。

**注意**：I2C 总线工作时，在 SDA 和 SCL 上需要上拉电路。通常使用 1 - 10 千欧的电阻，将 SDA/SCL 分别连接到 Vcc。如果没有上拉电阻，I2C 的行为就无法确定，可能出现从阻塞、意外看门狗复位到数值错误等各种情况。很多时候，MCU 开发板或传感器扩展板已经内置了上拉电路，但并非所有情况都是如此。

示例用法：
```python
from machine import I2C

i2c = I2C(freq=400000)     # 以400kHz频率创建I2C外设
                           # 依据硬件不同，可能需要额外参数
                           # 来选择要使用的外设和/或引脚

i2c.scan()                 # 扫描外设，返回7位地址列表

i2c.writeto(42, b'123')    # 向7位地址为42的外设写入3字节数据
i2c.readfrom(42, 4)        # 从7位地址为42的外设读取4字节数据

i2c.readfrom_mem(42, 8, 3) # 从地址42的外设内存地址8开始
                           # 读取3字节数据
i2c.writeto_mem(42, 2, b'\x10') # 从外设的地址2开始
                           # 向外设42的内存写入1字节数据
```

#### 构造函数：

- class machine.`I2C`(id, *, scl, sda, freq=400000, timeout=50000)

  利用以下参数构造并返回一个新的 I2C 对象：

  - `id`：用于标识特定的 I2C 外设。允许的值取决于具体的硬件。
  - `scl`：引脚对象，用于指定 SCL 要使用的引脚。
  - `sda`：引脚对象，用于指定 SDA 要使用的引脚。
  - `freq`：用于设置 SCL 最大频率的整数。
  - `timeout`：是I2C事务允许的最大时间（以微秒为单位）。某些硬件不允许使用此参数。

  某些硬件/开发板的 `scl` 和 `sda` 有默认值，可在这个构造函数中修改。而其它硬件/开发板的 `scl` 和 `sda` 值是固定的，无法更改。
<br><br>

- class machine.`SoftI2C`(scl, sda, *, freq=400000, timeout=50000)

  构造一个新的软件 I2C 对象。参数如下：

  - `scl`：引脚对象，用于指定 SCL 要使用的引脚。
  - `sda`：引脚对象，用于指定 SDA 要使用的引脚。
  - `freq`：用于设置 SCL 最大频率的整数。
  - `timeout`：是等待时钟延长（总线上其他设备将 SCL 拉低）的最大时间（以微秒为单位），超过该时间会引发 `OSError`(`ETIMEDOUT`)异常。

#### 通用方法

- I2C.`init`(scl, sda, *, freq=400000)

  使用以下参数初始化I2C总线：
  - `scl`：SCL信号线的引脚对象
  - `sda`：SDA信号线的引脚对象
  - `freq`：SCL时钟频率

  对于硬件I2C，实际时钟频率可能低于请求值，具体取决于平台硬件。可通过打印I2C对象查看实际频率。
<br><br>

- I2C.`deinit`()

  关闭I2C总线。  
<br><br>

- I2C.`scan`()

  扫描 0x08 至 0x77 范围内的所有 I2C 地址，并返回响应设备的列表。当设备在接收到其地址（含写位）后将 SDA 信号拉低，则视为响应。


#### 底层I2C操作

以下方法实现了 I2C 控制器的底层总线操作，可组合使用以构建任意 I2C 事务。如需更精细的总线控制，可使用这些方法；否则建议使用标准方法（见下文）。  

仅适用于 `machine.SoftI2C` 类

- I2C.`start`()

  在总线上生成起始条件（SCL 为高电平时，SDA 由高变低）。
<br><br>

- I2C.`stop`()

  在总线上生成停止条件（SCL 为高电平时，SDA 由低变高）。
<br><br>

- I2C.`readinto`(buf, nack=True, / )

  从总线读取数据并存储到 `buf` 中。读取的字节数等于 `buf` 的长度。除最后一个字节外，每接收一个字节后都会发送 ACK。若 `nack` 为 `True`，则在接收最后一个字节后发送 NACK；否则发送 ACK（此时外设会假设后续还会继续读取）。
<br><br>

- I2C.`write`(buf)

  将 `buf` 中的数据写入总线。检查每个字节写入后是否收到 ACK，若收到 NACK 则停止发送剩余字节。函数返回收到的 ACK 数量。


#### 标准总线操作

以下方法实现了针对特定外设的标准 I2C 控制器读写操作。

- I2C.`readfrom`(addr, nbytes, stop=True, / )

  从地址为 `addr` 的外设读取 `nbytes` 字节。若 `stop` 为 `True`，则在传输结束时生成停止条件。返回包含读取数据的 bytes 对象。
<br><br>

- I2C.`readfrom_into`(addr, buf, stop=True, / )

  从地址为 `addr` 的外设读取数据到 `buf` 中。读取的字节数等于 `buf` 的长度。若 `stop` 为 `True`，则在传输结束时生成停止条件。  

  返回值：None
<br><br>

- I2C.`writeto`(addr, buf, stop=True, / )

  将 `buf` 中的字节写入地址为 `addr` 的外设。若写入某个字节后收到 NACK，则停止发送剩余字节。若 `stop` 为 `True`，则在传输结束时生成停止条件（即使收到 NACK）。函数返回收到的 ACK 数量。
<br><br>

- I2C.`writevto`(addr, vector, stop=True, / )

  将 `vector` 中的数据写入地址为 `addr` 的外设。`vector` 是实现了缓冲区协议的元组或列表。先发送一次地址，然后依次写入 `vector` 中每个对象的字节。`vector` 中的对象可以长度为 0，此时不产生输出。

  若写入某个对象的字节后收到 NACK，则停止发送剩余字节和后续对象。若 `stop` 为 `True`，则在传输结束时生成停止条件。函数返回收到的ACK数量。


#### 内存操作

部分 I2C 设备可作为内存设备（或寄存器组）进行读写。此时 I2C 事务涉及两个地址：外设地址和内存地址。以下方法为与这类设备通信的便捷函数。

- I2C.`readfrom_mem`(addr, memaddr, nbytes, *, addrsize=8)

  从地址为 `addr` 的外设的 `memaddr` 内存地址开始读取 `nbytes` 字节。`addrsize` 参数指定地址位数。返回包含读取数据的 bytes 对象。
<br><br>

- I2C.`readfrom_mem_into`(addr, memaddr, buf, *, addrsize=8)

  从地址为 `addr` 的外设的 `memaddr` 内存地址开始读取数据到 `buf` 中。读取的数据数量等于 `buf` 的长度。`addrsize` 参数指定地址位数（ESP8266不支持此参数，地址位数固定为8位）。

  返回值：None。
<br><br>

- I2C.`writeto_mem`(addr, memaddr, buf, *, addrsize=8)

  将 `buf` 写入地址为 `addr` 的外设的 `memaddr` 内存地址开始的位置。`addrsize` 参数指定地址位数（ESP8266不支持此参数，地址位数固定为8位）。

  返回值：None。


### class I2S - 声音总线协议

I2S 是一种用于连接数字音频设备的同步串行协议。在物理层面，总线由3条信号线组成：SCK（串行时钟）、WS（字时钟）、SD（串行数据）。I2S 类支持控制器模式操作，不支持外设模式操作。  

当前 I2S 类为技术预览版。在预览期间，欢迎用户提供反馈。基于这些反馈，I2S 类的 API 和实现可能会发生变更。  

I2S 对象的创建与初始化示例  

```python  
from machine import I2S  
from machine import Pin  

# ESP32平台示例  
sck_pin = Pin(14)   # 串行时钟输出  
ws_pin = Pin(13)    # 字时钟输出  
sd_pin = Pin(12)    # 串行数据输出  

# PyBoards平台示例  
sck_pin = Pin("Y6") # 串行时钟输出  
ws_pin = Pin("Y5")  # 字时钟输出  
sd_pin = Pin("Y8")  # 串行数据输出  

# 初始化发送模式（音频输出）  
audio_out = I2S(2,  
                sck=sck_pin, ws=ws_pin, sd=sd_pin,  
                mode=I2S.TX,  
                bits=16,  
                format=I2S.MONO,  
                rate=44100,  
                ibuf=20000)  

# 初始化接收模式（音频输入）  
audio_in = I2S(2,  
               sck=sck_pin, ws=ws_pin, sd=sd_pin,  
               mode=I2S.RX,  
               bits=32,  
               format=I2S.STEREO,  
               rate=22050,  
               ibuf=20000)  
```  

**支持的3种操作模式**  
1. **阻塞模式**  
   ```python  
   num_written = audio_out.write(buf)  # 阻塞直到缓冲区数据发送完毕  
   num_read = audio_in.readinto(buf)   # 阻塞直到缓冲区数据接收完成  
   ```  

2. **非阻塞模式**  
   ```python  
   audio_out.irq(i2s_callback)         # 缓冲区空时调用回调函数  
   num_written = audio_out.write(buf)  # 立即返回  
   audio_in.irq(i2s_callback)          # 缓冲区满时调用回调函数  
   num_read = audio_in.readinto(buf)   # 立即返回  
   ```  

3. **异步模式（asyncio）**  
   ```python  
   swriter = asyncio.StreamWriter(audio_out)  
   swriter.write(buf)  
   await swriter.drain()               # 等待数据发送完成  

   sreader = asyncio.StreamReader(audio_in)  
   num_read = await sreader.readinto(buf)  # 异步读取数据  
   ```  

像 WM8960 或 SGTL5000 等编解码器设备，在与 I2S 类配合使用前需要单独初始化。这些设备通常需要独立的驱动程序，驱动程序还提供音量控制、音频处理等功能。


#### 构造函数  
 
- class machine.`I2S`(id, *, sck, ws, sd, mck=None, mode, bits, format, rate, ibuf)  

  创建指定 `id` 的 I2S 对象，参数说明：  
  - `id`：标识特定的 I2S 总线，具体取值取决于开发板和硬件

  支持所有硬件的关键字参数：  
  - `sck`：串行时钟线的引脚对象  
  - `ws`：字选择线（字时钟）的引脚对象  
  - `sd`：串行数据线的引脚对象  
  - `mck`：主时钟线的引脚对象，主时钟频率为采样率×256  
  - `mode`：操作模式（`I2S.RX`接收或`I2S.TX`发送）  
  - `bits`：采样位深（16位或32位）  
  - `format`：声道格式（`I2S.STEREO`立体声或`I2S.MONO`单声道）  
  - `rate`：音频采样率（Hz），对应 `ws` 信号的频率  
  - `ibuf`：内部缓冲区长度（字节）  

  **底层机制**：对于所有硬件均通过 DMA 在后台持续运行，允许用户应用程序在内部缓冲区与 I2S 外设单元传输数据时执行其他操作。增大内部缓冲区尺寸可延长应用程序执行非 I2S 操作的时间，减少下溢（如写入操作）或上溢（如读取操作）风险。  


#### 方法  

- I2S.`init`(sck, ...)  

  初始化 I2S 总线，参数说明与构造函数一致。
<br><br>

- I2S.`deinit`()  

  关闭 I2S 总线，释放资源。
<br><br>

- I2S.`readinto`(buf)

  将音频采样数据读取到缓冲区 `buf` 中。`buf` 需支持缓冲区协议，如 `bytearray` 或 `array`，字节序为小端序。对于 立体声（STEREO）格式，左声道样本在前，右声道样本在后；单声道（MONO）格式，仅使用左声道样本数据。

  返回实际读取的字节数。
<br><br>

- I2S.`write`(buf)

  将缓冲区 `buf` 中的音频采样数据写入总线。`buf` 需支持缓冲区协议，如 `bytearray` 或 `array`，字节序为小端序。对于 立体声（STEREO）格式，左声道样本在前，右声道样本在后；单声道（MONO）格式，仅使用左声道样本数据。
  
  返回实际写入的字节数。
<br><br>

- I2S.`irq`(handler)  

  设置回调函数。`handler` 触发条件：  
  - 发送模式下缓冲区空时（`write`操作）  
  - 接收模式下缓冲区满时（`readinto`操作）

  设置回调后，`write` 和 `readinto` 方法变为非阻塞模式。

  回调函数在 MicroPython 调度器上下文中运行。
<br><br>

- static I2S.`shift`(*, buf, bits, shift)

  对缓冲区 `buf` 中的所有样本执行按位移位操作。  

  `bits` 指定样本位深（位），`shift` 设置移位位数（正数左移，负数右移）。典型用途是音量控制（每移位1位对应音量变化6dB）。


#### 常量  

- I2S.`RX`

  用于初始化 I2S 总线为接收模式。
<br><br>

- I2S.`TX`
  
  用于初始化I2S总线为发送模式
<br><br>

- I2S.`STEREO`

  用于初始化I2S总线为立体声格式
<br><br>

- I2S.`MONO`

  用于初始化I2S总线为单声道格式



### class RTC - 实时时钟

RTC 是一个独立的时钟，用于跟踪日期和时间。

使用示例：
```py
rtc = machine.RTC()
rtc.datetime((2020, 1, 21, 2, 10, 32, 36, 0))
print(rtc.datetime())
```

#### 构造函数

- class machine.`RTC`(id=0, ...)

  创建一个 RTC 对象。有关初始化参数，请参见 `init` 方法。

#### 方法

- RTC.`datetime`([datetimetuple])

  获取或设置 RTC 的日期和时间。

  不带参数调用时，此方法返回一个包含当前日期和时间的 8 元组。带参数（8 元组）调用时，它会设置日期和时间。8 元组的格式如下：

    (year, month, day, weekday, hours, minutes, seconds, subseconds)

  `subseconds` 字段的含义取决于硬件。
<br><br>

- RTC.`init`(datetime)

  初始化 RTC。`datetime` 是一个形式为以下的元组：

    (year, month, day, hour, minute, second, microsecond, tzinfo)

  必须提供所有八个参数。`microsecond` 和 `tzinfo` 值当前会被忽略，但未来可能会使用。

  可用平台：CC3200、ESP32、MIMXRT、SAMD。stm32 和 renesas-ra 上的 `rtc.init()` 方法仅（重新）启动 RTC，不接受参数。
<br><br>

- RTC.`now`()

  获取当前的日期时间元组。

  可用平台：WiPy。
<br><br>

- RTC.`deinit`()

  将 RTC 重置为 2015 年 1 月 1 日的时间并重新启动。
<br><br>

- RTC.`alarm`(id, time, *, repeat=False)

  设置 RTC 闹钟。time 可以是一个毫秒值，用于将闹钟设置为当前时间 + 未来的毫秒数，也可以是一个日期时间元组。如果传递的 time 是以毫秒为单位的，则可以将 `repeat` 设置为 `True` 使闹钟周期性触发。
<br><br>

- RTC.`alarm_left`(alarm_id=0)

  获取闹钟触发前剩余的毫秒数。
<br><br>

- RTC.`alarm_cancel`(alarm_id=0)

  取消正在运行的闹钟。

  mimxrt 还将此函数作为 `RTC.cancel(alarm_id=0)`，但计划在 MicroPython 2.0 中移除。

- RTC.`irq`(*, trigger, handler=None, wake=machine.IDLE)

  创建一个由实时时钟闹钟触发的中断请求对象。
  - `trigger` 必须是 `RTC.ALARM0`
  - `handler` 是触发回调时要调用的函数。
  - `wake` 指定此中断可以从哪种睡眠模式唤醒系统。
<br><br>

- RTC.`memory`([data])

  `RTC.memory(data)` 会将数据写入 RTC 内存，其中 `data` 是任何支持缓冲区协议的对象（包括 bytes、bytearray、memoryview 和 array.array）。`RTC.memory()` 读取 RTC 内存并返回一个 bytes 对象。

  写入 RTC 用户内存的数据在重启（包括软重置和 `machine.deepsleep()`）后仍然保留。

  esp32 上 RTC 用户内存的默认最大长度为 2048 字节，esp8266 上为 492 字节。

  可用平台：esp32、esp8266。

#### 常量

- RTC.`ALARM0`

  中断触发源


### class Timer – 控制硬件定时器

硬件定时器用于处理时间段和事件的定时。定时器可能是微控制器（MCU）和片上系统（SoCs）中最灵活、异构性最强的硬件类型，不同型号之间差异很大。MicroPython 的 Timer 类定义了一个基准操作，即按给定周期（或以一定延迟后执行一次）执行回调函数，并允许特定开发板定义更多非标准行为（这些行为因此可能无法移植到其他开发板）。

**注意**: 在中断处理程序（IRQ）内无法分配内存，因此在处理程序中引发的异常不会提供太多信息。

如果使用 WiPy 开发板，请参考 machine.TimerWiPy 类而非此类。

#### 构造函数

- class machine.`Timer`(id, /, ...)

  构造一个具有给定 `id` 的新定时器对象。`id` 为 -1 时构造一个虚拟定时器（如果开发板支持）。`id` 不能作为关键字参数传递。

  初始化参数请参见 `init` 方法。

#### 方法

- Timer.`init`(*, mode=Timer.PERIODIC, freq=-1, period=-1, callback=None)

  初始化定时器。示例：
  
  ```python
  def mycallback(t):
      pass
  
  # 1kHz周期运行
  tim.init(mode=Timer.PERIODIC, freq=1000, callback=mycallback)
  
  # 100ms周期运行
  tim.init(period=100, callback=mycallback)
  
  # 1000ms后单次触发
  tim.init(mode=Timer.ONE_SHOT, period=1000, callback=mycallback)
  ```

  关键字参数：
  - `mode` 可以是以下值之一：
    - `Timer.ONE_SHOT` - 定时器运行一次，直到配置的通道周期结束。
    - `Timer.PERIODIC` - 定时器按配置的通道频率周期性运行。
  - `freq` - 定时器频率，单位为 Hz。频率上限取决于具体硬件。当同时提供 `freq` 和 `period` 参数时，`freq` 优先级更高，`period` 参数将被忽略。
  - `period` - 定时器周期，单位为毫秒。
  - `callback` - 定时器周期结束时调用的可调用对象。回调必须接受一个参数，该参数传递定时器对象本身。必须指定回调参数，否则定时器到期时将引发异常：`TypeError: 'NoneType' object isn't callable`
<br><br>

- Timer.`deinit`()

  关闭定时器。停止定时器并禁用定时器外设。

#### 常量

- `Timer.ONE_SHOT`
- `Timer.PERIODIC`

  定时器工作模式。

### class WDT – 看门狗定时器

看门狗定时器（WDT）用于在应用程序崩溃并进入不可恢复状态时重启系统。一旦启动，它就无法以任何方式停止或重新配置。启用后，应用程序必须定期**喂狗**，以防止看门狗超时并复位系统。

**示例用法**：

```python
from machine import WDT

wdt = WDT(timeout=2000)  # 启用一个2秒超时的看门狗
wdt.feed()               # 喂狗以重置计时器
```

**支持该类的硬件**：pyboard、WiPy、esp8266、esp32、rp2040、mimxrt。

#### 构造函数

- class machine.`WDT`(id=0, timeout=5000)
  
  创建并启动一个 WDT 对象。超时时间必须以毫秒为单位指定。一旦启动，超时时间无法更改，也无法停止 WDT。

  **注意**：
    - 在esp8266上无法指定超时时间，由底层系统决定。
    - 在rp2040设备上，最大超时时间为8388毫秒。

#### 方法

- WDT.`feed`()
  
  喂狗以防止系统复位。应用程序应在合适的位置调用此方法，确保仅在验证所有功能正常后才喂狗。


### class SD – 安全数字存储卡（仅适用于 cc3200）

⚠️ 警告：这是一个非标准类，仅在 cc3200 上可用。

SD 类允许配置和启用 WiPy 的存储卡模块，并将其作为文件系统的一部分自动挂载为 `/sd`。有几种引脚组合可用于将 SD 卡插槽连接到 WiPy，构造函数中可指定使用的引脚。请查阅引脚排列图和备用功能表，以获取有关可重映射用于SD卡的引脚的更多信息。

**示例用法**：
```python
from machine import SD
import vfs

# 必须传入时钟、命令和数据0引脚及其对应的备用功能
sd = machine.SD(pins=('GP10', 'GP11', 'GP15'))
vfs.mount(sd, '/sd')      # 挂载SD卡到/sd目录

# 执行常规文件操作
```

#### 构造函数

- class machine.`SD`(id, ...)

  创建一个 SD 对象。初始化参数见 `init()` 方法。

#### 方法

- SD.`init`(id=0, pins=('GP10', 'GP11', 'GP15'))  

  启用 SD 卡。初始化时需提供一个三元组：`(clk_pin, cmd_pin, dat0_pin)`，对应 (时钟引脚, 命令引脚, 数据0引脚)。
<br><br>

- SD.`deinit`()  

  禁用 SD 卡。


### class SDCard - 安全数字存储卡

SD卡是最常见的小尺寸可移动存储介质之一，它有多种尺寸和物理规格。MMC卡是类似的可移动存储设备，而 eMMC 设备则是在电气特性上相近、专为嵌入其他系统而设计的存储设备。这三种存储设备与主机系统通信时采用相同的协议，并且在高层支持方面看起来也完全一样。因此在 MicroPython 中，它们由 `machine.SDCard` 统一实现。

SD 和 MMC 接口均支持通过多种总线宽度进行访问。当使用 1 位宽接口访问时，可以采用 SPI 协议。不同的 MicroPython 硬件平台支持不同的总线宽度和引脚配置，不过对于大多数硬件平台而言，任何给定硬件都存在标准配置。一般情况下，在创建SDCard 对象时不传递任何参数，系统会将接口初始化为当前硬件的默认卡槽。下面列出的参数是为了使用非标准卡槽或非标准引脚分配时可能需要设置的通用参数。不同平台支持的具体参数子集会有所不同。

`SDCard(slot=1, width=1, cd=None, wp=None, sck=None, miso=None, mosi=None, cs=None, cmd=None, data=None, freq=20000000)`

这个类能够通过专用的 SD/MMC 接口硬件或 SPI 通道来访问 SD 或 MMC 存储卡。该类实现了 `vfs.AbstractBlockDev` 定义的块协议，这使得挂载 SD 卡变得十分简单，只需执行以下操作即可：

```python
vfs.mount(machine.SDCard(), "/sd")
```

构造函数包含以下参数：

- `slot`：用于选择要使用的可用接口。若不设置该参数，则会选择默认接口。
- `width`：用于选择 SD/MMC 接口的总线宽度。必须有相应数量的数据引脚连接到 SD 卡。
- `cd`：用于指定卡检测引脚。
- `wp`：用于指定写保护引脚。
- `sck`：用于指定SPI时钟引脚。
- `miso`：用于指定SPI的miso引脚。
- `mosi`：用于指定SPI的mosi引脚。
- `cs`：用于指定SPI的片选引脚。

以下附加参数仅在 ESP32 上存在：

- `cmd`：用于指定 SD 的 CMD 引脚（仅 ESP32-S3 支持）。
- `data`：用于指定 SD 数据总线引脚的列表或元组（仅 ESP32-S3 支持）。
- `freq`：用于选择 SD/MMC 接口的频率，单位为 Hz。

#### 实现细节

不同硬件上的 SDCard 类支持上述选项的不同子集。

- **PyBoard**：标准 PyBoard 只有一个卡槽，不需要也不支持任何参数。
- **ESP32**：SD卡支持以 SD/MMC 模式和更简单（但速度较慢）的 SPI 模式进行访问。SPI 模式会使用 SPI 主机外设，该外设不能同时用于其他 SPI 交互。`slot` 参数决定使用哪种模式。不同芯片支持不同的值：

  | 芯片     | Slot 0     | Slot 1     | Slot 2     | Slot 3     |
  |----------|------------|------------|------------|------------|
  | ESP32    | SD/MMC     | SPI (id=1) | SPI (id=0) | -          |
  | ESP32-C3 | SPI (id=0) | -          | -          | -          |
  | ESP32-C6 | SPI (id=0) | -          | -          | -          |
  | ESP32-S2 | SPI (id=1) | SPI (id=0) | -          | -          |
  | ESP32-S3 | SD/MMC     | SD/MMC     | SPI (id=1) | SPI (id=0) |

  不同的插槽支持不同的数据总线宽度（数据引脚数量）：

  | 插槽类型   | 支持的数据宽度 |
  |------------|----------------|
  | 0 (SD/MMC) | 1, 4, 8        |
  | 1 (SD/MMC) | 1, 4           |
  | 2 (SPI)    | 1              |
  | 3 (SPI)    | 1              |

  **注意**：大多数使用专用硬件提供 SD 卡槽的 ESP32 模块只连接了1个数据引脚，因此 `width` 的默认值为 1。
<br><br>

  **ESP32系列芯片的额外细节说明**

  根据所使用的ESP32系列芯片不同，具体细节会有所差异：

  - **原始ESP32**
      
    在SD/MMC模式（插槽1）下，原始ESP32的 SD/MMC 模式引脚分配是固定的。SPI 模式插槽（2和3）允许在构造函数中为引脚设置不同的值。

    默认引脚分配如下：

    | 信号 | Slot 1 | Slot 2 | Slot 3 | 是否可设置 |
    |------|--------|--------|--------|------------|
    | CLK  | 14     | -      | -      | 否         |
    | CMD  | 15     | -      | -      | 否         |
    | D0   | 2      | -      | -      | 否         |
    | D1   | 4      | -      | -      | 否         |
    | D2   | 12     | -      | -      | 否         |
    | D3   | 13     | -      | -      | 否         |
    | sck  | -      | 18     | 14     | 是         |
    | cs   | -      | 5      | 15     | 是         |
    | miso | -      | 19     | 12     | 是         |
    | mosi | -      | 23     | 13     | 是         |
    
    在这两种模式下，`cd` 和 `wp` 引脚都不是固定的，默认处于禁用状态，除非进行了设置。
<br><br>

  - **ESP32-S3**

    ESP32-S3芯片允许为 SD/MMC 和 SPI 模式访问设置不同的引脚值。

    如果未设置，默认引脚分配如下：
    
    | 信号 | Slot 0 | Slot 1 | Slot 2 | Slot 3 |
    |------|--------|--------|--------|--------|
    | CLK  | 14     | 14     | -      | -      |
    | CMD  | 15     | 15     | -      | -      |
    | D0   | 2      | 2      | -      | -      |
    | D1   | 4      | 4      | -      | -      |
    | D2   | 12     | 12     | -      | -      |
    | D3   | 13     | 13     | -      | -      |
    | D4   | 33*    | -      | -      | -      |
    | D5   | 34*    | -      | -      | -      |
    | D6   | 35*    | -      | -      | -      |
    | D7   | 36*    | -      | -      | -      |
    | sck  | 37*    | 14     | 37*    | 14     |
    | cs   | 34*    | 13     | 34*    | 13     |
    | miso | 37*    | 2      | 37*    | 2      |
    | mosi | 35*    | 15     | 35*    | 15     |

    **注意**
    
    - 插槽 0 和 1 不能同时使用。
    - 如果 ESP32-S3 开发板配置为 Octal SPI Flash 或 PSRAM，则表中带星号*的引脚必须从默认值更改。

    如果要在SD/MMC模式下访问卡，需将 `slot` 参数值设为0或1，并根据需要设置sck（用于CLK）、cmd和data参数来分配引脚。如果传递了data参数，则它应该是一个数据引脚或引脚编号的列表或元组，其长度应等于width参数。例如：

    ```python
    sd = SDCard(slot=0, width=4, sck=8, cmd=9, data=(10, 11, 12, 13))
    ```

    如果要在 SPI 模式下访问卡，需将 `slot` 参数值设为 2 或 3，并根据需要传递 `sck`、`cs`、`miso`、`mosi` 参数来分配引脚。

    在这两种模式下，`cd` 和 `wp` 引脚默认处于禁用状态，除非在构造函数中进行了设置。
<br><br>

  - **其他ESP32芯片**

    其他 ESP32 系列芯片没有硬件 SD/MMC 主机控制器，只能在 SPI 模式下访问 SD 卡。

    若要在 SPI 模式下访问卡，需将 `slot` 参数值设为2或3，并传递 `sck`、`cs`、`miso`、`mosi` 参数来分配引脚。

    **注意**：ESP32-C3 和 ESP32-C6 只有一个可用的 SPI 总线，因此唯一有效的 `slot` 参数值是 2。将此总线用于 SD 卡后，就无法再将其用于 `machine.SPI`。

- cc3200

  可以通过将元组作为 `pins` 参数传递来设置用于 SPI 访问的引脚。

  **注意**：当前 cc3200 的 SD 卡实现将这个类命名为 `machine.SD`，而不是 `machine.SDCard`。
<br><br>

- mimxrt

  mimxrt 的 SDCard 模块仅支持通过专用 SD/MMC 外设（USDHC），以4位模式、50MHz时钟频率进行访问。遗憾的是，MIMXRT1011 控制器不支持 USDHC 外设，因此该控制器没有 `machine.SDCard` 模块。

  由于只支持 4 位模式和 50MHz 时钟频率，接口得到了简化，构造函数签名如下：

  `class machine.SDCard(slot=1)`

  USDHC 外设使用的引脚必须在 `mpconfigboard.h` 中配置。mimxrt 支持的大多数控制器最多提供两个 USDHC 外设。因此，引脚配置使用宏 `MICROPY_USDHCx` 进行，其中 x 分别为 1 或 2。

  以下是USDHC1的示例配置：

  ```c
  #define MICROPY_USDHC1 \
  { \
      .cmd = { GPIO_SD_B0_02_USDHC1_CMD}, \
      .clk = { GPIO_SD_B0_03_USDHC1_CLK }, \
      .cd_b = { GPIO_SD_B0_06_USDHC1_CD_B },\
      .data0 = { GPIO_SD_B0_04_USDHC1_DATA0 },\
      .data1 = { GPIO_SD_B0_05_USDHC1_DATA1 },\
      .data2 = { GPIO_SD_B0_00_USDHC1_DATA2 },\
      .data3 = { GPIO_SD_B0_01_USDHC1_DATA3 },\
  }
  ```
  如果不使用卡检测引脚（cb_b引脚），则相应的条目必须填充以下虚拟值：
  ```c
  #define USDHC_DUMMY_PIN NULL , 0
  ```
  根据宏 `MICROPY_USDHC1` 和/或 `MICROPY_USDHC2` 的定义，`machine.SDCard` 模块支持一个或两个插槽。如果只提供了其中一个定义，调用 `machine.SDCard()` 或 `machine.SDCard(1)` 将返回使用相应 USDHC 外设的实例。当两个宏都被定义时，调用 `machine.SDCard(2)` 将返回使用 USDHC2 的实例。


### class USBDevice - USB设备驱动程序  

**注意**, `machine.USBDevice` 目前仅支持 esp32、rp2 和 samd。使用时需要硬件原生支持 USB，并非所有开发板都具备此功能。  

USBDevice 提供了一个底层 Python API，用于通过 Python 代码实现 USB 设备功能。  

**⚠️ 警告**, 该底层 API 要求使用者熟悉 USB 标准。`micropython-lib` 中有高层 USB 驱动模块，提供更简单的接口和更多内置功能。  

#### 术语说明  

- **运行时（Runtime）USB设备接口**：指 MicroPython 启动后通过此 Python API 定义的 USB 设备接口或驱动程序。  
- **内置（Built-in）USB设备接口**：指编译到 MicroPython 固件中的接口或驱动程序，始终可用。例如默认启用的 USB-CDC（串口），部分硬件支持内置 USB-MSC（大容量存储）。  

#### 生命周期管理  

管理运行时 USB 接口可能存在挑战，尤其是当通过同一 USB 设备的内置 USB-CDC 串口与 MicroPython 通信时：

- **软复位影响**：MicroPython 软复位会清除所有运行的 USB 接口，导致整个 USB 设备与主机断开连接。若固件包含内置 USB-CDC 串口，软复位后该串口会重新连接。  
  
  这意味着，当运行的 USB 接口处于活动状态时，针对 USB-CDC 串口的功能（如 `mpremote run`）会立即失败，因为软复位会断开串口连接。第二次尝试通常会成功，因为软复位后运行接口已清除。
  
- **开机自动配置**：若需每次启动时配置运行时 USB 设备，建议将配置代码写入设备 VFS 的 `boot.py` 文件。每次复位时，该文件会在 USB 子系统初始化前（及 `main.py` 之前）执行，确保运行时设备立即启动。

- **开发调试建议**：为方便开发或调试，可连接硬件串口 REPL 并完全禁用内置 USB-CDC 串口（目前仅 rp2 支持）。需在自定义固件中配置 `#define MICROPY_HW_USB_CDC (0)` 和 `#define MICROPY_HW_ENABLE_UART_REPL (1)`。  


#### 构造函数  

- class machine.`USBDevice`

  创建USBDevice对象。  

  **注意**：该对象为单例，每次调用构造函数均返回同一对象引用。  

#### 方法  

- USBDevice.`config`(desc_dev, desc_cfg, desc_strs=None, open_itf_cb=None, reset_cb=None, control_xfer_cb=None, xfer_cb=None)

  使用USB运行时设备状态和回调函数配置USBDevice单例对象：  
  - `desc_dev`：包含新 USB 设备描述符的类字节对象。  
  - `desc_cfg`：包含新 USB 配置描述符的类字节对象。  
  - `desc_strs`（可选）：存储USB字符串描述符值的对象（字符串或字节对象），可以是列表、字典或支持整数键索引的任意对象（索引为USB字符串描述符索引）。  
    
  字符串是可选功能，若设备和配置描述符未引用字符串或仅使用内置字符串，该参数可留空（默认）。  
    
  除索引 0 外，所有字符串值应为纯 ASCII。索引 0 为特殊"语言"描述符，格式需符合 USB 标准（字节对象）。若索引 0 返回 `None`，则使用默认"英语"语言描述符。  
    
  为了退回到给定索引提供内置字符串值，下标查找可以返回 `None`、`KeyError` 或 `IndexError`。  
  
  - **open_itf_cb**：当 USB 主机发送"设置配置"请求时（设备可用前的最后阶段），针对每个接口或接口关联描述符（IAD）调用一次该回调。
  
    回调参数为接口或 IAD 描述符的 memoryview 视图（包含所有关联描述符），该视图仅在回调函数返回前有效。
  - **reset_cb**：USB 主机执行总线复位时调用，无参数。所有进行中的传输将终止，主机可能重新枚举设备（调用描述符回调和 `open_itf_cb()`）。
    - **control_xfer_cb**：每次 USB 控制传输（设备端点0）时调用一次或多次，参数包括：  
    第一个参数扩展传输阶段
      - 1 代表 SETUP 阶段
      - 2 代表 DATA阶段
      - 3 代表 ACK 阶段

    第二个参数是用于读取此阶段 USB 控制请求数据的内存视图。memoryview 仅在回调函数返回之前有效。此内存视图中的数据在单个传输的三个阶段中的每一个阶段都是相同的。

    成功的传输包括按三个阶段的顺序调用此回调。一般来说，如果设备想对控制请求做出响应，那么最好等到 ACK 阶段确认主机控制器按预期完成了传输。  
  
    回调返回值：  
    - `False`：可暂停端点并拒绝传输。它不会进入后续阶段。  
    - `True`：继续传输至下一阶段。  
    - 在 SETUP 阶段，若请求的 `wLength` 字段非零，可返回缓冲区对象（OUT方向需可写，IN方向需包含数据）。  
  - **xfer_cb**：通过 `USBDevice.submit_xfer()` 提交的非控制传输完成时调用，参数包括：  
    1. 完成传输的端点号。  
    2. 结果（传输成功为 `True`，失败为 `False`）。  
    3. 成功传输的字节数（短传输时，结果为 `True` 但 `xferred_bytes` 小于缓冲区长度）。  

  **注意**：总线复位时（`USBDevice.reset()`），未完成的传输不会触发 `xfer_cb`。
<br><br>

- USBDevice.`active`(self, [value])

  返回运行时USB设备的当前激活状态（布尔值）。“激活”表示设备可与主机交互，不表示主机实际连接。
  
  如果传入 `value` 为真值，激活USB设备。

  如果传入 `value` 为假值，停用USB设备，此时主机无法检测到设备。

  如果需模拟设备断开并重新连接（如配置变更后），可依次调用 active(False) 和 active(True)。
<br><br>

- USBDevice.`builtin_driver`  

  该属性存储当前内置驱动配置，必须设置为 USBDevice 定义的内置常量之一，默认值为 `USBDevice.BUILTIN_NONE`。  

  设置此属性时，运行时设备必须处于非激活状态。必要时先调用 `active()` 停用，设置后重新激活。

  如果设置为非 `BUILTIN_NONE` 的值，`USBDevice.config()` 参数需遵循以下限制：  
  - `desc_cfg` 应以内置 USB 接口描述符数据开头，该数据可通过 `USBDevice.builtin_driver` 属性的 `desc_cfg` 字段访问。在内置配置描述符之后追加的描述符，应使用从 `USBDevice.builtin_driver` 属性中定义的 `itf_max`（最大接口号）、`str_max`（最大字符串索引）和 `ep_max`（最大端点号）开始的接口编号、字符串索引和端点编号。
  - 如果 `desc_cfg` 末尾追加了新接口，需更新内置配置描述符中的 `bNumInterfaces` 字段。  
  - `desc_strs` 应设置为 `None`，或为一个列表/字典，其中索引值小于 `USBDevice.builtin_driver.str_max` 的项需缺失或值为 `None`。这将为内置驱动保留这些字符串索引。若在这些索引中的任意位置放置不同的字符串，将覆盖内置驱动中的对应字符串。
<br><br>

- USBDevice.`remote_wakeup`(self)

  若设备处于挂起模式且主机启用了 `REMOTE_WAKEUP` 功能，唤醒主机。需在 USB 属性和主机端均启用此功能。成功唤醒返回 `True`。
<br><br>

- USBDevice.`submit_xfer`(self, ep, buffer)

  在端点 `ep` 上提交 USB 传输。`buffer` 必须为实现缓冲区接口的对象（IN端点需可读，OUT端点需可写）。

  **注意**：`ep` 不能为控制端点0，控制传输通过 `control_xfer_cb` 逐步处理。

  返回值：成功为 `True`，失败为 `False`（如设备未配置或端点已有传输队列）。

  传输完成后触发 `xfer_cb` 回调。

  若设备未激活，抛出 `OSError`（错误码 `MP_EINVAL`）。
<br><br>

- USBDevice.`stall`(self, ep, [stall])

  获取或设置设备端点的STALL状态。 
  - `ep`：端点号。  
  - `stall`（可选）：布尔值，设置 STALL 状态。  
  
  返回值：调用前的端点 STALL 状态。 

  端点设置为 STALL 后将保持该状态，直至再次调用此函数或主机自动清除。

  若设备未激活，抛出 `OSError`（错误码 `MP_EINVAL`）。  


#### 常量  
- USBDevice.`BUILTIN_NONE`  
- USBDevice.`BUILTIN_DEFAULT`  
- USBDevice.`BUILTIN_CDC`  
- USBDevice.`BUILTIN_MSC`  
- USBDevice.`BUILTIN_CDC_MSC`

  这些常量包含编译到MicroPython固件中的内置描述符数据。`BUILTIN_NONE` 和 `BUILTIN_DEFAULT` 始终存在，其他常量取决于固件构建配置和内置驱动。  

  **注意**：目前最多只有 `BUILTIN_CDC`、`BUILTIN_MSC`、`BUILTIN_CDC_MSC` 中的一个被定义，且与 `BUILTIN_DEFAULT` 为同一对象。这些常量用于运行时检测内置驱动（未来可能支持多选内置配置）。  

  每个常量对象包含以下只读字段：  
  - `itf_max`：内置配置描述符中使用的最高 bInterfaceNumber 值加1。  
  - `ep_max`：内置配置描述符中使用的最高 bEndpointAddress 值加1（不含IN标志位`0x80）。  
  - `str_max`：内置描述符使用的最高字符串描述符索引加1。  
  - `desc_dev`：内置 USB 设备描述符的字节对象。  
  - `desc_cfg`：完整内置 USB 配置描述符的字节对象。