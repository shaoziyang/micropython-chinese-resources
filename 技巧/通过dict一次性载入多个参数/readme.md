# 通过 dict 一次性载入多个参数

在编写程序时，我们通常会将一些参数放在一个单独的配置文件，在需要时载入配置文件，这样既方便使用，也容易修改。但是如果参数数量较多，或者需要多组不同配置参数时，就显得不够方便，也不容易维护。下面使用 class 和 dict 定义和保存参数，

首先创建一个配置文件（例如 CONFIG.py），在文件中设置一个 dict，里面保存配置参数。
```python
# 参数配置1
cfg_1 = {
    "v1":215,
    "v2":"123",
    "v3":[1,2,3,4]
}

# 参数配置2
cfg_2 = {
    "v1":15,
    "v2":"abc",
    "v3":[1,4,[1,2]]
}
```

然后在主程序中创建一个 class，class 的结构和 dict 相同。再通过遍历 class 就可以非常方便的将参数从 dict 中复制到 class。
```python
import sys
import CONFIG

class VAR:
    v1 = 0
    v2 = ""
    v3 = []

def loadcfg(class_var, dict_var):
    for attr in dir(class_var):
        if not attr.startswith('__') and attr in dict_var:
            setattr(class_var, attr, dict_var[attr])

def printcfg(class_var):
    for k,v in class_var.__dict__.items():
        if not k.startswith('__'):
            print(f'{k}: {v}')

cfg = VAR

print('load congif 1')
loadcfg(cfg, CONFIG.cfg_1)
printcfg(cfg)

print('load congif 2')
loadcfg(cfg, CONFIG.cfg_2)
printcfg(cfg)
```
