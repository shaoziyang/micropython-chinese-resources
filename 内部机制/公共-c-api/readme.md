# 公共 C API

公共 C API（应用程序编程接口）由 `py/` 目录下所有 C 头文件中定义的函数构成。大多数重要的核心运行时 C API 都在 `runtime.h` 和 `obj.h` 这两个头文件中暴露（供外部调用）。

以下是来自 `obj.h` 的公共 API 函数示例：
```c
mp_obj_t mp_obj_new_list(size_t n, mp_obj_t *items);
mp_obj_t mp_obj_list_append(mp_obj_t self_in, mp_obj_t arg);
mp_obj_t mp_obj_list_remove(mp_obj_t self_in, mp_obj_t value);
void mp_obj_list_get(mp_obj_t self_in, size_t *len, mp_obj_t **items);
```

从本质上来说，头文件中的所有函数和宏都属于公共 API 的范畴，可用于访问 MicroPython 底层的具体细节。头文件中的静态内联函数（static inline functions）同样适用，这类函数在被调用时，会被内联到代码中。

需要注意的是，`ports` 目录下的头文件仅对特定硬件的专属功能暴露（即仅该硬件的代码可调用）。
