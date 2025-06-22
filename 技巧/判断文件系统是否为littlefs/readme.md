# 判断文件系统是否为 LittleFs

```py
def check_for_littlefs():
    from flashbdev import bdev
    buf = bytearray(16)
    bdev.readblocks(0, buf)
    return buf[8:16] == b"littlefs"
check_for_littlefs()
```
