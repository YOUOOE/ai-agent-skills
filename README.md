# 🎯 AI Agent Skills 合集

> 即拿即用的 Agent Skills & Prompt 库 — 让你的 AI Agent 更聪明、更专业、更好用。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 适用平台

| 平台 | 安装方式 |
|------|----------|
| **Hermes Agent** | `skill_view(name="技能名")` |
| **Claude Code** | 将 SKILL.md 内容放入 `CLAUDE.md` 或直接粘贴到对话 |
| **Codex CLI** | `codex exec "加载 [技能名] 工作流"` |
| **OpenClaw** | 复制 markdown 内容到 Agent 系统提示词 |
| **Cursor** | 放入 `.cursorrules` 或项目根目录 |
| **通用** | 直接阅读 → 理解方法论 → 应用到你的项目 |

---

## 📦 Skills 列表

| # | Skill | 一句话 | 适用 |
|---|-------|--------|------|
| 01 | [代码审查](skills/code-review.md) | 挑逻辑漏洞、安全风险、回归隐患 | 所有Agent |
| 02 | [系统化调试](skills/debugger.md) | 先定位根因再修Bug，不复现不罢休 | 所有Agent |
| 03 | [安全审查](skills/security-review.md) | 上线前最后一关，扫认证/支付/数据安全 | 所有Agent |
| 04 | [测试工程师](skills/test-engineer.md) | 不追覆盖率，只追"最该测的地方" | 所有Agent |
| 05 | [代码简化](skills/code-simplifier.md) | 删冗余、理命名、顺结构，不改行为 | 所有Agent |
| 06 | [Vibe Coding 工作流](skills/vibe-coding.md) | 描述需求 → AI 生成 → 验证 → 迭代 | 所有Agent |
| 07 | [完整开发工作流](skills/full-dev-workflow.md) | 6 Agent 流水线，从探索到部署一条龙 | 所有Agent |
| 08 | [AI 对话人格设计](skills/ai-personality.md) | 让 AI 有人味、有判断力、有边界感 | 通用 |
| 09 | [自进化框架](skills/self-evolution.md) | 8 阶段循环，让 AI 自己改自己、持续变强 | 高级Agent |

---

## 🎯 Prompts 库

| # | Prompt | 用途 |
|---|--------|------|
| 01 | [系统提示词模板](prompts/system-prompts.md) | 通用/代码/写作/翻译/设计/审查 6 套 system prompt |
| 02 | [Prompt 设计指南](prompts/prompt-design-guide.md) | 怎么写好的 system prompt？结构和原则 |
| 03 | [场景 Prompt 合集](prompts/scenario-prompts.md) | 代码审查/调试/润色/翻译/分析等常用 prompt |

---

## 🚀 快速开始

### Hermes Agent
```bash
# 加载某个 skill
skill_view(name="code-review")
```

### Claude Code
```bash
# 项目中放 CLAUDE.md，或对话中引用
/skill: code-review
```

### 通用
直接复制对应 skill 的 markdown 内容，粘贴到你的 Agent 系统提示词中即可生效。

---

## 📐 Skill 格式

每个 skill 使用标准格式：

```yaml
---
name: skill-name
description: 一句话说明
platforms: ["hermes", "claude-code", "codex", "openclaw"]
tags: ["代码质量", "安全"]
---

# 标题

## 适用场景
在什么情况下使用这个 skill？

## 工作流
分步骤说明。

## 使用示例
...
```

---

## 🤝 贡献

欢迎提交 PR！请确保：
1. 使用标准 YAML frontmatter 格式
2. 内容实用、通用、无隐私信息
3. 附带使用示例

---

## 📄 许可

MIT — 随意使用、修改、分发。注明出处即可。

---

> 💡 AI Agents 的能力，取决于你给了它们什么样的 Skills。
