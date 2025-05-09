# marshal（Python 对象序列化）

marshal 模块实现 Python 对象和二进制格式之间的转换。该格式专属于 MicroPython，但不受机器架构的限制，因此数据可以在不同的 MicroPython 实例之间传输和使用，只要二进制数据的版本匹配（目前版本与 mpy 文件版本一致，请参阅 MicroPython .mpy 文件）。

## 函数

- marshal.`dumps`(value, /)

  将给定 `value` 转换为二进制格式并返回相应的字节对象。

  目前，代码对象是唯一可以转换的受支持值。
<br><br>

- marshal.`loads`(data, /)

  将给定的字节类型数据转换为相应的 Python 对象，并返回它。
  
