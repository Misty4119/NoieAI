# NEWS_DECAY.md

## News Domain Knowledge Decay Law

### Domain Characteristics

| Feature | Description |
|---------|-------------|
| **Information Metabolism Rate** | Extremely high |
| **Immediacy** | Extremely strong time sensitivity |
| **Fact Stability** | Facts may quickly become outdated |
| **Source Diversity** | Reliability varies widely |

### Decay Constants

$$\lambda^*_{\text{news}} \approx 0.8 - 1.0$$

### News Sub-Type Decay Characteristics

| Sub-Type | λ* Range | Half-Life |
|---------|-----------|-----------|
| Breaking News | 0.9 - 1.0 | Several hours |
| Same-Day News | 0.7 - 0.9 | 1-3 days |
| Feature Reports | 0.5 - 0.7 | 1-2 weeks |
| Commentary/Analysis | 0.3 - 0.5 | 1-3 months |
| In-Depth Investigations | 0.2 - 0.4 | Months to years |

### Decay Trigger Conditions

```python
NEWS_DECAY_TRIGGERS = [
    "New event developments",
    "Fact clarifications/corrections",
    "Related news reversals",
    "Official statements released",
    "Time elapsed beyond threshold"
]
```

### Calculation Formula

```python
FUNCTION ComputeNewsValidity(claim, current_intrinsic_clock):
    clock_delta = current_intrinsic_clock - claim.ν_stamp
    decay = exp(-lambda_news * clock_delta)
    source_reliability = GetSourceReliability(claim.source)
    RETURN min(decay * source_reliability * claim.corroboration_factor, 1.0)
```
