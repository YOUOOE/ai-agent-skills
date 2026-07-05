# Web Search Integration — 多引擎智能搜索整合

**AI Agent 的多源并行搜索、结果融合与缓存系统。**

## 适用场景

| 场景 | 说明 |
|------|------|
| 需要最新信息 | Agent 需要查询实时数据、新闻、价格 |
| 技术问题排查 | 搜索错误信息、API 文档、最佳实践 |
| 多源验证 | 同一问题需要多个信息源交叉验证 |
| 中国市场 | 服务器在中国，需绕过 GFW 获取搜索结果 |

## 架构总览

```
用户提问
  │
  ▼
Intent Detection
  ├─ 简单事实 → 缓存查询 → 命中? → 直接返回
  └─ 深度研究 → 并行搜索引擎
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    AnySearch    Bing Web    360 Search
    (主引擎)    (备选)       (国内补充)
        │           │           │
        └───────────┼───────────┘
                    ▼
            Result Fusion
            去重 + 排序 + 来源打分
                    │
                    ▼
            LLM 综合回答
            注入来源标签
```

## 引擎配置

### 1. 主引擎：AnySearch

AnySearch 是 AI 优化的搜索引擎，返回结构化 JSON 结果。

```python
import requests

ANYSEARCH_KEY = "as_sk_xxx"  # 从 anysearch.com/console/api-keys 获取
ANYSEARCH_URL = "https://api.anysearch.com/v1/search"

def search_anysearch(query: str, max_results: int = 5) -> list[dict]:
    """AnySearch 搜索"""
    resp = requests.post(
        ANYSEARCH_URL,
        headers={"Authorization": f"Bearer {ANYSEARCH_KEY}"},
        json={"query": query, "count": max_results}
    )
    if resp.status_code != 200:
        return []
    data = resp.json()
    # 注意：results 在 data.results 中，不是顶层
    results = data.get("data", {}).get("results", [])
    return [
        {
            "title": r.get("title", ""),
            "url": r.get("url", ""),
            "snippet": r.get("snippet", ""),
            "source": "anysearch"
        }
        for r in results[:max_results]
    ]
```

> **⚠️ 常见陷阱**: AnySearch 的响应格式是 `{code, message, data: {results: [...]}}`，**不是**顶层 `results` 字段。初次集成易踩。

### 2. 国内备选：Bing 中文站

中国服务器无法直连 Google/DuckDuckGo，Bing 中文站是稳定的 HTML 搜索源。

```python
import requests
from bs4 import BeautifulSoup

def search_bing_cn(query: str, max_results: int = 5) -> list[dict]:
    """Bing 中文搜索（HTML 解析）"""
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    }
    resp = requests.get(
        f"https://cn.bing.com/search?q={requests.utils.quote(query)}",
        headers=headers,
        timeout=10
    )
    if resp.status_code != 200:
        return []

    soup = BeautifulSoup(resp.text, "html.parser")
    results = []
    for li in soup.select(".b_algo"):
        title_el = li.select_one("h2 a")
        snippet_el = li.select_one(".b_caption p")
        if title_el:
            results.append({
                "title": title_el.get_text(strip=True),
                "url": title_el.get("href", ""),
                "snippet": snippet_el.get_text(strip=True) if snippet_el else "",
                "source": "bing"
            })
        if len(results) >= max_results:
            break
    return results
```

> **⚠️ 注意**: Bing 的 HTML 结构可能会变，建议加 `try/except` 兜底。

### 3. 国内补充：360 搜索

```python
def search_so(query: str, max_results: int = 5) -> list[dict]:
    """360 搜索（HTML 解析）"""
    headers = {"User-Agent": "Mozilla/5.0"}
    resp = requests.get(
        f"https://www.so.com/s?q={requests.utils.quote(query)}",
        headers=headers,
        timeout=10
    )
    if resp.status_code != 200:
        return []

    soup = BeautifulSoup(resp.text, "html.parser")
    results = []
    for item in soup.select(".res-list"):
        title_el = item.select_one(".res-title a")
        snippet_el = item.select_one(".res-desc")
        if title_el:
            results.append({
                "title": title_el.get_text(strip=True),
                "url": title_el.get("href", ""),
                "snippet": snippet_el.get_text(strip=True) if snippet_el else "",
                "source": "360"
            })
        if len(results) >= max_results:
            break
    return results
```

