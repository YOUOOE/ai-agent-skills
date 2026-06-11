---
name: security-review-agent
description: 上线前安全审查 — 认证、密钥、权限、支付安全、数据隔离全覆盖
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [security, audit, penetration-test, safe-deployment]
difficulty: advanced
---

# Security Review Agent

## 概述

安全是上线的最后一道关。本 Skill 提供系统化的安全审查流程，覆盖登录认证、密钥管理、支付安全、数据隔离、XSS/SQL 注入、Nginx 配置等常见安全问题。

## 七大审查域

### 1. 认证与授权

```python
AUTH_CHECKS = [
    ("密码存储", "使用 bcrypt/argon2 哈希，不是明文或 MD5"),
    ("API 认证", "每个敏感端点必须验证 api_key 或 JWT"),
    ("会话过期", "Token 有效期不超过 24h，过期需重新登录"),
    ("登录限流", "同一 IP 5 分钟内失败 5 次应临时封禁"),
    ("权限分级", "普通用户不能调用管理员接口"),
    ("CSRF", "敏感操作验证来源 Referer 或 Token"),
]
```

### 2. 密钥管理

```python
KEY_CHECKS = [
    ("硬编码检测", "扫描代码中的 api_key/password/token 关键字"),
    ("环境变量", "所有密钥必须用环境变量，不提交到 Git"),
    ("占位符检测", "检测 sk-xxx...xxx / *** 等无效占位符"),
    ("日志泄漏", "环境变量不应出现在日志或错误输出中"),
]
```

### 3. 支付安全

```python
PAYMENT_CHECKS = [
    ("金额防篡改", "金额在服务端计算，不接受前端传的金额"),
    ("回调验证", "微信支付回调必须验证签名和订单号"),
    ("重复支付", "同一订单号不能重复使用"),
    ("对账机制", "每日自动对账：收入 = 充值 - 退款"),
    ("防刷", "同一用户短时间不能大量创建订单"),
]
```

### 4. 数据隔离

```python
DATA_CHECKS = [
    ("跨用户访问", "用户 A 不能访问用户 B 的数据"),
    ("路径遍历", "文件操作必须过滤 ../ 和绝对路径"),
    ("SQL 注入", "所有 SQL 查询必须用参数绑定，不能拼字符串"),
    ("XSS", "用户输入输出到 HTML 必须转义"),
    ("数据脱敏", "日志中不能记录用户密码/完整手机号"),
]
```

### 5. 基础设施

```python
INFRA_CHECKS = [
    ("端口绑定", "内部 API 绑定 127.0.0.1 而非 0.0.0.0"),
    ("Nginx 安全头", "X-Content-Type-Options / X-Frame-Options / CSP"),
    ("速率限制", "API 端点有速率限制（30r/s，登录 5r/m）"),
    ("HTTPS", "全站 HTTPS，HSTS 头"),
    ("访问控制", "非中国 IP 应拦截（针对中国业务）"),
]
```

### 6. 安全头配置

```nginx
# Nginx 安全头示例
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' cdn.example.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'" always;
```

### 7. 自动化安全扫描

```python
#!/usr/bin/env python3
"""security_scanner.py — 自动化安全扫描"""

import re, os, json

SECURITY_PATTERNS = {
    "api_key_hardcoded": r'(sk-|pk-|AIza)[a-zA-Z0-9_-]{10,}',
    "password_hardcoded": r'(password|passwd|pwd)\s*[:=]\s*["\'][^"\']+["\']',
    "sql_injection": r'f["\'].*\{.*\}.*(SELECT|INSERT|DELETE|UPDATE)',
    "internal_ip": r'(127\.0\.0\.1|192\.168\.|10\.\d+\.\d+\.\d+)',
    "secret_path": r'(/\.env|/wp-admin|/\.git|/config\.)',
}

def scan_file(filepath):
    issues = []
    with open(filepath, 'r', errors='ignore') as f:
        content = f.read()
    
    for name, pattern in SECURITY_PATTERNS.items():
        if re.search(pattern, content):
            for match in re.finditer(pattern, content):
                line_no = content[:match.start()].count('\n') + 1
                issues.append({
                    "file": filepath,
                    "line": line_no,
                    "type": name,
                    "match": match.group()[:50],
                })
    return issues
```

## 审查报告模板

```markdown
# 安全审查报告

## P0 — 必须修复（上线阻塞）
- [ ] ...

## P1 — 建议修复（上线前完成）
- [ ] ...

## P2 — 长期优化
- [ ] ...
```

## 常见陷阱

### ⚠️ 只看代码不看配置
- 代码很安全，但 Nginx 暴露了内部端口
- 修复：安全审查包括 Nginx、systemd、Docker 配置

### ⚠️ 静态 Token 无过期
- 管理后台用永久 Token → 泄漏后无法收回
- 修复：用 JWT + 24h 过期 + 刷新机制

### ⚠️ 以为本地安全就是线上安全
- 本地用 HTTP，线上也是 HTTP → 中间人攻击
- 修复：线上强制 HTTPS

## 验证清单

- [ ] 无 API Key/密码硬编码
- [ ] 所有 API 端点有认证
- [ ] 支付金额服务端计算
- [ ] 用户数据隔离
- [ ] SQL 参数绑定
- [ ] Nginx 安全头配置
- [ ] 速率限制已生效
- [ ] 内部端口已绑定 127.0.0.1
