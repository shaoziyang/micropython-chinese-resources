# 包管理

## 使用 mip 安装包

具备网络功能的开发板包含 `mip` 模块，该模块可以从 `micropython-lib` 以及第三方网站（包括 GitHub、GitLab）安装包。

`mip`（“mip installs packages”，即 mip 安装包）在概念上与 Python 的 `pip` 工具类似，但它不使用 PyPI 索引，而是默认将 micropython-lib 用作其索引。从 micropython-lib 下载时，mip 会自动获取编译后的 `.mpy` 文件。

使用 mip 最常见的方式是通过 REPL：

```
>>> import mip
>>> mip.install("pkgname")  # 安装“pkgname”的最新版本（及依赖项）
>>> mip.install("pkgname", version="x.y")  # 安装“pkgname”的 x.y 版本
>>> mip.install("pkgname", mpy=False)  # 安装源代码版本（即 .py 文件而非 .mpy 文件）
```

mip 会通过在 `sys.path` 中搜索以 `/lib` 结尾的第一个条目，来确定文件系统上的合适位置。你可以使用 `target` 来覆盖目标位置，但请注意，该路径必须在 `sys.path` 中，以便后续能够导入：

```
>>> mip.install("pkgname", target="third-party")
>>> sys.path.append("third-party")
```

除了从 micropython-lib 索引下载包外，mip 还可以安装第三方库。最简单的方法是直接下载文件：

```
>>> mip.install("http://example.com/x/y/foo.py")
>>> mip.install("http://example.com/x/y/foo.mpy")
```

直接安装文件时，仍支持使用 `target` 参数设置目标路径，但会忽略 `mpy` 和 `version` 参数。

URL 也可以以 `github:` 或 `gitlab:` 开头，以此简便地指向托管在 GitHub 或 GitLab 上的内容：

```
>>> mip.install("github:org/repo/path/foo.py")  # 使用默认分支
>>> mip.install("github:org/repo/path/foo.py", version="branch-or-tag")  # 可选地指定分支或标签
>>> mip.install("gitlab:org/repo/path/foo.py")  # 使用默认分支
>>> mip.install("gitlab:org/repo/path/foo.py", version="branch-or-tag")  # 可选地指定分支或标签
```

更复杂的包（即包含多个文件或有依赖项的包）可以通过指定其 `package.json` 的路径来下载：

```
>>> mip.install("http://example.com/x/package.json")
>>> mip.install("github:org/user/path/package.json")
>>> mip.install("gitlab:org/user/path/package.json")
```

如果未指定 json 文件，则会隐式添加“package.json”：

```
>>> mip.install("http://example.com/x/")
>>> mip.install("github:org/repo")  # 使用该仓库的默认分支
>>> mip.install("github:org/repo", version="branch-or-tag")
>>> mip.install("gitlab:org/repo")  # 使用该仓库的默认分支
>>> mip.install("gitlab:org/repo", version="branch-or-tag")
```

### 在 Unix 移植版上使用 mip

在 Unix 移植版上，`mip` 既可以像上面那样在 REPL 中使用，也可以通过 `-m` 参数来使用：

```
$ ./micropython -m mip install pkgname-or-url
$ ./micropython -m mip install pkgname-or-url@version
```

可以设置 `--target path`、`--no-mpy` 和 `--index` 参数：
```
$ ./micropython -m mip install --target third-party pkgname
$ ./micropython -m mip install --no-mpy pkgname
$ ./micropython -m mip install --index https://host/pi pkgname
```

## 使用 mpremote 安装包

`mpremote` 工具也包含与 `mip` 相同的功能，可从主机 PC 上使用，向本地连接的设备（例如通过 USB 或 UART 连接的设备）安装包：

```
$ mpremote mip install pkgname
$ mpremote mip install pkgname@x.y
$ mpremote mip install http://example.com/x/y/foo.py
$ mpremote mip install github:org/repo
$ mpremote mip install github:org/repo@branch-or-tag
$ mpremote mip install gitlab:org/repo
$ mpremote mip install gitlab:org/repo@branch-or-tag
```

