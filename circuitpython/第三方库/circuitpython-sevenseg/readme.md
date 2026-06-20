# CircuitPython Sevenseg

轻量级的CircuitPython库，用于通过 GPIO 控制单个7段数码管显示 0-9、A-Z、a-z、符号等。

```
 ---a---
|       |
f       b
|       |
 ---g---
|       |
e       c
|       |
 ---d---   ● dp

引脚顺序: [a, b, c, d, e, f, g, dp]
```

## 使用方法

```py
import board
import time
from circuitpython_sevenseg import SevenSeg

seg = SevenSeg(
    pins=[board.GP2, board.GP3, board.GP4, board.GP5,
          board.GP6, board.GP7, board.GP8, board.GP9],
    common_anode=False
)

# Show a digit
seg.show(5)

# Show a letter
seg.show('A')

# Show with decimal point
seg.show(3, dp=True)

# Turn on decimal point only
seg.dot(True)

# Clear display
seg.clear()

# Release pins
seg.deinit()
```

## 相关链接

- https://github.com/kritishmohapatra/circuitpython-sevenseg

