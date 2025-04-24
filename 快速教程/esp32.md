# ESP32 快速使用教程

![esp32](esp32.webp)

- [General board control](#Generalboardcontrol)
- [Network](#Network)
- [Delay and timing](#Delayandtiming)
- [Timers](#Timers)
- [Pins and GPIO](#Pins)
- [引脚中断](#引脚中断)
- [UART](#UART)
- [PWM](#PWM)
- [ADC](#ADC)
- [Software SPI bus](#SoftwareSPIbus)
- [Hardware SPI bus](#HardwareSPIbus)
- [Software I2C bus](#SoftwareI2Cbus)
- [Hardware I2C bus](#HardwareI2Cbus)
- [I2S bus](#I2S)
- [Real time clock](#RTC)
- [WDT](#WDT)
- [Deep-sleep mode](#Deep-sleep)
- [SD card](#SDcard)
- [RMT](#RMT)
- [OneWire driver](#OneWiredriver)
- [NeoPixel and APA106 driver](#NeoPixel)
- [Capacitive touch](#Capacitivetouch)
- [DHT driver](#DHTdriver)
- [webrepl](#webrepl)

* * *

### <a name='Generalboardcontrol'>General board control</a>
通用控制

`machine` module
```py
import machine

machine.freq()          # get the current frequency of the CPU
machine.freq(240000000) # set the CPU frequency to 240 MHz
```

`esp` module
```py
import esp

esp.osdebug(None)       # turn off vendor O/S debugging messages
esp.osdebug(0)          # redirect vendor O/S debugging messages to UART(0)

# low level methods to interact with flash storage
esp.flash_size()
esp.flash_user_start()
esp.flash_erase(sector_no)
esp.flash_write(byte_offset, buffer)
esp.flash_read(byte_offset, buffer)
```

`esp32` module
```py
import esp32

esp32.hall_sensor()     # read the internal hall sensor
esp32.raw_temperature() # read the internal temperature of the MCU, in Fahrenheit
esp32.ULP()             # access to the Ultra-Low-Power Co-processor
```


### <a name=Network>Networking</a>
网络

`network` module:
```py
import network

wlan = network.WLAN(network.STA_IF) # create station interface
wlan.active(True)       # activate the interface
wlan.scan()             # scan for access points
wlan.isconnected()      # check if the station is connected to an AP
wlan.connect('essid', 'password') # connect to an AP
wlan.config('mac')      # get the interface's MAC address
wlan.ifconfig()         # get the interface's IP/netmask/gw/DNS addresses

ap = network.WLAN(network.AP_IF) # create access-point interface
ap.config(essid='ESP-AP') # set the ESSID of the access point
ap.config(max_clients=10) # set how many clients can connect to the network
ap.active(True)         # activate the interface
```

连接到本地 WiFi 网络的一个函数
```py
def do_connect():
    import network
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    if not wlan.isconnected():
        print('connecting to network...')
        wlan.connect('essid', 'password')
        while not wlan.isconnected():
            pass
    print('network config:', wlan.ifconfig())
```

### <a name='Delayandtiming'>Delay and timing</a>
```py
import time

time.sleep(1)           # sleep for 1 second
time.sleep_ms(500)      # sleep for 500 milliseconds
time.sleep_us(10)       # sleep for 10 microseconds
start = time.ticks_ms() # get millisecond counter
delta = time.ticks_diff(time.ticks_ms(), start) # compute time difference
```

### <a name='Timers'>Timers</a>
定时器
ESP32 有4个硬件定时器可以通过 machine.Timer 使用定位器，ID 范围是 0 到 3:

```py
from machine import Timer

tim0 = Timer(0)
tim0.init(period=5000, mode=Timer.ONE_SHOT, callback=lambda t:print(0))

tim1 = Timer(1)
tim1.init(period=2000, mode=Timer.PERIODIC, callback=lambda t:print(1))
```
period是毫秒

### <a name='Pins'>Pins and GPIO</a>
```py
from machine import Pin

p0 = Pin(0, Pin.OUT)    # create output pin on GPIO0
p0.on()                 # set pin to "on" (high) level
p0.off()                # set pin to "off" (low) level
p0.value(1)             # set pin to on/high

p2 = Pin(2, Pin.IN)     # create input pin on GPIO2
print(p2.value())       # get value, 0 or 1

p4 = Pin(4, Pin.IN, Pin.PULL_UP) # enable internal pull-up resistor
p5 = Pin(5, Pin.OUT, value=1) # set pin high on creation
```

可用引脚： 0-19, 21-23, 25-27, 32-39

- Pins 1 和 3 用于 REPL 的 TX 和 RX
- Pins 6, 7, 8, 11, 16 和 17 用于连接内置的 flash, 不能使用
- Pins 34-39 只能用于输入, 没有内部上拉电阻
- 部分引脚上拉可以设置为 Pin.PULL_HOLD 以减少 deepsleep 时功耗.

### <a name='引脚中断'>引脚中断</a>
```py
from machine import Pin

p0 = Pin(0, Pin.IN)
p0.irq(trigger=Pin.IRQ_RISING, handler=lambda t: print(1))
```
触发方式可以选择上升沿 ( Pin.IRQ_RISING )、下降沿( Pin.IRQ_FALLING )，或者两者的组合( Pin.IRQ_RISING|Pin.IRQ_FALLING )

### <a name='UART'>UART (serial bus)</a>
```py
from machine import UART

uart1 = UART(1, baudrate=9600, tx=33, rx=32)
uart1.write('hello')  # write 5 bytes
uart1.read(5)         # read up to 5 bytes
```
串口引脚可以重新分配，默认定义为：

| |UART0|UART1|UART2|
|-|-|-|-|
|tx|1|10|17|
|rx|3|9|16|

### <a name='PWM'>PWM (pulse width modulation)</a>
所有输出引脚都支持PWM，PWM频率范围从 1Hz 到 40MHz，随着频率增加，PWM占空比分辨率会降低。
```py
from machine import Pin, PWM

pwm0 = PWM(Pin(0))         # create PWM object from a pin
pwm0.freq()                # get current frequency (default 5kHz)
pwm0.freq(1000)            # set PWM frequency from 1Hz to 40MHz
pwm0.duty()                # get current duty cycle, range 0-1023 (default 512, 50%)
pwm0.duty(256)             # set duty cycle from 0 to 1023 as a ratio duty/1023, (now 25%)
pwm0.duty_u16(2**16*3//4)  # set duty cycle from 0 to 65535 as a ratio duty_u16/65535, (now 75%)
pwm0.duty_u16()            # get current duty cycle, range 0-65535
pwm0.duty_ns(250_000)      # set pulse width in nanoseconds from 0 to 1_000_000_000/freq, (now 25%)
pwm0.duty_ns()             # get current pulse width in ns
pwm0.deinit()              # turn off PWM on the pin

pwm2 = PWM(Pin(2), freq=20000, duty=512)  # create and configure in one go
print(pwm2)                               # view PWM settings
```

ESP 系列芯片有不同的硬件外设

|硬件规格|ESP32|ESP32-S2|ESP32-C3|
|-|-|-|-|
|Number of groups (speed modes)|2|1|1|
|Number of timers per group|4|4|4|
|Number of channels per group|8|8|6|
|Different of PWM frequencies (groups * timers)|8|4|4|
|Total PWM channels (Pins, duties) (groups * channels)|16|8|6|

### <a name='ADC'>ADC (analog to digital conversion)</a>
在 ESP32 上，引脚 32-39 提供 ADC 功能。请注意，使用默认配置时，ADC 引脚上的输入电压必须介于 0.0v 和 1.0v 之间（任何高于 1.0v 的值都将读取为 4095）。必须应用衰减以增加此可用电压范围。

```py
from machine import ADC

adc = ADC(Pin(32))          # create ADC object on ADC pin
adc.read()                  # read value, 0-4095 across voltage range 0.0v - 1.0v

adc.atten(ADC.ATTN_11DB)    # set 11dB input attenuation (voltage range roughly 0.0v - 3.6v)
adc.width(ADC.WIDTH_9BIT)   # set 9 bit return values (returned range 0-511)
adc.read()                  # read value using the newly configured attenuation and width
```

### <a name='SoftwareSPIbus'>Software SPI bus</a>
```py
from machine import Pin, SoftSPI

# construct a SoftSPI bus on the given pins
# polarity is the idle state of SCK
# phase=0 means sample on the first edge of SCK, phase=1 means the second
spi = SoftSPI(baudrate=100000, polarity=1, phase=0, sck=Pin(0), mosi=Pin(2), miso=Pin(4))

spi.init(baudrate=200000) # set the baudrate

spi.read(10)            # read 10 bytes on MISO
spi.read(10, 0xff)      # read 10 bytes while outputting 0xff on MOSI

buf = bytearray(50)     # create a buffer
spi.readinto(buf)       # read into the given buffer (reads 50 bytes in this case)
spi.readinto(buf, 0xff) # read into the given buffer and output 0xff on MOSI

spi.write(b'12345')     # write 5 bytes on MOSI

buf = bytearray(4)      # create a buffer
spi.write_readinto(b'1234', buf) # write to MOSI and read from MISO into the buffer
spi.write_readinto(buf, buf) # write buf to MOSI and read MISO back into buf
```

### <a name='HardwareSPIbus'>Hardware SPI bus</a>
有两个硬件 SPI 通道可以实现更快的传输速率（高达 80Mhz）。这些可用于支持所需方向的任何 IO 引脚，当在下面列出的默认引脚之外的引脚上使用时，硬件 SPI 通道被限制为 40MHz。

| |HSPI (id=1)|VSPI (id=2)|
|-|-|-|
|sck|14|18|
|mosi|13|23|
|miso|12|19|

```py
from machine import Pin, SPI

hspi = SPI(1, 10000000)
hspi = SPI(1, 10000000, sck=Pin(14), mosi=Pin(13), miso=Pin(12))
vspi = SPI(2, baudrate=80000000, polarity=0, phase=0, bits=8, firstbit=0, sck=Pin(18), mosi=Pin(23), miso=Pin(19))
```

### <a name='SoftwareI2Cbus'>Software I2C bus</a>
```py
from machine import Pin, SoftI2C

i2c = SoftI2C(scl=Pin(5), sda=Pin(4), freq=100000)

i2c.scan()              # scan for devices

i2c.readfrom(0x3a, 4)   # read 4 bytes from device with address 0x3a
i2c.writeto(0x3a, '12') # write '12' to device with address 0x3a

buf = bytearray(10)     # create a buffer with 10 bytes
i2c.writeto(0x3a, buf)  # write the given buffer to the peripheral
```

### <a name='HardwareI2Cbus'>Hardware I2C bus</a>
有两个硬件 I2C 外设，标识符为 0 和 1。任何可用的具有输出功能的引脚均可用于 SCL 和 SDA，但默认值如下所示。

| |I2C0|I2C1|
|-|-|-|
|scl|18|25|
|sda|19|26|

```py
from machine import Pin, I2C

i2c = I2C(0)
i2c = I2C(1, scl=Pin(5), sda=Pin(4), freq=400000)
```

### <a name='I2S'>I2S bus</a>
```py
from machine import I2S, Pin

i2s = I2S(0, sck=Pin(13), ws=Pin(14), sd=Pin(34), mode=I2S.TX, bits=16, format=I2S.STEREO, rate=44100, ibuf=40000) # create I2S object
i2s.write(buf)             # write buffer of audio samples to I2S device

i2s = I2S(1, sck=Pin(33), ws=Pin(25), sd=Pin(32), mode=I2S.RX, bits=16, format=I2S.MONO, rate=22050, ibuf=40000) # create I2S object
i2s.readinto(buf)          # fill buffer with audio samples from I2S device
```

### <a name='RTC'>Real time clock (RTC)</a>
```py
from machine import RTC

rtc = RTC()
rtc.datetime((2017, 8, 23, 1, 12, 48, 0, 0)) # set a specific date and time
rtc.datetime() # get date and time
```

### <a name='WDT'>WDT</a> (Watchdog timer)
```
from machine import WDT

# enable the WDT with a timeout of 5s (1s is the minimum)
wdt = WDT(timeout=5000)
wdt.feed()
```

### <a name='Deep-sleep'>Deep-sleep mode</a>
```
import machine

# check if the device woke from a deep sleep
if machine.reset_cause() == machine.DEEPSLEEP_RESET:
    print('woke from a deep sleep')

# put the device to sleep for 10 seconds
machine.deepsleep(10000)
```

**注**
- 不带参数调用 deepsleep() 将使设备无限期地休眠
- 软件复位不会改变复位原因
- 可能有一些漏电流流过启用的内部上拉。为了进一步降低功耗，可以禁用内部上拉：
`p1 = Pin(4, Pin.IN, Pin.PULL_HOLD)`
退出 deepsleep 后，可能需要通过以下方式明确取消保持引脚（例如，如果它是输出引脚）
`p1 = Pin(4, Pin.OUT, None)`

### <a name='SDcard'>SD card</a>
```py
import machine, os

# Slot 2 uses pins sck=18, cs=5, miso=19, mosi=23
sd = machine.SDCard(slot=2)
os.mount(sd, "/sd")  # mount

os.listdir('/sd')    # list directory contents

os.umount('/sd')     # eject
```

### <a name='RMT'>RMT</a>
```py
import esp32
from machine import Pin

r = esp32.RMT(0, pin=Pin(18), clock_div=8)
r   # RMT(channel=0, pin=18, source_freq=80000000, clock_div=8)
# The channel resolution is 100ns (1/(source_freq/clock_div)).
r.write_pulses((1, 20, 2, 40), start=0) # Send 0 for 100ns, 1 for 2000ns, 0 for 200ns, 1 for 4000ns
```

### <a name='OneWiredriver'>OneWire driver</a>
OneWire驱动是用软件实现的，可以在所有引脚上工作。
```py
from machine import Pin
import onewire

ow = onewire.OneWire(Pin(12)) # create a OneWire bus on GPIO12
ow.scan()               # return a list of devices on the bus
ow.reset()              # reset the bus
ow.readbyte()           # read a byte
ow.writebyte(0x12)      # write a byte on the bus
ow.write('123')         # write bytes on the bus
ow.select_rom(b'12345678') # select a specific device by its ROM code
```

读取 DS18S20 / DS18B20
```py
import time, ds18x20
ds = ds18x20.DS18X20(ow)
roms = ds.scan()
ds.convert_temp()
time.sleep_ms(750)
for rom in roms:
    print(ds.read_temp(rom))
```

### <a name='NeoPixel'>NeoPixel and APA106 driver</a>

```py
from machine import Pin
from neopixel import NeoPixel

pin = Pin(0, Pin.OUT)   # set GPIO0 to output to drive NeoPixels
np = NeoPixel(pin, 8)   # create NeoPixel driver on GPIO0 for 8 pixels
np[0] = (255, 255, 255) # set the first pixel to white
np.write()              # write data to all pixels
r, g, b = np[0]         # get first pixel colour
```

**APA106**
```py
from apa106 import APA106
ap = APA106(pin, 8)
r, g, b = ap[0]
```

底层驱动
```py
import esp
esp.neopixel_write(pin, grb_buf, is800khz)
```

### <a name='Capacitivetouch'>Capacitive touch</a>
电容触摸
```py
from machine import TouchPad, Pin

t = TouchPad(Pin(14))
t.read()              # Returns a smaller number when touched
```
TouchPad.read 返回一个相对于电容变化的数值。当一个引脚被触摸时，小数字（通常是几十）是常见的，当没有触摸时，大数字（超过一千）是常见的。然而，这些数值是相对的，可能会因电路板和周围的组成而有所不同，所以可能需要进行一些校准。

在ESP32上有10个支持电容式触摸的引脚可以使用：0、2、4、12、13 14、15、27、32、33。试图分配到任何其他引脚将导致ValueError。

触摸功能可以用于从休眠中唤醒
```py
import machine
from machine import TouchPad, Pin
import esp32

t = TouchPad(Pin(14))
t.config(500)               # configure the threshold at which the pin is considered touched
esp32.wake_on_touch(True)
machine.lightsleep()        # put the MCU to sleep until a touchpad is touched
```

### <a name='DHTdriver'>DHT driver</a>
```py
import dht
import machine

d = dht.DHT11(machine.Pin(4))
d.measure()
d.temperature() # eg. 23 (°C)
d.humidity()    # eg. 41 (% RH)

d = dht.DHT22(machine.Pin(4))
d.measure()
d.temperature() # eg. 23.6 (°C)
d.humidity()    # eg. 41.3 (% RH)
```

### <a name='webrepl'>WebREPL </a>(web browser interactive prompt)
```py
import webrepl
webrepl.start()

# or, start with a specific password
webrepl.start(password='mypass')
```
