---
name: streaming-response-handler
description: 构建高可靠的 AI 流式响应处理系统，支持多格式 SSE、断线重连、实时进度展示
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [streaming, sse, real-time, frontend, ux]
difficulty: intermediate
---

# Streaming Response Handler

## 概述

AI 应用的流式响应是用户体验的核心。本 Skill 提供一套完整的前后端方案，实现首字秒出、断线重连、思考过程可视化、实时进度渲染。

## 前置条件

- 后端支持 SSE (Server-Sent Events)
- 前端支持 EventSource 或 fetch ReadableStream

## 实现方案

### 1. 后端 SSE 标准格式

```
event: think
data: {"content": "分析中..."}

event: text
data: {"content": "最终回复"}

event: done
data: {}

event: error
data: {"message": "错误信息"}
```

**关键事件类型：**

| 事件 | 用途 | 时机 |
|------|------|------|
| `think` | 思考过程 | 推理/搜索/分析阶段 |
| `text` | 最终回复 | 逐字输出 |
| `done` | 完成信号 | 全部输出完毕 |
| `error` | 错误通知 | 异常时 |
| `progress` | 进度百分比 | 生图/耗时任务 |

### 2. 首字延迟优化

```javascript
// 关键：不等全量，逐字吐出
async function createStream(url, body, handlers) {
  const response = await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body)
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';

  const startTime = Date.now();
  let firstChar = true;

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split('\n');

    for (const line of lines.slice(0, -1)) {
      if (line.startsWith('data: ')) {
        try {
          const data = JSON.parse(line.slice(6));
          if (firstChar && data.content) {
            firstChar = false;
            console.log(`首字耗时: ${Date.now() - startTime}ms`);
          }
          handlers.onText(data.content || '');
        } catch {}
      } else if (line.startsWith('event: ')) {
        // 处理事件类型
      }
    }
    buffer = lines[lines.length - 1];
  }
}
```

### 3. 思考过程可视化

```html
<div class="think-block">
  <div class="think-header" onclick="this.nextElementSibling.classList.toggle('show')">
    <span class="think-icon">🧠</span>
    <span class="think-label">思考过程</span>
    <span class="think-arrow">▼</span>
  </div>
  <div class="think-content">
    <!-- 思考内容逐行插入 -->
  </div>
</div>
```

```css
.think-block {
  background: var(--bg-card);
  border-radius: 8px;
  margin: 12px 0;
  overflow: hidden;
}
.think-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  cursor: pointer;
  user-select: none;
  opacity: 0.7;
}
.think-content {
  display: none;
  padding: 0 12px 12px;
  font-size: 13px;
  line-height: 1.6;
  color: var(--text-muted);
}
.think-content.show {
  display: block;
}
```

### 4. 生图进度实时展示

```javascript
// 后端每1-2秒推送一次，百分比+描述
function renderProgress(data) {
  const bar = document.getElementById('progress-bar');
  const label = document.getElementById('progress-label');

  // 四阶段映射
  const phases = [
    { range: [0, 5], text: '分析你的描述...' },
    { range: [5, 30], text: '构思画面布局...' },
    { range: [30, 60], text: '正在绘制细节...' },
    { range: [60, 95], text: '优化最终效果...' }
  ];

  const phase = phases.find(p => data.progress >= p.range[0] && data.progress < p.range[1]);
  label.textContent = phase ? phase.text : '马上就好！';
  bar.style.width = data.progress + '%';

  // 不同阶段动画速度不同
  const speed = data.progress < 30 ? 'fast' : 'normal';
  bar.style.animationDuration = speed === 'fast' ? '0.8s' : '1.5s';
}
```

### 5. 断线重连机制

```javascript
class ReconnectingSSE {
  constructor(url, options = {}) {
    this.url = url;
    this.maxRetries = options.maxRetries || 5;
    this.baseDelay = options.baseDelay || 1000;
    this.retryCount = 0;
    this.connect();
  }

  connect() {
    this.es = new EventSource(this.url);

    this.es.addEventListener('text', (e) => {
      this.retryCount = 0; // 收到数据后重置重试计数
      this.options.onText?.(JSON.parse(e.data).content);
    });

    this.es.onerror = () => {
      this.es.close();
      if (this.retryCount < this.maxRetries) {
        const delay = this.baseDelay * Math.pow(2, this.retryCount);
        this.retryCount++;
        console.log(`断线重连 (${this.retryCount}/${this.maxRetries})，${delay}ms后重试`);
        setTimeout(() => this.connect(), delay);
      } else {
        this.options.onMaxRetries?.();
      }
    };
  }
}
```

## 常见陷阱

### ⚠️ 首字延迟长
- 后端首 token 生成慢 → 换模型或加流式输出
- 中间件缓存/缓冲 → 检查 Nginx 的 `proxy_buffering off`

### ⚠️ 进度条卡住不动
- 后端超时设置了 fixed delay 而非 time-based mapping
- 解决方案：用 `min(progress + random(0.5, 2), 95)` 确保一直动

### ⚠️ 断线后丢数据
- EventSource 不缓存历史 → 后端维护消息队列
- 重连时发送 `lastEventId` 从断点续传

### ⚠️ 手机端首字慢
- 移动网络 TCP 建立慢 → 加 WebSocket 保活
- X5 内核不支持 EventSource → 走 fetch ReadableStream

## 验证清单

- [ ] 首字耗时 < 500ms
- [ ] 进度条始终在动（不卡死超过 3s）
- [ ] 断线 3 次重连后自动恢复
- [ ] 手机端正常渲染气泡
- [ ] 思考块可折叠/展开
- [ ] 完成后不卡 loading
- [ ] 错误时友好提示
