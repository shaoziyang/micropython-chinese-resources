# bluetooth（底层蓝牙）
  
bluetooth 模块为开发板上的蓝牙控制器提供接口。目前支持低功耗蓝牙 (BLE) 的中心、外设、广播者和观察者角色，以及 GATT 服务器、客户端和 L2CAP 面向连接的通道。设备可同时在多个角色下运行，部分移植版支持配对（和绑定）功能。  

API 旨在匹配底层蓝牙协议，并为更高层次的抽象（如特定设备类型）提供构建模块。  

**注意**  

- 对于大多数应用，建议使用更高级的 [aioble 库](https://github.com/micropython/micropython-lib/tree/master/micropython/bluetooth/aioble)。  
- 该模块仍在开发中，其类、函数、方法和常量可能会发生变化。

## class BLE

### 构造函数  

- class bluetooth.`BLE`  

  返回单个 BLE 对象。  

### 配置  

- BLE.`active`([active, ]/)  

  可选地更改 BLE 射频的激活状态，并返回当前状态。

  在使用该类的任何其他方法之前，必须先激活射频。
<br><br>

- BLE.`config`('param', /)  
- BLE.`config`(*, param=value, ...)  

  获取或设置 BLE 接口的配置。若要获取配置参数，需用通过字符串引用参数名，且每次仅查询一个参数。若要设置参数，使用关键字语法，可一次设置一个或多个参数。  

  当前支持的参数值包括：  

  - 'mac'：当前使用的地址（取决于当前地址模式）。返回值是一个元组 (addr_type, addr)。  

    关于地址类型的详细信息，参见 `gatts_write`。  仅在接口当前处于激活状态时才可查询。  

  - 'addr_mode'：设置地址模式。取值可为：  

    - 0x00 - PUBLIC - 使用控制器的公共地址。  
    - 0x01 - RANDOM - 使用生成的静态地址。  
    - 0x02 - RPA - 使用可解析私有地址。  
    - 0x03 - NRPA - 使用不可解析私有地址。  

    默认情况下，接口模式会优先使用 PUBLIC 地址（若可用），否则使用 RANDOM 地址。

  - 'gap_name'：获取/设置服务 0x1800、特征 0x2a00 使用的 GAP 设备名称。此名称可随时设置且支持多次修改。  
  - 'rxbuf'：获取/设置用于存储传入事件的内部缓冲区大小（以字节为单位）。该缓冲区为整个 BLE 驱动的全局缓冲区，用于处理所有事件的传入数据（包括所有特征值）。增大此值可更好地处理突发传入数据（如扫描结果），并支持接收更大的特征值。  
  - 'mtu'：获取/设置 ATT MTU 交换过程中使用的 MTU 值。最终生效的 MTU 为该值与远程设备 MTU 的最小值。ATT MTU 交换不会自动触发（除非远程设备发起），必须通过 `gattc_exchange_mtu` 手动发起。可通过 `_IRQ_MTU_EXCHANGED` 事件查询特定连接的 MTU 值。  
  - 'bond'：设置配对过程中是否启用绑定功能。启用后，配对请求将设置 "bond" 标志，且密钥会被双方设备存储。  
  - 'mitm'：设置配对是否需要 MITM（中间人攻击）保护。  
  - 'io'：设置设备的 I/O 能力。  
    可用选项包括：
    ```
    _IO_CAPABILITY_DISPLAY_ONLY = const(0)
    _IO_CAPABILITY_DISPLAY_YESNO = const(1)
    _IO_CAPABILITY_KEYBOARD_ONLY = const(2)
    _IO_CAPABILITY_NO_INPUT_OUTPUT = const(3)
    _IO_CAPABILITY_KEYBOARD_DISPLAY = const(4)
    ```
  - 'le_secure'：设置是否需要 "LE Secure" 配对。默认值为`false`（即允许"传统配对"）。

### 事件处理

- BLE.`irq`(handler, / )

  为来自 BLE 协议栈的事件注册回调函数。处理程序接受两个参数：`event`（其值为以下代码之一）和 `data`（一个特定于事件的数值元组）。

  **注意**：为优化性能以避免不必要的内存分配，元组中的 `addr`、`adv_data`、`char_data`、`notify_data` 和 `uuid` 项均为只读的 `memoryview` 实例，这些实例指向蓝牙内部的环形缓冲区，且仅在 IRQ 处理函数调用期间有效。如果程序需要保存这些值中的某一个以便在 IRQ 处理函数返回后访问（例如，将其保存在类实例或全局变量中），则需要通过使用 bytes () 或 `bluetooth.UUID()` 对数据进行复制，示例如下：
  
  ```py
  connected_addr = bytes(addr) # 等效操作适用于adv_data、char_data或notify_data
  matched_uuid = bluetooth.UUID(uuid)
  ```

  例如，扫描结果的 IRQ 处理程序可能会检查 `adv_data` 以确定是否为目标设备，仅在确认后才会复制地址数据以供程序其他部分使用。若要在 IRQ 处理程序中打印数据，需使用 `print(bytes(addr))`。

  以下为展示所有可能事件的事件处理程序示例：

  ```py
  def bt_irq(event, data):
      if event == _IRQ_CENTRAL_CONNECT:
          # 中央设备已连接到此外围设备。
          conn_handle, addr_type, addr = data
      elif event == _IRQ_CENTRAL_DISCONNECT:
          # 中央设备已从此外围设备断开连接。
          conn_handle, addr_type, addr = data
      elif event == _IRQ_GATTS_WRITE:
          # 客户端已写入此特征或描述符。
          conn_handle, attr_handle = data
      elif event == _IRQ_GATTS_READ_REQUEST:
          # 客户端已发起读取请求。注意：此功能仅在STM32上支持。
          # 返回非零整数以拒绝读取（见下文），或返回零（或None）以接受读取。
          conn_handle, attr_handle = data
      elif event == _IRQ_SCAN_RESULT:
          # 单个扫描结果。
          addr_type, addr, adv_type, rssi, adv_data = data
      elif event == _IRQ_SCAN_DONE:
          # 扫描持续时间结束或手动停止。
          pass
      elif event == _IRQ_PERIPHERAL_CONNECT:
          # gap_connect()连接成功。
          conn_handle, addr_type, addr = data
      elif event == _IRQ_PERIPHERAL_DISCONNECT:
          # 已连接的外围设备已断开连接。
          conn_handle, addr_type, addr = data
      elif event == _IRQ_GATTC_SERVICE_RESULT:
          # 每次通过gattc_discover_services()发现服务时调用。
          conn_handle, start_handle, end_handle, uuid = data
      elif event == _IRQ_GATTC_SERVICE_DONE:
          # 服务发现完成后调用。
          # 注意：成功时状态为零，否则为特定于实现的值。
          conn_handle, status = data
      elif event == _IRQ_GATTC_CHARACTERISTIC_RESULT:
          # 每次通过gattc_discover_services()发现特征时调用。
          conn_handle, end_handle, value_handle, properties, uuid = data
      elif event == _IRQ_GATTC_CHARACTERISTIC_DONE:
          # 服务发现完成后调用。
          # 注意：成功时状态为零，否则为特定于实现的值。
          conn_handle, status = data
      elif event == _IRQ_GATTC_DESCRIPTOR_RESULT:
          # 每次通过gattc_discover_descriptors()发现描述符时调用。
          conn_handle, dsc_handle, uuid = data
      elif event == _IRQ_GATTC_DESCRIPTOR_DONE:
          # 服务发现完成后调用。
          # 注意：成功时状态为零，否则为特定于实现的值。
          conn_handle, status = data
      elif event == _IRQ_GATTC_READ_RESULT:
          # gattc_read()已完成。
          conn_handle, value_handle, char_data = data
      elif event == _IRQ_GATTC_READ_DONE:
          # gattc_read()已完成。
          # 注意：成功时状态为零，否则为特定于实现的值。
          conn_handle, value_handle, status = data
      elif event == _IRQ_GATTC_WRITE_DONE:
          # gattc_write()已完成。
          # 注意：成功时状态为零，否则为特定于实现的值。
          conn_handle, value_handle, status = data
      elif event == _IRQ_GATTC_NOTIFY:
          # 服务器已发送通知请求。
          conn_handle, value_handle, notify_data = data
      elif event == _IRQ_GATTC_INDICATE:
          # 服务器已发送指示请求。
          conn_handle, value_handle, notify_data = data
      elif event == _IRQ_GATTS_INDICATE_DONE:
          # 客户端已确认指示。
          # 注意：成功确认时状态为零，否则为特定于实现的值。
          conn_handle, value_handle, status = data
      elif event == _IRQ_MTU_EXCHANGED:
          # ATT MTU交换完成（由我方或远程设备发起）。
          conn_handle, mtu = data
      elif event == _IRQ_L2CAP_ACCEPT:
          # 新通道已被接受。
          # 返回非零整数以拒绝连接，或返回零（或None）以接受。
          conn_handle, cid, psm, our_mtu, peer_mtu = data
      elif event == _IRQ_L2CAP_CONNECT:
          # 新通道现已连接（连接或接受的结果）。
          conn_handle, cid, psm, our_mtu, peer_mtu = data
      elif event == _IRQ_L2CAP_DISCONNECT:
          # 现有通道已断开连接（状态为零），或连接尝试失败（非零状态）。
          conn_handle, cid, psm, status = data
      elif event == _IRQ_L2CAP_RECV:
          # 通道上有新数据可用。使用l2cap_recvinto读取。
          conn_handle, cid = data
      elif event == _IRQ_L2CAP_SEND_READY:
          # 先前返回False的l2cap_send现已完成，通道可再次发送数据。
          # 如果状态非零，则发送缓冲区溢出，应用程序应重新发送数据。
          conn_handle, cid, status = data
      elif event == _IRQ_CONNECTION_UPDATE:
          # 远程设备已更新连接参数。
          conn_handle, conn_interval, conn_latency, supervision_timeout, status = data
      elif event == _IRQ_ENCRYPTION_UPDATE:
          # 加密状态已更改（可能是配对或绑定的结果）。
          conn_handle, encrypted, authenticated, bonded, key_size = data
      elif event == _IRQ_GET_SECRET:
          # 返回存储的密钥。
          # 如果key为None，则返回此sec_type的第index个值。
          # 否则返回此sec_type和key对应的value。
          sec_type, index, key = data
          return value
      elif event == _IRQ_SET_SECRET:
          # 为此sec_type和key将密钥保存到存储中。
          sec_type, key, value = data
          return True
      elif event == _IRQ_PASSKEY_ACTION:
          # 在配对期间响应密钥请求。
          # 详情见gap_passkey()。
          # action将是与配置的"io"配置兼容的操作。
          # 如果action是"数值比较"，则passkey将非零。
          conn_handle, action, passkey = data
  ```
  
  事件代码如下：  
  ```py  
  from micropython import const  
  _IRQ_CENTRAL_CONNECT = const(1)          # 中央设备连接事件  
  _IRQ_CENTRAL_DISCONNECT = const(2)       # 中央设备断开连接事件  
  _IRQ_GATTS_WRITE = const(3)              # GATTS写事件  
  _IRQ_GATTS_READ_REQUEST = const(4)       # GATTS读请求事件  
  _IRQ_SCAN_RESULT = const(5)              # 扫描结果事件  
  _IRQ_SCAN_DONE = const(6)                # 扫描完成事件  
  _IRQ_PERIPHERAL_CONNECT = const(7)       # 外围设备连接事件  
  _IRQ_PERIPHERAL_DISCONNECT = const(8)    # 外围设备断开连接事件  
  _IRQ_GATTC_SERVICE_RESULT = const(9)     # GATTC服务发现结果事件  
  _IRQ_GATTC_SERVICE_DONE = const(10)      # GATTC服务发现完成事件  
  _IRQ_GATTC_CHARACTERISTIC_RESULT = const(11)  # GATTC特征发现结果事件  
  _IRQ_GATTC_CHARACTERISTIC_DONE = const(12)    # GATTC特征发现完成事件  
  _IRQ_GATTC_DESCRIPTOR_RESULT = const(13) # GATTC描述符发现结果事件  
  _IRQ_GATTC_DESCRIPTOR_DONE = const(14)   # GATTC描述符发现完成事件  
  _IRQ_GATTC_READ_RESULT = const(15)       # GATTC读操作结果事件  
  _IRQ_GATTC_READ_DONE = const(16)         # GATTC读操作完成事件  
  _IRQ_GATTC_WRITE_DONE = const(17)        # GATTC写操作完成事件  
  _IRQ_GATTC_NOTIFY = const(18)            # GATTC通知事件  
  _IRQ_GATTC_INDICATE = const(19)          # GATTC指示事件  
  _IRQ_GATTS_INDICATE_DONE = const(20)     # GATTS指示确认事件  
  _IRQ_MTU_EXCHANGED = const(21)           # MTU交换完成事件  
  _IRQ_L2CAP_ACCEPT = const(22)            # L2CAP通道接受事件  
  _IRQ_L2CAP_CONNECT = const(23)           # L2CAP通道连接事件  
  _IRQ_L2CAP_DISCONNECT = const(24)        # L2CAP通道断开事件  
  _IRQ_L2CAP_RECV = const(25)              # L2CAP数据接收事件  
  _IRQ_L2CAP_SEND_READY = const(26)        # L2CAP发送就绪事件  
  _IRQ_CONNECTION_UPDATE = const(27)       # 连接参数更新事件  
  _IRQ_ENCRYPTION_UPDATE = const(28)       # 加密状态更新事件  
  _IRQ_GET_SECRET = const(29)              # 获取密钥事件  
  _IRQ_SET_SECRET = const(30)              # 设置密钥事件  
  ```  
  
  针对 `_IRQ_GATTS_READ_REQUEST` 事件，可用的返回码如下：  
  ```python  
  _GATTS_NO_ERROR = const(0x00)            # 无错误  
  _GATTS_ERROR_READ_NOT_PERMITTED = const(0x02)  # 读取不允许  
  _GATTS_ERROR_WRITE_NOT_PERMITTED = const(0x03) # 写入不允许  
  _GATTS_ERROR_INSUFFICIENT_AUTHENTICATION = const(0x05)  # 认证不足  
  _GATTS_ERROR_INSUFFICIENT_AUTHORIZATION = const(0x08)  # 授权不足  
  _GATTS_ERROR_INSUFFICIENT_ENCRYPTION = const(0x0f)     # 加密不足  
  ```  
  
  针对 `_IRQ_PASSKEY_ACTION` 事件，可用的操作类型如下：  
  ```python  
  _PASSKEY_ACTION_NONE = const(0)          # 无操作  
  _PASSKEY_ACTION_INPUT = const(2)         # 输入密钥  
  _PASSKEY_ACTION_DISPLAY = const(3)       # 显示密钥  
  _PASSKEY_ACTION_NUMERIC_COMPARISON = const(4)  # 数值比较  
  ```  
  
  为节省固件空间，蓝牙模块中未包含这些常量。请根据上述列表将所需常量添加到程序中。
  
### 广播者角色（广告发送方）

- BLE.`gap_advertise`(interval_us, adv_data=None, *, resp_data=None, connectable=True)  

  以指定间隔（微秒）进行的广播。此间隔将向下舍入到最接近的 625 微秒。若要停止广播，将 `interval_us` 设为 `None`。  

  `adv_data` 和 `resp_data` 可以是任何实现了缓冲区协议的类型（例如bytes、bytearray、str）。`adv_data` 包含在所有广播中，`resp_data` 在响应主动扫描时发送。  

  **注意**：如果 `adv_data`（或 `resp_data`）为 `None`，则会复用上次调用 `gap_advertise` 时传入的数据。这允许广播者仅通过 `gap_advertise(interval_us)` 即可恢复广播。如果要清除广播载荷，可传入空字节，即 `b''`。

  

### 观察者角色（扫描器）  

- BLE.`gap_scan`(duration_ms, interval_us=1280000, window_us=11250, active=False, / )  

  执行持续指定时长（毫秒）的扫描操作。  
  - 如果要无限期扫描，将 `duration_ms` 设为 0。  
  - 如果要停止扫描，将 `duration_ms` 设为` None`。  

  可使用 `interval_us` 和 `window_us` 配置扫描占空比。扫描器将以每 `interval_us` 微秒为周期，每次扫描 `window_us` 微秒，总时长为 `duration_ms` 毫秒。默认间隔和窗口分别为 1.28 秒（1280000微秒）和 11.25 毫秒（11250微秒）（背景扫描模式）。  

  每次扫描结果将触发 `_IRQ_SCAN_RESULT` 事件，事件数据为 `(addr_type, addr, adv_type, rssi, adv_data)`。  
  
  addr_type 值表示公共地址或随机地址：  
  - 0x00 - PUBLIC（公共地址）  
  - 0x01 - RANDOM（随机地址，包括静态地址、RPA或NRPA，地址类型编码在地址本身中）  

  adv_type 值对应蓝牙规范中的广告类型：  
  - 0x00 - ADV_IND - 可连接且可扫描的非定向广告  
  - 0x01 - ADV_DIRECT_IND - 可连接的定向广告  
  - 0x02 - ADV_SCAN_IND - 可扫描的非定向广告  
  - 0x03 - ADV_NONCONN_IND - 不可连接的非定向广告  
  - 0x04 - SCAN_RSP - 扫描响应  

  如果需要在扫描结果中接收扫描响应，可将 `active` 设为 `True`。  

  当扫描停止时（因持续时间结束或显式停止），将触发 `_IRQ_SCAN_DONE` 事件。


### 中央设备角色  

中央设备可以连接到观察者角色（见 `gap_scan`）发现的外围设备，或连接到已知地址的外围设备。  

- BLE.`gap_connect`(addr_type, addr, scan_duration_ms=2000, min_conn_interval_us=None, max_conn_interval_us=None, / )  

  连接外围设备。地址类型的详细说明见前面 `gap_scan`。  
  
  若要提前取消未完成的连接尝试，可调用 `gap_connect(None)`。  
  
  成功连接时，将触发 `_IRQ_PERIPHERAL_CONNECT` 事件。取消连接尝试时，将触发 `_IRQ_PERIPHERAL_DISCONNECT` 事件。  
  
  连接间隔可以使用 `min_conn_interval_us` 和 `max_conn_interval_us` 中的一个或两个进行配置，以微秒为时间单位。否则，将使用默认时间间隔，通常在 30000 到 50000 微秒之间。较短的间隔会提高吞吐量，但会以功耗为代价。



### 外围设备角色  

外围设备用于发送可连接的广告（参见 `gap_advertise`）。它通常作为 GATT 服务器，需先通过 `gatts_register_services` 注册服务和特征。  

当中央设备连接时，将触发 `_IRQ_CENTRAL_CONNECT` 事件。  

### 中央设备与外围设备角色  

- BLE.`gap_disconnect`(conn_handle, / )  
  
  断开指定的连接句柄。根据设备角色不同：若作为外围设备，此句柄对应连接到本设备的中央设备；若作为中央设备，此句柄对应本设备先前连接的外围设备。  

  成功断开时，将触发 `_IRQ_PERIPHERAL_DISCONNECT`（中央设备断开外围设备）或 `_IRQ_CENTRAL_DISCONNECT`（外围设备断开中央设备）事件。  
  
  若连接句柄未处于连接状态，返回 `False`；否则返回 `True`。

  

### GATT 服务器  

GATT 服务器拥有一组已注册的服务。每个服务可能包含多个特征（characteristic），每个特征都有一个值。特征还可以包含描述符（descriptor），描述符本身也有对应的值。  

这些值存储在本地，并通过服务注册过程中生成的“值句柄（value handle）”进行访问。远程客户端设备可以对这些值进行读取或写入操作。此外，服务器可以通过连接句柄向已连接的客户端“通知（notify）”某个特征的变化。  

无论是中央设备还是外围设备角色，设备都可以充当 GATT 服务器，但在大多数情况下，外围设备作为服务器更为常见。  

特征和描述符的默认最大长度为20字节。客户端写入的任何数据若超过此长度将被截断。然而，本地写入操作会增加最大长度，因此如果希望允许客户端向某个特征写入更大的数据，可在注册后使用`gatts_write` 函数。例如：`gatts_write(char_handle, bytes(100))`。


- BLE.`gatts_register_services`(services_definition, / )  
  使用指定的服务配置服务器，替换任何现有服务。  

  `services_definition` 是一个**服务**列表，每个服务是一个包含 UUID 和特征列表的二元元组。  

  每个**特征**是一个二元或三元元组，包含 UUID、标志值，以及可选包含描述符列表。  

  每个**描述符**是一个包含 UUID 和标志值的二元元组。  

  **标志值**是以下定义的标志的按位或组合。这些标志既设置特征（或描述符）的行为，也设置安全和隐私要求。  

  返回值是一个列表（每个服务对应一个元素），其中每个元素是元组（每个元组元素为值句柄）。特征和描述符句柄按定义顺序展平到同一元组中。  

  以下示例注册了两个服务（心率服务和 Nordic UART 服务）：

  ```
  HR_UUID = bluetooth.UUID(0x180D)
  HR_CHAR = (bluetooth.UUID(0x2A37), bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY,)
  HR_SERVICE = (HR_UUID, (HR_CHAR,),)
  UART_UUID = bluetooth.UUID('6E400001-B5A3-F393-E0A9-E50E24DCCA9E')
  UART_TX = (bluetooth.UUID('6E400003-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_READ | bluetooth.  FLAG_NOTIFY,)
  UART_RX = (bluetooth.UUID('6E400002-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_WRITE,)
  UART_SERVICE = (UART_UUID, (UART_TX, UART_RX,),)
  SERVICES = (HR_SERVICE, UART_SERVICE,)
  ( (hr,), (tx, rx,), ) = bt.gatts_register_services(SERVICES)
  ```

  这三个值句柄（hr、tx、rx）可与 `gatts_read`、`gatts_write`、`gatts_notify` 和 `gatts_indicate` 配合使用。  
  
  **注意**：注册服务前必须停止广告广播。  

  特征和描述符可用的标志如下：
  ```
  from micropython import const
  _FLAG_BROADCAST = const(0x0001)
  _FLAG_READ = const(0x0002)
  _FLAG_WRITE_NO_RESPONSE = const(0x0004)
  _FLAG_WRITE = const(0x0008)
  _FLAG_NOTIFY = const(0x0010)
  _FLAG_INDICATE = const(0x0020)
  _FLAG_AUTHENTICATED_SIGNED_WRITE = const(0x0040)
  _FLAG_AUX_WRITE = const(0x0100)
  _FLAG_READ_ENCRYPTED = const(0x0200)
  _FLAG_READ_AUTHENTICATED = const(0x0400)
  _FLAG_READ_AUTHORIZED = const(0x0800)
  _FLAG_WRITE_ENCRYPTED = const(0x1000)
  _FLAG_WRITE_AUTHENTICATED = const(0x2000)
  _FLAG_WRITE_AUTHORIZED = const(0x4000)
  ```

  关于上述中断请求（IRQs），需将所有所需常量添加到您的Python代码中。  

- BLE.gatts_re`ad(value_handle, / )  
  读取此句柄的本地值（该值由 `gatts_write` 或远程客户端写入）。  

- BLE.`gatts_write`(value_handle, data, send_update=False, / )  
  写入此句柄的本地值，供客户端读取。  
  
  如果 `send_update` 为 `True`，则任何已订阅的客户端将收到关于此写入的通知（或指示，具体取决于客户端订阅的内容及特征支持的操作）。  

- BLE.`gatts_notify`(conn_handle, value_handle, data=None, / )  
  向已连接的客户端发送通知请求。  
  
  如果 `data` 为 `None`（默认值），将发送当前本地值（通过 `gatts_write` 设置的值）。  
  
  如果 `data` 不为 `None`，则该值将作为通知的一部分发送给客户端，本地值不会被修改。  

  **注意**：无论客户端是否订阅了此特征，通知都会发送。  

- BLE.`gatts_indicate`(conn_handle, value_handle, data=None, / )  
  向已连接的客户端发送指示请求。  

  如果 `data` 为 `None`（默认值），将发送当前本地值（通过 `gatts_write` 设置的值）。  

  如果 `data` 不为 `None`，则该值将作为指示的一部分发送给客户端，本地值不会被修改。  

  当收到确认（或失败，如超时）时，将触发 `_IRQ_GATTS_INDICATE_DONE` 事件。  

  **注意**：无论客户端是否订阅了此特征，指示都会发送。  

- BLE.`gatts_set_buffer`(value_handle, len, append=False, / )  
  设置值的内部缓冲区大小（以字节为单位），这将限制可接收的最大写入长度，默认值为20字节。  

  如果将 `append` 设为 `True`，所有远程写入将追加到当前值，而非替换当前值，最多可缓冲 `len` 字节的数据。使用 `gatts_read` 读取后，值将被清除。此功能在实现类似 Nordic UART 服务时非常有用。


### GATT 客户端  

GATT 客户端可以发现并读写远程 GATT 服务器上的特征（characteristics）。  

中央设备角色作为 GATT 客户端更为常见，但外围设备也可以作为客户端，用于发现连接到它的中央设备的信息（例如，从设备信息服务中读取设备名称）。  

- BLE.gattc_discover_services`(conn_handle, uuid=None, / )  

  查询已连接服务器的服务。  
  
  可选指定service uuid，仅查询该服务。  

  每次发现服务时，将触发 `_IRQ_GATTC_SERVICE_RESULT` 事件，完成后触发 `_IRQ_GATTC_SERVICE_DONE` 事件。  


- BLE.`gattc_discover_characteristics`(conn_handle, start_handle, end_handle, uuid=None, / )  

  查询已连接服务器在指定句柄范围内的特征。  

  可选指定 characteristic uuid，仅查询该特征。  

  可使用 `start_handle=1, end_handle=0xffff` 搜索任意服务中的特征。  

  每次发现特征时，将触发 `_IRQ_GATTC_CHARACTERISTIC_RESULT` 事件，完成后触发 `_IRQ_GATTC_CHARACTERISTIC_DONE` 事件。  


- BLE.`gattc_discover_descriptors`(conn_handle, start_handle, end_handle, / )  

  查询已连接服务器在指定句柄范围内的描述符。  

  每次发现描述符时，将触发 `_IRQ_GATTC_DESCRIPTOR_RESULT` 事件，完成后触发 `_IRQ_GATTC_DESCRIPTOR_DONE` 事件。  


- BLE.`gattc_read`(conn_handle, value_handle, / )  

  向已连接服务器的指定特征或描述符句柄发起远程读取请求。  

  当值可用时，将触发 `_IRQ_GATTC_READ_RESULT` 事件，随后触发 `_IRQ_GATTC_READ_DONE` 事件。  


- BLE.`gattc_write`(conn_handle, value_handle, data, mode=0, / )  

  向已连接服务器的指定特征或描述符句柄发起远程写入请求。  
  
  mode参数指定写入行为，当前支持的值为：  
  - mode=0（默认）：无响应写入，数据发送至远程服务器但不返回确认，也不触发事件。  
  - mode=1：有响应写入，请求远程服务器返回数据接收确认。  

  若收到远程服务器的响应，将触发 `_IRQ_GATTC_WRITE_DONE` 事件。  
  

- BLE.`gattc_exchange_mtu`(conn_handle, / )  

  使用通过 `BLE.config(mtu=value)` 设置的首选 MTU，与已连接服务器发起 MTU 交换。  

  MTU交换完成时，将触发 `_IRQ_MTU_EXCHANGED` 事件。

  **注意**：MTU交换通常由中央设备发起。当中央设备使用 BlueKitchen 协议栈时，不支持远程外围设备发起 MTU 交换；NimBLE 协议栈则支持双角色发起。



### L2CAP 面向连接通道  

此功能允许两个BLE设备之间进行类似套接字的数据交换。设备通过 GAP 连接后，任意一方均可监听对方通过数字 PSM（协议/服务多路复用器）发起的连接。  

**注意**：当前仅 STM32 和 Unix 系统上使用 NimBLE 协议栈时支持此功能（ESP32不支持）。同一时间仅允许激活一个 L2CAP 通道（即监听时无法连接）。  

活动的 L2CAP 通道通过其建立时的连接句柄（connection handle）和 CID（通道ID）标识。  

面向连接的通道内置基于信用的流量控制机制。与 ATT 协议中设备协商共享 MTU 不同，监听方和连接方各自设置独立的 MTU，该 MTU 限制了远程设备在数据被 `l2cap_recvinto` 完全消耗前可发送的未处理数据的最大量。  

- BLE.`l2cap_listen`(psm, mtu, / )  

  开始在指定 `psm` 上监听传入的 L2CAP 通道请求，本地 MTU 设置为 `mtu`。  

  当远程设备发起连接时，将触发 `_IRQ_L2CAP_ACCEPT` 事件，这使监听服务器有机会拒绝传入的连接（通过返回非零整数）。  

  连接接受后，将触发 `_IRQ_L2CAP_CONNECT` 事件，服务器可通过该事件获取通道 ID（CID）及本地和远程 MTU。  

  **注意**：目前无法停止监听。  


- BLE.`l2cap_connect`(conn_handle, psm, mtu, / )  

  使用本地 MTU 为 `mtu`，连接到指定 `psm` 上的监听设备。 
  
  连接成功时，触发 `_IRQ_L2CAP_CONNECT` 事件，客户端可获取 CID 及本地和远程（对端）MTU。  

  连接失败时，触发 `_IRQ_L2CAP_DISCONNECT` 事件，状态码非零。  


- BLE.`l2cap_disconnect`(conn_handle, cid, / )  

  断开指定 `conn_handle` 和 `cid` 的活动L2CAP通道。  


- BLE.`l2cap_send`(conn_handle, cid, buf, / )  

  在由 `conn_handle` 和 `cid` 标识的 L2CAP 通道上发送指定缓冲区 `buf`（必须支持缓冲区协议）。  

  指定的缓冲区大小不能超过远程（对端）MTU，且不能超过本地MTU的两倍。  

  若通道当前处于"停滞"状态，函数返回 `False`，此时必须等待触发 `_IRQ_L2CAP_SEND_READY` 事件（通常在远程设备接收并处理数据后授予更多信用时触发）后，才能再次调用 `l2cap_send`。  


- BLE.`l2cap_recvinto`(conn_handle, cid, buf, / )  

  从指定 `conn_handle` 和 `cid` 的通道接收数据到提供的 `buf` 中（buf 必须支持缓冲区协议，如 bytearray 或memoryview）。  

  返回从通道读取的字节数。若 `buf` 为 `None`，则返回可用字节数。  

  **注意**：收到 `_IRQ_L2CAP_RECV` 事件后，应用程序应继续调用 `l2cap_recvinto`，直至接收缓冲区无更多数据（通常最多为远程MTU大小）。在接收缓冲区清空之前，远程设备不会被授予更多通道信用，无法发送更多数据。


### 配对与绑定  

配对允许通过交换密钥（通过密钥认证提供可选的中间人攻击保护）对连接进行加密和认证。  

绑定是将这些密钥存储到非易失性存储器中的过程。绑定后，设备能够基于存储的身份解析密钥（IRK）从另一设备解析可解析私有地址（RPA）。  

要支持绑定，应用程序必须实现 `_IRQ_GET_SECRET` 和 `_IRQ_SET_SECRET` 事件。  

**注意**：当前仅ESP32、STM32 和 Unix 系统上使用 NimBLE 协议栈时支持此功能。  

- BLE.`gap_pair`(conn_handle, / )  
  
  发起与远程设备的配对。  

  调用前，请确保已通过 `config` 设置 `io`、`mitm`、`le_secure` 和 `bond` 配置选项。  

  配对成功时，将触发 `_IRQ_ENCRYPTION_UPDATE` 事件。  


- BLE.`gap_passkey`(conn_handle, action, passkey, / )  

  响应指定 `conn_handle` 和 `action` 的 `_IRQ_PASSKEY_ACTION` 事件。  

  `passkey` 为数字值，具体取决于操作类型（操作类型取决于设置的I/O能力）：  
  - 当 `action` 为 `_PASSKEY_ACTION_INPUT` 时，应用程序应提示用户输入远程设备上显示的密钥。  
  - 当 `action` 为 `_PASSKEY_ACTION_DISPLAY` 时，应用程序应生成一个随机6位密钥并显示给用户。  
  - 当 `action` 为 `_PASSKEY_ACTION_NUMERIC_COMPARISON` 时，应用程序应显示 `_IRQ_PASSKEY_ACTION` 事件中提供的密钥，然后返回0（取消配对）或1（接受配对）。


### class UUID  

**构造函数**  

- class bluetooth.`UUID`(value, / )  

  创建一个具有指定值的 UUID 实例。  

  值可以是以下两种形式之一：  
  - 16 位整数，例如 0x2908。  
  - 128 位 UUID 字符串，例如 '6E400001-B5A3-F393-E0A9-E50E24DCCA9E'。