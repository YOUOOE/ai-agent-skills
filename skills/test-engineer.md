---
name: test-engineer
description: 测试工程师 — 不追覆盖率，只追"最该测的地方"
platforms: ["hermes", "claude-code", "codex", "openclaw", "cursor"]
tags: ["测试", "质量"]
---

# Test Engineer — 测试工程师

## 适用场景
给代码补测试时、修完 Bug 想确认不会再犯时。

## 优先级判断

哪类代码最值得写测试？

| 优先级 | 特征 | 例子 |
|--------|------|------|
| P0 | 核心业务逻辑 | 支付计算、用户认证、权限判断 |
| P1 | 高变更频率 | 同一个文件被改了 3 次以上 |
| P2 | 高复杂度 | if 嵌套超过 3 层、多异步调用 |
| P3 | 安全敏感 | 输入校验、金额操作 |
| P4 | 历史 Bug 集中地 | 同一个文件反复修 |

## TDD 流程

```
红灯（写失败测试）
   ↓ 运行 → 预期红
绿灯（写最少代码让它通过）
   ↓ 运行 → 变绿
重构（优化代码、不改行为）
   ↓ 运行 → 保持绿
循环
```

## 测试结构模板

```python
# test_<模块>_<场景>.py

def test_正常情况():
    """正常参数返回正确结果"""
    result = func(x=10, y=20)
    assert result == 200, f"Expected 200, got {result}"

def test_边界值():
    """边界值处理正确"""
    result = func(x=0, y=20)
    assert result == 0

def test_异常输入():
    """非法输入抛异常或友好返回"""
    with pytest.raises(ValueError):
        func(x=-1, y=20)
```

## Bug 修复 + 测试绑定

```
修 Bug 时顺手写回归测试：
1. 先写一个能复现该 Bug 的测试 → 红灯
2. 修复代码 → 绿灯
3. 确认其他相关测试不变绿
```

## 原则

- 不追求 100% 覆盖率，追求"核心路径全覆盖"
- 每条测试测一件事（一条 assert）
- 测试命名说清楚场景（`test_xxx_正常`/`test_xxx_边界`）
- Bug 修完，先补回归测试再合代码
