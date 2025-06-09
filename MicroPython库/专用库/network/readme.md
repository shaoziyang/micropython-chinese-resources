# network（网络配置）

network 模块提供网络驱动和路由配置功能。要使用此模块，必须安装具备网络功能的 MicroPython 变体/版本。此模块中提供了特定硬件的网络驱动，用于配置硬件网络接口。配置好的接口所提供的网络服务可通过 socket 模块使用。

示例：
```py
# 连接/显示特定网络接口的IP配置
# 具体驱动示例见下文
import network
import time

nic = network.Driver(...)
if not nic.isconnected():
    nic.connect()
    print("等待连接...")
    while not nic.isconnected():
        time.sleep(1)
print(nic.ipconfig("addr4"))

# 现在可以像往常一样使用socket
import socket
addr = socket.getaddrinfo('micropython.org', 80)[0][-1]
s = socket.socket()
s.connect(addr)
s.send(b'GET / HTTP/1.1\r\nHost: micropython.org\r\n\r\n')
data = s.recv(1000)
s.close()
```

## 通用网络适配器接口

本节描述了 MicroPython 针对不同硬件端口实现的所有网络接口类的（隐含）抽象基类。这意味着 MicroPython 实际上并不提供 `AbstractNIC` 类，但后续章节中描述的任何实际 NIC 类都实现了此处描述的方法。

- class network.`AbstractNIC`(id=None, ...)

  实例化一个网络接口对象。参数取决于具体的网络接口。如果同一类型的接口有多个，第一个参数应为 `id`。
<br><br>

- AbstractNIC.`active`([ is_active ])

  如果传入布尔参数，则激活（"up"）或停用（"down"）网络接口。如果不提供参数，则查询当前状态。大多数其他方法要求接口处于激活状态（在未激活的接口上调用这些方法的行为未定义）。
<br><br>

- AbstractNIC.`connect`([ service_id, key=None, *, ... ])

  将接口连接到网络。此方法为可选方法，仅适用于并非"始终连接"的接口。如果不提供参数，则连接到默认（或唯一）的服务。如果提供单个参数，则该参数是要连接的服务的主要标识符。该参数可能会伴随一个访问该服务所需的密钥（密码）。根据网络介质类型和/或特定设备的不同，还可以有其他任意的仅限关键字参数。这些参数可用于：a) 指定替代的服务标识符类型；b) 提供额外的连接参数。对于不同的介质类型，有不同的预定义/推荐参数集，其中包括：
  - WiFi：使用 `bssid` 关键字连接到特定的 BSSID（MAC地址）
<br><br>

- AbstractNIC.`disconnect`()

  断开网络连接。
<br><br>

- AbstractNIC.`isconnected`()

  若已连接到网络则返回 `True`，否则返回 `False`。
<br><br>

- AbstractNIC.`scan`(*, ...)

  扫描可用的网络服务/连接，返回包含发现的服务参数的元组列表。
  
  对于不同的网络介质，预定义/推荐的元组格式不同，例如：
  - **WiFi**：元组格式为`(ssid, bssid, channel, RSSI, security, hidden)`，某些设备可能包含特定的额外字段。
  
  该函数可接受额外的关键字参数以过滤扫描结果（如按特定服务、频道或服务类型扫描），并影响扫描时长等参数。参数名称应尽可能与 `connect()` 方法中的参数一致。
<br><br>

- AbstractNIC.`status`([ param ])

  查询接口的动态状态信息。无参数调用时，返回值描述网络链路状态；若传入参数 `param`，则应指定要检索的状态参数名称（字符串）。
  
  返回类型和值取决于网络介质/技术，可能支持的参数包括：
  - **WiFi STA 模式**：使用 `'rssi'` 获取接入点信号的 RSSI 值。
  - **WiFi AP 模式**：使用 `'stations'` 获取连接到 AP 的所有 STA 列表，列表元素为 `(MAC, RSSI)` 格式的元组。
<br><br>

