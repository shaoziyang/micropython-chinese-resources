# ESP32: wake_on_gpio

通过 wake_on_gpio 方式从 deepsleep 中唤醒（注意只有部分 esp32 型号支持 wake_on_gpio 功能）。  
测试开发板：esp32c3 supermini

```py
import esp32
from machine import deepsleep, Pin
from time import sleep_ms

pBTN = Pin(0, Pin.IN)

esp32.wake_on_gpio([pBTN],  esp32.WAKEUP_ALL_LOW)
sleep_ms(2000)

print('enter deepsleep, use button wakeup')
deepsleep()

```
