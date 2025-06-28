# esptool.py

`esptool.py` 是 esp32/esp8266 专用的 flash 下载、上传、擦除工具，它是一个 python 的命令行程序，可以通过 python 的 pip 安装。下面是它的基本用法：

**擦除 Flash**
`esptool.py --port /dev/ttyUSB0 erase_flash`
`esptool.py --port COM10 erase_flash`
`esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash`

**下载固件**

- esp8266
    `esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 esp8266-20170108-v1.8.7.bin`
- esp32
    `esptool.py --chip esp32 --port /dev/ttyUSB0 write_flash -z 0x1000 esp32-20180511-v1.9.4.bin`

- esp32c3
`esptool.py --chip esp32c3 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x0 esp32c3-20220117-v1.18.bin`

**注**
- `--port`可以简化为 `-p`
- `--baud`可以简化为 `-b`

windows的批处理文件：

- `esptool.py -p %1 --baud 460800 write_flash --flash_size=detect 0 %2`
- `esptool.py --chip esp32 -p %1 write_flash -z 0x1000 %2`