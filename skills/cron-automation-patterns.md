---
name: cron-automation-patterns
description: AI Agent 的定时任务自动化体系 — 自动巡检、内容生成、数据收集、异常告警
platforms: [hermes, claude-code, cursor, openclaw]
tags: [cron, automation, scheduling, monitoring, ops]
difficulty: intermediate
---

# Cron Automation Patterns

## 概述

AI Agent 真正的价值在于「无人值守也能干活」。本 Skill 覆盖定时任务的完整模式：自动巡检、内容生产、营收监控、异常告警。

## 核心模式

### 模式 1：无 Agent 巡检（零 token 消耗）

```bash
# 脚本 cron：只在有异常时发消息
# 配置: no_agent=True, script=path/to/script.sh
# 空输出 = 不发消息，非零退出 = 自动告警
```

```python
#!/usr/bin/env python3
"""system_health.py — 系统健康巡检"""
import subprocess, json

def check_service(port, name):
    result = subprocess.run(
        ["curl", "-s", "-o", "/dev/null", "-w", "%{http_code}",
         f"http://127.0.0.1:{port}/health"],
        capture_output=True, text=True, timeout=10
    )
    return result.stdout.strip() == "200"

services = [
    (8000, "API服务"),
    (8080, "本地推理"),
    (9400, "框架大脑"),
]

issues = []
for port, name in services:
    if not check_service(port, name):
        issues.append(f"❌ {name} (:{port}) 不可达")

if issues:
    print("⚠️ 服务异常巡检报告")
    for i in issues:
        print(i)
# 无异常时不输出 → cron 不会发消息
```

### 模式 2：Agent 内容生产

```yaml
# 定时调用 AI Agent 生成内容
# 配置: prompt=自包含任务描述, skills=[需要的技能]
schedule: "0 8,20 * * *"
prompt: |
  从我们的实战经验中提炼一个 AI Agent 技能，
  写成标准 YAML 格式 skill 文件，
  保存到 /repo/skills/ 目录下。
  要求纯原创、有实际价值、含常见陷阱。
```

### 模式 3：数据收集 → 分析管道

```yaml
# 链式 cron：job A 收集数据，job B 分析数据
job_a:
  schedule: "0 * * * *"
  script: collect_metrics.py
  no_agent: true  # 纯脚本，零 token

job_b:
  schedule: "5 * * * *"
  context_from: [job_a]  # 自动注入 job A 的输出
  prompt: |
    分析上小时的监控数据，
    发现异常直接报告，
    正常则静默。
```

### 模式 4：营收/转化监控

```python
"""revenue_pulse.py — 每日营收简报"""
import json
from datetime import datetime, timedelta

def get_daily_revenue():
    """从数据库获取今日营收数据"""
    # orders created today
    today = datetime.now().strftime("%Y-%m-%d")
    
    orders = db.query("""
        SELECT COUNT(*), COALESCE(SUM(amount), 0)
        FROM orders
        WHERE DATE(created_at) = %s
        AND status = 'paid'
    """, (today,))
    
    new_users = db.query("""
        SELECT COUNT(*) FROM users
        WHERE DATE(created_at) = %s
    """, (today,))

    return {
        "date": today,
        "orders": orders[0][0],
        "revenue": float(orders[0][1]),
        "new_users": new_users[0][0],
    }
```

### 模式 5：空输出保护

```python
"""cron_spam_guard.py — 防 Cron 空消息轰炸"""

# 原理：连续 3 次空输出自动暂停任务
# 实现：记录每次运行输出长度到 SQLite
# 3 次 < 10 字符 → 自动 cronjob pause

class SpamGuard:
    def __init__(self, db_path="/tmp/cron_guard.db"):
        self.conn = sqlite3.connect(db_path)
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS guard (
                job_id TEXT PRIMARY KEY,
                empty_runs INTEGER DEFAULT 0,
                paused INTEGER DEFAULT 0
            )
        """)
    
    def check(self, job_id, output_length):
        cur = self.conn.execute(
            "SELECT empty_runs FROM guard WHERE job_id=?",
            (job_id,)
        )
        row = cur.fetchone()
        empty_runs = (row[0] if row else 0) + 1 if output_length < 10 else 0
        
        if empty_runs >= 3:
            print(f"🛑 {job_id}: 连续3次空输出，自动暂停")
            # 暂停逻辑
            self.conn.execute(
                "UPDATE guard SET paused=1 WHERE job_id=?",
                (job_id,)
            )
        
        self.conn.execute("""
            INSERT INTO guard (job_id, empty_runs)
            VALUES (?, ?)
            ON CONFLICT(job_id) DO UPDATE SET empty_runs=?
        """, (job_id, empty_runs, empty_runs))
        self.conn.commit()
```

## 最佳实践

| 模式 | 适用场景 | Token 消耗 |
|------|----------|-----------|
| no_agent 脚本 | 健康检查、数据采集、阈值告警 | 0 |
| Agent 内容生产 | 文章生成、技能编写、内容汇总 | 低-中 |
| 链式 cron | 收集→分析→报告 | 中 |
| 营收监控 | 每日消费报告 | 低 |

## 常见陷阱

### ⚠️ 空输出轰炸
- cron 持续发空消息或无用通知
- 修复：空输出保护 + no_agent 模式

### ⚠️ 宕机不自知
- Agent 进程挂了 → cron 也跑不了 → 没人知道
- 修复：心跳检测 + 独立于 Agent 的 systemd 重启

### ⚠️ 重复调度
- 多个 cron 定时写同一个文件 → 互相覆盖
- 修复：加锁或串行化写操作

## 验证清单

- [ ] no_agent 脚本空输出时不发消息
- [ ] Agent cron 自包含不需要上下文
- [ ] 链式 cron 数据传递正确
- [ ] cron 挂了有独立心跳检测
- [ ] 日志不无限膨胀（有轮转）
