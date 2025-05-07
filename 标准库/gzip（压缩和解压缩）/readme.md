# gzip（压缩和解压缩）

`gzip` 模块实现了 CPython 中 gzip 模块的子集，允许使用 gzip 文件格式所采用的 [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) 算法对二进制数据进行压缩和解压缩。

## 可用性

- 在官方的 MicroPython 固件发行版本中默认不存在 gzip 模块，因为它与 `deflate` 模块的功能重复。
- 可以从 `micropython-lib`（[源代码](https://github.com/micropython/micropython-lib/blob/master/python-stdlib/gzip/gzip.py)）安装（或冻结）该模块的副本。
- 只有在内置的 `deflate` 模块中启用了压缩功能时，才会提供压缩功能支持。

## 函数

- gzip.`open`(filename, mode, /)

  这是对内置 open() 函数的封装，调用后会返回一个 GzipFile 实例。
<br><br>

- gzip.`decompress`(data, /)

  将数据解压缩到字节对象。
<br><br>

- gzip.`compress`(data, /)
  
  压缩数据到字节对象。
<br><br>

## class

- class gzip.`GzipFile`(*, fileobj, mode)

  这个类可用于包装一个文件对象，该文件对象可以是任何类似流的对象，比如文件、套接字或流（包括 `io.BytesIO`）。它自身就是一个流，并实现了标准的 `read`（读取）、`readinto`（读取到）、`write`（写入）和 `close`（关闭）方法。
<br><br>
  当 `mode` 参数为 "rb" 时，从 `GzipFile` 实例中读取数据会对底层流中的数据进行解压缩，并返回解压缩后的数据。
<br><br>
  如果启用了压缩支持，那么 `mode` 参数可以设置为 "wb"，向 `GzipFile` 实例写入的数据将会被压缩并写入到底层流中。
<br><br>
  默认情况下，`GzipFile` 类将使用 `gzip` 文件格式来读取和写入数据，其中包括带有校验和的头部和尾部，以及大小为512字节的窗口。
<br><br>
  不支持 `file`（文件）、`compresslevel`（压缩级别）和 `mtime`（修改时间）参数。 必须始终指定 `fileobj`（文件对象）和 `mode`（模式）关键字参数。 

## 示例

`gzip.GzipFile` 的一个典型用例是从存储设备中读取或写入压缩文件。

```py
import gzip

# Reading:
with open("data.gz", "rb") as f:
    with gzip.GzipFile(fileobj=f, mode="rb") as g:
        # Use g.read(), g.readinto(), etc.

# Same, but using gzip.open:
with gzip.open("data.gz", "rb") as f:
    # Use f.read(), f.readinto(), etc.

# Writing:
with open("data.gz", "wb") as f:
    with gzip.GzipFile(fileobj=f, mode="wb") as g:
        # Use g.write(...) etc

# Same, but using gzip.open:
with gzip.open("data.gz", "wb") as f:
    # Use f.write(...) etc

# Write a dictionary as JSON in gzip format, with a
# small (64 byte) window size.
config = { ... }
with gzip.open("config.gz", "wb") as f:
    json.dump(config, f)
```