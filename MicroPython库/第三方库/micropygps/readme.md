# micropyGPS

micropyGPS 是 micropython 的完整功能 GPS NMEA 语法解析器，它与Python 3.x完全兼容。

**特征：**

- 将大多数重要的NMEA-0183输出消息解析并验证为易于处理的数据结构
- 提供解释，显示，记录和操作GPS数据的辅助方法
- 使用Micropython中提供的标准库以纯Python 3.x编写
- 在单个文件中实现为单个类，以便轻松集成到嵌入式项目中
- 分析器在编写时考虑了串行UART数据源；一次处理单个字符，为嘈杂的嵌入式环境提供强大的错误处理能力
- 建立在TinyGPS Arduino库基础上

**支持的语法**

- GPRMC
- GLRMC
- GNRMC
- GPGLL
- GLGLL
- GNGLL
- GPGGA
- GLGGA
- GNGGA
- GPVTG
- GLVTG
- GNVTG
- GPGSA
- GLGSA
- GNGSA
- GPGSV
- GLGSV

注：
- 虽然这个库不支持北斗，但是很容易添加进去。
 
* * *
https://github.com/inmcm/micropyGPS