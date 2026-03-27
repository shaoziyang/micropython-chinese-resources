# 使用 memoryview

memoryview 可以指向一个buffer类型数据（有点类似C语言的指针）或者它的一部分，不需要复制对象就能直接访问（如切片），提高了数据处理效率。
```python
rbuf = bytearray(10)
vbuf = memoryview(rbuf)[2:6]
vbuf[0] = 65
```

在micropython中，有时需要对一些可变长度的数据（如 i2c 中使用 memory 方式操作寄存器）进行处理，这时使用 memoryview 就会非常方便。
```python
def getRegs(self, reg, n=1):
    self.vbuf = memoryview(self.rbuf)[:n]
    self.i2c.readfrom_mem_into(self.addr, reg, self.vbuf) 
```
