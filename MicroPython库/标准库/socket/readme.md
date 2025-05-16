# socket

socket 模块提供了对 BSD 套接字访问接口。

**和 CPython 的不同：**

  为了保证高效率和一致性，MicroPython 的 socket 对象直接使用了流（类似文件）接口。在 CPython 中，需要使用 `makefile()` 方法将 socket 进行转换。MicroPython 也支持这种方法（但是不常用），如果需要兼容 CPython， 可以使用这种方法。

## Socket 地址格式

`socket` 模块的本地套接字地址格式是 `getaddrinfo` 函数返回的不透明数据类型，它必须用于解析文本地址（包括数字地址）：

```py
sockaddr = socket.getaddrinfo('www.micropython.org', 80)[0][-1]

# 即使对于数字地址，也必须使用getaddrinfo()
sockaddr = socket.getaddrinfo('127.0.0.1', 80)[0][-1]

# 现在可以使用该地址
sock.connect(sockaddr)
```

使用 `getaddrinfo` 是处理地址最有效（无论是内存使用和处理能力方面）和最通用的方式。

然而，套接字模块（注意与此处描述的原生 MicroPython 套接字模块的区别）提了 CPython 兼容的方式，即使用元组来指定地址。注意，根据 MicroPython 移植版本的不同，套接字模块可能是内置的，也可能需要从  `micropython-lib` 安装（如 MicroPython 的 Unix 版本），并且一些版本仍然只接受元组格式的数字地址，并且需要使用 `getaddrinfo` 函数来解析域名。

**总结**：
- 编写可移植应用程序时，请始终使用 `getaddrinfo`。
- 如果系统版本支持，下面描述的元组地址可以用作快速设置和交互式使用。

**套接字模块的元组地址格式：**
- IPv4：(ipv4_address, port)。其中 `IPv4_access` 是一个带小数点符号的数字字符串，例如 "8.8.8.8"；`port` 是 1-65535 范围内的整数端口号。注意，`ipv4_address` 不能使用域名，应首先使用 `socket.getaddrinfo()` 进行解析。
- IPv6:(ipv6_address, port, flowinfo, scopeid)。其中 `IPv6_address` 是一个带有冒号表示法的 IPv6 数字地址的字符串，例如 "2001:db8::1"；`port` 是 1-65535 范围内的整数端口号；`flowinfo` 必须为 0；`scopeid` 是链接本地地址的接口作用域标识符。注意，`ipv6_address` 不能使用域名，应首先使用 `socket.getaddrinfo()` 进行解析。ipv6 是否可用取决于 MicroPythony 移植版本。

## 函数

- socket.`getaddrinfo`(host, port, af=0, type=0, proto=0, flags=0, / )

  传递主机/端口到一个 5 个数据的元组。元组列表的结构如下:
  将主机/端口参数转换为一个包含5元组的序列，元组包含创建连接到该服务的套接字所需的全部参数。参数 `af`、`type` 和 `proto`（含义与 `socket()` 函数相同）可用于过滤返回的地址类型。若未指定参数或参数为0，则可能返回所有地址组合（需用户自行过滤）。

  ```
  (family, type, proto, canonname, sockaddr)
  ```

  下面代码显示了怎样连接到一个网址:

  ```py
  s = socket.socket()
  
  # 假设如果未指定 "type"，则将返回 SOCK_STREAM 的地址
  s.connect(socket.getaddrinfo('www.micropython.org', 80)[0][-1])
  ```

  推荐使用过滤参数：

  ```py
  s = socket.socket()
  
  # 保证返回可连接到的地址以进行流操作。
  s.connect(socket.getaddrinfo('www.micropython.org', 80, 0, SOCK_STREAM)[0][-1])
  ```

**与 CPython 的差异**

  CPython 在 `getaddrinfo` 函数出错时引发 `socket.gaierror` 异常（`OSError` 子类 ） 。 MicroPython 没 有 `socket.gaierror` 类 ， 直接引发 `OSError`。 注意 `getaddrinfo()` 的错误编号形成一个单独的命名空间，可能与 `errno` 模块的错误号不匹配。为了区分 `getaddrinfo()` 错误，它的错误码用负数表示，而标准系统错误是正数（通过异常对象的`e.args[0]`属性能够获取错误码）。使用负数只是暂时的做法，未来可能会有所调整。
<br><br>

