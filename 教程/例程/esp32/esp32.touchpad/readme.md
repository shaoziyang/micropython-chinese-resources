# esp32: TouchPad

esp32 的触摸按键，注意只有部分 esp32 的部分 GPIO 支持 TouchPad 功能。

测试开发板：S2 mini。

```py
from time import sleep_ms
from machine import TouchPad, Pin

tk1 = TouchPad(Pin(12))
tk2 = TouchPad(Pin(13))

while 1:
    sleep_ms(500)
    print(tk1.read(), tk2.read())
```
