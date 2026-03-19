# EXPLORATION_FRONTIER.md

## 探索境界管理

### 定義

探索境界とは、知識多様体において既知と未知の交界領域である。

### 実装

```python
FUNCTION ManageExplorationFrontier(knowledge_base):
    
    # 境界の識別
    frontier = IdentifyBoundary(knowledge_base)
    
    # 各境界点の価値を評価
    frontier_values = []
    for point in frontier:
        value = ComputeExplorationValue(point)
        frontier_values.append({
            "point": point,
            "value": value,
            "difficulty": EstimateDifficulty(point)
        })
    
    # ソート
    sorted_frontier = sort(frontier_values, key=lambda x: x.value)
    
    return ExplorationPlan(
        frontier=sorted_frontier,
        priority_targets=sorted_frontier[:K]
    )
```
