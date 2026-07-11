# RP2: 内部温度传感器

RP2040/RP2350 内部带有一个温度传感器，它使用了 ADC 的通道4，只要先读取通道4 的ADC电压，然后通过下面公式进行转换，就可以得到芯片的内部温度（摄氏度）。

$ T = 27 - \frac{(ADC\_voltage - 0.706)}{0.001721} $


```python
from machine import ADC
from time import sleep

sensor_temp = ADC(4)

def get_core_temperature(N=8):
    vt=0
    for i in range(N):
        vt += sensor_temp.read_u16()*3.3/65535
    return 27 - (vt/N - 0.706) / 0.001721

while True:
    print(f"RP2 core temperature: {get_core_temperature(32):.1f}°C")
    sleep(1)
```
