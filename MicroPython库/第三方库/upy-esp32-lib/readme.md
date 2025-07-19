# upy-esp32-lib   
[https://github.com/karfas/upy-esp32-lib](https://github.com/karfas/upy-esp32-lib)    
   
ESP32 的 micropython 通用工具库，包含 FIFO queue in RTC memory、mem\_map.py、rtc\_mem.py等。   
   
一个简单的先进先出队列，适用于深度休眠状态下的小量数据传输。使用uctypes来指定从/向队列传输的有效载荷。
   
```
import machine
import uctypes

from rtc_mem import RTCMemory
from mem_fifo import MemFifo

struct_def = {
    "time":             0 | uctypes.UINT32,
    "temperature":      4 | uctypes.INT16,       # 1/10 Celsius
    "pressure":         8 | uctypes.UINT16       # 1/10 hPa
}
samples = (
    [ 100, 20, 10320],
    [ 200, 21, 10321],
    [ 400,  5,  9000], # Storm!
    [ 500,  5, 10000], # Storm!
    )

# create memory map in RTC slow memory
rtc_mem = RTCMemory([
    'fifo',     50,             # space for ~3 entries + fifo header
    'something_else', 256
    ])

# create FIFO in the "fifo" area defined above
q = MemFifo(rtc_mem.fifo, struct_def)

def create_struct():
    arr = bytearray(uctypes.sizeof(struct_def))
    data = uctypes.struct(uctypes.addressof(arr), struct_def)
    return data

def overrun():
    for sample in samples:
        data = create_struct()
        data.time = sample[0]
        data.temperature = sample[1]
        data.pressure = sample[2]
        print("enqueue t={}".format(data.time))
        q.put_nowait(data)

def sleeptest():
    for sample in samples:
        data = create_struct()
        data.time = sample[0]
        data.temperature = sample[1]
        data.pressure = sample[2]
        print("enqueue t={}".format(data.time))
        q.put_nowait(data)
        machine.deepsleep(500)

def read_one():
    data = q.get_nowait()
    return data

def read():
    data = q.dequeue()
    while data is not None:
        print("data from queue: {} {} {}".format(data.time, data.temperature, data.pressure))
        data = q.get_nowait()
```
   
为内存中的某个区域赋予结构   
```
import uctypes
from mem_map import MemMap

a = bytearray(2048)     # some memory to manage
addr = uctypes.addressof(a)
map = MemMap(addr, [
    'fifo',     50,             # space for ~3 entries + fifo header
    'something_else', 256
    ])

fifo_addr = map.fifo.addr
something_addr = map.something_else.addr
```
   
