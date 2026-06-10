---
name: code-simplifier
description: 代码简化 — 删冗余、理命名、顺结构，不改行为
platforms: ["hermes", "claude-code", "codex", "openclaw", "cursor"]
tags: ["代码质量", "重构"]
---

# Code Simplifier — 代码简化

## 适用场景
接手旧代码想理清楚、重构前先简化、代码review发现冗余。

## 简化信号

| 信号 | 做法 |
|------|------|
| 同一模式出现 ≥3 次 | → 抽取公共函数 |
| 函数 >50 行 | → 按职责拆分 |
| if 嵌套 >3 层 | → 卫语句提前返回 |
| 变量名 a/b/tmp/data | → 改成有含义的名字 |
| 注释掉的代码块 | → 删了，Git历史里有 |
| 从未调用的函数 | → 删了 |
| 复杂条件 | → 抽取为命名变量 |

## 示例

```python
# 简化前
def process(data):
    if data is not None:
        if 'status' in data:
            if data['status'] == 'active':
                result = do_something(data)
                return result
            else:
                return None
        else:
            return None
    else:
        return None

# 简化后
def process(data):
    if not data or data.get('status') != 'active':
        return None
    return do_something(data)
```

## 原则

1. **不改变行为** — 输入相同，输出一定相同
2. **一次只做一件事** — 改一个点验证一个点
3. **每个函数只做一件事** — 单一职责
4. **改完用 git diff 确认** — 只改了该改的地方
