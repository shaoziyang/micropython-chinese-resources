# 编译rp2源码

编译rp2固件，需要先安装 cmake和 build-essential。
```
sudo apt install cmake build-essential
```

安装后就可以直接编译了，方法和stm32基本一样。
```
make -C ports/rp2 -j8
```
