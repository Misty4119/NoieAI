# CONTINUITY_INDEX.md

## Continuity Index Calculation

### Definition

The continuity index measures the semantic distance between adjacent steps in an inference chain.

### Implementation

```python
FUNCTION ComputeContinuityIndex(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    
    if len(steps) < 2:
        return 1.0
    
    # Compute geodesic distance between each pair of adjacent steps
    gaps = []
    for i in range(len(steps) - 1):
        gap = ComputeGeodesicDistance(steps[i], steps[i+1])
        gaps.append(gap)
    
    # Compute maximum gap
    max_gap = max(gaps)
    
    # Compute continuity index
    continuity_index = 1 / (1 + max_gap)
    
    return ContinuityIndex(
        value=continuity_index,
        max_gap=max_gap,
        steps=len(steps)
    )
```

### Thresholds

| EC Level | Threshold |
|----------|-----------|
| L0-L2 | 0.9 |
| L3-L4 | 0.7 |
| L5-L6 | 0.5 |
