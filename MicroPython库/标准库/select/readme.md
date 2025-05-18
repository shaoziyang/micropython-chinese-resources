# select（数据流等待事件）

select 模块实现了 CPython 相应模块的子集，提供了高效率等待多个流（即选择就绪可进行操作的流）上事件的功能。

## 函数

- select.`poll`()

  创建轮询 (poll) 实例。
<br><br>

- select.`select`(rlist, wlist, xlist[, timeout])

  等待活动对象。这个函数是为了兼容，效率不高，**推荐用 Poll 函数**。
<br><br>

## class Poll

### 方法

- poll.`register`(obj[, eventmask])

  为轮询注册流对象 `obj`。`eventmask` 支持下面参数的逻辑或（OR）操作:
  - `select.POLLIN` - 有数据可读取。
  - `select.POLLOUT` - 可以写入更多数据。
<br><br>

  注意 `select.POLLHUP` 和 `select.POLLERR` 标志位作为输入事件掩码是无效的（这些是主动上报的事件，无论是否请求，`poll()` 都会返回）。这符合 POSIX 规范。
<br><br>

  `eventmask` 默认值是 `select.POLLIN | select.POLLOUT`。
<br><br>

  可以对同一个 `obj` 多次调用此函数。连续调用会将 `obj` 的 eventmask 更新为 `eventmask` 的值（等效 `modify()`）。
<br><br>

- poll.`unregister`(obj)

  解除轮询对象。
<br><br>

- poll.`modify`(obj, eventmask)

  修改对象的 `eventmask`。如果对象 `obj` 未被注册，则会引发 `OSError` 异常，错误类型为 `ENOENT`。
<br><br>

- poll.`poll`([timeout])

  等待至少一个已注册的对象变为就绪状态或出现异常情况，可选择设置以毫秒为单位的超时时间（若未指定 `timeout` 参数或其值为 -1，则无超时限制）。
<br><br>

  返回一个由 (obj, event, ...) 元组组成的列表。元组中可能包含其他元素，具体取决于平台和版本，因此不要假设其长度固定为 2。`event` 元素表示流上发生的事件，是上述 `select.POLL*` 常量的组合。请注意，`select.POLLHUP` 和 `select.POLLERR` 标志可能随时返回（即使未主动请求），必须对其进行相应处理（相应的流从poll中注销并且可能已关闭），否则后续的所有 `poll()` 调用可能会立即返回，并再次为该流设置这些标志。
<br><br>

  若超时，返回空列表。
<br><br>

- poll.`ipoll`(timeout=-1, flags=0 , / )

  与 `poll.poll()` 类似，但返回一个迭代器，该迭代器生成一个被调用者拥有的元组。此功能提供了一种高效、无分配内存的流轮询方式。

  如果 `flags` 为 1，则对事件采用一次性触发模式：发生事件的流将自动重置其事件掩码（等同于 `poll.modify(obj, 0)`），因此在使用 `poll.modify()` 设置新掩码之前，不会处理该流的新事件。这种行为对异步 I/O 调度器很有用。
