# ssl（SSL/TLS 模块）

此模块为客户端和服务器端的网络套接字提供传输层安全性（以前广泛称为“安全套接字层”）加密和对等身份验证功能的访问。

##  函数
- ssl.`wrap_socket`(sock, server_side=False, keyfile=None, cert=None, cert_reqs=CERT_NONE, cadata=None, server_hostname=None, do_handshake=True)

  包装给定的套接字，并返回一个新包装的套接字对象。该函数的实现是首先创建一个`SSLContext`，然后在该上下文对象上调用 `SSLContext.wrap_socket` 方法。参数 `sock`、`server_side` 和 `server_hostname` 将原封不动地传递给方法调用。参数 `do_handshake` 作为 `do_handhake_on_connect` 传递。其它参数具有以下行为：

  - `cert_reqs` 确定对方（服务器或客户端）是否必须提供有效的证书。请注意，对于基于 MbedTLS 的版本，`ssl.CERT_NONE`（不要求证书）和 `ssl.CERT_OPTIONAL`（证书可选）不会验证任何证书，只有 `ssl.CERT_REQUIRED`（要求证书）会进行验证。
  - `cadata` 是一个字节对象，其中包含证书颁发机构（CA）证书链（采用 DER 格式），该证书链将用于验证对等方的证书。目前仅支持单个 DER 编码的证书。

  根据特定 MicroPython 移植版本中底层模块的实现情况，上述部分或全部关键字参数可能不被支持。
  
  
## class SSLContext

- class ssl.`SSLContext`(protocol, / )

  创建一个新的 SSLContext 实例。协议参数必须是 `protocol_*` 常量之一。
<br><br>

- SSLContext.`load_cert_chain`(certfile, keyfile)

  加载私钥和相应的证书。`certfile` 是一个包含证书文件路径的字符串。`keyfile` 是一个包含私钥文件路径的字符串。
<br>

  **与 Cpython 差异**
<br>

  MicroPython 扩展：`certfile` 和 `keyfile` 可以是字节对象，而不是字符串，在这种情况下，它们被解释为实际的证书 / 密钥数据。
<br><br>

- SSLContext.`load_verify_locations`(cafile=None, cadata=None)

  加载用于验证对等方证书的 CA 证书链。`cafile` 是 CA 证书的文件路径。`cadata` 是一个包含 CA 证书的字节对象。上述两个参数只应提供其中一个。
<br><br>

- SSLContext.`get_ciphers`()

  获取已启用密码的列表，以字符串列表的形式返回。
<br><br>

- SSLContext.`set_ciphers`(ciphers)

  设置在此上下文中创建的套接字的可用密码。`ciphers` 是一个字符串列表，采用互联网号码分配机构（IANA）密码套件格式。
<br><br>

- SSLContext.`wrap_socket`(sock, *, server_side=False, do_handshake_on_connect=True, server_hostname=None)

  获取一个流 sock（通常是 sock_stream 类型的 socket.socket 实例），并返回一个 ssl.SSLSocket 实例，包装底层流。返回的对象具有常见的流接口方法，如 `read()`、`write()` 等。
<br>

  - `server_side`（服务器端）参数用于选择被包装的套接字是处于服务器端还是客户端。服务器端的 SSL 套接字应当由非 SSL 监听服务器套接字调用 `accept()` 方法返回的普通套接字创建。
  - `do_handshake_on_connect`（连接时执行握手）决定了握手操作是作为 `wrap_socket` 的一部分立即执行，还是推迟到首次读写操作时执行。对于阻塞套接字而言，立即执行握手是标准做法。对于非阻塞套接字（即传入 `wrap_socket` 的sock处于非阻塞模式时），通常应当推迟握手，否则 `wrap_socket` 会阻塞直到握手完成。请注意，在 AXTLS 中，握手可以推迟到首次读写时执行，但此时会阻塞直到完成。
  - `server_hostname`（服务器主机名）供客户端使用，用于设置要与接收到的服务器证书进行比对的主机名。它还会设置服务器名称指示（SNI）的名称，使服务器能够呈现正确的证书。
<br><br>

  **警告**
  
  某些 `ssl` 模块的实现不会验证服务器证书，这使得建立的 SSL 连接容易受到中间人攻击。

  CPython 的 `wrap_socket` 返回一个 `SSLSocket` 对象，该对象具有 socket 的典型方法（如`send`、`recv`等）。而 MicroPython 的 `wrap_socket` 返回的对象更类似于 CPython 的 `SSLObject`，不直接提供这些套接字方法。
<br><br>

- SSLContext.`verify_mode`

  设置或获取对等证书验证的行为。必须是 `CERT_*` 常量之一。

  注：`ssl.CERT_REQUERED` 要求正确设置设备的日期/时间，例如使用 `mpremote rtc —set` 或 `ntptime`，客户端必须指定 `server_hostname`。

  
## 异常

- ssl.`SSLError`

  在 MicroPython 中，不存在 `ssl.SSLError` 异常，请使用它的基类 `OSError`。

## DTLS 支持

**与 CPython 的差异**

这是 MicroPython 的扩展功能。

该模块通过 `PROTOCOL_DTLS_CLIENT` 和 `PROTOCOL_DTLS_SERVER` 常量支持客户端和服务器模式的 DTLS（数据报传输层安全协议），这两个常量可作为 `SSLContext` 的 protocol 参数使用。

在此情况下，底层套接字应表现为数据报套接字（即类似于通过 socket.socket 创建的套接字，其中 `af` 参数为 `socket.AF_INET`，`type` 参数为 `socket.SOCK_DGRAM`）。

DTLS 仅在使用 mbed TLS 的版本上受支持，并且默认情况下未启用：需要在特定版本的配置中启用 `MBEDTLS_SSL_PROTO_DTLS` 才能使用。
  

## 常数

相关常数。

- ssl.`PROTOCOL_TLS_CLIENT`
- ssl.`PROTOCOL_TLS_SERVER`
- ssl.`PROTOCOL_DTLS_CLIENT`
- ssl.`PROTOCOL_DTLS_SERVER`

  协议参数支持的参数值。
<br><br>

- ssl.`CERT_NONE`
- ssl.`CERT_OPTIONAL`
- ssl.`CERT_REQUIRED`

  `cert_reqs` 参数和 `SSLContext.verify_mode` 属性。
