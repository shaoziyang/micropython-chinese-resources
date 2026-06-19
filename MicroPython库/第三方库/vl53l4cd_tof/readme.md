# VL53L4CD TOF 驱动和示例

适用于ST VL53L4CD TOF(飞行时间)激光距离传感器模块和BEAPER纳米电路的MicroPython驱动程序和示例程序。

- Sensors_Demo.py：使用BEAPER Nano LCD的演示程序，用于以类似雷达的方式显示参数，包括地传感器反射率、环境温度、电池电压、距离等，读取自 HC-SR04P SONAR距离传感器模块、VL53L0X ToF(飞行时用时)激光距离传感器模块或VL53L4CD ToF激光距离传感器模块。
- ToF_SONAR_Comparison.py：使用BEAPER Nano LCD的演示程序，将VL53L4CD ToF和HC-SR04P SONAR的距离与测量时间进行比较，并结合SONAR TRIG和ECHO信号的图形示波器视图进行比较。
- VL53L4CD.py：MicroPython VL53L4CD 驱动程序，由 ST 官方的 C ULD(超轻量级驱动程序)API v2.2.3 移植。
- VL53L4CD_test.py：一个简单的距离测量测试程序

**驱动程序功能**

- 完整传感器API：定时、偏移、交叉对话、信号和信号阈值、检测窗口、温度重新校准
- 非阻塞 `data_ready()/get_result()` 方法
- 简单(阻塞)：通过 `get_range()`
- 有意义的错误代码(而不是简单的 -1)
- 除了MicroPython标准库  (machine, time, micropython) 之外，没有任何依赖关系。

https://github.com/mirobotech/BEAPER-Nano/tree/main/MicroPython/VL53L4CD
