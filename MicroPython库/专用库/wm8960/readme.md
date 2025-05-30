# WM8960（WM8960编解码器驱动）

此驱动用于控制 WM8960 编解码器芯片。它是恩智浦/飞思卡尔为其 i.MX RT 系列 MCU 提供的 C 代码的 Python 版本。代码新增内容极少，仅修改或添加了几个与 API 相关的名称，以适配MicroPython的命名风格。

驱动的主要功能是初始化编解码器并设置其工作模式，**不负责处理编解码器的音频数据**（音频数据处理由独立的驱动完成）。

WM8960 除音频接口外还支持 I2C 接口，其连接方式取决于所使用的接口和系统中的设备数量。使用 I2C 接口时，需连接 SCL 和 SDA 引脚，当然还需连接 GND 和 VCC。I2C 默认地址为 0x1A。


### 构造函数  

- class `WM8960`(i2c, sample_rate, *, bits=16, swap=SWAP_NONE, route=ROUTE_PLAYBACK_RECORD, left_input=INPUT_MIC3, right_input=INPUT_MIC2, sysclk_source=SYSCLK_MCLK, mclk_freq=None, primary=False, adc_sync=SYNC_DAC, protocol=BUS_I2S, i2c_address=WM8960_I2C_ADDR)

  创建 WM8960 驱动对象，使用默认设置初始化设备并返回WM8960对象。
  
  **仅前两个参数为必填项**，其他参数均为可选。参数说明如下：

  - `i2c`：I2C 总线对象。
  - `sample_rate`：音频采样率，可选值为 8000、11025、12000、16000、22050、24000、32000、44100、48000、96000、192000、384000。注意：并非所有 I2S 硬件都支持所有采样率。
  - `bits`：音频字的位数，可选值为 16、20、24、32。
  - `swap`：若启用，交换左右声道；可选值见下文。
  - `route`：设置编解码器中的音频路径；可选值见下文。  
  - `left_input`：左输入通道的音频源；可选值见下文。  
  - `right_input`：右输入通道的音频源；可选值见下文。  
  - `play_source`：输出音频的目标；可选值见下文。  
  - `sysclk_source`：控制内部主时钟 "sysclk" 是直接取自 MCLK 输入，还是通过内部 PLL 从中衍生。通常无需修改此参数。  
  - `mclk_freq`：施加到编解码器MCLK引脚的时钟频率。若未设置，使用默认值。  
  - `primary`：设置 WM8960 为主设备或从设备，默认值为 `False` (从设备)。当设为 `False` 时，采样率和位数由 MCU 控制。  
  - `adc_sync`：设置 ADC 同步信号的输入源，默认使用 DACLRC 引脚。  
  - `protocol`：通信协议，默认值为I2S。可选值见下文。  
  - `i2c_address`：WM8960 的 I2C 地址，默认值为 0x1A。  

  若未设置 mclk_freq，默认值规则如下：  
  - 当 sysclk_source == SYSCLK_PLL 时：
    - 采样率为 44100、22050、11015 Hz 时，默认频率为 11.2896 MHz；  
    - 采样率低于 48000 Hz时，默认频率为 12.288 MHz；  
    - 其他情况，默认频率为采样率 × 256。  
  - 当 sysclk_source == SYSCLK_MCLK 时，默认频率为采样率 × 256。  

  注意：如果通过独立振荡器等方式提供 MCLK 信号，必须正确指定该频率以确保正常工作。

### 参数常量表

**交换参数**

数值 | 名称
:-:|:-
0 | SWAP_NONE（无交换）
1 | SWAP_INPUT（输入交换）
2 | SWAP_OUTPUT（输出交换）
<br>

**协议参数**

数值 | 名称
:-:|:-
2 | BUS_I2S（I2S 总线）
1 | BUS_LEFT_JUSTIFIED（左对齐总线）
0 | BUS_RIGHT_JUSTIFIED（右对齐总线）
3 | BUS_PCMA（PCMA 总线）
19| BUS_PCMB（PCMB 总线）
<br>

**输入源参数**

数值 | 名 | 类型
:-:|-|-
0 | INPUT_CLSED|（输入关闭）
1 | INPUT_MI1  | 单端
2 | INPUT_MC2  | 差分
3 | INPUT_MC3  | 差分
4 | INPUT_LINE2|（线路输入 2）
5 | INPUT_LINE3|（线路输入 3）
<br>

**路由参数**  

