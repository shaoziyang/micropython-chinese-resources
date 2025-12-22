# Softuart 软串口

简单的软件出口，可以使用任意 GPIO 作为串口输出。暂时只支持串口发送，不支持串口接收。

```py
from machine import Pin
from time import sleep_us

su_txpin = Pin(26)
su_delay = 100

def su_init(txpin, baudrate=9600):
    global su_txpin, su_delay
    
    su_txpin = txpin
    su_txpin.init(Pin.OUT, value = 1)
    su_delay = int(1000000/baudrate - baudrate/1000)

def su_write(buf):

    for i in range(len(buf)):
        
        d = buf[i]

        su_txpin(0)
        sleep_us(su_delay)

        for j in range(8):
            su_txpin(d&1)
            sleep_us(su_delay)
            d >>= 1

        su_txpin(1)
        sleep_us(su_delay*3)

su_init(Pin(26))
su_write(b'1'*20)

```
