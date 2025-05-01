# 编译esp8266源码

esp8266同样需要先安装独立的编译环境。首先安装需要依赖软件。
```
sudo apt-get install make unrar-free autoconf automake libtool gcc g++ gperf \
flex bison texinfo gawk libncurses-dev libexpat1-dev python-dev-is-python2\
python-is-python2 sed git unzip bash help2man wget bzip2
```

然后下载 esp-open-sdk仓库：
```
git clone --recursive https://github.com/pfalcon/esp-open-sdk.git
```

再更新一下子模块：
```
cd esp-open-sdk
git submodule update --init --recursive
```

完成上述步骤后，就可以在esp-open-sdk目录下构建编译所需的工具链。
```
make
```

## 编译时常见问题和解决方法
和esp32工具链构建一样，esp8266的工具链构建需要从github上下载很多文件，因此构建时间会受到网络速度和稳定性影响。在构建时，可能会因为一些脚本的历史比较悠久，其中部分文件因为版本升级或下载地址失效，造成构建失败，这时可以打开文件 `esp-open-sdk\crosstool-NG\build.log`，查看构建记录，通常在文件最后记录了失败的原因。下面是常见的几个问题和解决方法：

- **提示bash版本错误**
如果在构建时遇到 bash 版本问题，如：
```
checking for bash >= 3.1... no
configure: error: could not find bash >= 3.1
make[1]: *** [../Makefile:149: _ct-ng] Error 1
make[1]: Leaving directory '/home/free/esp-open-sdk/crosstool-NG'
make: *** [Makefile:145: crosstool-NG/ct-ng] Error 2
```
	
用nano或vi等编辑器打开文件esp-open-sdk/crosstool-NG/configure.ac，找到下面部分：
```
|$EGREP '^GNU bash, version (3\textbackslash.[1-9]|4)')
```
	
将它改为下面方式，取消bash版本限制即可正常构建。
```
|$EGREP '^GNU bash, version ([0-9\.]+)')
```


- **isl-0.14 无法下载**
这是因为原有下载地址失效造成。修改文件 `esp-open-sdk\crosstool-NG\lib\crosstool-ng-1.22.0-60-g37b07f6f-dirty\scripts\build\companion_libs\121-isl.sh`，将里面的 `http://isl.gforge.inria.fr` 改为 `https://libisl.sourceforge.io`。

- **expat-2.1.0无法下载**
这是因为expat版本升级造成原有地址失效。修改文件 `esp-open-sdk\crosstool-NG\config\companion_libs\expat.in`，将里面的 `2.1.0` 改为 `2.5.0`（expat经常更新，目前最新版是2.5.0，如果以后也2.5.0不能下载了，请到 `https://sourceforge.net/projects/expat/` 查看最新版本并更新版本号）。

- **无法下载newlib-2.0.0**
在终端下运行命令 `curl -o ./crosstool-NG/.build/tarballs/newlib-2.0.0.tar.gz ftp://sourceware.org/pub/newlib/newlib-2.0.0.tar.gz`，手动下载newlib-2.0.0并保存到构建临时目录。

完成构建后，生成的工具链文件保存在 `esp-open-sdk/xtensa-lx106-elf` 目录中，需要将 `esp-open-sdk/xtensa-lx106-elf/bin` 添加到环境变量中。可以编辑 .bashrc 文件进行添加或者用下面命令添加：
```
echo "export PATH=\$PATH:~/esp-open-sdk/xtensa-lx106-elf/bin">> ~/.bashrc
```

构建 esp-open-sdk需要的步骤较多，时间也比较长。如果不想自己构建，也可以从github下载已经构建好的工具链直接使用。
https://github.com/ChrisMacGregor/esp-open-sdk/releases

设置好编译环境后，就可以编译esp8266源码了：
```
make -C ports/esp8266 -j8
```
