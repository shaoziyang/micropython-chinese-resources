# 移植 MicroPython
MicroPython 项目包含多个针对不同微控制器系列及架构的移植版本。该项目的代码仓库中有一个 `ports` 目录，其中每个子目录对应一个受支持的移植版本。

一个移植版本通常包含对多个`开发板`的定义，每个`开发板`都是该移植版本可运行的特定硬件设备，例如开发套件或专用设备。

最小化移植版本（minimal port）是一个简化的 MicroPython 移植参考实现，可在主机系统和 STM32F4xx 系列微控制器（MCU）上运行。

通常而言，开始进行移植工作需要完成以下步骤：
- 搭建工具链（配置Makefile等文件）。
- 实现启动配置与CPU初始化。
- 初始化开发和调试所需的基础驱动（如通用输入输出接口GPIO、通用异步收发传输器UART）。
- 完成开发板特定配置。
- 实现移植版本专用模块。


## 最小化MicroPython固件

将 MicroPython 移植到新开发板的最佳方式是集成一个最小化的 MicroPython 解释器。在本指南中，先在 ports 目录下为新移植版本创建一个子目录：

```bash
$ cd ports
$ mkdir example_port
```

基础的 MicroPython 固件在主移植文件（例如`main.c`）中实现：

```c
#include "py/builtin.h"
#include "py/compile.h"
#include "py/gc.h"
#include "py/mperrno.h"
#include "py/stackctrl.h"
#include "shared/runtime/gchelper.h"
#include "shared/runtime/pyexec.h"

// 为MicroPython垃圾回收堆分配内存
static char heap[4096];

int main(int argc, char **argv) {
    // 初始化MicroPython运行时
    mp_stack_ctrl_init();
    gc_init(heap, heap + sizeof(heap));
    mp_init();

    // 启动常规REPL；在空行输入ctrl-D时退出
    pyexec_friendly_repl();

    // Deinitialise the runtime.
    gc_sweep_all();
    mp_deinit();
    return 0;
}

// 处理未捕获的异常（在正确的C实现中永远不应执行到此处）
void nlr_jump_fail(void *val) {
    for (;;) {
    }
}

// 执行一次垃圾回收循环
void gc_collect(void) {
    gc_collect_start();
    gc_helper_collect_regs_and_stack();
    gc_collect_end();
}

// 无文件系统，因此stat操作返回空
mp_import_stat_t mp_import_stat(const char *path) {
    return MP_IMPORT_STAT_NO_EXIST;
}

// 无文件系统，因此打开文件会引发异常
mp_lexer_t *mp_lexer_new_from_file(qstr filename) {
    mp_raise_OSError(MP_ENOENT);
}
```

此时还需要为该移植版本创建一个 Makefile：

```makefile
# 包含核心环境定义；这将设置$(TOP)
include ../../py/mkenv.mk

# 包含py核心编译定义
include $(TOP)/py/py.mk
include $(TOP)/extmod/extmod.mk

# 设置CFLAGS和库
CFLAGS += -I. -I$(BUILD) -I$(TOP)
LIBS += -lm

# 定义所需的源文件
SRC_C = \
    main.c \
    mphalport.c \
    shared/readline/readline.c \
    shared/runtime/gchelper_generic.c \
    shared/runtime/pyexec.c \
    shared/runtime/stdout_helpers.c \

# 定义包含qstrs的源文件
SRC_QSTR += shared/readline/readline.c shared/runtime/pyexec.c

# 定义所需的目标文件
OBJ = $(PY_CORE_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))

# 定义顶级目标，即主固件
all: $(BUILD)/firmware.elf

# 定义固件的构建方式
$(BUILD)/firmware.elf: $(OBJ)
	$(ECHO) "LINK $@"
	$(Q)$(CC) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(Q)$(SIZE) $@

# 包含剩余的核心编译规则
include $(TOP)/py/mkrules.mk
```

注意在Makefile中使用正确的制表符（tab）进行缩进。


## MicroPython 配置
完成上述最小化代码的集成后，下一步是为该移植版本创建 MicroPython 配置文件。编译时配置在 `mpconfigport.h` 中指定，而额外的硬件抽象函数（如时间管理）则在 `mphalport.h` 中定义。

以下是 `mpconfigport.h` 文件的示例：

```c
#include <stdint.h>

// Python internal features.
#define MICROPY_ENABLE_GC (1)
#define MICROPY_HELPER_REPL (1)
#define MICROPY_ERROR_REPORTING (MICROPY_ERROR_REPORTING_TERSE)
#define MICROPY_FLOAT_IMPL (MICROPY_FLOAT_IMPL_FLOAT)

// Fine control over Python builtins, classes, modules, etc.
#define MICROPY_PY_ASYNC_AWAIT (0)
#define MICROPY_PY_BUILTINS_SET (0)
#define MICROPY_PY_ATTRTUPLE (0)
#define MICROPY_PY_COLLECTIONS (0)
#define MICROPY_PY_MATH (0)
#define MICROPY_PY_IO (0)
#define MICROPY_PY_STRUCT (0)

// Type definitions for the specific machine.

typedef intptr_t mp_int_t; // must be pointer size
typedef uintptr_t mp_uint_t; // must be pointer size
typedef long mp_off_t;

// We need to provide a declaration/definition of alloca().
#include <alloca.h>

// Define the port's name and hardware.
#define MICROPY_HW_BOARD_NAME "example-board"
#define MICROPY_HW_MCU_NAME "unknown-cpu"

#define MP_STATE_PORT MP_STATE_VM
```

