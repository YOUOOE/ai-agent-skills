---
name: async-image-generation-workflow
description: 完整的 AI 异步生图系统 — 从后端 API 到前端进度展示、结果轮询、错误处理的完整链路
platforms: [hermes, cursor, claude-code, windsurf]
tags: [image-generation, async, polling, progress, ai-tools]
difficulty: intermediate
---

# Async Image Generation Workflow

## 概述

AI 生图通常需要 5-30 秒，不能同步等待。本 Skill 提供完整的前后端异步生图架构：提交任务→实时进度→轮询结果→兜底重试→结果展示。

## 架构

```
用户提交
   │
   ▼
POST /api/image/generate ──→ 创建任务(task_id) ──→ 入队列
   │                                                    │
   └── 返回 {task_id, state:"pending"}                   │
   │                                                    │
   ▼                                                    ▼
前端轮询 GET /api/image/query?task_id=xxx          Provider API
   │                                                    │
   ├─ pending ──→ 继续轮询 (每 2s)                       │
   ├─ processing ──→ 显示进度                            │
   └─ completed ──→ 显示结果                             │
        │                                                │
        ▼                                                ▼
  图片保存到本地 + 展示                             响应完成
```

## 后端实现

### 1. 任务提交端点

```python
@app.post("/api/image/generate")
async def image_generate(request: Request):
    data = await request.json()
    prompt = data.get("prompt")
    size = data.get("size", "1024x1024")

    # 创建任务记录
    task_id = str(uuid.uuid4())[:8]
    task = {
        "task_id": task_id,
        "prompt": prompt,
        "size": size,
        "state": "pending",
        "progress": 0,
        "created_at": time.time(),
        "image_url": None,
        "error": None
    }

    # 持久化到数据库
    db.insert_task(task)

    # 异步提交到 Provider
    asyncio.create_task(_process_image(task_id))

    return {"task_id": task_id, "state": "pending"}
```

### 2. 异步处理 + 进度模拟

```python
async def _process_image(task_id):
    """后台处理生图任务"""
    db.update_state(task_id, "processing", progress=0)

    try:
        # 调用 Provider API
        response = await call_provider(prompt, size)

        # 轮询查询进度（Provider 异步返回）
        for i in range(30):
            status = await query_provider_status(task_id)

            if status.get("state") == "completed":
                # 保存图片到本地
                image_url = await save_image_locally(status["image_url"])
                db.update_state(task_id, "completed",
                                progress=100, image_url=image_url)
                return

            elif status.get("state") == "failed":
                raise RuntimeError(status.get("error", "生图失败"))

            # 更新前端进度（模拟）
            progress = min(i * 3 + random.uniform(1, 5), 95)
            db.update_progress(task_id, progress)

            await asyncio.sleep(2)  # 每 2 秒查一次

        raise TimeoutError("Provider 超时")

    except Exception as e:
        # 兜底：尝试备用 Provider
        try:
            image_url = await _fallback_provider(prompt, size)
            db.update_state(task_id, "completed",
                            progress=100, image_url=image_url)
        except:
            db.update_state(task_id, "failed", error=str(e))
```

### 3. Provider 查询容错

```python
async def query_image_status(task_id, provider):
    """查询生图状态，Provider 不可用时不等于任务失败"""
    try:
        resp = await asyncio.wait_for(
            _provider_query(task_id, provider), timeout=5
        )
        # 有图即视为完成，忽略 state 字段
        if resp.get("data", {}).get("images") or resp.get("image_url"):
            return {"state": "completed", "image_url": _extract_url(resp)}
        return {"state": resp.get("state", "processing"),
                "progress": resp.get("progress", 50)}

    except (asyncio.TimeoutError, Exception):
        # Provider 不可用：返回 pending，让前端继续轮询
        # 不标记失败——任务可能还在 Provider 那边跑着
        return {"state": "pending", "progress": None}
```

### 4. 兜底 Reaper

