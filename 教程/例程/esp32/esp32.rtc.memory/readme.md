# esp32: rtc.memory

esp32 和 esp8266 包含了一个 RTC memory 存储空间，这个空间的数据在复位时保持不变（包括软复位和深度休眠），因此可以用来保存一些临时数据。

```py
from machine import RTC, deepsleep

rtc = RTC()
print(rtc.memory())

rtc.memory('12345')

deepsleep(5000)
```

注：
- RTC memory 的大小，在 esp32 上是 2048 字节，在 esp8266 上是 492 字节。 
- 在 micropython 中，rtc.memory 是一个 buffer protocol 类型的对象，如 bytes、bytearray、memoryview、array.array。
