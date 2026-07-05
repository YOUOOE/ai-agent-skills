# ai-agent-skills 🎯

**实用的 AI Agent Skills 和提示词库** — 让 AI Agent 更聪明、更有用。

Hermes Agent / Claude Code / Cursor / Windsurf / OpenClaw 通用标准格式。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-16-blue)](#skills)

## Skills

| 技能 | 描述 | 难度 |
|------|------|------|
| [Async Image Generation Workflow](skills/async-image-generation-workflow.md) | AI 异步生图全链路：提交→轮询→进度→兜底→展示 | ⭐⭐ |
| [Browser Automation Workflow](skills/browser-automation-workflow.md) | 浏览器自动化：页面操作、数据提取、视觉验证 | ⭐⭐ |
| [Code Review Agent](skills/code-review-agent.md) | 代码审查：逻辑漏洞、边界条件、回归风险 | ⭐⭐ |
| [Code Simplifier Agent](skills/code-simplifier-agent.md) | 代码简化：去重复、理命名、顺结构、清死代码 | ⭐⭐ |
| [Cron Automation Patterns](skills/cron-automation-patterns.md) | 定时任务自动化：巡检、内容生成、数据管道、空输出保护 | ⭐⭐ |
| [Dark Theme Design System](skills/dark-theme-design-system.md) | 暗色主题设计系统：CSS 变量、双模式切换、SVG 图标 | ⭐ |
| [Debugger Agent](skills/debugger-agent.md) | 系统化调试：复现→根因分析→多路径修复→验证闭环 | ⭐⭐ |
| [Full Dev Workflow](skills/full-dev-workflow.md) | 6 Agent 开发流水线：探索→审查→测试→简化→安全 | ⭐⭐⭐ |
| [Multi-Agent Coordination](skills/multi-agent-coordination.md) | 多 Agent 协作协议：共享文件系统通信、资源锁、冲突检测、工作负载均衡 | ⭐⭐⭐ |
| [Multi-Provider AI Fallback](skills/multi-provider-ai-fallback.md) | 多供应商路由：故障转移、熔断、成本优化 | ⭐⭐⭐ |
| [Security Review Agent](skills/security-review-agent.md) | 安全审查：认证、密钥、支付、数据隔离全覆盖 | ⭐⭐⭐ |
| [Self Evolution Framework](skills/self-evolution-framework.md) | 8 阶段自进化循环：Review→Modify→Verify→Gate→Loop | ⭐⭐⭐ |
| [Streaming Response Handler](skills/streaming-response-handler.md) | 流式响应系统：SSE 协议、断线重连、实时进度展示 | ⭐⭐ |
| [MCP Server Development Guide](skills/mcp-server-development-guide.md) | MCP 协议实现：Server 开发、工具注册、SSE 流式、stdio 传输 | ⭐⭐⭐ |
| [Test Engineer Agent](skills/test-engineer-agent.md) | 关键路径测试：按风险优先级（P0>P1>P2）补测试 | ⭐⭐ |
| [Vibe Coding Workflow](skills/vibe-coding-workflow.md) | AI 驱动编程：自然语言描述→AI 生成→你审核→迭代 | ⭐ |

## Prompts

| 文件 | 描述 |
|------|------|
| [System Prompt Design Guide](prompts/system-prompt-design-guide.md) | 系统提示词设计原则与模式 |
| [System Prompts](prompts/system-prompts.md) | 多场景系统提示词模板 |
| [Prompt Templates](prompts/prompt-templates.md) | 提示词模板合集 |
| [Scenario Prompts](prompts/scenario-prompts.md) | 场景化提示词示例 |

## 使用方式

### Hermes Agent
```bash
skill_list                          # 查看所有可用 skills
skill_view(name='debugger-agent')   # 加载某个技能
```

### Claude Code
```bash
# 将 .md 文件放入 ~/.claude/skills/
# 或通过 MCP 配置加载
```

### Cursor / Windsurf
将 `.md` 文件放入项目根目录的 `.cursor/skills/` 或 `.windsurf/skills/`。

## 贡献

欢迎提交 PR 添加新的 skills！要求：
- 纯原创，不侵权
- 有实际使用价值
- 标准 YAML 格式 frontmatter
- 含常见陷阱和验证清单

## 许可

MIT License — 随意使用、修改、商用，保留版权声明即可。