- AbstractNIC.`ipconfig`('param')
- AbstractNIC.`ipconfig`(param=value, ...)

  获取或设置接口特定的 IP 配置参数。支持的参数如下（具体可用性取决于硬件和网络接口）：

  - `dhcp4`（True/False）：通过 DHCP 获取 IPv4 地址、网关和 DNS 服务器。此方法不会阻塞，等待地址获取，可通过只读属性`has_dhcp4`检查是否获取到地址。
  - `gw4`：获取/设置 IPv4 默认网关。
  - `dhcp6`（True/False）：通过无状态 DHCPv6 获取 DNS 服务器（当前不支持通过 DHCPv6 获取 IP 地址）。
  - `autoconf6`（True/False）：通过路由器通告的网络前缀获取无状态 IPv6 地址，可通过只读属性 `has_autoconf6` 检查是否获取到地址。
  - `addr4`（如 `192.168.0.4/24`）：获取当前 IPv4 地址和子网掩码（格式为 `(ip, subnet)`），无论地址获取方式如何。也可通过传入`(ip, subnet)`或 CIDR 格式字符串设置静态 IPv4 地址。
  - `addr6`（如 `fe80::1234:5678`）：获取当前 IPv6 地址列表（格式为 `(ip, state, preferred_lifetime, valid_lifetime)`），包含链路本地地址、SLAAC 地址和静态地址。其中： `preferred_lifetime` 和 `valid_lifetime` 表示每个 IPv6 地址的剩余有效和首选生存期，单位为秒。`state` 表示地址的当前状态：
    - `0x08-0x0f`：表示地址是暂定的，正在计算发送的探测数。
    - `0x10`：地址已废弃（但仍有效）。
    - `0x30`：地址为首选（且有效）。
    - `0x40`：地址重复，不可使用。
  可通过传入 IPv6 地址字符串（如`fe80::1234:5678`）设置静态IPv6地址。
<br><br>

- AbstractNIC.`ifconfig`([ (ip, subnet, gateway, dns) ])

  **注意**：此函数已弃用，建议使用`ipconfig()`。

  获取/设置 IP 层网络接口参数（IP地址、子网掩码、网关、DNS服务器）。无参数调用时返回包含上述信息的四元组；传入四元组可设置参数，例如：
  ```python  
  nic.ifconfig(('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))  
  ```
<br>

- AbstractNIC.`config`('param')
- AbstractNIC.`config`(param=value, ...)

  获取或设置通用网络接口参数，支持标准 IP 配置（`ipconfig()`处理）以外的额外参数（如网络特定或硬件特定参数）。
  
  这些包括网络特定参数和硬件特定参数。对于设置参数，应使用关键字参数语法，并且可以一次设置多个参数。查询时，参数名称应以字符串形式引用，一次只能查询一个参数：
  
  ```python  
  # 设置WiFi接入点名称（SSID）和频道  
  ap.config(ssid='My AP', channel=11)  

  # 查询参数，每次仅能查询一个参数  
  print(ap.config('ssid'))     # 查询SSID  
  print(ap.config('channel'))  # 查询频道  
  ```


## 特定网络类实现  

以下类实现了 AbstractNIC 接口，并提供了控制各种网络接口的方法。  


### class WLAN – 控制内置 WiFi 接口

此类为 WiFi 网络处理器提供驱动。示例用法：  

```python  
import network  

# 启用站点接口并连接到WiFi接入点  
nic = network.WLAN(network.WLAN.IF_STA)  
nic.active(True)  
nic.connect('你的-SSID', '你的-密钥')  
# 现在可以像往常一样使用套接字  
```  

#### 构造函数  

- class network.`WLAN`(interface_id)
  
  创建WLAN网络接口对象。支持的接口类型包括：  
  - `network.WLAN.IF_STA`：站点模式，即客户端，连接到上游 WiFi 接入点。 
  - `network.WLAN.IF_AP`：接入点模式，允许其他 WiFi 客户端连接。

  以下方法的可用性取决于接口类型。例如，仅站点模式接口可调用 `WLAN.connect()` 连接到接入点。  


#### 方法  

- WLAN.`active`([ is_active ])
  
  若传入布尔参数，则激活（"up"）或停用（"down"）网络接口；若无参数，则查询当前状态。大多数其他方法要求接口处于激活状态。  
<br><br>

