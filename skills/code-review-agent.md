---
name: code-review-agent
description: 系统性代码审查 — 挑逻辑漏洞、边界条件、回归风险，确保代码能上线
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [code-review, code-quality, best-practices, pull-request]
difficulty: intermediate
---

# Code Review Agent

## 概述

代码审查不是走过场。本 Skill 提供一套完整的审查清单和自动化流程，确保每个改动在合并前经过逻辑、安全、性能、可维护性层层把关。

## 审查维度

### 1. 逻辑正确性

```python
REVIEW_LOGIC = [
    "条件分支全覆盖了吗？（if-else 有遗漏路径吗？）",
    "循环边界正确吗？(off-by-one、空列表)",
    "异步操作有错误处理吗？（try-catch / Promise.catch）",
    "并发竞态：多个请求同时修改同一数据会冲突吗？",
    "状态一致性：操作失败后回滚了吗？",
]
```

### 2. 边界条件

```python
BOUNDARY_CHECKS = [
    ("空值", "None/null/undefined 会崩吗？"),
    ("空集合", "[]/{} 作为输入会怎样？"),
    ("最大值", "超大数字/超长字符串会溢出吗？"),
    ("并发", "同时 N 个请求会死锁吗？"),
    ("超时", "外部 API 没响应会卡死吗？"),
    ("权限", "普通用户能调用管理员接口吗？"),
]
```

### 3. 回归风险

```python
REGRESSION_ANALYSIS = [
    "改了 A 模块 → 调 A 的所有页面是否受影响？",
    "改了数据库表 → 查询语句要不要同步更新？",
    "改了 API 返回格式 → 前端能兼容旧/新两种格式吗？",
    "改了公共函数 → 其他调用方是否仍正确？",
    "改了 CSS 变量 → 所有用了这个变量的组件都检查了吗？",
]
```

### 4. 安全审查

```python
SECURITY_QUICK_CHECK = [
    "API 端点有身份验证吗？",
    "用户输入直接拼到 SQL/Shell/HTML 了吗？",
    "密码/Token 硬编码在代码里了吗？",
    "金额/积分计算在服务端还是前端？",
    "文件路径有遍历风险吗？（../ 过滤了吗？）",
]
```

## 自动化审查脚本

```python
#!/usr/bin/env python3
"""auto_review.py — PR 自动审查"""

import subprocess, json, re

def get_changed_files(branch="HEAD~1"):
    """获取本次改动的文件列表"""
    result = subprocess.run(
        ["git", "diff", "--name-only", branch],
        capture_output=True, text=True
    )
    return result.stdout.strip().split("\n")

def review_file(filepath, content):
    issues = []
    
    # 检查硬编码密钥
    if re.search(r'sk-[a-zA-Z0-9]{20,}', content):
        issues.append("🔑 API Key 硬编码！")
    
    if re.search(r'password\s*=\s*["\']', content, re.I):
        issues.append("🔑 密码硬编码！")
    
    # 检查 console.log
    if '.py' in filepath and 'print(' in content:
        issues.append("📢 Python 代码中有 print()，需要改用 logger")
    
    # 检查 TODO
    if 'TODO' in content or 'FIXME' in content:
        issues.append("📝 有待办事项未处理")
    
    return issues

def main():
    files = get_changed_files()
    all_issues = []
    
    for f in files:
        if not f.endswith(('.py', '.js', '.ts', '.html', '.sh')):
            continue
        
        result = subprocess.run(["git", "diff", "HEAD~1", "--", f],
                               capture_output=True, text=True)
        issues = review_file(f, result.stdout)
        if issues:
            all_issues.append({f: issues})
    
    if all_issues:
        print(json.dumps(all_issues, indent=2, ensure_ascii=False))
    else:
        print("✅ 未发现问题")

if __name__ == "__main__":
    main()
```

## 审查清单模板

```markdown
## 代码审查清单

- [ ] 逻辑正确：所有条件路径都有覆盖
- [ ] 边界条件：空值、极限值、异常输入有处理
- [ ] 回归风险：改动不会破坏现有功能
- [ ] 安全：无密钥硬编码、有身份验证
- [ ] 性能：无 N+1 查询、无死循环风险
- [ ] 可维护性：代码有适当注释和类型注解
- [ ] 错误处理：所有异常场景都有友好提示
- [ ] 日志：关键操作有日志记录
```

## 常见陷阱

### ⚠️ 只检查格式不管逻辑
- 代码风格没问题但业务逻辑是错的
- 修复：先看「逻辑正确性」，再看代码风格

### ⚠️ 知道有问题但不说
- "这个不是我写的" → 放过问题
- 修复：审查是对代码负责，不管谁写的

### ⚠️ 改 A 漏 B
- 改 API 返回格式后，前端展示层忘了同步改
- 修复：CRUD 同步检查（数据层→API→前端展示→前端操作）

## 验证清单

- [ ] 逻辑正确性检查通过
- [ ] 边界条件全部覆盖
- [ ] 无回归风险
- [ ] 无硬编码密钥/密码
- [ ] 所有 API 有认证
- [ ] TODO/FIXME 已处理
