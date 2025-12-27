# ESP32: wake_on_touch

通过触摸唤醒 esp32。
开发板：DF FireBeetle ESP32

```py
from machine import Pin, TouchPad, deepsleep
from time import sleep_ms
import esp32

# 设置 LED，闪烁指示上电
led = Pin(2, Pin.OUT)
for i in range(8):
    led(not led())
    sleep_ms(100)

# 设置触摸键
tk = TouchPad(Pin(27, Pin.IN))
tk.read()

# 设置触摸门限
tk.config(800)
esp32.wake_on_touch(True)

deepsleep(10000)
```
