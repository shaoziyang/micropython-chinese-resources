# 查看磁盘大小和剩余空间

通过 `os.statvfs()`，可以获取文件系统的磁盘空间大小和剩余空间大小。

**磁盘大小**
```python
def disksize(d=''):
    try:
        a = os.statvfs(d)
        return a[0]*a[2]
    except:
        return 0
```

**剩余空间**
```python
def diskfree(d=''):
    try:
        a = os.statvfs(d)
        return a[0]*a[3]
    except:
        return 0
```