该配置文件包含机器特定的配置，涵盖多个方面，例如是否应启用不同的MicroPython功能（例如`#define MICROPY_ENABLE_GC (1)`）。若将该设置改为`(0)`，则会禁用该功能。

其他配置包括类型定义、根指针、开发板名称、微控制器名称等。

同样，一个最小化的`mphalport.h`文件示例如下：

```c
static inline void mp_hal_set_interrupt_char(char c) {}
```

## 标准输入/输出支持
MicroPython 至少需要一种输出字符的方式；若要实现交互式解释器（REPL），则还需要一种输入字符的方式。

此类功能的函数可在 `mphalport.c` 文件中实现，示例如下：

```c
#include <unistd.h>
#include "py/mpconfig.h"

// 接收单个字符，阻塞等待直到有字符可用
int mp_hal_stdin_rx_chr(void) {
    unsigned char c = 0;
    int r = read(STDIN_FILENO, &c, 1);
    (void)r;  // 消除未使用变量的编译警告
    return c;
}

// 发送指定长度的字符串
void mp_hal_stdout_tx_strn(const char *str, mp_uint_t len) {
    int r = write(STDOUT_FILENO, str, len);
    (void)r;  // 消除未使用变量的编译警告
}
```

上述输入输出函数需根据特定开发板的 API 进行修改。本示例使用的是标准输入/输出流（standard input/output stream）。

## 构建与运行
在此阶段，新移植版本的目录应包含以下文件：
```
ports/example_port/
├── main.c
├── Makefile
├── mpconfigport.h
├── mphalport.c
└── mphalport.h
```

现在可通过运行 `make` 命令构建该移植版本（具体命令可能因系统而异）。

若使用上述 Makefile 中的默认编译器设置，构建后会生成一个名为 `build/firmware.elf` 的可执行文件，可直接运行。若要启用功能正常的 REPL（交互式解释器），可能需要先将终端配置为原始模式（raw mode），执行以下命令：
```bash
$ stty raw opost -echo
$ ./build/firmware.elf
```

执行后应会进入 MicroPython 的 REPL 环境，此时可运行如下命令：
```
MicroPython v1.13 on 2021-01-01; example-board with unknown-cpu
>>> import sys
>>> sys.implementation
('micropython', (1, 13, 0))
>>>
```

按 `Ctrl+D` 可退出 REPL，之后运行 `reset` 命令可重置终端配置。

## 为移植版本添加模块
如果要添加 `myport` 这类自定义模块，首先需在 `modmyport.c` 文件中定义该模块，代码如下：

```c
#include "py/runtime.h"

static mp_obj_t myport_info(void) {
    mp_printf(&mp_plat_print, "info about my port\n");
    return mp_const_none;
}
// 定义无参数的常量函数对象，关联myport_info函数
static MP_DEFINE_CONST_FUN_OBJ_0(myport_info_obj, myport_info);

// 模块全局变量表，定义模块内的属性与关联对象
static const mp_rom_map_elem_t myport_module_globals_table[] = {
    { MP_OBJ_NEW_QSTR(MP_QSTR___name__), MP_OBJ_NEW_QSTR(MP_QSTR_myport) },  // 模块名称为"myport"
    { MP_ROM_QSTR(MP_QSTR_info), MP_ROM_PTR(&myport_info_obj) },  // 将"info"属性关联到myport_info_obj函数对象
};
// 基于全局变量表定义常量字典
static MP_DEFINE_CONST_DICT(myport_module_globals, myport_module_globals_table);

// 模块对象定义
const mp_obj_module_t myport_module = {
    .base = { &mp_type_module },  // 继承模块类型
    .globals = (mp_obj_dict_t *)&myport_module_globals,  // 关联模块的全局变量字典
};

// 注册模块，将模块名称"myport"与模块对象myport_module绑定
MP_REGISTER_MODULE(MP_QSTR_myport, myport_module);
```

此外，还需修改 Makefile：在 `SRC_C` 列表中添加 `modmyport.c`，并新增一行将该文件加入 `SRC_QSTR`（确保在新文件中搜索 qstr 字符串），修改示例如下：

```makefile
SRC_C = \
    main.c \
    modmyport.c \
    mphalport.c \
    ...

SRC_QSTR += modmyport.c
```

如果所有操作无误，重新构建后，即可导入该新模块并使用：
```python
>>> import myport
>>> myport.info()
info about my port
>>>
```