数值 | 名称  
:-:|:-
0 | ROUTE_BYPASS
1 | ROUTE_PLAYBACK
2 | ROUTE_PLAYBACK_RECORD  
5 | ROUTE_RECORD
<br>  

**主时钟源参数**

数值 | 名称
:-:|:-  
0 | SYSCLK_MCLK（系统时钟-主时钟）  
1 | SYSCLK_PLL（系统时钟-PLL锁相环）  
<br>

**模块名称**

数值 | 名称  
:-:|:-
0 | MODULE_ADC（模数转换器模块）  
1 | MODULE_DAC（数模转换器模块）  
2 | MODULE_VREF（基准电压模块）  
3 | MODULE_HEADPHONE（耳机模块）  
4 | MODULE_MIC_BIAS（麦克风偏压模块）  
5 | MODULE_MIC（麦克风模块）  
6 | MODULE_LINE_IN（线路输入模块）  
7 | MODULE_LINE_OUT（线路输出模块）  
8 | MODULE_SPEAKER（扬声器模块）  
9 | MODULE_OMIX（混音输出模块）  
10 | MODULE_MONO_OUT（单声道输出模块）  

**播放通道名称**

数值 | 名称  
:-:|:-
1 | PLAY_HEADPHONE_LEFT（耳机左声道播放）  
2 | PLAY_HEADPHONE_RIGHT（耳机右声道播放）  
4 | PLAY_SPEAKER_LEFT（扬声器左声道播放）  
8 | PLAY_SPEAKER_RIGHT（扬声器右声道播放）  

**adc_sync参数**

数值 | 名称
:-:|:-
0 | SYNC_ADC（模数转换器同步）  
1 | SYNC_DAC（数模转换器同步）

### 方法

除了初始化外，驱动程序还提供了一些用于控制其操作的实用方法：

- WM8960.`set_left_input`(input_source)

  指定左输入的源。输入源名称见上文。
<br><br>

- WM8960.`set_right_input`(input_source)

  指定右输入的源。输入源名称见上文。
<br><br>

- WM8960.`volume`(module, volume_l=None, volume_r=None)

  设置或获取某个模块的音量。

  如果未提供音量值，则返回实际音量元组。

  如果提供一个或两个值，则设置模块音量。若提供两个值，第一个用于左声道，第二个用于右声道；若仅提供一个值，则同时用于两个声道。声道数值范围在 0.0-100.0 之间（对数刻度）。  

  适用模块及 dB/步长见下表：

  **模块名称与dB步长**  
  | dB/步长 | 名称 |
  |---------|----------------|
  | 1.28    | MODULE_ADC     |
  | 1.28    | MODULE_DAC     |
  | 0.8     | MODULE_HEADPHONE |
  | 0.475   | MODULE_LINE_IN |
  | 0.8     | MODULE_SPEAKER |

- WM8960.`mute`(module, mute, soft=True, ramp=wm8960.MUTE_FAST)

  静音或取消静音输出。`mute` 为 `True` 时静音，为 `False` 时取消静音。

  如果 `soft` 设为 `True`，则静音将以软过渡方式进行。过渡时间由 `ramp` 定义，可选值为 `MUTE_FAST` 或 `MUTE_SLOW`。
<br><br>

- WM8960.`set_data_route`(route)

  设置音频数据路由。参数值/名称见上文表格。
<br><br>

- WM8960.`set_module`(module, active)

  启用或禁用模块，`active` 为 `False` 或 `True`。模块名称列表见上文表格。 

  注意：启用 `MODULE_MONO_OUT` 与 `WM8960.mono` 方法不同。前者启用输出3，而`WM8960.mono`方法将单声道混音发送到左右输出。
<br><br>

- WM8960.`enable_module`(module)

  启用模块。模块名称列表见上文表格。

- WM8960.`disable_module`(module)

  禁用模块。模块名称列表见上文表格。
<br><br>

- WM8960.`expand_3d`(level)

  启用立体声3D扩展。`level` 为 0-15 之间的数字，0 表示禁用扩展。

- WM8960.`mono`(active)

  如果 `active` 为 `True`，则将单声道混音发送到左右输出通道。这与启用 `MODULE_MONO_MIX` 不同，后者启用输出3。

