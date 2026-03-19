# NEWS_DECAY.md

## 新聞領域知識衰減律

### 領域特性

| 特性 | 描述 |
|------|------|
| **資訊代謝率** | 極高 |
| **即時性** | 時間敏感性極強 |
| **事實穩定性** | 事實可能快速過時 |
| **來源多樣性** | 可靠度差異大 |

### 衰減常數

$$\lambda^*_{\text{news}} \approx 0.8 - 1.0$$

### 新聞子類型衰減特徵

| 子類型 | λ* 範圍 | 半衰期 |
|--------|---------|--------|
| 即時新聞 | 0.9 - 1.0 | 數小時 |
| 當日新聞 | 0.7 - 0.9 | 1-3 天 |
| 專題報導 | 0.5 - 0.7 | 1-2 週 |
| 評論分析 | 0.3 - 0.5 | 1-3 月 |
| 深度調查 | 0.2 - 0.4 | 數月-數年 |

### 衰減觸發條件

```python
NEWS_DECAY_TRIGGERS = [
    "新事件發展",
    "事實澄清/更正",
    "相關新聞反轉",
    "官方聲明發布",
    "時間流逝超過閾值"
]
```

### 計算公式

```python
FUNCTION ComputeNewsValidity(claim, current_intrinsic_clock):
    clock_delta = current_intrinsic_clock - claim.ν_stamp
    decay = exp(-lambda_news * clock_delta)
    source_reliability = GetSourceReliability(claim.source)
    RETURN min(decay * source_reliability * claim.corroboration_factor, 1.0)
```
