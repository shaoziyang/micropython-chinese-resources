# MEMS 压力传感器 LPS22

LPS22xx 系列是 ST 公司的高性能MEMS nano压力传感器，可用作数字输出气压计。器件由一个传感元件和一个IC接口组成，通过I²C、MIPI I3C^{SM}或SPI接口实现从传感元件到应用的通信。

目前驱动只支持 I²C 方式，先将[驱动文件](https://gitee.com/microbit/mpy-lib/tree/master/sensor/LPS22)复制到设备中，然后通过 I²C 就可以获取数据了。

```
from machine import I2C
import LPS22

i2c = I2C(1)
lps = LPS22.LPS22(i2c)
lps.get()
```

如果需要在 irq 中获取数据，就将上面的 `lps.get()` 改为 `lps.get_irq()`。

