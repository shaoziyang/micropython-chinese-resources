# 用 C 语言扩展 MicroPython

介绍在 MicroPython 主代码仓库之外编写代码，以 C 语言实现额外功能的几种方式。第一种方式适用于构建自定义固件，可以为固件添加特定项目所需的额外模块或函数，这些模块和函数可在 Python 层面调用。第二种方式则用于构建可在运行时加载的模块。

若需了解如何构建位于 MicroPython 主代码仓库中的核心模块，请参阅“库”章节以获取更多信息。


## MicroPython 外部 C 模块

在开发用于 MicroPython 的模块时，可能会发现自己受到 Python 环境的限制，这通常是因为无法访问某些硬件资源，或是受到 Python 速度的限制。

如果无法通过《最大化 MicroPython 速度》（Maximising MicroPython speed）中的建议解决限制，那么将模块的部分或全部代码用 C 语言（以及/或者 C++，如果移植版本支持 C++）编写，会是一个可行的方案。

如果模块旨在访问通用硬件或与通用库协同工作，建议考虑将其在 MicroPython 源代码树中与同类模块一同实现，并以拉取请求（pull request）的形式提交。但如果模块针对的是小众或专有系统，那么将其置于 MicroPython 主代码库之外，可能会是更合理的选择。

本章将介绍如何将此类外部模块编译到 MicroPython 可执行文件或固件镜像中。目前支持 Make 和 CMake 两种构建工具，因此在编写外部模块时，最好为这两种工具都添加对应的构建文件，以便该模块能在所有移植版上使用。不过，在编译特定硬件时，只需使用其中一种构建方式（要么用 Make，要么用 CMake）即可。

另一种方法是在 .mpy 文件中使用原生机器码（Native machine code）。这种方式允许编写自定义 C 代码并将其放入 .mpy 文件中，该文件可在运行中的 MicroPython 系统里动态导入，无需重新编译主固件。

### 外部 C 模块的结构

一个 MicroPython 用户 C 模块是一个包含以下文件的目录：

- 模块的 *.c / *.cpp / *.h 源代码文件。
  这些文件通常包含所实现的底层功能，以及用于暴露函数和模块的 MicroPython 绑定函数。
  目前，编写这些函数/模块的最佳参考资料是在 MicroPython 代码树中找到类似的模块，并以它们为示例。

- 包含模块 Makefile 片段的 `micropython.mk`。
  在 `micropython.mk` 中，`$(USERMOD_DIR)` 表示模块目录的路径。由于它会为每个 C 模块重新定义，因此应在 `micropython.mk` 中将其展开为一个本地 make 变量，例如：`EXAMPLE_MOD_DIR := $(USERMOD_DIR)`
  因此 `micropython.mk` 必须将模块的源文件添加到 `SRC_USERMOD_C` 或 `SRC_USERMOD_LIB_C` 变量中。前者会被处理以查找 `MP_QSTR_` 和 `MP_REGISTER_MODULE` 定义，而后者则不会（例如那些非 MicroPython 专用的辅助代码和库代码）。这些路径应包含展开后的 `$(USERMOD_DIR)` 副本，例如：
  ```make
  SRC_USERMOD_C += $(EXAMPLE_MOD_DIR)/modexample.c
  SRC_USERMOD_LIB_C += $(EXAMPLE_MOD_DIR)/utils/algorithm.c
  ```
  同样，对于 C++ 源文件，使用 `SRC_USERMOD_CXX` 和 `SRC_USERMOD_LIB_CXX`。如果要包含汇编文件，使用 `SRC_USERMOD_LIB_ASM`。
  如果有自定义的编译器选项（如 -I 用于添加头文件搜索目录），这些选项应添加到 `CFLAGS_USERMOD`（针对 C 代码）和 `CXXFLAGS_USERMOD`（针对 C++ 代码）中。

