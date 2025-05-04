# binascii（binary/ASCII 转换）

这个模块实现二进制数据和ASCII形式之间各种编码的转换（双向）。

**函数**

* binascii.`hexlify`(data[, sep])

  转换二进制数据为16进制字符串。如：
  
  ```py
  binascii.hexlify(b'\x11\x22123')
  b'1122313233'
  ```
  
  
  如果指定了第二个参数，它将用于分隔两个HEX参数，如：


  ```py
  binascii.hexlify(b'\x11\x22123',' ')
  b'11 22 31 32 33'
  binascii.hexlify(b'\x11\x22123',',')
  b'11,22,31,32,33'
  ```

  如果sep设定了多个字符，只有第一个字符是有效的。  
<br>

* binascii.`unhexlify`(data)

  转换HEX数据为二进制字符串，功能和hexlify相反。

  ```py
  binascii.unhexlify('313233')
  b'123'
  ```


* binascii.`a2b_base64`(data)

  转换 Base64 编码数据为二进制字符串。  
<br>
  
* binascii.`b2a_base64`(data, *, newline=True)

  将二进制数据编码为 Base64 格式。  
<br>
  
* binascii.`crc32`(data [ , value ] )

  使用硬件功能快速计算crc32校验。value 是初始值，默认是 0。该算法与ZIP文件校验和一致。
