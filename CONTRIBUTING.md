# Contributing

欢迎贡献！欢迎提交 PR 分享你用得好的 Agent Skill。

## 规则

1. **标准格式** — 每个 skill 用 YAML frontmatter 开头：

```yaml
---
name: skill-name
description: 一句话说明
platforms: ["hermes", "claude-code", "codex", "openclaw", "cursor"]
tags: ["标签1", "标签2"]
---
```

2. **无隐私** — 不要包含 API Key、域名、服务器路径等敏感信息

3. **实用** — 每个 skill 解决一个真实问题，有具体使用场景和示例

4. **排版清晰** — 用中文写，markdown 格式，tab/space 统一

## 提交流程

```
Fork 仓库 → 创建分支 → 添加 skill → 提交 PR
```

PR 描述写清楚这个 skill 解决什么问题、适用什么平台。
