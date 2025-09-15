# 编译器

MicroPython 中的编译过程包含以下步骤：
- 词法分析器（lexer）将构成 MicroPython 程序的文本流转换为记号（tokens）。
- 语法分析器（parser）随后将记号转换为抽象语法（即语法分析树，parse tree）。
- 最后根据语法分析树生成字节码（bytecode）或原生代码（native code）。

为便于理解，我们将新增一个简单的语言特性 `add1`，它在 Python 中可按如下方式使用：
```python
>>> add1 3
4
>>>
```
其中，`add1` 语句接收一个整数作为参数，并将该整数加 1 后返回结果。


## 添加语法规则

MicroPython 的语法以 [CPython 语法](https://docs.python.org/3.5/reference/grammar.html)为基础，定义在 [py/grammar.h](https://github.com/micropython/micropython/blob/master/py/grammar.h) 文件中。该语法用于解析 MicroPython 源文件。

定义语法规则需了解两个宏：`DEF_RULE` 和 `DEF_RULE_NC`。其中，`DEF_RULE` 可用于定义带有关联编译函数的规则；而 `DEF_RULE_NC` 所定义的规则则不包含（NC，即“No Compile”）编译函数。

为我们新增的 `add1` 语句定义带有编译函数的简单语法，格式如下：
```c
DEF_RULE(add1_stmt, c(add1_stmt), and(2), tok(KW_ADD1), rule(testlist))
```

第二个参数 `c(add1_stmt)` 是对应的编译函数，需在 `py/compile.c` 文件中实现，作用是将该语法规则转换为可执行代码。

第三个必需参数可以是 `or` 或 `and`，用于指定与某条语句关联的节点数量。以本例为例，`add1` 语句类似汇编语言中的 `ADD1` 指令，需接收一个数值参数。因此，`add1_stmt` 关联两个节点：一个节点对应语句本身（即与 `KW_ADD1` 对应的字面量 `add1`），另一个节点对应其参数（`testlist` 规则，该规则是顶层表达式规则）。

**注意**  
- 此处的 `add1` 规则仅为示例，并非标准 MicroPython 语法的一部分。

本例中的第四个参数是与该规则关联的记号 `KW_ADD1`。需通过编辑 `py/lexer.h` 文件，在词法分析器（lexer）中定义该记号。

如果要定义不包含编译函数的同一规则，需使用 `DEF_RULE_NC` 宏，并省略编译函数参数，格式如下：
```c
DEF_RULE_NC(add1_stmt, and(2), tok(KW_ADD1), rule(testlist))
```

其余参数的含义保持不变。对于无编译函数的规则，所有可能将该规则作为节点的其他规则，都必须显式处理该规则。此类无编译函数规则（NC-rules）通常用于表示复杂语法结构的子部分，这些子部分无法用单条规则完整表达。

**注意**  
- 宏 `DEF_RULE` 和 `DEF_RULE_NC` 还可接收其他参数。若需深入了解支持的参数，请参考 `py/grammar.h` 文件。


## 添加词法记号

语法中定义的每一条规则，都应对应一个在 `py/lexer.h` 中定义的记号。可通过编辑 `_mp_token_kind_t` 枚举类型来添加该记号，具体如下：
```c
typedef enum _mp_token_kind_t {
    ...
    MP_TOKEN_KW_OR,       // 关键字“or”对应的记号
    MP_TOKEN_KW_PASS,     // 关键字“pass”对应的记号
    MP_TOKEN_KW_RAISE,    // 关键字“raise”对应的记号
    MP_TOKEN_KW_RETURN,   // 关键字“return”对应的记号
    MP_TOKEN_KW_TRY,      // 关键字“try”对应的记号
    MP_TOKEN_KW_WHILE,    // 关键字“while”对应的记号
    MP_TOKEN_KW_WITH,     // 关键字“with”对应的记号
    MP_TOKEN_KW_YIELD,    // 关键字“yield”对应的记号
    MP_TOKEN_KW_ADD1,     // 新增：关键字“add1”对应的记号
    ...
} mp_token_kind_t;
```

随后还需编辑 `py/lexer.c` 文件，添加该新关键字的字面文本：
```c
static const char *const tok_kw[] = {
    ...
    "or",       // 与 MP_TOKEN_KW_OR 对应
    "pass",     // 与 MP_TOKEN_KW_PASS 对应
    "raise",    // 与 MP_TOKEN_KW_RAISE 对应
    "return",   // 与 MP_TOKEN_KW_RETURN 对应
    "try",      // 与 MP_TOKEN_KW_TRY 对应
    "while",    // 与 MP_TOKEN_KW_WHILE 对应
    "with",     // 与 MP_TOKEN_KW_WITH 对应
    "yield",    // 与 MP_TOKEN_KW_YIELD 对应
    "add1",     // 新增：与 MP_TOKEN_KW_ADD1 对应
    ...
};
```

需注意，关键字的命名可根据需求自定义，但为保持一致性，应遵循相应的命名规范。

**注意**  
- `py/lexer.c` 中这些关键字的顺序，必须与 `py/lexer.h` 枚举类型中记号的顺序完全一致。


## 语法分析（Parsing）

在语法分析阶段，语法分析器（parser）会接收词法分析器（lexer）生成的记号（tokens），并将其转换为抽象语法树（Abstract Syntax Tree，简称 AST）或语法分析树（parse tree）。语法分析器的实现定义在 [py/parse.c](https://github.com/micropython/micropython/blob/master/py/parse.c) 文件中。

语法分析器还会维护一个常量表，用于语法分析各环节，其作用类似[符号表](https://steemit.com/programming/@drifter1/writing-a-simple-compiler-on-my-own-symbol-table-basic-structure)。此阶段会执行多项优化操作，例如：对整数的多数运算（如逻辑运算、二元运算、一元运算等）进行[常量折叠](http://compileroptimizations.com/category/constant_folding.htm)（constant folding）、优化表达式外层的括号结构，同时还会对字符串执行部分优化。

需要注意的是，文档字符串（docstrings）会在此阶段被丢弃，无法被编译器访问。即便像[字符串驻留](https://en.wikipedia.org/wiki/String_interning)（string interning）这类优化，也不会应用于文档字符串。


## 编译器遍历（Compiler Passes）

与许多编译器类似，MicroPython 会将所有代码编译为 MicroPython 字节码或原生代码。实现该功能的代码位于 [py/compile.c](https://github.com/micropython/micropython/blob/master/py/compile.c) 文件中。需要了解的最关键方法如下：

```c
mp_obj_t mp_compile(mp_parse_tree_t *parse_tree, qstr source_file, bool is_repl) {
    // 为当前模块创建上下文，并设置其全局字典
    mp_module_context_t *context = m_new_obj(mp_module_context_t);
    context->module.globals = mp_globals_get();

    // 将输入的语法分析树（parse_tree）编译为原始代码（raw-code）结构
    mp_compiled_module_t cm;
    cm.context = context;
    mp_compile_to_raw_code(parse_tree, source_file, is_repl, &cm);

    // 创建并返回一个可执行外部模块的函数对象
    return mp_make_function_from_proto_fun(cm.rc, cm.context, NULL);
}
```

编译器通过**四次遍历处理** (four passes) 完成代码编译，分别是：作用域（scope）、栈大小（stack size）、代码大小（code size）和代码生成（emit）。每一次遍历都会对相同的抽象语法树（AST）数据结构执行相同的 C 代码，但会根据前一遍历的结果计算不同的内容。


### 第一次遍历（First Pass）

在第一次遍历处理中，编译器会识别所有已知标识符（变量）及其作用域类型（如全局作用域、局部作用域、闭包作用域等）。同时，代码生成器（负责生成字节码或原生代码）也会计算生成代码所需的标签（labels）数量。

```c
// Compile pass 1.
comp->emit = emit_bc;
comp->emit_method_table = &emit_bc_method_table;

uint max_num_labels = 0;
for (scope_t *s = comp->scope_head; s != NULL && comp->compile_error == MP_OBJ_NULL; s = s->next) {
    if (s->emit_options == MP_EMIT_OPT_ASM) {
        compile_scope_inline_asm(comp, s, MP_PASS_SCOPE);
    } else {
        compile_scope(comp, s, MP_PASS_SCOPE);
        
        // Check if any implicitly declared variables should be closed over.
        for (size_t i = 0; i < s->id_info_len; ++i) {
            id_info_t *id = &s->id_info[i];
            if (id->kind == ID_INFO_KIND_GLOBAL_IMPLICIT) {
                scope_check_to_close_over(s, id);
            }
        }
    }
    ...
}
```


### 第二次与第三次遍历（Second and Third Passes）

第二次与第三次遍历处理主要用于计算 Python 栈的大小，以及字节码或原生代码的代码长度。第三次遍历处理结束后，代码长度将固定不变，否则会导致跳转标签（jump labels）失效。

```c
for (scope_t *s = comp->scope_head; s != NULL && comp->compile_error == MP_OBJ_NULL; s = s->next) {
    ...
    
    // 第二次遍历：计算 Python 栈的大小
    compile_scope(comp, s, MP_PASS_STACK_SIZE);

    // 第三次遍历：计算代码长度
    if (comp->compile_error == MP_OBJ_NULL) {
        compile_scope(comp, s, MP_PASS_CODE_SIZE);
    }
    
    ...
}
```

在第二次遍历处理开始前，会先选择待生成的代码类型：原生代码（native code）或字节码（bytecode），具体逻辑如下：

```c
// 选择代码生成器（emitter）类型
switch (s->emit_options) {
    // 若指定生成“原生 Python 代码”或“VIPER 代码”
    case MP_EMIT_OPT_NATIVE_PYTHON:
    case MP_EMIT_OPT_VIPER:
        if (emit_native == NULL) {
            emit_native = NATIVE_EMITTER(new)(&comp->compile_error, &comp->next_label, max_num_labels);
        }
        comp->emit_method_table = NATIVE_EMITTER_TABLE;
        comp->emit = emit_native;
        break;
    
    default:
        comp->emit = emit_bc;
        comp->emit_method_table = &emit_bc_method_table;
        break;
}
```

字节码（bytecode）是默认的代码生成选项，但原生代码（native code）选项有一个需特别注意的特性：它还支持通过 `VIPER` 模式生成。有关 VIPER 注解的更多详细信息，请参考 “生成原生代码（Emitting native code）” 章节。

此外，编译器还支持内联汇编代码（inline assembly code），汇编指令以 Python 函数调用的形式编写，但会直接生成对应的机器码。该汇编器仅需三次遍历处理（分别是作用域、代码大小、代码生成），且采用独立的实现逻辑，不依赖 `compile_scope` 函数。更多细节请参考[内联汇编教程](https://docs.micropython.org/en/latest/pyboard/tutorial/assembler.html#pyboard-tutorial-assembler)（inline assembler tutorial）。

### 第四次遍历（Fourth Pass）

第四次遍历处理会生成最终可执行的代码：要么是可在虚拟机中运行的字节码，要么是可由 CPU 直接执行的原生代码。

```c
for (scope_t *s = comp->scope_head; s != NULL && comp->compile_error == MP_OBJ_NULL; s = s->next) {
    ...

    // 第四次遍历：生成编译后的字节码或原生代码
    if (comp->compile_error == MP_OBJ_NULL) {
        compile_scope(comp, s, MP_PASS_EMIT);
    }
}
```


## 生成字节码

Python 代码中的语句通常会对应生成字节码，例如 `a + b` 会生成"压入 a"（push a）、"压入 b"（push b）、"二元运算加法"（binary op add）这一系列字节码指令。有些语句本身不会生成任何字节码，而是通过影响其他元素（如变量的作用域）发挥作用，例如 `global a`（声明 `a` 为全局变量）语句。

生成字节码的函数，其实现结构如下所示：
```c
void mp_emit_bc_unary_op(emit_t *emit, mp_unary_op_t op) {
    emit_write_bytecode_byte(emit, 0, MP_BC_UNARY_OP_MULTI + op);
}
```

此处以一元运算符表达式的字节码生成函数为例，但其他语句/表达式的字节码生成函数，其实现逻辑也与此类似。其中，`emit_write_bytecode_byte()` 是一个封装函数，它基于核心函数 `emit_get_cur_to_write_bytecode()` 实现所有生成字节码的函数，都必须通过调用这类封装函数或核心函数来完成字节码的生成。


## 生成原生代码

与字节码的生成方式类似，`py/emitnative.c` 文件中应为每条代码语句对应定义一个生成函数，示例如下：
```c
static void emit_native_unary_op(emit_t *emit, mp_unary_op_t op) {
    vtype_kind_t vtype;
    emit_pre_pop_reg(emit, &vtype, REG_ARG_2);
    if (vtype == VTYPE_PYOBJ) {
        emit_call_with_imm_arg(emit, MP_F_UNARY_OP, op, REG_ARG_1);
        emit_post_push_reg(emit, VTYPE_PYOBJ, REG_RET);
    } else {
        adjust_stack(emit, 1);
        EMIT_NATIVE_VIPER_TYPE_ERROR(emit,
            MP_ERROR_TEXT("unary op %q not implemented"), mp_unary_op_method_name[op]);
    }
}
```

两者的区别在于，原生代码生成需处理 *VIPER 类型*。VIPER 注解支持处理多种类型的变量：默认情况下，所有变量均为 Python 对象；但通过 VIPER，变量也可声明为机器类型（如原生整数、指针等）。可将 VIPER 视为 Python 的超集，普通 Python 对象的以常规方式处理，而原生机器变量则通过直接使用机器指令执行运算，以优化方式处理。需注意的是，VIPER 类型可能会打破 Python 的等价性，例如：VIPER 中的整数为原生整数，可能发生溢出（而 Python 整数可自动扩展为任意精度，不会溢出）。
