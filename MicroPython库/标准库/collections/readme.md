# 6. collections（集合和容器类型）   

这个模块实现了高级集合和容器类型，可以容纳各种对象。   
- class collections.`deque`(iterable, maxlen [, flags ] )   
  deques（双端队列）是一个类似列表的容器，支持从deques的两侧添加和弹出。使用以下参数创建新的deques：   
  - `iterable`必须是空元组，并且新的deque创建为空。   
  - 必须指定`maxlen`参数，deque将以此为最大长度。一旦deque已满，添加的任何新项都将丢弃另一端的项目。   
  - 可选标志位`flags`为1时，添加项目时将检查是否溢出。   
 
  deque对象除了支持bool、len、迭代和下标加载和存储外，还具有以下方法：   
  - `append`(x)，添加 x 到双端队列的右侧。如果启用了溢出检查并且没有多余空间，则引发`IndexError`。   
  - `appendleft`(x)，添加 x 到 双端队列左侧。 如果启用了溢出检查并且没有多余空间，则引发`IndexError`。   
  - `pop`(x)，从双端队列右侧移除并返回一个项目。如果没有项目存在，则引发`IndexError`。   
  - `popleft`()，从双端队列左侧移除并返回一个项目。如果不存在任何项，则引发`IndexError`。   
  - `extend`(iterable)，通过将 `iterable` 中的所有项附加到双端队列的右侧来扩展双端队列。如果启用了溢出检查并且双端队列中没有更多空间，则引发`IndexError`。   
<br>  

- classes collections.`namedtuple`(name, fields)   
  使用指定名称和字段创建新的命名元组类型。命名元组类型是元组的子集，不但可以用索引访问，也可以通过符号字段名访问，字段是指定名称的字符串序列。为了兼容 CPython，它也可以是用空格分隔的字符串字段名(但是效率很低)。基本使用方法:   
  ```py
  from collections import namedtuple
  
  MyTuple = namedtuple("MyTuple", ("id", "name"))
  t1 = MyTuple(1, "foo")
  t2 = MyTuple(2, "bar")
  print(t1.name)
  assert t2.name == t2[1]
  
  ```
  
- collections.`OrderedDict`(…)   
  字典类型的子集，它会按顺序保存添加的键值。当字典完成迭代，就会按照添加时的顺序返回:   
  ```py
  from collections import OrderedDict
  
  # 为了利用 ordered keys, 需要初始化OrderedDict
  d = OrderedDict([("z", 1), ("a", 2)])
  
  # 可以添加更多条目
  d["w"] = 5
  d["b"] = 3
  for k, v in d.items():
      print(k, v)
  
  ```
  输出结果为:   
  ```py
  z 1
  a 2
  w 5
  b 3
  
  ```
