---
name: code-simplifier-agent
description: 代码简化方法论 — 去冗余、理命名、顺结构、清死代码，不改行为只改质量
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [refactoring, clean-code, code-quality, simplification]
difficulty: intermediate
---

# Code Simplifier Agent

## 概述

## 简化六大法则

### 1. 删除重复代码（DRY）

```python
# ❌ 重复
def get_user_by_email(email):
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT * FROM users WHERE email = ?", (email,))
    return cur.fetchone()

def get_user_by_id(user_id):
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT * FROM users WHERE id = ?", (user_id,))
    return cur.fetchone()

# ✅ 提取公共函数
def get_user_by(field, value):
    allowed_fields = {"email", "id", "username"}
    if field not in allowed_fields:
        raise ValueError(f"Invalid field: {field}")
    conn = get_db()
    cur = conn.cursor()
    cur.execute(f"SELECT * FROM users WHERE {field} = ?", (value,))
    return cur.fetchone()
```

### 2. 拆分过长的函数

```python
# ❌ 上帝函数（> 50 行）
def process_order(order_data):
    # 验证
    # 扣款
    # 更新库存
    # 发通知
    # 写日志
    pass

# ✅ 单一职责
def validate_order(order_data): pass
def process_payment(order_data): pass
def update_inventory(order_data): pass
def send_notification(order_data): pass
def log_order(order_data): pass
```

### 3. 降低嵌套深度

```python
# ❌ 三层嵌套
def process(data):
    if data:
        if data.get("valid"):
            if data.get("complete"):
                return do_something(data)
            return "incomplete"
        return "invalid"
    return "empty"

# ✅ 提前返回
def process(data):
    if not data:
        return "empty"
    if not data.get("valid"):
        return "invalid"
    if not data.get("complete"):
        return "incomplete"
    return do_something(data)
```

### 4. 统一命名风格

```python
# ❌ 混乱命名
def get_user_data(uid):
    user_name = findUser(uid)
    return {"Name": user_name, "age": get_age_by_id(uid)}

# ✅ 统一 snake_case
def get_user_profile(user_id):
    username = find_user(user_id)
    age = get_user_age(user_id)
    return {"name": username, "age": age}
```

### 5. 移除死代码

```python
# 查找从未被调用的函数、未使用的 import、被注释掉的代码
import os, sys, json, re  # os, re 未使用

def old_function(): pass  # 从未被调用

# 删除这些无用的代码，Git 历史里能找回
```

### 6. 用守卫语句替代 if-else

```python
# ❌ 深层 if-else
if user:
    if user.is_active:
        if user.has_permission("admin"):
            # 业务逻辑
            pass
        else:
            return "no permission"
    else:
        return "inactive"
else:
    return "not found"

# ✅ 守卫语句
if not user:
    return "not found"
if not user.is_active:
    return "inactive"
if not user.has_permission("admin"):
    return "no permission"
# 业务逻辑
```

## 自动简化脚本

```python
#!/usr/bin/env python3
"""simplify.py — 自动代码简化"""

import ast, re

def find_long_functions(code):
    """找出超过 50 行的函数"""
    tree = ast.parse(code)
    long_funcs = []
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            line_count = node.end_lineno - node.lineno
            if line_count > 50:
                long_funcs.append((node.name, line_count))
    return long_funcs

def find_dead_imports(code):
    """找出未使用的 import"""
    tree = ast.parse(code)
    used = set()
    imports = {}
    
    for node in ast.walk(tree):
        if isinstance(node, ast.Name):
            used.add(node.id)
        elif isinstance(node, ast.Import):
            for alias in node.names:
                imports[alias.asname or alias.name] = node.lineno
        elif isinstance(node, ast.ImportFrom):
            for alias in node.names:
                imports[alias.asname or alias.name] = node.lineno
    
    unused = {name: line for name, line in imports.items()
              if name not in used}
    return unused
```

## 简化检查清单

```markdown
- [ ] 同一段逻辑出现 ≥ 3 次 → 提取函数
- [ ] 函数超过 50 行 → 拆分为小函数
- [ ] 嵌套超过 3 层 → 提前 return 或提取方法
- [ ] 变量名不清晰 → 重命名（不用 a/b/tmp/data）
- [ ] 注释说"做了什么" → 改为"为什么这么做"
- [ ] 有未使用的 import/变量 → 删除
- [ ] 有被注释的代码块 → 删除（Git 里有历史）
```

## 常见陷阱

### ⚠️ 简化过度
- 把 3 行逻辑硬拆成 5 个小函数 → 更难读
- 修复：函数有明确的职责边界再拆

### ⚠️ 重构时改了行为
- 提取公共函数时改了一个参数默认值 → 行为变了
- 修复：提取后用同样的测试用例验证

### ⚠️ 删了以为没用的代码
- 删掉的函数被其他模块通过 __import__ 加载
- 修复：先 grep 全局搜索确认再删

## 验证清单

- [ ] 简化后代码行数减少 ≥ 20%
- [ ] 简化后所有测试通过
- [ ] 没有引入新的 import
- [ ] 函数名/变量名有意义
- [ ] 没有未使用的变量和 import