- `micropython.cmake` 包含该模块的 CMake 配置。
  在 `micropython.cmake` 中，可以使用 `${CMAKE_CURRENT_LIST_DIR}` 表示当前模块的路径。
  `micropython.cmake` 中应定义一个 `INTERFACE` 库，并将源文件、编译定义和包含目录与其关联，该库应链接到 `usermod`。
  ```cmake
  add_library(usermod_cexample INTERFACE)
  target_sources(usermod_cexample INTERFACE
  ${CMAKE_CURRENT_LIST_DIR}/examplemodule.c
  )
  target_include_directories(usermod_cexample INTERFACE
  ${CMAKE_CURRENT_LIST_DIR}
  )
  target_link_libraries(usermod INTERFACE usermod_cexample)
  ```
  完整的使用示例见下文。

### 基础示例

`cexample` 模块提供了函数和类的示例。其中，`cexample.add_ints(a, b)` 函数接收两个整数参数，将其相加后返回结果；`cexample.Timer()` 类型可创建计时器，用于测量从该对象实例化开始所经过的时间。

该模块可在 MicroPython 源代码树的 `examples`（示例）目录中找到，包含一个源文件和一个 Makefile 片段，内容如前文所述，目录结构如下：

```
micropython/
└── examples/                      # 示例目录
    └── usercmodule/               # 用户C模块目录
        └── cexample/              # cexample模块目录
            ├── examplemodule.c    # 源文件
            ├── micropython.mk     # Makefile片段
            └── micropython.cmake  # CMake配置文件
```

有关更多说明，请参考这些文件中的注释。在 `cexample` 模块旁边还有一个 `cppexample` 模块，其工作方式与 `cexample` 相同，但展示了在 MicroPython 中混合使用 C 语言和 C++ 代码的方法。


### 将C模块编译到MicroPython中