可以设置 `--target=path`、`--no-mpy` 和 `--index` 参数：

```
$ mpremote mip install --target=/flash/third-party pkgname
$ mpremote mip install --no-mpy pkgname
$ mpremote mip install --index https://host/pi pkgname
```

`mpremote` 还可以从主机本地文件系统中存储的文件安装包：

```
$ mpremote mip install path/to/pkg.py
$ mpremote mip install path/to/app/package.json
$ mpremote mip install \\path\\to\\pkg.py
```

这在开发过程中测试包以及从 GitHub 仓库的本地克隆版安装包时特别有用。请注意，`package.json` 文件中的 URL 必须使用正斜杠（“/”）作为目录分隔符，即使在 Windows 系统上也是如此，这样才能与从网络安装的方式兼容。


## 手动安装包

也可以通过手动将文件（.py 或 .mpy 格式均可）复制到设备上来安装包。根据开发板的不同，可通过 USB 大容量存储、mpremote 工具（例如 `mpremote fs cp path/to/package.py :package.py`）、webrepl 等方式进行操作。


## 编写与发布包

将包发布到 micropython-lib 是让你的包被广大 MicroPython 用户轻松获取的最简单方式，这样的包会自动通过 `mip` 和 `mpremote` 工具供用户使用，并且会被编译为字节码。更多信息请参见 https://github.com/micropython/micropython-lib。

若要编写一个可通过 mip 或 mpremote 工具下载的“自托管”包，需要一个静态网络服务器（或 GitHub）来托管单个 `.py` 文件，或者托管一个与 `.py` 文件放在一起的 `package.json` 文件。

一个托管在 GitHub 上的 mlx90640 库示例可以通过以下命令安装：

```
$ mpremote mip install github:org/micropython-mlx90640
```

该包在 GitHub 上的文件结构可能如下：

```
https://github.com/org/micropython-mlx90640/
    package.json
    mlx90640/
        __init__.py
        utils.py
```

`package.json` 文件用于指定待安装文件的位置以及其他依赖项：

```json
{
    "urls": [
        ["mlx90640/__init__.py", "mlx90640/__init__.py"],
        ["mlx90640/utils.py", "mlx90640/utils.py"]
    ],
    "deps": [
        ["collections-defaultdict", "latest"],
        ["os-path", "latest"],
        ["github:org/micropython-additions", "main"],
        ["gitlab:org/micropython-otheradditions", "main"]
    ],
    "version": "0.2"
}
```

`urls` 列表按以下规则指定待安装的文件：
```
"urls": [
    [destination_path, source_url]
]
```

其中，`destination_path` 是文件在设备上的安装位置和名称，`source_url` 是待安装文件的 URL。源 URL 通常相对于包含 `package.json` 文件的目录来指定，但也可以是绝对 URL，例如：
```
["mlx90640/utils.py", "github:org/micropython-mlx90640/mlx90640/utils.py"]
```

该包依赖于 `collections-defaultdict` 和 `os-path`，这两个依赖会自动从 micropython-lib 安装。第三个依赖会按照 GitHub 仓库 `org/micropython-additions` 的 `main` 分支中 `package.json` 文件的定义来安装相关内容。


## 冻结包

当从设备文件系统导入 Python 模块或包时，它会在 RAM 中被编译为字节码，以便由虚拟机执行。对于 `.mpy` 文件，这种转换已经完成，但字节码仍然会存放在 RAM 中。

对于内存较小的设备，或者大型应用程序，从 ROM（即闪存）运行字节码可能更为有利。这可以通过将字节码“冻结”到 MicroPython 固件中来实现，然后将该固件刷写到设备上。运行时性能是相同的（不过导入速度更快），但这可以释放大量 RAM 供程序使用。

这种方法的缺点是开发速度会慢很多，因为每次都必须刷写固件，但冻结那些不常更改的依赖项仍然是有用的。

冻结操作通过编写清单文件并在构建过程中使用该文件来完成，这通常是自定义开发板定义的一部分。更多信息请参见 MicroPython 清单文件指南。

