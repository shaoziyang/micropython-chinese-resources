# jumping-jerboa

一个简单有趣的 python 和 circuitpython 项目，通过 USB 将计算机的鼠标和键盘传输到另一台计算机上。

**基本原理**
* pygame 脚本捕获本地计算机上的鼠标和键盘操作。
* 映射脚本将这些操作从pygame转为circuitpython值。
* pyserial脚本将这些动作作为二进制数据发送到USB-TTL UART模块。
* Raspberry Pi Pico接收UART数据包。
* Raspberry Pi Pico解码数据包，并在远程计算机上执行鼠标/键盘操作。

```
                                 LOCAL                                             
                                COMPUTER                              REMOTE       
      _______________________________________                        COMPUTER      
     |                          pygame to    |              _____________________  
     |   pygame capture  ---> circuitpython  |             |                     | 
     | (mouse & keyboard)       key mapper   |             |                     | 
     |                              |        |             |                     | 
     |     Create Binary            |        |             |                     | 
     |     Communication  <---------'        |             |  _____              | 
     |     Data Packets                      |             | | USB |             | 
     |           |                           |             |_|_____|_____________| 
     |           | Send Over                 |                  ^                  
     |           |     USB                   |                  |                  
     |         __V__                         |                  | USB              
     |        | USB |                        |                  | HID              
     |________|_____|________________________|                  | Commands         
                 |                                         _____|_____             
                 |                                        |   |USB|   |            
              ___V____                                    |   '---'   |            
             | |USB|  |                                   |     |     |            
             | '---'  |                                   |   decode  |            
             |        |  Binary Communication Over Uart   |     |     |            
             |      TX|---------------------------------->|RX(GP1)    |            
             |      RX|<----------------------------------|TX(GP0)    |            
             |     GND|-----------------------------------|GND        |            
             |________|                                   |___________|            
              TTL UART                                     RASPBERRY PI            
              TO USB                                       PICO (rp2040)           
              CONVERTER                                                            
		                                                                       
```


## 相关链接

- [github 仓库](https://github.com/rvl13/jumping-jerboa)
- [hackster.io](https://www.hackster.io/RVLAD/jumping-jerboa-transfer-mouse-keyboard-to-other-computer-68a134)
