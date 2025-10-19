# MCP401x 系列数字电位器

[MCP4017/18/19](https://www.microchip.com/en-us/product/MCP4017) 是 microchip 公司的 7 位不带记忆功能的数字电位器，内部通过7级电阻网络构成128级步距，通过 $I^2C$ 接口进行控制。

MCP401x 的 $I^2C$ 接口支持标准模式（100kb/s）和高速模式（400kb/s）,设备的 7 位 $I^2C$ 地址固定为 0x2f（47），$I^2C$ 通信采用了简单的直接读写方式。

```
+------------------------------------------------+
| |     地址    |1/0| | |        数据        | | |
+------------------------------------------------+
|S|0|1|0|1|1|1|1|R/W|A|X|D6|D5|D4|D3|D2|D1|D0|A|P|
+------------------------------------------------+

S：起始位
P：停止位
R/W：读=1/写=0
D6-D0：数据
A：应答
X：无关位
```


## 使用方法

先将 [mcp401x 驱动](https://gitee.com/microbit/mpy-lib/tree/master/misc/MCP401x) 复制到开发板中：

```python
from machine import I2C, Pin
import mcp401x

i2c = I2C(sda = Pin(5), scl=Pin(4))
mcp = mcp401x.MCP401x(i2c)
mcp.write(50)

```


## proteus 模拟效果

![](../../../../开发软件/模拟运行/proteus/mcp401x.gif)
