# vfs（虚拟文件系统）

vfs 模块包含用于创建文件系统对象并在虚拟文件系统中挂载 / 卸载它们的函数。

## 文件系统挂载

大部分移植版都提供虚拟文件系统（VFS），并支持在该 VFS 中挂载多个 "真实 文件系统。文件系统对象可以挂载到 VFS 的根目录，或根目录下的子目录中。这使得 Python 程序能够对文件系统进行动态灵活的配置。具备此功能的移植版本提供 `mount()` 和 `umount()` 函数，以及多个 VFS 类表示的各种文件系统接口。

- vfs.`mount`(fsobj, mount_point, *, readonly)

  将文件系统对象 `fsobj` 挂载到由 `mount_point` 字符串指定的 VFS 位置。`fsobj` 可以是具有 `mount()` 方法的 VFS 对象，或块设备。如果是块设备，则会自动检测文件系统类型（若无法识别文件系统，则会引发异常）。`mount_point` 可以是 '/'（将 `fsobj` 挂载到根目录）或 '/<name>'（挂载到根目录下的子目录）。

  如果 `readonly` 为 `True`，则文件系统以只读模式挂载。

  挂载过程中会调用文件系统对象的 `mount()` 方法。如果 `mount_point` 已被挂载，将引发 `OSError(EPERM)` 异常。
<br><br>

- vfs.`mount`()

  如果不向 `mount()` 传递任何参数，则返回一个元组列表，代表所有活动的挂载点。

  返回列表的格式为 `[(fsobj, mount_point), …]`。
<br><br>

- vfs.`umount`(mount_point)

  卸载文件系统。`mount_point` 可以是指定挂载位置的字符串，或先前已挂载的文件系统对象。卸载过程中会调用文件系统对象的 `umount()` 方法。

  如果未找到 `mount_point`，将引发 `OSError(EINVAL)` 异常。
<br><br>

- class vfs.`VfsFat`(block_dev)

  创建一个使用 FAT 文件系统格式的文件系统对象。FAT 文件系统的存储由 `block_dev` 提供。通过此构造函数创建的对象可使用 `mount()` 进行挂载。

  - static `mkfs`(block_dev)

    在 `block_dev` 上构建 FAT 文件系统。


