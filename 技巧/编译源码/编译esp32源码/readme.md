# 编译esp32源码

esp32使用了Xtensa内核，它与arm不同，不能使用Arm GNU Toolchain，所以在编译esp32源码前，需要先安装ESP-IDF（物联网开发框架，也就是SDK）。ESP-IDF包括开发ESP32时所需的库、RTOS、构建固件所需的环境和工具链。因为ESP-IDF更新很快，所以一个版本的 micropython 通常仅支持某些特定版本的 ESP-IDF，当micropython 升级后，可能需要更新 ESP-IDF 才能正确编译源码。当前版本的micropython（v1.21）就仅支持ESP-IDF 5.0.2，如果更新到其它版本，请参考源码中相关说明，使用相应版本的ESP-IDF。

编译esp32主要的困难在于编译工具链的安装和设置，一个原因是ESP-IDF更新较快，需要经常升级；另外安装编译工具时需要从github上拉取相当多的文件，对网络速度和稳定性有一定要求。安装过程中如果因为网络问题失败，通常可以从失败的位置继续安装，或者改用乐鑫的服务器上下载。

下面以 ESP-IDF 5.5.1 为例进行说明，其它版本安装方式类似，具体可以参考micropython的文档说明。

首先用git下载ESP-IDF文件（文件较大，可能需要较长时间）。
```
git clone -b v5.5.1 --recursive https://github.com/espressif/esp-idf.git
```

如果已经安装过ESP-IDF，可以通过git更新到 v5.5.1版本。
```
cd esp-idf
git checkout v5.5.1
git submodule update --init --recursive
```

构建前还需要安装python3-venv，否则构建时会提示没有安装并退出。
```
apt install python3-venv
```

成功拉取所有文件后，就可以构建ESP-IDF编译器。在 ESP-IDF目录下运行 install.sh，就可以自动开始构建。install.sh 命令只需要运行一次，只有在ESP-IDF更新后，才需要再次运行，更新工具链。构建过程中会从 github 下载数百兆相关文件，因此这个过程会受到网络很大影响。构建过程会占用数G的磁盘空间，因此需要预留出足够大的磁盘空间，在虚拟机中进行编译时要特别注意这个问题。
```
cd esp-idf
./install.sh
```

如果在构建过程中，从github拉取文件速度太慢，可以修改下载服务器，这样会极大加快下载速度。如：
```
cd esp-idf
export IDF_GITHUB_ASSETS="dl.espressif.cn/github_assets"
./install.sh
```

完成构建后，每次打开终端或Linux子系统开始编译前都需要运行一次ESP-IDF下 export.sh命令，设置编译环境，否则会提示找不到编译器。
```
source export.sh
```

设置好编译环境，就可以编译esp32源码了，方法和stm32基本是一样的。
```
make -C ports/esp32
make -C ports/esp32 BOARD=LOLIN_S2_MINI -j4
```

除了自行安装 esp-idf 工具链外，还可以从 github 下载其他人共享的 esp-idf 工具链。此外还可以通过 docker 方式，直接使用其他人创建好的编译环境，这里就不展开介绍了。
