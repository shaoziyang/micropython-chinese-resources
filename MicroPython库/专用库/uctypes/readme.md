
# uctypes（以结构化方式访问二进制数据）

uctypes 为 MicroPython 实现了"外部数据接口"。其设计理念与 CPython 的 `ctypes` 模块类似，但实际 API 有所不同，经过精简和优化以适配小体积场景。模块的核心思想是通过与 C 语言相当的能力定义数据结构布局，然后使用熟悉的点语法访问子字段。  

⚠️ 警告  
`uctypes` 模块允许访问机器的任意内存地址（包括I/O和控制寄存器）。不当使用可能导致系统崩溃、数据丢失甚至硬件故障。  

另请参阅 `struct` 模块，这是访问二进制数据结构的标准 Python 方式（对大型复杂结构支持不佳）。

使用示例：  
```python  
import uctypes

# 示例1：ELF文件头子集  
# https://wikipedia.org/wiki/Executable_and_Linkable_Format#File_header  
ELF_HEADER = {  
    "EI_MAG": (0x0 | uctypes.ARRAY, 4 | uctypes.UINT8),  
    "EI_DATA": 0x5 | uctypes.UINT8,  
    "e_machine": 0x12 | uctypes.UINT16,  
}  

# "f" 是以二进制模式打开的ELF文件  
buf = f.read(uctypes.sizeof(ELF_HEADER, uctypes.LITTLE_ENDIAN))  
header = uctypes.struct(uctypes.addressof(buf), ELF_HEADER, uctypes.LITTLE_ENDIAN)  
assert header.EI_MAG == b"\x7fELF"  
assert header.EI_DATA == 1, "Oops, 字节序错误。可尝试使用uctypes.BIG_ENDIAN重试。"  
print("machine:", hex(header.e_machine))  

# 示例2：带指针的内存数据结构  
COORD = {  
    "x": 0 | uctypes.FLOAT32,  
    "y": 4 | uctypes.FLOAT32,  
}

STRUCT1 = {  
    "data1": 0 | uctypes.UINT8,  
    "data2": 4 | uctypes.UINT32,  
    "ptr": (8 | uctypes.PTR, COORD),  
}  

# 假设 "addr" 中存放 STRUCT1 类型结构的地址  
# uctypes.NATIVE 为可选参数（默认使用）  
struct1 = uctypes.struct(addr, STRUCT1, uctypes.NATIVE)  
print("x:", struct1.ptr[0].x)  

# 示例3：访问 CPU 寄存器（STM32F4xx WWDG模块子集）  
WWDG_LAYOUT = {  
    "WWDG_CR": (0, {  
        # 此处BFUINT32表示WWDG_CR寄存器的大小  
        "WDGA": 7 << uctypes.BF_POS | 1 << uctypes.BF_LEN | uctypes.BFUINT32,  
        "T": 0 << uctypes.BF_POS | 7 << uctypes.BF_LEN | uctypes.BFUINT32,  
    }),  
    "WWDG_CFR": (4, {  
        "EWI": 9 << uctypes.BF_POS | 1 << uctypes.BF_LEN | uctypes.BFUINT32,  
        "WDGTB": 7 << uctypes.BF_POS | 2 << uctypes.BF_LEN | uctypes.BFUINT32,  
        "W": 0 << uctypes.BF_POS | 7 << uctypes.BF_LEN | uctypes.BFUINT32,  
    }),  
}  

WWDG = uctypes.struct(0x40002c00, WWDG_LAYOUT)  
WWDG.WWDG_CFR.WDGTB = 0b10  
WWDG.WWDG_CR.WDGA = 1  
print("Current counter:", WWDG.WWDG_CR.T)  
```  

### 定义结构布局  

结构布局由“描述符”定义——一个Python字典，键为字段名，值为访问所需的属性：  
```python  
{  
    "field1": <属性>,  
    "field2": <属性>,  
    ...  
}  
```  
当前，`uctypes` 要求为每个字段显式指定偏移量（以字节为单位，从结构起始位置开始计算）。  

