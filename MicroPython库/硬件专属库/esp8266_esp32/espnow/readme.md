# espnow（ESP-NOW 无线协议）

该模块提供了一个接口，用于访问乐鑫（Espressif）在 ESP32 和 ESP8266 设备上提供的 [ESP-NOW](https://www.espressif.com/en/products/software/esp-now/overview) 协议（[API 文档](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/network/esp_now.html)）。


## 简介

ESP-NOW 是一种无连接的无线通信协议，支持：
- 最多 20 个注册节点之间的直接通信：
  - 无需无线接入点（AP），
- 加密和未加密通信（最多 6 个加密节点），
- 消息大小最多 250 字节，
- 可在 ESP32 和 ESP8266 设备上与 Wi-Fi 功能（network.WLAN）同时运行。

它特别适用于小型物联网网络、对延迟敏感或对功耗敏感的应用（如电池供电设备）以及设备之间的长距离通信（数百米）。

该模块还支持跟踪节点设备的 Wi-Fi 信号强度（RSSI）。

一个简单的示例如下：

**发送端：**
```python
import network
import espnow

# 必须激活 WLAN 接口才能发送/接收
sta = network.WLAN(network.WLAN.IF_STA)  # 或 network.WLAN.IF_AP
sta.active(True)
sta.disconnect()  # 对于 ESP8266

e = espnow.ESPNow()
e.active(True)
peer = b'\xbb\xbb\xbb\xbb\xbb\xbb'  # 节点 Wi-Fi 接口的 MAC 地址
e.add_peer(peer)  # 必须先调用 add_peer() 再调用 send()

e.send(peer, "Starting...")
for i in range(100):
    e.send(peer, str(i)*20, True)
e.send(peer, b'end')
```

**接收端：**
```python
import network
import espnow

# 必须激活 WLAN 接口才能发送/接收
sta = network.WLAN(network.WLAN.IF_STA)
sta.active(True)
sta.disconnect()  # 因为 ESP8266 会自动连接到最后一个接入点

e = espnow.ESPNow()
e.active(True)

while True:
    host, msg = e.recv()
    if msg:  # 如果 recv() 超时，msg == None
        print(host, msg)
        if msg == b'end':
            break
```


## class ESPNow

### 构造函数

- class espnow.`ESPNow`

  返回单例 ESPNow 对象。由于这是一个单例，所有对 `espnow.ESPNow()` 的调用都返回对同一对象的引用。

  **注意**，由于 ESP8266 上的代码大小限制以及乐鑫 API 的差异，某些方法仅在 ESP32 上可用。


### 配置

- ESPNow.`active`([flag])

  根据可选参数 `flag` 的值初始化或关闭 ESP-NOW。

  `flag` 是任何可转换为布尔类型的 Python 值。
  - `True`：准备好使用 ESP-NOW 通信协议的软件和硬件，包括：
    - 初始化 ESPNow 数据结构，
    - 分配接收数据缓冲区，
    - 调用 `esp_now_init()` 以及
    - 注册发送和接收回调。
  - `False`：关闭 ESP-NOW 软件栈（`esp_now_deinit()`），禁用回调，释放接收数据缓冲区并注销所有节点。

  如果未提供 `flag` 参数，则返回当前 ESPNow 接口的状态。如果接口当前处于活动状态，则为 `True`，否则为 `False`。
<br><br>

- ESPNow.`config`(param=value, ...)
- ESPNow.`config`('param')（仅 ESP32）

  设置或获取 ESPNow 接口的配置值。要设置值，请使用关键字语法，一次可以设置一个或多个参数。要获取值，参数名称应作为字符串引用，并且一次仅查询一个参数。

  **注意**：ESP8266 不支持获取参数。

  **选项：**
  - `rxbuf`：设置或获取用于存储传入 ESPNow 数据包数据的内部缓冲区的大小（以字节为单位）。默认大小为526，选择为适合两个最大尺寸的 ESPNow 数据包（250 字节），以及相关的 mac 地址（6 字节）、消息字节计数（1 字节）和 RSSI 数据加上缓冲区开销。如果预期会收到大量大型数据包或突发传入流量，请增大此值。  
  **注意**：接收缓冲区由 `ESPNow.active()` 分配。更改后在下次调用 `ESPNow.active(True)` 之前不会生效。

  - `timeout_ms`：接收 ESPNow 消息的默认超时时间（以毫秒为单位），默认值为300000。如果 `timeout_ms` 小于零，则无限期等待。超时也可以作为参数提供给 `recv()`/`irecv()`/`recvinto()`。
  
  - `rate`：（仅 ESP32）设置 ESPNow 数据包的传输速度。必须设置为 `enum wifi_phy_rate_t` 中允许的数值之一。由于 ESP-IDF 不提供查询无线电接口速率参数的任何方法，因此该参数实际上是只写的。

  **返回** `None` 或被查询参数的值。

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `ValueError()`：如果配置选项或值无效。


### 发送和接收数据

必须先激活 Wi-Fi 接口（`network.WLAN.IF_STA` 或 `network.WLAN.IF_AP`），才能发送或接收消息，但无需连接或配置 WLAN 接口。例如：

```python
import network

sta = network.WLAN(network.WLAN.IF_STA)
sta.active(True)
sta.disconnect()  # 对于 ESP8266
```

**注意**：ESP8266 有一个特性，当设置 `active(True)` 时（即使在重启/复位后），它会自动连接到最后一个 Wi-Fi 接入点。这会降低接收 ESP-NOW 消息的可靠性（参见 ESPNow 与 Wi-Fi 操作）。可以通过在 `active(True)` 之后调用 `disconnect()` 来避免这种情况。


- ESPNow.`send`(mac, msg[, sync])
- ESPNow.`send`(msg)`（仅 ESP32）

  将 `msg` 中包含的数据发送到具有给定网络 `mac` 地址的节点。在第二种形式中，`mac=None` 且 `sync=True`。必须先使用 `ESPNow.add_peer()` 注册节点，然后才能发送消息。

  **参数：**
  - `mac`：正好为 `espnow.ADDR_LEN`（6 字节）长度的字节字符串或 `None`。如果 `mac` 为 `None`（仅 ESP32），消息将发送到所有已注册的节点，但不包括任何广播或多播 MAC 地址。
  - `msg`：字符串或字节字符串，最长为 `espnow.MAX_DATA_LEN`（250）字节。
  - `sync`：
    - `True`：（默认）将消息发送到节点并等待响应（或不响应）。
    - `False`：发送消息并立即返回。来自节点的响应将被丢弃。

  **返回：** 如果 `sync=False` 或者 `sync=True` 且所有节点都响应，则为 `True`，否则为 `False`。

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_FOUND")`：如果节点未注册。
  - `OSError(num, "ESP_ERR_ESPNOW_IF")`：Wi-Fi 接口未激活。
  - `OSError(num, "ESP_ERR_ESPNOW_NO_MEM")`：内部 ESP-NOW 缓冲区已满。
  - `ValueError()`：参数值无效。

   **注意**：如果节点的 Wi-Fi 接口处于活动状态并且设置为与发送方相同的信道，则无论其是否已初始化 ESP-NOW 系统或是否正在积极监听 ESP-NOW 流量，节点都会成功响应（参见乐鑫 ESP-NOW 文档）。
<br><br>

- ESPNow.`recv`([timeout_ms])

  等待传入消息并返回节点的 mac 地址和消息。注意：无需注册节点（使用 `add_peer()`）即可接收来自该节点的消息。

  **参数：**
  - `timeout_ms`：（可选）可以具有以下值。
    - 0：无超时。如果没有可用数据，立即返回；
    - \> 0：指定超时值（以毫秒为单位）；
    - < 0：不超时，即永远等待新消息；
    - `None`（或未提供）：使用 `ESPNow.config()` 设置的默认超时值。

  **返回：**
  - 如果在收到消息之前达到超时，则为 `(None, None)`,
  - `[mac, msg]`：其中：
    - `mac` 是包含发送消息的设备地址的字节字符串，
    - `msg` 是包含消息的字节字符串。

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `OSError(num, "ESP_ERR_ESPNOW_IF")`：如果 Wi-Fi 接口未激活。
  - `ValueError()`：`timeout_ms` 值无效。

  `ESPNow.recv()` 将为返回的列表以及节点和消息字节字符串分配新的存储空间。如果数据速率很高，这可能导致内存碎片。参见 `ESPNow.irecv()` 以获得内存友好的替代方案。
<br><br>

- ESPNow.`irecv`([timeout_ms])

  工作方式与 `ESPNow.recv()` 类似，但会重用内部字节数组来存储返回值：`[mac, msg]`，因此每次调用都不会分配新内存。

  `timeout_ms`：（可选）超时时间（以毫秒为单位）（参见 `ESPNow.recv()`）。

  **返回：** 与 `ESPNow.recv()` 相同，只是 `msg` 是字节数组，而不是字节字符串。在 ESP8266 上，`mac` 也将是字节数组。

  **异常：** 参见 `ESPNow.recv()`。

  **注意**：也可以通过迭代 ESPNow 对象来读取消息，这将使用 `irecv()` 方法进行无分配读取，例如：
  ```python
  import espnow
  
  e = espnow.ESPNow(); e.active(True)
  for mac, msg in e:
      print(mac, msg)
      if mac is None:
          # 超时后，mac、msg 将等于 (None, None)
          break
  ```
<br>

- ESPNow.`recvinto`(data[, timeout_ms])

  等待传入消息并返回消息的长度（以字节为单位）。这是 `recv()` 和 `irecv()` 用于读取消息的低级方法。

  **参数：**
  - `data`：至少包含两个元素的列表 `[peer, msg]`。`msg` 必须是足够大的字节数组以容纳消息（250 字节）。在 ESP8266 上，`peer` 应该是 6 字节的字节数组。发送方的 MAC 地址和消息将存储在这些字节数组中（参见下面关于 ESP32 的注意事项）。

  - `timeout_ms`：（可选）超时时间（以毫秒为单位）（参见 `ESPNow.recv()`）。

  **返回：** 消息的字节长度，如果在收到消息之前达到 `timeout_ms`，则为 0。

  **异常：** 参见 `ESPNow.recv()`。

  **注意**：在 ESP32 上：
  - 无需在数据列表的第一个元素中提供字节数组，因为它将被替换为对节点设备表中唯一节点地址的引用（参见 `ESPNow.peers_table`）。
  - 如果列表至少有 4 个元素，则 rssi 和时间戳值将保存为第 3 和第 4 个元素。
<br><br>

- ESPNow.`any`()

  检查是否有数据可通过 `ESPNow.recv()` 读取。

  对于更复杂的可用字符查询，请使用 `select.poll()`：

  ```python
  import select
  import espnow
  
  e = espnow.ESPNow()
  poll = select.poll()
  poll.register(e, select.POLLIN)
  poll.poll(timeout)
  ```

  **返回：** 如果有可用数据可读取，则为 `True`，否则为 `False`。
<br><br>

- ESPNow.`stats`()（仅 ESP32）

  **返回：** 一个 5 元组，包含发送/接收/丢失的数据包数量：
  
  `(tx_pkts, tx_responses, tx_failures, rx_packets, rx_dropped_packets)`

  当接收缓冲区已满时，传入的数据包会被丢弃。要减少数据包丢失，请增加 `rxbuf` 配置参数，并确保尽快读取消息。

  **注意**：已丢弃的数据包仍会向发送方确认已接收。


## 节点管理

在 ESP32 设备上，乐鑫 ESP-NOW 软件要求必须先使用 `add_peer()` 注册其他设备（节点），然后才能向它们发送消息（ESP8266 设备不强制执行此操作）。无需注册节点即可接收来自该节点的未加密消息。

要接收加密消息，接收设备必须首先注册发送方，并使用与发送方相同的加密密钥（PMK 和 LMK）（参见 `set_pmk()` 和 `add_peer()`）。


- ESPNow.`set_pmk`(pmk)

  设置主密钥（PMK），该密钥用于加密本地主密钥（LMK）。如果未设置，乐鑫 ESP-NOW 软件栈将使用默认 PMK。

  **注意**：只有在 `ESPNow.add_peer()` 中也设置了 `lmk` 时，消息才会被加密（参见乐鑫 API 文档中的安全性）。

  **参数：**
  - `pmk`：必须是长度为 `espnow.KEY_LEN`（16 字节）的字节字符串、字节数组或字符串。

  **返回：** `None`

  **异常：** `pmk` 值无效时引发 `ValueError()`。
<br><br>

- ESPNow.`add_peer`(mac[, lmk][, channel][, ifidx][, encrypt])
- ESPNow.`add_peer`(mac, param=value, ...)`（仅 ESP32）

  添加/注册提供的 `mac` 地址作为节点。也可以将附加参数指定为位置参数或关键字参数。

  **参数：**
  - `mac`：节点的 MAC 地址（作为 6 字节字节字符串）。
  - `lmk`：用于加密与此节点的数据传输的本地主密钥（LMK）（除非 `encrypt` 参数设置为 `False`）。必须是：
    - 长度为 `espnow.KEY_LEN`（16 字节）的字节字符串、字节数组或字符串，或者
    - 任何非 True 的 python 值（默认值= b''），表示空密钥，这将禁用加密。
  - `channel`：用于与此节点通信的 wifi 信道（2.4GHz）。必须是 0 到 14 之间的整数（默认值=0）。如果 channel 设置为 0，则将使用 wifi 设备的当前信道，如果 channel 设置为其他值，则必须与接口当前配置的信道匹配（参见 `WLAN.config()`）。
  - `ifidx`：（仅 ESP32）用于向此节点发送数据的 wifi 接口的索引。必须是设置为 `network.WLAN.IF_STA`（=0）或 `network.WLAN.IF_AP`（=1）的整数。（默认值是 0）。
  - `encrypt`：（仅 ESP32）如果设置为 `True`，则与此节点交换的数据将使用 PMK 和 LMK 进行加密。（如果 lmk 设置为有效密钥时默认值为 `True`，否则为 `False`）

  ESP8266 上不能使用关键字参数。

  **注意**：最多可注册 20 个节点（`espnow.MAX_TOTAL_PEER_NUM`），其中最多 6 个是启用加密的节点（`espnow.MAX_ENCRYPT_PEER_NUM`）（参见乐鑫 API 文档中的 [ESP_NOW_MAX_ENCRYPT_PEER_NUM](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/network/esp_now.html#c.ESP_NOW_MAX_ENCRYPT_PEER_NUM)）。

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `OSError(num, "ESP_ERR_ESPNOW_EXIST")`：如果 mac 已注册。
  - `OSError(num, "ESP_ERR_ESPNOW_FULL")`：如果已注册的节点太多。
  - `OSError(num, "ESP_ERR_ESPNOW_CHAN")`：设置的信道值与为此接口当前配置的信道不匹配。
  - `ValueError()`：关键字参数或值无效。
<br><br>

- ESPNow.`del_peer`(mac)

  注销与提供的 `mac` 地址关联的节点。

  **返回：** `None`

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_FOUND")`：如果 `mac` 未注册。
  - `ValueError()`：当 `mac` 无效。
<br><br>

- ESPNow.`get_peer`(mac) （仅 ESP32）

  返回已注册节点的信息。

  **返回值：** `(mac, lmk, channel, ifidx, encrypt)`，与给定 `mac` 地址关联的"节点信息"元组。

  **异常：**
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_INIT")`：如果未初始化。
  - `OSError(num, "ESP_ERR_ESPNOW_NOT_FOUND")`：如果 `mac` 地址未注册。
  - `ValueError()`：当 `mac` 值无效。
<br><br>

- ESPNow.`peer_count`() （仅 ESP32）

  返回已注册节点的数量：
  - `(peer_num, encrypt_num)`：其中
    - `peer_num` 是已注册节点的总数，
    - `encrypt_num` 是已启用加密的节点数量。
<br><br>

- ESPNow.`get_peers`() （仅 ESP32）

  返回所有已注册节点的"节点信息"参数（作为元组的元组）。
<br><br>

- ESPNow.`mod_peer`(mac, lmk, [channel], [ifidx], [encrypt]) （仅 ESP32）
- ESPNow.`mod_peer`(mac, 'param'=value, ...) （仅 ESP32）

  修改与提供的 `mac` 地址关联的节点参数。参数可以作为位置参数或关键字参数提供（参见 `ESPNow.add_peer()`）。未设置的任何参数（或设置为 `None`）将保留该参数的现有值。

## 回调方法

- ESPNow.`irq`(callback) （仅适用于 ESP32）

  设置一个回调函数，当接收到另一个 ESPNow 设备发送的消息后，该函数会尽快被调用。回调函数将以 ESPNow 实例对象作为参数被调用。为了更可靠地运行，建议在回调函数被调用时读取所有可用的消息，并将读取超时设置为零，例如：

  ```python
  def recv_cb(e):
      while True:  # 读取缓冲区中所有等待的消息
          mac, msg = e.irecv(0)  # 如果没有更多消息，不等待
          if mac is None:
              return
          print(mac, msg)
  e.irq(recv_cb)
  ```

  `irq()` 回调方法是一种处理传入消息的替代方法，特别是当数据传输速率适中且设备不太繁忙时，但也有一些注意事项：
  - 如果数据包以足够快的速率到达，或者其他 MicroPython 组件（例如，蓝牙、machine.Pin.irq()、machine.timer、i2s 等）占用调度器堆栈，调度器堆栈可能会溢出，从而导致回调函数被错过。对于处理消息突发、高吞吐量的情况，或者在设备正忙于处理其他硬件操作时，这种方法的可靠性可能较低。
  - 有关调度函数回调的更多信息，请参见：`micropython.schedule()`。


## 常量

  - espnow.`MAX_DATA_LEN` (=250)
  - espnow.`KEY_LEN` (=16)
  - espnow.`ADDR_LEN` (=6)
  - espnow.`MAX_TOTAL_PEER_NUM` (=20)
  - espnow.`MAX_ENCRYPT_PEER_NUM` (=6)


## 异常

如果底层的乐鑫 ESP-NOW 软件栈返回错误代码，MicroPython 的 espnow 模块将引发 `OSError(errnum, errstring)` 异常，其中 `errstring` 被设置为乐鑫 ESP-NOW 文档中指定的某个错误代码的名称。例如：

```python
try:
    e.send(peer, 'Hello')
except OSError as err:
    if len(err.args) < 2:
        raise err
    if err.args[1] == 'ESP_ERR_ESPNOW_NOT_INIT':
        e.active(True)
    elif err.args[1] == 'ESP_ERR_ESPNOW_NOT_FOUND':
        e.add_peer(peer)
    elif err.args[1] == 'ESP_ERR_ESPNOW_IF':
        network.WLAN(network.WLAN.IF_STA).active(True)
    else:
        raise err
```

## Wi-Fi 信号强度（RSSI） （仅适用于 ESP32）

ESPNow 对象会维护一个节点设备表，其中包含从所有主机接收到的最后一条消息的信号强度和时间戳。可以通过 `ESPNow.peers_table` 访问该节点设备表，它可用于跟踪设备的距离以及识别节点设备网络中最近的邻居。此功能在 ESP8266 设备上不可用。

- ESPNow.`peers_table`

  节点设备表的引用：一个包含已知节点设备和 RSSI 值的字典：

  `{peer: [rssi, time_ms], ...}`

  其中：
  - `peer` 是节点的 MAC 地址（以字节形式表示）；
  - `rssi` 是从该节点接收到的最后一条消息的 Wi-Fi 信号强度，单位为 dBm（范围为 -127 至 0）；
  - `time_ms` 是接收消息的时间（自系统启动以来的毫秒数，每 12 天循环一次）。

  **示例：**
  ```python
  >>> e.peers_table
  {b'\xaa\xaa\xaa\xaa\xaa\xaa': [-31, 18372],
   b'\xbb\xbb\xbb\xbb\xbb\xbb': [-43, 12541]}
  ```

  **注意**：
  - `recv()` 返回的 MAC 地址是对**节点设备表**中节点键值的引用。
  - 设备表中的 RSSI 和时间戳值仅在应用程序读取消息时才会更新。

## 支持 asyncio

有一个补充模块（`aioespnow`）可提供 `asyncio` 支持。

**注意**：所有 ESP32 目标设备以及包含 asyncio 模块的 ESP8266 开发板（即至少有 2MB 闪存的 ESP8266 设备）均支持 asyncio。

一个小型异步服务器示例：

```python
import network
import aioespnow
import asyncio

# 必须激活 WLAN 接口才能发送/接收数据
network.WLAN(network.WLAN.IF_STA).active(True)

e = aioespnow.AIOESPNow()  # 返回增强了异步支持的 AIOESPNow
e.active(True)
peer = b'\xbb\xbb\xbb\xbb\xbb\xbb'
e.add_peer(peer)

# 定期向节点发送 ping 包
async def heartbeat(e, peer, period=30):
    while True:
        if not await e.asend(peer, b'ping'):
            print("心跳：节点无响应：", peer)
        else:
            print("心跳：ping", peer)
        await asyncio.sleep(period)

# 将收到的任何消息回显给发送方
async def echo_server(e):
    async for mac, msg in e:
        print("回显：", msg)
        try:
            await e.asend(mac, msg)
        except OSError as err:
            if len(err.args) > 1 and err.args[1] == 'ESP_ERR_ESPNOW_NOT_FOUND':
                e.add_peer(mac)
                await e.asend(mac, msg)

async def main(e, peer, timeout, period):
    asyncio.create_task(heartbeat(e, peer, period))
    asyncio.create_task(echo_server(e))
    await asyncio.sleep(timeout)

asyncio.run(main(e, peer, 120, 10))
```

- class aioespnow.`AIOESPNow`

  `AIOESPNow` 类继承了 `ESPNow` 的所有方法，并通过以下异步方法扩展了接口。
<br><br>

- async AIOESPNow.`arecv`()

  为 `ESPNow.recv()` 提供 asyncio 支持。注意，此方法不接受超时值作为参数。
<br><br>

- async AIOESPNow.`airecv`()

  为 `ESPNow.irecv()` 提供 asyncio 支持。注意，此方法不接受超时值作为参数。
<br><br>

- async AIOESPNow.`asend`(mac, msg, sync=True)
- async AIOESPNow.`asend`(msg)`

  为 `ESPNow.send()` 提供 asyncio 支持。
<br><br>

- AIOESPNow.`__aiter__`() / async AIOESPNow.`__anext__`()

  `AIOESPNow` 还支持通过 `async for` 进行异步迭代来读取传入消息，例如：

  ```python
  e = AIOESPNow()
  e.active(True)
  
  async def recv_till_halt(e):
      async for mac, msg in e:
          print(mac, msg)
          if msg == b'halt':
              break
  
  asyncio.run(recv_till_halt(e))
  ```

## 广播和多播

所有处于活动状态的 ESPNow 客户端都会接收发送到其 MAC 地址的消息，并且所有设备（ESP8266 设备除外）还会接收发送到广播 MAC 地址（`b'\xff\xff\xff\xff\xff\xff'`）或任何多播 MAC 地址的消息。

所有 ESPNow 设备（包括 ESP8266 设备）也都可以向广播 MAC 地址或任何多播 MAC 地址发送消息。

要发送广播消息，必须先使用 `add_peer()` 注册广播（或多播）MAC 地址。对于广播，`send()` 始终返回 True，无论是否有设备接收到该消息。不允许对发送到广播地址或任何多播地址的消息进行加密。

注意：`ESPNow.send(None, msg)` 会向所有已注册的节点发送消息（广播地址除外）。要发送广播或多播消息，必须指定广播（或多播）MAC 地址作为节点。例如：

```python
bcast = b'\xff' * 6
e.add_peer(bcast)
e.send(bcast, "Hello World!")
```

## ESPNow 与 WiFi 操作

ESPNow 消息可以在任何处于活动状态的 WLAN 接口（`network.WLAN.IF_STA` 或 `network.WLAN.IF_AP`）上发送和接收，即使该接口已连接到 wifi 网络或配置为接入点也是如此。当 ESP32 或 ESP8266 设备连接到 wifi 接入点时，会发生以下影响 ESPNow 通信的情况：

1. Wifi 省电模式（`network.WLAN.PM_PERFORMANCE`）会自动激活；
2. ESP 设备上的无线电会更改 wifi 信道，以匹配接入点使用的信道。

**Wifi 省电模式**: （参见[乐鑫文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/wifi.html#esp32-wi-fi-power-saving-mode)）省电模式会导致设备定期关闭无线电（通常持续数百毫秒），这使得接收 ESPNow 消息变得不可靠。这可以通过以下任一方式解决：
1. 在 STA_IF 接口上禁用省电模式；
   - 使用 `sta.config(pm=sta.PM_NONE)`
2. 开启 AP_IF 接口，这将禁用省电模式。不过，此时设备将广播一个活动的 wifi 接入点。
   - 可以选择通过 AP_IF 接口发送消息，但这不是必需的。
   - ESP8266 节点必须向此 AP_IF 接口发送消息（见下文）。
3. 配置 ESPNow 客户端重试发送消息。

**接收来自 ESP8266 设备的消息**: 奇怪的是，使用上述方法 1 或 2 连接到 wifi 网络的 ESP32 设备，会接收来自另一个 ESP32 设备发送到 STA_IF MAC 地址的 ESPNow 消息，但会拒绝来自 ESP8266 设备的消息！！！要接收来自 ESP8266 设备的消息，必须将 AP_IF 接口设置为活动状态（`active(True)`），并且消息必须发送到 AP_IF MAC 地址。

**管理 Wi-Fi 信道**: 任何其他希望与已连接到 wifi 接入点的设备进行通信的 ESPNow 设备，都必须使用相同的信道。一种常见场景是，一个 ESPNow 设备连接到 wifi 路由器，并作为来自一组通过 ESPNow 连接的传感器的消息的代理：

代理：
```python
import network, time, espnow

sta, ap = wifi_reset()  # 重置 Wi-Fi，使 AP 关闭、STA 开启且断开连接
sta.connect('myssid', 'mypassword')
while not sta.isconnected():  # 等待连接...
    time.sleep(0.1)
sta.config(pm=sta.PM_NONE)    # ...然后禁用省电模式

# 在完成接入点连接后，打印使用的 Wi-Fi 信道
print("Proxy running on channel:", sta.config("channel"))
e = espnow.ESPNow(); e.active(True)
for peer, msg in e:
    # 接收 ESPNow 消息并通过 WiFi 转发到 MQTT 代理
```

传感器：
```python
import network, espnow

sta, ap = wifi_reset()  # 重置 Wi-Fi，使 AP 关闭、STA 开启且断开连接
sta.config(channel=6)   # 更改到上述代理使用的信道
peer = b'0\xaa\xaa\xaa\xaa\xaa'  # 代理的 MAC 地址
e = espnow.ESPNow(); e.active(True);
e.add_peer(peer)
while True:
    msg = read_sensor()
    e.send(peer, msg)
    time.sleep(1)
```

使用 ESPNow 和 Wi-Fi 时需要注意的其他问题：
- 在启动时将 wifi 设置为已知状态：软复位后，MicroPython 不会重置 wifi 外设。这可能导致意外行为。为确保软复位后 wifi 重置到已知状态，请在启动时将 STA_IF 和 AP_IF 接口停用，然后再将它们设置为所需状态，例如：

  ```python
  import network, time

  def wifi_reset():
      # 重置 wifi，使 AP 关闭、STA 开启且断开连接
      sta = network.WLAN(network.WLAN.IF_STA); sta.active(False)
      ap = network.WLAN(network.WLAN.IF_AP); ap.active(False)
      sta.active(True)

      while not sta.active():
          time.sleep(0.1)
      sta.disconnect()  # 针对 ESP8266

      while sta.isconnected():
          time.sleep(0.1)
      return sta, ap

  sta, ap = wifi_reset()
  ```
  请记住，每次连接到设备的 REPL 并输入 ctrl-D 时，都会发生软复位。

- STA_IF 和 AP_IF 始终在同一信道上运行：当您连接到 wifi 网络时，无论您为 AP_IF 设置了什么信道，AP_IF 会更改信道。毕竟设备上实际上只有一个 wifi 无线外设，由 STA_IF 和 AP_IF 虚拟设备共享。

- 禁用 wifi 路由器上的自动信道分配：如果您的 wifi 网络的路由器配置为自动分配 wifi 信道，当它检测到来自其他 wifi 路由器的干扰时，可能会更改网络的信道。发生这种情况时，连接到 wifi 网络的 ESP 设备也会更改信道以匹配路由器，但其他仅使用 ESPNow 的设备将保持在之前的信道上，从而导致通信中断。为缓解此问题，要么将 wifi 路由器设置为使用固定的 wifi 信道，要么配置设备在当前信道上找不到预期节点时重新扫描 wifi 信道。

- MicroPython 在尝试重连时会重新扫描 wifi 信道：如果 ESP 设备连接的 wifi 接入点出现故障，MicroPython 会自动开始扫描信道，试图重新连接到该接入点。这意味着在扫描接入点期间，ESPNow 消息将会丢失。可以通过 `sta.config(reconnects=0)` 禁用此功能，这也会在断开连接后禁用自动重连。

- 某些版本的 ESP IDF 仅允许从 STA_IF 接口向已在与 STA_IF 相同的 wifi 信道上注册的节点发送 ESPNow 数据包。

## ESPNow 与睡眠模式

`machine.lightsleep([time_ms])` 和 `machine.deepsleep([time_ms])` 函数可用于使 ESP32 及其外设（包括 wifi 和蓝牙）进入睡眠状态。这在许多应用中对节省电池电量很有用。但是，应用程序在进入浅度睡眠或深度睡眠之前，必须禁用 WLAN 外设（使用 `active(False)`）。否则，从睡眠状态唤醒后，wifi 可能无法正确初始化。如果 STA_IF 和 AP_IF 接口都已设置为 `active(True)`，那么在进入任何睡眠模式之前，都应将这两个接口设置为 `active(False)`。

**深度睡眠示例：**
```python
import network, machine, espnow

sta, ap = wifi_reset()          # 重置 Wi-Fi，使 AP 关闭、STA 开启且断开连接
peer = b'0\xaa\xaa\xaa\xaa\xaa' # 节点的 MAC 地址
e = espnow.ESPNow()
e.active(True)
e.add_peer(peer)                # 在 STA_IF 上注册节点

print('Sending ping...')
if not e.send(peer, b'ping'):
    print('Ping failed!')
e.active(False)                 # 睡眠前禁用 Wi-Fi
sta.active(False)
print('Going to sleep...')
machine.deepsleep(10000)        # 睡眠 10 秒后重启
```

**浅度睡眠示例：**
```python
import network, machine, espnow

sta, ap = wifi_reset()          # 重置 Wi-Fi，使 AP 关闭、STA 开启且断开连接
sta.config(channel=6)
peer = b'0\xaa\xaa\xaa\xaa\xaa' # 节点的 MAC 地址
e = espnow.ESPNow()
e.active(True)
e.add_peer(peer)                # 在 STA_IF 上注册节点

while True:
    print('Sending ping...')
    if not e.send(peer, b'ping'):
        print('Ping failed!')
    sta.active(False)           # 睡眠前禁用 Wi-Fi
    print('Going to sleep...')
    machine.lightsleep(10000)   # 睡眠 10 秒
    sta.active(True)
    sta.config(channel=6)       # 浅度睡眠后 Wi-Fi 会丢失配置
```