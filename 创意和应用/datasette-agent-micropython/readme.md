# Datasette Agent Micropython 

MicroPython 在 WASM 沙盒中作为 Datasette 代理的工具，可以在浏览器中的micropython/WASM沙箱中运行Python代码。该插件将`execute_micropython`工具添加到Datasette代理中。可用于纯计算、解析、数据转换、数学和检查算法。输出是从stdout和stderr捕获的，因此代码可以使用`print()`返回数据。


https://github.com/datasette/datasette-agent-micropython