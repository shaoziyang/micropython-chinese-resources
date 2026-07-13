# 自动判断系统

通过下面代码，可以自动识别当前运行的不同系统，如 micropython/circuitpython等。

```python
def get_sysplatform():
    platform = sys.version
    if 'MicroPython' in platform:
        return 'MicroPython'
    elif 'CircuitPython' in platform:
        return 'CircuitPython'
    else:
        return 'Other'     

SYS_PLATFORM = get_sysplatform()  
```