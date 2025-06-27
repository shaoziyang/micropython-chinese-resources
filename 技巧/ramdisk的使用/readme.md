# ramdisk 的使用

```py
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
        if op == 4: # get number of blocks
            return len(self.data) // self.block_size
        if op == 5: # get block size
            return self.block_size

import vfs

bdev = RAMBlockDev(512, 50)
vfs.VfsFat.mkfs(bdev)
vfs.mount(bdev, '/ramdisk')
```

注：
- 注意不能设置太大的ramdisk，需要预留足够空间。
- 低版本固件需要使用 os 模块 代替 vfs。
