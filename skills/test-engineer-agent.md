---
name: test-engineer-agent
description: 关键路径测试 — 按风险优先级补测试，确保支付/认证/核心功能不崩
platforms: [hermes, claude-code, cursor, windsurf, openclaw]
tags: [testing, quality-assurance, pytest, test-coverage]
difficulty: intermediate
---

# Test Engineer Agent

## 概述

测试不是追求 100% 覆盖率，而是用最少的测试覆盖最关键的风险点。本 Skill 提供一套按风险优先级的测试策略：P0（有钱相关）> P1（核心功能）> P2（边界异常）。

## 测试优先级策略

```python
TEST_PRIORITIES = {
    "P0": {
        "label": "💀 立即阻塞",
        "examples": ["支付流程", "用户登录", "余额查询", "数据写入"],
    },
    "P1": {
        "label": "⚠️ 强烈推荐",
        "examples": ["核心功能主路径", "API 200 验证", "数据库读写"],
    },
    "P2": {
        "label": "📝 建议补充",
        "examples": ["空数据处理", "网络超时", "边界值", "并发访问"],
    },
}
```

## P0 测试模板

### 支付全链路测试

```python
"""test_payment.py — 支付是底线，必须全链路覆盖"""

def test_wechat_pay_flow():
    """微信支付：创建订单 → 获取二维码 → 回调 → 余额更新"""
    # 1. 创建订单
    resp = client.post("/api/create_topup_order", json={
        "amount": 10, "payment": "wechat"
    })
    assert resp.status_code == 200
    order_id = resp.json["order_id"]
    assert order_id.startswith("ORD_")
    
    # 2. 模拟微信回调
    resp = client.post("/api/wechat/pay_notify", json={
        "order_id": order_id,
        "amount": 10,
        "status": "success"
    })
    assert resp.status_code == 200
    
    # 3. 验证余额更新
    resp = client.get(f"/api/balance?order_id={order_id}")
    assert resp.status_code == 200
    assert resp.json["balance"] >= 10
```

### 用户认证测试

```python
"""test_auth.py — 认证绕过等于没锁门"""

def test_login_flow():
    """注册 → 登录 → API 调用 → 退出"""
    # 注册
    resp = client.post("/api/user/register", json={
        "account": "test@test.com",
        "password": "test123456"
    })
    assert resp.status_code == 200
    assert "api_key" in resp.json
    
    api_key = resp.json["api_key"]
    
    # 用 api_key 调用受保护接口
    resp = client.get("/api/user/profile",
        headers={"X-API-Key": api_key})
    assert resp.status_code == 200

def test_auth_bypass():
    """未登录不能访问受保护接口"""
    resp = client.get("/api/user/profile")
    assert resp.status_code in (401, 403)
```

## P1 测试模板

### API 健康测试

```python
"""test_api.py — 核心 API 端点验证"""

def test_api_health():
    """所有核心端点返回 200"""
    endpoints = [
        ("GET", "/health"),
        ("GET", "/api/pricing"),
        ("GET", "/api/community/posts"),
    ]
    for method, path in endpoints:
        resp = getattr(client, method.lower())(path)
        assert resp.status_code == 200, f"{method} {path} failed"
```

### 数据库读写测试

```python
"""test_db.py — 数据层验证"""

def test_user_crud():
    """用户 CRUD 完整链路"""
    # Create
    user = db.create_user(email="test@test.com")
    assert user.id is not None
    
    # Read
    found = db.get_user(user.id)
    assert found.email == "test@test.com"
    
    # Update
    updated = db.update_user(user.id, {"name": "Test"})
    assert updated.name == "Test"
    
    # Delete
    db.delete_user(user.id)
    assert db.get_user(user.id) is None
```

## 测试环境配置

```python
"""conftest.py — 测试基础设施"""

import pytest
from fastapi.testclient import TestClient
from your_app import app

@pytest.fixture
def client():
    """测试客户端"""
    return TestClient(app)

@pytest.fixture
def test_db():
    """测试数据库（内存 SQLite，不影响生产）"""
    import sqlite3
    conn = sqlite3.connect(":memory:")
    yield conn
    conn.close()
```

## 常见陷阱

### ⚠️ 只测成功路径不测异常
- 用户付款成功测了，但付款失败/超时/重复回调没测
- 修复：每个成功路径配一个失败路径

### ⚠️ 测试数据库和生产混用
- 测试写入生产库 → 数据污染
- 修复：测试用独立数据库或内存数据库

### ⚠️ 跑测试前不清理数据
- 测试间互相依赖 → 先跑 A 再跑 B 能过，单独跑 B 就崩
- 修复：每个测试独立清理/创建数据

## 验证清单

- [ ] P0 测试（支付/认证）全部通过
- [ ] P1 测试（核心 API/CRUD）全部通过
- [ ] 测试不依赖外部服务（Mock API 调用）
- [ ] 测试数据库独立于生产环境
- [ ] 每个测试独立运行不互相依赖
- [ ] 失败时有清晰的断言信息
