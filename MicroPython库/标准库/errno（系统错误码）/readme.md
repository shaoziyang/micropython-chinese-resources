# 7. errno（系统错误码）

此模块实现了相应CPython模块的子集，提供OSError异常的符号错误代码，特定的代码清单取决于MicroPython移植版本。

## 常数

`EEXIST、EAGAIN`等，基于ANSI C/POSIX标准错误代码。所有错误代码都以 "E" 开头。如上所述，错误通常可以作为`exc.errno`访问，其中`exc`是OSError的一个实例：

```
try:
    os.mkdir("my_dir")
except OSError as exc:
    if exc.errno == errno.EEXIST:
        print("Directory already exists")
```

errno.`errorcode`，将数字错误代码映射到错误代码的字符串字典

```
>>> print(errno.errorcode[errno.EEXIST])
EEXIST
```
