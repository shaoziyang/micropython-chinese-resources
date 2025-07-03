# pyboard.py

在 micropython 的源码中的 tools 目录下，有一个文件 pyboard.py，可以用来通过串口控制 micropython 板，包括执行程序、复制文件等。实测复制大文件(400K)不死机，缺点是无进度提示。

**复制文件的用法（windows下）：**

`pyboard -d COM1 -f cp infile :outfile`
- `COM` 请替换为实际串口号
- `-f` 代表文件操作
- `infile` 代表PC上的文件
- `outfile` 代表复制到板子的文件，outfile 的冒号前需要有空格


**查看文件：**

`pyboard -d COM1 -f ls`
