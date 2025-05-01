# 下载micropython源码

要编译源码，就需要先下载源码。下载源码有两种方式：zip或者git，两种方式各有优缺点，大家可以根据情况灵活选择其中一种方式。

- git方式就是用git命令将github上的micropython项目文件克隆下来，这种方式最灵活，适合底层开发者。使用git管理文件，可以随时同步官方代码的更新，追溯源码变化，更新时也只需要下载更新的文件，如果希望为开源做贡献还能提交代码（(Pull Request，简称PR），缺点是使用复杂，占用空间大，除了micropython本身还需要下载源码引用的第三方仓库。
- zip方式就是下载micropython的 releases 版本，里面已经打包了所有需要的源码文件（包括第三方仓库，但不包括编译需要的工具链），可以直接编译。缺点是每次都需要重新下载源码，更新较慢。

## git 下载：
先从github上克隆源码。
```
git clone https://github.com/micropython/micropython.git
```

使用git方式下载后，通常还需要更新引用的其它子模块（第三方仓库）。因为micropython引用了较多第三方模块，拉取会需要一定的时间。如果一次没有成功，可以多尝试几次。
```
cd micropython
git submodule update —init
```

以后更新时，只需要在micropython目录中，使用git pull就可以更新，这种方法只下载改变的部分文件，速度比较快。例如：
```
cd micropython
git pull
```

## zip下载：
先下载源码，然后解压缩到一个文件夹中，例如下面将源码展开到micropython目录下。因为下载的源码压缩文件内部还有一级带版本号的子目录，这样不同版本就不会直接覆盖。
```
curl -L -o micropython.tar.xz https://github.com/micropython/micropython/releases/download/v1.21.0/micropython-1.21.0.tar.xz
mkdir micropython
tar vxf ./micropython.tar.xz -C micropython
```

上面是以v1.21.0版本为例，其它版本的地址会不同。

注：github访问有时会因为网络问题不稳定，造成下载速度非常慢甚至下载失败，可能需要多尝试几次，或在网络状况较好时再尝试。