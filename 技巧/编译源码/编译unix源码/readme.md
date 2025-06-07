# 编译unix源码

编译unix版本，需要额外安装 build-essential、libffi、pkg-config，方法是：
```
sudo apt install build-essential libffi-dev pkg-config
```

安装后，就可以直接编译了。
```
make -C ports/unix -j8
```