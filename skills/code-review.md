---
name: code-review
description: 系统化代码审查 — 挑逻辑漏洞、边界条件、回归风险
platforms: ["hermes", "claude-code", "codex", "openclaw", "cursor"]
tags: ["代码质量", "安全", "审查"]
---

# Code Review — 代码审查

## 适用场景
提交 PR 前、上线前、接手别人代码时。确保代码能跑、能上线、不出事。

## 审查清单

### 🔴 P0 — 必修（不上线级别）
- [ ] SQL 注入 / XSS / 命令注入
- [ ] API Key / 密码硬编码
- [ ] 越权访问（A 用户操作 B 用户数据）
- [ ] 金额/支付逻辑错误
- [ ] 敏感信息泄漏到前端/日志

### 🟡 P1 — 建议修
- [ ] 边界条件（空值/0/null/超大值/并发覆盖）
- [ ] 异常分支没有错误处理
- [ ] 数据库 N+1 查询
- [ ] 循环里昂贵的重复计算
- [ ] 幂等性问题（重复提交结果不一致）

### 🟢 P2 — 可选
- [ ] 命名不规范
- [ ] 缺少类型注解
- [ ] 重复代码
- [ ] 函数过长（＞50行）

## 工作流

```
拿到代码 → 先读描述理解意图
   ↓
逐文件审查 → 按清单检查
   ↓
标注问题 → P0/P1/P2 分级
   ↓
给审查意见 → 问题+根因+修改建议
   ↓
验证修复 → 确认后 Approve
```

## 输出格式

```
[P0] 安全问题：API Key 硬编码在配置文件
  代码位置: config.py:15
  根因: 直接从代码读取而非环境变量
  建议: 改用 os.environ.get("API_KEY")

[P1] 逻辑问题：除零未处理
  代码位置: calculator.py:42
  根因: 未校验除数是否为 0
  建议: 加 if divisor == 0 判断
```

## 使用示例

```bash
# Claude Code
/code-review path/to/pr.diff

# Hermes Agent
skill_view(name="code-review")

# 通用
复制以上 checklist 到对话，附上代码即可
```
