# Micropython程序优化实例

这个优化例子来自 Damien 在 pycomau 上的演讲使用MicroPython高效快速编程。

首先我们看下面的程序，它在循环中翻转LED，然后通过运行的时间和翻转次数，计算出每秒翻转的频率。

```py
from machine import Pin
import time

led = Pin('A13')
N = 200000

t0 = time.ticks_us()

for i in range(N):
    led.on()
    led.off()

t1 = time.ticks_us()
dt = time.ticks_diff(t1, t0)
fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
print(fmt.format(dt * 1e-6, dt / N, N / dt * 1e3))
```

我们将这段代码保存为文件led1.py，然后import led1执行。在pybv10或者pyboardCN上结果是：

> 3.381 sec, 16.905 usec/blink :    59.16 kblink/sec

在 MicroPython程序优化原则 中，提到尽量在程序中执行功能，不要在主程序中运行，因此可以将LED翻转放在函数中执行。

```py
from machine import Pin
import time

led = Pin('A13')
N = 200000

def blink_simple(n):
    for i in range(n):
        led.on()
        led.off()

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```


运行后的结果是：

> 2.902 sec, 14.509 usec/blink :    68.92 kblink/sec

可以看到，我们没有做什么实质的改到，就明显提高了速度。

循环是最消耗运行时间的，我们对循环中led.on()和led.off()两个动作进行优化，将它们预先载入内存，而无需循环中每次载入。

```
from machine import Pin
import time

led = Pin('A13')
N = 200000

def blink_simple(n):
    on = led.on
    off = led.off
    for i in range(n):
        on()
        off()

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```

运行结果是

> 1.617 sec,  8.086 usec/blink :   123.68 kblink/sec

速度提高了将近一倍。

进一步将循环中对 range(n) 也进行优化

```py
from machine import Pin
import time

led = Pin('A13')
N = 200000

def blink_simple(n):
    on = led.on
    off = led.off
    r = range(n)
    for i in r:
        on()
        off()

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```


运行结果是

> 1.121 sec,  5.607 usec/blink :   178.35 kblink/sec

效果非常明显。

进一步对循环中的操作优化，减少循环次数

```py
from machine import Pin
import time

led = Pin('A13')
N = 200000

def blink_simple(n):
    n //= 8
    on = led.on
    off = led.off
    r = range(n)
    for i in r:
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```

速度又有明显提升。

> 0.913 sec,  4.563 usec/blink :   219.16 kblink/sec

根据MicroPython的优化功能，可以将程序声明为native code（本地代码），它使用CPU的操作码（opcode），而不是字节码（bytecode）

```py
from machine import Pin
import time

led = Pin('A13')
N = 200000

@micropython.native
def blink_simple(n):
    n //= 8
    on = led.on
    off = led.off
    r = range(n)
    for i in r:
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()
        on()
        off()

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```

结果如下

> 0.704 sec,  3.521 usec/blink :   284.00 kblink/sec

除了native，还可以使用viper code模式，它进一步提升了整数计算和位操作性能

```py
from machine import Pin
import time, stm

led = Pin('A13')
N = 200000

@micropython.viper
def blink_simple(n:int):
    n //= 8
    p = ptr16(stm.GPIOB + stm.GPIO_BSRR)
    for i in range(n):
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4
        p[0] = 1 << 4
        p[1] = 1 << 4

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```

运行结果的确是大幅提升了性能

> 0.016 sec,  0.078 usec/blink : 12879.13 kblink/sec

最终我们还可以通过嵌入汇编方式，最大限度提升性能

```py
from machine import Pin
import time, stm

led = Pin('A13')
N = 200000

@micropython.asm_thumb
def blink_simple(r0):
    lsr(r0, r0, 3)
    movwt(r1, stm.GPIOB + stm.GPIO_BSRR)
    mov(r2, 1 << 4)
    label(loop)
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    strh(r2, [r1, 0])
    strh(r2, [r1, 2])
    sub(r0, 1)
    bne(loop)

def time_it(f, n):
    t0 = time.ticks_us()
    f(n)
    t1 = time.ticks_us()
    dt = time.ticks_diff(t1, t0)
    fmt = '{:5.3f} sec, {:6.3f} usec/blink : {:8.2f} kblink/sec'
    print(fmt.format(dt * 1e-6, dt / n, n / dt * 1e3))

time_it(blink_simple, N)
```

运行结果是

> 0.007 sec,  0.037 usec/blink : 27322.40 kblink/sec

这个结果已经非常接近极限了。

从前面的优化顺序，可以看到我们并没有大幅修改程序，就可以极高程序的性能。实际使用中，大家可以灵活选择，提高程序的性能。
