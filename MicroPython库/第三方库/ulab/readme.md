# ulab

ulab 是一个用于 micropython 和 CircuitPython 的类似 numpy 阵列处理库。该模块以C语言编写，且速度非常快，为一至四维的数值数据定义了紧凑型容器(ndarray)。该库是一个仅限软件的标准微Python用户模块，没有硬件依赖关系，可针对任何平台进行编译。支持8位、16位有符号和无符号整数，以及浮点数，并可选支持复数。

- [github 代码仓库](https://github.com/v923z/micropython-ulab)
- [文档](https://micropython-ulab.readthedocs.io/en/latest/ulab-intro.html)
- [ulab 范例](https://github.com/rcolistete/ulab_samples)

ulab需要编译到固件中才能使用，最简单的方法是在编译时，将ulab的代码作为用户模块进行编译。例如在esp32s3中，正常编译使用命令：
```
make -C ports/esp32/ BOARD=ESP32_GENERIC_S3 -j8
```

假设 ulab 源码通过 git 下载到 ~/projects/ulab 中，那么就可以在编译时使用下面命令，将ulab 编译到固件中：
```
make -C ports/esp32/ BOARD=ESP32_GENERIC_S3 -j8 USER_C_MODULES=~/projects/ulab/code/micropython.cmake
```

esp32 使用 cmake 工具链，所以上面的用户模块文件是 micropython.cmake，如果是使用其它 mcu，如 pyboard，使用下面命令：
```
make -C ports/stm32/ -j8 USER_C_MODULES=~/projects/ulab/code/
```
