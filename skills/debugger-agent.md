---
name: debugger-agent
description: 系统化调试方法论 — 复现→分析根因→多路径推演→最小修复→验证闭环
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [debugging, troubleshooting, bug-fixing, root-cause]
difficulty: intermediate
---

# Debugger Agent

## 概述

系统化调试的三个铁律：①先复现再修 ②找根因不打补丁 ③验证闭环（改完要测）。本 Skill 提供一套完整的调试流程，覆盖前端、后端、API 各类故障场景。

## 三段式调试流程

### Phase 1：复现与定位

```python
def diagnose(problem_description, codebase):
    """定位问题根因"""
    
    # 1. 缩小范围
    # 前端问题：URL → Nginx → 浏览器 → JS → API → 后端
    # 后端问题：日志 → 数据 → 代码 → 依赖
    # API问题：curl → 状态码 → 响应体 → 请求头
    
    # 2. 收集证据
    evidence = {
        "logs": grep_logs("ERROR|Traceback|500"),
        "recent_changes": git_log("--oneline -10"),
        "service_status": check_services(),
        "api_health": curl_health_endpoints(),
    }
    
    # 3. 二分排除
    # 前端先看控制台有没有报错（JS语法错误会把整个页面杀死）
    # 再查网络请求（CDN被墙、API 404/500）
    # 最后查逻辑（数据流断在哪）    
```

### Phase 2：多路径修复方案

```python
def generate_fixes(root_cause):
    """
    每个根因给出 3 种修复路径：
    A. 最小修复 — 改最少的代码解决问题（保守）
    B. 根治修复 — 重构消除根因（彻底）
    C. 防护修复 — 加守卫层防止复发（防御）
    """
    return {
        "approach_a": "最小修复",
        "approach_b": "根治修复",
        "approach_c": "防护修复",
    }
```

### Phase 3：验证闭环

```python
VALIDATION_TASKS = [
    "语法检查: node --check / python -m py_compile",
    "API验证: curl -s -o /dev/null -w %{http_code} == 200",
    "功能测试: 核心路径走一遍",
    "回归检查: 关联功能不受影响",
    "错误处理: 异常场景不会崩",
]
```

## 常见故障模式

### 模式 1：前端所有功能同时失效
```
诊断链：
  浏览器控制台报错 → 找第一个红线错误
  → 大概率是 JS 语法错误 (brace 不平衡/引号未闭合)
  → 整个 <script> 块被浏览器丢弃
  → 所有依赖这个块的函数全部 undefined
修复：花括号平衡检查 + node --check
```

### 模式 2：API 返回 500 但日志正常
```
诊断链：
  curl 本地直调 → 确认后端状态
  → 本地 200，线上 500 → Nginx 问题
  → 检查 proxy_pass 路径是否有尾部斜杠（/api/ → /xxx vs /api → /api/xxx）
  → 检查超时设置（proxy_read_timeout）
```

### 模式 3：功能一阵好一阵坏
```
诊断链：
  检查是否有多个进程监听同一端口
  → 一个好进程 + 一个坏进程，请求随机路由
  → 检查是否有旧进程残留 (ps aux | grep python)
  → systemctl restart 后旧进程没清干净
修复: 重启前先 kill 旧进程
```

### 模式 4：用户说"不行"但你自己测是好的
```
诊断链：
  问用户"打开的是哪个 URL？"
  → Nginx 可能有多个 root 配置
  → 用户看到的跟开发改的可能不是同一份文件
  → X5 浏览器缓存问题（302/query string/no-cache 都无效）
修复: 换全新 URL 路径名
```

### 模式 5：改之前好的，改完就坏了
```
诊断链：
  git diff 看改了哪些文件
  → 是不是改了不该改的地方
  → 回滚到改之前验证
  → 如果是 write_file 全量覆盖，检查是不是用 offset/limit 读了部分内容
修复: 备份恢复后重新精准修改
```

## 调试检查清单

| 问题类型 | 第一步 | 第二步 | 第三步 |
|---------|--------|--------|--------|
| 页面空白 | 浏览器控制台 | HTML 结构完整？ | JS 语法通过？ |
| API 500 | curl 本地 | 检查 Nginx 日志 | 检查后端日志 |
| 数据不对 | 查数据库 | 查 API 返回 | 查前端渲染 |
| 功能消失 | 最近改了啥？ | DOM 还在？ | 事件绑定还在？ |
| 性能慢 | 哪一步最慢？ | 加耗时日志 | 优化瓶颈 |

## 常见陷阱

### ⚠️ 跳过复现直接猜
- 用户说「用不了」→ 我去看代码 → 发现不了问题
- 修复：先问「具体什么操作？URL 是什么？」再动手

### ⚠️ 只修表面不挖根因
- 加了个 try-catch 吞异常 → 问题消失了但根因还在
- 修复：找到异常抛出的真实原因再处理

### ⚠️ 改完不验证
- 改 JS 不跑 node --check → 语法错误上线 → 页面白屏
- 修复：改任何代码后必须语法验证 + 功能验证

## 验证清单

- [ ] 所有功能同时失效 → 先查控制台报错
- [ ] 单个功能失效 → 先问用户具体操作
- [ ] API 问题 → curl 验证再判断前后端责任
- [ ] 改完必须自验 → 语法 + 功能 + 回归
- [ ] 根因修复 → 加教训记录防止再犯
