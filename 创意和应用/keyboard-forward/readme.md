# Keyboard Forward

📝 该项目旨在利用 BLE（低功耗蓝牙）微控制器作为 HID（人机接口设备）桥接，将电脑的键盘输入转发到蓝牙设备上。


## 工作原理：
* BLE 微控制器通过 USB 连接电脑，暴露一个 Web Serial 数据端口。
* 网页端 (index.html) 连接该串口，捕获键盘输入或发送文本流。
* 固件通过 usb_cdc.data 接收二进制按键指令。
* 微控制器将这些指令重放为 BLE 键盘信号，发送给配对的目标设备。

## 环境要求与设置
**硬件**
* 支持 BLE 的 MCU
  * 已在 Seeed Xiao nrf52840 上测试
* USB-C 线

**软件**
* CircuitPython 10
* CircuitPython libraries on the device:
  * adafruit_ble
  *	adafruit_hid
* Chromium 内核浏览器，如 Chrome、Edge 或 Brave

## 项目结构

```
keyboard-forward/
├── README.md
├── CLAUDE.md
├── design.md
├── index.html
└── firmware/
    ├── boot.py
    └── code.py
```

## 相关链接

* [github 仓库](https://github.com/urfdvw/keyboard-forward)
