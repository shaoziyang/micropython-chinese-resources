# framebuf（帧缓冲区操作）

framebuf 模块提供了一个通用的帧缓冲区，可用于创建位图图像，然后将其发送到显示器。

### class FrameBuffer

FrameBuffer 类提供了一个像素缓冲区，可在上面绘制像素、线条、矩形、椭圆、多边形、文本，甚至其它 FrameBuffer 对象。在为显示器生成输出时，这个类非常有用。

例如：

```python
import framebuf

# 对于RGB565格式，每个像素 FrameBuffer 需要2个字节
fbuf = framebuf.FrameBuffer(bytearray(100 * 10 * 2), 100, 10, framebuf.RGB565)
fbuf.fill(0)
fbuf.text('MicroPython!', 0, 0, 0xffff)
fbuf.hline(0, 9, 96, 0xffff)
```

### 构造函数

- class framebuf.`FrameBuffer`(buffer, width, height, format, stride=width, /)

  构造一个 FrameBuffer 对象。参数如下：
  - `buffer`：一个实现了缓冲区协议的对象，其大小必须足够容纳由 FrameBuffer 的宽度、高度和格式定义的每个像素。
  - `width`：FrameBuffer 的宽度（以像素为单位）
  - `height`：FrameBuffer 的高度（以像素为单位）
  - `format`：指定FrameBuffer中使用的像素类型；允许的值在下面的常量部分列出。这些值设置了用于编码颜色值的位数，以及这些位在缓冲区中的布局。当一个颜色值`c`被传递给某个方法时，`c`是一个小整数，其编码方式取决于FrameBuffer的格式。
  - `stride`：FrameBuffer中每行像素之间的像素数。默认值为`width`，但在另一个更大的FrameBuffer或屏幕中实现FrameBuffer时，可能需要进行调整。缓冲区大小必须适应增加的步长。

  必须指定有效的缓冲区、宽度、高度、格式，以及可选的步长。无效的缓冲区大小或维度可能会导致意外错误。

### 绘制基本形状

以下方法可在FrameBuffer上绘制各种形状：

- FrameBuffer.`fill`( c )

  用指定颜色填充整个 FrameBuffer。
<br><br>

- FrameBuffer.`pixel`(x, y [, c])

  如果未提供`c`，则获取指定像素的颜色值。如果提供了`c`，则将指定像素设置为给定颜色。
<br><br>

- FrameBuffer.`hline`(x, y, w, c)
- FrameBuffer.`vline`(x, y, h, c)
- FrameBuffer.`line`(x1, y1, x2, y2, c)

  使用给定颜色绘制一条从一组坐标开始的线，线宽为1像素。`line` 方法绘制直线到第二组坐标，而 `hline` 和 `vline` 方法分别绘制指定长度的水平线和垂直线。
<br><br>

- FrameBuffer.`rect`(x, y, w, h, c [, f])

  在指定位置绘制一个具有指定大小和颜色的矩形。

  可选参数 `f` 可设置为 `True` 以填充矩形。否则只绘制一个1像素的轮廓。
<br><br>

- FrameBuffer.`ellipse`(x, y, xr, yr, c [, f, m])

  在指定位置绘制一个椭圆。半径 `xr` 和 `yr` 定义了椭圆的形状；如果值相等，则绘制一个圆。参数 `c` 定义颜色。

  可选参数 `f` 设置为 `True` 将填充椭圆。否则只绘制一个1像素的轮廓。

  可选参数 `m` 允许将绘制限制在椭圆的某些象限。低四位决定绘制哪些象限，位0指定第一象限，位1指定第二象限，位2指定第三象限，位3指定第四象限。象限按逆时针编号，第一象限为右上角。
<br><br>

- FrameBuffer.`poly`(x, y, coords, c [, f])

  给定一个坐标列表，使用给定颜色在指定的x、y位置绘制一个任意（凸或凹）的闭合多边形。

  `coords` 必须指定为一个整数数组，例如 `array('h', [x0, y0, x1, y1, ... xn, yn])`。

  可选参数 `f` 设置为 `True` 将填充多边形。否则只绘制一个1像素的轮廓。
<br><br>

### 绘制文本

- FrameBuffer.`text`(s, x, y [, c])

  使用坐标作为文本的左上角，将文本写入 FrameBuffer。文本的颜色可以由可选参数定义，否则默认为1。所有字符的尺寸为8x8像素，目前无法更改字体。

### 其他方法

- FrameBuffer.`scroll`(xstep, ystep)

  将 FrameBuffer 的内容按给定方向移动。这可能会在 FrameBuffer 中留下先前颜色的痕迹。
<br><br>

- FrameBuffer.`blit`(fbuf, x, y, key=-1, palette=None)

  在当前 FrameBuffer 的指定坐标上绘制另一个 FrameBuffer。如果指定了 `key`，则它应该是一个颜色整数，相应的颜色将被视为透明：所有具有该颜色值的像素都不会被绘制。（如果指定了 `palette`，则将 `key` 与 `palette` 中的值进行比较，而不是直接与 `fbuf` 中的值进行比较。）

  `palette` 参数允许在不同格式的 FrameBuffer 之间进行位块传输。典型用法是将单色或灰度字形/图标渲染到彩色显示器上。`palette` 是一个 FrameBuffer 实例，其格式与当前 FrameBuffer 的格式相同。`palette` 的高度为1像素，其像素宽度是源 FrameBuffer 中的颜色数量。N位 `palette` 的需要 $ 2^N $ 个像素；单色源的 `palette` 将有 2 个像素，分别代表背景色和前景色。应用程序为 `palette` 中的每个像素分配一种颜色。当前像素的颜色将是 `palette` 中 x 位置与相应源像素颜色对应的像素的颜色。
<br><br>

### 常量

- framebuf.`MONO_VLSB`

  单色（1位）颜色格式。此格式定义了一种映射方式，其中字节中的位按垂直方式映射，位0最靠近屏幕顶部。因此，每个字节占用8个垂直像素。后续字节出现在连续的水平位置，直到到达最右边的边缘。更多的字节从最左边的边缘开始，在低8个像素的位置渲染。
<br><br>

- framebuf.`MONO_HLSB`

  单色（1位）颜色格式。此格式定义了一种映射方式，其中字节中的位按水平方式映射。每个字节占用8个水平像素，位7在最左边。后续字节出现在连续的水平位置，直到到达最右边的边缘。更多的字节在下一行，低1个像素的位置渲染。
<br><br>

- framebuf.`MONO_HMSB`

  单色（1位）颜色格式。此格式定义了一种映射方式，其中字节中的位按水平方式映射。每个字节占用8个水平像素，位0在最左边。后续字节出现在连续的水平位置，直到到达最右边的边缘。更多的字节在下一行，低1个像素的位置渲染。
<br><br>

- framebuf.`RGB565`

  红绿蓝（16位，5+6+5）颜色格式。
<br><br>

- framebuf.`GS2_HMSB`

  灰度（2位）颜色格式。
<br><br>

- framebuf.`GS4_HMSB`

  灰度（4位）颜色格式。
<br><br>

- framebuf.`GS8`

  灰度（8位）颜色格式
