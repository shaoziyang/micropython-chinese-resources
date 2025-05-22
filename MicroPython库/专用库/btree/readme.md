# btree（简单 B 树数据库）

btree模块使用外部存储（磁盘文件，或一般情况下的随机访问流）实现了一个简单的键值数据库。键在数据库中按排序存储，除了通过键值高效检索外，数据库还支持高效的有序范围扫描（检索键在给定范围内的值）。在应用程序接口方面，B树数据库的工作方式尽可能接近标准字典类型，一个显著区别是键和值都必须是类似字节的对象（因此，如果需要存储其他类型的对象，需要先将其序列化为str、bytes或其他支持缓冲区协议的类型）。  

该模块基于著名的BerkelyDB库1.xx版本。  

**示例**：  
```py
import btree  
# 首先，需要打开一个存储数据库的流  
# 通常是文件，但也可以是使用io.BytesIO的内存数据库、原始闪存分区等  
# 通常需要在数据库文件不存在时创建，存在时打开。以下习惯用法可处理此情况  
# 请勿使用"a+b"访问模式打开数据库  
try:  
    f = open("mydb", "r+b")  
except OSError:  
    f = open("mydb", "w+b")  
    
# 现在打开数据库本身  
db = btree.open(f)  

# 添加的键将在数据库内部排序  
db[b"3"] = b"three"  
db[b"1"] = b"one"  
db[b"2"] = b"two"  

# 除非显式刷新（或关闭数据库），否则所有更改将缓存在内存中。每次"事务"结束时刷新数据库  
db.flush()  

# 输出：b'two'  
print(db[b"2"])  

# 遍历数据库中从b"2"开始到末尾的排序键，仅返回值  
# 注意：传递给values()方法的参数是*键*值  
# 输出：  
# b'two'  
# b'three'  
for word in db.values(b"2"):  
    print(word)  

del db[b"2"]  

# 不再存在，输出False  
print(b"2" in db)  

# 输出：  
# b"1"  
# b"3"  
for key in db:  
    print(key)  
db.close()  

# 不要忘记关闭底层流！  
f.close()  
```


### 函数  

- btree.`open`(stream, *, flags=0, pagesize=0, cachesize=0, minkeypage=0)  

  从随机访问流（如打开的文件）中打开数据库。所有其他参数均为可选，用于调整数据库操作的高级参数（大多数用户无需使用）：  
  - `flag` - 当前未使用。  
  - `pagesize` - B树节点使用的页大小，取值范围为512-65536。若为0，则使用特定于移植版本的默认值，该值针对内存使用和/或性能进行了优化。  
  - `cachesize` - 建议的内存缓存大小（以字节为单位）。对于内存充足的开发板，使用更大的值可能提升性能。缓存策略如下：不会一次性分配整个缓存，而是在访问数据库中的新页时为其分配内存缓冲区，直至达到 `cachesize` 指定的值。之后，这些缓冲区将使用 LRU（最近最少使用）策略进行管理。如有需要（例如，数据库包含较大的键和/或值时），仍可能分配更多缓冲区。已分配的缓存缓冲区不会回收。  
  - `minkeypage` - 每页存储的最小键数。默认值0等效于2。  

  返回一个 BTree 对象，该对象实现了字典协议（一组方法）以及下文描述的其他方法。  

### 方法  

- btree.`close`()  

  关闭数据库。处理结束后必须关闭数据库，因为某些未写入的数据可能仍存在于缓存中。**注意**：这不会关闭打开数据库时使用的底层流，底层流应单独关闭（这也是确保数据从缓冲区刷新到底层存储的必要操作）。  

- btree.`flush`()  

  将缓存中的所有数据刷新到底层流。  

- btree.`__getitem__`(key)  
- btree.`get`(key, default=None, / )  
- btree.`__setitem__`(key, val)  
- btree.`__delitem__`(key)  
- btree.`__contains__`(key)  
  标准字典方法  

- btree.`__iter__`()  

  BTree对象可通过直接迭代（类似字典）方式按顺序访问所有键。  

- btree.`keys`([start_key [, end_key [, flags ]]] )  
- btree.`values`([start_key [, end_key [, flags ]]] )  
- btree.`items`([start_key [, end_key [, flags ]]] )  

  这些方法类似于标准字典方法，但还可以接受可选参数以迭代键的子范围，而非整个数据库。**注意**：对于所有三个方法，`start_key` 和 `end_key` 参数均表示键值。例如，`values()` 方法将迭代与给定键范围对应的 值。`start_key` 为 `None` 表示"从第一个键开始"，`end_key` 为 `None` 或未指定表示"到数据库末尾"。  

  默认情况下，范围包含 `start_key` 且不包含 `end_key`。若需在迭代中包含 `end_key`，可传递标志 `btree.INCL`；若需按键的降序迭代，可传递标志 `btree.DESC`。标志值可按位或组合使用。  

### 常量  

- btree.`INCL`  

  用于`keys()`、`values()`、`items()`方法的标志，指定扫描应包含结束键。  

- btree.`DESC`  

  用于`keys()`、`values()`、`items()`方法的标志，指定扫描按照键的降序方向进行。
  