- class vfs.`VfsLfs1`(block_dev, readsize=32, progsize=32, lookahead=32)

  创建一个使用 [littlefs v1 文件系统格式](https://github.com/ARMmbed/littlefs/tree/v1)的文件系统对象。littlefs 文件系统的存储由**block_dev**提供，该块设备必须支持扩展接口。通过此构造函数创建的对象可使用 `mount()` 进行挂载。

  - static mkfs(block_dev, readsize=32, progsize=32, lookahead=32)

    在 **block_dev** 上构建 Lfs1 文件系统。

    **注意**

    有报告称 littlefs v1 在某些情况下会出现故障，详细信息请参见 [littlefs问题347](https://github.com/ARMmbed/littlefs/issues/347)。


- class vfs.`VfsLfs2`(block_dev, readsize=32, progsize=32, lookahead=32, mtime=True)

  创建一个使用 [littlefs v2 文件系统格式](https://github.com/ARMmbed/littlefs)的文件系统对象。littlefs 文件系统的存储由**block_dev**提供，该块设备必须支持扩展接口。通过此构造函数创建的对象可使用 `mount()` 进行挂载。

  **mtime参数**用于启用文件的修改时间戳（通过littlefs属性存储）。该选项可在每次挂载时单独启用或禁用，且时间戳仅在 `mtime` 启用时才会添加或更新，否则保持不变。不含时间戳的 littlefs v2 文件系统无需重新格式化即可正常工作，且现有文件在以写入模式打开时会自动添加时间戳。当 mtime 启用时，对无时间戳的文件调用 `os.stat()` 时时间戳属性将返回为0。

  - static `mkfs`(block_dev, readsize=32, progsize=32, lookahead=32)

    在**block_dev**上构建Lfs2文件系统。

    **注意**

    有报告称 littlefs v2 在某些情况下会出现故障，详细信息请参见 [littlefs问题295](https://github.com/ARMmbed/littlefs/issues/295)。


- class vfs.`VfsPosix`(root=None)

  创建一个访问主机 POSIX 文件系统的文件系统对象。  若指定 `root` 参数，则应为主机文件系统中用作 VfsPosix 对象根目录的路径；若未指定，则使用主机文件系统的当前目录。


## 块设备

块设备是实现块协议的对象，它使设备能够支持 MicroPython 文件系统。物理硬件由用户定义的类表示。`AbstractBlockDev` 类是此类的模板：MicroPython 实际并未提供该类，但实际的块设备类必须实现后面描述的方法。

此类的具体实现通常允许访问硬件的类似内存的功能（如闪存）。块设备可格式化为任何支持的文件系统，并使用 `os` 方法挂载。


### 简单接口与扩展接口

为支持多种用例，`readblocks` 和 `writeblocks` 方法有两种兼容的签名。给定的块设备可实现其中一种形式，或同时实现两种。第二种形式（带 `offset` 参数）称为**扩展接口**。

某些需要更精细控制写入操作的文件系统（如 littlefs），在不擦除的情况下写入子块区域，可能要求块设备支持扩展接口。

- class vfs.`AbstractBlockDev`(...)

  构造一个块设备对象。构造函数的参数取决于具体的块设备。

  - `readblocks`(block_num, buf)
  - `readblocks`(block_num, buf, offset)

    第一种形式读取对齐的多个块。从索引 `block_num` 指定的块开始，将设备中的块读取到 `buf`（字节数组）中。要读取的块数由 `buf` 的长度决定，该长度必须是块大小的倍数。
    第二种形式允许在块内的任意位置读取任意长度的数据。从块索引 `block_num` 和该块内的字节偏移量 `offset` 开始，读取设备数据到 `buf`（字节数组）中。要读取的字节数由 `buf` 的长度决定。
<br><br>

  - `writeblocks`(block_num, buf)
  - `writeblocks`(block_num, buf, offset)

    第一种形式写入对齐的多个块，且要求在写入前（如有必要）先擦除目标块。从索引 `block_num` 指定的块开始，将 `buf`（字节数组）中的块写入设备。要写入的块数由 `buf` 的长度决定，该长度必须是块大小的倍数。
    第二种形式允许在块内的任意位置写入任意长度的数据。仅修改要写入的字节，调用此方法的代码必须确保通过先前的 `ioctl` 调用擦除相关块。从块索引 `block_num` 和该块内的字节偏移量 `offset` 开始，将 `buf`（字节数组）中的数据写入设备。要写入的字节数由 `buf` 的长度决定。

    注意如果指定了 `offset` 参数，实现就必须永不隐式擦除块，即使 `offset` 为零。

  - `ioctl`(op, arg)

    控制块设备并查询其参数。要执行的操作由 `op` 指定，`op` 为以下整数之一：
    - **1** – 初始化设备（未使用 `arg`）
    - **2** – 关闭设备（未使用 `arg`）
    - **3** – 同步设备（未使用 `arg`）
    - **4** – 获取块数，返回一个整数（未使用 `arg`）
    - **5** – 获取块中的字节数，返回一个整数；若返回 `None`，则使用默认值 512（未使用 `arg`）
    - **6** – 擦除块，`arg` 为要擦除的块号

    最低要求必须拦截 `ioctl(4, ...)`；对于 littlefs，还必须拦截 `ioctl(6, ...)`。其他操作的需求取决于硬件。

    在调用 `writeblocks(block, ...)` 之前，littlefs 会先调用 `ioctl(6, block)`。这使设备驱动程序能够在写入前（若硬件要求）擦除块。驱动程序也可以拦截 `ioctl(6, block)` 并返回0（成功），此时驱动程序需自行负责检测擦除需求。

    除非另有说明，`ioctl(op, arg)` 可返回 `None`。因此，实现可以忽略未使用的 `op` 值。当拦截 `op` 时，操作 4 和 5 的返回值如上所述。其他操作成功时应返回 0，失败时返回非零值（返回值为 OSEerrno 代码）。

  
