# esp32: temperature
获取 mcu 温度。注意在 esp32 上有两个不同函数，分别用于不同型号的 esp32。

```py
import esp32

esp32.mcu_temperature()   # 摄氏度 esp32c3, esp32c6, esp32s2, esp32s3
esp32.raw_temperature()   # 华氏度 esp32 
```

兼容两种芯片的函数，可以指定输出摄氏度还是华氏度：

```py
import esp32

def esp32_temperature(mode='C'):
    try:
        r = esp32.mcu_temperature()
    except:
        try:
            r = round((esp32.raw_temperature()-32)/1.8)
        except:
            r = 0
    if mode =='F' or mode == 'f':
        r = round(r*1.8+32)

    return r

esp32_temperature()
```