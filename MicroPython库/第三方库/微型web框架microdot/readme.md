# 微型 Web 框架 microdot

用于 Python 和 MicroPython 的小得不可思议的 Web 框架。Microdot 是一个受 Flask 启发的简约 Python Web 框架，旨在在微控制器等资源有限的系统上运行。它在标准 Python 和 MicroPython 上运行。

```py
from microdot import Microdot

app = Microdot()

@app.route('/')
def index(request):
    return 'Hello, world!'

app.run()
```

**资源**
- [Documentation](https://microdot.readthedocs.io/en/latest/)
- [Change Log](https://github.com/miguelgrinberg/microdot/blob/main/CHANGES.md)

**其它**
- [A Better Web Server for Raspberry Pi Pico W](http://www.doctormonk.com/2022/09/a-better-web-server-for-raspberry-pi.html)
