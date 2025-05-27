# neopixel（WS2812/NeoPixel 控制）

该模块为 WS2818/NeoPixel LED 提供驱动。

**注意**：默认情况下，`neopixel` 模块仅包含在 ESP8266、ESP32 和 RP2 版本固件中。在 STM32/Pyboard 等其它硬件平台上，可通过 `mip` 安装 `neopixel` 软件包，或直接从 `micropython-lib` 下载模块并复制到文件系统中。

### class NeoPixel

此类存储连接到引脚的 WS2812 LED 灯带的像素数据。应用程序应先设置像素数据，然后需要更新时调用 `NeoPixel.write()`。

**示例：**
```python
import neopixel
# 连接到X8引脚的32灯LED灯带
p = machine.Pin.board.X8
n = neopixel.NeoPixel(p, 32)

# 绘制红色渐变
for i in range(32):
    n[i] = (i * 8, 0, 0)

# 更新灯带
n.write()
```

### 构造函数

- class neopixel.`NeoPixel`(pin, n, *, bpp=3, timing=1)

  创建 NeoPixel 对象，参数说明：
  - `pin`：machine.Pin 实例（引脚对象）。
  - `n`：灯带中的 LED 数量。
  - `bpp`：RGB LED 为 3，RGBW LED 为 4（颜色通道数）。
  - `timing`：0 表示 400KHz 速率，1 表示 800kHz 速率（大多数 LED 为 800kHz）。也可提供 machine.bitstream() 支持的时序元组。

### 像素访问方法

- NeoPixel.`fill`(pixel)

  将所有像素的值设置为指定的像素值（即 RGB/RGBW 元组）。
<br><br>

- NeoPixel.`__len__`()

  返回灯带中的 LED 数量。
<br><br>

- NeoPixel.`__setitem__`(index, val)

  将索引为 `index` 的像素设置为 `val` 值（RGB/RGBW 元组）。
<br><br>

- NeoPixel.`__getitem__`(index)

  返回索引为 `index` 的像素值（RGB/RGBW 元组）。
<br><br>

### 输出方法

- NeoPixel.`write()`

  将当前像素数据写入 LED。
