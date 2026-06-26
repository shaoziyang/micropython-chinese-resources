# CircuitPython MCP Server

使用 adafruit_httpserver 为 CircuitPython 项目提供最小 MCP JSON-RPC 服务器助手。

此库允许 CircuitPython 板直接在设备上运行真正的 MCP 服务器。主机仅用于部署、串行日志和测试。

已实现的HTTP端点：
- `GET/`：JSON状态
- `POST/mcp`：基于HTTP的mcp JSON-RPC
- `GET/mcp`：方法不允许的解释
- `DELETE/mcp`：方法不允许的解释
- `OPTIONS/mcp`：允许CORS
- `GET/sse`和`POST/messages`：当安装的`adafruit_httpserver`具有`SSEResponse`时的遗留sse兼容性

## 基本用法

创建 adafruit_httpserver.Server，将其传递给 MCPServer，注册工具回调，然后在主循环中调用 `poll()`。

```python
from adafruit_httpserver import Server
from circuitpython_mcp_server import MCPServer, schema_object, tool_result

http_server = Server(pool, debug=False)
mcp = MCPServer(
    http_server,
    name="example-board",
    title="Example Board MCP Server",
    version="0.1.0",
    instructions="Controls hardware on this CircuitPython board.",
)

def hello(arguments):
    return tool_result("Hello from CircuitPython.", {"ok": True, "action": "hello"})

mcp.add_tool("hello", "Return a greeting.", schema_object(), hello, title="Hello")
mcp.start(ip_address, port=5000)

while True:
    mcp.poll()
```

工具回调从 params.arguments 接收 JSON 对象。返回一个有效的 MCP 工具结果，通常使用 tool_result() 构建。验证失败将引发 `ValueError` 或 `ToolError`，该库会将其映射到 JSON-RPC -32602。

## 相关链接

https://github.com/speccy88/circuitpython-mcp-server