- WM8960.`alc_mode`(channel, mode=ALC_MODE)

  启用或禁用自动电平控制(ALC)模式。参数：  
  - `channel`：启用并设置 ALC 通道。参数值：  
    - `ALC_OFF`：关闭 ALC  
    - `ALS_RIGHT`：使用右输入通道  
    - `ALC_LEFT`：使用左输入通道  
    - `ALC_STEREO`：使用两个输入通道  
  - `mode`：设置 ALC 模式。输入值：  
    - `ALC_MODE`：作为 ALC 工作  
    - `ALC_LIMITER`：作为限幅器工作  
<br><br>

- WM8960.`alc_gain`(target=-12, max_gain=30, min_gain=-17.25, noise_gate=-78)

  设置目标电平、最高和最低增益电平以及噪声门限（dB级别）。允许范围：  
  - `target`：-22.5 至 -1.5 dB  
  - `max_gain`：-12 至 30 dB  
  - `min_gain`：-17 至 25 dB  
  - `noise_gate`：-78 至 -30 dB  

  超出范围的值将被限制在允许范围内。`noise_gate` 为 -78 或更低时禁用噪声门功能。
<br><br>

- WM8960.`alc_time`(attack=24, decay=192, hold=0)

  设置ALC的动态特性（毫秒值）。允许范围：  
  - `attack`：6 至 6140  
  - `decay`：24 至 24580  
  - `hold`：0 至 43000  

  超出范围的值将被限制在允许范围内。
<br><br>

- WM8960.`deemphasis`(active)

  启用或禁用去加重滤波器，`active` 为 `False` 或 `True`。该滤波器仅适用于 32000、44100 和 48000 的采样率，其他采样率会静默忽略该设置。

- WM8960.deinit()

  关闭所有模块。


### 示例

在从模式（默认）下运行 WM8960：  
```python
# MicroPython WM8960编解码器驱动
# 使用默认设置将驱动设置为从模式
from machine import Pin, I2C
import wm8960

i2c = I2C(0)
wm = wm8960.WM8960(i2c, 32000, left_input=wm8960.INPUT_MIC1)
wm.set_volume(wm8960.MODULE_HEADPHONE, 100)
```

在主模式下运行 WM8960：  
```python
# MicroPython WM8960编解码器驱动
# 使用特定音频格式设置将驱动设置为主模式
from machine import Pin, I2C
import wm8960

i2c = I2C(0)
wm = wm8960.WM8960(i2c, 44100, primary=True, bits=16)
```

在 MIMXRT10xx_DEV 开发板上以从模式（默认）运行 WM8960：  
```python
# MicroPython WM8960编解码器驱动
# 使用默认设置将驱动设置为从模式
# 交换输入通道，使连接到右输入的MIMXRT开发板麦克风分配给左音频通道
from machine import Pin, I2C
import wm8960

i2c = I2C(0)
wm = wm8960.WM8960(
    i2c, 
    sample_rate=16_000,
    adc_sync=wm8960.SYNC_DAC,
    swap=wm8960.SWAP_INPUT,
    sysclk_source=wm8960.SYSCLK_MCLK
)
```

使用Sparkfun WM8960扩展板与Teensy在从模式（默认）下录音：
```python
# MicroPython WM8960编解码器驱动
# 扩展板使用固定的24MHz MCLK，因此必须使用内部PLL作为系统时钟（主音频时钟）
# Sparkfun板在板上连接了RX和TX的WS引脚，因此必须将adc_sync设置为sync_adc，将其ADCLRC引脚配置为输入
from machine import Pin, I2C
import wm8960

i2c = I2C(0)
wm = wm8960.WM8960(
    i2c, 
    sample_rate=16_000,
    adc_sync=wm8960.SYNC_ADC,
    sysclk_source=wm8960.SYSCLK_PLL,
    mclk_freq=24_000_000,
    left_input=wm8960.INPUT_MIC1,
    right_input=wm8960.INPUT_CLOSED
)
```

使用Sparkfun WM8960扩展板与Teensy在从模式（默认）下播放：  
```python
# 扩展板使用固定的24MHz MCLK，因此必须使用内部PLL作为系统时钟（主音频时钟）
# Sparkfun板在板上连接了RX和TX的WS引脚，因此必须将adc_sync设置为sync_adc，将其ADCLRC引脚配置为输入
from machine import I2C
import wm8960

i2c = I2C(0)
wm = wm8960.WM8960(
    i2c, 
    sample_rate=44_100,
    adc_sync=wm8960.SYNC_ADC,
    sysclk_source=wm8960.SYSCLK_PLL,
    mclk_freq=24_000_000
)
wm.set_volume(wm8960.MODULE_HEADPHONE, 100)
```
