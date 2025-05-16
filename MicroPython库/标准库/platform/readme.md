# platform（硬件平台标识）

platform 模块提供了与硬件平台相关的 API，可以用于识别特定硬件环境、编译器版本等。

## 函数

- platform.`platform`()

  返回一个标识基础平台的字符串。此字符串由以下几个子字符串组成，用减号（-）分隔：

  - 平台系统的名称（例如 Unix、Windows 或 MicroPython）
  - MicroPython 版本
  - 平台的架构
  - 底层平台的版本
  - MicroPython 链接到的 libc 的名称及其相应版本的串联。

  例如，在 esp32-c3 上显示为： `MicroPython-1.22.0-preview-riscv-IDFv5.0.2-with-newlib4.1.0`。 在 PYB V10 上显示为： `MicroPython-1.22.0-preview-arm-HAL1.16.0-with-newlib4.3.0`。
<br><br>

- platform.`python_compiler`()

  返回编译 MicroPython 的编译器版本，如：'GCC 12.3.1'。
<br><br>

- platform.`libc_ver`()

  显示 MicroPython 链接到的 libc 名称和版本，如：('newlib', '4.3.0')。
