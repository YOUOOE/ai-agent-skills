---
name: mcp-server-development-guide
description: 从零搭建MCP Server的完整实战指南 - 协议解读、工具注册、最佳实践与常见陷阱
platforms: [hermes, claude-code, codex, cursor, windsurf]
tags: [mcp, server, protocol, development, tools, integration]
difficulty: intermediate
time_required: 2-3小时
prerequisites: Python/Node.js基础
---

# 🛠 MCP Server 开发实战指南

## 概述

MCP (Model Context Protocol) 正在成为AI Agent工具的标准协议。本指南从零带你搭建一个生产级MCP Server，包括协议选型、工具注册、安全加固、部署运维全套流程。

> **适用场景**：你想让AI Agent能访问文件系统、数据库、搜索引擎，或者自定义API接口。

## 一、选型决策

| 场景 | 推荐方案 | 资源占用 |
|------|----------|----------|
| 快速原型 | Python + FastMCP | ~20MB |
| 高性能生产 | TypeScript + SDK | ~50MB |
| 已有代码库 | Python stdio模式 | ~15MB |

**建议**：Python起步。语言门槛低、生态系统成熟、排错方便。

## 二、最小实现 (Python)

```python
# server.py
# 安装: pip install "mcp[cli]>=1.0.0"

from mcp.server.fastmcp import FastMCP
import httpx

# 1. 创建Server实例
mcp = FastMCP("my-tools", host="127.0.0.1", port=8007)

# 2. 注册工具 - 无需装饰器
def get_current_time():
    """返回当前时间"""
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

def web_search(query: str):
    """搜索互联网

    Args:
        query: 搜索关键词
    """
    try:
        r = httpx.get(f"https://api.anysearch.com/v1/search?q={query}",
                      headers={"Authorization": "Bearer YOUR_KEY"},
                      timeout=10)
        results = r.json()["data"]["results"][:3]
        return "\n".join(f"[{i['title']}]({i['url']})\n{i['snippet']}"
                         for i in results)
    except Exception as e:
        return f"搜索失败: {e}"

# 3. 注册到MCP
mcp.add_tool(get_current_time)
mcp.add_tool(web_search)

# 4. 启动
if __name__ == "__main__":
    mcp.run(transport="stdio")  # 推荐stdio模式
```

## 三、Transport模式对比

| 模式 | 优点 | 缺点 | 适用 |
|------|------|------|------|
| **stdio** | 零配置、安全隔离、进程级 | 不能远程调用 | **推荐**，99%场景 |
| SSE/HTTP | 可远程、多客户端 | 需端口暴露、安全性低 | 团队分享、Web场景 |

> **经验**：优先stdio。AI Agent本来就是本地进程，stdio天然安全、不需端口管理。

## 四、类型声明 (Type Safety)

MCP协议强依赖函数签名。Python用类型注解，LLM通过签名理解工具用法。

```python
def search_images(query: str, count: int = 5, safe: bool = True) -> str:
    """搜索图片

    Args:
        query: 搜索关键词
        count: 返回数量，默认5
        safe: 安全过滤，默认开启
    """
    ...
```

**铁律**：
- ✅ 参数必须写类型注解 (`str`, `int`, `bool`, `list[str]`)
- ✅ 必须写docstring描述功能和每个参数作用
- ❌ 不要用`**kwargs`
- ❌ 不要在docstring里写"请使用英文"这类prompt注入

## 五、安全加固清单

```python
from pathlib import Path
import re

# 0. 绑定localhost（不开放端口）
mcp = FastMCP("safe-server", host="127.0.0.1")

# 1. 路径穿越防护
SAFE_ROOT = Path("/data/safe_dir")
SAFE_ROOT.mkdir(parents=True, exist_ok=True)

def read_file(path: str) -> str:
    abs_path = (SAFE_ROOT / path).resolve()
    if not str(abs_path).startswith(str(SAFE_ROOT)):
        return "Error: 路径越界"
    return abs_path.read_text()

# 2. 命令注入防护
BLOCKED = re.compile(r'[;&|`$(){}\[\]<>]')
def safe_command(cmd: str) -> str:
    if BLOCKED.search(cmd):
        return "Error: 不安全的命令"
    ...  # 用白名单命令替代

# 3. 速率限制
import time
_calls = []
def rate_limited(func):
    def wrapper(*args, **kwargs):
        now = time.time()
        _calls[:] = [t for t in _calls if now - t < 60]
        if len(_calls) >= 30:  # 每分钟最多30次
            return "Error: 请求过于频繁"
        _calls.append(now)
        return func(*args, **kwargs)
    return wrapper
```

## 六、与Hermes Agent集成

```yaml
# config.yaml
mcp_servers:
  my-tools:
    transport: stdio
    command: python3
    args: ["/path/to/server.py"]
    env:
      API_KEY: "${MY_API_KEY}"
```

## 七、常见陷阱

| 陷阱 | 现象 | 修复 |
|------|------|------|
| docstring缺失 | Agent不会调工具 | 每个参数写清楚作用 |
| 阻塞时间长 | 整个对话卡住 | 设timeout ≤10s |
| 未处理异常 | 返回空/乱码 | try/except全覆盖 |
| 日志没写 | 出错不知道什么 | 用`logging`记每步 |
| 路径没限定 | 安全漏洞 | Path.resolve().startswith |

## 八、调试命令

```bash
# 1. 验证Server能启动
python3 server.py --help

# 2. 手动stdio测试
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | python3 server.py

# 3. 查看工具列表
python3 -c "
from mcp import ClientSession, StdioServerParameters
import asyncio
async def main():
    async with StdioServerParameters(command='python3', args=['server.py']).run() as streams:
        async with ClientSession(*streams) as session:
            await session.initialize()
            tools = await session.list_tools()
            print('已注册工具:', [t.name for t in tools.tools])
asyncio.run(main())
"
```

## 九、生产部署

```bash
# systemd服务
cat > /etc/systemd/system/my-mcp.service << 'EOF'
[Unit]
Description=My MCP Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/mcp/server.py
Restart=always
RestartSec=10
User=nobody
Group=nogroup
MemoryMax=256M
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now my-mcp
```

---

## 许可证

MIT - 自由使用、修改、分发。
