# 测试工程师方法论

补最关键路径的测试，不是追求覆盖率，而是追求"最值得测的地方都测了"。

## 关键路径判断标准

哪种代码最值得写测试？

1. **核心业务逻辑** — 用户付钱的流程、核心算法的输入输出
2. **高变更频率** — 经常改的代码，测了防止回归
3. **高复杂度** — 条件分支多、嵌套深、异步调用多
4. **安全敏感** — 认证、权限、支付、数据完整性
5. **历史Bug多发区** — 同一个文件修了多次的地方

## 测试金字塔

```
     ╱╲           E2E (少量)
    ╱  ╲          端到端测试核心流程
   ╱    ╲
  ╱──────╲         Integration (适中)
 ╱         ╲       接口/API集成测试
╱───────────╲
  Unit (大量)      单元测试(纯函数/工具类)
```

## 测试结构

```python
# 命名：test_<模块>_<场景>.py
def test_calculate_price_normal():
    """正常情况：¥10商品+正常汇率"""
    result = calculate_price(10, rate=7.2)
    assert result == 72

def test_calculate_price_zero():
    """边界：价格为0"""
    result = calculate_price(0, rate=7.2)
    assert result == 0

def test_calculate_price_negative():
    """异常：负价格"""
    with pytest.raises(ValueError):
        calculate_price(-1, rate=7.2)
```

## TDD 快速模式
```
红灯(写失败测试) → 绿灯(写最少代码通过) → 重构(优化代码)
循环直到功能完整
```

## 经验法则
- 一个Bug修完后，先写一个重现该Bug的测试，再修
- API测试保证关键端点返回200+正确数据格式
- 前端测试保证关键交互不报错
