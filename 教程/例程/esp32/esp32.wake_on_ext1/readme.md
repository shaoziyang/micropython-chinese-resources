# ESP32: wake_on_ext1

通过 ext1 从 deepsleep 中唤醒（注意只有部分 esp32 型号支持 ext1 功能）。  
测试开发板：nodemcu-32

```py
import esp32
from machine import deepsleep, Pin
from time import sleep_ms

pBTN = Pin(0, Pin.IN)

esp32.wake_on_ext1([pBTN],  esp32.WAKEUP_ALL_LOW)
sleep_ms(200)

print('enter deepsleep, use button wakeup')
deepsleep()
```
