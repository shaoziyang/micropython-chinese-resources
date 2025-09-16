# MicroPython 字符串驻留

MicroPython 采用字符串驻留（string interning）技术来节省 RAM（随机存取存储器）和 ROM（只读存储器）空间，避免存储相同字符串的重复副本。该技术主要适用于代码中的标识符（identifiers），因为像函数名或变量名这类标识符，极有可能在代码的多个位置出现。在 MicroPython 中，经过驻留处理的字符串被称为 QSTR（即“唯一字符串”，uniQue STRing）。

QSTR 值（类型为 qstr）是 QSTR 池（QSTR pools）链表中的一个索引。QSTR 会存储自身的长度以及内容的哈希值，以便在去重（de-duplication）过程中实现快速比较。所有用于处理字符串的字节码操作，均使用 QSTR 作为参数。

## 编译时生成 QSTR

在 MicroPython 的 C 代码中，所有需在最终固件中进行驻留处理的字符串，均以 `MP_QSTR_Foo` 的形式编写。在编译阶段，该形式会解析为一个 `qstr` 类型的值，该值指向"`Foo`"在 QSTR 池中的索引位置。

`Makefile` 中的多步骤流程实现了这一机制。概括而言，该流程包含三个部分：
1. 在代码中查找所有 `MP_QSTR_Foo` 标记。
2. 生成一个静态 QSTR 池，其中包含所有字符串数据（包括字符串长度和哈希值）。
3. （通过预处理器）将所有 `MP_QSTR_Foo` 替换为其对应的索引。

`MP_QSTR_Foo` 标记的搜索来源分为两类：
1. 所有在 `$(SRC_QSTR)` 中引用的文件。这类文件包含所有 C 代码（即 `py`、`extmod`、`ports/stm32` 目录下的代码），但不包括 `lib` 等第三方代码。
2. 额外的 `$(QSTR_GLOBAL_DEPENDENCIES)`（其中包含 `mpconfig*.h` 系列文件）。

**注意**：`frozen_mpy.c`（由 mpy-tool.py 生成）拥有独立的 QSTR 生成流程和 QSTR 池。

某些无法用 `MP_QSTR_Foo` 语法表示的额外字符串（例如包含非字母数字字符的字符串），会通过 `$(QSTR_DEFS)` 变量在 `qstrdefs.h` 和 `qstrdefsport.h` 中显式定义。

QSTR 的处理分为以下阶段：
1. **生成 qstr.i.last 文件**：将所有输入文件通过 C 预处理器处理后拼接，生成此文件。此过程会移除所有条件性禁用的代码，并展开宏定义，确保不会将最终固件中未使用的字符串加入 QSTR 池。由于此阶段（借助 QSTR_GEN_CFLAGS 所添加的 NO_QSTR 宏）未定义 `MP_QSTR_Foo`，因此该标记会原样保留。该文件还包含预处理器生成的注释，其中包含行号信息。**注意**：此步骤仅处理已修改的文件，即 qstr.i.last 仅包含自上次编译后发生变更的文件中的数据。
2. **生成 qstr.split 文件**：对 qstr.i.last 运行 `makeqstrdefs.py split` 脚本后，会创建这个空文件。它仅作为一种依赖标记，用于标识该步骤已执行。该脚本会为每个输入的 C 文件生成一个对应文件（路径为 genhdr/qstr/...file.c.qstr），文件中仅包含匹配到的 QSTR，且每个 QSTR 均以 `Q(Foo)` 的形式呈现。此步骤的作用是将现有文件与 qstr.i.last 中增量更新生成的新数据进行合并。
3. **生成 qstrdefs.collected.h 文件**：运行 `makeqstrdefs.py cat` 脚本，将 genhdr/qstr/ 目录下的所有文件拼接，生成该文件。此时文件中包含代码中所有找到的 `MP_QSTR_Foo`（已转换为 `Q(Foo)` 格式），每行一个，且可能存在重复项。**仅当 QSTR 集合发生变化时，该文件才会更新**。QSTR 数据的哈希值会写入另一个文件（qstrdefs.collected.h.hash），用于跟踪不同编译版本间的变化。
4. **生成 qstrdefs.preprocessed.h 文件**：首先生成一个枚举（enumeration），其中每个条目将 `MP_QSTR_Foo` 映射到其对应的索引。具体过程为：先将 qstrdefs.collected.h 与 qstrdefs*.h 拼接，再将每行的 `Q(Foo)` 转换为 `"Q(Foo)"`（确保其通过预处理器时不被修改）；接着使用预处理器处理 qstrdefs*.h 中的条件编译逻辑；最后将格式还原为 `Q(Foo)`，并保存为 qstrdefs.preprocessed.h。
5. **生成 qstrdefs.generated.h 文件**：运行 `makeqstrdata.py` 脚本，以 qstrdefs.preprocessed.h 为输入（同时包含一些额外的硬编码 QSTR），为其中每个 `Q(Foo)` 生成对应的 `QDEF(MP_QSTR_Foo, (const byte*)"hash" "Foo")` 条目，最终生成该文件。

在主编译流程中，`qstrdefs.generated.h` 会用于以下两件事：
1. 在 `qstr.h` 中，每个 `QDEF` 会成为枚举（enum）中的一个条目，使得 `MP_QSTR_Foo` 可在代码中使用，且其值等于该字符串在 QSTR 表中的索引。
2. 在 `qstr.c` 中，实际的 QSTR 数据表会以 `mp_qstr_const_pool->qstrs` 元素的形式生成。
 

## 运行时 QSTR 生成

在运行时可创建额外的 QSTR 池，以便向其中添加字符串。例如，对于代码：

```Python
foo[x] = 3
```

需要为 `x` 的值创建一个 QSTR，使其能被“加载属性”（load attr）字节码使用。

此外，在编译 Python 代码时，需要为标识符（identifiers）和字面量（literals）创建 QSTR。**注意**：仅长度小于 10 个字符的字面量会被创建为 QSTR。这是因为堆（heap）上的常规字符串至少会占用 16 字节空间（一个垃圾回收块，GC block），而 QSTR 能将这些字符串更高效地打包存储到池中。

QSTR 池（以及用于存储字符串数据的底层“数据块”，chunks）会根据需求在堆上分配空间，且分配时会有最小大小限制。
