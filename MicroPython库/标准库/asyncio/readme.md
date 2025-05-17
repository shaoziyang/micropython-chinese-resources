# asyncio（异步 I/O 调度器）

本模块实现了对应 CPython 模块的子集。更多信息请参考 CPython 官方文档：[asyncio](https://docs.python.org/3.8/library/asyncio.html)。

```py
import asyncio

async def blink(led, period_ms):
    while True:
        led.on()
        await asyncio.sleep_ms(5)
        led.off()
        await asyncio.sleep_ms(period_ms)

async def main(led1, led2):
    asyncio.create_task(blink(led1, 700))
    asyncio.create_task(blink(led2, 400))
    await asyncio.sleep_ms(10_000)

# 在 pyboard 上运行
from pyb import LED
asyncio.run(main(LED(1), LED(2)))

# 在其它通用开发板上运行
from machine import Pin
asyncio.run(main(Pin(1), Pin(2)))
```

## 核心函数

- asyncio.`create_task`(coro)

  从给定的协程创建一个新任务，并调度执行。

  返回对应的 `Task` 对象。
<br><br>

- asyncio.`current_task`()

  返回与当前正在运行的任务关联的 `Task` 对象。
<br><br>

- asyncio.`run`(coro)

  从给定的协程创建一个新任务，并运行直到完成。

  返回协程 `coro` 返回的值。
<br><br>

- asyncio.`sleep`(t)

  休眠 `t` 秒（可以为浮点数）。这是一个协程。
<br><br>

- asyncio.`sleep_ms`(t)

  休眠 `t` 毫秒。这是一个协程，也是 MicroPython 的扩展。
<br><br>


## 扩展函数

- asyncio.`wait_for`(awaitable, timeout)

  等待 `awaitable` 对象完成，但如果执行时间超过 `timeout` 秒则将其取消。如果 `awaitable` 不是任务，则会将其创建为任务。

  若发生超时，它会取消任务并抛出 `asyncio.TimeoutError`，调用者应捕获此异常。任务会收到 `asyncio.CancelledError`，可通过 `try...except` 或 `try...finally` 忽略或捕获该异常以执行清理代码。

  返回 `awaitable` 对象的返回值。

  这是一个协程。
<br><br>

- asyncio.`wait_for_ms`(awaitable, timeout)

  与 `wait_for` 类似，但 `timeout` 是以毫秒为单位的整数。

  这是一个协程，也是 MicroPython 的扩展功能。
<br><br>

- asyncio.`gather`(*awaitables, return_exceptions=False)

  并发运行所有 `awaitables` 对象。任何非任务的 `awaitables` 对象都会被提升为任务。

  返回所有 `awaitables` 对象的返回值列表。

  这是一个协程。
<br><br>


## class Task

- class asyncio.`Task`

  将协程包装为一个正在运行的任务。可以使用 `await task` 等待任务完成，这将阻塞直到任务结束并返回其返回值。

  注意：不应直接创建任务实例，而应使用 `create_task` 函数来创建。
<br><br>

- Task.`cancel`()

  通过向任务注入 `asyncio.CancelledError` 异常来取消任务。任务可能会忽略此异常。可通过捕获该异常或使用 `try...finally` 块来执行清理代码。
<br><br>


## class Event

- class asyncio.`Event`

  创建一个可用于任务同步的新事件。事件初始处于清除状态。
<br><br>

- Event.`is_set`()

  如果事件已设置，返回 `True`，否则返回 `False`。
<br><br>

- Event.`set`()

  设置事件。所有等待该事件的任务将被调度运行。

  **注意**：此方法必须在任务中调用。从中断处理程序（IRQ）、调度器回调或其他线程中调用不安全。请参见 `ThreadSafeFlag`。
<br><br>

- Event.`clear`()

  清除事件。
<br><br>

- Event.`wait`()

  等待事件被设置。如果事件已设置，则立即返回。

  这是一个协程。
<br><br>


## class ThreadSafeFlag

- class asyncio.`ThreadSafeFlag`

  创建一个新标志，用于与在 `asyncio` 事件循环外运行的代码，如其他线程、中断处理程序或调度器回调进行同步。标志初始处于清除状态。
<br><br>

- ThreadSafeFlag.`set`()

  设置标志。如果有任务在等待该标志，将调度运行。
<br><br>

- ThreadSafeFlag.`clear`()

  清除标志。这可用于确保在等待标志前，清除标志可能的已设置状态。
<br><br>

- ThreadSafeFlag.`wait`()

  等待标志被设置。如果标志已设置，则立即返回。`wait` 返回后，标志会自动重置。

  注意：同一时刻只能有一个任务等待该标志。

  这是一个协程。
<br><br>


## class Lock

- class asyncio.`Lock`

  创建一个可用于协调任务的新锁。锁初始处于未锁定状态。

  除以下方法外，锁还可在 `async with` 语句中使用。
<br><br>

- Lock.`locked`()

  如果锁处于锁定状态，返回 `True`，否则返回 `False`。
<br><br>

- Lock.`acquire`()

  等待锁处于未锁定状态，然后以原子操作锁定它。任一时刻只能有一个任务获取该锁。

  这是一个协程。
<br><br>

- Lock.`release`()

  释放锁。如果有任务在等待该锁，则调度队列中的下一个任务运行，且锁保持锁定状态；否则，如果没有任务等待，锁变为未锁定状态。
<br><br>


## TCP 流连接

- asyncio.`open_connection`(host, port, ssl=None)

  打开给定主机和端口的 TCP 连接。`host` 地址将通过 `socket.getaddrinfo` 解析（该操作目前是阻塞的）。如果 `ssl` 是 `ssl.SSLContext` 对象，则使用该上下文创建传输；如果 `ssl` 为 `True`，则使用默认上下文。

  返回一对流对象：一个读取流和一个写入流。如果无法解析主机或建立连接，将抛出特定于套接字的 `OSError`。

  这是一个协程。
<br><br>

- asyncio.`start_server`(callback, host, port, backlog=5, ssl=None)

  用给定 `host` 和 `port` 启动 TCP 服务器。当连接有传入时，将调用 `callback`，并传入两个参数：连接对应的读取流和写入流。

  如果 `ssl` 是 `ssl.SSLContext` 对象，则使用该上下文创建传输。

  返回一个 `Server` 对象。

  这是一个协程。
<br><br>


### class asyncio.`Stream`

表示 TCP 流连接。为了精简代码，此类同时实现了读取器和写入器的功能，因此 `StreamReader` 和 `StreamWriter` 均为此类的别名。

- Stream.`get_extra_info`(v)

  获取由参数 `v` 指定的流额外信息。`v` 的有效值为：`peername`（对等方地址）。
<br><br>

- Stream.`close`()

  关闭流。
<br><br>

- Stream.`wait_closed`()

  等待流关闭。

  这是一个协程。
<br><br>

- Stream.`read`(n=-1)

  读取最多 `n` 字节并返回。如果未提供 `n` 或 `n` 为 `-1`，则读取所有字节直至遇到 EOF（文件结束符）。如果在读取任何字节前就已遇到 EOF，将返回空字节对象 `b''`。

  这是一个协程。
<br><br>

- Stream.`readinto`(buf)

  将最多 `n` 字节读取到缓冲区 `buf` 中（`n` 为缓冲区长度）。

  返回实际读取到缓冲区中的字节数。

  这是一个协程，也是 MicroPython 的扩展功能。
<br><br>

- Stream.`readexactly`(n)

  精确读取 `n` 字节并以字节对象形式返回。

  如果在读取 `n` 字节前流已结束，将抛出 `EOFError` 异常。

  这是一个协程。
<br><br>

- Stream.`readline`()

  读取一行并返回。

  这是一个协程。
<br><br>

- Stream.`write`(buf)

  将 `buf` 数据累积到输出缓冲区中。数据仅在调用 `Stream.drain()` 时才会被刷新。建议在此函数调用后立即调用 `Stream.drain()`。
<br><br>

- Stream.`drain`()

  将所有缓冲的输出数据刷新（写入）到流中。

  这是一个协程。
<br><br>

### class asyncio.Server

表示从 `start_server()` 函数返回的服务器对象。它可用于 `async with` 语句中，以在退出时自动关闭服务器。

- Server.`close`()

  关闭服务器。
<br><br>

- Server.`wait_closed`()

  等待服务器关闭。

  这是一个协程。
<br><br>

## Event Loop

- asyncio.`get_event_loop`()

  返回用于调度和运行任务的事件循环。 
<br><br>

- asyncio.`new_event_loop`()  

  重置事件循环并返回它。  

  **注意**：由于 MicroPython 只有一个事件循环，此函数仅重置循环的状态，而不会创建新的循环。  
<br><br>

### class asyncio.Loop  

  这表示用于调度和运行任务的对象。无法直接创建该对象，用 `get_event_loop` 获取。  

- Loop.`create_task`(coro)  

  用给定的 `coro` 协程创建任务，并返回新的 `Task` 对象。  
<br><br>

- Loop.`run_forever`()  

  运行事件循环，直至调用 `stop()` 方法。  
<br><br>

- Loop.`run_until_complete`(awaitable)  

  运行给定的 `awaitable` 对象，直至其完成。如果 `awaitable` 不是任务，将被提升为任务。  
<br><br>

- Loop.`stop`()  

  停止事件循环。  
<br><br>

- Loop.`close`()  

  关闭事件循环。  
<br><br>

- Loop.`set_exception_handler`(handler)  

  设置当 `Task` 引发未捕获的异常时调用的异常处理程序。处理程序应接受两个参数：(`loop`, `context`)。  
<br><br>

- Loop.`get_exception_handler`()  

  获取当前的异常处理程序。返回值是异常处理程序，若未设置自定义处理程序则返回 `None`。  
<br><br>

- Loop.`default_exception_handler`(context)  

  被调用的默认异常处理程序。  
<br><br>

- Loop.`call_exception_handler`(context)  

  调用当前的异常处理程序。参数 `context` 会被传递，它是一个包含以下键的字典：'message'、'exception'、'future'。
