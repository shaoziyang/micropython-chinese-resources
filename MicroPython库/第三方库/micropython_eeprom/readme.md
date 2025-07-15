# 支持多种存储芯片的 micropython_eeprom 模块

**micropython_eeprom** 是一个支持多种存储器的 micropython 驱动，可以将储存器挂载为本地磁盘，直接以文件方式操作。目前支持 eeprom、 flash、FRAM、SPIRAM等，社区测试了 eeprom 和 flash。

| Manufacturer | Part | Interface | Bytes | Technology |
| --- | --- | --- | --- | --- |
| Various | Various | SPI 4096 | <=32MiB | Flash |
| STM | M95M02-DR | SPI 256 | 256KiB | EEPROM |
| Microchip | 25xx1024 | SPI 256 | 128KiB | EEPROM |
| Microchip | 25xx512* | SPI 256 | 64KiB | EEPROM |
| Microchip | 24xx512 | I2C 128 | 64KiB | EEPROM |
| Microchip | 24xx256 | I2C 128 | 32KiB | EEPROM |
| Microchip | 24xx128 | I2C 128 | 16KiB | EEPROM |
| Microchip | 24xx64 | I2C 128 | 8KiB | EEPROM |
| Microchip | 24xx32 | I2C 32 | 4KiB | EEPROM |
| Adafruit | 4719 | SPI n/a | 512KiB | FRAM |
| Adafruit | 4718 | SPI n/a | 256KiB | FRAM |
| Adafruit | 1895 | I2C n/a | 32KiB | FRAM |
| Adafruit | 4677 | SPI n/a | 8MiB | SPIRAM |

使用方法

24LC512（64KB容量）：

```py
import vfs
from machine import I2C, Pin
from eeprom_i2c import EEPROM, T24C512
i2c = I2C(0, sda=Pin(23), scl=Pin(18),freq=400000)
eep = EEPROM(i2c, T24C512)

#vfs.VfsLfs2.mkfs(eep)
vfs.mount(eep,'/e')
```

W25Q32
```py
from machine import Pin, SPI
import vfs
from flash_spi import FLASH

PIN_CS = 5
PIN_MI = 19
PIN_MO = 23
PIN_SCK = 18

PIN_D2 = 25
PIN_D3 = 26

Pin(PIN_D2, Pin.OUT, value=1)
Pin(PIN_D3, Pin.OUT, value=1)

cspins = (Pin(PIN_CS, Pin.OUT, value=1), )
flash = FLASH(SPI(2, baudrate=2000000), cspins)

vfs.mount(flash, '/f')
```

对于W25Q64/W25Q128等，需要增加 `cmd5=False` 参数。

`flash = FLASH(SPI(2, baudrate=2000000), cspins, cmd5=False)`

第一次挂载时，需要先用 `vfs.VfsLfs2.mkfs` 或 `vfs.VfsFat.mkfs` 函数创建文件系统，以后就可以直接挂载了。

- https://github.com/peterhinch/micropython_eeprom