- socket.`inet_ntop`(af, bin_addr)

  将给定地址族 `af` 的二进制网络地址 `bin_addr` 转换为文本表示形式：

  ```py
  >>> socket.inet_ntop(socket.AF_INET, b"\x7f\0\0\1")
  '127.0.0.1'
  ```
<br>

- socket.`inet_pton`(af, txt_addr)

  将给定地址族 `af` 的文本网络地址 `txt_addr` 转换为二进制表示形式：
    
  ```
  >>> socket.inet_pton(socket.AF_INET, "1.2.3.4")
  b'\x01\x02\x03\x04'
  ```


## 常量

- socket.`AF_INET`
- socket.`AF_INET6`

  地址 family 类型，它的可用性依赖于 MicroPython 的移植版本。
<br><br>

- socket.`SOCK_STREAM`
- socket.`SOCK_DGRAM`

  socket 类型。
<br><br>

- socket.`IPPROTO_UDP`
- socket.`IPPROTO_TCP`

  IP 协议编号。其可用性取决于具体的 MicroPython 移植版。请注意，在调用 `socket.socket()` 时无需指定这些协议号，因为 `SOCK_STREAM` 套接字类型会自动选择 `IPPROTO_TCP`，而 `SOCK_DGRAM` 会自动选择 `IPPROTO_UDP`。因此，这些常量的实际用途仅作为 `setsockopt()` 的参数。
<br><br>

- socket.`SOL_*`

    Socket 选项级别( `setsockopt()` 函数的参数)，具体可用选项取决于 MicroPython 移植版本。
<br><br>

- socket.`SO_*`

    Socket 选项 ( `setsockopt()` 函数的参数)，具体的常量取决于 MicroPython 移植版本。
<br><br>

- socket.`IPPROTO_SEC`

    创建 SSL 兼容套接字的特殊协议值。


## class socket

- class socket.`socket`(af=AF_INET, type=SOCK_STREAM, proto=IPPROTO_TCP, / )

  使用指定的地址族、套接字类型和协议编号创建一个新的套接字。请注意，在大多数情况下不需要（也不建议）指定 `proto` 参数，因为某些 MicroPython 移植版可能未提供 `IPPROTO_*` 常量。相反，`type` 参数会自动选择所需的协议：

  ```py
  # 创建 STREAM TCP socket
  socket(AF_INET, SOCK_STREAM)
  # 创建 DGRAM UDP socket
  socket(AF_INET, SOCK_DGRAM)
  ```
<br><br>

## 方法

socket 相关方法。

- socket.`close`()

  将套接字标记为已关闭并释放所有资源。一旦执行此操作，套接字对象的所有后续操作都将失败。如果协议支持，远程端将收到EOF指示。

  套接字在被垃圾回收时会自动关闭，但建议在使用完毕后立即显式调用 `close()` 关闭它们。
<br><br>

- socket.`bind`(address)

  将套接字绑定到地址，套接字不能是已经绑定的。
<br><br>

- socket.`listen`([backlog])

  允许服务器接收连接。如果指定了 backlog，它不能小于 0 (如果小于 0 将自动设置为 0)；超出后系统将拒绝新的连接。如果没有指定，将使用默认值。
<br><br>

- socket.`accept`()

  接受一个连接。套接字必须已绑定到某个地址并正在监听连接。返回值是一个元组 (conn, address)，其中 conn 是一个新的套接字对象，用于在该连接上发送和接收数据，而 address 是连接另一端的套接字所绑定的地址。
<br><br>

- socket.`connect`(address)

  连接到指定地址的远端套接字。
<br><br>

- socket.`send`(bytes)

  发送数据，套接字需要已连接到远程。返回发送数据的数量，它可能比实际数据长度小 ("short write" 短写)。
<br><br>

- socket.`sendall`(bytes)

  将所有数据发送到套接字。套接字必须已连接到远端。与 `send()` 不同，此方法会尝试连续分块发送数据，直至全部发送完毕。

  此方法在非阻塞套接字上的行为未定义。因此，在 MicroPython 中，建议使用 `write()` 方法替代，对于阻塞套接字，`write()` 保持相同的"不短写"策略（即确保发送全部数据）；对于非阻塞套接字，`write()` 返回实际发送的字节数。
<br><br>

- socket.`recv`(bufsize)

  接收数据，返回值是接收数据的字节对象。`bufsize` 参数是接收数据的最大数量。
