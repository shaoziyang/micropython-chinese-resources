# ESP-NOW

按下按钮发送消息，收到消息后闪灯。

- 注意根据开发板硬件调整 LED 和 SW 引脚。
- 使用广播方式收发数据。

```py
import network
import espnow
import time
from machine import Pin

# 根据开发板定义 LED 和 SW 引脚
if 'ESP32C6' in r.machine:
    LED= Pin(15, Pin.OUT)
    SW = Pin(9, Pin.IN)
else:
    LED= Pin(12, Pin.OUT)
    SW = Pin(9, Pin.IN)

# WLAN接口必须处于活动状态才能 send()/recv()
sta = network.WLAN(network.STA_IF) # 或 network.AP_IF
sta.active(True)
sta.disconnect() # 对于 ESP8266

e = espnow.ESPNow()
e.active(True)
peer = b'\xbb\xbb\xbb\xbb\xbb\xbb' # 配对设备wifi接口的MAC地址，这里设置为广播地址
e.add_peer(peer) # 发送前必须 add_peer()

# 按键中断
def isr_sw(t):
    e.send(peer, str(time.ticks_ms()))
    
def erecv():
    while True:
        host, msg = e.recv()
        if msg:
            LED(not LED())
            print(host, msg)

SW.irq(trigger=Pin.IRQ_FALLING, handler=isr_sw)

erecv()
```
