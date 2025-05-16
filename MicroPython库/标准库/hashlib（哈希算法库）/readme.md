# hashlib（哈希算法库）

这个模块实现二进制数据的哈希算法，可以使用的算法与具体硬件有关，目前包括了 SHA256、SHA1、MD5等算法。选择 SHA256 是经过仔细考虑的，因为它是流行的、安全的算法。这意味着一个单一的算法可以涵盖“任何哈希算法”以及和安全相关的应用。

* SHA256，现代哈希算法（SHA2系列）。它用于加密安全，包含在 MicroPython 内核中，任何开发板都推荐使用，除非有特定的代码大小限制。
* SHA1，上一代算法。不推荐用于新应用，但 SHA1 是许多互联网标准和现有应用程序的一部分，因此针对网络连接和交互操作性的开发板将尝试提供这一功能。
* MD5，一种传统算法，现在不再认为是安全的。只有针对与旧应用程序操作的特定板卡提供此功能。


## 构造函数

哈希算法构造函数：

* class hashlib.`sha256`([data])

  创建一个 SHA256 散列对象，并可选择将数据填充到其中。
<br><br>

* class hashlib.`sha1`([data])

  创建一个 SHA1 散列对象。
<br><br>

* class hashlib.`md5`([data])

  创建一个 MD5 散列对象。



## 方法

哈希算法相关方法：

* hash.`update`(data)

  填充数据。
<br><br>

* hash.`digest`()

  返回被散列的数据，结果是字节对象。调用这个方法后，不能再写入数据。
<br><br>

* hash.`hexdigest`()

  此方法未实现。使用 `binascii.hexlify(hash.digest())` 来实现类似的功能。
  

## 使用方法

  哈希算法模块基本使用方法如下：

```py
>>> import hashlib
>>> hash = hashlib.sha256()
>>> buf = b'123456789'
>>> hash.update(buf)
>>> hash.digest()
b'\x15\xe2\xb0\xd3\xc38\x91\xeb\xb0\xf1\xef`\x9e\xc4\x19B\x0c \xe3 \xce\x94\xc6_\xbc\x8c3\x12D\x8e\xb2%'
```
