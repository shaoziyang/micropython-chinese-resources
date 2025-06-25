# 通过 PA0 唤醒 pyboard

在低功耗应用中，为了降低功耗，我们需要让单片机休眠，然后通过外部按键或者RTC唤醒。但是直到v1.11版中，micropython中仍没有提供直接休眠后通过PA0唤醒功能。不过我们可以通过寄存器方法去设置，使用并不复杂。

通过PA0唤醒，也就是WKUP pin功能，需要将PWR_CSR寄存器的EWUP设置为1，就可以通过PA0引脚上的上升沿信号将pyb唤醒。

方法如下：

```py
import pyb, machine

if machine.reset_cause() == machine.DEEPSLEEP_RESET:
    led = pyb.LED(1)
    print('deep sleep')
else:
    led = pyb.LED(2)
    print('start')

for i in range(8):
    led.toggle()
    pyb.delay(100)

import stm

print(hex(stm.mem32[stm.PWR + stm.PWR_CSR]))

stm.mem32[stm.PWR + stm.PWR_CSR] |= 0x100
```

注：
- 对于STM32L4系列，支持的WKUP pin更多，还支持上升沿或下降沿唤醒，唤醒使用的寄存器也不同，但是方法类似。
- 因为machine.deepsleep()后唤醒相当于复位，所以需要先通过machine.reset_cause()判断复位原因，然后进行不同处理。
