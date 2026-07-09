# ESPNOW: RSSI

定时发送数据，以中断方式接收数据，同时获取信号强度（RSSI）。

```python
import network
import espnow
import time
from machine import Pin, Signal
import random

LED = Signal(Pin(8, Pin.OUT), invert=1)

sta = network.WLAN(network.STA_IF)
sta.active(True)

e = espnow.ESPNow()
e.active(True)

peer = b'\xFF'*6
e.add_peer(peer)

queue = []

def on_recv(e):
    while True:
        mac, msg = e.recv(0)
        if mac is None:
            return
        d = e.peers_table
        rssi = d.get(mac)[0]
        queue.append((mac, msg, rssi))
        if len(queue)>20:
            queue.pop(0)

e.irq(on_recv)

def erecv():
    T0 = time.ticks_ms()
    print('start', peer, T0)
    while True:

        while len(queue) > 0:
            LED(1)
            d = queue.pop(0)
            print(f"> {d[0]}: {d[1]}, RSSI: {d[2]} dBm")
            LED(0)

        T1 = time.ticks_ms()
        if time.ticks_diff(T1, T0) > 1500:
            T0 = T1
            print('<', str(T1))
            e.send(peer, str(T1))
            e.send(peer, str(T1))
            e.send(peer, str(T1))
            time.sleep_ms(20)

        time.sleep_ms(10)

erecv()

```