以下是不同字段类型的编码示例：  
- **标量类型**：
  ```python
  "field_name": offset | uctypes.UINT32`  
  ```
  即结果为标量类型标识符与字段偏移量（字节）的按位或结果。
<br><br>

- **递归结构**：  
  ```python  
  "sub": (offset, {  
      "b0": 0 | uctypes.UINT8,  
      "b1": 1 | uctypes.UINT8,  
  })  
  ```  
  值为二元组，第一个元素为偏移量，第二个为结构描述符字典（注：递归描述符中的偏移量相对于当前结构）。递归结构可通过名称引用已定义的描述符字典，而非仅限字面量字典。
<br><br>

- **基本类型数组**：  
  ```python
  "arr": (offset | uctypes.ARRAY, size | uctypes.UINT8),
  ```  
  值为二元组，第一个元素为ARRAY标志与偏移量的按位或结果，第二个为标量元素类型与数组元素数量的按位或结果。
  <br><br>

- **聚合类型数组**：  
  ```python  
  "arr2": (offset | uctypes.ARRAY, size, {"b": 0 | uctypes.UINT8}),  
  ```  
  值为三元组，第一个元素为 ARRAY 标志与偏移量的按位或结果，第二个为元素数量，第三个为元素类型描述符。
  <br><br>

- **指向基本类型的指针**：  
  ```python
  "ptr": (offset | uctypes.PTR, uctypes.UINT8),
  ```  
  值为二元组，第一个元素为 PTR 标志与偏移量的按位或结果，第二个为标量元素类型。
  <br><br>

- **指向聚合类型的指针**：  
  ```python  
  "ptr2": (offset | uctypes.PTR, {"b": 0 | uctypes.UINT8}),  
  ```  
  值为二元组，第一个元素为 PTR 标志与偏移量的按位或结果，第二个为指向类型的描述符。
  <br><br>

- **位字段**：  
  ```python  
  "bitf0": offset | uctypes.BFUINT16 | lsbit << uctypes.BF_POS | bitsize << uctypes.BF_LEN,  
  ```  
  值包含以下部分的按位或结果：  
  - 包含位字段的标量类型（类型名以BF开头，类似标量类型）  
  - 标量值偏移量  
  - 位字段在标量中的起始位位置（左移BF_POS位）  
  - 位字段长度（左移BF_LEN位）  

  位字段位置从标量的最低有效位（位置0）开始计数，表示字段最右侧位的位置（即提取位字段时标量需右移的位数）。  
  例如，若 `lsbit=0` 且 `bitsize=8`，则实际访问 UINT16 的最低有效字节。  
  注意：位字段操作与目标字节序无关（如上述示例在小端和大端结构中均访问 UINT16 的最低有效字节），仅依赖最低有效位为0的编号规则。尽管某些目标平台的原生 ABI 可能使用不同编号，但 `uctypes` 始终采用上述标准化编号。  

### 模块内容  

- class uctypes.`struct`(addr, descriptor, layout_type=NATIVE, / )  
  根据内存中的结构地址、描述符（字典形式）和布局类型（见下文）实例化"外部数据结构"对象。
<br><br>

- uctypes.`LITTLE_ENDIAN`  
  小端序紧凑结构的布局类型（紧凑表示每个字段严格占用描述符定义的字节数，对齐方式为1）。
<br><br>

- uctypes.`BIG_ENDIAN`
  大端序紧凑结构的布局类型。
<br><br>

- uctypes.`NATIVE`  
  原生结构的布局类型，数据字节序和对齐方式符合 MicroPython 运行系统的 ABI。
<br><br>

- uctypes.`sizeof`(struct, layout_type=NATIVE, /  )   
  返回数据结构的字节大小。`struct`参数可以是结构类、具体实例化的结构对象（或其聚合字段）。
<br><br>

- uctypes.`addressof`(obj)  
  返回对象的地址。参数应为bytes、bytearray或其他支持缓冲区协议的对象（实际返回缓冲区的地址）。
<br><br>

- uctypes.`bytes_at`(addr, size)  
  将给定地址和大小的内存捕获为 bytes 对象。由于 bytes 不可变，内存的内容会被复制，后续内存变化不影响已创建的对象。
<br><br>

- uctypes.`bytearray_at`(addr, size)  
  将给定地址和大小的内存捕获为 bytearray 对象。与 `bytes_at()` 不同，此方法通过引用捕获内存，支持写入操作，直接访问当前内存地址的值。
<br><br>

- uctypes.`UINT8`
- uctypes.`INT8`
- uctypes.`UINT16`
- uctypes.`INT16`  
- uctypes.`UINT32`
- uctypes.`INT32`  
- uctypes.`UINT64`
- uctypes.`INT64`  
  整数类型（用于结构描述符），包含 8、16、32 和 64 位无符号和有符号整数。
<br><br>

- uctypes.FLOAT32
- uctypes.FLOAT64  
  浮点类型（用于结构描述符）。
<br><br>

- uctypes.`VOID`  
  是 UINT8 的别名，用于方便定义 C 语言的 void 指针：`(uctypes.PTR, uctypes.VOID)`。
<br><br>

- uctypes.`PTR`
- uctypes.`ARRAY`  
  指针和数组的类型常量。注意：结构类型无显式常量，隐含规则为：不带 PTR 或 ARRAY 标志的聚合类型即为结构。


### 结构描述符与实例化结构对象

给定一个结构描述符字典及其布局类型，可使用 `uctypes.struct()` 构造函数在指定内存地址实例化具体的结构对象。内存地址通常来源于以下途径：

- 预定义地址：在裸机系统中访问硬件寄存器时使用。这类地址需从特定 MCU/SoC 的数据手册中查询。
- FFI（外部函数接口）函数的返回值。
- 通过 `uctypes.addressof()` 获取：用于向 FFI 函数传递参数，或访问 I/O 数据（如从文件或网络套接字读取的数据）。


### 结构对象

结构对象支持使用标准点语法访问单个字段，例如：`my_struct.substruct1.field1`。

- 如果字段为标量类型，获取其值将返回对应字段的原始值（Python 整数或浮点数），标量字段也支持赋值操作。
- 如果若字段为数组，可使用标准下标运算符 `[]` 读写单个元素。
- 如果字段为指针，可使用 `[0]` 语法引用（类似 C 语言的*运算符，在 C 语言中 `[0]` 同样有效）。指针也支持使用非 0 整数下标，语义与 C 语言一致。

总结来看，结构字段的访问方式总体遵循 C 语言语法，唯一区别是指针引用需使用 `[0]` 而非 `*`。


### 局限性

1. 访问非标量字段会分配中间对象：这意味着在内存分配被禁用的场景（如中断处理）中访问结构时需特别注意。建议如下：

    - 避免访问嵌套结构：例如，避免使用 `mcu_registers.peripheral_a.register1`，可为每个外设定义独立的布局描述符，直接通过 `peripheral_a.register1` 访问；或缓存特定外设对象，如 `peripheral_a = mcu_registers.peripheral_a`。若寄存器包含多个位字段，需缓存寄存器引用，如 `reg_a = mcu_registers.peripheral_a.reg_a`。

    - 避免使用其他非标量数据（如数组）：例如，避免使用 `peripheral_a.register[0]`，可改为 `peripheral_a.register0`。另一种方式是缓存中间值，如 `register0 = peripheral_a.register[0]`。

2. `uctypes` 模块支持的偏移量范围有限：具体支持范围属于实现细节，一般建议将结构定义的范围控制在几 KB 到几十 KB 以内。在大多数场景下，这是合理的（例如，将 MCU 的所有寄存器定义在一个结构中并无意义，应按外设模块拆分）。在极端情况下（如访问包含数 MB 数组的原生数据结构），可能需要人为将结构拆分为多个部分（尽管这种场景非常少见）。

