# CONTINUITY_INDEX.md

## 連續性指標計算

### 定義

連續性指標衡量推論鏈中相鄰步驟之間的語義距離。

### 實現

```python
FUNCTION ComputeContinuityIndex(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    
    if len(steps) < 2:
        return 1.0
    
    # 計算每對相鄰步驟之間的測地距離
    gaps = []
    for i in range(len(steps) - 1):
        gap = ComputeGeodesicDistance(steps[i], steps[i+1])
        gaps.append(gap)
    
    # 計算最大間隙
    max_gap = max(gaps)
    
    # 計算連續性指標
    continuity_index = 1 / (1 + max_gap)
    
    return ContinuityIndex(
        value=continuity_index,
        max_gap=max_gap,
        steps=len(steps)
    )
```

### 閾值

| EC 等級 | 閾值 |
|---------|------|
| L0-L2 | 0.9 |
| L3-L4 | 0.7 |
| L5-L6 | 0.5 |
