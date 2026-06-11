---
name: full-dev-workflow
description: 6 Agent 开发流水线 — 从探索需求到安全上线，每个环节有专门 Agent 把关
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [workflow, development, code-quality, testing, security]
difficulty: advanced
---

# Full Dev Workflow

## 概述

开发不只是写代码。本 Skill 定义了一条完整的开发流水线：6 个专用 Agent 各司其职，从阅读项目、调试 bug、审查代码、写测试、简化代码到安全检查，步步把关。

## 6 Agent 流水线

```
需求 → ①探索 Agent → ②调试 Agent → ③审查 Agent → ④测试 Agent → ⑤简化 Agent → ⑥安全 Agent → 上线
```

### ① 探索 Agent — 先读项目再动手

```bash
# 阅读项目结构，生成项目地图
explore-agent "理解 /path/to/project 的整体架构"
# 输出：
# - 目录结构
# - 核心模块职责
# - 数据流/调用链
# - 外部依赖清单
# - 常见坑（许可证、过时库等）
```

```python
def project_map(root_dir):
    """生成项目地图"""
    return {
        "languages": detect_languages(root_dir),
        "entry_points": find_entry_points(root_dir),
        "modules": analyze_modules(root_dir),
        "dependencies": find_dependencies(root_dir),
        "test_coverage": estimate_test_coverage(root_dir),
        "code_quality": lint_all_files(root_dir),
    }
```

### ② 调试 Agent — 先定位根因再修

```python
def debug_flow(error_message, codebase):
    """系统化调试：先复现 → 分析 → 修复 → 验证"""
    # Phase 1: 复现
    reproduce_steps = trace_error(error_message)
    
    # Phase 2: 分析根因
    root_causes = analyze_stack_trace(reproduce_steps)
    
    # Phase 3: 生成多路径修复方案
    fixes = []
    for cause in root_causes:
        fix_a = minimal_fix(cause)      # 最小改动
        fix_b = refactor_fix(cause)     # 重构根治
        fix_c = guard_fix(cause)        # 加防护层
        fixes.append({cause: [fix_a, fix_b, fix_c]})
    
    # Phase 4: 验证
    for fix in fixes[0]:  # 先试根因方案
        if apply_and_test(fix):
            commit(fix)
            return f"已修复: {fix.description}"
    
    return "修复失败，需进一步分析"
```

### ③ 审查 Agent — 挑逻辑漏洞和边界

```python
REVIEW_CHECKLIST = [
    "数据流是否完整？有没有 null/undefined 路径没覆盖？",
    "边界条件：空列表、负数、超大值、并发访问？",
    "外部依赖：API 返回格式变了怎么办？连接超时？",
    "回归风险：这个改动会不会影响已有的功能？",
    "硬编码：有没有写死的 Key、路径、端口？",
    "状态管理：用户切换/刷新页面后状态恢复吗？",
]
```

### ④ 测试 Agent — 补关键路径测试

```python
def test_priority(codebase):
    """按风险排测试优先级"""
    risks = []
    
    # P0: 支付/资金相关
    if has_payment(codebase):
        risks.append("P0: 支付全链路测试")
    
    # P0: 用户认证
    if has_auth(codebase):
        risks.append("P0: 登录/注册/权限测试")
    
    # P1: 核心功能
    main_features = find_core_features(codebase)
    risks.extend([f"P1: {f} 主路径测试" for f in main_features])
    
    # P2: 边界异常
    risks.append("P2: 空数据/网络异常/超时测试")
    
    return risks
```

### ⑤ 简化 Agent — 去冗余整结构

```python
SIMPLIFICATION_RULES = [
    ("重复代码 ≥ 3 次", "提取成函数"),
    ("单函数 > 50 行", "拆成小函数"),
    ("嵌套 > 3 层", "提前 return 或提取方法"),
    ("变量名 < 3 字符", "重命名为有意义的名称"),
    ("注释说明 '做了什么'", "改为描述 '为什么这么做'"),
    ("未使用的 import 和变量", "直接删除"),
]
```

### ⑥ 安全 Agent — 上线前最后一关

```python
SECURITY_CHECKS = {
    "认证绕过": "检查所有 API 端点是否有身份验证",
    "密钥泄露": "扫描代码中的 API Key、密码、Token",
    "SQL 注入": "验证所有数据库查询用参数绑定",
    "XSS": "用户输入是否转义后输出到 HTML",
    "路径穿越": "文件操作路径是否加固",
    "支付安全": "金额计算在服务端完成，金额参数防篡改",
    "数据隔离": "用户 A 不能看到用户 B 的数据",
}
```

## 使用方式

```bash
# 改代码前
explore-agent "理解项目结构"

# 改出 bug 了
debugger-agent "复现并定位根因"

# 改完了
code-review-agent "审查本次所有改动"

# 要上线了
test-engineer-agent "补关键路径测试"
code-simplifier-agent "清理冗余代码"
security-review-agent "安全扫描"
```

## 常见陷阱

### ⚠️ 跳过探索直接改
- 不了解项目结构就动手 → 改错文件
- 修复：改前先跑 explore-agent

### ⚠️ 审查流于形式
- 只看格式不看逻辑 → 漏掉严重 bug
- 修复：按照 CHECKLIST 逐项检查

### ⚠️ 安全审查漏了
- 测试全过但没做安全扫描 → 上线就炸
- 修复：部署前安全审查是必须步骤

## 验证清单

- [ ] 改前跑探索 Agent 生成项目地图
- [ ] 改后跑全 6 个 Agent 审查
- [ ] 所有 API 端点有认证保护
- [ ] 代码中无硬编码密钥/路径
- [ ] 支付逻辑在服务端完成
- [ ] 用户数据有隔离保护
