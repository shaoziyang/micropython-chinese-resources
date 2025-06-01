# 去掉支持乌克兰图标

Thonny 的 4.0 以后版本中，在菜单和工具链加入了支持乌克兰按钮，虽说不影响用，但是看着总有些不顺眼。Thonny 软件是用 python 开发的，而且提供了源码，因此去掉这个按钮也很容易。   
   
1. 找到软件的安装目录   
    - 如果使用默认安装，路径是：`C:\Program Files (x86)\Thonny\`。   
    - 如果是便携版本，路径就是软件所在目录。   
2. 找到并打开软件安装目录下的文件： `Lib\site-packages\thonny\workbench.py`。   
3. 定位到 690 行左右，或者查找相应文字(`SupportUkraine`)，把以下代码段的内容注释掉（每行前面加上`#`）或者直接删除：   
    ```py
        self.add_command(
           "SupportUkraine",
           "help",
           tr("Support Ukraine"),
           self._support_ukraine,
           image="Ukraine",
           caption=tr("Support"),
           include_in_toolbar=False,
           group=101,
        )
    ```
4. 定位到790行左右，注释掉下面命令：   
    ```py
    self._init_support_ukraine_bar()
    ```
5. 保存文件，重新启动 Thonny 后就没有支持乌克兰的那个图标了，连菜单里也没有。  