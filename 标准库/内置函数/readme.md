# 内置函数

下面是MicroPython的内置函数列表：

* abs()
* all()
* any()
* bin()
* class bool
* class bytearray
* class bytes
* callable()
* chr()
* classmethod()
* compile()
* class complex
* delattr(obj, name)
* class dict
* dir()
* divmod()
* enumerate()
* eval()
* exec()
* filter()
* class float
* class frozenset
* getattr()
* globals()
* hasattr()
* hash()
* hex()
* id()
* input()
* class int
	* classmethod from_bytes(bytes, byteorder)
	* to_bytes(size, byteorder
* isinstance()
* issubclass()
* iter()
* len()
* class list
* locals()
* map()
* max()
* class memoryview
* min()
* next()
* class object
* oct()
* open()
* ord()
* pow()
* print()
* property()
* range()
* repr()
* reversed()
* round()
* class set
* setattr()
* class slice
* sorted()
* staticmethod()
* class str
* sum()
* super()
* class tuple
* type()
* zip()

内置函数都是属于 `builtins` 模块，但可以不用导入 `builtins` 模块就能直接使用，因为系统会自动导入它。 micropython 的 `builtins` 模块和 python 中的 \_\_builtins\_\_ 模块是对应的，但是功能要少一些。

大部分的内置函数，功能、用法和CPython是已知的，因此可以通过python的帮助文档和参考书去查看这些函数的详细用法。
