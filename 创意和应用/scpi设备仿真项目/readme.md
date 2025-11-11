# 基于树莓派 Pico 与 MicroPython 的 SCPI 设备仿真项目

使用树莓派 Pico（Raspberry Pi Pico）+ MicroPython 实现 SCPI 设备仿真的实验性仓库

**简单易用**
1. 从官方渠道获取最新的 MicroPython UF2 固件： https://micropython.org/download/RPI_PICO/
2. 将固件烧录到 Pico 开发板中
3. 将 main.py、MicroScpiDevice.py 和 RaspberryScpiPico.py 三个文件复制到目标设备（Pico）的根目录
4. 重启设备，即可使用

**自行编译固件**
1. 克隆本仓库
2. 执行 `make docker` 命令构建 Docker 镜像
3. 执行 `make firmware` 命令编译 UF2 固件，编译产物将生成在 build 目录下

项目 github 链接：
https://github.com/K4zuki/pipico-micropython-scpi 
