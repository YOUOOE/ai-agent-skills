---
name: multi-provider-ai-fallback
description: 构建高可用多供应商 AI 模型路由系统，实现自动降级、故障转移、成本优化
platforms: [hermes, claude-code, openclaw, cursor]
tags: [llm, provider, fallback, resilience, cost-optimization]
difficulty: advanced
---

# Multi-Provider AI Fallback

## 概述

单一大模型提供商（如 DeepSeek/OpenAI）总有不可用的时候。本 Skill 提供一个经过生产验证的多供应商路由系统，实现故障自动转移、成本感知调度、健康检测和熔断保护。

## 架构

```
请求 → Provider Router
        ├── Tier 1: 主供应商（高优、低成本）
        │     └── 健康检查 → 可用 → 响应
        │     └── 失败 → 标记降级
        ├── Tier 2: 备用供应商（中等成本）
        │     └── 尝试 → 可用 → 响应
        │     └── 失败 → 标记降级
        └── Tier 3: 兜底（免费/本地）
              └── 尝试 → 可用 → 响应
              └── 全挂 → 友好错误
```

## 实现

### 1. Provider 注册与路由

```python
PROVIDERS = {
    "deepseek": {
        "base_url": "https://api.deepseek.com/v1",
        "api_key_env": "DEEPSEEK_API_KEY",
        "models": ["deepseek-v4-flash", "deepseek-chat"],
        "priority": 1,
        "cost_per_1k": 0.0005,
        "timeout": 30,
    },
    "siliconflow": {
        "base_url": "https://api.siliconflow.com/v1",
        "api_key_env": "SILICONFLOW_API_KEY",
        "models": ["sf-qwen3-8b", "sf-glm-4-9b"],
        "priority": 2,
        "cost_per_1k": 0.0003,
        "timeout": 30,
    },
    "local": {
        "base_url": "http://127.0.0.1:8080/v1",
        "api_key_env": None,
        "models": ["qwen3-4b"],
        "priority": 3,
        "cost_per_1k": 0,
        "timeout": 60,
    },
}
```

### 2. 健康检测与熔断器

```python
import time
from collections import defaultdict

class CircuitBreaker:
    def __init__(self, failure_threshold=3, recovery_timeout=60):
        self.failure_count = defaultdict(int)
        self.last_failure = {}
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout

    def is_open(self, provider):
        """熔断器是否打开（跳过此 provider）"""
        now = time.time()

        # 在恢复窗口内
        if provider in self.last_failure:
            elapsed = now - self.last_failure[provider]
            if elapsed < self.recovery_timeout:
                if self.failure_count[provider] >= self.failure_threshold:
                    return True  # 熔断中

        # 过了恢复窗口，尝试半开
        if provider in self.last_failure:
            if elapsed >= self.recovery_timeout:
                self.failure_count[provider] = 0  # 重置
        return False

    def record_failure(self, provider):
        self.failure_count[provider] += 1
        self.last_failure[provider] = time.time()

    def record_success(self, provider):
        self.failure_count[provider] = 0
```

### 3. 全链路 Fallback

```python
import asyncio
import logging

logger = logging.getLogger(__name__)

async def call_ai(messages, model, providers, breaker, timeout=30):
    """带全链路 fallback 的 AI 调用"""
    last_error = None

    for provider in sorted(providers, key=lambda p: p["priority"]):
        if breaker.is_open(provider["name"]):
            logger.warning(f"熔断跳过: {provider['name']}")
            continue

        try:
            response = await asyncio.wait_for(
                _call_single_provider(provider, messages, model),
                timeout=timeout
            )
            breaker.record_success(provider["name"])
            return response

        except asyncio.TimeoutError:
            logger.error(f"{provider['name']} 超时 ({timeout}s)")
            breaker.record_failure(provider["name"])
            last_error = "超时"

        except Exception as e:
            logger.error(f"{provider['name']} 失败: {e}")
            breaker.record_failure(provider["name"])
            last_error = str(e)

    # 所有 provider 全挂
    raise RuntimeError(f"所有 AI 提供商不可用: {last_error}")
```

### 4. 实时 Provider 健康监听

```python
async def health_check_loop(providers, breaker, interval=60):
    """每分钟检测所有 Provider 可用性"""
    while True:
        for provider in providers:
            try:
                response = await asyncio.wait_for(
                    _call_single_provider(provider, [
                        {"role": "user", "content": "ping"}
                    ], provider["models"][0]),
                    timeout=10
                )
                breaker.record_success(provider["name"])
                logger.info(f"✅ {provider['name']} 健康")
            except Exception as e:
                breaker.record_failure(provider["name"])
                logger.warning(f"❌ {provider['name']} 异常: {e}")

        await asyncio.sleep(interval)
```

### 5. 前端模型切换提示

```javascript
function switchModel(modelId, provider) {
  // 显示切换提示
  const pill = document.getElementById('model-pill');
  const tip = document.getElementById('model-tip');

  tip.textContent = `已切换至 ${provider}`;
  tip.classList.add('show');

  setTimeout(() => tip.classList.remove('show'), 2000);

  // 持久化选择
  localStorage.setItem('preferred_model', modelId);
  localStorage.setItem('preferred_provider', provider);
}
```

## 常见陷阱

### ⚠️ 单线程阻塞
- Uvicorn 单 Worker 下，同步请求会卡死所有人
- 修复：用 `run_in_executor` 或 `ThreadPoolExecutor`

### ⚠️ 健康检查击穿
- 所有请求同时发现 Provider 挂了，同时触发 fallback
- 修复：加熔断器 + 异步健康检查循环

### ⚠️ 金额不足不报错
- API 返回 200 但内容提示余额不足
- 修复：检查响应中的 `error`/`message` 字段

### ⚠️ 死 Key 阻塞
- 环境变量中的 Key 是占位符（如 `sk-xxx...xxx`）
- 修复：启动时验证 Key 有效性，无效则跳过

## 验证清单

- [ ] 主 Provider 返回 500 时 2s 内自动 fallback
- [ ] 所有 Provider 挂了时有友好错误提示
- [ ] 恢复后自动切回主 Provider
- [ ] 健康检测不泄露敏感信息
- [ ] 3 次连续失败后熔断 60s
- [ ] 熔断恢复后尝试半开请求
