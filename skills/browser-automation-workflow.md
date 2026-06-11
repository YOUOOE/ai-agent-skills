---
name: browser-automation-workflow
description: AI 驱动的浏览器自动化方案 — 网页操作、数据提取、表单填写、视觉验证
platforms: [hermes, cursor, claude-code, windsurf]
tags: [browser, automation, scraping, web, testing]
difficulty: intermediate
---

# Browser Automation Workflow

## 概述

AI Agent 操作浏览器的能力让它可以处理很多「人类操作」场景：登录网站、填表单、抓数据、截图验证。本 Skill 提供一套稳定的浏览器自动化工作流。

## 架构

```
AI Agent
  │
  ├─ navigate → 打开页面
  ├─ snapshot → 获取页面交互元素 (可点击/可输入)
  ├─ click/type → 操作元素
  ├─ scroll → 滚动加载更多
  └─ vision → 视觉截图分析
```

## 实现

### 1. 导航到页面

```python
async def navigate(url: str) -> dict:
    """打开 URL 并获取页面摘要"""
    # 初始化浏览器（如无头 Chrome）
    page = await browser.new_page()
    await page.goto(url, wait_until="networkidle")
    
    # 获取页面摘要（交互元素列表）
    snapshot = await page.accessibility.snapshot()
    
    return {
        "url": page.url,
        "title": await page.title(),
        "snapshot": _simplify_snapshot(snapshot),
        "interactive_elements": _get_interactive_elements(page)
    }
```

### 2. 智能元素定位

```python
def find_element(snapshot, target_text, element_type="button"):
    """智能定位目标元素"""
    candidates = []
    
    def _search(node, depth=0):
        if depth > 10:
            return
        label = node.get("name", "").lower()
        role = node.get("role", "")
        
        if target_text.lower() in label:
            if element_type == "button" and role in ("button", "link"):
                candidates.append(node)
            elif element_type == "input" and role == "textbox":
                candidates.append(node)
            elif element_type == "link" and role == "link":
                candidates.append(node)
        
        for child in node.get("children", []):
            _search(child, depth + 1)
    
    _search(snapshot)
    return candidates[0] if candidates else None
```

### 3. 截图 + 视觉分析

```python
async def screenshot_and_analyze(page, question=""):
    """截图并用视觉 AI 分析页面内容"""
    # 截取当前页面
    screenshot = await page.screenshot(full_page=True)
    
    # 保存到临时文件
    path = f"/tmp/screenshot_{int(time.time())}.png"
    with open(path, "wb") as f:
        f.write(screenshot)
    
    if question:
        # 调用视觉分析
        analysis = await vision_analyze(path, question)
        return {"path": path, "analysis": analysis}
    
    return {"path": path}
```

### 4. 表单填写模式

```python
async def fill_form(page, fields: dict):
    """智能填写表单"""
    for label, value in fields.items():
        # 查找 label 对应的输入框
        element = await page.query_selector(f'input[id="{label}"]')
        if not element:
            element = await page.query_selector(f'input[name="{label}"]')
        if not element:
            element = await page.query_selector(f'input[placeholder="{label}"]')
        
        if element:
            await element.fill(value)
        else:
            print(f"⚠️ 找不到字段: {label}")
```

### 5. 常见场景模板

```python
SCENARIOS = {
    "login": [
        ("navigate", "https://example.com/login"),
        ("type", {"input[name='email']": "user@example.com"}),
        ("type", {"input[name='password']": "password123"}),
        ("click", "button[type='submit']"),
        ("wait", "5000"),
        ("verify", "用户已登录"),
    ],
    "search_and_extract": [
        ("navigate", "https://example.com/search?q=AI+Agent"),
        ("wait", ".result-item"),
        ("extract", ".result-item .title, .result-item .url"),
        ("scroll_to_bottom", ""),
        ("extract_more", ".pagination .next"),
    ],
    "form_submit": [
        ("navigate", "https://example.com/submit"),
        ("fill_form", {"name": "...", "email": "...", "message": "..."}),
        ("click", "button:has-text('提交')"),
        ("verify_success", ".success-message"),
    ],
}
```

## 常见陷阱

### ⚠️ 等待不够
- 页面是 SPA 动态渲染， DOM 元素存在但内容为空
- 修复：用 `wait_for_selector` 或 `wait_for_function`

### ⚠️ 弹窗/验证码
- 登录页面有图形验证码
- 修复：视觉分析识别 → 或检测到验证码时提示用户介入

### ⚠️ 无 headless 检测
- 部分网站检测 headless Chrome 并拦截
- 修复：添加 `--disable-blink-features=AutomationControlled` 参数

## 验证清单

- [ ] 可以成功导航到目标 URL
- [ ] 能正确识别页面上的交互元素
- [ ] 表单填写准确（字段匹配正确）
- [ ] 截图清晰可用于视觉分析
- [ ] 处理了弹窗/验证码等特殊情况
- [ ] 超时30s后有明确错误反馈
