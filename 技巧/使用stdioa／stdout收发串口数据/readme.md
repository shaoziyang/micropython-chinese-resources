# 使用stdioa／stdout收发串口数据

REPL是通过串口访问的，需要占用一个硬件串口。在一些MCU中，串口数量比较少（如esp8266），往往会给应用带来不便。这时可以通过sys.stdin和sys.stdout收发数据，以缓解串口不足的问题。下面是一个简单的数据读取发送的例子：

```
import sys
import select
import time

while True:
  time.sleep(0.1)
   
  while sys.stdin in select.select([sys.stdin], [], [], 0)[0]:
    ch = sys.stdin.read(1)
    print(ch)
```

使用print发送数据时，在最后会附加 '\x0D\x0A' ， 也就是回车换行符，需要自行处理一下，如`print(ch, end='')`。另外不能发送 '\x03'，因为这时REPL的键盘中断命令。如果可能会发送这个数据，就需要先关闭键盘中断。

print也可以改为sys.stdout.write()，但是需要注意它会除了会增加 '\x0D\x0A' 外，还会在 '\x0D\x0A' 前增加一个数字，代表发送的字节数。