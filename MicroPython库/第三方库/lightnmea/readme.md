# LightNMEA

速度极快、零依赖、单次 NMEA 0183 解析器，内存分配少，专为 MicroPython 和 Python（CPython）编写。为 32 位微控制器（ESP32、STM32、RP2040）设计，可与高速 GNSS 接收器（10-50Hz多星座GPS/GLONASS/北斗/伽利略模块）配合使用。

- 比micropyGPS快15倍（！）
- 支持七个星座（GPS、GLONASS、伽利略、北斗、QZSS、NavIC、多模GNSS）
- 解析循环中零内存分配
- 集成 MCU RTC（MicroPython）
- 非阻塞 UART 流读取

其他流行的解析器（pynmea2、pynmeagps）由于依赖于CPython特定的模块（数据类、类型）而无法在MicroPython上工作。

## 基准

为确保公平和全面的比较，基准脚本实现了以下条件：

- 真实世界有效负载：使用300000到1000000个NMEA 0183数据包（包括标准RMC和GGA句子）的代表性流执行测试。
- 预热阶段：在实际测量之前，每个解析器都会经历一个预热阶段（2次完整运行），以消除解释器初始化开销。
- 属性访问验证：基准测试不仅仅调用解析器函数；它执行严格的属性访问模拟（`_ = gps.valid` 和 `_ = gps.latitude`），以强制代码执行完整的数据解码和验证。
- 三次结果中最佳的：最终结果基于3次独立运行的最短运行时间，防止操作系统后台任务峰值影响数据。
- 垃圾回收：每次运行前都会调用显式`gc.collect()`，以确保精确的内存状态基线。

**基准结果**

| 平台/环境 | `light_nmea` | 竞争对手 | 对比 |
| --- | --- | --- | --- |
| CPython (Ryzen 7 2700) | 122,861 pps | `pynmea2`: 81,976 pps | 快1.5倍 |
| CPython (Ryzen 7 2700) | 124,991 pps | `pynmeagps`: 48,845 pps | 快2.6倍 |
| CPython (Ryzen 7 2700) | 125,000 pps | `adafruit_gps`: 76,277 pps | 快1.6倍 |
| CPython (Ryzen 7 2700) | 127,143 pps | `micropyGPS`: 35,322 pps | 快3.6倍 |
| CPython (Ryzen 7 2700) | 115,875 pps | `nmea_parser`: 59157 pps | 快2倍 |
| MicroPython (RP2040 @ 133MHz) | 375 pps | `micropyGPS`: 25 pps | 快12-15倍 |

注意：`pynmea2`、`pynmeagps`和`adafruit_gps`在MicroPython上不能运行，因为它们依赖于CPython模块（`dataclasses`、扩展`typing`、`circuitpython_type`）。`micropyGPS`是MicroPython的唯一替代NMEA解析器，但自2018年以来一直没有更新，而且速度明显较慢。在CPython上，差距较小（3.6倍），但在RP2040微控制器上，由于对象创建的开销，差距达到了15倍。

## 使用方法

基本用法

```python
from light_nmea import LightNMEA

# Initialize the parser
parser = LightNMEA(trust_gga_fix=True)

# Raw byte string from UART or log file
raw_sentence = b"$GNRMC,091530,A,5575.2057,N,03762.6130,E,15.0,120.5,220626,,,A*7F"

# Parse the string
if parser.parse_line(raw_sentence):
    if parser.valid:
        print("Latitude: ", parser.latitude)
        print("Longitude:", parser.longitude)
        print("Speed:    ", parser.speed, "km/h")
        print("Time:     ", parser.time.decode('ascii'))
```

数据流用法
```python
from light_nmea import LightNMEA

parser = LightNMEA(trust_gga_fix=True)

# Simulate a data stream from a log or UART
stream_packets = (
    b"$GNRMC,091530,A,5575.2057,N,03762.6130,E,15.0,120.5,220626,,,A*7F",
    b"$GNGGA,091530,5575.2057,N,03762.6130,E,1,12,1.0,156.3,M,0.0,M,,*42"
)

for raw_sentence in stream_packets:
    if parser.parse_line(raw_sentence):
        print("--- Packet processed successfully ---")
        print("Fix status: ", parser.valid)
        print("UTC Time:   ", parser.time.decode('ascii') if parser.time else "None")
        
        if parser.has_coordinates():
            print("Coordinates:", parser.latitude, ",", parser.longitude)
            
        if parser.has_navigation():
            print("2D Nav:     ", parser.speed, "km/h,", parser.course, "deg.")
            
        if parser.has_3d_fix():
            print("3D Fix:     ", parser.altitude, "meters")
            
        print("Satellites: ", parser.satellites)
```

## 相关链接

- [github 仓库](https://github.com/octaprog7/light-nmea-micropython)

