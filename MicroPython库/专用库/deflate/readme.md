# deflate（DEFLATE 压缩与解压缩）

该模块可利用DEFLATE算法（常见于zlib库和gzip归档工具）对二进制数据进行压缩和解压缩操作。

可用性：

- 在MicroPython v1.21版本中新增。
- 解压缩功能：通过 `MICROPY_PY_DEFLATE` 编译选项启用，在具备"额外功能"级别及以上（大多数开发板都属于此类）中默认开启。
- 压缩功能：通过 `MICROPY_PY_DEFLATE_COMPRESS` 编译选项启用，在具备"完整功能"级别及以上的版本中默认开启（一般而言，要启用此功能需自行编译固件）。

### class

- class deflate.`DeflateIO`(stream, format=AUTO, wbits=0, close=False, / )

  这个类能够对诸如文件、套接字或流（包括 `io.BytesIO`）等任何类似流的对象进行包装。其自身也属于流的一种，并实现了标准的 read/readinto/write/close 方法。

  该流必须为阻塞流，目前暂不支持非阻塞流。

  `format` 参数能够设置为下面定义的任意常量，默认值为 AUTO。在解压缩时，AUTO 会自动检测 gzip 或 zlib 流；在压缩时，则会生成原始流。

  `wbits` 参数用于设定 DEFLATE 字典窗口大小，它是 2为底的对数。例如，将 `wbits` 设为 10 时，窗口大小是 1024 字节。有效取值范围是 5 到 15（包含 5 和 15），对应的窗口大小为 32 到 32k 字节。

  如果 `wbits` 设为 0（即默认值），在压缩时会使用 256 字节的窗口大小（和 wbits 被设为 8 一样）。在解压缩时，窗口大小则取决于格式：
  - RAW 格式使用 256 字节（相当于 wbits 设为 8）。
  - ZLIB格式（或者 AUTO 自动检测为 zlib）会采用 zlib 头部中的值。
  - GZIP格式（或者 AUTO 自动检测为 gzip）会使用 32千字节（相当于 wbits 设为 15）。

  当 `close` 参数设为 `True` 时，在 `deflate.DeflateIO` 流关闭的同时，底层流也会自动关闭。要是你想返回一个包装了其他流的 `deflate.DeflateIO` 流，且不想让调用者操心底层流的管理，这个设置就很实用。

  若启用了压缩功能，给定的 `deflate.DeflateIO` 实例既能进行读取操作，也能执行写入操作。例如，可以对像套接字这样的双向流进行包装，从而实现双向的压缩和解压缩。

### 常量
- deflate.`AUTO`
- deflate.`RAW`
- deflate.`ZLIB`
- deflate.`GZIP`

  这些都是 format 参数的可用值。

### 示例

`deflate.DeflateIO` 的一个典型应用场景是对存储中的压缩文件进行读写操作：

```python
import deflate

# 写入一个zlib压缩流（使用默认的256字节窗口大小）
with open("data.gz", "wb") as f:
    with deflate.DeflateIO(f, deflate.ZLIB) as d:
        # 使用d.write(...)等方法

# 读取一个zlib压缩流（自动检测窗口大小）
with open("data.z", "rb") as f:
    with deflate.DeflateIO(f, deflate.ZLIB) as d:
        # 使用d.read()、d.readinto()等方法
```

由于 `deflate.DeflateIO` 属于流，所以它能和 `json.dump()`、`json.load()`（以及其他任何可以使用流的地方）配合使用：

```python
import deflate, json

# 以gzip格式将一个字典作为JSON写入，使用小窗口（64字节）
config = { ... }
with open("config.gz", "wb") as f:
    with deflate.DeflateIO(f, deflate.GZIP, 6) as f:
        json.dump(config, f)

# 读取回该字典
with open("config.gz", "rb") as f:
    with deflate.DeflateIO(f, deflate.GZIP, 6) as f:
        config = json.load(f)
```

如果源数据并非流格式，可借助 `io.BytesIO` 将其转换为适合 `deflate.DeflateIO` 使用的流：

```python
import deflate, io

# 解压缩一个bytes/bytearray值
compressed_data = get_data_z()
with deflate.DeflateIO(io.BytesIO(compressed_data), deflate.ZLIB) as d:
    decompressed_data = d.read()

# 压缩一个bytes/bytearray值
uncompressed_data = get_data()
stream = io.BytesIO()
with deflate.DeflateIO(stream, deflate.ZLIB) as d:
    d.write(uncompressed_data)
compressed_data = stream.getvalue()
```

### DEFLATE窗口大小

窗口大小对（解）压缩器在流中能够引用的回溯距离起到限制作用。增大窗口大小虽能提升压缩效果，但会占用更多内存，还会使压缩速度变慢。

如果输入流是用特定窗口大小压缩的，那么在解压缩过程中，若 `DeflateIO` 使用的窗口大小更小，当回溯引用的距离超过解压缩器的窗口大小时，会抛出 `OSError` 错误。不过，要是原始未压缩数据的长度小于窗口大小，使用较小的窗口大小也有可能完成解压缩。

**解压缩**

zlib 格式的头部包含了用于压缩数据的窗口大小信息，这表明解压缩该流所需的最大窗口大小。要是头部的值小于指定的 wbits 值（或者 wbits 未设置），就会采用头部的值。

gzip 格式的头部没有包含窗口大小信息，它假定所有 gzip 压缩器（如 gzip 或 CPython 的 gzip.GzipFile）都会使用 32kiB 的最大窗口大小。因此，若未设置 wbits 参数，解压缩器会使用 32kiB 的窗口大小（相当于 wbits 设为 15）。这意味着要解压缩任意 gzip 流，至少需要这么多的可用 RAM。要是你能控制源数据，可以考虑使用窗口大小更小的 zlib 格式。

原始格式没有头部，所以不包含任何关于窗口大小的信息。若未设置 wbits，就会默认使用 256 字节的窗口大小，这对于某些流来说可能不够大。所以，使用原始格式时，建议始终明确设置 wbits。

**压缩**

在压缩方面，MicroPython 对于所有格式默认都会使用 256 字节的窗口大小。这样能在保证一定压缩效果的同时，最大限度地减少内存使用并加快压缩速度，而且生成的输出能被任何解压缩器处理。
