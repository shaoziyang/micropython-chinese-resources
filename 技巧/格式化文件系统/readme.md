# 格式化文件系统

- ESP8266 / ESP32
	
  ```py
  import vfs

  vfs.umount('/')
  vfs.VfsLfs2.mkfs(bdev)
  vfs.mount(bdev, '/')
  ```

- stm32

  ```py
  import vfs, pyb

  vfs.umount('/flash')
  vfs.VfsFat.mkfs(pyb.Flash(start=0))
  vfs.mount(pyb.Flash(start=0), '/flash')
  vfs.chdir('/flash')
  ```

- rp2
  ```py
  import vfs, rp2

  bdev = rp2.Flash()
  vfs.VfsLfs2.mkfs(bdev, progsize=256)
  vf = vfs.VfsLfs2(bdev, progsize=256)
  vfs.mount(vf, "/")
  ```

注：
- 有时需要复位后新的文件系统才能生效。
- 在较低版本固件中，没有 `vfs` 模块，需要使用 `os` 模块。