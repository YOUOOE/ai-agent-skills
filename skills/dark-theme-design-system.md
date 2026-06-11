---
name: dark-theme-design-system
description: 完整的暗色主题设计系统 — 从 CSS 变量体系、主题色盘到深色/浅色双模式切换
platforms: [hermes, cursor, claude-code, windsurf]
tags: [design, css, theming, ui, dark-mode, frontend]
difficulty: beginner
---

# Dark Theme Design System

## 概述

暗色主题不是简单的把背景变黑文字变白。本 Skill 提供一个经过生产验证的完整暗色主题设计系统，覆盖 CSS 变量体系、双模式切换、主题色管理、SVG 图标集成。

## CSS 变量体系

### 1. 核心变量定义

```css
:root {
  /* 基础色板 */
  --bg: #0d0d12;         /* 最深背景 */
  --bg2: #141418;        /* 面板/卡片背景 */
  --bg3: #1a1a20;        /* 悬浮/输入框背景 */
  --bd: #2a2a32;         /* 边框颜色 */
  --bd-hover: #3a3a46;   /* 悬浮边框 */
  
  /* 文字层级 */
  --txt: #e8e8ed;        /* 主文字 */
  --txt2: #9c9ca8;       /* 次要文字 */
  --txt3: #6b6b78;       /* 禁用/占位文字 */
  
  /* 主色 */
  --accent: #01c2c3;     /* 主色（青蓝） */
  --accent-hover: #2dd4d4;
  --accent-dim: rgba(1, 194, 195, 0.1);
  
  /* 语义色 */
  --green: #22c55e;
  --red: #ef4444;
  --yellow: #eab308;
  --blue: #3b82f6;
  
  /* 间距 */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 12px;
  --space-lg: 16px;
  --space-xl: 24px;
  
  /* 圆角 */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 14px;
  
  /* 字体 */
  --font-sans: 'PingFang SC', -apple-system, 'Helvetica Neue', sans-serif;
  --font-mono: 'JetBrains Mono', 'Consolas', monospace;
  
  /* 阴影 */
  --shadow-card: 0 1px 3px rgba(0,0,0,0.3);
  --shadow-elevated: 0 4px 12px rgba(0,0,0,0.4);
}
```

### 2. 浅色主题覆盖

```css
[data-theme="light"] {
  --bg: #f5f5f7;
  --bg2: #ffffff;
  --bg3: #f0f0f2;
  --bd: #e5e5ea;
  --bd-hover: #d1d1d6;
  --txt: #1c1c1e;
  --txt2: #636366;
  --txt3: #aeaeb2;
  --accent: #10b981;
  --accent-hover: #059669;
  --accent-dim: rgba(16, 185, 129, 0.08);
  --shadow-card: 0 1px 3px rgba(0,0,0,0.08);
  --shadow-elevated: 0 4px 12px rgba(0,0,0,0.12);
}
```

### 3. 多主题色支持

```css
/* 例如：青色（默认）、绿色、紫色、琥珀 */
[data-accent="green"] {
  --accent: #22c55e;
  --accent-hover: #16a34a;
  --accent-dim: rgba(34, 197, 94, 0.1);
}

[data-accent="purple"] {
  --accent: #8b5cf6;
  --accent-hover: #7c3aed;
  --accent-dim: rgba(139, 92, 246, 0.1);
}

[data-accent="amber"] {
  --accent: #f59e0b;
  --accent-hover: #d97706;
  --accent-dim: rgba(245, 158, 11, 0.1);
}
```

## 切换逻辑

### 4. 主题切换器

```html
<button id="themeToggle" onclick="toggleTheme()">
  <!-- 暗色：显示月亮，亮色：显示太阳 -->
  <svg id="themeIcon" width="20" height="20" viewBox="0 0 24 24"
       fill="none" stroke="currentColor" stroke-width="2">
    <!-- 太阳 (light) -->
    <path id="sunPath" d="M12 3v1m0 16v1m-9-9H2m20 0h-1M4.22 4.22l.7.7m14.16 14.16l.7.7M4.22 19.78l.7-.7m14.16-14.16l.7-.7"/>
    <circle id="sunCircle" cx="12" cy="12" r="5"/>
    <!-- 月亮 (dark) -->
    <path id="moonPath" d="M12 3a9 9 0 000 18 7 7 0 010-14z" display="none"/>
  </svg>
</button>

<script>
function toggleTheme() {
  const html = document.documentElement;
  const current = html.getAttribute('data-theme') || 'dark';
  const next = current === 'dark' ? 'light' : 'dark';
  html.setAttribute('data-theme', next);
  localStorage.setItem('theme', next);
  updateThemeIcon(next);
}

function updateThemeIcon(theme) {
  document.getElementById('sunPath').style.display =
    theme === 'light' ? 'block' : 'none';
  document.getElementById('sunCircle').style.display =
    theme === 'light' ? 'block' : 'none';
  document.getElementById('moonPath').style.display =
    theme === 'dark' ? 'block' : 'none';
}

// 初始化
(function() {
  const saved = localStorage.getItem('theme') ||
    (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  document.documentElement.setAttribute('data-theme', saved);
  updateThemeIcon(saved);
})();
</script>
```

### 5. 主题色切换

```javascript
function setAccent(color) {
  document.documentElement.setAttribute('data-accent', color);
  localStorage.setItem('accent', color);
}

// 初始化
const savedAccent = localStorage.getItem('accent') || 'cyan';
document.documentElement.setAttribute('data-accent', savedAccent);
```

## 组件模板

### 6. 卡片组件

```css
.card {
  background: var(--bg2);
  border: 1px solid var(--bd);
  border-radius: var(--radius-lg);
  padding: var(--space-lg);
  transition: border-color 0.2s;
}

.card:hover {
  border-color: var(--bd-hover);
}

/* 三态按钮 */
.btn {
  background: var(--accent);
  color: white;
  border: none;
  border-radius: var(--radius-md);
  padding: var(--space-sm) var(--space-lg);
  font-size: 14px;
  cursor: pointer;
  transition: background 0.15s;
}

.btn:hover {
  background: var(--accent-hover);
}

.btn:active {
  transform: scale(0.97);
}
```

### 7. SVG 图标集成

```svg
<!-- 自托管 SVG 图标，不依赖 Font Awesome CDN -->
<svg class="icon" width="18" height="18" viewBox="0 0 24 24"
     fill="none" stroke="currentColor" stroke-width="2">
  <path d="M12 5v14M5 12h14"/>
</svg>
```

## 常见陷阱

### ⚠️ 硬编码颜色
- CSS 里写了 `color: #fff` 而不是 `var(--txt)`
- 修复：所有颜色必须通过 CSS 变量引用

### ⚠️ 忘记 border 适配
- 暗色模式边框可见，亮色模式太刺眼
- 修复：`--bd` 在暗色用 `#2a2a32`，亮色用 `#e5e5ea`

### ⚠️ 主题闪烁
- 页面加载先闪白再变暗
- 修复：在 `<head>` 内联 JS 优先读取 localStorage

### ⚠️ 多页面不一致
- 每个页面独立写切换逻辑 → 主题状态不同步
- 修复：封装成共享 `theme.js`，所有页面引用同一份

## 验证清单

- [ ] 暗色/亮色切换顺滑，无闪烁
- [ ] 所有颜色都走 CSS 变量
- [ ] 卡片、按钮、输入框在三态（normal/hover/active）下正确
- [ ] 手机端主题切换正常
- [ ] 切换后刷新页面，主题持久化
- [ ] SVG 图标颜色跟随主题（用 `currentColor`）
