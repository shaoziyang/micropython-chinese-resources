# MicroPython 清单文件

## 概述

MicroPython 具有一项功能，允许将 Python 代码"冻结"到固件中，以此作为从文件系统加载代码的替代方案。

这一功能具有以下优势：

- 代码会被预编译为字节码，避免了在加载时对 Python 源代码进行编译的需求。
- 字节码可以直接从 ROM（即闪存）执行，而无需复制到 RAM 中。同样，任何常量对象（字符串、元组等）也会从 ROM 中加载。这能为应用程序释放出显著更多的可用内存。
- 在没有文件系统的设备上，这是加载 Python 代码的唯一方式。

在开发过程中，通常不建议使用冻结功能，因为这会显著拖慢开发周期 —— 每次更新都需要重新写入整个固件。不过，有选择地冻结一些很少变动的依赖项（如第三方库）仍然是有用的。

列出要冻结到固件中的 Python 文件的方式是通过"清单"来实现，清单是一个会在构建过程中被解析的 Python 文件。通常，你会在板级定义中编写清单文件，但也可以编写一个独立的清单文件，并将其与现有的板级定义一起使用。

清单文件可以定义对来自 micropython-lib 的库、文件系统上的 Python 文件以及其他清单文件的依赖关系。

## 编写清单文件

清单文件是一个包含一系列函数调用的 Python 文件。请参见下文定义的可用函数。

清单文件中使用的任何路径都可以包含以下变量，这些变量均会解析为绝对路径。

- $(MPY_DIR) —— 指向 MicroPython 代码仓库的路径。
- $(MPY_LIB_DIR) —— 指向 micropython-lib 子模块的路径。建议使用 `require()`。
- $(PORT_DIR) —— 指向当前移植版本的路径（例如 `ports/stm32`）
- $(BOARD_DIR) —— 指向当前开发板的路径（例如 `ports/stm32/boards/PYBV11`）

自定义清单文件不应存放在 MicroPython 主代码仓库中，而是将它们与项目的其他部分放在一起。

通常，用于编译固件的清单需要包含硬件清单，已经清单可能包含开发板正常运行所需的冻结模块。如果只想向现有开发板添加额外的模块，那么只需包含开发板清单（它会依次包含硬件清单）。


### 使用自定义清单进行构建

可以在 `make` 命令行中指定清单，命令如下：

```bash
$ make BOARD=MYBOARD FROZEN_MANIFEST=/path/to/my/project/manifest.py
```

这适用于所有硬件，包括基于 CMake 的移植版本（如 esp32、rp2），因为 Makefile 包装器会将该参数传递到 CMake 构建过程中。

### 向板级定义添加清单

如果有自定义的板级定义，可以使其自动包含自定义清单。在基于 make 的移植版（大多数硬件）中，在 `mpconfigboard.mk` 中设置 `FROZEN_MANIFEST` 变量：

```makefile
FROZEN_MANIFEST ?= $(BOARD_DIR)/manifest.py
```

在基于 CMake 的硬件（如 esp32、rp2）中，则使用 `mpconfigboard.cmake`：
```cmake
set(MICROPY_FROZEN_MANIFEST ${MICROPY_BOARD_DIR}/manifest.py)
```

### 高级函数

注意：可以在各种函数上设置`opt`关键字参数，它用于控制交叉编译器使用的优化级别。参见`micropython.opt_level()`。

- `add_library(library, library_path, prepend=False)`

  注册外部命名库的路径。

  使用`require`时，会自动搜索`library_path`这个路径。默认情况下，添加的库会被添加到待搜索库列表的末尾。若传递`True`给`prepend`，则会将其添加到列表的开头。

  此外，通过使用`require("name", library="library")`可以显式请求添加的库。
<br><br>

- `package(package_path, files=None, base_path='.', opt=None)`

  这相当于将“package_path”目录复制到设备（作为冻结代码除外）。

  在最简单的情况下，要冻结当前目录中的“foo”包：

  ```
  package("foo")
  ```

  会递归包含`foo`中的所有`.py`文件，并将其冻结为`foo/**/*.py`。

  如果包不在与清单文件相同的目录中，请使用`base_path`：

  ```
  package("foo", base_path="path/to/libraries")
  ```

  可以在`base_path`中使用上述变量，例如`$(PORT_DIR)`。

  要限制包中的某些文件，请使用`files`（注意：路径应相对于包）：`package("foo", files=["bar/baz.py"])`.
