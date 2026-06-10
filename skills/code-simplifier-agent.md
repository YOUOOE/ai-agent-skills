# 代码简化方法论

删除冗余、整理命名、理顺结构，不改变行为。

## 简化的信号

识别需要简化的代码：
1. **重复** — 同一模式出现≥3次 → 抽取函数/类
2. **过长** — 一个函数/方法>50行 → 拆分为多个小函数
3. **嵌套深** — if嵌套>3层 → 提前return/卫语句
4. **命名模糊** — 变量名像a/b/tmp/data/result → 用有业务含义的名字
5. **死代码** — 注释掉的代码/从未调用的函数 → 删除
6. **过度工程** — 为可能的需求写的抽象 → 恢复为直接实现

## 简化流程

### 1. 先理解完整逻辑
先把整个函数/文件读完，画数据流图
- 输入是什么？
- 每个中间步骤在做什么？
- 输出是什么？

### 2. 逐项简化
```
较长函数 → 按职责拆分为小函数
重复代码 → 抽取公共函数/参数化
深嵌套if → 卫语句提前返回
长条件 → 抽取为命名清晰的布尔变量
```

### 3. 验证行为不变
- 输入相同→输出相同
- 原有调用方不受影响
- 边界条件一致

## 示例

```python
# 简化前
def process(data):
    if data is not None:
        if 'status' in data:
            if data['status'] == 'active':
                result = do_something(data)
                return result
            else:
                return None
        else:
            return None
    else:
        return None

# 简化后
def process(data):
    if not data or data.get('status') != 'active':
        return None
    return do_something(data)
```

## 原则
- **不改变行为** — 简化≠重构，输入输出不变
- **一次只做一件事** — 改完一个点就验证
- **每个函数只做一件事** — 职责单一
