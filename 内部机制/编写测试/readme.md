# 编写测试

MicroPython 中的测试文件位于路径 `tests/` 下。以下是关键目录及测试运行脚本 `run-tests.py` 的结构清单：

```
.
├── basics       （基础功能测试目录）
├── extmod       （扩展模块测试目录）
├── float        （浮点运算测试目录）
├── micropython  （MicroPython 专属功能测试目录）
├── run-tests.py （测试运行脚本）
...
```

项目中通过子文件夹对测试用例进行分类管理。若需添加新测试，可在现有子文件夹中创建新文件，或新建一个子文件夹存放。对于自定义硬件，建议在 `tests` 文件夹外单独编写自定义测试用例。

例如，在 `tests/unix/` 子目录下创建文件 `print.py`，并写入以下代码：
```python
def print_one():
    print(1)

print_one()
```

运行测试后，该测试会出现在测试输出结果中：
```bash
$ cd ports/unix  # 进入 Unix 端口目录
$ make tests     # 执行测试
skip unix/extra_coverage.py  # 跳过该测试文件
pass unix/ffi_callback.py    # 该测试通过
pass unix/ffi_float.py       # 该测试通过
pass unix/ffi_float2.py      # 该测试通过
pass unix/print.py           # 新增的 print.py 测试通过
pass unix/time.py            # 该测试通过
pass unix/time2.py           # 该测试通过
```

测试的执行逻辑是：将测试目标的输出结果与 CPython 的输出结果进行对比。因此，所有测试用例都应通过 `print` 语句输出内容，以便判断测试结果是否正确。

对于无法与 CPython 输出对比的测试（如 MicroPython 专属功能测试），可提供一个 `.py.exp` 文件（预期结果文件），测试时将以该文件中的内容作为对比基准。

当测试目标不是 Unix 版本（如硬件开发板）时，可采用以下方式运行测试：
```bash
$ cd tests          # 进入 tests 目录
$ ./run-tests.py    # 执行测试（默认针对本地环境）
```

在开发板上运行测试（通过 USB 端口 `/dev/ttyACM0` 连接开发板执行测试）
```bash
$ ./run-tests.py -t /dev/ttyACM0
```

运行指定范围的测试（如特定目录或文件）
```bash
$ ./run-tests.py -d basics         # 仅运行 basics 目录下的所有测试
$ ./run-tests.py float/builtin*.py # 仅运行 float 目录下以“builtin”开头的 .py 测试文件
```
