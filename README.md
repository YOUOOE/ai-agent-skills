<p align="center">
  <img src="https://img.shields.io/badge/AI%20Agent%20Skills-v1.0-01c2c3?style=for-the-badge" alt="AI Agent Skills">
  <img src="https://img.shields.io/github/stars/ugooe123/ai-agent-skills?style=for-the-badge&color=gold" alt="Stars">
  <img src="https://img.shields.io/github/license/ugooe123/ai-agent-skills?style=for-the-badge&color=green" alt="License">
</p>

<h1 align="center">🎯 AI Agent Skills</h1>

<p align="center">
  <b>让你的 AI Agent 变强 — 即拿即用的 Skills & Prompt 库</b><br>
  复制给你的 Hermes / Claude Code / Codex / OpenClaw / Cursor，装上就能用
</p>

<p align="center">
  <a href="#-skills">Skills</a> ·
  <a href="#-prompts">Prompts</a> ·
  <a href="#-快速开始">快速开始</a> ·
  <a href="#-格式">格式</a> ·
  <a href="#-贡献">贡献</a>
</p>

---

## 📦 安装方式

| 平台 | 一句话安装 |
|------|-----------|
| **Hermes Agent** | `skill_view(name="skill名称")` |
| **Claude Code** | 复制 skill 内容到对话即可 |
| **Codex CLI** | `codex exec "加载 code-review 工作流"` |
| **OpenClaw** | 粘贴到 Agent 系统提示词 |
| **Cursor** | 放入 `.cursorrules` 或项目根目录 |
| **通用** | 直接阅读 → 理解方法论 → 应用 |

---

## 📋 Skills 一览

| # | Skill | 作用 | 适合谁 |
|---|-------|------|--------|
| 01 | **🔍 code-review** | 代码审查 checklist — 逻辑漏洞、安全风险、回归隐患 | 所有开发者 |
| 02 | **🐛 debugger** | 系统化调试 — 先定位根因再修 Bug，不复现不罢休 | 所有开发者 |
| 03 | **🛡️ security-review** | 安全审查 — 上线前扫一遍，防泄漏防越权防漏洞 | 上线前必过 |
| 04 | **🧪 test-engineer** | 测试策略 — 不追覆盖率，只追"最该测的地方" | 追求质量的团队 |
| 05 | **🧹 code-simplifier** | 代码简化 — 删冗余、理命名、顺结构，不改行为 | 接手旧代码时 |
| 06 | **🎨 vibe-coding** | AI 协程编程 — 描述需求 → AI 生成 → 验证 → 迭代 | 快速原型开发 |
| 07 | **🏗️ full-dev-workflow** | 完整开发流水线 — 探索→实现→审查→测试→简化→安全 | 新功能开发 |
| 08 | **🧬 self-evolution** | 自适应优化框架 — 8阶段方法论，AI可持续自我改进的设计思路 | 追求自优化的团队 |

---

## 🎯 Prompts 库

| # | 内容 | 用途 |
|---|------|------|
| 01 | [系统提示词模板](prompts/system-prompts.md) | 通用/代码/写作/翻译/设计/审查 6 套即用模板 |
| 02 | [Prompt 设计指南](prompts/prompt-design-guide.md) | 怎么写好的 system prompt？结构+原则+陷阱 |
| 03 | [场景 Prompt 合集](prompts/scenario-prompts.md) | 8 个高频场景：代码审查/调试/简化/润色/翻译等 |
| 04 | [通用 Prompt 模板](prompts/prompt-templates.md) | 覆盖学习/分析/设计/写作/Agent身份定义 |

---

## 🚀 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/ugooe123/ai-agent-skills.git

# 2. 打开 skills 目录，选一个你需要的
cd ai-agent-skills/skills

# 3. 复制内容给你的 Agent
# Hermes: skill_view(name="code-review")
# Claude: 粘贴内容到对话
# Cursor: 放 .cursorrules
```

---

## 📐 Skill 格式规范

每个 skill 使用统一格式，方便任何 Agent 解析：

```yaml
---
name: skill-name
description: 一句话说清有什么用
platforms: ["hermes", "claude-code", "codex", "openclaw", "cursor"]
tags: ["分类标签"]
---

# 标题

## 适用场景
在什么情况下用

## 工作流/清单
具体的操作步骤

## 示例
真实的输出样例
```

---

## 🤝 贡献

欢迎提交 PR！要求：
- 内容实用，解决真实问题
- 纯原创，无外部引用
- 使用统一 YAML frontmatter 格式
- 零隐私信息

---

## 📄 许可

MIT — 随意使用、修改、分发。保留版权声明即可。

---

<p align="center">
  <b>AI Agent 的能力，取决于你给了它什么样的 Skills。</b><br>
  <sub>用得好，请给 ⭐ 让更多人看到</sub>
</p>
