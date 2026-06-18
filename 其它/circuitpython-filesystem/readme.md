# CircuitPython/MicroPython Filesystem

通过串口REPL访问 CircuitPython / MicroPython 设备的文件系统，适合不支持原生 USB 系统的设备，如 ESP32, ESP32-C3, ESP32-C6 等。 文件系统可以通过三种方式使用: FUSE、FTP 或 WebDAV。

| 工具 | 协议 | 用法 |
| --- | --- | --- |
| `circuitpython_fuse.py` | FUSE / WinFsp | 加载为系统驱动器（如 z:），任何程序都可以访问 |
| `circuitpython_ftp.py` | FTP | 跨平台/远程访问(FileZilla、WinSCP 等) |
| `circuitpython_webdav.py` | WebDAV | 通过 http 加载 （Explorer / Finder / Nautilus） |


## 使用方法

```py
# Mount as drive Z: (Windows)
python circuitpython_fuse.py -p COM12 -m Z:

# FTP server on :2121
python circuitpython_ftp.py -p COM12 --ftp-port 2121

# WebDAV server on :8080
python circuitpython_webdav.py -p COM12 --http-port 8080

# One-off CLI
python cpfs.py -p COM12 ls :/
python cpfs.py -p COM12 cp ./local.py :/code.py     # upload
python cpfs.py -p COM12 cp :/code.py ./backup.py    # download
```

## 性能

主要瓶颈在串口通信，写入速度约 4KB/s，读取速度约 8-30KB/s。

`--fast`选项可降低串行延时（大约快20-40%），但可能会导致某些设备上的数据损坏。

## 相关链接
- [github仓库](https://github.com/MakerClassCZ/circuitpython-filesystem)
