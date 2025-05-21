# STM32 不自动载入 sdcard

在默认情况下，stm32 在启动时，会自动选择文件系统。如果没有插入 sdcard，将使用内部 flash 作为主文件系统启动；如果插入了 sdcard，会自动挂载 sdcard，并作为主文件系统。

这样虽然很方便，但是有时也会带来一些问题。比如我们将程序放在 flash 中，而 sdcard 只是用于储存数据，这时一旦复位就会从 sdcard 开始运行程序而不是从 flash 运行程序。

如果需要在插入 sdcard 的情况下从 flash 启动，可以在 flash 上创建一个空文件，文件名是 `/flash/SKIPSD`，这样上电/复位后就会从 flash 启动（此时 sdcard 不会被挂载，但后续仍可在程序中使用 vfs.mount 挂载并使用）。