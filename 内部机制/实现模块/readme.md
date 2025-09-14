# 实现模块

本章详细介绍如何在 MicroPython 中实现一个核心模块。MicroPython 模块可分为以下几类：
- **内置模块（Built-in module）**：作为 MicroPython 代码仓库一部分的通用模块。
- **用户模块（User module）**：适用于特定项目的模块，通常维护在你自己的代码仓库或私有代码库中。
- **动态模块（Dynamic module）**：可在运行时部署到设备并导入使用的模块。

在 MicroPython 中，模块可在以下位置之一实现：
- `py/`：对应 CPython 核心功能的核心库。
- `extmod/`：适用于跨多个端口共用的 CPython 兼容模块或 MicroPython 专属模块。
- `ports/<port>/`：针对特定端口（如 stm32、unix 等）的专属模块。

**注意**  
- 本章介绍的是在 py/ 目录下实现的模块或核心模块。有关外部模块的实现细节，请参考“使用 C 语言扩展 MicroPython”（Extending MicroPython in C）；有关特定硬件模块的实现细节，请参考“移植 MicroPython（Porting MicroPython）”。


# 实现核心模块

与 CPython 类似，MicroPython 也包含核心内置模块，可通过 `import` 语句导入使用。例如在“内存管理”章节中讨论过的 `gc` 模块（垃圾回收模块）：
```python
>>> import gc
>>> gc.enable()
>>>
```

MicroPython 还拥有其他多个内置标准/核心模块，如 `io`（输入输出模块）、`array`（数组模块）等。新增一个核心模块需进行多项修改，具体步骤如下：

首先，在 `py/` 目录下创建对应的 C 语言文件。本示例中，我们将在 `modsubsystem.c` 文件中新增一个假设的 `subsystem` 模块，代码如下：
```c
#include "py/builtin.h"
#include "py/runtime.h"

#if MICROPY_PY_SUBSYSTEM  // 条件编译：仅当启用该模块配置时才编译

// info()
static mp_obj_t py_subsystem_info(void) {
    return MP_OBJ_NEW_SMALL_INT(42);  // 返回小整数 42
}
// 定义函数对象：无参数（0 表示参数个数），关联上述函数
MP_DEFINE_CONST_FUN_OBJ_0(subsystem_info_obj, py_subsystem_info);

// 模块全局变量表：定义模块内的属性和函数映射
static const mp_rom_map_elem_t mp_module_subsystem_globals_table[] = {
    { MP_ROM_QSTR(MP_QSTR___name__), MP_ROM_QSTR(MP_QSTR_subsystem) }, // 模块名称：subsystem
    { MP_ROM_QSTR(MP_QSTR_info), MP_ROM_PTR(&subsystem_info_obj) },    // 模块函数：info() 关联到函数对象
};
// 将全局变量表转换为常量字典
static MP_DEFINE_CONST_DICT(mp_module_subsystem_globals, mp_module_subsystem_globals_table);

// 定义模块对象
const mp_obj_module_t mp_module_subsystem = {
    .base = { &mp_type_module },  // 继承模块基类
    .globals = (mp_obj_dict_t *)&mp_module_subsystem_globals,  // 关联模块全局字典
};

// 注册模块：将模块加入 MicroPython 系统，使其可被 import
MP_REGISTER_MODULE(MP_QSTR_subsystem, mp_module_subsystem);

#endif
```

上述实现包含以下关键部分：
1. 定义模块相关的所有函数（如示例中的 `py_subsystem_info`）；
2. 将函数添加到模块全局变量表 `mp_module_subsystem_globals_table` 中，建立“函数名-函数对象”的映射；
3. 通过 `mp_obj_module_t` 结构体创建模块对象 `mp_module_subsystem`；
4. 调用 `MP_REGISTER_MODULE` 宏将模块注册到整个系统中，使其成为 MicroPython 可识别的内置模块。

对修改后的 MicroPython 进行编译并运行后，新模块即可通过 `import` 导入使用：
```python
>>> import subsystem  # 导入新增的 subsystem 模块
>>> subsystem.info()  # 调用模块的 info() 函数
42
>>>
```

当前示例中，`info()` 函数仅返回一个固定数值（42），实际开发中可扩展其功能以实现任意逻辑；同理，也可向该新模块中添加更多函数。

