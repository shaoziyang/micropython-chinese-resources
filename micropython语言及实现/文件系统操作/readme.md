# 文件系统操作

本教程介绍 MicroPython 如何提供设备端文件系统，使得标准的 Python 文件 I/O 方法可以用于持久化存储。

MicroPython 会自动创建默认配置并自动检测主文件系统，因此本教程主要适用于需要修改分区、文件系统类型或使用自定义块设备的场景。

文件系统通常以设备的内部闪存为存储载体，但也可以使用外部闪存、RAM 或自定义块设备。

在某些硬件（例如 STM32）上，文件系统还可以通过 USB MSC 供主机 PC 访问。`pyboard.py` 工具也提供了一种让主机 PC 访问所有端口上文件系统的方式。

注意：这主要适用于裸机版本，如 STM32 和 ESP32。在带有操作系统的硬件（例如 Unix 版本）上，文件系统由主机操作系统提供。


## 虚拟文件系统（VFS）

MicroPython 实现了类 Unix 的虚拟文件系统（VFS）层。所有挂载的文件系统都被整合到一个单一的虚拟文件系统中，从根目录 `/` 开始。文件系统被挂载到该结构中的各个目录，并且在启动时，工作目录会切换到主文件系统的挂载位置。

在 STM32 / Pyboard 上，内部闪存被挂载到 `/flash` 目录，SD 卡（可选）被挂载到 `/sd` 目录。在 ESP8266/ESP32 上，主文件系统被挂载到 `/` 目录。


## 块设备

块设备是实现了 `vfs.AbstractBlockDev` 协议的类的实例。

### 内置块设备

各移植版提供内置块设备以访问其主闪存。

上电时，MicroPython 会尝试检测默认闪存上的文件系统，并自动进行配置和挂载。如果未找到文件系统，MicroPython 会尝试创建一个覆盖整个闪存的 FAT 文件系统。各移植版还可以提供“恢复出厂设置”主闪存的机制，通常是通过上电时按特定组合的按键来实现。

#### STM32 / Pyboard

`pyb.Flash` 类提供对内部闪存的访问。在一些配备了更大容量外部闪存的开发板（如 Pyboard D）上，该类会改用外部闪存。创建实例时应始终指定 `start` 参数，例如 `pyb.Flash(start=0)`。

注意：为保持向后兼容性，当无参数构造实例时（即 `pyb.Flash()`），它仅实现简单块接口，并映射到 USB MSC 所呈现的虚拟设备（即其开头包含一个虚拟分区表）。

#### ESP8266

内部闪存作为块设备对象暴露出来，该对象在启动时由 `flashbdev` 模块创建。默认情况下，此对象会作为全局变量存在，因此通常可直接通过 `bdev` 访问。它实现了扩展接口。

#### ESP32

`esp32.Partition` 类为开发板上定义的分区实现了块设备功能。与 ESP8266 类似，存在一个全局变量 `bdev`，它指向默认分区。该类实现了扩展接口。

### 自定义块设备

下面的类实现了一个简单的块设备，它使用 `bytearray` 在 RAM 中存储数据：

```python
class RAMBlockDev:
    def __init__(self, block_size, num_blocks):
        self.block_size = block_size
        self.data = bytearray(block_size * num_blocks)
    
    def readblocks(self, block_num, buf):
        for i in range(len(buf)):
            buf[i] = self.data[block_num * self.block_size + i]
    
    def writeblocks(self, block_num, buf):
        for i in range(len(buf)):
            self.data[block_num * self.block_size + i] = buf[i]
    
    def ioctl(self, op, arg):
        if op == 4:  # 获取块数量
            return len(self.data) // self.block_size
        if op == 5:  # 获取块大小
            return self.block_size
```

其使用方法如下：

```python
import vfs
bdev = RAMBlockDev(512, 50)
vfs.VfsFat.mkfs(bdev)
vfs.mount(bdev, '/ramdisk')
```

下面是一个同时支持简单接口和扩展接口（即同时支持 `vfs.AbstractBlockDev.readblocks()` 和 `vfs.AbstractBlockDev.writeblocks()` 方法的两种签名和行为）的块设备示例：

```python
class RAMBlockDev:
    def __init__(self, block_size, num_blocks):
        self.block_size = block_size
        self.data = bytearray(block_size * num_blocks)
    
    def readblocks(self, block_num, buf, offset=0):
        addr = block_num * self.block_size + offset
        for i in range(len(buf)):
            buf[i] = self.data[addr + i]
    
    def writeblocks(self, block_num, buf, offset=None):
        if offset is None:
            # 先擦除，再写入
            for i in range(len(buf) // self.block_size):
                self.ioctl(6, block_num + i)
            offset = 0
        addr = block_num * self.block_size + offset
        for i in range(len(buf)):
            self.data[addr + i] = buf[i]
    
    def ioctl(self, op, arg):
        if op == 4:  # 块数量
            return len(self.data) // self.block_size
        if op == 5:  # 块大小
            return self.block_size
        if op == 6:  # 块擦除
            return 0
```

由于它支持扩展接口，因此可与 `littlefs` 配合使用：

```python
import vfs
bdev = RAMBlockDev(512, 50)
vfs.VfsLfs2.mkfs(bdev)
vfs.mount(bdev, '/ramdisk')
```