- WLAN.`connect`(ssid=None, key=None, *, bssid=None)  

  使用指定密钥连接到指定无线网络。若提供`bssid`，则连接将限制为具有该MAC地址的接入点（此时必须同时指定`ssid`）。
<br><br>

- WLAN.`disconnect`()  

  断开与当前连接的无线网络的连接。
<br><br>
  
- WLAN.`scan`()  

  扫描可用的无线网络。若 WLAN 接口支持，隐藏网络（未广播SSID的网络）也会被扫描。  

  **仅站点模式接口支持扫描**。返回包含WiFi接入点信息的元组列表：  
  `(ssid, bssid, channel, RSSI, security, hidden)`  

  `bssid` 是接入点的硬件地址（二进制形式，返回为bytes对象），可使用 `binascii.hexlify()` 转换为ASCII格式。

  - `security` 取值：  
    - `0`：开放网络  
    - `1`：WEP  
    - `2`：WPA-PSK  
    - `3`：WPA2-PSK  
    - `4`：WPA/WPA2-PSK  
  - `hidden` 取值：  
    - `0`：可见
    - `1`：隐藏
<br><br>

- WLAN.`status`([ param ])

  返回无线连接的当前状态。  

  无参数调用时，返回值描述网络链路状态，可能的状态在 `network` 模块中定义为常量：  
  - `STAT_IDLE`：未连接且无活动  
  - `STAT_CONNECTING`：连接中  
  - `STAT_WRONG_PASSWORD`：因密码错误失败  
  - `STAT_NO_AP_FOUND`：因无接入点响应失败  
  - `STAT_CONNECT_FAIL`：因其他问题失败  
  - `STAT_GOT_IP`：连接成功

  传入参数 `param` 时（字符串），根据WiFi模式支持不同参数：  
  - **站点模式**：传入 `'rssi'` 返回信号强度值（格式因硬件而异，CC3200除外）。  
  - **接入点模式**：传入 `'stations'` 返回已连接的 WiFi 站点列表（格式因硬件而异，可能包含原始BSSID、IP地址或两者，CC3200除外）。
<br><br>

- WLAN.`isconnected`()  

  - **站点模式**：若连接到 WiFi 接入点且拥有有效 IP 地址，返回 `True`；否则返回 `False`。  
  - **接入点模式**：若有站点连接，返回 `True`；否则返回 `False`。
<br><br>

- WLAN.`ifconfig`( [ (ip, subnet, gateway, dns) ] )  

  获取/设置 IP 层网络接口参数（IP地址、子网掩码、网关、DNS服务器）。无参数调用时返回四元组；传入四元组可设置参数，例如：

  ```python  
  nic.ifconfig(('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))
  ```
<br>

- WLAN.`config`('param')
- WLAN.`config`(param=value, ...)

  获取或设置通用网络接口参数（支持标准 IP 配置以外的额外参数）。

  - **设置参数**：使用关键字参数语法，可一次性设置多个参数。
  - **查询参数**：传入参数名称字符串（每次仅能查询一个参数）。

  ```python  
  # 设置WiFi接入点名称（SSID）和频道  
  ap.config(ssid='我的AP', channel=11)  

  # 依次查询网络参数
  print(ap.config('ssid'))  # 查询SSID  
  print(ap.config('channel'))  # 查询频道  
  ```  

  下面是常用支持参数（具体参数可用性取决于网络技术类型、驱动和 MicroPython 移植版本）  

  | 参数         | 描述                                                                       |
  |--------------|--------------------------------------------------------------------------|
  | `mac`        | MAC地址（bytes类型）                                                         |
  | `ssid`       | WiFi接入点名称（字符串）                                                     |
  | `channel`    | WiFi频道（整数，部分端口仅在AP接口支持此参数）                                |
  | `hidden`     | 是否隐藏SSID（布尔值）                                                       |
  | `security`   | 支持的安全协议（枚举值，见模块常量）                                          |
  | `key`        | 接入密钥（字符串）                                                           |
  | `hostname`   | 发送给DHCP（STA模式）和mDNS（若支持）的主机名（已弃用，改用`network.hostname()`） |
  | `reconnects` | 重连尝试次数（整数，`0`=不尝试，`-1`=无限次）                                  |
  | `tx_power`   | 最大发射功率（dBm，整数或浮点数）                                             |
  | `pm`         | WiFi电源管理设置（取值见下方常量）                                  |

