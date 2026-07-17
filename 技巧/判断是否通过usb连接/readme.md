# 判断是否通过 USB 连接

micropython 开发板或设备是通过 USB 或者串口连接的，有时我们需要在程序中判断当前是通过 USB 还是串口进行连接的，但 micropython 中并没有提供一个接口用于 USB 连接的判断，因此需要通过一些方法进行识别。

**STM32**

对于 stm32，判断 usb 连接比较简单，因为 micropython 中在 pyb 中提供了一个类 `pyb.USB_VCP`，它可以用来进行识别。

```python
import pyb

def stm32_usb_isconnect():
    usb = pyb.USB_VCP()
    return usb.isconnected()
```

**ESP32**

对于 esp32 系列芯片，不是全部都带有 usb 接口的，而且不同型号差异很大。

对于 esp32s2 和 esp32s3，可以用如下方法进行识别：
```python
from machine import Pin

def esp32sx_usb_isconnect():
    dp = Pin(20, Pin.IN, Pin.PULL_UP)
    dn = Pin(19, Pin.IN, Pin.PULL_UP)
    dp1 = dp()
    dn1 = dn()
    dp = Pin(20, Pin.IN, Pin.PULL_DOWN)
    dn = Pin(19, Pin.IN, Pin.PULL_DOWN)
    dp2 = dp()
    dn2 = dn()
    return dp1 and dp2 and not dn1 and not dn2 
```


