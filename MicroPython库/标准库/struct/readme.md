# struct（打包和解包基本数据类型）

该模块实现了 CPython 相应模块的子集。

支持以下字节序：

| 字符 | 字节序 | 大小 | 对齐 |
| -- | -- |  -- | -- |
| @ | 本地 | 本地 | 本地 |
| < | 小端  |标准 | 无 |
| > | 大端 | 标准 | 无 |
| ! | 网络（大端） | 标准 | 无 |

支持下面的数据类型:

| 格式 | C 类型 | Python 类型 | 标准大小 |
| -- | -- | -- | -- |
| b | signed char | integer | 1 |
| B | unsigned char | integer | 1 |
| h | short | integer | 2 |
| H | unsigned short | integer | 2 |
| i | int | integer | 4 |
| I | unsigned int | integer | 4 |
| l | long | integer | 4 |
| L | unsigned long | integer | 4 |
| q | long long | integer | 8 |
| Q | unsigned long long | integer | 8 |
| e | n/a (半精度浮点数) | float | 2 |
| f | float | float | 4 |
| d | double | float | 8 |
| s | char[] | bytes | |
| P | void * | integer | |

与 CPython 的差异

格式字符串中不支持空白字符。

##  函数

struct 相关函数。

- struct.`calcsize`(fmt)

  返回存储指定格式字符串 fmt 所需的字节数。
<br><br>

- struct.`pack`(fmt, v1, v2, ...)

  根据格式字符串 `fmt` 对值 `v1`, `v2`, ... 进行打包。返回值是一个编码了这些值的字节对象。
<br><br>

- struct.`pack_into`(fmt, buffer, offset, v1, v2, ...)

  根据格式字符串 `fmt`，将值 `v1`, `v2`, ... 打包到 `buffer` 的 `offset` 位置中。`offset` 可以为负值，表示从 `buffer` 的末尾开始计数。
<br><br>

- struct.`unpack`(fmt, data)

  根据格式字符串 `fmt` 从 `data` 中解包数据。返回值是一个包含解包后值的元组。
<br><br>

- struct.`unpack_from`(fmt, data, offset=0, / )

  根据格式字符串 `fmt`，从 `data` 的 `offset` 位置开始解包数据。`offset` 可以为负值，表示从 `data` 的末尾开始计数。返回值是一个包含解包后值的元组。


##  使用方法

struct 的基本使用方法是：

```py
>>> import struct
>>> struct.calcsize('hhl')
8
>>> struct.pack('hhl',1,2,3)
b'\x01\x00\x02\x00\x03\x00\x00\x00'
>>> struct.pack('llh0l', 1, 2, 3)
b'\x01\x00\x00\x00\x02\x00\x00\x00\x03\x00'
>>> struct.unpack('hhl',b'\x01\x00\x02\x00\x03\x00\x00\x00')
(1, 2, 3)
```