挂载后，无论文件系统类型如何，都可以像在常规 Python 代码中那样使用它，例如：

```python
with open('/ramdisk/hello.txt', 'w') as f:
    f.write('Hello world')
print(open('/ramdisk/hello.txt').read())
```


## 文件系统

MicroPython 可提供 `FAT`、`littlefs v1` 和 `littlefs v2` 文件系统的实现。

下表展示了在给定硬件/开发板组合中，固件默认包含哪些文件系统，不过在自定义固件构建中，这些文件系统也可按需启用。

| 开发板                | FAT  | littlefs v1 | littlefs v2 |
|-----------------------|------|-------------|-------------|
| pyboard 1.0、1.1、D   | 是   | 否          | 是          |
| 其他 STM32 开发板     | 是   | 否          | 否          |
| ESP8266（1M 闪存）    | 否   | 否          | 是          |
| ESP8266（2M+ 闪存）   | 是   | 否          | 是          |
| ESP32                 | 是   | 否          | 是          |


### FAT

FAT 文件系统的主要优势在于，在支持的开发板（如 STM32）上，无需在主机 PC 上安装额外驱动，即可通过 USB MSC 访问该文件系统。

不过，FAT 在写入过程中若遇到断电情况，容错性较差，可能会导致文件系统损坏。对于不需要 USB MSC 的应用，建议改用 littlefs。

使用 FAT 格式化整个闪存的方法：

```python
# ESP8266 和 ESP32
import vfs
vfs.umount('/')
vfs.VfsFat.mkfs(bdev)
vfs.mount(bdev, '/')

# STM32
import os, vfs, pyb
vfs.umount('/flash')
vfs.VfsFat.mkfs(pyb.Flash(start=0))
vfs.mount(pyb.Flash(start=0), '/flash')
os.chdir('/flash')
```


### Littlefs

[Littlefs](https://github.com/littlefs-project/littlefs) 是专为基于闪存的设备设计的文件系统，对文件系统损坏的抵抗能力更强。

> **注意**  
> 有报告称 littlefs v1 和 v2 在某些情况下会出现故障，详情参见 littlefs 的 [issue 347](https://github.com/littlefs-project/littlefs/issues/347) 和 [issue 295](https://github.com/littlefs-project/littlefs/issues/295)。

使用 littlefs v2 格式化整个闪存的方法：

```python
# ESP8266 和 ESP32
import vfs
vfs.umount('/')
vfs.VfsLfs2.mkfs(bdev)
vfs.mount(bdev, '/')

# STM32
import os, vfs, pyb
vfs.umount('/flash')
vfs.VfsLfs2.mkfs(pyb.Flash(start=0))
vfs.mount(pyb.Flash(start=0), '/flash')
os.chdir('/flash')
```

通过 [littlefs FUSE 驱动](https://github.com/littlefs-project/littlefs-fuse)，仍可在 PC 上通过 USB MSC 访问 littlefs 文件系统。注意，必须指定 `--block_size` 和 `--block_count` 选项以覆盖默认值。例如（构建好 littlefs-fuse 可执行文件后）：

```bash
$ ./lfs --block_size=4096 --block_count=512 -o allow_other /dev/sdb1 mnt
```

这样就可以在 `mnt` 目录下访问开发板的 littlefs 文件系统了。要获取正确的 `block_size` 和 `block_count` 值，可使用以下代码：

```python
import pyb
f = pyb.Flash(start=0)
f.ioctl(1, 1)  # 以 littlefs 原始块模式初始化闪存
block_count = f.ioctl(4, 0)
block_size = f.ioctl(5, 0)
```


### 混合分区（STM32）

通过给 `pyb.Flash` 传入 `start` 和 `len` 参数，可以创建覆盖闪存设备一部分区域的块设备。

例如，将前 256KiB 配置为 FAT 格式（可通过 USB MSC 访问），其余部分配置为 littlefs 格式：

```python
import os, vfs, pyb
vfs.umount('/flash')
p1 = pyb.Flash(start=0, len=256*1024)
p2 = pyb.Flash(start=256*1024)
vfs.VfsFat.mkfs(p1)
vfs.VfsLfs2.mkfs(p2)
vfs.mount(p1, '/flash')
vfs.mount(p2, '/data')
os.chdir('/flash')
```

这种配置的用处在于，可通过 USB MSC 访问 Python 文件、配置以及其他很少修改的内容，同时让频繁变化的应用数据存储在 littlefs 中，以获得更好的抗断电等特性。

偏移量为 0 的分区会自动挂载（文件系统类型也会自动检测），但可以在 `boot.py` 中添加以下代码来挂载数据分区：

```python
import vfs, pyb
p2 = pyb.Flash(start=256*1024)
vfs.mount(p2, '/data')
```


### 混合分区（ESP32）

在 ESP32 上，如果构建自定义固件，可以修改 `partitions.csv` 文件来定义任意的分区布局。

启动时，名为“vfs”的分区默认会挂载到 `/` 目录，但可以在 `boot.py` 中使用以下代码挂载其他任何分区：

```python
import esp32, vfs
p = esp32.Partition.find(esp32.Partition.TYPE_DATA, label='foo')
vfs.mount(p, '/foo')
```