#### 常量  

- WLAN.PM_PERFORMANCE
- WLAN.PM_POWERSAVE  
- WLAN.PM_NONE

  用于 `WLAN.config(pm=...)` 的参数值。
  - `PM_PERFORMANCE`：启用电源管理，平衡省电与性能
  - `PM_POWERSAVE`：启用深度省电模式，牺牲部分性能
  - `PM_NONE`：禁用电源管理


### class WLANWiPy – WiPy 专用 WiFi 控制

**注意**: 此类是 WiPy 的非标准 WLAN 实现。在 WiPy 上可直接通过 `network.WLAN` 访问，但在以下文档中命名为 `network.WLANWiPy`，以区别于更通用的 `network.WLAN` 类。

此类为 WiPy 中的 WiFi 网络处理器提供驱动。示例用法：

```python
import network
import time

# 设置为工作站模式
wlan = network.WLAN(mode=WLAN.STA)
wlan.connect('your-ssid', auth=(WLAN.WPA2, 'your-key'))
while not wlan.isconnected():
    time.sleep_ms(50)
print(wlan.ipconfig("addr4"))

# 现在可以像往常一样使用 socket
...
```

#### 构造函数

- class network.`WLANWiPy`(id=0, ...)

  创建一个 WLAN 对象，并可选择配置它。有关配置参数，请参见 `init()`。

  **注意**: WLAN 构造函数具有特殊性：如果只提供 `id` 参数，它将返回已存在的 WLAN 实例而不重新配置。这是因为 WLAN 是 WiPy 的系统功能。如果已存在的实例未初始化，它将与其他构造函数一样，使用默认值进行初始化。

#### 方法

- WLANWiPy.`init`(mode, *, ssid, auth, channel, antenna)

  设置或获取 WiFi 网络处理器配置。参数包括：
  - `mode`：可以是 `WLAN.STA` 或 `WLAN.AP`。
  - `ssid`：WiFi 名称字符串，仅在 `mode` 为 `WLAN.AP` 时需要。
  - `auth`：元组 `(sec, key)`，其中 `sec` 可为 `None`、`WLAN.WEP`、`WLAN.WPA` 或 `WLAN.WPA2`，`key` 为网络密码字符串。若 `sec` 为 `WLAN.WEP`，`key` 必须是表示十六进制值的字符串（例如 `'ABC1DE45BF'`）。仅在 `mode` 为 `WLAN.AP` 时需要。
  - `channel`：1-11 范围内的数字，仅在 `mode` 为 `WLAN.AP` 时需要。
  - `antenna`：选择内置或外置天线，可为 `WLAN.INT_ANT` 或 `WLAN.EXT_ANT`。

  示例：
  ```python
  # 创建并配置为接入点
  wlan.init(mode=WLAN.AP, ssid='wipy-wlan', auth=(WLAN.WPA2, 'www.wipy.  io'), channel=7, antenna=WLAN.INT_ANT)
  
  # 或配置为工作站
  wlan.init(mode=WLAN.STA)
  ```

- WLANWiPy.`connect`(ssid, *, auth=None, bssid=None, timeout=None)

  使用给定的 SSID 和其他安全参数连接到 WiFi 接入点。
  - `auth`：元组 `(sec, key)`，含义同上。
  - `bssid`：要连接的 AP 的 MAC 地址，在存在多个同名 SSID 的 AP 时有用。
  - `timeout`：连接成功的最大等待时间（毫秒）。
<br><br>

- WLANWiPy.`scan`()

  执行网络扫描并返回包含 `(ssid, bssid, sec, channel, rssi)` 的命名元组列表。注意 `channel` 始终为 `None`，因为 WiPy 不提供此信息。
<br><br>

- WLANWiPy.`disconnect`()

  断开与 WiFi 接入点的连接。
<br><br>

- WLANWiPy.`isconnected`()

  在 STA 模式下：若已连接到 WiFi 接入点并获得有效 IP 地址，返回 `True`。在 AP 模式下：若有工作站连接，返回 `True`，否则返回 `False`。
