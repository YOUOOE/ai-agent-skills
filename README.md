# ai-agent-skills 🎯

**实用的 AI Agent Skills 和提示词库** — 让 AI Agent 更聪明、更有用。

Hermes Agent / Claude Code / Cursor / Windsurf / OpenClaw 通用标准格式。

## Skills

| 技能 | 描述 | 难度 |
|------|------|------|
| [Streaming Response Handler](skills/streaming-response-handler.md) | 构建流式响应系统：SSE、断线重连、实时进度展示 | ⭐⭐ |
| [Async Image Generation Workflow](skills/async-image-generation-workflow.md) | AI 异步生图全链路：提交、轮询、进度、兜底 | ⭐⭐ |
| [Multi-Provider AI Fallback](skills/multi-provider-ai-fallback.md) | 多供应商路由：故障转移、熔断保护、成本优化 | ⭐⭐⭐ |
| [Full Dev Workflow](skills/full-dev-workflow.md) | 6 Agent 开发工作流：探索→审查→测试→简化→安全 | ⭐⭐⭐ |
| [Self Evolution](skills/self-evolution.md) | 8 阶段自进化循环：AI 自主迭代升级 | ⭐⭐⭐ |
| [Vibe Coding](skills/vibe-coding.md) | AI 驱动编程：自然语言描述→代码→成品 | ⭐⭐ |
| [Code Review Agent](skills/code-review.md) | 代码审查：逻辑漏洞、边界条件、回归风险 | ⭐⭐ |
| [Security Review Agent](skills/security-review.md) | 安全审查：认证、密钥、支付、数据安全 | ⭐⭐ |
| [Debugger Agent](skills/debugger.md) | 调试：定位根因、多路径推演、最小修复 | ⭐⭐ |
| [Test Engineer Agent](skills/test-engineer.md) | 测试：补关键路径、边界、异常测试 | ⭐⭐ |
| [Code Simplifier Agent](skills/code-simplifier.md) | 代码简化：去重复、理结构、收命名 | ⭐⭐ |

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
# 查看所有可用 skills
skill_list

# 加载 skill
skill_view(name='streaming-response-handler')
```

### Claude Code
```bash
# 将技能文件放入 ~/.claude/skills/
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

MIT License
