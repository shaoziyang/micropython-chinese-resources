# json（JSON 编码解码）

提供 Python 对象到 [JSON](https://docs.python.org/3.5/library/json.html)（JavaScript Object Notation） 数据格式的转换。

## 函数

* json.`dump`(obj, stream, separators=None)

  将 `obj` 序列化为 JSON 字符串，并将其写入给定的流。如果指定了分隔符，则分隔符应为 (item_separator，key_separator)元组。默认为 (', ', ': ')。要获得最紧凑的JSON表示，可以指定 (',', ':') 以消除空格。
<br><br>

* json.`dumps`(obj, separators=None)

  返回表示为 JSON 字符串的 `obj`。
<br><br>

* json.`load`(stream)

  解析给定的stream，将其解释为JSON字符串，作为Python对象返回。分析将持续到文件末尾。如果流中的数据不正确，将引发 `ValueError` 异常。
<br><br>

* json.`loads`(str)

  解析 JSON 字符串并返回对象。如果字符串格式错误将引发 `ValueError` 异常。
