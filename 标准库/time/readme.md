# time（时间函数）

time 模块提供了获取时间和日期、计算时间间隔、延时等函数。

**初始时刻**: Unix 使用了 POSIX 的系统标准，从 1970-01-01 00:00:00 UTC开始计数。某些嵌入式版本是从 2000-01-01 00:00:00 UTC 开始计算的，起始时间可以通过 `time.gmtime(0)` 函数查看。

**实际日历的日期/时间维护**: 需要一个实时时钟 (RTC)。在系统底层 (包括一些 RTOS 中)， 已经包含了 RTC 功能。设置时间是通过 OS/RTOS，而不是 MicroPython 完成的，查询日期/时间也需要通过系统 API。对于裸机系统，时钟依赖于 `machine.RTC()` 对象。设置时间是通过 `machine.RTC().datetime(tuple)` 函数，并通过下面方式维护:

- 后备电池 (可能是可选附件、扩展板等)。
- 使用网络时间协议 (需要用户设置)。
- 每次上电时手工设置 (许多开发板在硬复位后仍能保持实时时钟时间，有些开发板可能需要再次设置时间)。
 
如果时间不是通过系统/MicroPython 的 RTC 进行维护，那么下面与时间相关函数的结果可能和预期的不完全相同。

## 函数

time 模块的函数。

- time.`gmtime`([secs])
- time.`localtime`([secs])

  将从初始时间开始计算的秒转换为对应的元组: (年, 月, 日, 时, 分, 秒, 星期, yearday) 。如果 `secs` 未提供或者是 `None`，那么使用 RTC 的当前时间。
  
  `gmtime()` 函数返回 UTC 格式的日期时间元组，`localtime()` 则返回本地时间格式的日期时间元组，很多时候它们是相同的。

  - 年 (包括了世纪的数字，例如 2014)
  - 月范围是 1-12
  - 日范围是 1-31
  - 小时范围是 0-23
  - 分钟范围是 0-59
  - 秒范围是 0-59
  - 星期范围是 0-6，代表周一到周日
  - yearday 范围是 1-366
<br><br>

  例如：
  ```py
  >>> time.localtime(0)
  (2000, 1, 1, 0, 0, 0, 5, 1)
  >>> time.localtime()
  (2015, 1, 1, 0, 2, 3, 3, 1)
  >>> time.localtime(480000000)
  (2015, 3, 18, 13, 20, 0, 2, 77)
  ```
<br>

- time.`mktime`()

  localtime 的反函数，它的参数是一个完整 8 元素的元组，返回值是从 2000 年 1月 1 日开始的秒数。

    
  ```py
  >>> time.mktime((2015, 1, 1, 0, 3, 1, 3, 1))
  473385781
  >>> time.mktime((2017, 5, 10, 22, 20, 12, 2, 130))
  547770012
  >>> time.localtime(547770012)
  (2017, 5, 10, 22, 20, 12, 2, 130)
  ```
<br>

- time.`sleep`(seconds)

  延时按照指定的秒数进行休眠。部分开发板能够接收浮点数类型的秒数，以此实现亚秒级的精确休眠。不过，有些开发板可能不支持浮点型参数，为了保证代码在这些开发板上的兼容性，建议使用 `sleep_ms()` 和 `sleep_us()` 函数。
<br><br>

- time.`sleep_ms`(ms)

  按给定的毫秒数延迟，数值应为正数或 0。

  此函数将至少延迟指定的毫秒数，但如果有其他处理（如中断处理程序或其他线程）必须执行，实际延迟时间可能更长。传入 0 毫秒仍会允许执行其他处理。如需更精确的延迟，请使用 `sleep_us()` 函数。
<br><br>

- time.`sleep_us`(us)

  功能和 sleep_ms 函数类似，但是延时时间是微秒。
<br><br>

- time.`ticks_ms`()

  返回一个从任意参考点开始递增的毫秒计数器，该计数器在达到某个值后会重新开始计数(回绕)。
  
  计数值本身没有特定意义，但为了简化讨论，我们将其称为 `TICKS_MAX`。

  计数值的周期为 $ TICKS_PERIOD = TICKS_MAX + 1 $。TICKS_PERIOD 的数值一定是 2 的指数，但是具体数值在不同的移植版本上差异很大。为简化起见，`ticks_ms()`、`ticks_us()`、`ticks_cpu()` 函数使用相同的周期值。因此，这些函数返回的值范围为 [0 .. TICKS_MAX]（包含端点），共有 TICKS_PERIOD 个值，请注意这里不使用负数。在大多数情况下，应将这些函数返回的值视为不可直接操作的 "不透明值"，仅可对其使用以下描述的 `ticks_diff()` 和 `ticks_add()` 函数进行操作。

  注意：直接对这些值执行标准数学运算（+、-）或关系运算符（<、<=、>、>=）是没有直接意义的。先执行数学运算再将结果作为参数传递给 `ticks_diff()` 或 `ticks_add()`，也会导致返回错误结果。
<br><br>

- time.`ticks_us`()

  和 ticks_ms()函数类似，只是返回值是微秒。
<br><br>

- time.`ticks_cpu`()

  与 `ticks_ms()` 和 `ticks_us()` 类似，但使用系统中可能的最高分辨率计时单位，这通常是 CPU 时钟周期，因此函数如此命名。但它不一定是 CPU 时钟，系统中可用的其他定时源（如高分辨率定时器）也可能被使用。此函数的确切计时单位（分辨率）未在 `time` 模块层面指定，但特定端口的文档可能提供更具体的信息。该函数适用于极高精度的基准测试或极严格的实时循环，应避免在可移植代码中使用。
  
  某些移植版中没有这个函数。
<br><br>

