# CONTINUITY_INDEX.md

## 連続性指標計算

### 定義

連続性指標は推論チェーン中の隣接ステップ間の意味距離を度量する。

### 実装

```python
FUNCTION ComputeContinuityIndex(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    
    if len(steps) < 2:
        return 1.0
    
    # 各隣接ステップ間の測地距離を計算
    gaps = []
    for i in range(len(steps) - 1):
        gap = ComputeGeodesicDistance(steps[i], steps[i+1])
        gaps.append(gap)
    
    # 最大ギャップを計算
    max_gap = max(gaps)
    
    # 連続性指標を計算
    continuity_index = 1 / (1 + max_gap)
    
    return ContinuityIndex(
        value=continuity_index,
        max_gap=max_gap,
        steps=len(steps)
    )
```

### 閾値

| EC等レベル | 閾値 |
|---------|------|
| L0-L2 | 0.9 |
| L3-L4 | 0.7 |
| L5-L6 | 0.5 |