<br><br>

- WLANWiPy.`ipconfig`('param')
- WLANWiPy.`ipconfig`(param=value, ...)

  参见 `AbstractNIC.ipconfig`。支持的参数有：`dhcp4`、`addr4`、`gw4`。
<br><br>

- WLANWiPy.`mode`( [mode] )

  获取或设置 WLAN 模式。
<br><br>

- WLANWiPy.`ssid`( [ssid] )

  获取或设置 AP 模式下的 SSID。
<br><br>

- WLANWiPy.`auth`( [auth] )

  获取或设置 AP 模式下的认证类型。
<br><br>

- WLANWiPy.`channel`( [channel] )

  获取或设置信道（仅适用于 AP 模式）。
<br><br>

- WLANWiPy.`antenna`( [antenna] )

  获取或设置天线类型（外置或内置）。
<br><br>

- WLANWiPy.`mac`( [mac_addr] )

  获取或设置包含 MAC 地址的 6 字节 `bytes` 对象。
<br><br>

- WLANWiPy.`irq`(*, handler, wake)

  创建在 `machine.SLEEP` 模式下发生 WLAN 事件时触发的回调。事件由 socket 活动或 WLAN 连接/断开触发。
  - `handler`：IRQ 触发时调用的函数。
  - `wake`：必须为 `machine.SLEEP`。  

  返回一个 IRQ 对象。

### 常量
- WLANWiPy.`STA`
- WLANWiPy.`AP`
  
  选择 WLAN 模式。
<br><br>

- WLANWiPy.`WEP`
- WLANWiPy.`WPA`
- WLANWiPy.`WPA2`
  
  选择网络安全类型。
<br><br>

- WLANWiPy.`INT_ANT`
- WLANWiPy.`EXT_ANT`

  选择天线类型。


### class WIZNET5K – 控制 WIZnet5x00 以太网模块

该类允许你控制基于 W5200 和 W5500 芯片组的 WIZnet5x00 以太网适配器。固件支持的特定芯片组在编译时通过 `MICROPY_PY_NETWORK_WIZNET5K` 选项选择。

**注意**: esp32 也可支持 WIZnet W5500 芯片组，但使用了 `network.LAN` 接口。  

**示例用法**：

```python
import network

# 初始化WIZNET5K网卡
nic = network.WIZNET5K(pyb.SPI(1), pyb.Pin.board.X5, pyb.Pin.board.X4)
print(nic.ipconfig("addr4"))   # 打印IP地址

# 之后可像通常一样使用socket
...
```

要使这个程序正常运行，WIZnet5x00 模块必须进行以下连接：
- **MOSI** 连接到 **X8**
- **MISO** 连接到 **X7**
- **SCLK** 连接到 **X6**
- **nSS** 连接到 **X5**（片选信号）
- **nRESET** 连接到 **X4**（复位信号）

也可以使用其他 SPI 总线和引脚作为 nSS 和 nRESET。

#### 构造函数

- class network.`WIZNET5K`(spi, pin_cs, pin_rst)

  创建一个 WIZNET5K 驱动对象，使用给定的 SPI 总线和引脚初始化 WIZnet5x00 模块，并返回 WIZNET5K 对象。

  **参数说明**：
  - `spi`：SPI 对象，表示 WIZnet5x00 连接的 SPI 总线（对应 MOSI、MISO 和 SCLK 引脚）。
  - `pin_cs`：Pin 对象，连接到 WIZnet5x00 的 nSS 引脚（片选信号）。
  - `pin_rst`：Pin 对象，连接到 WIZnet5x00 的 nRESET 引脚（复位信号）。

  这些对象将由驱动自动初始化，无需手动初始化。例如：
  ```python
  nic = network.WIZNET5K(pyb.SPI(1), pyb.Pin.board.X5, pyb.Pin.board.X4)
  ```

#### 方法

实现了 `AbstractNIC` 的大部分方法，具体文档见相关部分。新增方法如下：

- WIZNET5K.`regs`()  

  转储 WIZnet5x00 的寄存器值，用于调试。


### class LAN – 控制以太网模块

此类可用于控制以太网接口。PHY 硬件类型因开发板而异。

