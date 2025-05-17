# zlib（压缩和解压缩）

zlib 模块允许使用 `DEFLATE` 算法（该算法普遍应用于 zlib 库和 gzip 压缩程序）对二进制数据进行压缩和解压缩。

建议优先使用 `deflate.DeflateIO`，而非本模块中的函数。因为前者提供了流式压缩和解压缩接口，在向文件、套接字或流中读写压缩数据时，该接口不仅简单方便，而且更节省内存。

- 从 MicroPython v1.21 开始，默认情况下固件中将不在提供 zlib 模块，因为它与 `deflate` 模块的功能存在重复。
- 可以通过 `micropython-lib` 安装（冻结）zlib 模块。
- 依赖于内置的 `deflate` 模块（从 MicroPython v1.21 开始）。
- 只有在内置 `deflate` 模块中启用压缩功能时，才能使用压缩功能（默认是关闭的）。


## 函数

- zlib.`decompress`(data, wbits=15, / )

  将数据 data 解压缩为字节对象。

  wbits 参数的工作方式与 `zlib.compress()` 相同，具有以下额外的有效值：
  - 0：根据 zlib 标头自动确定窗口大小（`data` 必须为 zlib 格式）。
  - 35 到 47：自动检测 zlib 或 gzip 格式。

  与 `zlib.compress()` 一样，有关 wbits 参数的更多信息，请参阅 [CPython 中 zlib 的文档](https://docs.python.org/3.5/library/zlib.html#module-zlib)。与 `zlib.compress()` 类似，MicroPython 也支持比 CPython 更小的窗口大小。有关 MicroPython 特定的详细信息，请参阅 `deflate` 模块的文档。

  如果待解压缩的数据需要更大的窗口大小，解压缩时会失败。
<br><br>

- zlib.`compress`(data, wbits=15, / )

  将数据压缩到字节对象中。

  wbits 允许重新配置 `DEFLATE` 字典窗口的大小和输出格式。设置窗口大小允许在内存使用情况和压缩级别之间进行权衡，较大的窗口大小将允许压缩器在输入中引用更靠前的片段。输出格式为"原始" DEFLATE（无标头/页脚）、zlib 和 gzip，其中后两者包括标头和校验和。

  wbits 绝对值的低四位设置 DEFLATE 字典窗口大小，它是以 2 为底的对数。例如，wbits=10、wbits=-10 和 wbits=26 都将窗口大小设置为 1024 字节。有效的窗口大小范围为 5 到 15（包括 5 到 15），对应于 32 到 32k 字节。

  在-5 和-15 之间的 wbits 负数值对应于"原始"输出模式，在 5 和 15 之间的正数值对应于 zlib 输出模式，而在 21 和 31 之间的正数值则对应于 gzip 输出模式。

  有关 wbits 参数的更多信息，请参阅 [zlib 的 CPython 文档](https://docs.python.org/3.5/library/zlib.html#module-zlib)。注意 MicroPython允许更小的窗口大小，这在内存受到限制时非常有用，同时仍能实现合理的压缩级别。它还加快了压缩的速度。