```python
# image_task_reaper.py — 每 2 分钟 cron
async def reap_stuck_tasks():
    """清理卡住的任务"""
    stuck = db.get_stuck_tasks(timeout=120)  # 超时 2 分钟

    for task in stuck:
        try:
            # 最后再查一次 Provider
            status = await query_provider(task["task_id"])
            if status.get("state") == "completed":
                db.update_state(task["task_id"], "completed",
                                image_url=status["image_url"])
            else:
                db.update_state(task["task_id"], "failed",
                                error="Provider 超时")
        except:
            db.update_state(task["task_id"], "failed", error="重试失败")
```

## 前端实现

### 1. 提交 + 轮询

```javascript
async function generateImage(prompt, size) {
  // 1. 提交任务
  const res = await fetch('/api/image/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ prompt, size })
  });
  const { task_id } = await res.json();

  // 2. 显示进度
  showProgressBar(task_id);

  // 3. 轮询结果
  pollResult(task_id);
}

async function pollResult(taskId, retries = 40) {
  for (let i = 0; i < retries; i++) {
    const res = await fetch(`/api/image/query?task_id=${taskId}`);
    const data = await res.json();

    if (data.state === 'completed') {
      hideProgress();
      showResult(data.image_url);
      return;
    }

    if (data.state === 'failed') {
      hideProgress();
      showError(data.error);
      // 显示「检查结果」按钮让用户手动重试
      showRetryButton(taskId);
      return;
    }

    updateProgress(data.progress || estimateProgress(i));
    await sleep(2000);
  }

  // 超时：显示「检查结果」按钮
  hideProgress();
  showCheckButton(taskId);
}
```

### 2. 进度模拟（永不卡死）

```javascript
function estimateProgress(round) {
  // 时间曲线映射：前 30% 快走，中间慢走，最后平稳
  const mapping = [
    { range: [0, 3], min: 2, max: 15 },
    { range: [3, 8], min: 15, max: 35 },
    { range: [8, 20], min: 35, max: 70 },
    { range: [20, 40], min: 70, max: 95 }
  ];
  const p = mapping.find(m => round >= m.range[0] && round < m.range[1]);
  if (!p) return 95;
  const t = (round - p.range[0]) / (p.range[1] - p.range[0]);
  return Math.min(p.min + (p.max - p.min) * t, 95);
}
```

### 3. 结果卡片

```html
<div class="gl-img-card">
  <div class="gl-img-wrap">
    <img src="{image_url}" alt="生成结果"
         onclick="openViewer(this.src)">
  </div>
  <div class="gl-card-tools">
    <button onclick="downloadFile('{image_url}')">
      <i class="fa-solid fa-download"></i> 下载
    </button>
    <button onclick="shareImage('{image_url}')">
      <i class="fa-solid fa-share"></i> 分享
    </button>
    <button onclick="reGenerate()">
      <i class="fa-solid fa-rotate"></i> 重做
    </button>
  </div>
</div>
```

## 常见陷阱

### ⚠️ Provider 查询超时 ≠ 任务失败
- Provider API 返回 502 时任务可能仍在处理
- 修复：返回 `pending` 让前端继续轮询

### ⚠️ 重复提交
- 用户快速点击发送多次
- 修复：前端 2s 防抖 + 后端 5s 内容去重

### ⚠️ 结果图片 404
- Provider CDN 链接有时效
- 修复：生成后立刻保存到本地服务器

### ⚠️ 进度条卡在 95%
- 忘了映射超时场景
- 修复：超时后显示「检查结果」按钮而不是卡 loading

## 验证清单

- [ ] 生图全链路：提交→进度→结果→展示
- [ ] Provider 502 时任务不丢失
- [ ] 超时 60s 后不卡 loading
- [ ] 进度条每秒都在动
- [ ] 结果图片页面刷新后仍在（本地持久化）
- [ ] 错误提示友好，告诉用户不计算费
