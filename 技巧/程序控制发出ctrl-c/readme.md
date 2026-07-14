# 程序控制发出 ctrl-c

在 micropython 中，可以在 repl 中通过 ctrl-c 终止当前运行的程序，这对于代码调试非常方便。某些情况下，我们也可以利用这一特性，在用户程序中自动发送 ctrl-c，停止当前程序。

```python
# 用户程序

if 条件 == True:
    raise KeyboardInterrupt
```

如果程序中捕捉了异常，可能会造成 ctrl-c 不能直接退出，这时需要单独捕捉 KeyboardInterrupt 异常。
```python
try:
    # 用户程序

except KeyboardInterrupt:
    # 通过 break 等方式退出
except Exception as e:
    print(e)
```