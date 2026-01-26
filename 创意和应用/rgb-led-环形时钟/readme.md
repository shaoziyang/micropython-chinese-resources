# RGB LED 环形时钟

该项目使用树莓派 Pi Pico 和 Waveshare RTC（实时时钟）模块，将 12 RGB LED 环形变为工作时钟。LED 灯会点亮不同颜色，显示时针和分针：

* 时针是蓝色 LED 灯。
* 分针随着时间从红色变绿而变化。
* 如果时针和分针占用同一空间，那么 LED 再次循环显示一系列颜色

设计中还包括一个夏令时开关、USB-C 电源输入，以及 Waveshare RTC 模块内置的纽扣电池，即使拔掉电源，时钟也能保持时间。

![](https://cdn-blog.adafruit.com/uploads/2026/01/20250119clock.gif)

**相关链接：**
* [项目说明](https://www.instructables.com/RGB-LED-Ring-Clock/)
* [github代码](https://github.com/TellinStories/RGB-LED-Ring-Clock-Pico)
