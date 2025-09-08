# 入门指南

本指南详细介绍了一系列分步操作流程，包括版本控制的设置、获取并构建移植版源代码副本、生成文档、运行测试，以及对 MicroPython 代码库的目录结构说明。

## 使用 Git 进行源代码控制

MicroPython 托管于 [GitHub](https://github.com/micropython/micropython) 平台，并采用 [Git](https://git-scm.com) 进行源代码控制。其工作流程为：代码从主仓库拉取（pull），修改后再推送（push）回主仓库。请根据您所使用的操作系统，安装相应版本的Git，以便继续完成后续步骤。

**注意**  
- 有关安装说明的参考信息，请查阅 [Git安装指南](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。如需了解Git基本命令，可参考《[Git手册](https://guides.github.com/introduction/git-handbook/)》（Git Handbook）或互联网上的其他相关资源。
- 代码库中包含一个`.git-blame-ignore-revs`文件，该文件可避免`git blame`命令的输出结果因某些提交（commit）而变得混乱 —— 这些提交仅用于代码格式调整，未涉及功能变更。关于如何使用此文件，请参考 [git blame 命令的官方文档](https://git-scm.com/docs/git-blame#Documentation/git-blame.txt---ignore-revltrevgt)。

## 获取代码

建议为开发需求维护一个 MicroPython 仓库的分支（fork）。获取源代码的步骤如下：

1. 分支（fork）仓库：https://github.com/micropython/micropython
2. 此时您将拥有一个分支，地址为：<https://github.com/<您的用户名>/micropython>。
3. 使用以下命令克隆（clone）该分支仓库：
   ```
   $ git clone https://github.com/<您的用户名>/micropython
   ```

然后，[配置远程仓库](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes)以方便参与 MicroPython 项目的协作。

配置上游（upstream）远程仓库：
```
$ cd micropython
$ git remote add upstream https://github.com/micropython/micropython
```

在分支仓库上配置 upstream 和 origin 是常见做法，有助于共享代码更改。您可以维护自己的映射关系，但建议将 origin 映射到您的分支仓库，将 upstream 映射到 MicroPython 的主仓库。

完成上述配置后，您的设置应类似于以下情况：
```
$ git remote -v
origin      https://github.com/<您的用户名>/micropython (fetch)
origin      https://github.com/<您的用户名>/micropython (push)
upstream    https://github.com/micropython/micropython (fetch)
upstream    https://github.com/micropython/micropython (push)
```

现在已获取了源代码副本。默认情况下，代码指向 master 分支。为便于后续开发，建议在开发分支上进行工作：
```
$ git checkout -b dev-branch
```

可以给这个分支起任何名称。每当切换到不同分支时，都需要重新编译 MicroPython。


### 编译并构建代码

编译 MicroPython 时，需要编译特定的移植版本（port），通常以特定的开发板（board）为目标。首先安装所需的依赖项，然后在成功编译和构建之前，需要先构建 MicroPython 交叉编译器。这一点特别适用于使用 Linux 进行编译的情况。Windows 系统的操作说明将在后面的章节中提供。

#### 获取依赖项

为 Linux 安装所需的依赖项：
```bash
$ sudo apt-get install build-essential libffi-dev git pkg-config
```

对于 stm32，需要 ARM 交叉编译器：
```bash
$ sudo apt-get install gcc-arm-none-eabi libnewlib-arm-none-eabi
```

有关最新详情，请参阅 [ARM GCC 工具链](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)。

此外还需要安装 Python。虽然目前仍支持 Python 2，但建议使用 Python 3。检查是否已安装 Python：
```bash
$ python3
Python 3.5.0 (default, Jul 17 2020, 14:04:10)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

所有支持的移植版都有不同的依赖要求，请参阅它们各自的 [readme 文件](https://github.com/micropython/micropython/tree/master/ports)。

#### 构建 MicroPython 交叉编译器

几乎所有版本都需要先构建 mpy-cross，以便对将包含在固件中的 Python 代码进行预编译：
```bash
$ cd mpy-cross
$ make
```

**注意**
- 请注意，`mpy-cross` 必须针对主机架构而非目标架构进行构建。

  如果构建成功，应该会看到类似以下的消息：
  ```bash
  LINK mpy-cross
    text         data     bss     dec       hex filename
  279328         776      880  280984     44998 mpy-cross
  ```
- 可以使用 `make -C mpy-cross` 命令在一个语句中构建交叉编译器，而无需切换到 `mpy-cross` 目录，否则，需要执行 `cd ..` 才能进行后续步骤。
 

#### 构建 MicroPython 的 Unix 版本

MicroPython 的 Unix 版本是可在 Linux、macOS 及其他类 Unix 操作系统上运行的版本。它对于 MicroPython 开发极具实用价值，因为它无需将代码部署到硬件设备上即可进行测试。在诸多方面，其功能与 CPython 的 python 可执行文件十分相似。

要为 Unix 版本构建程序，需先确保已按照“所需依赖项”章节中的详细说明，安装好所有与 Linux 相关的依赖项。请参考“获取依赖项”部分，确认已为其安装了全部必要依赖。同时，还需确保环境中已配置好可正常工作的 `gcc` 编译器和 `GNU make` 工具。以下示例基于 Ubuntu 20.04 系统，但其他类 Unix 系统在稍作调整后也应能正常使用：

```bash
$ gcc --version
gcc (Ubuntu 9.3.0-10ubuntu2) 9.3.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.then build:
```

随后执行构建操作：
```bash
$ cd ports/unix    # 进入 Unix 版本对应的代码目录
$ make submodules  # 初始化并更新子模块
$ make             # 执行构建
```

若 MicroPython 构建成功，将看到如下信息：
```bash
LINK micropython
   text         data     bss      dec     hex filename
 412033         5680    2496   420209   66971 micropython
```

运行构建生成的程序：
```bash
$ ./micropython
MicroPython v1.13-38-gc67012d-dirty on 2020-09-13; linux version
Use Ctrl-D to exit, Ctrl-E for paste mode
>>> print("hello world")
hello world
>>>
```


#### 构建 Windows 版本

Windows 版本包含一个 Visual Studio 项目文件 `micropython.vcxproj`，可通过该文件构建 `micropython.exe` 可执行文件。

既可以在 Visual Studio 中打开该项目文件进行构建，也可以使用 `msbuild` 工具通过命令行构建。此外，还可借助 MinGW 构建 —— 既支持在 Windows 系统中结合 Cygwin 环境构建，也支持在 Linux 系统中构建。有关更多信息，请参阅 [Windows 版本文档](https://github.com/micropython/micropython/tree/master/ports/windows)。  


#### 构建 STM32 版本

与 Unix 版本类似，需要先按照“获取依赖项”章节中的详细说明安装部分必要依赖，然后执行以下命令进行构建：
```bash
$ cd ports/stm32   # 进入 STM32 端口的代码目录
$ make submodules  # 初始化并更新子模块
$ make             # 执行构建
```

有关固件烧录的更多详细信息，请参考 [STM32 文档](https://github.com/micropython/micropython/tree/master/ports/stm32)。

**注意**  
请参考“获取依赖项”部分，确保已安装所有必要依赖。需要使用交叉编译器，且 `arm-none-eabi-gcc` 编译器需处于系统的 `$PATH` 环境变量中，或通过 `CROSS_COMPILE` 环境变量指定，或者在 `make` 命令的命令行参数中指定。

也可以指定要针对的开发板型号（构建特定开发板的固件），命令如下：
```bash
$ cd ports/stm32                      # 进入 STM32 端口的代码目录
$ make BOARD=<开发板型号> submodules  # 指定开发板并更新子模块
$ make BOARD=<开发板型号>             # 针对指定开发板执行构建
```

可支持的开发板列表请查看 [ports/stm32/boards](https://github.com/micropython/micropython/tree/master/ports/stm32/boards) 目录，例如 "PYBV11" 或 "NUCLEO_WB55"。


### 构建文档

MicroPython 文档通过 `Sphinx` 生成。如果已安装 Python，可通过 pip 工具安装 `Sphinx`。建议在**虚拟环境**中进行安装（避免依赖冲突），操作步骤如下：
```bash
$ python3 -m venv env      # 创建名为“env”的 Python 虚拟环境
$ source env/bin/activate  # 激活虚拟环境（Linux/macOS 系统命令）
$ pip install -r docs/requirements.txt  # 安装文档构建所需依赖
```

进入文档所在目录：
```bash
$ cd docs  # 切换到“docs”文件夹（存放文档源文件的目录）
```

构建文档（生成 HTML 格式）：
```bash
$ make html  # 执行构建命令，生成 HTML 格式的文档
```

在浏览器中打开 `docs/build/html/index.html` 文件，即可在本地查看构建完成的文档。如果需将文档部署到 Read the Docs（在线文档托管平台），请参考[相关文档](https://docs.readthedocs.io/en/stable/intro/import-guide.html)了解具体的导入流程。


### 运行测试

若要在 Unix 版本上运行测试套件中的所有测试，可使用以下命令：
```bash
$ cd ports/unix  # 进入 Unix 端口的代码目录
$ make test      # 执行所有测试
```

若要在通过 USB 连接的开发板/设备上运行指定测试，可使用以下命令：
```bash
$ cd tests                        # 进入测试代码所在目录
$ ./run-tests.py -t /dev/ttyACM0  # 通过 USB （/dev/ttyACM0）在目标设备上运行指定测试
```

另可参考“编写测试”（Writing tests）相关内容。


### 目录结构

在了解特定实现细节的位置时，有几个目录需要重点关注。以下是对源代码中顶层文件夹的详细解析：

- **py**  
  包含编译器、运行时环境以及核心库的实现代码。

- **mpy-cross**  
  存放 MicroPython 交叉编译器，该编译器用于将 Python 脚本预编译为字节码。

- **ports**  
  包含所有受支持移植版对应的 MicroPython 版本代码。

- **lib**  
  存放所有移植版可共用的底层 C 库，其中大部分是第三方库。

- **drivers**  
  包含特定硬件的驱动程序，这些驱动设计为可在多个移植版间通用。

- **extmod**  
  包含更多非核心模块的 C 语言实现代码（非核心模块指不包含在 MicroPython 核心功能中、按需使用的扩展模块）。

- **docs**  
  存放标准文档的源文件，在线文档可通过 https://docs.micropython.org/ 访问。

- **tests**  
  包含测试套件的实现代码，用于验证 MicroPython 功能的正确性。

- **tools**  
  包含构建流程、持续集成（CI）过程中使用的脚本，以及用户工具（如串口交互工具 `pyboard.py`、远程设备管理工具 `mpremote`）。

- **examples**  
  包含示例代码，涵盖“将 MicroPython 作为库使用”以及“原生模块开发”等场景的演示。
