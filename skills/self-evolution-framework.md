---
name: self-evolution-framework
description: 8 阶段自主进化循环 — AI Agent 自我迭代、评测、门控、固化的完整框架
platforms: [hermes, claude-code, openclaw, cursor]
tags: [evolution, self-improvement, automation, iterative-development]
difficulty: advanced
---

# Self Evolution Framework

## 概述

如果 AI 只能做你让它做的事，它就只是个工具。Self Evolution 赋予 AI 自我迭代的能力——Review 自己的表现、发现改进点、修改自身代码、验证效果、固化经验。

## 8 阶段循环

```
Phase 1: Review (回顾)  →  Phase 2: Ideate (构思)
     ↑                              ↓
Phase 8: Loop (循环)    ←  Phase 3: Modify (修改)
     ↑                              ↓
Phase 7: Log (记录)     ←  Phase 4: Commit (提交)
     ↑                              ↓
Phase 6: Gate (门控)    ←  Phase 5: Verify (评测)
```

### Phase 1: Review — 回顾

```python
def review_cycle():
    """回顾最近的表现，找出改进机会"""
    signals = {
        "git_log": git_log("--oneline -20"),
        "trace": read_latest_trace(),
        "errors": grep_logs("ERROR|failed|timeout"),
        "results": read_results_tsv(),
    }
    
    opportunities = []
    for signal_type, data in signals.items():
        issues = analyze_signal(signal_type, data)
        opportunities.extend(issues)
    
    return opportunities
```

### Phase 2: Ideate — 构思

```python
def ideate(opportunities):
    """对每个改进机会，构思修复方案"""
    for opp in opportunities:
        # 反事实诊断
        diagnostic = f"If we had {opp['solution']}, then {opp['problem']} would not have occurred"
        
        # 按 6 级优先级选择
        priority = classify_priority(opp)
        
        # 一轮只改 1 件事
        if priority >= 5:
            return {
                "change": opp['solution'],
                "layers": identify_layers(opp),
                "files": identify_files(opp),
            }
```

### Phase 3: Modify — 修改

```python
MODIFICATION_LAYERS = {
    1: "Trigger words / prompts（提示词）",
    2: "SKILL.md 文档（技能描述）",
    3: "Script / code（脚本代码）",
}

def modify(plan):
    """单点原子修改——一轮只改 1 件事"""
    # 描述不能出现 "和/同时/并且"
    assert "和" not in plan['change']
    assert "同时" not in plan['change']
    
    # 最多改 5 个文件
    assert len(plan['files']) <= 5
    
    # 按层修改
    for file in plan['files']:
        edit_single_file(file)
```

### Phase 4: Commit — 提交

```bash
# 先 commit 再验证
git add -A
git commit -m "evolution: 修改位置 | 依据 | 目的"
```

### Phase 5: Verify — 评测

```python
def verify():
    """三层评测"""
    # L1: 快速门卫
    assert structure_ok()          # 代码结构完整
    assert security_scan_pass()    # 11 条安全检查
    assert gt_sample_test_pass()   # 3 条 GT 抽测
    
    # L2: Dev 全量评测
    dev_score = run_dev_eval()     # 全量测试套件
    assert dev_score >= baseline   # 不低于基线
    
    # L3: Holdout 评测（每 10 轮）
    holdout_score = run_holdout_eval()
    assert holdout_score >= baseline
```

### Phase 6: Gate — 门控

```python
FIVE_GATES = ["safe", "stable", "effective", "performant", "compliant"]

def gating():
    results = {}
    for gate in FIVE_GATES:
        results[gate] = check_gate(gate)
    
    if all(results.values()):
        print("✅ 五维门控全部通过，保留改动")
        return True
    else:
        print("❌ 门控不通过，自动回滚")
        run_git_revert()
        return False
```

### Phase 7: Log — 记录

```python
def log_cycle(results):
    """记录进化数据"""
    import csv
    with open("results.tsv", "a") as f:
        w = csv.writer(f, delimiter="\t")
        w.writerow([
            cycle_number, date, change_description,
            dev_score, holdout_score,
            all_gates_passed, trace_path
        ])
    
    # 同步到经验库
    sync_to_experience_library(results)
```

### Phase 8: Loop — 循环

```python
def should_stop():
    """判断是否终止"""
    # 连续 3 轮同一层无优化 → 升级层次
    # 连续 5 轮丢弃 → 切换激进策略
    # 三层都完成无提升 → 终止
    
    if no_improvement_for(3):
        upgrade_level()
    if discarded_for(5):
        switch_to_aggressive()
    if all_levels_done():
        return True
    return False
```

## 常见陷阱

### ⚠️ 改完不验证
- 提交了代码但没跑测试 → 改坏了也不知道
- 修复：Phase 5 的评测不可跳过

### ⚠️ 一轮改多个地方
- 描述写「同时优化 A 和 B」→ 改坏了不确定是哪一步的问题
- 修复：一轮只能改 1 件事

### ⚠️ 门控形同虚设
- 所有门控都返回 True 但实际没检查
- 修复：每个门控必须有可验证的检查函数

## 验证清单

- [ ] 每一轮只改 1 件事
- [ ] 改前先 git commit（不改就回滚）
- [ ] L1 门卫通过（安全/结构/GT 抽查）
- [ ] L2 Dev 全量评测不低于基线
- [ ] 五维门控全通过
- [ ] 失败时自动 git revert
- [ ] 结果写入经验库