## 并行搜索引擎

同时调用多个搜索引擎，取最先返回的结果。

```python
import concurrent.futures

def parallel_search(query: str, engines: list = None, max_per_engine: int = 5) -> list[dict]:
    """多引擎并行搜索"""
    if engines is None:
        engines = [search_anysearch, search_bing_cn, search_so]

    with concurrent.futures.ThreadPoolExecutor(max_workers=len(engines)) as executor:
        futures = {executor.submit(engine, query, max_per_engine): engine.__name__ for engine in engines}
        all_results = []
        for future in concurrent.futures.as_completed(futures, timeout=15):
            try:
                results = future.result()
                all_results.extend(results)
            except Exception as e:
                # 单个引擎失败不中断整体
                continue
    return all_results
```

## 结果融合

去重 + 来源权重排序 + 智能截断。

```python
from urllib.parse import urlparse

def fuse_results(results: list[dict], max_total: int = 10) -> list[dict]:
    """多源结果融合去重"""
    seen_urls = set()
    fused = []

    # 来源权重
    source_weight = {
        "anysearch": 3,  # AI 优化，质量最高
        "bing": 2,       # 通用搜索
        "360": 1,        # 补充
    }

    # URL 规范化去重
    for r in sorted(results, key=lambda x: source_weight.get(x.get("source", ""), 0), reverse=True):
        url = r.get("url", "")
        # 规范化 URL 用于去重
        normalized = urlparse(url)._replace(fragment="").geturl()
        if normalized and normalized not in seen_urls:
            seen_urls.add(normalized)
            fused.append(r)
        if len(fused) >= max_total:
            break

    return fused
```

## 缓存层

避免同一问题反复搜索，降低延迟和外部 API 消耗。

```python
import json
import time
from pathlib import Path

class SearchCache:
    """简单文件缓存"""

    def __init__(self, cache_dir: str = "/tmp/search_cache", ttl: int = 3600):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(parents=True, exist_ok=True)
        self.ttl = ttl

    def _cache_path(self, key: str) -> Path:
        safe = key.replace("/", "_").replace(" ", "_")[:100]
        return self.cache_dir / f"{safe}.json"

    def get(self, query: str) -> list[dict] | None:
        path = self._cache_path(query)
        if not path.exists():
            return None
        try:
            data = json.loads(path.read_text())
            if time.time() - data["ts"] < self.ttl:
                return data["results"]
        except (json.JSONDecodeError, KeyError):
            pass
        return None

    def set(self, query: str, results: list[dict]):
        path = self._cache_path(query)
        path.write_text(json.dumps({
            "ts": time.time(),
            "query": query,
            "results": results
        }, ensure_ascii=False))
```

## 安全过滤

中国服务器搜索结果必须安全合规，三层过滤。

```python
import re

# 黑名单关键词
BLOCKED_KEYWORDS = [
    r"暴力", r"色情", r"赌博", r"毒品", r"枪支",
    r"翻墙", r"VPN.*破解", r"盗版",
]

def filter_results(results: list[dict]) -> list[dict]:
    """过滤不安全搜索结果"""
    safe = []
    for r in results:
        text = f"{r.get('title', '')} {r.get('snippet', '')}"
        if any(re.search(kw, text) for kw in BLOCKED_KEYWORDS):
            continue
        safe.append(r)
    return safe
```

## LLM 综合回答

将搜索结果注入 LLM 上下文，带来源标签生成回答。

