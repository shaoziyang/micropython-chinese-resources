# 在 CircuitPython 中使用传感器

使用 CircuitPython 在串行上读取模拟和数字传感器数据
 
![](https://cms.krill.systems/commons/blog/a45f9e03-d42c-4724-86be-7ea4045733c1.png)
 
**步骤:**

- 将电路板连接到USB端口，然后将相应的固件文件放入其中 —— 它将重新启动并显示为驱动器。代码保存到文件code.py，它会自动运行。
- 下载绑定库，大多数项目都需要用到它。在绑定库中找到合适的文件，并将其复制到lib文件夹中。
- 使用像 Mu 这样的编辑器来编辑code.py。这是一个专为 CircuitPython 设计的简单编辑器，可轻松编写和上传代码到设备。
 
```py
import time
import board
from adafruit_onewire.bus import OneWireBus
from adafruit_ds18x20 import DS18X20

# Change D2 if you used a different pin for the yellow DATA wire
ow_bus = OneWireBus(board.D2)

devices = ow_bus.scan()
print("Devices found:", len(devices))

if not devices:
    raise RuntimeError("No DS18B20 found. Check wiring and the 4.7K pull-up resistor.")

for i, device in enumerate(devices):
    print("Device", i, "ROM:", [hex(x) for x in device.rom], "Family:", hex(device.family_code))

```

https://krillswarm.com/posts/2026/03/11/sensors/
 