# UC8151/IL0373 电子纸显示驱动

UC8151/IL0373 MicroPython电子纸显示驱动程序，支持灰度和快速更新。与电子纸显示器的其他驱动程序相比，此驱动程序有点不同：

*  它对所有大于零的更新速度使用计算查找表（LUT）（对于速度0，使用内部OTP LUT）。通常，驱动程序使用其他驱动程序获得的固定 LUT 、应用程序注释或手工制作的表。计算 LUT 允许在质量和速度之间具有不同折衷的情况下提供更多的刷新模式（速度参数可以是浮点，如 2.5）。最重要的是，计算 LUT 是可以理解的，而不是魔法。这种方法还使用更少的内存，并使尝试不同的刷新策略变得更加容易。
*  防闪烁刷新模式（Anti-flickering）。如果选择此选项，LUT波形将针对特殊模式进行修改，在特殊模式下，在所有屏幕更新过程中，显示器将不会像电子墨水屏幕正常情况下那样闪烁。这是以不同级别的重影为代价的（重影的严重程度取决于速度）。我只是碰巧更讨厌 EPD 的闪烁，而不是延迟，一般来说，对于许多应用程序（想象一个时钟）来说，闪烁会影响显示，显示器不时地执行全局刷新，用新图像重新显示。
*  此驱动程序支持显示高达32级灰度的图像！，即使显示器本身是单色的。
*  驱动程序在注释中对芯片的操作细节进行了说明。因此，阅读它，您可以了解显示器是如何设置和使用的。
*  在这个驱动程序中快速模式仍然使用 100Hz，而不是 200Hz：它在测试中效果更好。
*  使用 +10V 高/低电压，公共电压也设置为默认值（-0.1V）。其他驱动器使用 11V 和/或不同的 DCOM 电压来提高对比度，并且可能会对硬件造成更大的压力。

除了上述技术变化外，该驱动程序的目标，特别是对于 MicroPython 用户和 Badger 2040 所有者来说，提供官方 Badger 软件的替代方案，以便使用最新的官方 Raspberry Pico MicroPython。

![](uc8151.webp)

```py
from machine import SPI, Pin
from uc8151 import UC8151

spi = SPI(0, baudrate=12000000, phase=0, polarity=0, sck=Pin(18), mosi=Pin(19), miso=Pin(16))
eink = UC8151(spi,cs=17,dc=20,rst=21,busy=26,speed=2)

# Then write something into the framebuffer and update the display.
eink.fb.text("Test",10,10,1)
eink.update()
```

https://github.com/antirez/uc8151_micropython