要构建此类模块，需编译MicroPython（参见[入门指南](https://github.com/micropython/micropython/wiki/Getting-Started)），并进行两项修改：
1. 设置编译时标志`USER_C_MODULES`，使其指向想要包含的模块。对于使用Make工具），该变量应指向一个目录，系统会自动在此目录中搜索模块；对于使用CMake工具，该变量应指向一个文件，该文件中包含待构建的模块。详情如下。
2. 将对应的C预处理器宏设置为 1，以启用模块。仅当要构建的模块未被自动启用时，才需要执行此操作。

若要构建MicroPython自带的示例模块，对于Make工具，需将`USER_C_MODULES`设置为`examples/usercmodule`目录；对于CMake工具，需将其设置为`examples/usercmodule/micropython.cmake`文件。

例如，以下是构建包含示例模块的 unix 版本的方法：
```bash
cd micropython/ports/unix
make USER_C_MODULES=../../examples/usercmodule
```

当在构建中加入新的用户模块时，可能需要先执行一次`make clean`。构建输出会显示找到的模块：
```make
...
Including User C Module from ../../examples/usercmodule/cexample
Including User C Module from ../../examples/usercmodule/cppexample
...
```

对于基于CMake的构建（如rp2），操作会略有不同（注意：CMake实际上是通过make调用的）：
```bash
cd micropython/ports/rp2
make USER_C_MODULES=../../examples/usercmodule/micropython.cmake
```

同样，为了让CMake识别到用户模块，可能需要先执行`make clean`。CMake的构建输出会按名称列出模块：
```cmake
...
Including User C Module(s) from ../../examples/usercmodule/micropython.cmake
Found User C Module(s): usermod_cexample, usermod_cppexample
...
```

顶层`micropython.cmake`文件的内容可用于控制哪些模块被启用。

在您自己的项目中，将自定义代码放在MicroPython主源代码树之外会更方便，因此典型的项目目录结构如下：

```
my_project/
├── modules/  # 模块目录
│   ├── example1/  # 示例1目录
│   │   ├── example1.c  # 示例1的C源文件
│   │   ├── micropython.mk  # 示例1的Make配置文件（用于Make构建系统）
│   │   └── micropython.cmake  # 示例1的CMake配置文件（用于CMake构建系统）
│   ├── example2/  # 示例2目录
│   │   ├── example2.c  # 示例2的C源文件
│   │   ├── micropython.mk  # 示例2的Make配置文件
│   │   └── micropython.cmake  # 示例2的CMake配置文件
│   └── micropython.cmake  # 顶层CMake配置文件（用于统筹管理所有模块）
└── micropython/  # MicroPython源代码根目录
    ├── ports/  # MicroPython端口目录（不同硬件/平台的适配代码）
    ... ├── stm32/  # STM32平台的端口目录
        ...
```

使用Make构建时，需将`USER_C_MODULES`设置为`my_project/modules`目录。例如，构建STM32端口的命令如下：
```bash
cd my_project/micropython/ports/stm32
make USER_C_MODULES=../../../modules
```

使用CMake构建时，位于`my_project/modules`目录下的顶层`micropython.cmake`文件，需包含所有希望启用的模块配置，示例内容如下：
```cmake
include(${CMAKE_CURRENT_LIST_DIR}/example1/micropython.cmake)  # 包含示例1的模块配置
include(${CMAKE_CURRENT_LIST_DIR}/example2/micropython.cmake)  # 包含示例2的模块配置
```

之后执行以下命令进行构建（以ESP32为例）：
```bash
cd my_project/micropython/ports/esp32
make USER_C_MODULES=../../../../modules/micropython.cmake
```

**注意**：由于ESP32的主`CMakeLists.txt`文件路径特殊，其相对路径需要多添加一个`..`。也可以为`USER_C_MODULES`指定绝对路径（避免相对路径出错）。

`USER_C_MODULES`变量指定的所有模块（使用Make时，会自动搜索该目录下的模块；使用CMake时，通过`include`命令添加的模块）都会被编译，但只有**已启用**的模块才能在MicroPython中导入使用。用户模块通常默认启用（是否默认启用由模块开发者决定），这种情况下，只需按照上述步骤设置`USER_C_MODULES`即可，无需额外操作。

若某个模块**未默认启用**，则必须手动启用对应的C预处理器宏。该宏的名称可通过搜索模块源代码中的`MP_REGISTER_MODULE`语句找到（该语句通常位于主源文件末尾）。此宏通常会被`#if X / #endif`代码块包裹，需通过`CFLAGS_EXTRA`将配置项`X`设为1，才能使模块可用；若不存在`#if X / #endif`代码块，则说明该模块默认启用。

例如，`examples/usercmodule/cexample`模块默认启用，因此其源代码中有如下语句：
```c
MP_REGISTER_MODULE(MP_QSTR_cexample, example_user_cmodule);  // 注册模块，默认启用
```

若要将该模块改为“默认禁用、可通过预处理器配置启用”，则代码需修改为：
```c
#if MODULE_CEXAMPLE_ENABLED  // 仅当MODULE_CEXAMPLE_ENABLED为1时，才注册模块
MP_REGISTER_MODULE(MP_QSTR_cexample, example_user_cmodule);
#endif
```

这种情况下，启用模块的方法是在 `make` 命令中添加 ` CFLAGS_EXTRA=-DMODULE_CEXAMPLE_ENABLED=1` 或编辑硬件配置文件（ `mpconfigport.h` ）或开发板配置文件（ `mpconfigboard.h` ），添加以下宏定义：
```c
#define MODULE_CEXAMPLE_ENABLED (1)
```

**注意**：具体启用方法因硬件而异（不同硬件的目录结构不同）。若配置不当，模块虽能正常编译，但在MicroPython中导入时会提示“找不到模块”。

### 在MicroPython中使用模块

一旦模块被编译到 MicroPython 副本中，就可以在 Python 代码里像使用其他任何内置模块一样调用它，例如：

```python
import cexample            # 导入自定义的cexample模块
print(cexample.add_ints(1, 3))
# 输出结果应为4
```

```python
from cexample import Timer # 从cexample模块中导入Timer类
from time import sleep_ms

watch = Timer()
sleep_ms(1000)
print(watch.time())        # 调用watch实例的time方法，获取从实例创建到现在的时间
# 输出结果约为1000
```

## 包含原生机器码的 .mpy 文件

本节将介绍如何构建和使用包含非 Python 语言原生机器码的 .mpy 文件。通过这种方式，可以使用 C 等语言编写代码，将其编译并链接为 .mpy 文件，之后像导入普通 Python 模块一样导入该文件。这种方法可用于实现对性能要求较高的功能，或集成其他语言编写的现有库。

使用原生 .mpy 文件的主要优势之一是：原生机器码可由脚本动态导入，无需重新构建 MicroPython 主固件。这与 MicroPython 外部 C 模块形成对比 —— 外部 C 模块虽也支持用 C 语言定义自定义模块，但必须编译到主固件镜像中。

本文档的重点是使用 C 语言构建原生模块，但理论上，任何可编译为独立机器码的语言，其代码都可封装到 .mpy 文件中。

原生 .mpy 模块需使用 `mpy_ld.py` 工具构建，该工具位于项目的 `tools/` 目录下。此工具接收一组目标文件（.o 文件），将它们链接在一起，生成原生 .mpy 文件。使用该工具需要安装 CPython 3，以及版本不低于 0.25 的 `pyelftools` 库。

### 支持的特性与限制

一个 .mpy 文件可包含 MicroPython 字节码和/或原生机器码。若文件中包含原生机器码，则该 .mpy 文件会关联特定的架构。目前支持的架构如下（这些是 `ARCH` 变量的有效选项，详见下文）：
- x86（32位）
- x64（64位 x86）
- armv6m（ARM Thumb 指令集，如 Cortex-M0 处理器）
- armv7m（ARM Thumb 2 指令集，如 Cortex-M3 处理器）
- armv7emsp（ARM Thumb 2 指令集，单精度浮点，如 Cortex-M4F、Cortex-M7 处理器）
- armv7emdp（ARM Thumb 2 指令集，双精度浮点，如 Cortex-M7 处理器）
- xtensa（非窗口模式，如 ESP8266 芯片）
- xtensawin（窗口模式，窗口大小为 8，如 ESP32、ESP32S3 芯片）
- rv32imc（RISC-V 32位，含压缩指令，如 ESP32C3、ESP32C6 芯片）

在编译和链接原生 .mpy 文件时，必须选择对应的架构，且生成的文件仅能在该架构上导入使用。关于 .mpy 文件的更多细节，可参考《MicroPython .mpy 文件》文档。

原生代码必须编译为**位置无关代码**（PIC，Position Independent Code）并使用**全局偏移表**（GOT，Global Offset Table），不过具体实现细节因架构而异。当导入包含原生代码的 .mpy 文件时，导入机制可对原生代码执行一些基础的重定位操作，包括对代码段（text）、只读数据段（rodata）和未初始化数据段（BSS）的重定位。

链接器与动态加载器支持的特性
- 可执行代码（代码段 text）
- 只读数据（只读数据段 rodata），包括字符串和常量数据（数组、结构体等）
- 零初始化数据（未初始化数据段 BSS）
- 代码段中指向代码段、只读数据段和 BSS 段的指针
- 只读数据段中指向代码段、只读数据段和 BSS 段的指针

已知限制
- **不支持数据段（data section）**
  解决方法：使用 BSS 段数据，并显式初始化数据值。
- **不支持静态 BSS 变量（static BSS variables）**
  解决方法：使用全局 BSS 变量。
- **rv32imc 架构不支持线程本地存储变量（thread-local storage variables）**
  解决方法：使用全局 BSS 变量，或在堆（heap）上分配空间存储此类变量。

因此，如果 C 代码包含可写数据，需确保数据定义为全局变量（无初始化值），且仅在函数内部对其进行写操作。

原生模块不会自动链接 `libm.a`、`libgcc.a` 等标准静态库，这可能导致“未定义符号”错误。可以在 Makefile 中设置 `LINK_RUNTIME = 1`，以此链接运行时库。对于自定义静态库，也可通过添加 `MPY_LD_FLAGS += -l path/to/library.a`（路径需替换为实际库文件路径）来实现链接。需注意，这些库会被链接到原生模块内部，无法与其他模块或系统共享。

**链接器限制**：原生模块不会链接完整 MicroPython 固件的符号表，而是链接 `mp_fun_table`（位于 `py/nativeglue.h` 文件中）中的显式导出符号表——该表在固件编译时就已固定。因此，例如像 HAL（硬件抽象层）、OS（操作系统）、RTOS（实时操作系统）或系统的任意函数，都无法直接调用，除非这些函数位于固定地址。如需调用固定地址的函数，可通过 `--externs` 命令行参数，将包含一系列符号名及其固定地址的链接脚本（linkerscript）路径传递给 `mpy_ld.py` 工具。这样链接脚本中定义的符号会优先于目标文件（.o 文件）提供的符号，但目前目标文件中的实现代码仍会保留在最终生成的 .mpy 文件中。需要说明的是，链接脚本解析器的功能存在限制，目前仅用于解析 ESP8266 的 ROM 符号列表（可参考 `ports/esp8266/boards/eagle.rom.addr.v6.ld` 文件）。

可在该表的末尾添加新符号，并重新编译固件。同时，还需在 `tools/mpy_ld.py` 文件的 `fun_table` 字典中对应位置添加这些符号。这样一来，`mpy_ld.py` 工具就能识别到这些新符号，并在导入 .mpy 文件时为它们执行重定位操作。最后，若该符号对应一个函数，需在 `py/dynruntime.h` 文件中添加相应的宏或存根（stub），以便更便捷地调用该函数。


### 定义原生模块

原生 .mpy 模块由一组用于构建 .mpy 文件的文件定义。其文件系统结构包含两个主要部分：源文件和 Makefile（编译配置文件），具体说明如下：

- 在最简单的场景中，仅需一个 C 源文件即可，该文件包含所有将被编译到 .mpy 模块中的代码。此 C 源文件必须包含 `py/dynruntime.h` 文件（以访问 MicroPython 动态 API），且至少需定义一个名为 `mpy_init` 的函数。该函数是模块的入口点，会在模块被导入时调用。
  
  在需要时，模块可拆分为多个 C 源文件；模块的部分功能也可通过 Python 实现。所有源文件（包括 C 源文件以及将被包含到最终 .mpy 文件中的所有 Python 文件）均需在 Makefile 中列出，具体方式是将它们添加到 `SRC` 变量中（详见下文）。

- Makefile 包含模块的构建配置，并列出用于构建 .mpy 模块的源文件。在 Makefile 中，需定义 `MPY_DIR`，指定 MicroPython 代码仓库的路径（用于查找头文件、相关的 Makefile 片段以及 `mpy_ld.py` 工具）；定义 `MOD`，指定模块的名称；定义 `SRC`，指定源文件列表；通过 `ARCH` 变量指定机器架构（可选）；最后包含 `py/dynruntime.mk` 文件。

### 最小示例

本节提供一个可完整运行的简单模块示例，模块名为 `factorial`。该模块仅提供一个函数 `factorial.factorial(x)`，用于计算输入值的阶乘并返回结果。

**目录结构**
```
factorial/          # 模块根目录
├── factorial.c     # C语言源文件（实现阶乘计算逻辑）
└── Makefile        # 构建配置文件（定义模块编译规则）
```


**factorial.c 文件内容**
```c
// 包含头文件以访问 MicroPython API
#include "py/dynruntime.h"

// 阶乘计算辅助函数
static mp_int_t factorial_helper(mp_int_t x) {
    if (x == 0) {
        return 1;  // 0的阶乘为1（递归终止条件）
    }
    return x * factorial_helper(x - 1);  // 递归计算阶乘
}

// 此函数将在Python中被调用，对应 factorial(x)
static mp_obj_t factorial(mp_obj_t x_obj) {
    // 从MicroPython输入对象中提取整数
    mp_int_t x = mp_obj_get_int(x_obj);
    // 计算阶乘
    mp_int_t result = factorial_helper(x);
    // 将结果转换为MicroPython整数对象并返回
    return mp_obj_new_int(result);
}

// 定义Python层面对上述函数的引用
static MP_DEFINE_CONST_FUN_OBJ_1(factorial_obj, factorial);

// 模块入口函数，在模块被导入时调用
mp_obj_t mpy_init(mp_obj_fun_bc_t *self, size_t n_args, size_t n_kw, mp_obj_t *args) {
    // 必须放在开头，用于初始化全局字典及其他必要资源
    MP_DYNRUNTIME_INIT_ENTRY
    
    // 将函数添加到模块命名空间，使其在Python中可访问
    mp_store_global(MP_QSTR_factorial, MP_OBJ_FROM_PTR(&factorial_obj));
    
    // 必须放在结尾，用于恢复全局字典
    MP_DYNRUNTIME_INIT_EXIT
}
```

**Makefile 文件内容**
```makefile
# 顶层MicroPython目录的路径（根据实际目录结构调整）
MPY_DIR = ../../..

# 模块名称（需与Python中导入的模块名一致）
MOD = factorial

# 源文件列表（支持.c文件或.py文件）
SRC = factorial.c

# 目标构建架构（可选值：x86、x64、armv6m、armv7m、xtensa、xtensawin、rv32imc）
ARCH = x64

# 引入编译和链接模块的通用规则（来自MicroPython源码）
include $(MPY_DIR)/py/dynruntime.mk
```


### 编译模块

构建原生 .mpy 文件所需的前置工具如下：
- MicroPython 代码仓库（至少包含 py/ 和 tools/ 目录）。
- CPython 3，以及 pyelftools 库（例如通过命令 `pip install 'pyelftools>=0.25'` 安装）。
- GNU make 工具。
- 适用于目标架构的 C 编译器（若使用 C 源文件）。
- （可选）从 MicroPython 代码仓库构建的 mpy-cross 工具（使用 .py 源文件）。

请务必为待运行的目标平台选择正确的 `ARCH`（架构）。之后执行以下命令进行构建：
```bash
$ make
```

如果不修改 Makefile，也可通过以下命令指定目标架构：
```bash
$ make ARCH=armv7m
```


### 在 MicroPython 中使用该模块

模块构建完成后，会生成一个名为 `factorial.mpy` 的文件。将该文件复制到你的 MicroPython 系统文件系统中，且确保其位于导入路径（import path）下。此后，就可以像使用其他任何模块一样，在 Python 中访问该模块，示例如下：

```python
import factorial                # 导入模块
print(factorial.factorial(10))  # 调用模块中的阶乘函数，计算10的阶乘
# 输出结果应为 3628800
```


### 构建模块时使用 Picolibc

将 Picolibc 用作 C 标准库不仅受支持，事实上，它还是 rv32imc 平台的默认标准库。不过，有几点值得注意，以确保构建代码时不会遇到问题。

部分预编译的 Picolibc 版本（例如，Ubuntu Linux 提供的 `picolibc-arm-none-eabi`、`picolibc-riscv64-unknown-elf` 和 `picolibc-xtensa-lx106-elf` 软件包）假设运行时存在线程本地存储（TLS，Thread-Local Storage），但遗憾的是，MicroPython 模块在部分架构（即 rv32imc）上并不支持 TLS。这意味着 Picolibc 提供的部分功能会默认使用 TLS，进而在编译或链接过程中返回错误。

关于这一点可能产生的影响，可参考示例 `examples/natmod/btree`，该示例模块中包含一个规避方案，以确保 `errno` 能正常工作（可在其 Makefile 中查找 `__PICOLIBC_ERRNO_FUNCTION`，并顺着相关配置追踪具体实现）。


### 更多示例

更多示例可参考 `examples/natmod/` 目录，这些示例展示了原生 .mpy 模块的众多可用特性。此类特性包括：
- 使用多个 C 源文件
- 在 C 代码中搭配 Python 代码
- 只读数据（rodata）和未初始化数据（BSS）的使用
- 内存分配
- 浮点数的使用
- 异常处理
- 引入外部 C 库
