# 微型 HTTP 工具包 uht   
运行 [MicroPython](https://github.com/micropython/micropython) 或 [CircuitPython](https://github.com/adafruit/circuitpython) 的微型设备（ESP32、Raspberry Pi Pico 等）上的最小 HTTP/1.0 服务器。与 MicroPython 1.21+ 兼容。
   
安装
```
mpremote mip install logging
mpremote mip install "https://github.com/nmattia/uht/releases/latest/download/uht.py" # .mpy is also available
```
   
运行
```
from uht import HTTPServer

app = HTTPServer()

@app.route("/")
async def index(req, resp):
    await resp.send(b"Hello, world!")

app.run()  # Starts the server on 127.0.0.1:8081
```
   
[https://github.com/nmattia/uht](https://github.com/nmattia/uht)
