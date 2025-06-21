# 字符串/bytes 和 hex 相互转换

通过内置模块 `binascii` 中的 `hexlify` 和 `unhexlify` 函数，就可以方便的将字符串和16进制数据转换。

```
>>> import binascii
>>> binascii.hexlify('12345')
b'3132333435'
>>> binascii.hexlify('12345', ' ')
b'31 32 33 34 35'
>>> binascii.hexlify('12345', ',')
b'31,32,33,34,35'
>>> binascii.hexlify('12345', '-')
b'31-32-33-34-35'
>>> binascii.hexlify(b'12345', ' ')
b'31 32 33 34 35'
>>> binascii.unhexlify('3132333435')
b'12345'
```

注:
- hexlify 函数可以指定任意一个字符作为分隔符，但是 unhexlify 函数不可指定。
