# esp（与 ESP8266 和 ESP32 相关的函数）

`esp` 模块包含与 ESP8266 和 ESP32 相关的特定函数，有些函数仅在其中一个平台上可用。

## 函数

- esp.`sleep_type`([sleep_type])

  **注意**：仅适用于ESP8266。
  
  获取或设置睡眠类型。
  
  如果提供了 `sleep_type` 参数，则将设置睡眠类型；如果不带参数调用函数，则返回当前睡眠类型。
  
  可能的睡眠类型由以下常量定义：
  
  - `SLEEP_NONE` - 所有功能启用;
  - `SLEEP_MODEM` - 调制解调器睡眠，关闭 WiFi 调制解调器电路;
  - `SLEEP_LIGHT` - 浅睡眠，关闭 WiFi 调制解调器电路并定期暂停处理器。

  系统会在可能时自动进入设置的睡眠模式。
<br><br>

- esp.`deepsleep`(time_us=0, /)

  **注意**：仅适用于ESP8266，ESP32 请使用 `machine.deepsleep()`。
  
  进入深度睡眠模式。
  
  除RTC时钟电路外，整个模块断电。如果引脚 16 连接到复位引脚，RTC 可在指定时间后重启模块；否则模块将睡眠直至手动复位。
<br><br>

- esp.`flash_id`()

  **注意**：仅适用于ESP8266。
  
  读取闪存的设备ID。
<br><br>

- esp.`flash_size`()

  读取闪存的总容量。
<br><br>

- esp.`flash_user_start`()

  读取用户闪存空间的起始内存偏移量。
<br><br>

- esp.`flash_read`(byte_offset, length_or_buffer)
- esp.`flash_write`(byte_offset, bytes)
- esp.`flash_erase`(sector_no)
<br><br>

- esp.`osdebug`(uart_no)

  **说明**：这是 ESP8266 版本的函数说明。

  更改 OS 串行调试日志消息的级别。启动时，OS 串行调试日志消息默认禁用。
  
  `uart_no` 是接收 OS 级输出的 UART 外设编号，若为 None 则禁用 OS 串行调试日志消息。

- esp.`osdebug`(uart_no [, level])

  **说明**：这是 ESP32 版本的函数说明。

  更改 OS 串行调试日志消息的级别。启动时，OS 串行调试日志消息默认仅限制为错误输出。
  
  函数行为取决于传入的参数，支持以下组合：
  - `osdebug(None)`：恢复默认OS调试日志级别（`LOG_ERROR`）；
  - `osdebug(0)`：启用所有可用的OS调试日志消息（默认构建配置中为 `LOG_INFO`）；
  - `osdebug(0, level)`：将OS调试日志级别设置为指定值。日志级别由以下常量定义：
    - `LOG_NONE` - 无日志输出；
    - `LOG_ERROR` - 严重错误，软件模块无法自行恢复；
    - `LOG_WARN` - 已采取恢复措施的错误条件；
    - `LOG_INFO` - 描述事件正常流程的信息消息；
    - `LOG_DEBUG` - 正常使用非必需的额外信息（值、指针、大小等）；
    - `LOG_VERBOSE` - 大量调试信息或可能导致输出溢出的频繁消息。
  - **说明**：为节省空间，`LOG_DEBUG` 和 `LOG_VERBOSE` 默认不编译到 MicroPython 二进制文件中。如需查看这些级别的输出，需使用修改后的 "sdkconfig" 源文件进行自定义构建。
  - **说明**：ESP32 在"Raw REPL"模式下会自动暂停日志输出，以避免通信问题。这意味着使用 `mpremote run` 等工具时，永远看不到 OS 级日志。
<br><br>

- esp.`set_native_code_location`(start, length)

  **注意**：仅适用于ESP8266。
  
  设置原生代码编译后执行的存储位置。当函数被 `@micropython.native`、`@micropython.viper` 和 `@micropython.asm_xtensa` 装饰器修饰时，会生成原生代码。ESP8266 必须从 iRAM 或闪存的低 1MB区域（内存映射）执行代码，该函数用于控制存储位置。

  如果 `start` 和 `length` 均为 `None`，则原生代码位置设置为 iRAM1 区域末尾的未使用内存部分。该未使用部分的大小取决于固件，通常很小（约500字节），足以存储几个非常小的函数。使用 iRAM1 区域的优点是不会因写入而磨损。
  
  如果 `start` 和 `length` 均不为 `None`，则应为整数。`start` 指定原生代码在闪存中的字节偏移量，`length` 指定从 `start` 开始可用于存储原生代码的闪存字节数。`start` 和 `length` 应为扇区大小（4096字节）的倍数。写入前会自动擦除闪存，因此请确保使用未被固件或文件系统等其他用途占用的闪存区域。
  
  使用闪存存储原生代码时，start+length 必须小于或等于 1MB。请注意，频繁擦除（和写入）会导致闪存磨损，因此请谨慎使用此功能。特别地，原生代码每次启动（包括从深度睡眠唤醒）时都需要重新编译并写入闪存。
  
  在上述两种情况（使用 iRAM1 或闪存）中，如果指定区域没有剩余空间，对函数使用原生装饰器将导致在编译该函数时引发 MemoryError 异常。
