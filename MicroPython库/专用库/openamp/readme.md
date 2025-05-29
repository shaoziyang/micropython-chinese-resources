
# openamp（标准非对称多处理 (AMP) 支持）

`openamp` 模块为 MicroPython 提供了标准的处理器间通信基础设施。该模块处理 OpenAMP 的所有细节，例如设置共享资源表、初始化虚拟环（vring）等。它通过 `Endpoint` 类提供使用 RPMsg 总线基础设施的 API，并通过 `RemoteProc` 类提供处理器生命周期管理（LCM）支持，例如加载固件、启动和停止远程内核。  

**示例用法：**  
```python  
import openamp  

def ept_recv_callback(src, data):  
    print("Received message on endpoint", data)  

# 创建新的 RPMsg 端点以与远程内核通信  
ept = openamp.Endpoint("vuart-channel", callback=ept_recv_callback)  

# 创建 RemoteProc 对象，加载其固件并启动  
rproc = openamp.RemoteProc("virtual_uart.elf")  # 或入口点地址（如  0x081E0000）  
rproc.start()  

while True:  
    if ept.is_ready():  
        ept.send("data")  
```  

### 函数  

- openamp.`new_service_callback`(ns_callback)

  设置新服务回调函数。

  `ns_callback` 参数是一个函数，当远程处理器宣告新服务时会调用该函数。此时，主机处理器可以选择创建所宣告的端点（如果支持该服务），或忽略（如果不支持）。如果未设置此函数，主机处理器应先在本地注册端点，当远程宣告服务时会自动绑定。  


### class Endpoint

- class openamp.`Endpoint`(name, callback, src=ENDPOINT_ADDR_ANY, dest=ENDPOINT_ADDR_ANY)

  构造新的 RPMsg 端点。端点是两个内核之间的双向通信通道。

  **参数：**
  - `name`：端点名称。
  - `callback`：当端点接收数据时调用的函数，参数为远程端点的源地址和以字节引用传递的数据。
  - `src`：端点源地址。若未提供，库会为端点分配一个地址。
  - `dest`：端点目标地址。如果通过 `new_service_callback` 创建端点，必须提供此参数且需与远程端点的源地址匹配。如果端点在宣告前本地注册，目标地址会在绑定时由库分配。
<br><br>

- Endpoint.`deinit`()

  销毁端点并释放所有资源。
<br><br>

- Endpoint.`is_ready`()

  如果端点已准备好发送数据（即已分配源地址和目标地址），返回 `True`。
<br><br>

- Endpoint.`send`(src=-1, dest=-1, timeout=-1)

  通过该端点向远程处理器发送消息。

  **参数：**  
  - `dest`：消息的目标端点地址。若未提供，使用端点绑定的目标地址。
  - `timeout`：等待可用缓冲区的时间（毫秒）。默认情况下函数为阻塞模式。
  - `src`：消息的源端点地址。若未提供，使用端点绑定的源地址。


### class RemoteProc

- class openamp.`RemoteProc`(entry)

  `RemoteProc` 对象提供处理器生命周期管理（LCM）支持，例如加载固件、启动和停止远程内核。  
  `entry` 参数可以是固件镜像的路径（此时固件从文件加载到目标内存），或入口点地址（此时固件必须已加载到给定地址）。
<br><br>

- RemoteProc.`start`()

  启动远程处理器。
<br><br>

- RemoteProc.`stop`()

  停止远程处理器。具体行为因硬件而异。例如，在 STM32H7 上无法停止后重启 Cortex-M4 内核，因此调用此函数会执行系统完全复位。
<br><br>

- RemoteProc.`shutdown`()

  关闭远程处理器并释放所有资源。具体行为因硬件而异，通常会禁用远程内核的电源和时钟。此函数也用作终结器（即当 `RemoteProc` 对象被回收时调用）。注意，在 STM32H7 上，停止后无法重启 Cortex-M4 内核，因此调用此函数会执行系统完全复位。
