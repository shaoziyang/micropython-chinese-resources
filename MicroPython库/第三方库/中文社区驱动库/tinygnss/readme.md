# tinyGNSS

极简的GNSS驱动，只解析 RMC 消息，获取基本的定位信息和UTC时间等。

使用时注意串口需要有足够大的缓冲区，避免数据丢失造成解析失败。

使用方法：

```python
from machine import Pin, UART
from time import sleep

gu = UART(2, 9600, tx=Pin(42), rx=Pin(41), rxbuf=512)
tg = tinyGNSS(gu)

while 1:
    sleep(1)
    tg.update(1)

```

**驱动地址**
- https://gitee.com/shaoziyang/mpy-lib/tree/master/gnss/tinyGNSS
