# EXPLORATION_FRONTIER.md

## 探索邊界管理

### 定義

探索邊界是知識流形中已知與未知的交界區域。

### 實現

```python
FUNCTION ManageExplorationFrontier(knowledge_base):
    
    # 識別邊界
    frontier = IdentifyBoundary(knowledge_base)
    
    # 評估每個邊界點的價值
    frontier_values = []
    for point in frontier:
        value = ComputeExplorationValue(point)
        frontier_values.append({
            "point": point,
            "value": value,
            "difficulty": EstimateDifficulty(point)
        })
    
    # 排序
    sorted_frontier = sort(frontier_values, key=lambda x: x.value)
    
    return ExplorationPlan(
        frontier=sorted_frontier,
        priority_targets=sorted_frontier[:K]
    )
```