<br><br>

- socket.`sendto`(bytes, address)

  发送数据。套接字不能已连接到远端，目标套接字由 `address` 参数指定。
<br><br>

- socket.`recvfrom`(bufsize)

  从套接字接收数据。返回值是 (bytes, address)对，其中 `bytes` 是接收数据的字节对象，`address` 是发送数据的套接字地址。
<br><br>

- socket.`setsockopt`(level, optname, value)

  设置套接字的选项。需要的符号常量在套接字模块 (`SO_*` 等)中定义。`value` 可以是整数或字节对象。
<br><br>

- socket.`settimeout`(value)
  **注意**：并非每个移植版本都支持此方法。
  
  设置阻塞模式套接字超时时间。`value` 参数可以是代表秒的非负浮点数或 `None`。如果设定大于 0 的参数，在操作完成前超时会引发 `OSError` 异常。如果参数是 0，套接字将使用非阻塞模式。如果是 `None`，套接字使用阻塞模式。并不是每个 MicroPython 移植版都支持这种方法。一个更可移植和通用的解决方案是使用 `select.poll` 对象。这允许同时等待多个对象（不仅限于套接字，还包括支持轮询的通用流对象）。如：

  ```py
  # 不要使用:
  s.settimeout(1.0) # 超时时间 1 秒
  s.read(10)        # 可能超时
  
  # 而是使用:
  poller = select.poll()
  poller.register(s, select.POLLIN)
  res = poller.poll(1000)  # 时间是毫秒
  
  if not res:
      # res 未就绪，操作超时
  ```

  **和 CPython 的差异**

  在 CPython 中超时后将引发 `socket.timeout` 异常，它是 `OSError` 的子类，而 MicroPython 直接引发 `OSError` 异常。如果使用 `except OSError:` 捕捉异常，代码就可以同时用在 MicroPython 和 CPython 中。
<br><br>

- socket.`setblocking`(flag)

  设置阻塞或非阻塞模式: 如果 `flag` 是 `False`，设置非阻塞模式，否则设置阻塞模式。这是调用 `settimeout()` 的一种简便方法：

  - `sock.setblocking(True)` 等于 `sock.settimeout(None)`。
  - `sock.setblocking(False)` 等于 `sock.settimeout(0)`。
<br><br>

- socket.`makefile`(mode='rb', buffering=0 , / )

  返回关联到套接字的文件对象，返回值类型与指定的参数有关。仅支持二进制模式("rb"、"wb" 和 "rwb")，CPython 的 `encoding`、`errors` 和 `newline` 不被支持。

  **和 CPython 的差异**

  - 因为 MicroPython 不支持缓冲流，因此参数 `buffering` 被忽略，并认为是 0（不缓存）
  - 关闭 `makefile()` 返回的文件对象也会同时关闭套接字。
<br><br>

- socket.`read`([size])

  从套接字中最多读取 size 字节的数据，返回一个字节对象。如果未指定 `size` 参数，该方法会持续读取套接字中所有可用的数据，直到遇到文件结束符（EOF），也就是说，该方法会一直阻塞，直到套接字被关闭。此函数会尽量读取所有请求的数据（不会进行"短读"）。然而，在非阻塞模式的套接字下，这可能无法实现，此时将返回较少的数据。
<br><br>

- socket.`readinto`(buf[, nbytes])

  读取数据到缓冲区 `buf`。如果指定了 `nbytes`，那么最多只读取 `nbytes` 字节，否则最多读取 `len(buf)` 字节。和 `read()` 函数相同，这个函数使用了 "非短读取"策略。

  返回值是读取并存入 `buf` 的字节数。
<br><br>

- socket.`readline`()

  读取一行，以换行符结束。返回读取的行。
<br><br>

- socket.`write`(buf)

  将字节缓冲区 `buf` 写入套接字。此函数会尽量将所有数据写入套接字（不会进行"短写"）。然而，在非阻塞模式的套接字下，这可能无法实现，此时返回的字节数将少于 buf 的长度。

  返回值：实际写入的字节数。
<br><br>

- exception socket.`error`

  MicroPython 不支持此异常。在CPython中，曾经有一个 `socket.error` 异常，但现在它已被弃用，并且实际上是 `OSError` 的别名。在MicroPython中，应直接使用 `OSError`。