对于内置 LAN 支持的开发板，示例用法如下：
```
import network
nic = network.LAN(0)
print(nic.ipconfig("addr4"))

# 现在可以像往常一样使用socket
...
```

#### 构造函数

- class network.`LAN`(id, *,  phy_type=<board_default>, phy_addr=<board_default>, ref_clk_mode=<board_default>)

  创建一个 LAN 驱动对象，使用给定的 PHY 驱动名称初始化 LAN 模块，并返回 LAN 对象。
  
  参数说明：
  - `id`：以太网端口号，可选 0 或 1。
  - `phy_type`：PHY 驱动名称。大多数开发板需要使用板载 PHY，这也是默认设置。合适的值因端口而异。
  - `phy_addr`：指定 PHY 接口的地址。与 `phy_type` 一样，大多数开发板需要使用硬件预设值，该值为默认值。
  - `ref_clk_mode`：指定数据时钟是由以太网控制器提供还是由 PHY 接口提供。默认值与开发板匹配。如果设置为 `LAN.OUT` 或 `Pin.OUT` 或 `True`，则时钟由以太网控制器驱动；如果设置为 `LAN.IN` 或 `Pin.IN` 或 `False`，则时钟由 PHY 接口驱动。
  
  例如，对于 Seeed Arch Mix 开发板，可以使用：
  
  `nic = LAN(0, phy_type=LAN.PHY_LAN8720, phy_addr=1, ref_clk_mode=Pin.IN)`
  
  **注意**, 在 esp32 上，构造函数需要不同的参数。

#### 方法

- LAN.`active`([ state ])

  带参数时，如果 `state` 为 `True`，则激活接口；否则禁用接口。不带参数时，返回当前状态。
<br><br>

- LAN.`isconnected`()

  如果物理以太网链路已连接且正常工作，则返回 `True`；否则返回 `False`。
<br><br>

- LAN.`status`()

  返回LAN状态。
<br><br>

- LAN.`ifconfig`([ (ip, subnet, gateway, dns) ])

  获取/设置IP地址、子网掩码、网关和 DNS。
  
  不带参数调用时，此方法返回包含上述信息的四元组。要设置参数，请传递包含所需信息的四元组。例如：

  ```
  nic.ifconfig(('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))
  ```
<br><br>

- LAN.config(config_parameters)

  设置或获取 LAN 接口的参数。唯一可以获取的参数是 MAC 地址，使用方法如下：

  `mac = LAN.config("mac")`
  
  可以设置的参数有：
  - `trace=n`：设置跟踪级别；合适的值为：
    – 2：跟踪TX
    – 4：跟踪RX
    – 8：完整跟踪
  - `low_power=bool`：设置或清除低功耗模式，有效值为 `False` 或 `True`。

**特定LAN类的实现**

在 mimxrt 上，`phy_type` 构造函数参数的合适值为： PHY_KSZ8081、PHY_DP83825、PHY_DP83848、PHY_LAN8720、PHY_RTL8211F。


### PPP 类 - 通过串行 PPP 创建网络连接  

通过串口使用 PPP 协议创建网络连接。  

**注意**, 当前默认固件版本中仅 ESP32 启用了 PPP 支持。在 STM32 和 RP2 的自定义构建中，可通过启用网络支持并将 `MICROPY_PY_NETWORK_PPP_LWIP` 设置为 1 来启用 PPP 支持。  

**示例用法**：  
```python  
import network  

ppp = network.PPP(uart)  
ppp.connect()  

while not ppp.isconnected():  
    pass  

print(ppp.ipconfig("addr4"))  

# 像往常一样使用 socket 模块等  

ppp.disconnect()  
```  

#### 构造函数  

- class network.`PPP`(stream)

  创建一个 PPP 驱动对象。  
  参数说明：  
  - `stream`：支持流协议的任意对象，通常为 `machine.UART` 实例。该流对象必须具有 `irq()` 方法和 `IRQ_RXIDLE` 常量，供 `PPP.connect` 使用。  

#### 方法  

