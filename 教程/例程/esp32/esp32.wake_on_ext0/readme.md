# ESP32: wake_on_ext0

通过 ext0 从 deepsleep 中唤醒。  
测试开发板：nodemcu-32

```py
import esp32
from machine import deepsleep, Pin
from time import sleep_ms

pBTN = Pin(0, Pin.IN)

esp32.wake_on_ext0(pBTN,  esp32.WAKEUP_ALL_LOW)
sleep_ms(2000)

print('enter deepsleep, use button wakeup')
deepsleep()
```