<br><br>

- `module(module_path, base_path='.', opt=None)`

  将单个Python文件作为模块包含进来。

  如果文件在当前目录中：

  ```
  module("foo.py")
  ```

  否则，使用`base_path`来定位文件：

  ```
  module("foo.py", base_path="src/drivers")
  ```

  可以在`base_path`中使用上述变量，例如`$(PORT_DIR)`。
<br><br>

- `require(name, library=None)`

  从`micropython-lib`中按名称（及其依赖项）引入一个包。

  可以选择指定`library`（一个字符串），以引用先前通过`add_library`注册的库中的包。否则，将使用库路径列表。
<br><br>

- `include(manifest_path)`

  包含另一个清单。

  通常，用于编译固件的清单需要包含硬件清单，硬件清单可能包含开发板正常运行所需的冻结模块。

  `manifest`参数可以是一个字符串（文件名）或字符串的可迭代对象。

  相对路径是相对于当前清单文件解析的。

  如果路径指向一个目录，则会隐式包含该目录内的`manifest.py`文件。

  可以在`manifest_path`中使用上述变量，例如`$(PORT_DIR)`。
<br><br>

- `metadata(description=None, version=None, license=None, author=None)`

  为该清单文件定义元数据。这对于`micropython-lib`包的清单很有用。


### 低级函数

记录这些函数是为了内容的完整性，但除了`freeze_as_str`之外，所有功能都可以通过高级函数来访问。

- `freeze(path, script=None, opt=0)`

  冻结由`path`指定的输入，并自动确定其类型。`.py`脚本会先被编译为`.mpy`文件，然后再被冻结；而`.mpy`文件则会被直接冻结。

  `path`必须是一个目录，作为开始搜索文件的基准目录。导入冻结后的模块时，模块名称会从`path`之后开始，也就是说，`path`不会包含在模块名称中。

  如果`path`是相对路径，它会被解析为相对于当前`manifest.py`的路径。

  如果`script`为`None`，则`path`中的所有文件都会被冻结。

  如果`script`是一个可迭代对象，那么会对该可迭代对象的所有元素调用`freeze()`（同时传递相同的`path`和`opt`参数）。

  如果`script`是一个字符串，它指定了要冻结的文件或目录，并且在文件或最后一个目录之前可以包含额外的目录。该文件或目录会在`path`中进行搜索。如果`script`是一个目录，那么该目录中的所有文件都会被冻结。

  `opt`是将`.py`编译为`.mpy`时传递给`mpy-cross`的优化级别。这些级别在`micropython.opt_level()`中有描述。
<br><br>

- `freeze_as_str(path)`

  将给定的`path`及其包含的所有`.py`脚本作为字符串冻结，这些字符串会在导入时进行编译。
<br><br>

- `freeze_as_mpy(path, script=None, opt=0)`

  通过先将`.py`脚本编译为`.mpy`文件，然后冻结生成的`.mpy`文件来对输入进行冻结。有关参数的更多详细信息，请参见`freeze()`。
<br><br>

- `freeze_mpy(path, script=None, opt=0)`

  对输入进行冻结，输入必须是直接被冻结的`.mpy`文件。有关参数的更多详细信息，请参见`freeze()`。


## 示例

要冻结当前目录中的单个文件（可通过`import mydriver`导入），请使用：

```python
module("mydriver.py")
```

要冻结当前目录的子目录“mydriver”中的所有文件（可通过`import mydriver`导入），请使用：

```python
package("mydriver")
```

要冻结micropython-lib中的“hmac”库，请使用：

```python
require("hmac")
```

一个针对PYBD_SF2开发板的自定义manifest.py文件的更完整示例如下：

```python
# 包含开发板的默认清单
include("$(BOARD_DIR)/manifest.py")
# 添加自定义驱动
module("mydriver.py")
# 从micropython-lib添加aiorepl
require("aiorepl")
```

之后可以按以下方式编译该开发板：

```bash
$ cd ports/stm32
$ make BOARD=PYBD_SF2 FROZEN_MANIFEST=~/src/myproject/manifest.py
```

请注意，大多数开发板没有自己的 `manifest.py`，而是直接使用硬件的清单，在这种情况下，清单应改为包含`include("$(PORT_DIR)/boards/manifest.py")`。