- PPP.`connect`(security=SEC_NONE, user=None, key=None)

  使用给定参数发起 PPP 连接：  
  - `security`：安全类型，可选 `PPP.SEC_NONE`、`PPP.SEC_PAP` 或 `PPP.SEC_CHAP`。  
  - `user`：安全模式中可选的用户名。  
  - `key`：安全模式中可选的密码。  

  调用此方法时，底层流的中断会配置为通过 `stream.irq(ppp.poll, stream.IRQ_RXIDLE)` 调用 `PPP.poll`。这确保在流上有数据可用时，会轮询流并将数据传递到 PPP 协议栈。连接在后台异步进行。  
<br><br>

- PPP.`disconnect`()

  终止连接。必须调用此方法以干净地关闭 PPP 连接。  
<br><br>

- PPP.`isconnected`()

  如果 PPP 链路已连接且正常工作，返回 `True`；否则返回 `False`。  
<br><br>

- PPP.`status`()

  返回 PPP 状态。  
<br><br>

- PPP.`config`(config_parameters)

  设置或获取 PPP 接口的参数。唯一可读写的参数是底层流，使用方式如下：  
  ```python  
  stream = PPP.config("stream")  # 获取流  
  PPP.config(stream=stream)      # 设置流  
  ```  
<br><br>

- PPP.`ipconfig`('param')
- PPP.`ipconfig`(param=value, ...)

  参见 `AbstractNIC.ipconfig`。  
<br><br>

- PPP.`ifconfig`([(ip, subnet, gateway, dns)])

  参见 `AbstractNIC.ifconfig`。  
<br><br>

- PPP.`poll`()

  轮询底层流中的数据并传递到 PPP 协议栈。如果流是带有 RXIDLE 中断的 UART，此方法会自动调用，通常无需手动调用。  

#### 常量  

- `PPP.SEC_NONE`  
- `PPP.SEC_PAP`  
- `PPP.SEC_CHAP`  

  安全连接的类型。


## 网络功能  

以下是网络模块中可用的函数。  

- network.`country`([ code ])

  获取或设置用于无线电合规性的双字母 ISO 3166-1 Alpha-2 国家/地区代码。  
  - 如果提供 `code` 参数，将把国家/地区代码设置为该值。  
  - 如果不带参数调用，将返回当前国家/地区代码。  

  默认代码"XX"代表"全球"区域。  
<br><br>

- network.`hostname`([ name ])

  获取或设置用于在网络中标识设备的主机名，该主机名将被所有网络接口使用。  

  主机名的用途包括：  
  - 向 DHCP 服务器发送客户端请求时使用（如果使用 DHCP）。  
  - 通过 mDNS 进行广播时使用（如果已启用）。  
  
  如果提供 `name` 参数，将把主机名设置为该值。如果不带参数调用，将返回当前主机名。  

  主机名的变更通常仅在连接时生效。对于 DHCP 场景，这是因为主机名是 DHCP 客户端请求的一部分；而大多数硬件的 mDNS 实现仅在连接时初始化一次主机名。因此，必须在激活/连接网络接口之前设置主机名。  

  主机名长度限制为 32 个字符。出于内存原因，MicroPython 可能设置更低的限制。如果给定名称超出限制，将引发 `ValueError` 异常。  

  默认主机名通常为开发板的名称。  
<br><br>

- network.`ipconfig`('param')
- network.`ipconfig`(param=value, ...)

  获取或设置全局 IP 配置参数。支持的参数如下（具体参数的可用性取决于硬件和特定网络接口）：  
  - `dns`：获取/设置 DNS 服务器，支持 IPv4 和 IPv6 地址。  
  - `prefer (4/6)`：如果域名同时存在 A 记录和 AAAA 记录，指定返回的地址类型。**注意**：此操作不会清除本地 DNS 缓存，因此先前获取的地址可能不会改变。  
<br><br>

- network.`phy_mode`([ mode ])

  获取或设置 PHY 模式。如果提供 `mode` 参数，则设置 PHY 模式；如果不带参数调用，将返回当前 PHY 模式。

  可用模式定义为常量：  
  - `MODE_11B` – IEEE 802.11b  
  - `MODE_11G` – IEEE 802.11g  
  - `MODE_11N` – IEEE 802.11n  
  
  **适用范围**：ESP8266。
  