```python
SEARCH_SYSTEM_PROMPT = """你是一个搜索增强型 AI 助手。

当你需要搜索时，你会自动调用搜索工具获取最新信息。
在回答中，对关键信息可以自然带出来源标签（如「据 CSDN/官方文档」）。

回答原则：
1. 结论先行，简洁有力
2. 关键事实标注来源
3. 多个来源冲突时说明分歧
4. 不确定的信息标注「据网络信息」
"""

def build_search_context(results: list[dict]) -> str:
    """将搜索结果构建为上下文"""
    if not results:
        return ""

    ctx = "## 搜索结果\n\n"
    for i, r in enumerate(results, 1):
        source_icon = {"anysearch": "🔍", "bing": "🌐", "360": "🔎"}.get(r.get("source", ""), "📄")
        ctx += f"{i}. {source_icon} [{r.get('title', '无标题')}]({r.get('url', '#')})\n"
        ctx += f"   {r.get('snippet', '')}\n"
        ctx += f"   *来源: {r.get('source', 'unknown')}*\n\n"
    return ctx
```

## 搜索策略路由

根据查询类型选择不同搜索策略。

```python
def search_strategy(query: str, cache: SearchCache = None) -> list[dict]:
    """智能搜索路由"""
    # 1. 查缓存
    if cache:
        cached = cache.get(query)
        if cached:
            return cached

    # 2. 决定搜索策略
    query_lower = query.lower()
    is_technical = any(kw in query_lower for kw in
                       ["how to", "bug", "error", "install", "api", "code", "python", "javascript"])
    is_current = any(kw in query_lower for kw in
                     ["news", "latest", "today", "2026", "price", "release"])

    if is_technical:
        # 技术查询：多引擎并行，提高覆盖率
        engines = [search_anysearch, search_bing_cn]
    elif is_current:
        # 时事查询：AnySearch 优先，Bing 补充
        engines = [search_anysearch, search_bing_cn]
    else:
        # 通用查询：单引擎够用
        engines = [search_anysearch]

    results = parallel_search(query, engines=engines)
    results = filter_results(results)
    results = fuse_results(results)

    # 3. 写缓存
    if cache and results:
        cache.set(query, results)

    return results
```

## 常见陷阱

| # | 陷阱 | 解决方案 |
|---|------|----------|
| 1 | AnySearch 响应格式误判 | 结果在 `data.results` 非顶层 `results` |
| 2 | Bing HTML 结构变更 | 加 `try/except`，降级到纯文本解析 |
| 3 | 360 搜索反爬 | 设置合理 User-Agent + 5s 以上间隔 |
| 4 | 搜索结果含不安全内容 | 三层过滤：前端 JS + 后端 content_safety + 结果过滤 |
| 5 | 缓存穿透/雪崩 | TTL 随机化 ±20%，热点 key 加锁 |
| 6 | 并行搜索超时 | ThreadPoolExecutor 设 15s 超时，失败不阻塞 |
| 7 | 中国服务器无法 Google | 用 Bing 中文站 + 360 + AnySearch 组合替代 |
| 8 | LLM 编造来源标签 | System prompt 明确要求「不确定标注来源」 |

## 验证

```bash
# 测试 AnySearch
curl -s "https://api.anysearch.com/v1/search" \
  -H "Authorization: Bearer $ANYSEARCH_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"AI Agent 最佳实践","count":3}' | python3 -m json.tool

# 测试 Bing 中文搜索
curl -s "https://cn.bing.com/search?q=AI+Agent+框架" | python3 -c "
import sys, html
from bs4 import BeautifulSoup
soup = BeautifulSoup(sys.stdin.read(), 'html.parser')
for li in soup.select('.b_algo')[:3]:
    title = li.select_one('h2 a')
    if title: print(title.get_text(strip=True))
"

# 测试缓存
python3 -c "
from web_search import SearchCache
c = SearchCache()
c.set('test', [{'title': 'test'}])
print(c.get('test'))
"
```

## 依赖

```bash
pip install requests beautifulsoup4 lxml
# 可选：AnySearch API Key（免费无限用）
# 注册：https://anysearch.com/console/api-keys
```
