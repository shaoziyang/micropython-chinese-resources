# Micropython程序优化原则

下面是MicroPython的作者Damien在pyconau 2018的视频中介绍的程序优化原则：

不在heap中分配内存的功能：

**表达式**

* if、while、for和try
* 局部变量
* 小整数
* 调用函数/方法
* 内置功能：all, any, callable, getatr, hasattr, isinstance, issubclass, len, max, min, ord, print, sum


**在heap中分配内存的功能：**

* import
* 定义函数和class
* 第一次分配全局变量
* 创建数据结构



**CPU运行时间**

* 使用函数，而不是全局方式
* 使用局部变量
* 局部缓存函数和对象
* 缓存自变量
* 用长表达式，不要分成多个
* 1<<3 这样的方式更好，它会被优化!



**RAM使用**

* 在可能时不要使用heap
* 短变量名，如 x, y, i, len, var
* 临时缓存：self.buf1 = bytearray(1)
* 使用 XXXX_into 方法
* 不使用 * 或 ** 参数
* from micropython import const; X = const(1)
* 简化脚本
* 使用mpy-cross产生mpy文件
* 终极方案：freeze 脚本到固件中


