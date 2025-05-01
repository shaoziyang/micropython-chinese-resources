# 编译windows源码

编译windows版本要稍微复杂一点，因为需要安装额外编译器。在Linux下需要安装Mingw-w64交叉编译器，在windows下可以使用Cygwin、MSYS2或者Visual Studio。下面仅介绍Linux下的编译方法，其它方式读者可以参考相关文档。
先安装 Mingw-w64：
```
sudo apt-get install build-essential gcc-mingw-w64
```

编译时，需要指定编译器名称CROSS_COMPILE=i686-w64-mingw32-，注意名称的最后有个减号。
```
make -C ports/windows/ CROSS_COMPILE=i686-w64-mingw32- -j8
```

stm32/esp32版本可以在芯片中使用，unix/windwos版本有什么用处呢？它们可以用于逻辑和算法验证、数值计算、文件处理等等，这些功能在pc上进行比直接在芯片上要快速方便。相比PC上完整的python软件，它们非常小巧，简单灵活，可以作为unix/windows上独立的程序集成到其它项目中作为功能模块使用。