- time.`ticks_add`(ticks, delta)

  将 ticks 值偏移给定的增量`delta`（可正可负）。对于给定的 ticks 值，该函数可根据计时值的模运算（参见上文 `ticks_ms()`）计算 `ticks` 值之前或之后 `delta` 个计时单位的变化。`ticks` 参数必须是 `ticks_ms()`、`ticks_us()`、`ticks_cpu()` 函数的直接返回值（或之前 `ticks_add()` 的调用结果），但 `delta` 可以是任意整数或数值表达式。`ticks_add()` 可用于计算事件 / 任务的截止时间（注意：必须使用 `ticks_diff()` 函数处理截止时间）。
  
  ```py
  # 100ms 之前的 ticks 值
  print(ticks_add(time.ticks_ms(), -100))
  
  # 计算截止时间并执行操作
  deadline = ticks_add(time.ticks_ms(), 200)
  while ticks_diff(deadline, time.ticks_ms()) > 0:
      do_a_little_of_something()
  
  # 打印当前系统的 TICKS_MAX 值
  print(ticks_add(0, -1))
  ```


- time.`ticks_diff`(ticks1, ticks2)

  计算 `ticks_ms()`, `ticks_us()`, 或 `ticks_cpu()` 函数返回值之间的时间差，返回一个可能发生溢出的有符号数值。
  
  参数顺序与减法运算符一致，`ticks_diff(ticks1, ticks2)` 的含义等同于 ticks1 - ticks2。然而，ticks_ms() 等函数返回的值可能发生回绕，因此直接对其使用减法将可能产生错误结果。这正是需要 `ticks_diff()` 的原因 —— 它实现了模运算（或更具体地说，环形运算），即使对于溢出值也能产生正确结果（前提是两者间隔不超过一定范围，见下文）。函数返回的**有符号值**范围为 [-TICKS_PERIOD/2 .. TICKS_PERIOD/2-1]（这是有符号二进制整数的典型范围定义）。若结果为负数，表示 `ticks1` 在时间上早于 `ticks2`；否则表示 `ticks1` 发生在 `ticks2` 之后。这一结论**仅**在 `ticks1` 和 `ticks2` 的间隔不超过 TICKS_PERIOD/2-1 个计时单位时成立。若超出此范围，将返回错误结果。具体来说：
  
  通俗理解上述限制的逻辑：假设你被锁在一个房间里，除了一个标准的 12 小时制时钟外没有其它方式知道时间的变化。如果你现在看了一眼时钟，然后睡了 13 小时后再次查看，你可能会误以为只过去了 1 小时。为避免这种误判，你需要定期查看时钟。你的应用程序也应遵循同样的逻辑 —— 不要让你的系统运行任何单一任务时间过长，而是将任务拆分为多个步骤，并在步骤间进行计时。

  `ticks_diff()` 函数被设计为适合多种使用场景，如：

  - **带超时的轮询**：在这种情况下，事件的先后顺序是已知的，并且只需处理 `ticks_diff()` 函数的正结果部分。
  
  ```py
  # 等待 GPIO 变化，最长 500us
  start = time.ticks_us()
  while pin.value() == 0:
      if time.ticks_diff(time.ticks_us(), start) > 500:
          raise TimeoutError
  ```
  
  - **事件调度**：在这种情况下，如果事件已过期，ticks_diff() 的结果可能为负数。
  
  ```py
  # 这个代码段没有进行优化
  now = time.ticks_ms()
  scheduled_time = task.scheduled_time()
  if ticks_diff(now, scheduled_time) > 0:
      print("Too early, let's nap")
      sleep_ms(ticks_diff(now, scheduled_time))
      task.run()
  elif ticks_diff(now, scheduled_time) == 0:
      print("Right at time!")
      task.run()
  elif ticks_diff(now, scheduled_time) < 0:
      print("Oops, running late, tell task to run faster!")
      task.run(run_faster=True)
  ```
  
  **注意**：不要将 `time()` 的值传递给 `ticks_diff()`，而应直接对其使用常规数学运算。但需注意，`time()` 的值也可能（并且会）溢出，这就是众所周知的 2038 年问题。
 
  https://en.wikipedia.org/wiki/Year_2038_problem
<br><br>

- time.`time`()

  返回从纪元开始时间计算的秒数（整数），这里假设 RTC 已经按照前面方法设置好并正常运行。如果没有设置 RTC，函数将返回参考点开始计算的秒数 (对于没有后备电池的开发板，通常是上电或复位后的时间)。如果开发可移植版的 MicroPython 应用程序，不要依赖这个函数来提供超过秒级精度的时间。更高精度的绝对时间戳，请使用 `time_ns()`；如果可以接受相对时间，请使用 `ticks_ms()` 和 `ticks_us()` 函数；如果需要日历时间，不带参数的 `gmtime()` 或 `localtime()` 是更好的选择。
  
  ```py
  >>> time.time()
  473385600
  >>> time.time()
  473385602
  ```

  **与 CPython 的区别**

  在 CPython 中，这个函数用浮点数返回从 Unix 纪元时间（1970-01-01 00:00UTC）的秒数，通常是微秒级的精度。在 MicroPython 中，只有 Unix 移植版才使用相同的纪元时间，并且在浮点精度允许的情况下返回亚秒级精度。嵌入式硬件通常无法同时用浮点精度表示长时间范围和亚秒级精度，因此它们使用秒级精度的整数值。某些嵌入式系统硬件不支持 RTC 电池供电方式，因此返回自上次上电或其他特定于硬件的相对时间点（如复位）以来的秒数。
<br><br>

- time.`time_ns`()

  类似于 `time()`，但以整数方式返回自初始时间以来的纳秒（通常是一个大整数，因此将在堆上分配空间）。
